# 8. OOP Mini-Project: Experiment Results

> **Learning goal:** Combine value models, an interface, composition, validation, and tests in a small ML-related example.

## Contents

- [The design](#the-design)
- [Complete implementation](#complete-implementation)
- [Behavioral tests](#behavioral-tests)
- [Project organization](#project-organization)
- [Extensions and interview review](#extensions-and-interview-review)

## The design

We want to record a named experiment result, reject invalid metrics, reject duplicate run IDs, and retrieve saved runs. We will use:

- An immutable `RunResult` value with validation.
- A `RunRepository` protocol describing storage operations.
- An in-memory repository for local use and tests.
- An `ExperimentService` that depends on that interface.

This example stores one score between zero and one. Real projects need a metric name, direction, evaluation dataset identity, code/data versions, and potentially multiple metrics; those are deliberate extensions rather than implied by a generic field called `score`.

## Complete implementation

```python
from dataclasses import dataclass
from math import isfinite
from typing import Protocol


@dataclass(frozen=True)
class RunResult:
    run_id: str
    model_name: str
    score: float

    def __post_init__(self):
        for name in ("run_id", "model_name"):
            value = getattr(self, name)
            if not isinstance(value, str) or not value.strip():
                raise ValueError(f"{name} must be a nonempty string")
        if isinstance(self.score, bool) or not isinstance(self.score, (int, float)):
            raise TypeError("score must be numeric")
        if not isfinite(self.score) or not 0 <= self.score <= 1:
            raise ValueError("score must be finite and between zero and one")


class DuplicateRunError(ValueError):
    pass


class RunRepository(Protocol):
    def add(self, run: RunResult) -> None:
        """Add a run, raising DuplicateRunError when its ID already exists."""
        ...

    def get(self, run_id: str) -> RunResult:
        """Return a run, raising KeyError when absent."""
        ...


class InMemoryRunRepository:
    def __init__(self):
        self._runs: dict[str, RunResult] = {}

    def add(self, run: RunResult) -> None:
        if run.run_id in self._runs:
            raise DuplicateRunError(f"run already exists: {run.run_id}")
        self._runs[run.run_id] = run

    def get(self, run_id: str) -> RunResult:
        return self._runs[run_id]


class ExperimentService:
    def __init__(self, repository: RunRepository):
        self.repository = repository

    def record(self, run_id: str, model_name: str, score: float) -> RunResult:
        run = RunResult(run_id, model_name, score)
        self.repository.add(run)
        return run


repository = InMemoryRunRepository()
service = ExperimentService(repository)
saved = service.record("run-001", "baseline", 0.82)
assert repository.get("run-001") == saved
```

The repository owns uniqueness. This avoids a service-level “check, then add” contract that would still need database enforcement in concurrent implementations. The in-memory implementation itself is single-threaded teaching code; a database version should enforce uniqueness with an actual constraint.

Returning a frozen value makes accidental field mutation harder. Since these fields are scalar values, there are no nested mutable containers to protect.

## Behavioral tests

Run this after the implementation above, or import the classes into a test module.

```python
import unittest


class ExperimentTests(unittest.TestCase):
    def setUp(self):
        self.repository = InMemoryRunRepository()
        self.service = ExperimentService(self.repository)

    def test_round_trip(self):
        expected = self.service.record("one", "baseline", 0.75)
        self.assertEqual(self.repository.get("one"), expected)

    def test_duplicate_does_not_replace_original(self):
        original = self.service.record("one", "baseline", 0.75)
        with self.assertRaises(DuplicateRunError):
            self.service.record("one", "replacement", 0.9)
        self.assertEqual(self.repository.get("one"), original)

    def test_invalid_score_is_not_saved(self):
        with self.assertRaises(ValueError):
            self.service.record("bad", "baseline", float("nan"))
        with self.assertRaises(KeyError):
            self.repository.get("bad")

    def test_zero_is_valid(self):
        self.assertEqual(self.service.record("zero", "baseline", 0).score, 0)


result = unittest.TestResult()
unittest.defaultTestLoader.loadTestsFromTestCase(ExperimentTests).run(result)
assert result.wasSuccessful(), (result.failures, result.errors)
```

These tests check meaningful behavior: valid round trips, boundaries, failure preservation, and uniqueness. They do not depend on the private dictionary name or every internal method call.

Useful test doubles include a **fake** implementation with working simplified behavior, a **stub** with predetermined results, and a **mock** used to verify interactions. Use interaction assertions when the interaction itself is the contract, such as sending exactly one event after a successful commit.

## Project organization

```text
experiment_tracker/
├── pyproject.toml
├── src/
│   └── experiment_tracker/
│       ├── __init__.py
│       ├── models.py
│       ├── repositories.py
│       └── services.py
└── tests/
    ├── test_models.py
    └── test_services.py
```

Split modules when responsibilities justify it; the single-file example is intentionally easy to run. Use dependency injection at the application entry point to select the repository. Keep UI, database connections, and training code out of the core value model.

For a larger system, test storage implementations against the same repository contract. Integration tests verify actual database behavior, while unit tests keep domain rules fast and isolated.

## Extensions and interview review

1. Add a metric name and explicit maximize/minimize direction.
2. Add a timestamp using an injected clock, then test deterministically.
3. Implement JSON export with an explicit serialization schema.
4. Implement SQLite storage with uniqueness and transaction handling.
5. Add source-code commit and dataset-version identifiers.

Before extending, decide whether new information belongs to the value object, service, repository, or presentation layer.

Quick interview prompts: explain `self` and `cls`; compare a Protocol with an ABC; describe MRO and `super`; explain shallow versus deep immutability; state the equality/hash contract; describe when composition is preferable; explain why annotations do not validate runtime data; choose between a property, descriptor, and metaclass.

Return to the [OOP index](README.md).
