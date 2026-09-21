# Generators and Iterators in Python

> **Purpose:** Build lazy, composable data pipelines and understand exactly when work happens, what state is retained, and how iteration ends.

## Contents

- [Iterable, iterator, and generator](#iterable-iterator-and-generator)
- [Writing generators](#writing-generators)
- [Generator expressions](#generator-expressions)
- [Laziness and memory](#laziness-and-memory)
- [Yield from and return values](#yield-from-and-return-values)
- [Send, throw, and close](#send-throw-and-close)
- [Streaming pipelines and itertools](#streaming-pipelines-and-itertools)
- [Custom iterators and async generators](#custom-iterators-and-async-generators)
- [Testing and revision](#testing-and-revision)

## Iterable, iterator, and generator

| Concept | Meaning | Examples |
|---|---|---|
| Iterable | Can produce an iterator through `iter()` | List, tuple, range, file, generator |
| Iterator | Produces successive values with `next()` | `iter(a_list)`, an open text file |
| Generator function | Function containing `yield` | A function that yields records |
| Generator object | Iterator created by calling a generator function | `records()` |

Every generator object is an iterator, and every iterator is iterable. Not every iterable is itself an iterator. A list can create fresh iterators; a particular iterator has a current position and is usually consumed once.

```python
values = [10, 20]
iterator = iter(values)
assert iter(iterator) is iterator
assert next(iterator) == 10
assert next(iterator) == 20
assert next(iterator, "finished") == "finished"
assert list(values) == [10, 20]
```

`next(iterator)` without a default raises `StopIteration` at exhaustion. A `for` loop repeatedly requests values and handles that normal stopping signal.

## Writing generators

```python
def countdown(start):
    if start < 0:
        raise ValueError("start must be nonnegative")
    while start:
        yield start
        start -= 1


generator = countdown(3)
assert next(generator) == 3
assert list(generator) == [2, 1]
assert list(generator) == []
```

Calling `countdown(3)` creates the generator; it does not run the body to its first yield. Each `next()` resumes from where execution paused. Local state survives between resumptions. `return` or reaching the end terminates the generator.

Validation inside a generator body is also deferred. If invalid arguments must fail at call time, validate in a regular outer function and return an inner generator.

```python
def checked_countdown(start):
    if not isinstance(start, int) or isinstance(start, bool):
        raise TypeError("start must be an integer")
    if start < 0:
        raise ValueError("start must be nonnegative")

    def generate():
        yield from range(start, 0, -1)

    return generate()


assert list(checked_countdown(3)) == [3, 2, 1]
```

## Generator expressions

```python
squares = (number * number for number in range(5))
assert sum(squares) == 30
assert sum(squares) == 0  # It is exhausted.

total = sum(number * number for number in range(5))
assert total == 30
```

A list comprehension builds a list immediately; a generator expression computes values on demand. The leftmost iterable expression is evaluated immediately, while subsequent work happens during iteration. Names used later can observe later rebinding, so be careful with closures and mutable inputs.

Generators do not generally support indexing, `len()`, or rewinding. Materialize into a list when repeated random access is part of the requirement, or create a fresh generator from a factory for each pass.

## Laziness and memory

A pipeline can process a huge source with small working memory if each stage consumes and emits incrementally. Laziness does not guarantee constant memory:

- A generator may retain a large input or large local objects in its suspended frame.
- `list(generator)`, `sorted(generator)`, and many aggregations materialize data.
- `itertools.tee` buffers values when its consumers advance at different rates.
- Recursive generators retain nested execution state.
- An individual yielded item can itself be huge.

Short-circuiting consumers such as `any()` stop early. That is useful for performance, but unfinished resources still need explicit lifetime management.

```python
seen = []


def numbers():
    for value in range(10):
        seen.append(value)
        yield value


assert any(value == 2 for value in numbers())
assert seen == [0, 1, 2]
```

## Yield from and return values

`yield from iterable` forwards values from another iterable. With a generator subroutine it also delegates `send`, `throw`, and close-related protocol behavior.

```python
def combined():
    yield "start"
    yield from [1, 2]
    yield "end"


assert list(combined()) == ["start", 1, 2, "end"]
```

A generator's `return value` becomes the value attached to its terminating `StopIteration`. Ordinary `for` loops do not expose that return value; `yield from` can receive it.

```python
def child():
    yield 1
    return "complete"


def parent():
    result = yield from child()
    yield result


assert list(parent()) == [1, "complete"]
```

Do not explicitly raise `StopIteration` inside a generator to stop it. Use `return`; an escaping `StopIteration` raised within generator code is converted into `RuntimeError` in modern Python. Protocol details are specified in the [generator expression and yield reference](https://docs.python.org/3/reference/expressions.html#yield-expressions).

## Send, throw, and close

`yield` is an expression: it can receive a value when the generator resumes through `send(value)`.

```python
def running_total():
    total = 0
    while True:
        increment = yield total
        if increment is None:
            return
        total += increment


accumulator = running_total()
assert next(accumulator) == 0  # Prime to the first yield.
assert accumulator.send(4) == 4
assert accumulator.send(6) == 10
accumulator.close()
```

Before the first yield, only `send(None)` or `next()` is valid. `next(g)` resumes like `g.send(None)`. `throw(exception)` raises an exception at the suspended point; the generator may handle it or terminate.

`close()` injects `GeneratorExit`, allowing `finally` blocks to release resources. A generator must not yield another value while responding to closure. Do not depend on garbage collection timing for important cleanup.

```python
from contextlib import closing

events = []


def resource_values():
    try:
        yield 1
        yield 2
    finally:
        events.append("closed")


with closing(resource_values()) as values:
    assert next(values) == 1
assert events == ["closed"]
```

These generator-based coroutine techniques are useful to understand, but `async def` and `await` are the normal foundation for asynchronous I/O APIs.

## Streaming pipelines and itertools

```python
from io import StringIO


def meaningful_lines(stream):
    for number, line in enumerate(stream, start=1):
        text = line.strip()
        if text and not text.startswith("#"):
            yield number, text


def parse_numbers(lines):
    for number, text in lines:
        try:
            yield int(text)
        except ValueError as exc:
            raise ValueError(f"invalid integer on line {number}") from exc


with StringIO("# values\n3\n\n5\n") as stream:
    assert sum(parse_numbers(meaningful_lines(stream))) == 8
```

The consumer must run while the source remains open. Returning a generator over a file that was already closed by an enclosing `with` is a common bug. Either let the generator own the file context, or make the caller own the open stream as above.

| Tool | Use | Caveat |
|---|---|---|
| `itertools.islice` | Consume a bounded segment | It advances the source |
| `chain` / `chain.from_iterable` | Concatenate streams | Does not recursively flatten all nesting |
| `count`, `cycle`, `repeat` | Repeated or infinite streams | Bound consumption; `cycle` stores encountered values |
| `accumulate` | Running reductions | First result is usually the first input |
| `pairwise` | Adjacent pairs | Available in Python 3.10+ |
| `groupby` | Consecutive groups | Not SQL-style global grouping; group iterators share input |
| `tee` | Multiple logical consumers | Lagging consumers can force large buffers |
| `product`, `permutations`, `combinations` | Combinatorial outputs | Output size can grow rapidly; inputs may be pooled |

```python
from itertools import islice, count, pairwise, groupby

assert list(islice(count(10, 2), 3)) == [10, 12, 14]
assert list(pairwise([1, 4, 9])) == [(1, 4), (4, 9)]
groups = [(key, list(items)) for key, items in groupby("AAABBBA")]
assert groups == [("A", ["A", "A", "A"]), ("B", ["B", "B", "B"]), ("A", ["A"])]
```

For batches, recent Python versions offer `itertools.batched` (Python 3.12+). A portable teaching implementation is:

```python
def batches(iterable, size):
    if size < 1:
        raise ValueError("size must be positive")
    iterator = iter(iterable)
    while batch := tuple(islice(iterator, size)):
        yield batch


assert list(batches(range(5), 2)) == [(0, 1), (2, 3), (4,)]
```

## Custom iterators and async generators

A custom iterator implements `__iter__` returning itself and `__next__` producing a value or raising `StopIteration`. A container usually returns a fresh iterator instead.

```python
class Countdown:
    def __init__(self, start):
        self.remaining = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.remaining <= 0:
            raise StopIteration
        value = self.remaining
        self.remaining -= 1
        return value


assert list(Countdown(3)) == [3, 2, 1]
```

Prefer a generator when it expresses the state machine more clearly. Use a class when the iterator needs additional methods or explicit inspectable state.

An async generator contains `yield` inside `async def`. Consume it with `async for`; it can await asynchronous work between values.

```python
import asyncio


async def async_values():
    for value in range(3):
        await asyncio.sleep(0)
        yield value


async def collect_values():
    return [value async for value in async_values()]


assert asyncio.run(collect_values()) == [0, 1, 2]
```

An async generator is not consumed with ordinary `list()`. Use `aclose()` or `contextlib.aclosing` for deterministic async cleanup. Async execution does not automatically parallelize CPU-bound computations.

## Testing and revision

Test empty input, one item, normal exhaustion, partial consumption, repeated attempts to consume, deferred failures, and cleanup on early exit. Checking only that calling the function returns a generator does not test its body.

Practice: stream filtered CSV records; implement a moving window with bounded storage; compare `groupby` on sorted and unsorted input; show when a generator retains a large object; implement a restartable iterable that creates a fresh generator on each `iter()` call.

References: [itertools](https://docs.python.org/3/library/itertools.html), [generator types](https://docs.python.org/3/library/stdtypes.html#generator-types). Related: [recursion](recursion.md), [file handling](file-handling.md).
