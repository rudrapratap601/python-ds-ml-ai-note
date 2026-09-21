# Advanced Python Notes

Read these guides after the [core Python notes](../01_core_python/). Each guide combines explanation, examples, pitfalls, and practice prompts.

## Reading order

| Order | Guide | Main topics |
|---|---|---|
| 1 | [Namespaces and scope](namespaces-and-scope.md) | Names, bindings, LEGB, closures, imports |
| 2 | [Exception handling](exception-handling.md) | Propagation, recovery, chaining, cleanup, exception groups |
| 3 | [File handling](file-handling.md) | Paths, modes, text/bytes, CSV/JSON, streaming, replacement |
| 4 | [Recursion](recursion.md) | Call stack, memoization, divide and conquer, DFS, backtracking |
| 5 | [Generators and iterators](generators-and-iterators.md) | Lazy execution, iterator protocols, pipelines, async generators |
| 6 | [Decorators](decorators.md) | Wrappers, factories, ordering, caching, async behavior, typing |
| 7 | [GUI development and Streamlit](gui-development-and-streamlit.md) | Desktop foundations, Streamlit state/cache, complete CSV app, testing |

## How to use the examples

Core notes generally target Python 3.11+, with version-specific features identified where relevant. Run blocks in their document order when they build on earlier definitions. GUI applications are explicitly marked as standalone programs; save and run those separately.

The guides are broad learning references rather than copies of every API entry. Follow official links for detailed signatures, platform behavior, and newly introduced features. Install third-party libraries in a project environment and record versions before reproducing an application.

Next: [Object-oriented programming](../03_oop_principles/README.md) and [data science libraries](../04_data_science/README.md).

## Validation snapshot

Examples were checked with Python 3.12.14 on 2026-09-21. The three standalone Streamlit examples passed AppTest checks with Streamlit 1.64.0, including slider, counter, category-filter, and empty-selection interactions. The Tkinter example was syntax-checked; a desktop window was not launched. Hosted deployment and browser layout are separate checks.
