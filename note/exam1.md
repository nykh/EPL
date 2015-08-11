## Problem 2 : move\_iterator

```cpp
template <typename IT>
struct Adapt : public IT {
	using IT::IT;
    using T = typename iterator_traits<IT>::value_type;
    
    T&& operator*() {
    	return std::move(this->IT::operator*());
    }
};


template <typename T>
auto mbegin(T x) -> Adapt<decltype(begin())> { return { begin() }; }
template <typename T>
auto mend(T x) -> Adapt<decltype(end())> { return { end() }; }
```

## Problem 4 : LargeCell

My mistake was providing the destructor without a copy/move constructor. And in this case you definitely want a move constructor.

```cpp
template <typename T> struct LargeCell {
	T* data;
    LargeCell(T const& d) : data(new T{d}) {}
    operator T&(void) { return *data; }
    
    ~LargeCell() { delete data; }
    LargeCell(LargeCell<T>&& rhs) {
    	data = rhs.data;
        rhs.data = nullptr;
    }
};
```

## Problem 5: Reboot

1. reboot takes a `T&` as first argument, pointing to a variable being rebooted.
2. can have zero or more additional arguments for constructors
3. destruct x before reinitializing x

My mistake was mainly using `delete`, not realizing `delete` will also deallocate the memory from free store...

```cpp
template <typename T, typename... Args>
void construct(T* p, Args... args) { new (p) { args... }; }

tempalate <typename T, typename... Args>
void reboot(T& x, Args... args) {
	x.~T();  // NOT delete
    construct(&x, args...);
}
```

## Problem 6: IsPrintable

### part (a)

```cpp
template <typename T, typename = decltype(cout << declval<T>{}) >
true_type checkPrint(int) { return true_type{}; }

template <typename T>
false_type checkPrint(...) { return false_type{}; }

template <tpyename T>
constexpr bool isPrintable(void) {
	using c = decltype(checkPrint<T>(0));
    return c{};
}
```

### part (b) use the function above

```cpp
struct pstream {
	ostream& os;
    pstream(ostream& o) : os{o} {}
    template <typename T>
    pstream& operator<<(T const& x) {
    	Safe<isPrintable<T>(), T> s {x};
        out << s.val();
        return *this;
    }
};

template <bool, typename T>
struct Safe {
	Safe(T const&) {}
    const char* val(void) { return "<UNPRINTABLE>"; }
};

template <typanme T>
struct Safe <true, T> {
	T const& x;
	Safe (T const& _x) : x{_x} {} 
    T const& val() { return x; }
}
```