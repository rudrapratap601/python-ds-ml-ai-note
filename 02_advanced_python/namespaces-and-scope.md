# Namespaces, Scope, and Closures

> **Purpose:** Predict which object a name refers to, avoid scope bugs, and understand the foundations of decorators and modules.

## Contents

- [Names refer to objects](#names-refer-to-objects)
- [Namespaces and scope](#namespaces-and-scope)
- [LEGB lookup](#legb-lookup)
- [Assignment, global, and nonlocal](#assignment-global-and-nonlocal)
- [Closures and late binding](#closures-and-late-binding)
- [Classes and comprehensions](#classes-and-comprehensions)
- [Modules, packages, and imports](#modules-packages-and-imports)
- [Inspection and practical rules](#inspection-and-practical-rules)

## Names refer to objects

Assignment binds a name to an object. It does not automatically copy the object or give each name independent storage.

```python
first = [1, 2]
second = first
second.append(3)
assert first == [1, 2, 3]
assert first is second

second = [9]  # Rebind just this name.
assert first == [1, 2, 3]
assert second == [9]
```

**Mutation** changes an existing object; **rebinding** changes which object a name refers to. `del name` removes a binding, not necessarily the object: other references may still exist.

`is` checks identity; `==` checks equality according to the objects' comparison behavior. Use `is None` for the singleton `None`, not `is` to compare arbitrary strings or numbers.

## Namespaces and scope

A **namespace** associates names with objects. A **scope** is a region of code where a namespace is directly accessible through name lookup.

| Namespace | Created around | Typical contents |
|---|---|---|
| Built-ins | Interpreter setup | `len`, `str`, `Exception` |
| Module globals | Module execution/import | Imported names and top-level definitions |
| Function locals | Function invocation | Parameters and local bindings |
| Enclosing function state | Nested function creation/execution | Captured nonlocal bindings |
| Class namespace | Class body execution | Methods and class attributes |

Separate calls normally get separate local state. A closure may keep selected enclosing bindings alive after the outer function has returned.

## LEGB lookup

For ordinary names inside ordinary nested functions, lookup searches **Local → Enclosing functions → Global module → Built-ins**.

```python
label = "module"


def outer():
    label = "enclosing"

    def inner():
        return label

    return inner()


assert outer() == "enclosing"
assert label == "module"
```

This is lexical scope: the function's definition context matters, not the caller's local names. Attribute lookup such as `object.name` is a different mechanism from resolving a bare `name`.

Do not name a variable `list`, `sum`, or `open` when you still need the built-in. Shadowing is legal, but it makes code less clear and can produce confusing errors.

The [Python classes tutorial](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces) explains the ordinary namespace model. Specialized annotation scopes in newer Python versions have additional rules; LEGB is a useful starting point, not a complete specification of every execution context.

## Assignment, global, and nonlocal

Assigning a name anywhere in a function normally makes it local throughout that function body, unless declared otherwise.

```python
count = 10


def broken_increment():
    count += 1


try:
    broken_increment()
except UnboundLocalError:
    pass
else:
    raise AssertionError("the local count has no initial binding")
```

`count += 1` must first read the local `count`, which has not been assigned. It does not automatically read and update the global binding.

### Global

```python
total = 0


def add_to_total(amount):
    global total
    total += amount


add_to_total(3)
assert total == 3
```

`global` refers to the module namespace in which the function was defined. It does not mean one universal namespace shared by every module. Shared mutable globals make dependencies and tests harder to manage; explicit parameters and returned values are usually simpler.

### Nonlocal

```python
def make_counter(start=0):
    count = start

    def increment():
        nonlocal count
        count += 1
        return count

    return increment


left = make_counter()
right = make_counter(10)
assert [left(), left(), right()] == [1, 2, 11]
```

`nonlocal` rebinds an existing name in the nearest enclosing function scope containing it. It cannot create an arbitrary enclosing binding or target a module global. A missing enclosing binding is a syntax error.

Mutating an already reachable list does not require `nonlocal`; rebinding the list name does. This distinction is about bindings, not whether an operation feels like an update.

## Closures and late binding

A closure is a function retaining access to bindings from its defining environment.

```python
def multiplier(factor):
    def multiply(value):
        return value * factor
    return multiply


triple = multiplier(3)
assert triple(7) == 21
```

Closures capture bindings, not necessarily a frozen snapshot of each value. Loop variables are a common surprise:

```python
functions = [lambda: number for number in range(3)]
assert [function() for function in functions] == [2, 2, 2]

snapshots = [lambda number=number: number for number in range(3)]
assert [function() for function in snapshots] == [0, 1, 2]
```

The default argument is evaluated when each function is created. It stores the reference then; it does not deep-copy mutable objects. A factory function like `multiplier` is another way to create distinct bindings.

Default arguments are evaluated once per function definition, which also explains why a mutable default list can leak state across calls. Use `None` and initialize a new list in the function when each call should own one.

## Classes and comprehensions

A method's bare-name lookup does not automatically search its class's namespace. Access instance or class state explicitly:

```python
class Settings:
    timeout = 5

    def describe(self):
        return self.timeout


assert Settings().describe() == 5
```

`self.timeout` uses attribute lookup and can find an instance override or class attribute. `timeout` alone inside the method is a different name lookup.

In Python 3, comprehension iteration variables have their own scope and do not normally leak into the containing scope. Class-body comprehensions have additional scope subtleties; avoid assuming class-local names are enclosing function variables.

```python
number = 99
squares = [number * number for number in range(3)]
assert number == 99
assert squares == [0, 1, 4]
```

An `except Error as exc` target is cleared after the handler to avoid retaining traceback reference cycles. Save only the information you need outside the handler.

## Modules, packages, and imports

```text
project/
├── app.py
└── processing/
    ├── __init__.py
    ├── loaders.py
    └── transforms.py
```

- `import processing.transforms` binds a module access path.
- `from processing.transforms import clean` binds the current exported object to a local name.
- `import numpy as np` binds an alias; it does not rename the library globally.
- `from module import *` obscures name origins; avoid it in maintainable code.
- A regular package generally has `__init__.py`; namespace packages can span directories without one.
- Relative imports such as `from .loaders import read` depend on package context. Run package entry points with `python -m package.module` when appropriate.

Imports normally reuse modules cached in `sys.modules`. Importing a module executes its top-level code the first time it is loaded in that interpreter context. Keep expensive work and external side effects out of import-time code.

If module `a` rebinds `a.value`, a previously executed `from a import value` does not automatically rebind the importing module's name. Refer to `a.value` when you need subsequent changes through the module object.

Circular imports can expose partially initialized modules. Move shared contracts to a lower-level module or reorganize dependencies before adding local imports as a workaround.

The entry-point guard is:

```python
def main():
    return "ready"


if __name__ == "__main__":
    print(main())
```

It controls whether the guarded code runs as the entry point. It does not stop other top-level code from running on import.

## Inspection and practical rules

`globals()` exposes the current module dictionary. `locals()` exposes local bindings with semantics that depend on the execution scope and Python version; do not use changes to its returned mapping as a portable way to assign function locals. `vars(obj)` commonly exposes an object's `__dict__`, if it has one. `dir(obj)` lists discoverable names but is not a guarantee that all are ordinary stored attributes.

Prefer explicit data flow, small scopes, clear import aliases, and functions whose dependencies are visible. Use a closure for small private state; use an object when state has several related operations or a richer lifecycle.

Practice: predict the output of a nested function before running it; fix a late-binding callback loop; replace a global accumulator with an explicit return value; explain why rebinding and mutation behave differently.

References: [execution model](https://docs.python.org/3/reference/executionmodel.html), [import system](https://docs.python.org/3/reference/import.html). Next: [decorators](decorators.md).
