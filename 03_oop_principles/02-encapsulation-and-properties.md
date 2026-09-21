# 2. Encapsulation, Properties, and Invariants

> **Learning goal:** Make valid object states easy to create and invalid transitions hard to express.

## Contents

- [Public interfaces and internal state](#public-interfaces-and-internal-state)
- [Properties and validation](#properties-and-validation)
- [State transitions and invariants](#state-transitions-and-invariants)
- [Mutable data and defensive interfaces](#mutable-data-and-defensive-interfaces)
- [Abstraction and API design](#abstraction-and-api-design)
- [Common mistakes and practice](#common-mistakes-and-practice)

## Public interfaces and internal state

Encapsulation groups state with the operations that preserve its rules. In Python, most privacy is by convention, not access-control enforcement.

| Name style | Convention |
|---|---|
| `name` | Public interface |
| `_name` | Internal implementation detail |
| `__name` | Class name mangling to reduce accidental subclass collisions |
| `__name__` | Special protocol name when defined by Python |

`__balance` in class `Account` is typically mangled to `_Account__balance`. It is still accessible and is not encryption or a security boundary. Do not invent arbitrary double-underscore protocol names.

Use ordinary public attributes when no rule is needed. Properties allow a stable attribute-style API when validation or computation becomes necessary.

## Properties and validation

```python
from math import isfinite


class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if isinstance(value, bool) or not isinstance(value, (int, float)):
            raise TypeError("temperature must be numeric")
        if not isfinite(value) or value < -273.15:
            raise ValueError("temperature must be finite and above absolute zero")
        self._celsius = float(value)

    @property
    def fahrenheit(self):
        return self.celsius * 9 / 5 + 32


temperature = Temperature(0)
assert temperature.fahrenheit == 32
temperature.celsius = 100
assert temperature.fahrenheit == 212
```

The setter validates before changing state, so a failed assignment leaves the previous value intact. Constructor assignment uses the same validation path. The computed Fahrenheit property avoids storing two values that could disagree.

Inside a setter, assign the backing name such as `_celsius`. Writing `self.celsius = value` inside that setter would call the setter recursively.

A property without a setter is read-only through that public attribute. It does not make the entire object immutable. Properties should act like attributes; explicit methods are clearer for network calls, costly operations, or actions with important side effects.

## State transitions and invariants

An **invariant** is a rule that must hold whenever an object is externally observable in a valid state. For inventory, stock must never be negative.

```python
class Inventory:
    def __init__(self, stock=0):
        self._validate_quantity(stock, allow_zero=True)
        self._stock = stock

    @staticmethod
    def _validate_quantity(value, allow_zero=False):
        if isinstance(value, bool) or not isinstance(value, int):
            raise TypeError("quantity must be an integer")
        minimum = 0 if allow_zero else 1
        if value < minimum:
            raise ValueError(f"quantity must be at least {minimum}")

    @property
    def stock(self):
        return self._stock

    def receive(self, quantity):
        self._validate_quantity(quantity)
        self._stock += quantity

    def reserve(self, quantity):
        self._validate_quantity(quantity)
        if quantity > self._stock:
            raise ValueError("insufficient stock")
        self._stock -= quantity


inventory = Inventory(5)
inventory.reserve(2)
assert inventory.stock == 3
try:
    inventory.reserve(10)
except ValueError:
    pass
assert inventory.stock == 3
```

Operations such as `reserve` express domain intent better than a general `set_stock`. They can enforce rules involving old and new state.

Checking then updating is not automatically thread-safe or database-transaction-safe. Concurrent access requires synchronization or transactional storage with the invariant enforced at the appropriate boundary.

## Mutable data and defensive interfaces

```python
class ReadingList:
    def __init__(self, titles=()):
        self._titles = list(titles)

    @property
    def titles(self):
        return tuple(self._titles)

    def add(self, title):
        if not isinstance(title, str) or not title.strip():
            raise ValueError("title must be a nonempty string")
        self._titles.append(title)


source = ["Python"]
reading = ReadingList(source)
source.append("Outside change")
reading.add("Data science")
assert reading.titles == ("Python", "Data science")
```

Copying the incoming sequence prevents external list mutation from changing the container. Returning a tuple avoids handing out the internal list. If the elements themselves were mutable objects, callers could still mutate those objects; shallow protection is not deep immutability.

The constructor above focuses on ownership; a production version should apply the same item validation during construction as `add` if untrusted initial values are allowed. A convenient implementation initializes an empty list and calls `add` for each item.

## Abstraction and API design

Abstraction exposes useful capabilities while leaving implementation details changeable. A `Repository.get(id)` may read from memory, a file, or a database; the caller should rely on the documented result and error behavior.

Good interfaces document:

- Accepted inputs, types, units, and boundaries.
- Returned values and whether they are mutable or shared.
- Possible exceptions and whether failure changes state.
- Resource lifetime, persistence, and concurrency assumptions.
- Whether an operation is expensive, lazy, or performs I/O.

Avoid adding getters and setters mechanically for every attribute. A setter that accepts any value adds little protection. Prefer direct attributes for plain data, validated properties for attribute-like rules, and named methods for meaningful actions.

## Common mistakes and practice

| Mistake | Better design |
|---|---|
| Store both derived and source values without synchronization | Compute the derived value or centralize updates |
| Validate after mutating | Validate first, then commit the transition |
| Return an internal mutable collection unintentionally | Return a copy, immutable view, or explicit iterator |
| Assume `_name` enforces privacy | Treat it as a convention and design a clear public API |
| Put slow external work behind a property | Use an explicit action method |
| Allow different constructor and update rules | Reuse validation logic |

Practice: add a bounded capacity to inventory; implement a read-only duration derived from start/end values; test that failed updates preserve state; compare a tuple of mutable lists with a tuple of immutable strings.

Reference: [property](https://docs.python.org/3/library/functions.html#property). Next: [Inheritance and polymorphism](03-inheritance-and-polymorphism.md).
