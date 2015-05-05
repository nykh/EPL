#EPL Lecture 23, 2015/4/30

## Structure of Project3

## Laws of simulation

### Encounter abstract method

The lifeforms spend most their time staying on some border.

When another lifeform crosses the border, it scans for closest lifeforms. If x is very close to more than one objects, it can just encounter one of them (the very closest one). This is unrealistic but we are ok with it.

```cpp
x->border_cross();
y = space.nearest(x.pos);
x_action = x->encounter(x->info_about(y));
// no time has elapsed since x->encounter
y_action = y->encounter(y->info_about(x);
roll = drand48();
if(roll < eat_success_chance(x->energy, y->energy)) x->eats(y);
roll = drand48();
if(roll < eat_success_chance(y->energy, x->energy)) y->eats(x);

```

Because there is no time between the two function, x doesn't really have a substantial advantage.

The lifeforms are not allowed to know absolute location, but they can know the what's close to you. No global variable, you can only do message passing by the encountered object's name as `std::string species`. **However**, you are allowed to put whatever into the species name.

### Kill

It now becomes easy to kill off lifeform because of smart pointer.

When x eats y

```cpp
y->is_alive = false;
y->die();

x->gain_energy(y->energy * eat_efficiency);
```

### stale data

In this virtual world, the object info are updated only when there is an event that happen on them. 

But since you are moving and crossing borders, it is more likely that anything close to you will already been affected by a resizing event, and therefore has current info.

### perceive

### reproduce

## How to write function object template

Design the tmeplate so that

```cpp
template <typename T>
class Fun {
	___ op()(___) {}
};
```

We can declare

```cpp
Fun<<oid(double)> f = ...;
```

How can we extract the `void` and `double` from the composite type `void(double`?

Left as exercise.