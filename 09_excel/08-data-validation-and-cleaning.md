# 08 · Data Validation, Cleaning, and Quality Checks

A clean-looking sheet can still contain duplicate keys, text numbers, missing records, and inconsistent units. Separate prevention, transformation, and verification.

## Contents

- [Input validation](#input-validation)
- [Cleaning workflow](#cleaning-workflow)
- [Duplicates and missing values](#duplicates-and-missing-values)
- [Reconciliation checks](#reconciliation-checks)
- [Import and export risks](#import-and-export-risks)
- [Practice](#practice)

## Input validation

Data → Data Validation supports whole-number, decimal, list, date/time, text-length, and custom-formula rules. Add an input message explaining the expected value and a useful error message explaining how to fix it.

Examples for the baseline Sales layout:

| Column/range | Rule | Purpose |
|---|---|---|
| I2:I11 | Whole number greater than 0 | Positive quantities |
| J2:K11 | Whole number greater than or equal to 0 | Nonnegative cent values |
| L2:L11 | List of Paid, Pending, Cancelled | Consistent status labels |
| A2:A11 | Custom `=AND(A2<>"",COUNTIF($A$2:$A$11,A2)=1)` | Nonempty unique line IDs within this fixed range |

For the custom rule, set Ignore blank appropriately and expand its range for new rows. A controlled list on a separate sheet, referenced through a defined name, is easier to maintain than repeated hard-coded dropdown strings. Modern spill lists may be usable through a name referring to a spill, but test on the recipient's version.

Validation is a user-input aid, not a database constraint or security boundary. Pasting, imports, and other operations can introduce invalid values. Run separate quality checks on the resulting data.

## Cleaning workflow

1. Preserve an untouched raw copy or reproducible source connection.
2. State table grain, keys, expected columns, units, date locale, and allowed labels.
3. Remove irrelevant header/footer rows and empty records based on explicit rules.
4. Normalize types and whitespace; map categories through a maintained mapping table.
5. Identify duplicate keys, missing required fields, and failed conversions.
6. Resolve or quarantine exceptions rather than silently dropping them.
7. Compare row counts and control totals before and after each material step.
8. Automate repeated steps with [Power Query](11-power-query-and-m.md).

A data dictionary should specify not only type but meaning: OrderDate might mean placed date, shipped date, or paid date. Identical labels do not guarantee identical business meaning across sources.

## Duplicates and missing values

Duplicate complete rows and duplicate business keys are different issues. The same OrderID appearing twice is valid in the fixture because it has two products. SaleID appearing twice needs investigation.

Remove Duplicates is an in-place transformation based on the selected columns. Keep a source copy and determine a winner rule before using it. For a timestamp-based winner, use an explicit sort/ranking/group rule in the transformation pipeline; do not assume an arbitrary retained row is the latest record.

Blank, zero, not applicable, and unknown are distinct. Replacing missing cost with zero overstates profit. Replacing a missing date with today's date invents an event. Track missingness rather than hiding it.

## Reconciliation checks

Use a dedicated Checks sheet or Analysis area with expected/actual/pass columns:

```excel
=ROWS(Sales[SaleID])=10
=COUNTBLANK(Sales[SaleID])=0
=SUM(Sales[RevenueCents])=36000
=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid")=28000
=SUMPRODUCT(--(Sales[Quantity]<=0))=0
```

All should return TRUE for the baseline. Fixed totals test the fixture; a live workbook needs dynamic reconciliations against trusted source totals rather than yesterday's constants.

For each row, `=COUNTIF(Sales[SaleID],[@SaleID])=1` tests uniqueness. `=COUNTIF(Products[ProductID],[@ProductID])=1` tests an exactly-one-match product relationship. Check both orphan keys and duplicate dimension keys.

Create control totals before merging data. If a lookup join unexpectedly increases line count or revenue, inspect right-side duplicate keys. Do not compensate by dividing the total by an observed multiplier.

## Import and export risks

- Automatic date conversion can change codes that resemble dates.
- Numeric inference can remove leading zeros or round long IDs.
- Decimal/thousands separators and delimiters vary by locale.
- CSV has no type metadata, formatting, multiple sheets, or privacy controls.
- Text beginning with formula-significant characters can be interpreted as a formula when opened through some spreadsheet import paths. For untrusted exports, use an explicit text-safe import/export policy and verify the destination behavior; do not assume CSV quoting prevents formula interpretation.
- Hidden sheets and filter settings do not remove underlying data from a shared workbook.

## Practice

On a copy, add a duplicate SaleID, a trailing space in Paid, a text quantity, and an unknown ProductID. Make each issue appear in a visible quality report. Repair the data and verify that row counts and monetary totals return to their baseline values.

Next: [PivotTables](09-pivottables-and-pivotcharts.md).
