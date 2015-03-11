#EPL Lecture 13, 2015/3/10

## Goal

Create mathematic function object that we can pass to some other object to execute. We want the following capability

- evaluate function
- arithmetic (+,-,\*,/)
- (optional) composition (the syntax should be f(g) )

## Protocal

```cpp
auto f = IdentityFunction{};
cout << f(21) << endl;          // we can evaluate functions
auto g = f + f;                 // we can add them
```

## Solution \#1

```cpp
struct IdentityFunction {
  double operator()(double n) { return n; }
  using result_type = double;
}

template<typename F, typename G>
struct AddFun {
  F f;
  G g;
  double operator()(double x) { return f(x) + g(x); }
  using result_type = double;
}

template<typename F, typename G>
operator+(F a, G b) {
  return AddFun<F, G>(a,b);
}
```

### Proxy

An object that takes the place of something else. Here we want to use proxy object to take the place of actual evaluated value, in order to defer evaluation. `AddFun` is one of these example.

The actual value will only be evaluated when we explicitly call the `f()` evaluation operation.

### Warning

We need to be careful about some unusual calling convention like

```cpp
auto h = f + 42
```

Because things like these will still match template signature with `G=int`, it can then try to call `int` which is a disaster. We don't want the function template to be so promiscuous.

We need everything we work with to satisfy the requirement of being a function.

### Concept

The above is an expression of a **concept**, which we use generic programming to enforce.

```cpp
struct IdentityFunction {
  double operator()(double x) { return x; }
  using result_type = double;
}

struct ConstantWrapper {
	double val;
    ConstantWrapper(int val_) : val(val_) {}
	double operator()(double) { return val; }
	using result_type = double;
}
```

But this is not enough, type conversion and template doesn't work well together as we have seen before. It seems we have to force the compiler to instantiate the template with `ConstWrapper` in mind.

```cpp
template <typename F>
AddFun<F, ConstWrapper> operator+(F f, int val) {
  return AddFun<F, ConstWrapper>(f, ConstantWrapper(val));
}
```

**HOWEVER this still sucks.** this only work for `int`. The compiler won't mix template parameter deduction and type conversion. Even worse, since `+` is a binary operator, we need to have copies of `operator+` both ways.

```cpp
template <typename F>
AddFun<F, ConstWrapper> operator+(F f, double val) {
  return AddFun<F, ConstWrapper>(f, ConstantWrapper(val));
}

template <typename F>
AddFun<F, ConstWrapper> operator+(int val, F f) {
  return AddFun<F, ConstWrapper>(f, ConstantWrapper(val));
}

template <typename F>
AddFun<F, ConstWrapper> operator+(double val, F f) {
  return AddFun<F, ConstWrapper>(f, ConstantWrapper(val));
}
```

This totally destroy the point of template, imo.

## Solution \#2

To really be flexible, we want to encode infromation into type to remove run-time `if`. A run-time `if...else` requires variables in both clauses to be syntactically correct.

```cpp
template <int ArgPos = 0, typename... ArgTypes>
struct select_arg;

template <typename FirstArg, typename...OtherArgs>
	struct select_arg<0, FirstArg, OtherArgs...> {
	using type = FirstArg;
	static FirstArg select(First x, OtherArgs...) { return x; }
}

template <int ArgPos, typename FirstArg, typename... OtherArgs>
struct select_arg<ArgPos-1, FirstArg, OtherArgs...> {
	using type = typename select_arg<ArgPos-1, FirstArg, OtherArgs...>::type;
    static type select(FirstArg, OtherArgs... args) {
    	return select_arg<ArgPos-1, OtherArgs...>::select(args...);
    }
}

template <int argPos, typename... Args>
typename select_arg<argPos, Args...>::type selectArg(Args... args) {
	return select_arg<argPos, Args...>::select(args...);
}
```

We can then use this like

```cpp
template <typename F, typename G>
struct AddProxy {
	F f;
	G g;
	double operator()(double x) { return f(x) + g(x); }
	double operator()(double x, double y) {
	return f(selectArg<F::argPos>(x, y))
           + g(selectArg<G::argPos>(x, y));
	}
};
```