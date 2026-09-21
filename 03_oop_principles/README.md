# Object-Oriented Python: Basic to Advanced

This section follows one progression: model objects, protect their contracts, compose implementations, use Python protocols, then learn advanced customization and test a complete design.

## Reading order

| Chapter | Guide | What you will learn |
|---|---|---|
| 1 | [Classes and objects](01-classes-and-objects.md) | Instances, attributes, methods, construction, identity, copying |
| 2 | [Encapsulation and properties](02-encapsulation-and-properties.md) | Public interfaces, validation, invariants, mutable state |
| 3 | [Inheritance and polymorphism](03-inheritance-and-polymorphism.md) | Overriding, MRO, super, ABCs, Protocols, composition |
| 4 | [Special methods and protocols](04-special-methods-and-protocols.md) | Representation, equality/hash, operators, containers, context managers |
| 5 | [Dataclasses and type modeling](05-dataclasses-and-type-modeling.md) | Records, frozen/slots behavior, enums, typed dictionaries, serialization |
| 6 | [Descriptors and class customization](06-descriptors-and-class-customization.md) | Attribute precedence, binding, subclass hooks, metaclasses |
| 7 | [Design principles and patterns](patterns.md) | SOLID, strategy, factories, adapters, repositories, design tradeoffs |
| 8 | [Mini-project and testing](08-oop-project-and-testing.md) | Experiment results, dependency injection, validation, behavioral tests |

## Prerequisites and examples

Know [functions](../01_core_python/functions-lambda-map-filter.md), [namespaces](../02_advanced_python/namespaces-and-scope.md), [exceptions](../02_advanced_python/exception-handling.md), and basic collections. Examples use Python 3.11+ and the standard library. Run examples in document order within each chapter. The final project is self-contained.

## Revision route

Revisit method kinds, class versus instance attributes, properties, MRO, composition versus inheritance, equality/hash consistency, ABCs versus Protocols, shallow versus deep immutability, and when a descriptor or metaclass is justified.

For practical work, start with the [mini-project](08-oop-project-and-testing.md), change one requirement, and identify which component should own it. A good object model makes those changes local and testable.
