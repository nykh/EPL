# EPL Lecture 1, 2015/1/20

C\++ basics

```cpp
struct Foo {
private:
	int x;
    Foo(void) { x = 42; }
    ~Foo(void) {}
	void doit(void) { x += 1; }
}

int main(void) {
	Foo f;
    f.doit(); // A
}
```

`Foo()` instance is suppsed to be 4 bytes in C\++, because the member functions live elsewhere.

What happens here at location *A*, is an implicit reference to f is passed to the `doit()` function as `this`.

---
```cpp
Foo *p;
p = new Foo;
```
`new` and`delete` keywords allocate memory and call constructor for the new object instance. This is equivalent to (per new syntax)

```cpp
p = operator new(sizeof(Foo));
new(p) Foo{};
```
First line calls `new` operator. Second line calls the constructor with implicit argument `p`,

---
## Collection- vector

```cpp
std::vector<int> v;

while(auto x ...) {
	v.push_back(x)
;}
```

### Implementation of vector in STL
A static array of 3 pointers + a big allocated array
1. begin -> the beginnign of big array
2. used end -> 
3. end -> one beyond the end of big array