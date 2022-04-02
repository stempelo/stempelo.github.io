# Design Patterns
- program to an interface, not an implementation
- favor compostion over inheritance
## Object-oriented design
- Inheritance
- Polymorphism
- Abstraction
- Encapsulation
## Benefits of design patterns
- Not reinventing the wheel
- Building software that is resilient to change
## What are design patterns
- Experience
- They are not algorithms
- They are not code
They are defined by:
- a definition
- a class diagram
Design patterns are:
- not specific solutions for any kinds of software
- general solutions for common problems
# The strategy pattern
## Inheritance
If **all** your class relationships are IS-A relationships, take a closer look at your design.
**Not** overuse inheritance. If you overuse inheritance, your design become inflexible and very difficult to change.
## Interfaces
An interface:
- defines the methods an object must have in order to be considered a particular type
- is an abstract type that specifies a behavior tha classes must implement
- allow different classes to share similarities
Not all classes need to have the same behavior
Problem with the use of only interfaces:
- destroy code reuse
- maintenance problem on all classes that implement an interface
- doesn't allow for runtime changes in behaviors, for example if a class need to implement another interfaces
Starting from this principle:
    ***Encapsulate what varies***
    identify the aspects og your application that vary and separate them from stays the same
This principle is fundamental to almost every design pattern.
If some aspect of your code is changing, that's a sign you should pull it out and separate it so you can extend or alter them whothout affecting the rest of the code.

Starting from this second principle:
    ***Program to an interface, not an implementation***
    Clients remain unaware of the specific types of object they use, as long as the objects adhere to the interface that clients expect

**The Strategy Pattern**
    This pattern defines a family of algorithms, encapsulate each one, and makes them interchangeable. This lets the algorithm vary independently from clients that use it.

Remember: **HAS-A is better than IS-A**

Principle:
    **Favor composition over inheritance**
    Classes should achieve code reuse using composition rather than inheritance froma superclass.

Schema



  







  