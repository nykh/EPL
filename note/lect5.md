#EPL Lecture 5, 2015/2/3

## Logistics
Yes test case is broken. Will fix.
Next week: iterator()

## `std::move` syntax
Move: Taking advantge of shallow copy when shallow copy is the right thing to do. Many scenarios this is more efficient than copying.

**C++11 Solution to the move problem/**
```
operator=(const T&);
operator=(T&&);
```

where `T&&` is a **rvalue reference**, always needes to be changed later so not **const**.

- rvalue reference type version is called whenever an argument is a **temporary object** and has no name.
- **rvalue reference** can convert to **lvalue reference** but not the other way around. Especially, when rvalue is captured as an argument and used it implicitly converts to **lvalue reference**.
- `T&` can be convereted to `const T&` but not the other way arouund

You need to manually run the destructor for the object after moveing it. Normally after copy-constructor, the destructor for the object is still called and executes correctly, but not so after it is moved.

`std::move()` smart typecast to rvalue reference.

```
void move(T&&);

// move constructor
T::T(T&& that) {
	this->move(std::move(that));
}

// move assignment
T& T::operator=(T&& rhs) {
	destroy();
	this->move(std::move(rhs));
}
```