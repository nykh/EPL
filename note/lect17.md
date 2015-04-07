;#EPL Lecture 17, 2015/4/7

## Project 2

**Name requirement for iterator type?**
There is none. (iterators will be created using `auto`.)

**Apply and Accumulate signatures**

```cpp
x.apply(std::plus<int>{})
```

Only requirement is to use the function object. Function pointer is not tested.
You can assume there is a property.

```cpp
F::result_type;
```

Also assume they are very light-weight so you can always store a copy of the function object

**Should we Support direct assignment of valarray**

```cpp
valarray x(10), y(20);
x = y;
```

This should behave as assignment, but the size of `x` **SHOULD NOT CHANGE**, in this example, x will be the 10 first elements of `y`. If x is instead longer than y, then only the first few `y.size()` elements will be overwritten.

**How to do promotion from scalar to `scalar_wrapper`?**

write the operators in `valarray`

## Inheritance

We have used inheritance quite a lot, not for polymorphic behavior, but just for inheritance itself. We should talk more about inheritance. Multiple inheritance, etc.

Hopefully we have enough time.

### First point: Inhertiance in C\++, Java, C# is One language construct to solve two kinds of different problems

One, **code reuse**.
Two, **sub-typing polymorphism**. This is the OOP part.

Example being our expression tree:

```cpp
switch(node->type)
case ADD  ... break;
case MUL  ... break;
...
```

That was not OOP.
This is OOP, you don't care about type the object is, you can just tell the node to evaluate itself.

```cpp
node->eval()
```

In some languages (ex. Sather) you can inherit from a class without becoming a subclass of it. But in C\++ etc the two features merged.

```cpp
struct Base {
	int x;
    int y;
   void doit(void) { this->x = x+y; }
}
```

For a simple class like this, the compiler will generate something like

| element | offset |
|----|
| x | 0 |
| y | 4 |

Also a function

```cpp
doit(Base* this) { this[0] = this[0] + this[4]; }
```

For a derived class

```cpp
class Derived : Base {
	int z;
    void doit2(void) { z = x+y; }
}
```

The compiler should generate this object, note it puts the inherited part **first**

| element| offset |
|----|
| x | 0 |
| y | 4 |
| z | 8 |

and

```cpp
doit2(Derived* this) { this[8] = this[0] + this[4]; }
```

What about the inherited function? When one call a base class function from a derived class `d.doit()`. Compiler first look at derived class for the function, not finding it, it then looking at the Base class, and use it instead. It will still work, beccause `x` and `y` still correspond to the 0th and 4th element in the object. In fact, `doit()` would not even realize that it is operating on a new class.

So far so good. This convention has been true since the very beginning of C\++.

### upcast

The reason we can not upcast is because the compiler could not verify the validity of the address. C\++ does not rely on run-time type information, therefore once an object is made, there is no easy way for the compiler to know what type it is.

```cpp
Base b;
Derived d;
Base* bd = &d;
Derived* dq = bd;  // this line is an error
```

