# Polars: Expression-Based DataFrames

> **Purpose:** Learn typed columnar data processing with expressions, eager and lazy execution, joins, windows, and streaming-aware query design. The library is named **Polars** and imported as `polars`.

## Contents

- [Setup and core model](#setup-and-core-model)
- [Expressions and contexts](#expressions-and-contexts)
- [Types and missing values](#types-and-missing-values)
- [Grouping and windows](#grouping-and-windows)
- [Joins, concatenation, and reshaping](#joins-concatenation-and-reshaping)
- [Strings, dates, and nested data](#strings-dates-and-nested-data)
- [Lazy queries and optimization](#lazy-queries-and-optimization)
- [Files and streaming](#files-and-streaming)
- [Pandas translation and performance](#pandas-translation-and-performance)
- [Testing and practice](#testing-and-practice)

## Setup and core model

```bash
python -m pip install polars
```

```python
import polars as pl

sales = pl.DataFrame({
    "order_id": [101, 102, 103, 104],
    "region": ["North", "South", "North", "South"],
    "units": [2, 1, 3, 2],
    "price": [10., 20., 10., 15.],
    "date": ["2026-01-01", "2026-01-02", "2026-01-02", "2026-01-03"],
})
assert sales.shape == (4, 5)
assert sales.schema["units"] == pl.Int64
```

A `Series` is one typed column, a `DataFrame` is an eager table, a `LazyFrame` is a query plan, and an `Expr` describes a computation. Polars has no pandas-style implicit row index; join and alignment keys are explicit columns.

Polars can parallelize suitable native operations. This does not mean every query, data size, or Python callback will be faster than every alternative. Measure the complete workload, including I/O and conversion.

## Expressions and contexts

```python
sales = sales.with_columns(
    (pl.col("units") * pl.col("price")).alias("revenue")
)
selected = sales.select("order_id", "revenue")
filtered = sales.filter((pl.col("region") == "North") & (pl.col("units") >= 2))
assert selected.columns == ["order_id", "revenue"]
assert filtered["order_id"].to_list() == [101, 103]
```

| Context | Purpose |
|---|---|
| `select` | Produce selected/computed output columns |
| `with_columns` | Add or replace columns while retaining others |
| `filter` | Keep rows matching Boolean expressions |
| `group_by(...).agg(...)` | Evaluate aggregates within groups |

Expressions describe operations rather than immediately holding a column result. The same expression can be reused in different contexts. See [expressions and contexts](https://docs.pola.rs/user-guide/concepts/expressions-and-contexts/).

Sibling expressions in one `with_columns` call should not depend on aliases created by another sibling. Chain contexts when one result depends on another:

```python
derived = sales.with_columns(
    (pl.col("revenue") * 0.1).alias("tax")
).with_columns(
    (pl.col("revenue") + pl.col("tax")).alias("gross")
)
assert derived["gross"].to_list() == [22., 22., 33., 33.]
```

Use `pl.lit("text")` when an expression argument must be a literal string rather than a column name. `pl.when(...).then(...).otherwise(...)` is expression-based conditional selection; branches must be valid independently, so do not rely on it as Python-style short-circuit control flow.

## Types and missing values

```python
raw = pl.DataFrame({"quantity": ["2", "bad", None]})
clean = raw.with_columns(pl.col("quantity").cast(pl.Int64, strict=False))
assert clean["quantity"].to_list() == [2, None, None]
assert clean["quantity"].null_count() == 2

floats = pl.DataFrame({"value": [1., float("nan"), None]})
fixed = floats.with_columns(pl.col("value").fill_nan(0).fill_null(0))
assert fixed["value"].to_list() == [1., 0., 0.]
```

`null` is missing data; floating-point NaN is a numeric special value. `fill_null` and `fill_nan` solve different problems. `count` counts non-null values, while `pl.len()` counts rows in an aggregation. A filter keeps `True` rows; null predicate values do not act like `True`.

Use explicit schemas or `schema_overrides` for important identifiers and numeric columns. `strict=False` converts failed casts to null; inspect how many failures occurred. Integer identifiers with leading zeros should generally stay strings.

Common types include signed/unsigned integers, floats, Boolean, String, Date, Datetime, Duration, Decimal, List, Array, Struct, Categorical, and Enum. A List has variable-length elements; an Array has a fixed inner shape. Categorical/Enum behavior and interoperability can be version-sensitive; consult the installed version's guide.

## Grouping and windows

```python
summary = sales.group_by("region").agg(
    pl.len().alias("orders"),
    pl.col("revenue").sum().alias("revenue"),
).sort("region")
assert summary["orders"].to_list() == [2, 2]

with_shares = sales.with_columns(
    (pl.col("revenue") / pl.col("revenue").sum().over("region")).alias("share")
)
assert with_shares.height == sales.height
```

`group_by` reduces to grouped results; `.over(...)` broadcasts a window result back to rows. Do not assume group output order unless you request or establish it. Sorting final results gives an explicit deterministic order; preserving group encounter order can constrain optimization.

Useful expressions include `sum`, `mean`, `median`, `min`, `max`, `n_unique`, `first`, `last`, `rank`, `shift`, `cum_sum`, and rolling operations. Sort by the relevant time/key columns before order-dependent calculations.

For time-window aggregation, `group_by_dynamic` forms time bins on a suitably sorted temporal column. Define bin interval, period, boundaries, and grouping keys explicitly. A rolling row window and a calendar-duration window are not interchangeable.

## Joins, concatenation, and reshaping

```python
regions = pl.DataFrame({"region": ["North", "South"], "manager": ["Asha", "Dev"]})
joined = sales.join(regions, on="region", how="left", validate="m:1")
assert joined.height == sales.height
assert joined["manager"].null_count() == 0

wide = pl.DataFrame({"student": ["A", "B"], "math": [8, 9], "python": [9, 10]})
long = wide.unpivot(index="student", on=["math", "python"], variable_name="subject", value_name="score")
assert long.shape == (4, 3)
```

Use inner, left, full, semi, anti, or cross joins according to the question. A semi join keeps matching left rows without bringing right columns; an anti join finds unmatched left rows. Null keys do not match by default; request alternate behavior explicitly only when appropriate.

Check join cardinality and row counts; duplicates can multiply data. As-of joins need sorted keys and a deliberate strategy/tolerance. Execution-engine support for validation or other join options can differ, so test the actual plan you deploy.

`pl.concat` supports vertical and horizontal combinations and schema-aware variants. Prefer explicit compatible schemas over silently relaxing types. `pivot` creates wide tables; dynamic output columns complicate lazy schemas. `unpivot` creates long data; `explode` expands list values.

## Strings, dates, and nested data

```python
prepared = sales.with_columns(
    pl.col("date").str.to_date("%Y-%m-%d"),
    pl.col("region").str.to_lowercase().alias("region_key"),
)
assert prepared.schema["date"] == pl.Date
assert prepared["region_key"][0] == "north"

nested = pl.DataFrame({"id": [1, 2], "scores": [[8, 9], [10]]})
lengths = nested.select(pl.col("scores").list.len().alias("count"))
assert lengths["count"].to_list() == [2, 1]
assert nested.explode("scores", empty_as_null=True).height == 3
```

String operations live under `.str`, temporal operations under `.dt`, and list operations under `.list`. Struct fields can bundle related columns and later be selected with `.struct.field(...)` or expanded.

Parse datetimes with a known format where possible. Assigning a timezone and converting between timezones are different operations; daylight-saving behavior requires an explicit policy. Avoid using a text column as a time key when lexical ordering does not match chronological ordering.

## Lazy queries and optimization

```python
query = (
    sales.lazy()
    .filter(pl.col("units") >= 2)
    .group_by("region")
    .agg(pl.col("revenue").sum().alias("revenue"))
    .sort("region")
)
result = query.collect()
assert result["revenue"].to_list() == [50., 30.]
plan = query.explain()
assert isinstance(plan, str)
```

Lazy execution allows the optimizer to see the complete query. It may push filters and column selections toward the source, simplify expressions, and choose execution strategies. `collect()` executes and materializes the result; `explain()` helps inspect the plan. See [lazy API usage](https://docs.pola.rs/user-guide/lazy/using/).

Calling `.lazy()` after `read_csv()` does not undo the eager read. Use `scan_csv` or `scan_parquet` when the source should participate in a lazy query. Avoid frequent intermediate `collect()` calls that split the plan and allocate unnecessary tables.

## Files and streaming

This example creates a temporary Parquet file so it can be run without an external dataset:

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    path = Path(directory) / "sales.parquet"
    sales.write_parquet(path)
    query = (
        pl.scan_parquet(path)
        .filter(pl.col("units") >= 2)
        .select("order_id", "revenue")
        .sort("order_id")
    )
    result = query.collect(engine="streaming")
    assert result["order_id"].to_list() == [101, 103, 104]
```

Modern Polars accepts `collect(engine="streaming")`; older examples may show different flags. Check the [current collect reference](https://docs.pola.rs/api/python/stable/reference/lazyframe/api/polars.LazyFrame.collect.html) against your installed version.

Streaming can reduce intermediate memory, but `collect` still returns a materialized result and not every operation has identical streaming support. Large joins, sorts, group state, or final results may remain expensive. Supported `sink_*` operations can write a lazy result directly when materializing the whole output is unnecessary.

CSV is text with inferred types; Parquet carries typed columnar data and often supports more effective column/filter pruning. JSON Lines is useful for record streams. Database/cloud connectors require their own dependencies and credentials.

## Pandas translation and performance

| pandas habit | Polars approach |
|---|---|
| `df.loc[mask, columns]` | `df.filter(expression).select(columns)` |
| `df.assign(...)` | `df.with_columns(...)` |
| `groupby(...).agg(...)` | `group_by(...).agg(...)` |
| Group-aligned `transform` | Window expression with `.over(...)` |
| Implicit index alignment | Explicit keys and joins |
| Row-wise Python `apply` | Native expressions whenever possible |
| `melt` | `unpivot` |

Avoid translating pandas line for line. Build expressions that let Polars operate on columns. Python UDFs such as `map_elements` can be slower, limit optimization, and require explicit return-type handling. Use them only when no suitable native expression exists.

Conversions through `to_pandas`, `to_numpy`, or Arrow can allocate and alter dtype/null representations. Additional dependencies may be needed. Include those conversions in benchmarks; a faster isolated transform does not guarantee a faster full application.

## Testing and practice

```python
from polars.testing import assert_frame_equal

expected = pl.DataFrame({"region": ["North", "South"], "orders": [2, 2], "revenue": [50., 50.]})
assert_frame_equal(summary, expected, check_dtypes=False)
```

Use dtype checks in production tests when schema is part of the contract; this example disables them only because row-count aggregate widths may differ from manually inferred integers. Check schema, null counts, key uniqueness, row counts, and sorted output where ordering matters.

Practice: recreate the pandas revenue pipeline; inspect a scan/filter/select plan; compare eager and lazy results; process nested list columns; detect unmatched dimension keys with an anti join; measure memory with and without materializing intermediate results.

References: [Polars user guide](https://docs.pola.rs/user-guide/), [Python API](https://docs.pola.rs/api/python/stable/reference/index.html).
