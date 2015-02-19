#EPL Lecture 9, 2015/2/19

I missed one lecture on **template specialization**, **function object**, **template metaprogramming**.

## Recap

### Why iterator category is a type?

Only because we are using a template, in some generic function. We want to be able to write this function to have it work differently between *powerful* and *weak* iterators.

```cpp
template<typename T> // T is an iterator
  void doit(T p) {
  	if(T is powerful) {
    	...
    } else {
    	...
    }
  }
```

One way to do it is to make it a value

```cpp
template<typename T> // T is an iterator
  void doit(T p) {
  	if(T::power_level > 1) {
    	func1();
    } else {
    	func2();
    }
  }
```

For this code example, `func2` will probbly be optimized out when power_level is a `constexpr`, because compiler will search and remove **dead code**. However, both `func`s still need to exist and be syntax checked.

Instead, using function overloading method

```cpp
template<type T>
  void doit(T p) {
  	T::dependend_type x {};
    method(p, x);
  }

template <typename T>
  void method(T p, type1) {...}
template <typename T>
  void method(T p, type2) {...}
```

This solution would allow compiler to select the suitable overloading.

## Function Object
We can only sort objects that have a `<` operation. In C, we used to be able to put function pointer. C\++ has implemented a better way. Instead of calling function by pointer, we accept a **function object** that has some function methods.

```cpp
struct IntCompThing {
	bool operator()(int x, int y) { return x < y; }
}

template <typename FI, typename Comp>
FI partition(FI b, FI e, Comp f) {
...
}

partition(&x[0], &x[1], &intLessThan);
```

To reduce code duplicate

```cpp
template<typename FI, typename Comp>
  void quickSort(FI b, FI e, Comp f) {
  
  }

template<typename T>
  struct DefaultCompare {
    bool operator()(const T& x, const T& y) { return x < y; }
  };

template<typename FI>
  void quicksort(FI b, FI e) {
    using T = typename std::iterator_traits<FI>::value_type;
    // compare actual value and not iterator
    quickSort(b, e, DefaultCompare<T>{});
  }

```

## Invalidated Iterator

Why can't we use ponter anymore?

Design problem and syntax problem. Especially **invalidated pointer**

```cpp
auto p = x.begin();
x.push_back();
*p = 3;        // use an invalidated iterator, undefined behavior
```

Operation that changes the *set of position* will typically invalidate iterators on the containers. Changing the values is ok and shouldn't invalidate iterators.

### Solution:

We can give the iterator a **revision number** when created. If some operation will change the set of positions in the containers it will issue a new revision number. If you use an iterator with an older revision number, throw a Use of Invalidated Iterator exception. This is what Java does.

## Phase C

### Severity

To make it interesitng, we recommend you throw exceptions that label the severity of error.

- severe
- moderate
- mild

### Constructors

write a member template constructor

#### Partial copy

```cpp
template <typename FI>
  vector(FI b, FI e) {
    ...
  }
```

#### Initializer list

```cpp
vector(std::initializer_list<T> init);
```