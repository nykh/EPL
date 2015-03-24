#EPL Lecture 13, 2015/3/24

## Project 2
Use EPL vector, this is provided by grader and is a testing mock that log number of copies, etc, for easier grading.

```cpp
valarray<int> x(10);
valarray<double> y(10);
valarray<complex<double>> z(10);
x = 1;
y = 3.5;
z = x * y + (x + x);
```

##### Type promotion

Here the `x*y` should be calculated in `double`. `x+x` in `int`.

Should create a tree that looks like

             =
          z      + (t3)
              * (t1) + (t2)
             x y   x x

Encode information in nodes. But problem is how do you handle mutiple occurence of the same element?

### So, What is valarray? Viewed in the lens of Concept

A substitute of vector? No, it is actually a interface adapter for the builtin `vector` type. It allows you to perform arithmetic and assignment on `vector` but nothing else.

Why does this matter? Because we start off wrapping `vector`, but as the tree level leaves the leaf, we want the node type to be `valarray` conceptually. Actually, a `valarray<Proxy>`.  This decoration allows me to perform the same operations on Proxy, a vector like thing.

c\++ doesn't use the term `interface` maybe to avoid conflict with Java, but it calls *concept*. `vector` is itself a concept. `vector` and `Proxy` both implements this concept. Our `valarray` should be able to wrap anything that is of `vector` concept.


### Conceptualization

```cpp
template <typename Left, typename Right>
class Proxy {
	Left& l;
    Right& r;
}
```

For **t1** `Left` is `valarray<int>`, `Right` is `valarray<double>`.

Now make the decorator:

```cpp
template <typename vector>
class valarray : public vector {...}
```

This can work on any vector concept as long as it

- has `operator[]`
- has `size()`
- has member type `valueType`

### T3, rvalue reference...

So the two prentheses in `(x*y)+(x+x)` create rvalue **t1** and **t2**, **t3** has references for **t1** on the left and **t2** on the right. However, the compiler doesn't know how long to keep **t1** and **t2** and will throw them away after **t3 add(t1, t2)** returns. This is subtle and will be VERY HARD to debug.

#### Solution

Instead of reference, `t3` should contain **copies** of **t1** and **t2**. We want `Proxy` to contain either **reference to vector** or **copy of Proxy**. How to specialize template to allow this?

```cpp
template <typename Left, typename Right>
class Proxy {
	// the Left and Right implement the "vector" concept
    typename to_ref_or_not_to_ref<Left>::type l;
	...
}
```

Here we write a metafunction to decide whether it should be a reference or not. If it is a vector, return a reference for it, if it is Proxy, return a copy of it.

Convention for metafunction in C\++ is to return a type as `type`, a value as `value`.

```cpp
// most generic case: return plain copy of it
template <typename T>
struct to_ref_or_not_to_ref {
	using type = T;
};

// specialization for vector
template <typename T>
struct to_ref_or_not_to_ref <vector<T>> {
	using type = vector<T>&;
};
```

To make the syntax of using less ugly. We use type alias.

```cpp
template <typename T>
using Ref = typename to_ref_or_not_to_ref<T>::type;
```

Then our code can look like

```cpp
template <typename Left, typename Right>
class Proxy {
	// the Left and Right implement the "vector" concept
    Ref<Left> l;
	...
}
```