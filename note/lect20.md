#EPL Lecture 20, 2015/4/21

## Project 3

1. Phase I: Simulatror
2. Phase II: Implement the model

The simulator will be an Abstract Base Class (abc).

The simulator will simulate an eco-system, with various lifeforms in a disk.

By the way, Chase thinks there are a few things students should have written during college:

1. Writing a micro-kernal
2. Compiler or interpreter
3. event-driven simulator

The lifeforms can

- see
- eat food
- move (accelerate-deaccelerate)
- can control when exactly and what kind of offspring it will bear

Factory method in the class should be implemented to make new objects easily.

We want to embed as many common behavior into the ABC as possible. For example `move()`, but this will require it to have local states (posiitons, etc). This means the Lifeform ABC is actually not purely virtual.

```cpp
class Lifeform {
private:
	double xpos, ypos;
    Lifeform();    // this prevents you to cheat
    Lifeform(...); // other constructors
public:
    virtual ~Lifeform();
	virtual string species_name(void) const;
};
```

### Detour: invoking methods from within constructor

If I have

```cpp
class Craig : public Lifeform {
	...
public:
	Craig() { ... }
};
```

When we write a constructor, we might have to call another function. What happens when a constructor invokes a virtual function, but the virtual function depends on some state in the class to fun correctly?

```cpp
class Craig {
	Craig() { doit(); }
    virtual doit()
}
```

Pure virtual functions are useful to fail people to override them.

### Dynamic Polymorphism

All behavior of Lifeform should be done polymorphically.

## Simulators

There are two kinds of simulators.

- discrete time simulator
- discrete event-driven simulator

They differ in the central question: How does computer update the state of the simulation. In Discrete Time Simulation, the computer updates states globally by a rate of time.  In Discrete Event-Driven Simulation, however, we don't need a tight coupling beteween real time and the virtual time.

For example our Lifeforms can move, they may update the behavior un-uniformly. When they are far away they may all be making simple decisions (moving etc), but as they approach each other, their behaviors can become very complex (fight or flight), and the precision of the time scale should reflect that complexity.

So what if we flip the relation around, instead of event happens based on time, **time flows based on events**.

###  Software structure of Discrete Event-driven Simulatoor

```cpp
int main() {
	initialize_world();
    while(No Winner) {
    	Event e = queue.next();
        now = e.time_stamp();
        e.handler();
    }
}

void initialize_world() {
	initialize_world_state();
    prime_event_queue();
}
```