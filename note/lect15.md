#EPL Lecture 15, 2015/3/26

## Project 2

For reduction and accumulation. If there is only one element, you should return the element. If the vector is empty, you return `T{}`




## Metaprogramming

C\++ Metaprogramming is now very powerful but also very complex.

We start with STL, and wrote our first template metaprogram. `std::iterator_traits<IT>`.  We check the `value_type` and `iterator_category` returned by this metafunction. His opinion is that these conventions are not necessary today with the use of metafunctions.

Today we introduce alternative that doesn't require these conventions. The codes is uploaded in the svn folder

> example/Metaprogramming2/

### `auto` and `decltype`

Chase is skeptical still about `auto` and writing programs without knowing the type of variables in the programs. But it is useful sometimes.

`decltype(expr)` → the type of expression. With this metafunction actually we don't really need the `value_type` convention. The only problem is when we instantiate this template b might not exist yet.

```cpp
template <typename IT>
auto foo(IT b, IT, e) -> decltype(*b) {
	decltype(*b) m = *b;
	...    
}
```

#### Question: Why do we need the `->` syntax??

We cannot do
```cpp
decltype(*b) foo(IT b, IT, e) {
```

Because when you `decltype`, the variable `b` is not declared yet.

### how about `iterator_category`?

```cpp
template <typename IT>
auto distance(IT b, IT e, int) -> decltype(e - b) { return e - b; }
```

Now we consider this `e - b` expression. It is only possible (compiles without error) if the iterators are `random_access_iterator`. In Java it's not the case because their convention is to support everything, even when time complexity is not good. In the case where `b` and `e` are not random access iterators, `e-b` will simply be undefined, `decltype` will fail to return a type, this function signature won't even have a return type. **Note** pointers act like random access iterators, so pointers can be used in this function too.

### And other categories?

Used to be, we used the conventional `iterator_category` to create a tag to and **overloading** to select the correct version of functions (*random_access*, *bidirectional_iterator*, etc).

Let's add:

```cpp
template <typename IT>
int16_t distance(IT b, IT e) {
	int64_t d = 0;
    while(b != e) { ++b; d += i; }
    return d;
}
```

Because of **SFINAE** this will work for forward iterator.

### But wait! (Doesn't work)

Now that we have added the version of `distance` that works for forward iterator. The compiler will be confused about which to pick if given a `random_access` iterator. So **SFINAE** doesn't work in this case. We need something else. We want to tell the compiler to pick the better one when both are available, but how?

There is an obscure rule about **variadic function**

> If given **n** argument, a function that expects exactly n argument is picked over a variadic function that can have n element.

So we can do modify the forward iterator version as 

```cpp
template <typename IT>
int16_t distance(IT b, IT e, ...) {
```

And then the compiler will pick the random access version when both are available to it. This seems to only work for 2 different priority levels...

**No this doesn't work in fact**

### But wait! (Revisited)

Instead, this works, for some reason.

```cpp
template <typename IT>
auto _distance(IT b, IT e, int) -> decltype(e-b) { ...

template <typename IT>
int16_t _distance(IT b, IT e, ...) { ...

// The gateway function everybody uses
template <typename IT>
int64_t distance(IT b, IT e) { return _distance(b, e, 0); }
```




## Enable_If

What is `std::enable_if`? A mock implementation.

```cpp
template <bool, typename T> struct enable_if;
template <typename T> struct enable_if<true, T> {
	using value = T;
}

template <typaname T> struct enable_if<false, T> {};
```

Thus if I use `enable_if<condition, T>::value`, it will compile only if `condition` is true, otherwise `enable_if::value` wouldn't exist and there will be a compile-time error.

We run out of time, we will come back and finish it next lecture.

Maybe this can be used to improve the `distance` above.