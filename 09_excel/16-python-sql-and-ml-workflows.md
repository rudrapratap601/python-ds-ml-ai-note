# 16 · Excel with Python, SQL, and Machine Learning

Excel is useful for inspection, business inputs, and understandable reports. Python and SQL can provide repeatable transformations, larger-scale processing, and model evaluation. Define the boundary between them instead of copying data manually through many disconnected files.

## Contents

- [Choose an integration route](#choose-an-integration-route)
- [A pandas import and report](#a-pandas-import-and-report)
- [Working with formulas and file libraries](#working-with-formulas-and-file-libraries)
- [Python in Excel](#python-in-excel)
- [SQL and leakage-safe ML](#sql-and-leakage-safe-ml)
- [Practice](#practice)

## Choose an integration route

| Route | Useful for | Boundary |
|---|---|---|
| pandas read/write Excel | Analysis and new reports outside Excel | Not a full workbook-feature preservation tool |
| openpyxl | Editing supported .xlsx content | Does not calculate Excel formulas |
| XlsxWriter | Creating new reports with formatting/charts | Does not edit an existing workbook or calculate formulas |
| Python in Excel | Python analysis within supported Excel | Managed cloud runtime and curated environment |
| Power Query database connector | Refreshable SQL-backed reporting | Connector credentials, permissions, folding, refresh support |

Use the existing [pandas notes](../04_data_science/pandas.md), [NumPy notes](../04_data_science/numpy.md), and [SQL section](../08_sql/README.md) for deeper analysis topics.

## A pandas import and report

Prerequisites: Python, pandas, and openpyxl in your environment; a saved copy of the practice workbook with the Sales Table starting at A1 on the Sales sheet, no title rows, and Total Row disabled. This example reads raw columns and calculates its own revenue instead of depending on cached worksheet formula results. It writes a **new** `paid-summary.xlsx` in the working directory and refuses to overwrite an existing one.

```python
from pathlib import Path
import pandas as pd

source = Path("excel-practice.xlsx")
destination = Path("paid-summary.xlsx")
if destination.exists():
    raise FileExistsError(f"Choose a new output path: {destination}")

columns = ["SaleID", "OrderID", "OrderDate", "Region", "ProductID",
           "Quantity", "UnitPriceCents", "UnitCostCents", "Status"]
sales = pd.read_excel(
    source, sheet_name="Sales", usecols=columns,
    dtype={"SaleID": "string", "ProductID": "string", "Status": "string"},
    engine="openpyxl",
)
if sales[columns].isna().any().any():
    raise ValueError("Required practice fields contain missing values")
if sales["SaleID"].duplicated().any():
    raise ValueError("Duplicate SaleID")

sales["OrderDate"] = pd.to_datetime(sales["OrderDate"], errors="raise")
for name in ["OrderID", "Quantity", "UnitPriceCents", "UnitCostCents"]:
    sales[name] = pd.to_numeric(sales[name], errors="raise")
    if (sales[name] % 1 != 0).any():
        raise ValueError(f"Expected integers: {name}")
if (sales["Quantity"] <= 0).any():
    raise ValueError("Quantity must be positive")
if (sales[["UnitPriceCents", "UnitCostCents"]] < 0).any().any():
    raise ValueError("Price/cost cannot be negative")
if not sales["Status"].isin(["Paid", "Pending", "Cancelled"]).all():
    raise ValueError("Unexpected status")

paid = sales.loc[sales["Status"].eq("Paid")].copy()
paid["RevenueCents"] = paid["Quantity"] * paid["UnitPriceCents"]
paid["MonthStart"] = paid["OrderDate"].dt.to_period("M").dt.to_timestamp()
summary = paid.groupby("MonthStart", as_index=False)["RevenueCents"].sum()

assert paid["OrderID"].nunique() == 6       # Fixture-specific checks
assert paid["RevenueCents"].sum() == 28000
with pd.ExcelWriter(destination, engine="openpyxl") as writer:
    summary.to_excel(writer, sheet_name="Monthly", index=False)
```

Those fixed assertions validate the teaching fixture. A live pipeline should compare against source control totals and declared invariants, not permanently require six orders.

## Working with formulas and file libraries

File-writing libraries can store formula strings without calculating their results. A cached value can be missing or stale until Excel or another compatible engine recalculates it. Reading with openpyxl `data_only=True` returns cached results, not a newly evaluated formula.

Round-tripping a workbook through pandas or a generic file library can lose unsupported charts, macros, connections, or model features. Write a new report unless you have deliberately tested preservation for the features in the original. Do not convert an .xlsm file to .xlsx and expect macros to survive.

Specify sheet names, header position, columns, types, and date parsing. `read_excel` reads worksheet cells, not an abstract Excel Table object: notes or totals placed in the same sheet can become records if the range is not controlled. [pandas read_excel](https://pandas.pydata.org/docs/reference/api/pandas.read_excel.html), [openpyxl usage](https://openpyxl.readthedocs.io/en/stable/usage.html)

## Python in Excel

Python in Excel is a separate integration from running a local .py script. It depends on a qualifying Microsoft 365 license, platform, update channel, and organizational settings. Consult [current availability](https://support.microsoft.com/en-us/excel/python/python-in-excel-availability) rather than assuming a perpetual Excel purchase includes it.

In a Python cell, a typical starting point is:

```python
sales = xl("Sales[#All]", headers=True)
paid = sales.loc[sales["Status"].eq("Paid")].copy()
paid["RevenueCents"] = paid["Quantity"] * paid["UnitPriceCents"]
paid.groupby("Region", as_index=False)["RevenueCents"].sum()
```

Keep the Sales Total Row off when using `[#All]` this way. The `xl()` helper belongs to Python in Excel; it is not available in an ordinary local Python session. Choose the appropriate output type in Excel for a Python object versus returned grid values.

Python in Excel executes in a Microsoft cloud container with a curated environment. Its Python code does not have ordinary access to your local filesystem or network, and it is not a VBA replacement for editing workbook objects. [Microsoft runtime/security details](https://support.microsoft.com/en-us/excel/python/data-security-and-python-in-excel)

## SQL and leakage-safe ML

For recurring relational data, query only the necessary columns and rows through a supported connector or parameterized database client. Credentials belong in supported credential storage, not cells, connection strings committed to Git, or Python source.

For model training, define entity keys, prediction timestamp, target horizon, feature availability, and evaluation split. A spreadsheet containing the latest status can leak future knowledge into a historical training row. The practice dataset stores current Status and cannot alone establish what was known at each past cutoff.

Fit preprocessing using training data only. Keep a data dictionary, controlled missing-value policy, fixed source snapshot, and versioned transformation code. Export probabilities/predictions alongside stable row keys so results can be joined back correctly; never rely solely on current row order.

Excel can review a model's predictions or accept scenario inputs, but moving formulas between train/test sheets does not itself prevent leakage. Use the [SQL analytics notes](../08_sql/10-analytics-and-ml-patterns.md) for point-in-time examples.

## Practice

Reconcile the pandas summary with SUMIFS and Power Query. Change row order and verify key-based results remain valid. Explain why a pandas export can be appropriate for a new report while being inappropriate for preserving a complex original workbook.

Next: [Performance, auditing, and protection](17-performance-auditing-and-protection.md).
