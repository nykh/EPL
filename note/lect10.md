#EPL Lecture 10, 2015/2/24

## Logistics
Midterm and project 2 coming. Exact schedule TBD.

## Variadic template

### Intro

Big problem with `printf` in C
 
1. limits the printable types to those native to C. Your custom type can't directly feed to `printf`
2. The format string needs to match the datatype of the ptinted data. this burden falls on the programmer.

Now we try to imporove it


```cpp
void Print(const cchar* s) {
	uint32_t k = 0;
    while(s[k]) {
		if(s[k] == '%') {
throw missing_argument_exception("too few arguments supplied");
        }
    	cout << s[k];
        k += 1;
    }
}

int main() {
	int a = 42;
    const char* c = "hello world";
    double c = 3.14159;

    // we want this use case
	Print("Hello World\n", 42, c, b);
}
```

we add a template overloading

```cpp
template <typename T>
  void Print(const char* s, T arg) {
  uint32_t k = 0;
    while(s[k]) {
        if(s[k] == '%') {
            cout << arg;
            Print(&s[k+1]);
            return;
        }
    	cout << s[k];
		k += 1;
    }
throw extra_argument_exception("too many arguments supplied");
}
```

### Exception handling

like Java, like C++

```cpp
struct Print_exception {
	const char* msg;
    Print_exception(const char* s) : msg(s) {}
}; 

struct extra_argument_exception : public Print_exception {
	using Print_exception::Print_exception;
}

struct missing_argument_exception : public Print_exception {
	using Print_exception::Print_exception;
}


int main() {
	// ...
    try {
        Print(...)
    } catch(Print_exception e) {
        cout << "Print exception: " << e.msg << endl;
    }
	// ...
}
```

**Note** the `using` statment in the `extra_` and `missing_argument_exception` blocks. This declares it inherits the constructor of its base class.

**Also Note** the `public` declaration in the inheritance list. In C\++ it is possible to do **public inheritance**, i.e. inherit all the public members of the base class as public members, and **private inheritance**, all public memebers in base class becomes private. **The default in C\++**, for some reason, **is Private inheritance**. Therefore you need to explicitly say `public` if you want public inheritance. (Apparently, there is also `protected inheritance`, Prof. Chase felt it may be useful)

### Now comes Variadic template

Now we want the same function to work for multiple arguments. Stealing code from chase's example:

```cpp
template<typename T1, typename ExtraArgTypes ... >
void Print(const char *s, T1 arg, ExtraArgTypes... extra_args) {
	while (s[k]) {
		if (s[k] == '%') {
			if (s[k+1] == '%') {
				k += 1;
			} else {
				cout << arg;
				Print(&s[k+1], others...);
				return;
			}
		}
		cout << s[k];
		k += 1;
	}
throw extra_argument_exception("too many extra arguments");
}
```

**Note** the way the elipsis type list is unpacked. The function signature is `void Print(const char *s, T1 arg, ExtraArgTypes... extra_args);`, and the rest of the list is passed recursively as `Print(&s[k+1], others...);`, in effect unpack the head of the list into `T1 arg`.

#### Remaining problem
What if the function is called like
```cpp
Print("blah blah %")
```
? Does it break?

In fact it shouldn't, because the compiler will choose the non-variadic version of the function.

## emplace back
This is even better than using `move` constructor. Where the client construcst the object, and then you move it to the place. Why can't you and client just agree, to let the client **construct the object in the lace you desire**? This is the idea behind `emplace_back`.

```cpp
template<typename... Arg>
void emplace_back(Arg... args) {
	// ensure capacity
	new(this->data + this->endIndex) T{args};
    endIndex += 1;
}
```

Remaing problem is, in the `{args}` passed to instialization list constructor, some of the element may be **rvalue** and we want to smartly choosing to move them, without ruining **lvalue**. `std::forward` is the perfect tool here. It strips the rvalue off and put them back there. The optimized code becomes

```cpp
template<typename... Arg>
void emplace_back(Arg&&... args) {
	// ensure capacity
	new(this->data + this->endIndex) T{ std::forward(args)... };
    endIndex += 1;
}
```