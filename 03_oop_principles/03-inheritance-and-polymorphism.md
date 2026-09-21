# 3. Inheritance, Polymorphism, and Composition

> **Learning goal:** Share behavior without making objects difficult to substitute, test, or change.

## Contents

- [Inheritance and overriding](#inheritance-and-overriding)
- [Super and method resolution order](#super-and-method-resolution-order)
- [Duck typing and protocols](#duck-typing-and-protocols)
- [Abstract base classes](#abstract-base-classes)
- [Composition and delegation](#composition-and-delegation)
- [Substitutability and design tradeoffs](#substitutability-and-design-tradeoffs)

## Inheritance and overriding

Inheritance is useful when a subclass truly supports its parent's contract. A subclass can extend or override inherited behavior.

```python
class Document:
    def __init__(self, title):
        self.title = title

    def describe(self):
        return self.title


class TaggedDocument(Document):
    def __init__(self, title, tags):
        super().__init__(title)
        self.tags = tuple(tags)

    def describe(self):
        return f"{super().describe()} [{', '.join(self.tags)}]"


document = TaggedDocument("Python notes", ["study", "reference"])
assert document.describe() == "Python notes [study, reference]"
assert isinstance(document, Document)
assert issubclass(TaggedDocument, Document)
```

If a subclass defines `__init__`, the parent's initializer is not automatically called. Call `super().__init__` when that initialization is required.

Avoid exact checks such as `type(value) is Base` unless exact type identity is genuinely the requirement. `isinstance` admits subclasses; duck typing often avoids a concrete type check altogether.

## Super and method resolution order

`super()` delegates to the **next implementation in the method resolution order (MRO)**, not simply to a named parent. This matters with multiple inheritance.

```python
class A:
    def route(self):
        return ["A"]


class B(A):
    def route(self):
        return ["B"] + super().route()


class C(A):
    def route(self):
        return ["C"] + super().route()


class D(B, C):
    def route(self):
        return ["D"] + super().route()


assert D().route() == ["D", "B", "C", "A"]
assert [cls.__name__ for cls in D.__mro__] == ["D", "B", "C", "A", "object"]
```

Python uses C3 linearization, preserving specified local ordering and consistent inheritance relationships. An inconsistent hierarchy raises `TypeError` at class creation.

Cooperative multiple inheritance requires compatible method signatures and a convention that each participant calls `super()` once as appropriate. Calling named parent implementations directly can bypass or duplicate work in a diamond hierarchy.

A **mixin** provides a focused reusable capability rather than representing a standalone domain entity. Keep mixins small, document required methods/attributes, and avoid hidden initialization dependencies. Deep hierarchies are often a sign that composition would be easier.

## Duck typing and protocols

Polymorphism allows the same client code to work with different compatible implementations. Python often uses duck typing: the required operation matters more than the exact class name.

```python
class UpperFormatter:
    def format(self, text):
        return text.upper()


class BracketFormatter:
    def format(self, text):
        return f"[{text}]"


def render(text, formatter):
    return formatter.format(text)


assert render("notes", UpperFormatter()) == "NOTES"
assert render("notes", BracketFormatter()) == "[notes]"
```

A `typing.Protocol` describes a structural interface for static type checkers without requiring implementations to inherit from it:

```python
from typing import Protocol


class Formatter(Protocol):
    def format(self, text: str) -> str:
        ...


def typed_render(text: str, formatter: Formatter) -> str:
    return formatter.format(text)


assert typed_render("notes", UpperFormatter()) == "NOTES"
```

Annotations are not automatic runtime enforcement. `@runtime_checkable` permits limited runtime protocol checks, but checks member presence rather than full signature compatibility or semantic correctness. See [typing protocols](https://docs.python.org/3/library/typing.html#typing.Protocol).

## Abstract base classes

An ABC defines a nominal interface and can share implementation. An abstract class cannot be instantiated until its abstract methods are implemented.

```python
from abc import ABC, abstractmethod


class Exporter(ABC):
    @abstractmethod
    def export(self, rows):
        """Return a serialized representation of rows."""


class LineExporter(Exporter):
    def export(self, rows):
        return "\n".join(str(row) for row in rows)


assert LineExporter().export([1, 2]) == "1\n2"
```

An abstract method may contain shared implementation, but subclasses still need to override it. ABCs do not automatically prove that signatures, returned values, and semantics match the promised contract.

| Approach | Relationship | Good fit |
|---|---|---|
| Duck typing | Required behavior at runtime | Small flexible APIs |
| Protocol | Structural typing | Typed libraries supporting unrelated implementations |
| ABC | Explicit inheritance or registration | Shared contracts and implementation |

See the [abc module](https://docs.python.org/3/library/abc.html) for abstract methods and virtual subclasses. Virtual registration does not copy methods into a class.

## Composition and delegation

Composition means an object **has a** collaborator rather than **is a** specialized version of it.

```python
class Report:
    def __init__(self, formatter):
        self.formatter = formatter

    def render(self, title):
        return self.formatter.format(title)


report = Report(BracketFormatter())
assert report.render("Quarterly") == "[Quarterly]"
report.formatter = UpperFormatter()
assert report.render("Quarterly") == "QUARTERLY"
```

Delegation forwards an operation to the collaborator. Dependency injection supplies that collaborator from outside, making the dependency visible and easy to replace in a test.

Composition often allows runtime variation and independent lifetimes. Inheritance often couples behavior to a fixed type hierarchy. Neither is universally superior; choose based on the contract and expected changes.

## Substitutability and design tradeoffs

A subtype should preserve the expectations clients rely on. It should not unexpectedly reject valid parent inputs, weaken promised results, or violate parent invariants. For example, a read-only collection should not claim a mutable interface whose `append` always fails without that restriction being part of the contract.

Questions before adding inheritance:

1. Can a caller use the child wherever the parent is expected?
2. Are the shared methods semantically the same, not merely similar code?
3. Would a composed helper make the dependencies clearer?
4. Does overriding require knowledge of fragile internal implementation details?
5. Can each implementation be tested against the same behavioral contract?

Practice: add a JSON exporter to the ABC example; express the same interface as a Protocol; implement logging as a composed wrapper; trace a diamond hierarchy and explain every `super()` step.

Next: [Special methods and protocols](04-special-methods-and-protocols.md).
