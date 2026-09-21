# 4. Special Methods and Python Protocols

> **Learning goal:** Make custom objects work naturally with Python syntax while preserving the expectations of built-in operations.

## Contents

- [What special methods do](#what-special-methods-do)
- [Representation and value semantics](#representation-and-value-semantics)
- [Operators and reflected operations](#operators-and-reflected-operations)
- [Containers and iteration](#containers-and-iteration)
- [Context managers](#context-managers)
- [Callable objects](#callable-objects)
- [Attribute customization and pitfalls](#attribute-customization-and-pitfalls)

## What special methods do

Special methods, often called dunder methods, implement protocols behind syntax such as `len(obj)`, `obj[key]`, `a + b`, `with obj`, and `for item in obj`.

Implement the corresponding methods and use the normal syntax in client code. Python commonly looks up implicit special operations on the type, so assigning a special method only to one instance may not affect the operator.

| Syntax | Main protocol methods |
|---|---|
| `repr(obj)`, `str(obj)` | `__repr__`, `__str__` |
| `len(obj)`, `bool(obj)` | `__len__`, `__bool__` |
| `a == b`, `a < b` | `__eq__`, `__lt__` and related comparisons |
| `hash(obj)` | `__hash__` |
| `a + b`, `b + a` | `__add__`, `__radd__` |
| `obj[key]` | `__getitem__`, optionally `__setitem__`, `__delitem__` |
| `item in obj` | `__contains__` or iteration fallback |
| `iter(obj)`, `next(obj)` | `__iter__`, `__next__` |
| `obj(...)` | `__call__` |
| `with obj` | `__enter__`, `__exit__` |
| `async with`, `async for` | Async context and iteration protocols |

The [data model reference](https://docs.python.org/3/reference/datamodel.html#special-method-names) is the authoritative protocol catalog. Implement only operations that make sense for the object.

## Representation and value semantics

`__repr__` should be useful for debugging; `__str__` should be suitable for human-facing display. Both must return strings. Avoid exposing secrets through either representation.

```python
class Label:
    def __init__(self, text):
        self.text = text

    def __repr__(self):
        return f"Label({self.text!r})"

    def __str__(self):
        return self.text

    def __eq__(self, other):
        if type(other) is not type(self):
            return NotImplemented
        return self.text == other.text


assert str(Label("study")) == "study"
assert repr(Label("study")) == "Label('study')"
assert Label("study") == Label("study")
```

Returning `NotImplemented` for unsupported operand types lets Python try the other operand's implementation or fallback rules. `NotImplemented` is a special value, not the exception `NotImplementedError`.

Equal hashable objects must have equal hashes. A key's equality/hash-relevant state must not change while it is in a set or dictionary. Defining `__eq__` without an appropriate `__hash__` normally makes instances unhashable. Mutable value objects should usually remain unhashable.

Use frozen dataclasses for simple immutable value records when appropriate; see [data modeling](05-dataclasses-and-type-modeling.md).

## Operators and reflected operations

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Vector2:
    x: float
    y: float

    def __add__(self, other):
        if not isinstance(other, Vector2):
            return NotImplemented
        return Vector2(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):
        if not isinstance(scalar, (int, float)):
            return NotImplemented
        return Vector2(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        return self * scalar


assert Vector2(1, 2) + Vector2(3, 4) == Vector2(4, 6)
assert 3 * Vector2(1, 2) == Vector2(3, 6)
assert len({Vector2(1, 2), Vector2(1, 2)}) == 1
```

Reflected operations allow cases where the left operand does not implement the operation for the right type. `__iadd__` controls in-place addition when supplied; without a suitable implementation, `+=` can fall back to addition and rebinding. In-place syntax does not always mean mutation.

Do not give `+` an unexpected meaning such as saving a file. Consistent operators should preserve intuitive algebraic behavior where the domain supports it.

## Containers and iteration

```python
from collections.abc import Sequence


class ReadingQueue(Sequence):
    def __init__(self, titles):
        self._titles = tuple(titles)

    def __len__(self):
        return len(self._titles)

    def __getitem__(self, index):
        return self._titles[index]


queue = ReadingQueue(["Python", "OOP", "NumPy"])
assert len(queue) == 3
assert queue[-1] == "NumPy"
assert queue[1:] == ("OOP", "NumPy")
assert "OOP" in queue
assert list(queue) == ["Python", "OOP", "NumPy"]
```

`Sequence` supplies useful mixins given the required operations. Supporting slices means handling a `slice` object, not just integers; delegating to a tuple provides that behavior here.

`__len__` must return a nonnegative integer. `bool(obj)` first uses `__bool__` when present, then length when available; otherwise an ordinary object is truthy. `__bool__` must return a boolean.

A container should usually produce a fresh iterator for each traversal. An iterator returns itself from `__iter__` and raises `StopIteration` on exhaustion. These are different contracts; see [generators and iterators](../02_advanced_python/generators-and-iterators.md).

## Context managers

```python
class RecordingContext:
    def __init__(self, events):
        self.events = events

    def __enter__(self):
        self.events.append("enter")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self.events.append("exit")
        return False  # Do not suppress exceptions.


events = []
with RecordingContext(events):
    events.append("body")
assert events == ["enter", "body", "exit"]
```

The `as` target receives the return from `__enter__`, not necessarily the manager itself. If `__enter__` raises, that manager's `__exit__` is not invoked; acquisition code must handle its own partial failures. `__exit__` sees exception details and may suppress the exception only by returning a truthy value.

Use `contextlib.contextmanager` for simple acquire/yield/release patterns and `ExitStack` for dynamic resource collections. Async managers use `__aenter__` and `__aexit__` and are awaited.

## Callable objects

```python
class Threshold:
    def __init__(self, minimum):
        self.minimum = minimum

    def __call__(self, value):
        return value >= self.minimum


passes = Threshold(60)
assert passes(75) is True
assert passes(40) is False
```

Callable instances combine configuration and behavior. A closure can represent the same small idea; choose a class when additional state management or operations make it clearer.

## Attribute customization and pitfalls

`__getattr__` runs after ordinary attribute lookup fails. `__getattribute__` participates in every ordinary attribute lookup, so a naive access to `self.name` inside it can recurse. `__setattr__` similarly needs careful delegation, often via `object.__setattr__`.

Prefer properties and descriptors before overriding all attribute access. Preserve `AttributeError` for missing attributes so `hasattr` and inspection behave correctly.

Practice: implement an immutable coordinate value; create a read-only sequence; write a context manager that records cleanup during an exception; explain why `NotImplemented` and `NotImplementedError` are different; test equality symmetry and hashing consistency.

Next: [Dataclasses and type modeling](05-dataclasses-and-type-modeling.md).
