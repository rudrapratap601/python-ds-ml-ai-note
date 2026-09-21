# Exception Handling in Python

> **Purpose:** Understand error propagation, recovery, cleanup, custom exceptions, and useful diagnostics. Core examples require Python 3.11+ where exception groups or notes are used.

## Contents

- [Errors and the exception hierarchy](#errors-and-the-exception-hierarchy)
- [Try, except, else, and finally](#try-except-else-and-finally)
- [Raising and chaining](#raising-and-chaining)
- [Designing custom exceptions](#designing-custom-exceptions)
- [Cleanup and context managers](#cleanup-and-context-managers)
- [Exception groups](#exception-groups)
- [Logging, retries, and API boundaries](#logging-retries-and-api-boundaries)
- [Testing and common mistakes](#testing-and-common-mistakes)

## Errors and the exception hierarchy

A syntax error prevents a source unit from being parsed. An exception during execution interrupts the normal flow and searches outward through active calls for a matching handler. If none handles it, the program or task reports a traceback.

The traceback shows where the exception propagated; the exception type and message describe the failure. Read both the final message and the relevant frames.

```text
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── ValueError
    ├── TypeError
    ├── LookupError
    │   ├── KeyError
    │   └── IndexError
    ├── OSError
    │   ├── FileNotFoundError
    │   └── PermissionError
    ├── ArithmeticError
    │   └── ZeroDivisionError
    └── RuntimeError
        └── RecursionError
```

This is a partial hierarchy. Catching a parent catches its subclasses. `except Exception` catches most ordinary application failures, but intentionally excludes direct `BaseException` subclasses such as `KeyboardInterrupt`.

| Exception | Typical meaning |
|---|---|
| `TypeError` | Wrong kind of object or unsupported operation |
| `ValueError` | Appropriate type, invalid value |
| `KeyError` | Requested mapping key is absent |
| `IndexError` | Sequence index is outside bounds |
| `AttributeError` | Requested attribute is absent |
| `NameError` | Name cannot be resolved |
| `UnboundLocalError` | A local name is read before being assigned |
| `ImportError` / `ModuleNotFoundError` | Import could not be completed |
| `AssertionError` | An assertion failed |

## Try, except, else, and finally

```python
def reciprocal_from_text(text):
    try:
        number = float(text)
    except ValueError:
        return "Please enter a numeric value."
    else:
        if number == 0:
            return "Zero has no reciprocal."
        return 1 / number


assert reciprocal_from_text("4") == 0.25
assert reciprocal_from_text("no") == "Please enter a numeric value."
```

| Clause | When it runs | Usual role |
|---|---|---|
| `try` | Initially | Small region expected to fail |
| `except SomeError as exc` | Matching exception from `try` | Recover or translate |
| `else` | `try` completes normally | Work that should not use those handlers |
| `finally` | When control leaves through normal completion or exception | Cleanup |

Handlers for a `try` do not catch exceptions raised by its own `else` or handler bodies. Put specific handlers before general ones. Use `except (TypeError, ValueError) as exc:` when recovery is identical for both.

During normal Python control flow, `finally` runs even when `try` returns. Forced process termination and some catastrophic failures can bypass cleanup. A `return` or new exception in `finally` can replace a pending return or exception, so avoid control-flow exits there. These semantics are detailed in the [Python exception tutorial](https://docs.python.org/3/tutorial/errors.html).

```python
events = []


def demonstrate_cleanup():
    try:
        events.append("work")
        return 7
    finally:
        events.append("cleanup")


assert demonstrate_cleanup() == 7
assert events == ["work", "cleanup"]
```

## Raising and chaining

Use `raise` to enforce a contract at the point where it becomes invalid:

```python
def require_positive(value):
    if isinstance(value, bool) or not isinstance(value, (int, float)):
        raise TypeError("value must be numeric")
    # This also rejects NaN; +infinity requires a separate finite-value rule.
    if not value > 0:
        raise ValueError("value must be positive")
    return value


assert require_positive(3) == 3
```

Inside a handler, bare `raise` reraises the active exception while preserving the original traceback context. Prefer it to `raise exc` when simply propagating the same error.

```python
class ConfigurationError(Exception):
    """The supplied configuration cannot be used."""


def parse_port(text):
    try:
        port = int(text)
    except (TypeError, ValueError) as exc:
        raise ConfigurationError("port must be an integer") from exc
    if not 1 <= port <= 65535:
        raise ConfigurationError("port must be between 1 and 65535")
    return port


assert parse_port("8000") == 8000
try:
    parse_port("eight")
except ConfigurationError as exc:
    assert isinstance(exc.__cause__, ValueError)
```

`raise NewError(...) from exc` explicitly records a cause. `from None` suppresses the earlier exception in the displayed chain; use it only when a simplified error is helpful and the hidden detail is not needed by the caller.

Python 3.11+ supports `exc.add_note("additional context")`. Add context such as a record number without replacing the original error. Do not put credentials or sensitive payloads into messages or notes.

## Designing custom exceptions

Use a small hierarchy that allows callers to distinguish recoverable cases:

```python
class ImportJobError(Exception):
    """Base class for import failures."""


class InvalidRowError(ImportJobError):
    def __init__(self, row_number, reason):
        self.row_number = row_number
        self.reason = reason
        super().__init__(f"row {row_number}: {reason}")


class SourceUnavailableError(ImportJobError):
    pass


error = InvalidRowError(4, "missing customer ID")
assert error.row_number == 4
assert str(error) == "row 4: missing customer ID"
```

Inherit ordinary custom errors from `Exception` or a suitable existing subclass, not directly from `BaseException`. Use attributes for structured information rather than requiring callers to parse message strings.

Raise exceptions for failed operations. Use ordinary return values for expected alternatives, such as a search that finds no match, when the API contract supports that. Do not return an exception object when you mean to raise it.

## Cleanup and context managers

Use `with` for files, locks, and other resources with context-manager support. It expresses lifetime locally and reduces cleanup duplication.

```python
from contextlib import contextmanager


@contextmanager
def recorded_resource(events):
    events.append("acquire")
    try:
        yield "resource"
    finally:
        events.append("release")


events = []
try:
    with recorded_resource(events) as resource:
        assert resource == "resource"
        raise ValueError("example failure")
except ValueError:
    pass
assert events == ["acquire", "release"]
```

`contextlib.suppress(SpecificError)` deliberately ignores a narrow known failure. `ExitStack` manages a dynamic number of context managers. Neither is a reason to hide unexpected failures.

For a class-based manager, `__exit__` returning a truthy value suppresses the exception. Return `False` or `None` when the exception should propagate. See [advanced OOP protocols](../03_oop_principles/04-special-methods-and-protocols.md).

## Exception groups

Python 3.11+ can represent multiple failures with `ExceptionGroup`; `except*` handles matching subgroups. This is useful when independent tasks fail together, including structured concurrency.

```python
handled = []
try:
    raise ExceptionGroup("batch failed", [ValueError("bad value"), TypeError("bad type")])
except* ValueError as group:
    handled.append(("value", len(group.exceptions)))
except* TypeError as group:
    handled.append(("type", len(group.exceptions)))

assert handled == [("value", 1), ("type", 1)]
```

Unmatched exceptions continue propagating. `except*` and ordinary `except` cannot be mixed on the same `try`. Unlike an ordinary handler selection, more than one `except*` clause can run. Avoid flattening groups into one message that loses individual causes.

## Logging, retries, and API boundaries

### Log where the error is handled

`logging.exception("Import failed")` inside a handler records the active traceback. In reusable libraries, prefer a module logger from `logging.getLogger(__name__)`; let the application configure output.

Avoid logging and reraising at every level, which produces duplicate tracebacks. Log once at the boundary that can decide what the failure means for the operation.

### Retry only suitable failures

A retry policy needs a bounded attempt count, a timeout for each external operation, backoff, and a rule for which failures are transient. Validation failures will not usually fix themselves. Retrying a payment or write can duplicate work unless the operation is idempotent or uses an idempotency key.

```python
def retry_transient(operation, attempts=3):
    """Small teaching example; production I/O also needs timeouts and backoff."""
    if attempts < 1:
        raise ValueError("attempts must be at least one")
    for attempt in range(attempts):
        try:
            return operation()
        except TimeoutError:
            if attempt == attempts - 1:
                raise


calls = []


def eventually_ready():
    calls.append(1)
    if len(calls) < 2:
        raise TimeoutError("temporarily unavailable")
    return "ready"


assert retry_transient(eventually_ready) == "ready"
```

For asynchronous code, preserve cancellation and use the async library's timeout and task-management APIs. Blocking sleep inside an async function blocks the event loop.

### EAFP and LBYL

**EAFP:** Attempt an operation and handle a specific failure. **LBYL:** Check a condition before attempting it. Use explicit checks for clear input contracts; use operation-and-handler patterns where a precheck can race with external state. Keep handlers narrow so an unrelated bug is not mistaken for an expected failure.

## Testing and common mistakes

```python
import unittest


class PortTests(unittest.TestCase):
    def test_invalid_port(self):
        with self.assertRaises(ConfigurationError):
            parse_port("invalid")

    def test_boundary(self):
        self.assertEqual(parse_port("65535"), 65535)


result = unittest.TestResult()
unittest.defaultTestLoader.loadTestsFromTestCase(PortTests).run(result)
assert result.wasSuccessful()
```

| Mistake | Better approach |
|---|---|
| Bare `except:` | Catch the specific expected class |
| `except Exception: pass` | Recover explicitly or let the failure surface |
| Catching around a huge function body | Isolate the operation that may legitimately fail |
| Returning from `finally` | Reserve `finally` for cleanup |
| Using `assert` for user validation | Raise an explicit exception; assertions can be disabled |
| Losing the original failure | Use exception chaining |
| Retrying every error forever | Bound retries and classify failures |

Practice: implement a CSV loader that distinguishes unavailable files, invalid encoding, and invalid rows; test each failure independently. Explain when an error should be translated at an API boundary and when it should propagate unchanged.

References: [built-in exceptions](https://docs.python.org/3/library/exceptions.html), [contextlib](https://docs.python.org/3/library/contextlib.html), [logging](https://docs.python.org/3/library/logging.html).
