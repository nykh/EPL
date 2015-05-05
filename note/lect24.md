#EPL Lecture 24, 2015/5/5

## Pointer to member function

In C++, pointer to functions and pointer to method functions are mutual exclusive set. They can't convert to each other. To correctly point to a member function look at the example below:

```cpp
struct LifeForm {
	void doit(void) {}
    static void sdoit(void) {}
};

void (LifeForm::*some_pointer) (void);
some_pointer = &LifeForm::doit();
LifeForm fred;
(fred.*some_pointer)();
```

This is because every member function has an implicit argument (`this`). But there is another calculation cost hidden behind member function calling: Because of multiple-inheritance, the offset added to `this` may not be 0 and must be calculated every time a member function is called.

But `static` function could work because they don't require a `this`.

```cpp
p = &LifeForm::sdoit();
p();
```

## function pointer that points to any function

How can we bridge this gap and make a pointer that can point to **BOTH**?

An example can be found at `examples/Function``