# pandas: Data Cleaning and Analysis

> **Purpose:** Work with labeled tables from ingestion through validation, joins, aggregation, time series, and export. Examples run in order in one session. Version-sensitive behavior is called out explicitly.

## Contents

- [Setup and table concepts](#setup-and-table-concepts)
- [Loading and inspecting data](#loading-and-inspecting-data)
- [Selecting and updating](#selecting-and-updating)
- [Types, missing data, and duplicates](#types-missing-data-and-duplicates)
- [Strings, dates, and categories](#strings-dates-and-categories)
- [Groupby, aggregation, and transform](#groupby-aggregation-and-transform)
- [Joins and concatenation](#joins-and-concatenation)
- [Reshaping](#reshaping)
- [Time series and windows](#time-series-and-windows)
- [Copy-on-Write and performance](#copy-on-write-and-performance)
- [A small validated pipeline](#a-small-validated-pipeline)
- [Testing, pitfalls, and practice](#testing-pitfalls-and-practice)

## Setup and table concepts

```bash
python -m pip install pandas
```

```python
import pandas as pd
import numpy as np

sales = pd.DataFrame({
    "order_id": [101, 102, 103, 104],
    "region": ["North", "South", "North", "South"],
    "units": [2, 1, 3, 2],
    "price": [10., 20., 10., 15.],
    "date": ["2026-01-01", "2026-01-02", "2026-01-02", "2026-01-03"],
})
assert sales.shape == (4, 5)
assert isinstance(sales["units"], pd.Series)
```

A Series is one labeled array; a DataFrame is a labeled table with potentially different column dtypes. The index labels rows and participates in alignment. It is not necessarily a primary key and can contain duplicates. See the [pandas introduction](https://pandas.pydata.org/docs/user_guide/10min.html).

Arithmetic and assignment often align by labels rather than position:

```python
left = pd.Series([10, 20], index=["a", "b"])
right = pd.Series([1, 2], index=["b", "a"])
assert (left + right).to_dict() == {"a": 12, "b": 21}
```

Convert to positional arrays only when deliberately discarding alignment. Check index uniqueness where joins or lookups require it.

## Loading and inspecting data

| Source | Read | Write | Notes |
|---|---|---|---|
| CSV | `read_csv` | `to_csv` | Specify types, null markers, encoding, and `index=False` when appropriate |
| Excel | `read_excel` | `to_excel` | Requires a compatible engine |
| Parquet | `read_parquet` | `to_parquet` | Typed columnar storage; requires an engine |
| JSON | `read_json`, `json_normalize` | `to_json` | Choose orientation and line-delimited behavior |
| SQL | `read_sql_query` | `to_sql` | Parameterize values; use transactions and suitable connections |

```python
from io import StringIO

raw = StringIO("id,quantity\n001,2\n002,\n")
loaded = pd.read_csv(raw, dtype={"id": "string", "quantity": "Int64"})
assert loaded.loc[0, "id"] == "001"
assert pd.isna(loaded.loc[1, "quantity"])
```

Inspect `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()`, `nunique()`, and missing-value counts. `memory_usage(deep=True)` helps estimate object/string memory. Inference can turn identifiers into numbers or parse mixed data inconsistently; explicit schemas are more reliable.

Use `usecols` and `dtype` to reduce work. `read_csv(..., chunksize=...)` iterates chunks, but combining all chunks immediately into one huge frame removes the memory advantage.

## Selecting and updating

```python
assert sales.loc[0, "order_id"] == 101
assert sales.iloc[0, 0] == 101
subset = sales.loc[(sales["region"] == "North") & (sales["units"] >= 2), ["order_id", "units"]]
assert subset["order_id"].tolist() == [101, 103]

sales = sales.assign(revenue=lambda frame: frame["units"] * frame["price"])
assert sales["revenue"].tolist() == [20., 20., 30., 30.]
sales.loc[sales["order_id"] == 102, "units"] = 2
sales["revenue"] = sales["units"] * sales["price"]
```

| Selector | Interpretation |
|---|---|
| `df["column"]` | One Series |
| `df[["a", "b"]]` | A DataFrame with selected columns |
| `df.loc[rows, columns]` | Label-based selection; label slice endpoints are inclusive |
| `df.iloc[rows, columns]` | Position-based selection; normal half-open slicing |
| `df.at[label, column]` | A scalar by label |
| `df.iat[row, column]` | A scalar by position |

Use one `.loc` assignment instead of chained assignment such as `df[mask]["x"] = value`. Parenthesize vector comparisons and use `&`, `|`, and `~`. `query()` is convenient for trusted expressions; do not treat it as a safe evaluator of untrusted user input.

`sort_values` sorts data values; `sort_index` sorts labels. `set_index` changes row labels; `reset_index` moves them back into columns unless `drop=True` discards them.

## Types, missing data, and duplicates

```python
values = pd.Series(["10", "bad", None])
numeric = pd.to_numeric(values, errors="coerce")
assert numeric.isna().sum() == 2

nullable = pd.Series([1, None, 3], dtype="Int64")
assert str(nullable.dtype) == "Int64"
assert nullable.fillna(0).tolist() == [1, 0, 3]
```

`Int64` is a nullable integer dtype, distinct from NumPy `int64`. pandas supports missing markers including `np.nan`, `pd.NA`, and `NaT`, depending on dtype. Use `isna`/`notna`, not equality to a missing marker.

Choose a missing-data policy based on meaning: reject, retain, impute, or remove. `fillna(0)` changes missing to a real zero; it is not universally correct. `ffill()` propagates earlier values only when ordering and domain semantics justify it. `dropna(subset=[...])` targets required columns.

`duplicated(subset=...)` detects duplicates; `drop_duplicates` removes according to an explicit keep rule. Define the business key first. Two rows with different timestamps may be distinct events even if other values match.

`astype` changes dtype and can fail; `to_numeric(errors="coerce")` makes failures missing. Record how many values were coerced so data-quality problems do not disappear silently.

## Strings, dates, and categories

```python
labels = pd.Series([" North ", "SOUTH", None], dtype="string")
cleaned = labels.str.strip().str.lower()
assert cleaned.iloc[0] == "north"
assert cleaned.str.contains("south", regex=False, na=False).tolist() == [False, True, False]

sales["date"] = pd.to_datetime(sales["date"], utc=True)
assert sales["date"].dt.year.eq(2026).all()
sales["region"] = sales["region"].astype("category")
```

String methods live under `.str`; datetime components under `.dt`. `str.contains` treats patterns as regex by default, so use `regex=False` for literal substrings.

`tz_localize` attaches a timezone to naive local timestamps; `tz_convert` changes the timezone of already aware timestamps. Daylight-saving ambiguity and nonexistent local times require explicit policy. `utc=True` is suitable when the input contract represents UTC or includes offsets, not a substitute for knowing the source timezone.

Categorical dtype can save memory and define ordering for repeated labels. Assigning a new category requires adding it or converting the dtype. pandas 3.0 changed default string inference, so avoid code that assumes every text column has dtype `object`; specify intended dtypes and check the [3.0 migration notes](https://pandas.pydata.org/docs/whatsnew/v3.0.0.html).

## Groupby, aggregation, and transform

```python
summary = sales.groupby("region", observed=True, as_index=False).agg(
    orders=("order_id", "size"),
    units=("units", "sum"),
    revenue=("revenue", "sum"),
)
assert summary["orders"].sum() == 4

sales["region_total"] = sales.groupby("region", observed=True)["revenue"].transform("sum")
sales["share"] = sales["revenue"] / sales["region_total"]
np.testing.assert_allclose(sales.groupby("region", observed=True)["share"].sum(), 1)
```

`agg` reduces each group. `transform` returns values aligned to the original rows, making it useful for group totals, ranks, and standardized values. `filter` keeps or removes whole groups. `apply` is flexible but often slower and more version-sensitive than a built-in operation.

`size` counts rows; `count` counts nonmissing entries in a column. Use `dropna=False` when null grouping keys should form a group. Specify `observed` deliberately for categoricals to control unobserved category combinations and avoid relying on changing defaults.

## Joins and concatenation

```python
regions = pd.DataFrame({"region": ["North", "South"], "manager": ["Asha", "Dev"]})
enriched = sales.merge(regions, on="region", how="left", validate="many_to_one", indicator=True)
assert len(enriched) == len(sales)
assert enriched["_merge"].eq("both").all()
```

| Join | Rows retained |
|---|---|
| Inner | Keys matched on both sides |
| Left | All left rows plus matches |
| Right | All right rows plus matches |
| Outer | All keys from both sides |
| Cross | Every left/right pair |

Duplicate keys on both sides can multiply rows. Use `validate="one_to_one"`, `"many_to_one"`, or another appropriate contract. Use `indicator=True` to inspect unmatched records. pandas can match null keys to null keys, unlike typical SQL equality joins; decide whether to exclude those keys first.

`concat([...], ignore_index=True)` stacks compatible rows without matching a business key. `axis=1` combines columns by index alignment. Repeatedly concatenating one row at a time is inefficient; build records or frames and concatenate once.

## Reshaping

```python
wide = pd.DataFrame({"student": ["A", "B"], "math": [8, 9], "python": [9, 10]})
long = wide.melt(id_vars="student", var_name="subject", value_name="score")
assert long.shape == (4, 3)
restored = long.pivot(index="student", columns="subject", values="score")
assert restored.loc["A", "python"] == 9
```

`pivot` requires unique index/column combinations. `pivot_table` aggregates duplicates using the selected `aggfunc` (mean by default), which can hide a data-quality issue if aggregation was not intended. `crosstab` builds frequency tables. `stack` and `unstack` move index levels; `explode` expands list-like values into rows.

For a MultiIndex, think of each row label as a tuple of level values. Name levels, sort when appropriate, and use explicit selectors rather than relying on visually displayed repeated labels.

## Time series and windows

```python
daily = sales.set_index("date")["revenue"].sort_index().resample("D").sum()
moving = daily.rolling(window=2, min_periods=1).mean()
assert len(moving) == len(daily)
previous = daily.shift(1)
assert pd.isna(previous.iloc[0])
```

`resample` groups into time bins; `asfreq` selects/reindexes to a frequency without aggregation. `rolling(7)` means seven observations, while a time-based window such as `rolling("7D")` means a duration on a suitable ordered time index. `expanding` includes all earlier observations; `ewm` applies exponentially decaying weights.

Sort within groups before lag or rolling features. Avoid using future observations in training features. For modern calendar frequencies use documented aliases such as `ME` for month end; consult the current time-series guide when porting older notebooks.

## Copy-on-Write and performance

pandas 3.0 uses Copy-on-Write as its default and only mode: objects derived from others behave independently when modified through pandas operations. Chained assignment cannot update the original frame as intended. Use `df.loc[mask, "column"] = value`. Earlier versions have different defaults; see the [Copy-on-Write guide](https://pandas.pydata.org/docs/user_guide/copy_on_write.html).

`copy(deep=True)` does not recursively clone Python objects stored inside object-dtype cells. A NumPy array returned by `to_numpy` may share storage and can be read-only under Copy-on-Write; request a copy when independent writable storage is needed.

Prefer vectorized operations and built-in aggregations over row-wise `apply` or `iterrows`. If row iteration is necessary, `itertuples` often has lower overhead. Select columns early, use appropriate dtypes, and profile actual bottlenecks. `inplace=True` does not guarantee zero allocation or better performance.

## A small validated pipeline

```python
def revenue_by_region(frame):
    required = {"order_id", "region", "units", "price"}
    if not required.issubset(frame.columns):
        raise ValueError("required columns are missing")
    if frame["order_id"].isna().any() or frame["order_id"].duplicated().any():
        raise ValueError("order_id must be present and unique")
    result = frame.loc[:, ["order_id", "region", "units", "price"]].copy()
    for column in ["units", "price"]:
        result[column] = pd.to_numeric(result[column], errors="raise")
        if result[column].isna().any() or not np.isfinite(result[column]).all():
            raise ValueError(f"{column} must contain finite numbers")
        if (result[column] < 0).any():
            raise ValueError(f"{column} must be nonnegative")
    return (
        result.assign(revenue=lambda data: data["units"] * data["price"])
        .groupby("region", observed=True, dropna=False, as_index=False)
        .agg(revenue=("revenue", "sum"))
        .sort_values("revenue", ascending=False)
        .reset_index(drop=True)
    )


report = revenue_by_region(sales)
assert report["revenue"].sum() == sales["revenue"].sum()
```

This pipeline allows missing regions as their own group and fractional units; change those rules if the domain requires a region and whole-item quantities. Write these decisions into the contract rather than relying on implicit cleaning.

## Testing, pitfalls, and practice

Use `pd.testing.assert_frame_equal` and `assert_series_equal` to verify values, labels, and dtypes. Check row counts before/after joins, uniqueness of expected keys, allowed ranges, and missingness. Sorting before comparison may be appropriate only when row order is not part of the contract.

Avoid label/position confusion, hidden null-key matches, many-to-many join explosions, accidental index columns in CSV exports, mixed timezone assumptions, and statistics fitted before train/test splitting.

Practice: clean a messy CSV while reporting rejected rows; validate a dimension-table join; build a monthly cohort table; reshape repeated measurements; compute lag features without leakage; compare the same pipeline in [Polars](polars.md).

References: [pandas user guide](https://pandas.pydata.org/docs/user_guide/index.html), [API reference](https://pandas.pydata.org/docs/reference/index.html).
