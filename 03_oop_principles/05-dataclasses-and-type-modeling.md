# 5. Dataclasses, Enums, and Typed Models

> **Learning goal:** Choose a representation that communicates the data contract without unnecessary boilerplate.

## Contents

- [Choosing a model](#choosing-a-model)
- [Dataclass basics](#dataclass-basics)
- [Validation and computed fields](#validation-and-computed-fields)
- [Equality, ordering, frozen objects, and slots](#equality-ordering-frozen-objects-and-slots)
- [Class variables and initialization inputs](#class-variables-and-initialization-inputs)
- [Enums and typed dictionaries](#enums-and-typed-dictionaries)
- [Serialization and boundaries](#serialization-and-boundaries)
- [Practice](#practice)

## Choosing a model

| Representation | Good for | Tradeoff |
|---|---|---|
| `dict` | Flexible records and external payloads | Keys and types need validation |
| `TypedDict` | Static structure for dictionaries | Still a plain dictionary at runtime |
| `NamedTuple` | Small immutable tuple-compatible records | Positional coupling; nested values may remain mutable |
| `dataclass` | Named fields with generated methods | Runtime validation is your responsibility |
| Ordinary class | Rich behavior and lifecycle | More explicit implementation |
| `Enum` | Closed set of named alternatives | External serialization needs a chosen representation |

Do not use a class merely to rename a dictionary if its contract remains undefined. Choose whether the object represents a value, a mutable entity, configuration, or a service.

## Dataclass basics

```python
from dataclasses import dataclass, field


@dataclass
class Experiment:
    name: str
    seed: int = 42
    metrics: dict[str, float] = field(default_factory=dict)


first = Experiment("baseline")
second = Experiment("candidate")
first.metrics["accuracy"] = 0.8
assert second.metrics == {}
assert first.seed == 42
```

Dataclasses generate methods such as `__init__`, `__repr__`, and equality from annotated fields. Use `default_factory` for a fresh mutable default per instance. A mutable value created once at class definition would otherwise be shared or rejected by dataclass checks.

Fields without defaults generally precede fields with defaults across generated initialization, including inherited fields. Keyword-only fields can make larger constructors clearer. See the [dataclasses reference](https://docs.python.org/3/library/dataclasses.html).

## Validation and computed fields

```python
from math import isfinite


@dataclass
class TrainingConfig:
    learning_rate: float
    epochs: int
    description: str = field(init=False)

    def __post_init__(self):
        if not isfinite(self.learning_rate) or self.learning_rate <= 0:
            raise ValueError("learning rate must be finite and positive")
        if isinstance(self.epochs, bool) or not isinstance(self.epochs, int):
            raise TypeError("epochs must be an integer")
        if self.epochs < 1:
            raise ValueError("epochs must be positive")
        self.description = f"{self.epochs} epochs at {self.learning_rate}"


config = TrainingConfig(0.01, 20)
assert config.description == "20 epochs at 0.01"
```

`__post_init__` is called by the generated initializer. A handwritten `__init__` must arrange its own validation. Type annotations do not reject a wrong runtime type automatically; boundary validation may need stricter checks than this teaching example.

Validation during construction does not prevent later mutation. Use properties, frozen value objects, or explicit state transitions when the invariant must hold throughout the lifetime.

## Equality, ordering, frozen objects, and slots

```python
from dataclasses import replace


@dataclass(frozen=True, slots=True)
class ModelKey:
    name: str
    version: int


key = ModelKey("baseline", 1)
new_key = replace(key, version=2)
assert key.version == 1
assert new_key == ModelKey("baseline", 2)
assert len({key, ModelKey("baseline", 1)}) == 1
```

| Option | Effect | Caveat |
|---|---|---|
| `eq=True` | Compare fields for equal instances of the same class | Exclude non-semantic fields with `compare=False` |
| `order=True` | Generate ordering by field sequence | Field order must match domain ordering |
| `frozen=True` | Block ordinary field rebinding | Nested mutable objects can still mutate |
| `slots=True` | Generate slots for fields | Changes storage and some inheritance behavior |
| `kw_only=True` | Make fields keyword-only in initialization | Callers must use named arguments |
| `repr=False` on a field | Exclude it from generated representation | Does not secure the value elsewhere |

Frozen dataclasses can generate a hash when equality settings allow it, but all fields used in that hash must be hashable. A frozen dataclass containing a list is not automatically a safe dictionary key. Avoid `unsafe_hash=True` unless you fully understand the mutation/equality contract.

Slots can reduce per-instance storage and prevent unintended new attributes, but do not make objects private or automatically immutable. A base class with `__dict__` can retain dynamic storage in subclasses. Weak references require appropriate support. Measure memory before using slots solely as an optimization.

## Class variables and initialization inputs

```python
from dataclasses import InitVar
from typing import ClassVar


@dataclass
class NormalizedName:
    category: ClassVar[str] = "label"
    raw: InitVar[str]
    value: str = field(init=False)

    def __post_init__(self, raw):
        self.value = raw.strip().lower()


name = NormalizedName("  STUDY  ")
assert name.value == "study"
assert NormalizedName.category == "label"
```

`ClassVar` marks a class-level value rather than an instance field. `InitVar` is an initialization input passed to `__post_init__`, not an ordinary stored field. If inheriting from a non-dataclass base, its initializer is not automatically called by a generated dataclass initializer; arrange necessary initialization deliberately.

## Enums and typed dictionaries

```python
from enum import Enum
from typing import TypedDict, NotRequired, NamedTuple


class RunStatus(Enum):
    PENDING = "pending"
    COMPLETE = "complete"
    FAILED = "failed"


class RunPayload(TypedDict):
    name: str
    status: str
    message: NotRequired[str]


class Point(NamedTuple):
    x: float
    y: float


payload: RunPayload = {"name": "baseline", "status": RunStatus.COMPLETE.value}
assert RunStatus(payload["status"]) is RunStatus.COMPLETE
assert Point(2, 3).x == 2
```

An Enum member has a name and a value; serialize whichever the external contract specifies. `TypedDict` helps static checking but does not turn dictionary construction into runtime schema validation. `NotRequired` concerns whether a key may be absent, which differs from allowing `None` as its value.

## Serialization and boundaries

`dataclasses.asdict` recursively converts dataclasses and copies nested structures. It can be more expensive than extracting selected fields. Do not blindly expose every internal field in a public API or log.

JSON conversion still needs policies for enums, datetimes, decimals, and custom values. Deserializing a dictionary using `Class(**payload)` is not complete validation of arbitrary nested input. Validate schema, missing/extra fields, value ranges, and version compatibility at the boundary.

External schema libraries can help larger applications, but their validation and coercion rules must be understood and tested. Keep domain invariants explicit regardless of the tool.

## Practice

Build a frozen experiment identifier, a validated training configuration, and a mutable result record. Decide which fields participate in equality and serialization. Test that mutable defaults are independent and explain why a frozen list-containing object is not deeply immutable.

Next: [Descriptors and class customization](06-descriptors-and-class-customization.md).
