# Python Decorators: Basic to Advanced

> **Purpose:** Add reusable behavior to functions and classes while preserving their contracts, metadata, and execution semantics.

## Contents

- [Functions are objects](#functions-are-objects)
- [Decorator syntax and execution](#decorator-syntax-and-execution)
- [A reusable wrapper](#a-reusable-wrapper)
- [Parameterized decorators](#parameterized-decorators)
- [Stacking and ordering](#stacking-and-ordering)
- [Built-in and standard-library decorators](#built-in-and-standard-library-decorators)
- [Classes and registration](#classes-and-registration)
- [Async and generator functions](#async-and-generator-functions)
- [Typing, methods, and state](#typing-methods-and-state)
- [Testing and pitfalls](#testing-and-pitfalls)

## Functions are objects

Functions can be assigned to names, passed as arguments, and returned from other functions. A decorator is a callable that accepts a target and returns the object to bind in its place. Many decorators return wrappers, but wrapping is not mandatory.

```python
def greet(name):
    return f"Hello, {name}"


alias = greet
assert alias("Asha") == "Hello, Asha"
```

Read [namespaces and closures](namespaces-and-scope.md) first if returning an inner function is unfamiliar.

## Decorator syntax and execution

```text
@decorate
def work(...):
    ...

is approximately:

def work(...):
    ...
work = decorate(work)
```

Decoration happens when execution reaches the definition, often during import. The wrapper body runs when the decorated function is called. Do not accidentally execute the target during decoration unless that is specifically the intended behavior.

## A reusable wrapper

```python
from functools import wraps
from time import perf_counter


def timed(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        started = perf_counter()
        try:
            return function(*args, **kwargs)
        finally:
            elapsed = perf_counter() - started
            print(f"{function.__name__}: {elapsed:.6f}s")
    return wrapper


@timed
def total(values):
    """Sum the supplied values."""
    return sum(values)


assert total([2, 3]) == 5
assert total.__name__ == "total"
assert total.__doc__ == "Sum the supplied values."
assert total.__wrapped__([2, 3]) == 5
```

The wrapper accepts positional and keyword arguments and forwards the return value. `finally` records elapsed time even when the target raises, without suppressing that error. A real application would usually use configurable logging rather than unconditional printing.

`functools.wraps` preserves useful metadata and establishes `__wrapped__`; inspection tools can follow that reference. It does not make arbitrary changes to behavior safe or automatically enforce the original signature. See the [functools reference](https://docs.python.org/3/library/functools.html#functools.wraps).

## Parameterized decorators

A decorator factory adds an outer layer for configuration:

```python
def require_minimum(minimum):
    def decorate(function):
        @wraps(function)
        def wrapper(value, *args, **kwargs):
            if value < minimum:
                raise ValueError(f"value must be at least {minimum}")
            return function(value, *args, **kwargs)
        return wrapper
    return decorate


@require_minimum(0)
def square(value):
    return value * value


assert square(4) == 16
```

Three stages are involved: configure with `require_minimum(0)`, decorate `square`, then call the wrapper. This example deliberately assumes the first argument is the numeric value; it is not a universal validator for arbitrary signatures.

For optional decorator arguments, accepting `function=None` and returning `functools.partial(...)` is a common pattern, but support both forms only when it meaningfully improves the API.

## Stacking and ordering

```text
@outer
@inner
def function(): ...

function = outer(inner(function))
```

Decorators apply bottom-up. Calls enter the outermost wrapper first; return processing unwinds in reverse.

```python
events = []


def tag(name):
    def decorate(function):
        @wraps(function)
        def wrapper():
            events.append(f"enter {name}")
            result = function()
            events.append(f"exit {name}")
            return result
        return wrapper
    return decorate


@tag("outer")
@tag("inner")
def operation():
    events.append("body")
    return 1


assert operation() == 1
assert events == ["enter outer", "enter inner", "body", "exit inner", "exit outer"]
```

Ordering affects semantics: timing outside a cache measures cache hits too; timing inside measures only cache misses. Authorization outside a shared cache ensures access is checked on each call. A cache key must still capture all inputs affecting the returned data.

## Built-in and standard-library decorators

| Decorator | Purpose | Important detail |
|---|---|---|
| `@property` | Attribute-style managed access | A getter should avoid surprising expensive side effects |
| `@classmethod` | Receive `cls` | Useful for alternate constructors |
| `@staticmethod` | No automatic `self` or `cls` | A module function may be simpler |
| `@dataclass` | Generate data-model methods | Annotations do not enforce runtime types |
| `@lru_cache(maxsize=...)` | Memoize recent calls | Hashable arguments; retains references |
| `@cache` | Unbounded memoization | Plan cache lifetime and growth |
| `@cached_property` | Compute an instance attribute lazily | Requires suitable instance storage; invalidate deliberately |
| `@singledispatch` | Dispatch by the first argument's type | Not arbitrary multi-argument overload resolution |
| `@contextmanager` | Define a context manager using one yield | Use `try/finally` for cleanup |
| `@abstractmethod` | Declare an abstract member | Combine with `ABC`; place innermost under `classmethod` |

### Cache example

```python
from functools import lru_cache


@lru_cache(maxsize=128)
def cube(number):
    return number ** 3


assert cube(4) == cube(4) == 64
assert cube.cache_info().hits >= 1
cube.cache_clear()
```

Avoid caching generators, coroutine objects, or functions intended to perform a side effect on every call. A cached mutable result may be shared between callers. Caching instance methods usually includes `self` in the key and can keep instances alive.

### Single dispatch

```python
from functools import singledispatch


@singledispatch
def describe(value):
    return f"object: {value}"


@describe.register(int)
def _(value):
    return f"integer: {value}"


@describe.register(list)
def _(value):
    return f"list with {len(value)} items"


assert describe(3) == "integer: 3"
assert describe([1, 2]) == "list with 2 items"
```

Use `singledispatchmethod` for appropriate method dispatch. Ordinary Python does not overload functions merely because multiple definitions have different parameter annotations; a later definition rebinds the name.

## Classes and registration

A class decorator receives a class after its creation and returns the class or a replacement. It can register classes, attach metadata, or transform their interface.

```python
registry = {}


def register(name):
    def decorate(cls):
        if name in registry:
            raise ValueError(f"duplicate registration: {name}")
        registry[name] = cls
        return cls
    return decorate


@register("upper")
class Uppercase:
    def transform(self, text):
        return text.upper()


assert registry["upper"]().transform("notes") == "NOTES"
```

Registration is an import-time side effect. Ensure required modules are actually imported, define duplicate behavior, and keep tests from leaking registry state.

A callable object with `__call__` can also implement a decorator or wrapper. Function wrappers are usually easier for method binding; a callable-instance wrapper may need the descriptor protocol (`__get__`) to behave correctly when stored on a class.

## Async and generator functions

Calling an `async def` function creates a coroutine; its work runs when awaited. A synchronous timing wrapper around the call would mostly time creation, not execution.

```python
import asyncio


def async_passthrough(function):
    @wraps(function)
    async def wrapper(*args, **kwargs):
        return await function(*args, **kwargs)
    return wrapper


@async_passthrough
async def async_double(value):
    return value * 2


assert asyncio.run(async_double(3)) == 6
```

In an environment already running an event loop, use `await async_double(3)` instead of nesting `asyncio.run()`.

Calling a generator function returns an iterator without consuming it. A wrapper that wants to observe yielded work must participate in iteration. `yield from` delegates ordinary generator iteration and protocol operations; an async generator needs `async for` and async-aware handling. Do not turn lazy results into lists accidentally.

```python
def relay(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        yield from function(*args, **kwargs)
    return wrapper


@relay
def small_numbers():
    yield 1
    yield 2


assert list(small_numbers()) == [1, 2]
```

## Typing, methods, and state

`ParamSpec` describes forwarded parameters for static type checkers:

```python
from collections.abc import Callable
from typing import ParamSpec, TypeVar

P = ParamSpec("P")
R = TypeVar("R")


def passthrough(function: Callable[P, R]) -> Callable[P, R]:
    @wraps(function)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        return function(*args, **kwargs)
    return wrapper


@passthrough
def add(left: int, right: int) -> int:
    return left + right


assert add(2, 4) == 6
```

Annotations support tooling; they do not perform runtime validation by themselves. Wrappers that add or remove arguments need a different type contract, potentially `Concatenate`.

A normal function wrapper on a class receives `self` through method binding. If a decorator closure contains mutable state, that state may be shared across all instances using the decorated method. Use per-instance storage when the behavior should be instance-specific. Locks or other coordination may be necessary for shared mutable state under concurrency.

## Testing and pitfalls

Test return values, exceptions, positional and keyword forwarding, metadata, stacked ordering, method binding, and repeated calls. For asynchronous or lazy functions, test actual execution or consumption, not just object creation.

| Pitfall | Consequence |
|---|---|
| Forgetting `return wrapper` | Decorated name becomes `None` |
| Forgetting `return function(...)` | Successful results are discarded |
| Omitting `wraps` | Names, docs, and introspection become misleading |
| Catching every exception | Wrapper hides bugs or cancellation |
| Implicitly caching external state | Stale or cross-user results |
| Repeating operations in a retry wrapper | Duplicate side effects without idempotency |
| Generic validation without binding the signature | Positional and keyword calls behave differently |

Practice: build a call counter with a reset method; compare timing inside and outside `lru_cache`; create a registration decorator with duplicate detection; write an async timing wrapper that preserves exceptions.

**Remember:** A useful decorator adds one clear behavior while preserving the target's documented contract.
