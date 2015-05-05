#EPL Lecture 22, 2015/4/28

## function object

#### function pointer (C)
#### class with some standard interface (C++, Java)


```cpp
struct plus {
	T operator()(T x, T y) { return x + y; }
};
```

#### something else? Lambda

the problem with callable classes is it is hard/impossible to inline them.
What if however, we can refactor the following

```cpp
class border_cross_handler_thing {
	LifeForm *obj;
public:
	void operator()(void) {
    	obj->border_cross();
    }
};
```

using lambda expression

```cpp
[state](parameter){ function_body }
```

as

```cpp
[obj](void) { obj->border_cross(); }
```

The compiler will do this

1. generate a class with a `operator()` method, give it a random name
	1. it declares `std::function` as a base class, with the provided parameter and deduced return type information from the signature.
2. call the constructor for this class

### more use of lambda

```cpp
template <typename IT>
void sort(IT b, IT e) {
	using T = decltype(*b);
    using T2 = decltype(*e);
	static_assert(std::is_same<T, T2>::value);
    sort(b, e, [](T x, T2 y){ return x < y; });    // >>
}
```



