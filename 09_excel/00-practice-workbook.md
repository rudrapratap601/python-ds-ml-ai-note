# 00 · Shared Practice Workbook

These notes use one small sales dataset. Build it once, save a baseline copy, and use a separate copy for exercises that change data. No real customer information is needed.

## Contents

- [Workbook layout](#workbook-layout)
- [Sales data](#sales-data)
- [Calculated columns](#calculated-columns)
- [Supporting tables](#supporting-tables)
- [Expected answers](#expected-answers)
- [Data dictionary and assumptions](#data-dictionary-and-assumptions)

## Workbook layout

Create a workbook named `excel-practice.xlsx` with sheets `Readme`, `Sales`, `Products`, `Budget`, `Analysis`, and `Dashboard`. Put the dataset description and refresh instructions on Readme. Keep the spelling of table and column names below: later formulas use these exact names.

Examples use English function names and commas as argument separators. A regional Excel installation may use semicolons or translated function names. Dates must be actual Excel dates, not text that merely looks like a date.

## Sales data

Copy the following CSV into a temporary UTF-8 text file named `sales-practice.csv`. In Excel use **Data → From Text/CSV**, choose a comma delimiter, and select Transform Data. Set SaleID/ProductID to Text; OrderID, Quantity, UnitPriceCents, and UnitCostCents to Whole Number; and OrderDate to Date. Confirm the dates represent January and February 2026. Load to `Sales!A1` as a Table, then use **Table Design → Table Name** to name it `Sales`.

If your Excel lacks that import interface, paste the records and use Text to Columns with a comma delimiter, selecting YMD for the date column. Verify the import rather than trusting automatic type inference.

```csv
SaleID,OrderID,OrderDate,Region,Customer,ProductID,Product,Category,Quantity,UnitPriceCents,UnitCostCents,Status
S001,101,2026-01-02,North,Asha,P10,Notebook,Books,2,1000,600,Paid
S002,101,2026-01-02,North,Asha,P20,Python course,Learning,1,3000,1200,Paid
S003,102,2026-01-05,North,Asha,P10,Notebook,Books,1,1000,600,Paid
S004,103,2026-01-03,South,Dev,P20,Python course,Learning,2,3000,1200,Paid
S005,104,2026-01-05,North,Mira,P30,Data toolkit,Tools,1,5000,2500,Cancelled
S006,105,2026-02-01,South,Dev,P10,Notebook,Books,3,1000,600,Paid
S007,105,2026-02-01,South,Dev,P30,Data toolkit,Tools,1,5000,2500,Paid
S008,106,2026-02-03,West,Noor,P20,Python course,Learning,1,3000,1200,Pending
S009,107,2026-02-05,North,Mira,P30,Data toolkit,Tools,1,5000,2500,Paid
S010,108,2026-02-05,North,Asha,P20,Python course,Learning,1,3000,1200,Paid
```

There are 10 data rows: the base data occupies A1:L11 including headers. One row represents **one order line**, not one order. Order 101 and order 105 each have two lines. Do not delete these legitimate repetitions.

## Calculated columns

After loading the raw table, add these headers in M1:Q1. Enter each formula in its first data cell and let the Table fill the calculated column. These columns belong to this learning workbook; a query refresh that changes the output schema can require checking their placement and formulas.

| Column | Header | Formula in first data row |
|---|---|---|
| M | RevenueCents | `=[@Quantity]*[@UnitPriceCents]` |
| N | CostCents | `=[@Quantity]*[@UnitCostCents]` |
| O | ProfitCents | `=[@RevenueCents]-[@CostCents]` |
| P | PaidRevenueCents | `=IF([@Status]="Paid",[@RevenueCents],0)` |
| Q | MonthStart | `=DATE(YEAR([@OrderDate]),MONTH([@OrderDate]),1)` |

Format MonthStart as `mmm yyyy`; it still stores a date. An input formatted as Text may display a formula literally: change the format to General and re-enter it.

On Analysis, test `=ISNUMBER(Sales!C2)`; it should return TRUE. Test `=ROWS(Sales[SaleID])`; it should return 10. Use a blank Analysis area for formulas that spill across several cells.

## Supporting tables

Create this table on Products and name it `Products`. ProductID is a unique text key. Products can exist without sales.

```csv
ProductID,Product,Category,ListPriceCents,UnitCostCents
P10,Notebook,Books,1000,600
P20,Python course,Learning,3000,1200
P30,Data toolkit,Tools,5000,2500
P40,SQL workbook,Books,2000,1000
```

Create this table on Budget and name it `Budget`. Enter MonthStart using `=DATE(2026,1,1)` and `=DATE(2026,2,1)` if date parsing is uncertain. These are calendar-month targets for **paid revenue**.

| MonthStart | TargetCents |
|---|---:|
| 2026-01-01 | 10000 |
| 2026-02-01 | 15000 |

Optional lookup practice: create a `Tiers` table on Analysis with numeric LowerBound and Discount columns, sorted by LowerBound ascending.

| LowerBound | Discount |
|---:|---:|
| 0 | 0% |
| 5000 | 5% |
| 10000 | 10% |

## Expected answers

| Check | Expected |
|---|---:|
| Sales lines, all statuses | 10 |
| Distinct orders, all statuses | 8 |
| Paid lines | 8 |
| Distinct paid orders | 6 |
| Quantity, all statuses | 14 |
| Paid quantity | 12 |
| RevenueCents, all statuses | 36000 |
| Paid revenue cents | 28000 |
| Paid cost cents | 13400 |
| Paid profit cents | 14600 |
| Paid gross margin | 14600 / 28000 ≈ 52.142857% |
| Average paid order cents | 28000 / 6 ≈ 4666.666667 |
| January paid revenue cents | 12000 |
| February paid revenue cents | 16000 |
| North / South paid revenue cents | 14000 / 14000 |
| Paid Books / Learning / Tools revenue cents | 6000 / 12000 / 10000 |
| Total target cents | 25000 |
| Total actual minus target cents | 3000 |

These answers deliberately distinguish line count from order count, paid amounts from all-status amounts, and total margin from the average of line margins.

## Data dictionary and assumptions

- SaleID uniquely identifies a line. OrderID groups lines belonging to an order.
- Prices and costs are integer cents in one fictional currency. Divide by 100 for currency-unit presentation; formatting alone does not convert cents to units.
- UnitPriceCents and UnitCostCents are historical line values. A current catalog price must not overwrite historical transactions.
- Paid revenue excludes Pending and Cancelled rows. Refunds, tax, shipping, exchange rates, and payment timing are outside this fixture.
- Customer names are teaching labels. Real systems should use stable customer IDs.
- Status is the current status, not a history. This dataset is insufficient for leakage-safe historical ML features without additional timestamps/versioned data.

Next: [Workbook fundamentals](01-workbook-fundamentals.md).
