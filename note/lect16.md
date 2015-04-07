#EPL Lecture 16, 2015/3/31

A different way of doing meta-predicate, this time with real function.

```cpp
template <typename T> bool is_pointer(T x) { return false; }
template <typename T> bool is_pointer(T* x) { return true; }
```

But it doesn't work where both are avaialble. (When it is a pointer)

```cpp
template <typename T, typename unused=decltype(*x)>
	bool is_pointer(T x) { return false; }
template <typename T> bool is_pointer(T* x) { return true; }
```
## Project 2

### Scalar type promotion
