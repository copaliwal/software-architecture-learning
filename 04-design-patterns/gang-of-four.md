# Design Pattern - Gang of Four (GoF)

The **Gang of Four (GoF)** refers to four authors (Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides) who wrote the famous book _“Design Patterns: Elements of Reusable Object-Oriented Software”_.

They introduced **23 design patterns** that solve common software design problems.  
These are grouped into **three categories**: **Creational, Structural, Behavioral**.

## 1\. **Creational Patterns - (How objects are created)**

Help you **create objects in flexible ways** instead of directly using `new`.

These deal with object creation mechanisms.

| Pattern | Description | Common Use Case |
| --- | --- | --- |
| [**Singleton**](1-creational-patterns/5-singleton.md) | Ensures a class has only one instance | Logging, configuration, caching |
| [**Factory Method**](1-creational-patterns/1-factory-method.md) | Creates objects without specifying exact class | Dependency injection, extensibility |
| [**Abstract Factory**](1-creational-patterns/2-abstract-factory.md) | Creates families of related objects - Difference between [Abstract Factory vs Factory method](1-creational-patterns/2.1-abstractfactory-vs-factory-method.md) | UI themes, cross-platform toolkits |
| [**Builder**](1-creational-patterns/3-builder-pattern.md) | Constructs complex objects step-by-step | Fluent APIs, configuration builders |
| [**Prototype**](1-creational-patterns/4-prototype.md) | Clone existing objects instead of creating new ones | Object copying, undo functionality |

## 2\. **Structural Patterns - (How objects/classes are composed)**

Structural patterns concern class and object composition. They help ensure that if one part of a system changes, the entire structure does not need to do so. Help you **organize and connect classes/objects** for easier maintenance.

These help compose classes and objects.

| Pattern | Description | Common Use Case |
| --- | --- | --- |
| [**Adapter**](2-structural-patterns/1-adapter.md) | Converts one interface to another, make incompatible classes work together | Legacy integration |
| [**Bridge**](2-structural-patterns/2-bridge.md) | Decouples abstraction from implementation | UI rendering, device drivers |
| [**Composite**](2-structural-patterns/3-composite.md) | Tree structures where individual & groups are treated the same | Tree structures, menus |
| [**Decorator**](2-structural-patterns/4-decorator.md) | Adds behavior dynamically | Logging, validation, caching |
| [**Facade**](2-structural-patterns/5-facade.md) | Simplifies complex subsystems | API wrappers, service gateways |
| [**Flyweight**](2-structural-patterns/6-flyweight.md) | Shares common state to reduce memory | UI elements, game objects |
| [**Proxy**](2-structural-patterns/7-proxy.md) | Controls access to an object | Lazy loading, security, remote access |

## 3\. **Behavioral Patterns - (How objects interact)**

Behavioral patterns deal with algorithms and the assignment of responsibilities between objects.

These manage algorithms, relationships, and responsibilities.

| Pattern | Description | Common Use Case |
| --- | --- | --- |
| [**Observer**](3-behavioral-patterns/1-observer.md) | Notifies dependents of state changes | Event handling, UI updates |
| [**Strategy**](3-behavioral-patterns/2-strategy.md) | Selects algorithm at runtime | Sorting, validation, business rules |
| [**Command**](3-behavioral-patterns/3-command.md) | Encapsulates a request as an object | Undo/redo, task queues |
| [**Chain of Responsibility**](3-behavioral-patterns/4-chain-of-responsibility.md) | Passes request along a chain | Middleware, logging pipelines |
| [**State**](3-behavioral-patterns/5-state.md) | Alters behavior based on internal state | Workflow engines, UI controls |
| [**Template Method**](3-behavioral-patterns/6-template-method.md) | Defines skeleton of an algorithm | Base classes, extensibility hooks |
| [**Mediator**](3-behavioral-patterns/7-mediator.md) | Centralizes communication between objects | UI coordination, messaging |
| [**Memento**](3-behavioral-patterns/8-memento.md) | Captures and restores object state | Undo functionality |
| [**Interpreter**](3-behavioral-patterns/9-interpreter.md) | Implements a grammar | Expression parsing, rule engines |
| [**Iterator**](3-behavioral-patterns/11-iterator.md) | Traverses a collection without exposing its representation | Collection APIs, tree traversal |
| [**Visitor**](3-behavioral-patterns/10-visitor.md) | Adds operations to object structure | AST traversal, serialization |

These patterns are widely used in object-oriented design to solve recurring design problems and create more flexible, maintainable, and scalable software.