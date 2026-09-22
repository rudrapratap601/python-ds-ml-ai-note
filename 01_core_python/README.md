# Core Python Notes

Start here to learn how Python stores data, makes decisions, repeats operations, and organizes reusable logic. These notes provide the foundation for the repository's advanced Python, object-oriented programming, and data science sections.

## Reading order

| Order | Guide | Main topics |
|---|---|---|
| 1 | [Python Data Structures](Python%20Data%20Structures.md) | Strings, lists, tuples, sets, dictionaries, indexing, slicing, methods, comprehensions, and nested collections |
| 2 | [Conditional Statements and Loops](conditional-statement-loops.md) | if/elif/else, truthiness, match-case, for/while, range, enumerate, zip, break, continue, and loop else |
| 3 | [Functions, Lambda, Map, and Filter](functions-lambda-map-filter.md) | Parameters, return values, default and keyword arguments, args/kwargs, scope, docstrings, type hints, lambdas, map, and filter |

The guides overlap intentionally: collections supply the data, control flow processes it, and functions make the processing reusable. Revisit comprehensions after learning loops if their syntax is unfamiliar.

## Before you begin

Be comfortable assigning variables, using basic arithmetic and comparisons, and running a small Python script or notebook cell. Use Python 3.10 or later for the match-case examples. Read any additional requirements attached to individual examples.

These files are learning notes, not complete scripts. Some blocks demonstrate mistakes, partial syntax, or an infinite loop; read the surrounding explanation before running them. Examples that refer to earlier variables need those definitions first.

## How to study

1. Read a concept and predict the example's output before executing it.
2. Change one input and explain why the result changes.
3. Recreate the example without copying it, using your own data.
4. Test an empty collection, a missing key, repeated values, and boundary conditions where relevant.
5. Use each guide's quick reference and interview questions for revision.

Prefer clear code before shortening it. A named function or explicit loop can be easier to understand than a complicated lambda or comprehension.

## Practice milestones

| After studying | Try this |
|---|---|
| Data structures | Store student records as a list of dictionaries, extract names, and find unique subjects |
| Conditions and loops | Classify scores, skip missing values, and stop searching when a matching record is found |
| Functions | Turn score classification into a function with clear inputs and a return value |
| All three guides | Build a small record-processing program that filters valid records and summarizes them by category |

Before moving on, be able to explain the difference between mutable and immutable values, choose a collection for a task, trace a loop, distinguish print from return, and pass arguments to a function deliberately.

## Continue learning

- [Advanced Python](../02_advanced_python/README.md): namespaces, exceptions, files, recursion, generators, decorators, and GUI development.
- [Object-Oriented Programming](../03_oop_principles/README.md): classes, composition, inheritance, and advanced object behavior.
- [Data Science Libraries](../04_data_science/README.md): numerical computing, tabular analysis, and visualization.

Return to the [repository index](../README.md).
