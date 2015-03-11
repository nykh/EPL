#EPL Lecture 12, 2015/3/3

## Project 2

`vsipl` and `BLAS` are some examples of linear algebra libraries. The project two is going to be about implementing efficient vectors that support these libraries.

for example, in `BLAS`  there is a function called `saxpy` that does thing like this:

```cpp
// saxpy
for(k=0; k < n; k += 1) {
  z[k] = a * x[k] + y[k];
}
```
**saxpy** = **Single-precicion** **A**\***X** **P**lus **Y**. Basically a building block for vector arithmetic.

The reason they have this kind of inflexible functions is the efficiency.

So how about if we have this kind of funtion

```cpp
vector<T> operator* (double a; const vector<T> &x) {
  vector<T> tmp;
  for(...) {
    tmp[k] = a * x[k];
  }
  return tmp;
}

vector<T> operator* (const vector<T> &x, const vector<T> &y) {
  vector<T> tmp;
  for(...) {
    tmp[k] = x[k] + y[k];
  }
  return tmp;
}
```

Notice however, all the assignment in the loop calls (hopefully) the move assignment, which caused another allocation. Even with **move** assignment, you have one extra allocation for each element, compared to the original function. In a highly performance-critical application, this is too much.

Another problem is that the compiler cannot parallelize the addition and multiplication, since in the formula

```cpp
z = a*x + y
```

the addition will have to wait for multiplication to finish. This leaves room for **software pipelining**.

### Goal
In project 2, we want to write code such that compiler convert the normal formula code into something like `saxpy` and increase performance.

## Lazy evaluation

We **postpone some evaluation** to a later point of program and make them happen there. In most programming languages, assignment `=` operation forces evaluation immediately because the type of the lvalue needs to be known before use.

## construct heterogenous tuple on the fly

Realize this snippet

**Note** plus (+) is **right-associative**.

```cpp
auto x = pack{} + 42 + "Hello world" + 3.14159;
```

### Implemenation

```cpp
struct pack{};

template <typename T>
one_field_tuple {
  T field;
};

template <typename T>
one_field_tuple<T>
opeerator+(pack, T x) {
  return one_field_tuple<T>{x};
}

template<typename T1, typename T2>
two_field_tuple {
  T1 field1; T2 field2;
}

template<typename T1, typename T2>
two_field_tuple<T1, T2>
operator+(const one_field_tuple<T1>& x, const T2& y) {
  return two_field_tuple<T1, T2>{ x.field, y };
}
```

This obviously doesn't scale. Instead we can do this

```cpp
struct pack{};

template <typename T, typename ... Tail>
struct tuple : public tuple<Tail...> {
  T x;
  tuple(T x_, const tuple<Tail...> &tail) : tuple<Tail...>(tail), x{x_} {}
};

// base case
template <typename T>
struct tuple {
  T x;
};

template <typename T>
tuple<T>
operator+(pack, constT& x) {
  return tuple<T>{x};
}

template <typename T, typename... Tail>
tuple<T, Tail...>
operator+(const tuple<Tail...> &x, constT& y) {
  return tuple<T, Tail...> { y, x };
}
```
this again result in compile time recursive behavior.

### Question: Why inheritance over composite?

Chase thinks that's a good question.

### Printing

```cpp
template <typename T>
void PrintFields(const tuple<T>& t) {
  cout << t.x;
}

template <typename T, typename... Args>
void PrintFields(const tuple<T, Args...> &t) {
  PrintFields<Args...> (t);
  cout << ", " << t.x;
} 
```

The problem is, C++ compiler doesn't seem to recognize pattern matching so smartly, possibly because it's a conflict with its functional overloading feature. What we want is this kind of functionality:

Let's read [this article](http://en.cppreference.com/w/cpp/language/parameter_pack) about parameter packing and pattern matching

```cpp
template <typename typename... Args>
void PrintFields(const tuple<Args...> &t) {
  using FirstType = ___;
  using TailType  = ___;
  PrintFields<Args...> (t);
  cout << ", " << t.x;
} 
```

We then need to write our own template type function to do this.

```cpp
template <typename T> struct head_type<T> { using type = T; };

template <typename T, typename... Tail>
struct<T, Tail...> { using type = T; };

template<typename T...> struct head_type;
```

This solves it for `head_type`, We can't however assign an alias to an elipsis type collectionn. We will probably have to go with making them a tuple.

```cpp
template <typname... T>
struct tail type.

template<typename T> struct tail_type<T> { using type = T; };

template<typename T, typename... Tail>
struct tail_type<T, Tail...> {
  using type = tuple<Tail...>;
};
```