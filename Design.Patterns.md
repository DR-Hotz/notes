# Design Patterns

+   Program to an interface, not an implementation
+   Favor object composition over class inheritance
    *   Parent implementation can NOT be changed at run-time
    *   Inheritance breaks encapsulation coz it exposes a subclass to details of its parent's implementation
    *   Object composition is defined dynamically at run-time
    *   Composition requires objects to respect each other's interface


## Delegation

+   A receiving object(A) delegates operations to its delegate(B)
    *   A passes itself to B, let the delegated operation refer to the receiver(A)


## Redesign Causes

+   Creating an object by specifying a class explicitly, Specifying  a class name commits you to a particular implementation instead of a particular interface
    *   Tips : create objects indirectly
    *   Design patterns : Abstract Factory, Factory Method, Prototype

+   Dependence on specific operations, where you commit to one way of satisfying a request
    *   Design patterns : Chain of Resposibility, Command

+   Dependence on hardware and software platform
    *   Design patterns : Abstract Factory, Bridge

+   Dependence on object representations or implementations, clients know how an object is represented, stored, located or implemented.
    *   Design patterns : Abstract Factory, Bridge, Memento, Proxy

+   Algorithmic dependencies
    *   Design patterns : Builder, Iterator, Strategy, Template, Method, Visitor

+   Tight coupling, classes that are tightly coupled are hard to reuse
    *   Design patterns : Abstract factory, Bridge, Chain of responsibility, Command, Facade, Mediator, Observer

+   Extending functionality by subclassing may cause subclassing for just a tiny change.

+   Inability to alter classes conveniently
    *   Design patterns : Adapter, Decorator, Visitor



## Composite

Intent:
+   Compose objects into tree structures to represent part-whole hierarchies
+   Let clients treat individual objects and compositions of objects uniformly
+   Emphasize **transparency over safety** in this design pattern

Bad@:
+   can make your design overly general
+   it's hard to restrict the components of a composite(type system won't help you coz all components are treated equally), you have to use rum-time checks instead.

Tis:
+   Keep a component's parent reference in every component, only modify it when adding or removing any component
+   Maximize the Component interface, even though some interfaces do NOT have meaning in either Leaf or Composite
+   Declare the child management operations in Component rather than composite subclasses, use "Component getComposite()" to return itself(same as run-time checking)
+   Should component keep a list of its related components?
+   Child ordering - **Iterator**
+   Caching to improve performance, cache info in composite
+   Who should delete components?
+   What's the best data structure for storing components? - **Interpreter**


## Strategy(Policy)

Intent:
+   Define a family of algorithms, encapsulate each one, and make them interchangeable
+   Lets the algorithm vary independently from clients that use it

Bad@:
+   Clients must be aware of different Strategies, hence clients might be exposed to implementation issues, therefore, use the Strategy pattern only when the variation in behavior is relevant to clients
+   Communication overhead between Strategy and Context, some info in Context may be never used by some concrete strategies, but Context has to instantiate them all the time
+   Increased number of objects -  implement strategies as stateless objects that contexts can share - **Flyweight**

Tips:
+   Define the Strategy and Context interfaces for info exchange
    *   Context pass itself to Strategy
    *   Context pass all the needed info
    *   The strategy can store a reference to its context
+   Making Strategy objects optional


## Decorator(Wrapper)

Intent:
+   Attach additional responsibilities to an object dynamically
+   Add responsibilities to individual objects, not to an
entire class

Bad@:
+   narito936530
+   NARITO936530