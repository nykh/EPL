#EPL Lecture 18, 2015/4/9

## Recall

Implementing **is-a** relation: When we initialize a derived object, the inherited part from the base class is initialized as a contiguous whole and put in the beginning of the object, just for easier implementation.

## Name conflict, hiding

What happens when derived class has a member with the same name as in the Base?

```cpp
struct Base {
	int x;
    void doit(void) { x = 42; } // writes to x in base class
};

struct Derived : public Base {
	int x;
    void doit(void) { cout < "hi!\n"; }
};

...
d.x = 0;           // write to x in derived class
d.doit();          // calls the doit in derived class
d.Base::doit();    // explicitly call the base class version
cout << d.Base::x; // you get 42
```

Compiler will always pick the name with the closest scope. (First search the derived class, then base class(s) if not found.)

In C\++, when you hide a function in derived class, you hide all the functions of the same name with all the overloading variations. **This is an error.**

```cpp
struct Base {
	int x;
    void doit(int v) { x = v; } // writes to x in base class
};

struct Derived : public Base {
	int x;
    void doit(void) { cout < "hi!\n"; }
};

...
d.doit(42);   // ERROR
```

Chase thinks this rule is strange. How do you get around it? Add this to the derived class.

```cpp
void doit(int v) { Base::doit(v); }
```

This is annoying but ok. However, more problem ensues if someone comes in and refactor the Base class, adding a new signature to `doit`. The addition will be hidden to the user of `Derived` until that addition is explicitly added into `Derived`. There is no way around this problem. Again why does C\++ decided on this rule? Chase doesn't understand.

## Downcast

What happens when we do this?

```cpp
Derived d;
Base b = d;
```

This assignment is required to be legal, it will automatically takes only the inherited part.

The language, in the spirit of OOP, allows a pointer of Base to point to Derived, but for a pointer of Derived to point to Base cast is required (**static_cast**, **dynamic_cast**, etc)

## Multiple Inheritance

A class that inherits from multiple classes is more complicated. It will have all the members from all the inherited classes, as contiguous blocks, of course. The problem is: which inherited class gets to be put before others in the memory representation?

```cpp
class Base {
	int x;
    void doit();
};

class Base2 {
	int y;
    void doneit();
};

class Derived : public Base, public Base2 {
	int z;
};
```

Now in memory which comes first? This is completely arbitrary up to the compiler. They can be in any order.

When a method is to resolve, our experiment shows that for methods inherited from a class that gets put **not on the first place in memory**, `this` is not the same as the address of the derived class, but with an offset to point to the beginning of the inherited base class. This is the only way that will allow the **this** to work in those to access its data members.

### Memory Management

Given

```cpp
Derived* d = new Derived;

Base1 *p = d;
Base2 *q = d;
```

How can I deconstruct `d`? Which ones of these work?

```cpp
delete p;
delete q;
delete d;
```

In experiment it turns out `delete q` doesn't work.

### Failure of polymorphism?

```cpp
struct Cake {
	void eat() { cout << "yum"; }
};

struct ChocolateCake : public Cake {
	void eat() { cout << "OMG"; }
};

void consume(Cake& c) { c.eat(); }

ChocolateCake d;
consume(d);
```

Looks good, except because `consume()` takes a pointer (ok, reference) of `Cake`, it can only output **yum** even for a ChocolateCake. Obviously, when generating code for `consume`, the compiler won't be able to know what exactly type `c` is, if not just Cake. Heck, at this time `ChocolateCake` may still not be invented!

To have true polymorphism, we need to use **virtual function** table.

```cpp
struct Cake {
	virtual void eat() { cout << "yum"; }
};

struct ChocolateCake : public Cake {
	void eat() { cout << "OMG"; }
};
```

Now it will work. How is this implemented internally? The compiler will have to generate **virtual function table** for each class. Each derived class object will have a pointer to the respective table, the compiler will know the pointer and the offset into that table, and perform a indirect jump.

This virtual function table is a statically allocated data member. The memory overhead for this is the size of this table (`ADDRESS_WIDTH * num_of_virtual_fnctions`) for each class (base or derived), and a pointer to this table for each object.

####  Disadvantage

An indirect jump can not be inlined, and slooooooow. If the virtual function is huge, this might be justified, but if it's really short you will probably be better off using template.

## Next week

How does multiple inheritance factos into this virtual function resolution?