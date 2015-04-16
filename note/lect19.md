#EPL Lecture 18, 2015/4/16

## Virtual function (cont.)

C\++ is always statically type-checked.

In normal case, when base and derived classes share functions, everything is statically typed checked. The linker will know at compile-time which function is getting called. Thus there is no ambiguity, it can inline the function.

```cpp
Base b;
Derived d;

d.foo();
d.Base.foo();
```

In case of virtual function however, the compiler cannot know at compile time which function is getting called (indirect function call). Thus it is difficult to inline a virtual function.

Whenever there is a virtual function in a class. A virtual function table is created for it. The ordering is arbitrary.

The virtual function pointer can only be in the **base** part of the object, so it is accessible to all objects with the same offset. What this means, however, is when we do a down-cast. `b = d;`, the compiler must **avoid** copying the vft pointer over in order to keep the type of `b` intact. (calling `b.foo()` should call the version in b's vft) 

**Note** even when a derived class inherits all (overrides none) of the base class virtual functions. The compiler still creates a new table that is **identical** to the base class's vft and points the vft pointer to this one. The benefit is that this pointer value can serve as a **run-time type identifier** for us.

## casting

The idea of a `dynamic_cast` is built around the idea of vftp, even though the language does not guarantee explicitly that there is such a pointer.

```cpp
Base *bp;
Derived *dp;
d = static_cast<Derived*>(b);
```

What happens when I statically cast a pointer, the compiler assumes I know what I am doing, and just point the pointer at the beginning of object `b`. **Problem!**

## `dynamic_cast` and type safety feature

dynamic cast, on the other hand, is java-style. The compiler will use the virtual function table (**!**) to determine the type and then edit it to point to a correct new table. However, if `*b` is not a type of `Derived`, the compiler will return a `nullptr`. If it's a reference cast, you get an exception.

You cannot dynamic cast unless `Base` and `Derived` classes have virtual function tables.

```cpp
Base b;
Derived *p = dynamic_cast<Derived*>(&b);
```
What happens is similar to the pseudo-implementation below

```cpp
dynamic_cast<NT>(OT* old) {
	return (old->ptr == &NT..VFT) ? old : nullptr;
}
```

## virtual destructor

In OO style, it is often much more easy to use pointers instead of object allocated on the stack. This allows us to have the polymophic behavior.

```cpp
Base *p = new Derived();
```

But what happens when we `delete` it?

What happens when we delete, we run the destructor, and then deallocate the chunk of memory. What we worry is the destructor. What if a derived class has additional external resources? The solution is to **make the destructor virtual**.

###  When do we *NOT* need a virtual destructor?

**When the default destructor is ok?** Sadly, no, because derived class may have external resources, and will need a virtual destructor.

The overhead associating with indirect jump is also invalid since the `delete` needs to deallocate the memory, which takes a lot of time already. An indirect jump doesn't matter in comparison.

So the real answer is: **Always**.

### Can we inherit destructor?

Wrong question, since base class is an implicit member in derived class, so its destructor will definitely be invoked.






## Abstract Class

Sometimes there are **concepts** out there that is **abstract**, you don't want to implement any method for it. You want to declare but not define the virtual function. The correct syntax is this

```cpp
virtual void foo() = 0;
```

`=0` is definitely ugly but that's the standard.

### Implementing the dysfunctional class?

What happens under the hood? The class is not actually created, the compiler will  forbid you from instantiate an abstract class.