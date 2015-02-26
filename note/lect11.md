#EPL Lecture 11, 2015/2/26

## Type conversion

### explicit type cast

Type casts in C\++ are **type functions**. The classical type-casts in C\++ are

- `static_cast<T>(expr)`
	
	Done at static time

- `dynamic_cast<T>(expr)`
	
    this works just like type conversion in Java. Done at compile time.

-  `const_cast<T>(expr)`

	Add or remove `const`ness
    
- `reinterate_cast<T>(expr)`
	
    most like C-style cast `(T) expr`. Have the fewest restrictions but carries risks

### Implicit type cast

C++ also can supply implicit compression using constructors of objects.

```cpp
String s("Hello World");
if(s == "Hello World")
	//...
```

When this code is compiled. The compiler looks up a member function of `String` that matches

```cpp
bool operator==(const char*);
```

Not able to find one. It then realize `String` has a constuctor

```cpp
String(const char*);
```

and use it to perform implicit type conversion. Therefore the code is interpreted as:

```cpp
if(s == String("Hello World")) //...
```

### to suprress implicit type conversion

Sometimes there are constructors that we don't want the compiler to use in implicit conversion. For example, the `vector` class we wrote has a constructor 

```cpp
vector(int n);
```
We certainly don't want the compiler to use this to **convert** an `int` to a `vector`. therefore we declare this as `explicit`.

### Warning: Don't allow type demotion

When there are two types that need to work together. Choose the one that is *superior* and *promote* the other one to this. Type demotion is a bad practice as it throws away useful data. **In particular, if you actually define conversion both way** the compiler will be confused and issue compile error.

### Work with `rel_ops`?

Can this work? The compiler instantiate operator `!=` in `main()` by deducing that `T` is `String`.

```cpp
template<typename T>
bool operator!=(const T& x, const T& y) {
	return !(x == y)
}

int main(void) {
	String s;
	if(s != "Hello World") // ...
}
```

Answer is **NO**. The compiler **will not mix type conversion with implicit template type deduction**. It can only do one or the other.

## template generic function

I want to write some generic function like 

```cpp
template<typename T>
T& max(T& x, T& y) {
if(x < y) return yl
else return consst_cast<T&>(x); // NOOOOOO
}
```

But I find users might pass rvalue to it like `max(x, 42)`.


### Solution #1: `const_cast`

```cpp
template<typename T>
T& max(const T& x, const T& y) {
if(x < y) return yl
else return consst_cast<T&>(x); // NOOOOOO
}
```

That's too much of a hack.

### Solution #2: write both!

```cpp
template<typename T>
T& max(T& x, T& y) {
if(x < y) return yl
else return consst_cast<T&>(x); // NOOOOOO
}


template<typename T>
T& max(const T& x, const T& y) {
if(x < y) return yl
else return consst_cast<T&>(x); // NOOOOOO
}
```
### Debugging type

Sometimes it can be hard to debug with all the conversion and type deduction. Here is some trick to help debugging.

```cpp
cout << "T was deduced as type " << typeid(T),name() << endl;
```

But this doesn't help us in this case. `typeid` strips away any reference or lvalue-rvalue properties and only tell you it's an `i` (for `int`). If we want to find out the storage property of a type, we can write some type function.

### Dreaded template meta-programming example

```cpp
template<typename T>
struct is_reference {
	static constexpr bool value = false;
};

template<typename T>
struct is_reference<T&> {
	static constexpr bool value = true;
};
```

#### Arcanery in C++

`T&& &` becomes `T&`
`T&& &&` becomes `T&&`

#### Expand on `is_reference`

We not only want to use `is_reference` in some if statement. We might also want to do conditional compilation. So we use the standard library's way of representing truthiness.

```cpp
template <typename T>
struct is_reference : public std::false_type {}

template <typename T>
struct is_reference<T&> : public std::true_type {}
```

This allows us to write **compile time predicate**.

## Solution #3: apply the above discussion on `max`

We introduce a second problem, we want the user to be able to use `max` like `max(5, 6.0)`. One is an `int` the other is `double`. Again, in this situation it is necessary to choose the *superior* type.

In fact, let's just rank them: `double` is level 3, `float` is level 2, `int` is level 1.

```cpp
// very silly meta-function
template <typename T>
struct rank;

template<> struct rank<int> { static constexpr int value = 1; }
template<> struct rank<float> { static constexpr int value = 2; }
template<> struct rank<int> { static constexpr int value = 3; }

// very silly meta-function #2
template <int R>
struct stype;

template<> struct stype<1> { using type = int; }
template<> struct stype<2> { using type = float; }
template<> struct stype<3> { using type = double; }

template<typename T1, typename T2>
struct choose_type {
	constexpr r1 = rank<T1>::value;
    constexpr r2 = rank<T2>::value;
	constexpr max_rank = (r1 < r2) ? r2 : r1;
	using type = typename stype<max_rank>::type;
};

template<typename T1, typename T2>
typename choose_type<T>::type max(const T1& x, const T2& y) {
if(x < y) return yl
else return consst_cast<T&>(x);
}
```

### Another example: `Complex<T1, T2>`

What if I want my `max` to suppot the `Complex` type? We need to add more terms to meta functions.

```cpp
template<typename T> struct rank<std::complex<T>> { static constexpr int value = rank<T>::value; }
```


