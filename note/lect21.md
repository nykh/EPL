#EPL Lecture 21, 2015/4/23

## Schedule

1. move, collision
2. smart ptr

### Project 3

Phase A: framework
Phase B: designing a Lifeform

## Simulating event

We want to avoid making time and space discrete for realistic simulation.

### Question

These two questions are equal:

1. given Lifeform X, what is the closest LifeForm to X?

2. given a circle (described as center position and radius) yield a list of Lifeform inside that circle.

### Solution

Pseudo-solution:
```
foreach Lifeform a:
  if distance(a, X) < ARBITRARY_CLOSENESS: close .= a;
```
This is $$$ O(n^2) $$$, far from satisfactory.

Instead we use a better data structure: QuadTree

### Datastructure: QuadTree

from [Wiki article on QuadTree](http://en.wikipedia.org/wiki/Quadtree)

> A quadtree is a tree data structure in which each internal node has exactly four children. Quadtrees are most often used to partition a two-dimensional space by recursively subdividing it into four quadrants or regions.

## Smart Pointer

It is modeled on the standard shared pointer. This is the thing to use for any serious C\++ developer.

In modern C\++, one should never create ` T* p = new ...` or `delete p`. Those are two dangerous. Instead, we use shared pointer

```cpp
shared_ptr<T> p = make_shared<T> (...);
```

`smart_ptr` is an object that can be used just like a regular `T*`. (supports `->` operator, etc). This eliminates the need to call `delete`.

### Principle of operation

There is a global **control block** associated with a pointer. p actually doesn't point at the object directly, but points to the control block, and the control block contains pointer to the actual object. The control block also keeps track of the **number of pointer** that points at the object. This information is used for garbage collection.

Copy constructor or assignment will increment the "number of pointer".

```cpp
shared_ptr<T> q = p;
```

When the number of pointer pointing to an object becomes zero. It can be GC'ed.

### Problem: Cyclical in a reference

```cpp
Node* x1 = new Node(), *x2 = new Node();
x1->next = x2; x2->next = x1;

Node *l = x1;
```

Even as `l` goes out of scope and the linked structure becomes unreacheable, the number of pointer on each node is still non-zero. This will prevent GC from working.