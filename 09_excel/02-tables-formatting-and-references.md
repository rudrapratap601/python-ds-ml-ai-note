# 02 · Tables, Formatting, and References

Good structure makes formulas easier to read and harder to break. Use the [Sales fixture](00-practice-workbook.md) throughout this chapter.

## Contents

- [Excel Tables](#excel-tables)
- [Relative and absolute references](#relative-and-absolute-references)
- [Structured references](#structured-references)
- [Names and assumptions](#names-and-assumptions)
- [Formatting and conditional formatting](#formatting-and-conditional-formatting)
- [Sorting and filtering](#sorting-and-filtering)
- [Practice](#practice)

## Excel Tables

Select a rectangular dataset and use Insert → Table (Ctrl+T on Windows). Confirm that it has headers. Give the table a descriptive name, such as Sales, rather than retaining Table1.

Tables provide filter buttons, calculated columns, expandable references, optional totals, and a stable source for PivotTables. Formatting a range with alternating colors alone does not make it a Table.

Append records immediately below the Table and verify it expands. A fixed range such as A2:A11 does not automatically express “all future sales”; `Sales[SaleID]` does. A newly appended row can still be excluded from an old PivotTable result until its refresh completes.

## Relative and absolute references

| Reference | What changes when copied? | Use |
|---|---|---|
| A2 | Row and column | Same calculation for nearby records |
| $A$2 | Neither | Fixed assumption/input |
| $A2 | Row only | Always use column A while moving down |
| A$2 | Column only | Always use row 2 while moving across |

Suppose B2 is quantity, C2 unit price, and F1 a discount rate. `=B2*C2*(1-$F$1)` can be filled downward: record references move, while the discount stays fixed. F4 cycles reference locking while editing on Windows; keyboards may require Fn.

For a multiplication grid with column labels in B1:D1 and row labels in A2:A4, enter `=$A2*B$1` in B2 and fill across/down. Mixed references solve a different problem from making every reference absolute.

## Structured references

| Expression | Meaning |
|---|---|
| `Sales[Quantity]` | All data values in Quantity |
| `[@Quantity]` | Quantity on the current Table row |
| `Sales[[#Headers],[Quantity]]` | Quantity header cell |
| `Sales[#Data]` | Table data without headers/totals |
| `Sales[#All]` | Headers, data, and totals if enabled |

```excel
=[@Quantity]*[@UnitPriceCents]
=SUM(Sales[RevenueCents])
=SUBTOTAL(109,Sales[RevenueCents])
```

The first formula belongs in a Sales calculated column. The others can go on Analysis. SUM includes filtered-out rows; SUBTOTAL with 109 sums visible rows while excluding both filtered-out and manually hidden rows. Codes 1–11 and 101–111 differ in how manually hidden rows are handled. Neither function implies that only paid sales should count unless you apply that filter or use the appropriate column.

Use the Table Total Row when helpful; inspect its chosen calculation. It is a presentation summary, not another transaction record.

## Names and assumptions

Name a single input cell `TaxRate` or `ReportDate` through the Name Box or Name Manager. Names can refer to cells, ranges, constants, or formulas and may have workbook or worksheet scope. Name conflicts can resolve differently depending on the active sheet.

Avoid names that look like addresses, contain spaces, or collide with special reference syntax. Document units: TaxRate should contain 0.18 formatted as 18%, not 18. Names improve readability only when their definition is clear.

## Formatting and conditional formatting

| Format | Use |
|---|---|
| `0` | Whole numbers |
| `0.00` | Display two decimal places |
| `#,##0` | Grouped large integers |
| `0.0%` | Display a fraction as a percent |
| `yyyy-mm-dd` | Unambiguous date presentation |
| `mmm yyyy` | Month label while retaining the date |
| `[h]:mm` | Elapsed hours beyond 24 hours |

Formats change presentation, not units or stored types. A 28000-cent total must be divided by 100 to show 280 currency units; adding a currency symbol to 28000 does not do that conversion.

Conditional formatting can highlight thresholds, duplicate keys, negative values, missing inputs, or distribution with data bars. Prefer rules with a clear business meaning; colors should supplement text, not be the only signal.

To highlight entire paid rows in the baseline range A2:Q11, create a formula rule `=$L2="Paid"` applied to `$A$2:$Q$11`. The fixed column and relative row are deliberate. To flag duplicate SaleIDs, use `=COUNTIF($A$2:$A$11,$A2)>1` on the baseline range. Expand the rule's scope when new records are added.

Review rule order, Stop If True, and Applies To ranges. Copy/paste can fragment a simple rule into many overlapping rules. Excessive full-sheet formatting can increase file size and editing cost.

## Sorting and filtering

Sort the **entire table**, not one isolated column, to preserve row relationships. Use multiple keys for deterministic sorting: OrderDate followed by SaleID. Text numbers sort differently from real numbers.

Filtering hides rows; it does not remove them. Reconcile filtered totals with a clearly defined metric. Slicers provide visible filter controls for supported Tables/PivotTables, but each slicer must be connected to the intended report.

## Practice

Add an input named DiscountRate and a test formula referencing it. Explain the difference between `Sales[Quantity]` and `[@Quantity]`. Filter Status to Paid and verify the visible revenue total is 28000, then clear the filter and verify 36000.

Next: [Formula foundations](03-formula-foundations-and-errors.md).
