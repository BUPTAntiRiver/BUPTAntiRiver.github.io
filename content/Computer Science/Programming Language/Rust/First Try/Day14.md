# Object-Oriented Programming

Though we don't have something like `class` explicitly, but we have the idea of Objected-Oriented Programming. We have `pub` keyword for `struct` which can hold methods and we can `impl` the methods of that `struct`, so that we achieve the requirements of OOP: An **object** packages both data and the procedures that operate on that data. The procedures are typically called **methods** or **operations**.

## Inheritance

_Inheritance_ is a mechanism whereby an object can inherit elements from another object's definition, thus gaining the parent object's data and behavior without you copying the code. However Rust does not have exactly this idea, but if you want to use inheritance is some extent, we can use trait to re-implement a method of some `struct`.

Actually the idea of inheritance is that the child class can be used at the same places as the parent type, which means you can substitute multiple objects for each other at runtime if they have the share certain characteristics. Like in Python, if some object acts like a duck than it is a duck and we can apply methods on ducks on it.

However inheritance usually shares more code than necessary which can make the program design less flexible, so Rust has some trade-off, we don't use common inheritance but the trait system.

So how do we work with a scenario like we have many components on screen like buttons, select boxes and so on. They all have a `draw` methods to print them on the screen. In inheritance programming languages, we can just define `Draw` class and other classes respectively. With trait, we first define a trait and `impl` corresponding methods `for` these different `struct`s.
