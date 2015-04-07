# Project 2 Design/Implementation note

## Concepts in the problem domain

### our `vector_concept` (in math sense)

- These are inherited from `vector` class
	- push/pop back/front
	- `[]` operator
- Arithmetic operations
	- \+, \-, \*, /
	- negate
- can be iterated over
    - `Iterator`

The following are specific of the `vector` class

- `size()`
- `emplace_back`/`front`
- Functional operations
	- reduce
	- apply
	- (?) filter
- example of apply: `sqrt`

What can implement a vector concept:

- vector
- Proxy
- scalar




### Proxy

is a placeholder for a node in the evaluation tree.

- subnode(s)
  	
    These nodes can either be `vector` or other `Proxy` (or `Scalar`)
	- **(Binary)** Left, Right
	- **(Unary)**  Child
- operation
- ~~can be evaluated: `eval`: evaluate subnodes and perform the **operation** on them~~
	I think this can just be done with a combination of `operator[]` and iterator.

### scalar

In the evalutation tree we can have scalar object to support things like `x+5`.

- Evaluation: just return the scalar value throughout the loop




## Problem in implementation

- template function won't automatically upcast to `scalar` from scalar type.


### How to represent the operation

Should be in a composition (*has-a*) relationship with `Proxy`. I think we can just use `std::function`

But what should the signature be? Let's think

| Left | Right |  operation::
|-------|--------|
|  int      |  int      | int, int => int |
| double    |   int     | double, double => double |
| double    | double    | double, double => double |
| `complex<double>`  | `complex<double>`  |  `complex<double>&, complex<double>& => complex<double>`   |
|  double   | `complex<double>`  | `complex<double>&, complex<double>& => complex<double>`   |
### Should type conversion be supported for Unary Proxy?
For example,

```cpp
double my_sqrt(int a) { return std::sqrt(a); }
valarray<int> x { 1,2,3 };
auto y = x.apply(my_sqrt);
```

Can this work to transform `int` into `double`?

Right now my implementation says **NO**
