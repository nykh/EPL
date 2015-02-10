#EPL Lecture 6, 2015/2/10

## Forcast
- Phase C
	- iterator!
	- not sure but may add generic algorithm
- template metafunction
- smart pointer

## Iterator
Purpose of the iterator is: give client a handle to access the element of an opaque collection. In C\++, this is more true to say: C\++ Iterator gives client a handle to the **position** of the element.

If we expose the pointer to the client, they can write code like this
```cpp
Cell<T> *p = list.head();
while( p != list.dummy) {
	cout << p->data;
	p = p->next;
}
```
This is *bad*, because the client code will be tightly coupled with our code. And our implementation detail leaks to interface.

### How Java solves this problem (1st Generation)
```java
Iter p = list.Iterator();
while(p.hasNext()) {
	x = p.read();
}
```

Two reasons Chase considers this bad
1. You can only check `hasNext` meaning you can only iterate the whole collection, not some subset of it.
2. It couples `read` operation to the pionter.

### How C++ solves this (2nd Generation)
It solves this by levying C pointer convention, so its iterator looks exactly like pointer.

In C++, you grab one ~~pointer~~ iterator to the beginning, one at the end. Iterate until they meet. This solves the problem of iterating a subset of the collection.

```cpp
auto p = list.begin();
while( p != list.end()) {
	x = *p;
	++p;
}
```

### STL algorithm
1. Designed to be generic as respect to element data type.
2. Generic with respect to data structure too.
3. Has predictable time complexity

	In Java, it provides the same functions to every collection, for example, random access on a **linked list**. In C\++ the philosophy is: If it is not suitable, don't provide.

4. efficient -- on par with hand-written code.

### What do the STL iterator provide?

Pointer operations
1. `*p` - **Dereference**
2. `++p` - increment = **advance**
3. `p == q` - **equality comparison**. Similarly, `!=` is just the inverse predicate.
	These three are the most important.
4. `--p` - decrement = **retreat**
5. `p+k`, **random access**,this combines with `*p` can be shorthanded as `p[n]` == `*(p+n)`
	This one rquires certain properties to be O(1).
6. `p-q`, **difference**, produces a number that denotes the number of elements in between.

Note, as stated before, **Not all data structure supports all of these operations**.

STL classifies collections into 3 (5?) categories.
1. forward_iterator: supports the first 3 operations
2. bidirectional iterator: supports the first 4
3. random_access: supports all 6 

### Conventions
It still exists, but argurably some only for legacy reason. Some new features in C++11 has negated the need for them.

1. Nested type
	the convention requires the iterators to be nested type inside the collection. Particularly, the two types `iterator` and `const_iterator` are conventional names. A collection of constant objects should return `const_iterator`.
2. begin() and end()
3. Class Structure
	Convention requires an iterator to support one fo the three categories described above. Also, it requires it to have the following data member:
	
	- `value_type` is a type alias to the element type
	- `iterator_category` is a tag name to one of the three categories described above.
	
	This is argurably no longer necessary with the introduction of `decltype` and `auto`.

### Example
#### Post-increment and Pre-increment
```

```

#### C++ style `typedef` = `using`
Prefer this to the C style `typedef`!
```cpp
using value_type = ElemType;
using iterator_category = std::bidirectional_iterator_tag;
```

