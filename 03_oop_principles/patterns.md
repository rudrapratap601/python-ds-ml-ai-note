# 7. OOP Design Principles and Patterns

> **Learning goal:** Recognize recurring design problems and choose a small, testable solution. Patterns are vocabulary, not a requirement to add more classes.

## Contents

- [Cohesion, coupling, and SOLID](#cohesion-coupling-and-solid)
- [Strategy and dependency injection](#strategy-and-dependency-injection)
- [Factories](#factories)
- [Adapter and facade](#adapter-and-facade)
- [Decorator and proxy](#decorator-and-proxy)
- [Observer, command, and state](#observer-command-and-state)
- [Repository and unit of work](#repository-and-unit-of-work)
- [Singleton and other tradeoffs](#singleton-and-other-tradeoffs)
- [Testing and refactoring](#testing-and-refactoring)

## Cohesion, coupling, and SOLID

**Cohesion** describes how closely a component's responsibilities belong together. **Coupling** describes how much components depend on each other's details. Aim for clear responsibilities and visible, stable dependencies.

| Principle | Practical interpretation | Example |
|---|---|---|
| Single responsibility | Group things that change for the same reason | Separate parsing from chart rendering |
| Open/closed | Add variations through a stable interface when useful | Add a new exporter without editing client logic |
| Liskov substitution | Preserve the parent/interface contract | A subtype must accept the promised inputs |
| Interface segregation | Give clients the small interface they need | A reader need not implement writes |
| Dependency inversion | Depend on useful abstractions at boundaries | Inject a storage interface into a service |

These are design heuristics. Do not create an interface and factory for every two-line function. Duplication can be cheaper than an abstraction built before the real variation is understood.

## Strategy and dependency injection

Strategy makes a varying algorithm replaceable. In Python, the strategy may be a function instead of a class.

```python
def total_without_discount(amount):
    return amount


def student_discount(amount):
    return amount * 0.9


class Checkout:
    def __init__(self, pricing):
        self.pricing = pricing

    def total(self, amount):
        if amount < 0:
            raise ValueError("amount must be nonnegative")
        return self.pricing(amount)


assert Checkout(total_without_discount).total(100) == 100
assert Checkout(student_discount).total(100) == 90
```

Dependency injection means providing a collaborator from outside. It can be as simple as this constructor parameter; it does not require a framework. Keep a clear return/input contract for every strategy.

## Factories

A factory centralizes construction when callers should select a capability without knowing its concrete class.

```python
import json


class JsonExporter:
    def export(self, rows):
        return json.dumps(rows)


class TextExporter:
    def export(self, rows):
        return "\n".join(map(str, rows))


def make_exporter(format_name):
    constructors = {"json": JsonExporter, "text": TextExporter}
    try:
        constructor = constructors[format_name]
    except KeyError as exc:
        raise ValueError(f"unsupported format: {format_name}") from exc
    return constructor()


assert make_exporter("json").export([1, 2]) == "[1, 2]"
```

A simple factory is a function like this. Factory Method delegates a construction decision to a method that subclasses can override. Abstract Factory provides related families of objects. Use the more elaborate forms only when multiple coordinated variations justify them.

## Adapter and facade

An adapter translates an existing interface into the one your application expects. A facade provides a simpler entry point to a larger subsystem.

```python
class LegacyFormatter:
    def convert_text(self, text, uppercase):
        return text.upper() if uppercase else text


class FormatterAdapter:
    def __init__(self, legacy):
        self.legacy = legacy

    def format(self, text):
        return self.legacy.convert_text(text, uppercase=True)


assert FormatterAdapter(LegacyFormatter()).format("notes") == "NOTES"
```

At a data-science boundary, an adapter might convert external model output into your application's prediction record. A facade might expose `generate_report(...)` over loading, validation, aggregation, and export. Keep failure details and configuration available where they matter; simplicity should not hide incorrect assumptions.

## Decorator and proxy

The object-oriented Decorator pattern wraps an object, preserves its interface, and adds behavior. It is related to but distinct from Python's `@decorator` syntax.

```python
class CountingExporter:
    def __init__(self, wrapped):
        self.wrapped = wrapped
        self.calls = 0

    def export(self, rows):
        self.calls += 1
        return self.wrapped.export(rows)


exporter = CountingExporter(JsonExporter())
assert exporter.export([3]) == "[3]"
assert exporter.calls == 1
```

A proxy controls access to another object, for example through lazy initialization or a remote boundary. Both patterns introduce another layer; preserve exceptions, return values, resource ownership, and concurrency behavior.

## Observer, command, and state

| Pattern | Problem solved | Python implementation idea | Watch for |
|---|---|---|---|
| Observer | Notify interested listeners of changes | List of callbacks | Unsubscribe, listener lifetime, error policy |
| Command | Represent an action as data/behavior | Callable or command object | Undo is not always possible |
| State | Behavior changes with lifecycle phase | Enum plus transitions, or state objects | Reject invalid transitions |
| Template Method | Shared algorithm skeleton with variation hooks | Base method calling overridable steps | Fragile inheritance assumptions |
| Builder | Construct a complex object in stages | Explicit builder or validated configuration | Partially valid intermediate state |

Start with ordinary functions and enums. Introduce classes when the action or state needs a meaningful lifecycle and several operations.

An observer notification should define whether one failing listener stops the others. A command for an external write should define idempotency. A state machine should document valid transitions rather than allowing arbitrary status assignment.

## Repository and unit of work

A repository presents domain-oriented access to stored entities, hiding query mechanics behind a useful interface. It should not merely mirror every database method.

A unit of work coordinates a group of changes that should commit or roll back together. A context manager can express its lifetime, but transaction semantics come from the actual persistence layer; a `with` statement alone does not make arbitrary operations atomic.

For ML systems, separate experiment metadata, model artifacts, and training execution. They have different storage and lifecycle needs. A model registry interface is not the same thing as storing large model binaries directly in a Python object.

## Singleton and other tradeoffs

A Singleton restricts construction to one shared instance. In Python, module-level objects already provide a form of shared process-local state. Both approaches can hide dependencies, complicate tests, and require synchronization.

Prefer passing a shared service explicitly unless global uniqueness is a real requirement. One singleton per process does not imply one instance across multiple worker processes or machines.

Avoid service locators that make every dependency invisible, inheritance added only to reuse a few lines, and factories whose only job is to return one obvious concrete class. Favor composition when behavior changes independently.

## Testing and refactoring

Test observable contracts, not private call sequences. A fake collaborator with a small in-memory implementation often gives clearer tests than a large mock configuration.

Refactor in small steps: identify a responsibility, describe the current behavior with tests, extract a function or collaborator, then compare behavior. Apply a pattern because it resolves a concrete problem in the code.

Practice: build interchangeable CSV/JSON exporters; adapt a legacy parser; inject a clock into a timestamping service; model an experiment lifecycle; explain why a singleton would complicate two independent tests.

Next: [OOP mini-project and testing](08-oop-project-and-testing.md).
