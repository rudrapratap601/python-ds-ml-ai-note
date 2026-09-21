# Data Science Library Notes

Seven separate guides cover the main numerical, tabular, scientific, and visualization tools. Each includes examples, interpretation, common failures, and practice tasks.

## Library map and reading order

| Library | Guide | Primary role |
|---|---|---|
| NumPy | [Arrays and numerical computing](numpy.md) | Shapes, broadcasting, numerical operations, random sampling, linear algebra |
| pandas | [Data cleaning and analysis](pandas.md) | Labeled tables, missing data, joins, groups, reshaping, time series |
| SciPy | [Scientific algorithms](scipy.md) | Optimization, statistics, integration, signals, sparse/spatial computation |
| Polars | [Expression-based DataFrames](polars.md) | Typed tables, expressions, lazy plans, streaming-aware processing |
| Matplotlib | [Scientific figures](matplotlib.md) | Explicit figure/axes control and publication-style exports |
| Seaborn | [Statistical visualization](seaborn.md) | Distributions, grouped estimates, facets, exploratory views |
| Plotly | [Interactive visualization](plotly.md) | Interactive figures, hover, subplots, HTML export, dashboard integration |

Start with NumPy, then pandas and Matplotlib. Add SciPy for scientific algorithms, Seaborn for statistical exploration, Polars for another table-processing model, and Plotly for interactive delivery.

## Environment setup

```bash
python -m venv .venv
# Activate the environment using the command appropriate to your shell.
python -m pip install numpy pandas scipy polars matplotlib seaborn plotly
```

Install optional engines only when needed: Excel, Parquet interoperability, static Plotly exports, and GUI frameworks have additional requirements. Keep project dependency versions recorded rather than assuming that all future releases behave identically.

## Example conventions

- Run a guide's Python blocks in order in one session unless a block is marked as a standalone application.
- Examples use small local synthetic datasets and temporary files where possible.
- Plotting examples build figures without opening windows automatically. Use `plt.show()` or `fig.show()` to display them in an appropriate environment.
- A successful numerical example does not establish statistical assumptions or validate a real dataset.
- Official documentation links identify version-sensitive behavior such as pandas Copy-on-Write and Polars streaming execution.

## Practical workflow

1. Define the question, unit of observation, schema, and expected keys.
2. Load only the required data and inspect types, missingness, and duplicates.
3. Validate joins and transformations with row counts and invariants.
4. Analyze using methods whose assumptions match the data.
5. Choose a visualization that answers the question with clear units and uncertainty.
6. Preserve code, data versions, environments, seeds, and outputs needed for reproduction.

Build an interactive result using the [Streamlit guide](../02_advanced_python/gui-development-and-streamlit.md). Use [OOP notes](../03_oop_principles/README.md) when a project needs reusable services and well-defined state.

## Validation snapshot

The Python examples were executed on 2026-09-21 with Python 3.12.14, NumPy 2.5.3, pandas 3.0.6, SciPy 1.18.1, Polars 1.44.2, Matplotlib 3.11.2, Seaborn 0.13.2, and Plotly 7.1.0. PNG generation and Plotly HTML serialization were checked; interactive browser rendering and optional static Plotly export were not exercised.

Seaborn 0.13.2 emitted upstream deprecation warnings involving newer Matplotlib/pandas internals while its examples passed. Check compatible library versions when reproducing the notes; do not suppress warnings indiscriminately.
