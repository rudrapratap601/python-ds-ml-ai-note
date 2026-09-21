# 1. Classes and Objects

> **Learning goal:** Model related state and behavior with clear object contracts. Start here, then follow the [OOP reading order](README.md).

## Contents

- [What OOP means](#what-oop-means)
- [Classes, instances, and methods](#classes-instances-and-methods)
- [Instance and class attributes](#instance-and-class-attributes)
- [Construction and object lifetime](#construction-and-object-lifetime)
- [Instance, class, and static methods](#instance-class-and-static-methods)
- [Identity, equality, and copying](#identity-equality-and-copying)
- [Modeling and practice](#modeling-and-practice)

## What OOP means

Object-oriented programming organizes behavior around objects with state and an interface. A class defines a kind of object; an instance is a particular object created from that class.

Python supports OOP alongside procedural and functional styles. A class is useful when a concept has state, invariants, and several related operations. A simple calculation may be clearer as a function.

| Term | Meaning |
|---|---|
| Attribute | A value or behavior accessed through an object |
| Method | A function bound through an object or class |
| Encapsulation | Keep state changes behind an intentional interface |
| Abstraction | Expose useful operations while hiding unnecessary details |
| Inheritance | Define a class in terms of another class |
| Polymorphism | Use different objects through a compatible interface |
| Composition | Build an object from collaborating objects |

## Classes, instances, and methods

```python
class Course:
    """A course with an ordered collection of unique lesson names."""

    def __init__(self, title):
        if not title.strip():
            raise ValueError("title cannot be empty")
        self.title = title
        self.lessons = []

    def add_lesson(self, name):
        if not name.strip():
            raise ValueError("lesson name cannot be empty")
        if name in self.lessons:
            raise ValueError("lesson already exists")
        self.lessons.append(name)

    def lesson_count(self):
        return len(self.lessons)


python = Course("Python")
python.add_lesson("Functions")
python.add_lesson("Classes")
assert python.lesson_count() == 2
assert Course.lesson_count(python) == 2
```

`self` is the conventional first parameter of an instance method. Python supplies the instance when you call a bound method: `python.lesson_count()` behaves like `Course.lesson_count(python)` for this ordinary method.

`self` is not a reserved keyword, but using another name makes code harder to read. Each instance can hold different state while sharing the class's method definitions.

This first example exposes a mutable `lessons` list. That is acceptable for introducing attributes, but callers can bypass its validation. The [encapsulation chapter](02-encapsulation-and-properties.md) improves that design.

## Instance and class attributes

```python
class Notebook:
    category = "study"  # Shared class attribute.

    def __init__(self, owner):
        self.owner = owner
        self.pages = []  # A fresh list for every instance.


first = Notebook("Asha")
second = Notebook("Dev")
first.pages.append("OOP")
assert second.pages == []
assert first.category == second.category == "study"
first.category = "work"  # Instance attribute shadows the class attribute.
assert second.category == Notebook.category == "study"
```

Put per-instance mutable state in `__init__`, not in a shared class-level list. Class attributes are appropriate for constants or deliberately shared configuration.

For ordinary attributes, lookup may fall back from the instance to its class and base classes. Descriptors such as properties affect precedence; the full rules appear in the [advanced chapter](06-descriptors-and-class-customization.md).

`obj.attribute = value` normally sets an instance attribute, whereas `Class.attribute = value` modifies the class. Mutating a shared class-level list through an instance still mutates the shared object.

## Construction and object lifetime

Calling a class normally invokes creation through `__new__`, then initialization through `__init__` when the returned object is an instance of the class. Most user classes only implement `__init__`.

- `__new__` creates/returns the instance; it matters for immutable subclasses and specialized construction.
- `__init__` initializes an already created instance and must return `None`.
- `__del__` is a finalizer, not a reliable substitute for explicit resource management.
- Use context managers or `close()` for files, sockets, and similar resources.

CPython uses reference counting plus cyclic garbage collection, but portable code must not depend on the exact moment an object is finalized. Cycles and interpreter shutdown complicate finalization. The [Python data model](https://docs.python.org/3/reference/datamodel.html#objects-values-and-types) defines the language-level object concepts.

## Instance, class, and static methods

```python
class Rectangle:
    def __init__(self, width, height):
        if width <= 0 or height <= 0:
            raise ValueError("dimensions must be positive")
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    @classmethod
    def square(cls, side):
        return cls(side, side)

    @staticmethod
    def unit_name():
        return "square units"


rectangle = Rectangle.square(4)
assert rectangle.area() == 16
assert Rectangle.unit_name() == "square units"
```

| Method kind | Automatic first argument | Good use |
|---|---|---|
| Instance | `self` | Read/change instance state |
| Class | `cls` | Alternate constructors and class-aware factories |
| Static | None | Closely related utility needing no object state |

Using `cls(...)` in an alternate constructor supports compatible subclasses. A static method is optional organization; a module-level helper is often equally clear.

Python does not provide Java-style method overloading by writing several methods with the same name. The last definition replaces earlier ones. Use default parameters, separate descriptive methods, or suitable dispatch tools.

## Identity, equality, and copying

```python
from copy import copy, deepcopy

original = {"topics": ["Python"]}
alias = original
shallow = copy(original)
deep = deepcopy(original)
original["topics"].append("OOP")
assert alias is original
assert shallow is not original
assert shallow["topics"] == ["Python", "OOP"]
assert deep["topics"] == ["Python"]
```

Assignment shares a reference. A shallow copy creates a new outer object but shares nested references. A deep copy recursively copies supported state and tracks already copied objects; it is not universally appropriate for files, locks, database connections, or identity-sensitive objects.

Unless customized, ordinary user objects compare by identity. Implement equality only when the domain has meaningful value semantics; hashing must stay consistent with equality. See [special methods](04-special-methods-and-protocols.md).

## Modeling and practice

Start by writing a sentence describing the object and its responsibilities. Define valid states before adding methods. Separate calculations, persistence, and presentation when they change for different reasons.

Avoid classes with no meaningful state or behavior, large classes that handle every concern, and mutable class attributes used accidentally as instance data. Type annotations describe intent but do not validate values at runtime.

Practice: create a `Playlist` with add/remove operations; explain the difference between two equal value objects and two references to the same instance; add an alternate constructor; demonstrate and fix a shared-list class attribute bug.

Next: [Encapsulation and properties](02-encapsulation-and-properties.md).
