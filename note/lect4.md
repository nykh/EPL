#EPL Lecture 4, 2015/1/29

## Q&A
###### Why  we use operator new() instead of malloc()
They might use different brackets. So use
```cpp
p = operator new(sizeof(T));
new (p) T {args};
delete p;
```

###### Why not size_t?
Because **the size of size_t variable** itself is not same across platforms, and in principle *System dependent size* is not a good idea.

# template LinkedList Example
Continuing from last lecture

```cpp
template <typename ElemType>
class LinkedList {
private:
	struct CellBase {
		CellBase *next = this;
        CellBase *prev = this;
    }
	
	struct Cell: public CellBase {
    	Cell(const ElemType& x) : data{x} {}
    	ElemType data;
    };
    Cell* dummy = new Cell{};
    // compile time limitation: If ElemType does not have a zero argument constructor
   // this code does not compile.

    uint64_t length = 0;

public:
    LinkedList(void) {}
	LinkedList(int initial_size) {
		for(int k = 0; k < initial_size; k += 1 ){
			ElemType t = ElemType{};
			append(x);
		}
	}
	LinkedList(const LinkedList<ElemType>& that) { copy(that) }
    ~LinkedList(void) { destroy(); }

	LinkedList<ElemType> operator=(const LinkedList<ElemType>& that) {
    	if(this != that) {
        	destroy();
       		copy(that);
        }
        return *this;
    }

	void append(const ElemType x) {
    	Cell * p = new Cell{x};  // copy constructor, most objects are copy-constructable
		p->data = x;
		p->next = tail()->next;
        p->prev = tail();
        tail()->next->prev = p;
		tail()->next = p;
     }

private:
	Cell*& head(void) { return dummy->next; }
    Cell*& tail(void) { return dummy->prev; }
};
```

The head note will keep a value of length and will always point to dummy. Also it is a cyclic Linked List, i.e. last node's `next` points to the `dummy`.

When expanding a template, it is required to spell out the template for each member function.

```cpp
template <typename T>
void LinkedList<T>::destroy(void) {
    Cell* p = dummy->next;
    while(p != dummy) {
    	Cell* q = p;
		p = p->next;
        delete q;
    }
	delete dummy;
}
```

Another wierd thing about template is you need two template arguments on different line to specify a template with multiple typename arguments.
```cpp
template <typename LHS_ELEM>
template <typename RHS_ELEM>
LinkedList<LSH_ELEM>::LinkedList(const LinkedList<RHS_ELEM>& that) {
	// basically same code with copy constructor
	Cell* p = rhs.head();
    while(p != rhs.dummy) {
    	insert((LHS_ELEM) p->data);
		p=p->next;
    }
}
```

Everytime you provide a type argument to **instantiate** a type, it asks the compiler to generate a new type. So `LinkedList<A>` is totally unrelated to `LinkedList<B>`.

## friend syntax
```cpp
template<typename T>
friend class LinkedList;
```

## keyword explicit
constrain a constructor that it can only be used explicitly as a construtor, not as implicity type conversion tool. Used on any constructor that has an argument of another type.

## constructor
For an *automatic* variable.
```cpp
int x;           // not initialize, some random value
int x{};         // initialize to 0
int x = 42;
int x = int{42};
int x{42};       // equivalent to "int x = 42"
```

## Iterator
Encapsulation for iterable containers. Users do not need ot know the implemenation and boundary conditions.

### Syntax to use
```cpp
LinkedList<int> fred;
LinkedList<int>::iterator p = fred.begin();
while(p != fred.end()) {
	cout << *p;
	++p;
}
```
### also for-each style
`const auto&` is usually the best practicce.
```cpp
for(const auto& x : fred) {
	cout << x;
}
```
