# Interfaces and abstraction

## Overview

Interfaces and abstraction are key concepts in modular code design. Abstraction allows you to hide complex implementation details and show only the necessary features. Interfaces define a contract that classes can implement, ensuring a consistent API.

### Abstraction

Abstraction is a fundamental concept in programming that enables simplification by reducing the complexity of systems. It allows developers to focus on high-level design without needing to delve into intricate implementation details at every step. By concealing complex realities through simpler interfaces, abstraction promotes cleaner and more understandable code.

One of the ways abstraction aids in simplification is through de-duplication. By abstracting common patterns or functionalities into single units or interfaces, repeated code is minimized, which not only reduces errors but also enhances maintainability and readability of the code.

Abstraction also supports deferred execution and decision-making by providing mechanisms to define operations abstractly. This means certain processes can be implemented at a later phase or modified without impacting the overall system, allowing for flexibility and scalability in software development.

See this [Training - Data Designer](https://docs.google.com/document/d/e/2PACX-1vSxclErigJAaf8hZEPsj1pyLBZ51gjuG909I42GxSBCBshulz9lxB7E2EgrXJBDbvHyJgMpbpN5c65_/pub) document that demonstrates how to use object-oriented programming and modularization to achieve separation of data and code.

### Modularization

Modularization is a design technique that involves splitting a large software system into separate, interchangeable modules with specific functionalities, thereby achieving higher levels of abstraction and reducing complexity.

- **Achieves Abstraction:** By focusing on specific functionalities, modules encapsulate details and hide them behind well-defined interfaces. This helps minimize the knowledge a programmer must know about the system as a whole.
- **Minimizes Merge Conflicts:** With clear boundaries, teams can work on different modules concurrently with less risk of overlapping changes, thereby reducing merge conflicts and promoting parallel development.

In practice, modularization can significantly improve the maintainability and scalability of a system, allowing developers to update or enhance individual modules with minimal impact on others.

See this [Training - Team Lead](https://docs.google.com/document/d/e/2PACX-1vQs8WzASDjSAtwd4x2jIG3iIz6Ol7N-kqZv4hb4_M3F7caY9Jb7OjVwY3nKcXNAATn_r98orXhCsNXM/pub) document that demonstrates how to use modularization to organize a team project for making a game.

See this [Training - Game Programmer](https://docs.google.com/document/d/e/2PACX-1vTDe4MmI-z9iy7CjaZFyTKsD8Npp5TXGZJGlUVrYcIHG2sczzfdtUy6l3oWR_8T88HkoNMr-LickfJa/pub) document that demonstrates how to use modularization to organize functions for game development.

### SOLID

Besides encapsulation, inheritance, and polymorphism, there are other sets of principles for guiding object-oriented programming. [SOLID](https://www.digitalocean.com/community/conceptual-articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design) is another set of 5 principles, first described by Robert C. Martin.

### Single-responsibility Principle

An entity should have one, and only one, reason to change. This principle emphasizes that a class should only have a single responsibility pertaining to a specific functionality in a program.

### Open-closed Principle

Software entities like classes and methods should be open for extension but closed for modification. This means that you should be able to add new functionality to a class without altering its existing code.

### Liskov Substitution Principle

Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program. This principle ensures that derived classes extend the base class without changing its behavior.

### Interface Segregation Principle

A client should not be forced to implement interfaces it does not use. This principle suggests that many specific interfaces are better than a large, general-purpose interface.

### Dependency Inversion Principle

High-level modules should not depend on low-level modules. Both should depend on abstractions. Additionally, abstractions should not depend on details, but details should depend on abstractions. This principle underscores the importance of relying on interfaces or abstract classes rather than concrete implementations.

See this [Training - Quality Assurance](https://docs.google.com/document/d/e/2PACX-1vSK4lEIrhhIrYfHB0IEejAZLPKlrnTN527epFxiYUTcfclF_IcRzwMkL_eEJJZaqWiv_wWyGeS6_9jy/pub) document that demonstrates how to apply some of the SOLID principles to unit testing.

## Conclusion

While the above principles are common ways to achieve abstraction where it is necessary, developers should note that abstraction does not always bringa net benefit. Applying abstraction is a tradeoff: while it brings a reduction in duplicated code and generalization of code, it also splits up application logic into multiple places, and lengthens the chain of logic for debugging, which can make it harder to understand the flow of data.

Developers and teams should continually assess the marginal benefit of each additional abstraction to decide if it would be beneficial to the codebase.
