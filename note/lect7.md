#EPL Lecture 7, 2015/2/12

## Q&A
###  confusion in `std::move`
C++ gives you flexibility in choosing to do shallow copy instead of deep copy.

`T&&`  = rvalue reference

Stroustrup said `std::move()` should just be named `rvalue()`. It is essentially just a type cast to smartly cast lvalue into rvalue.

The compiler will choose `T&&` version of function if you provided one. If there is only a lvalue version, it will convert the rvalue back to lvalue and pass along.

```cpp
class String {
	String(const String &x) {...}  // lvalue version
	String(String &&x) {...}       // rvalue version
}
```
**Note:** As soon as you give a rvalue a name, it ceases being a rvalue.

If I don't write a move ctor, a call like
```cpp
new (q) T { std::move(p) };
```
will just run the copy ctor.

**The bottom line is:**  If an object is constructed, it must be destructed. It doesn't matter if it's copy- or move-constructed.

## Generic Algorithm Example: Sorting
See `example/Sort/`

### recap: the four goals of generic algorithm are

1. Designed to be generic as respect to element data type.
2. Generic with respect to data structure too.
3. Has predictable time complexity
4. efficient -- on par with hand-written code.

### recap: iterator operations

1. `*p`
2. `++p`
3. `p == q`
4. `--p`
5. `p+k`
6. `p-q`

When writing a generic algorithm, it is important to be decliplined and only use the minimal set of iterator operators. Unless you want some containers to be not supported. Chase went as far as to prefer `!(p==q)` to `p!=q`. By doing so, the algorithm can be completely independent of container type.

### To extend and try to use more operations
```cpp
uint64_t distance(I b, I e) {
	if(I::iterator_category >= RANDOM_ACCESS)
		return distance_RAI(b,e);
	else
		return distance_FI(b,e);
}
```
Why doesn't this work instead of function overloading? Because the compiler will then generate code for both `distance_FI` and `distance_RAI`, and for a FI-only container, the operation in RAI version (`e-b`) is illegal and won't compile. Thie is against our rule that

> if it work, it should compile

So istead, we use function overloading:

```cpp
uint64_t distance(RAI b, RAI e, std::random_access_iterator_tag t) {
	return e - b;
}

uint64_t distance(FI b, FI e, std::forward_iterator_tag t) {
	...
}

uint64_t distance(I b, I e) {
	I::iterator_category x{};
	return distance(b, e, x);
}
```