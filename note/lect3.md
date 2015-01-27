#EPL Lecture 3, 2015/1/27

## Project 1
Project 1 is split into 3 parts to be done over the weeks.

**Topic**

1. Implement vector container.
2. `move()` semantic and `copy()` semantic
3. initialization list, iterator, template

---
Part A, due **next Thursday (2/5)**
- A* = OK
- A\** = graduate requirement

## makefile
**make** A bulid program that automates preparing compilation and linking.

**makefile**defines actions on `.c` and `.o` files.

```
CXXSRCS=$(shell ls *.cpp)
SRC=$(DXXSRC)
OBJS=$(CXXSRC:.cpp=.o)

all: $(PROGRAM)

clean:
	-rm -rf *.o
```

Choose: `Eclipse -> New -> Project... -> C/C++ -> Makefile project with pre-existing files`

## String class

Instead of comparing `char` against `char`, we can read 64bits (8 `char`s) at a time and compare them as an integer.

** Problem**: **Endianness** (on a little endian machine) and imcompatible lengths

** Solution**: padding to multiple of 8 and store backward

### constexpr
`constexpr` A new keyword in C++11, that declares something is constant. 

Previously, `const` can be ambiguous, because C++ supported `extern const` = not compile time constant, but can't be writen to.

Solution is `constexpr` means actually compile time constant.

Great thing is: now you can support function that returns `constexpr`. In this case, the evaulation is done at compile time and takes no calling overhead. Different from `inline` keyword, which can be runtime.

### Exception mechanism
Unlike Java, C++ doesn't require the object to be thrown to implement some basic class (like Java's `Throwable` interface)

When an function in Java ends, it just ends. Immediately aborts. In C++, when a function exits, it performs **stack unwinding**, calling all the destructors of the objects going out of scope.

**RAII rule** - Resource Acquisition is Initialization. Gurantees stack unwinding will work in the face of exception.

### returns l-value and r-value

```
char& operator[](uint32_t k) {
	// return a lvalue copy to a character
}

char operator[](uint32_t k) const {
	// return a copy (rvalue) for an immmutable string
}
```

## Template

```
template <typename ElemType>
class LinkedList {
private:
	class Cell {
    	ElemType x;
		Cell* next, prev;
    };
    Cell* dummy;

public:
    LinkedList(void) {
    	dummy = new Cell;
		uint64_t length = 0;
    }
};
```
