# 07 · Dynamic Arrays, LET, and LAMBDA

This chapter targets modern Excel. Check each function's availability: support for FILTER does not imply that every newer array helper exists in your version.

## Contents

- [Spill behavior](#spill-behavior)
- [Filter, sort, and unique](#filter-sort-and-unique)
- [Array construction and reshaping](#array-construction-and-reshaping)
- [LET](#let)
- [LAMBDA and helper functions](#lambda-and-helper-functions)
- [Empty inputs and performance](#empty-inputs-and-performance)
- [Practice](#practice)

## Spill behavior

A dynamic formula can return multiple values into adjacent cells. Enter the formula in one anchor cell and leave its output area empty. The `#` operator refers to the complete spill, such as `A2#`.

Spill formulas belong **outside Excel Tables**; they can refer to Table columns. Only the anchor cell is editable. Occupied cells, merged cells, or insufficient worksheet space can cause #SPILL!. Cross-workbook dynamic-array links have limitations, including when the source workbook is closed. [Microsoft spill behavior](https://support.microsoft.com/en-us/excel/dynamic-array-formulas-and-spilled-array-behavior)

The `@` implicit-intersection operator requests a single value in contexts where a range/array could otherwise produce several. In `[@Quantity]`, @ also identifies the current Table row. Do not remove it without understanding whether a row value or an array is intended.

## Filter, sort, and unique

Run each formula in its own empty area on Analysis:

```excel
=SORT(UNIQUE(Sales[Region]))
=FILTER(Sales[[SaleID]:[Status]],Sales[Status]="Paid","No paid rows")
=SORTBY(Sales[[SaleID]:[Status]],Sales[OrderDate],1,Sales[SaleID],1)
=UNIQUE(Sales[OrderID])
```

The first produces North, South, West. UNIQUE returns distinct values by default; its optional exactly_once argument instead returns values appearing only once. In this fixture `=UNIQUE(Sales[OrderID],FALSE,TRUE)` excludes orders 101 and 105 because they have multiple lines.

FILTER accepts a Boolean include array. Multiplication combines AND conditions; addition can represent OR. When adding conditions, compare the sum to `>0` if you need a clean Boolean expression. Errors in include propagate and should be diagnosed.

## Array construction and reshaping

```excel
=SEQUENCE(5)
=SEQUENCE(3,2,10,10)
=TRANSPOSE({1,2,3})
```

The second generates rows `(10,20)`, `(30,40)`, `(50,60)`. Array-constant separators are locale-dependent; the shown braces use English Excel conventions.

Newer array functions include TAKE/DROP, CHOOSECOLS/CHOOSEROWS, HSTACK/VSTACK, TOCOL/TOROW, WRAPROWS/WRAPCOLS, and EXPAND. Verify availability before designing a shared workbook around them. [Microsoft array-function catalog](https://support.microsoft.com/en-us/excel/lookup-and-reference-functions-reference)

For versions with TAKE:

```excel
=TAKE(SORTBY(Sales[[SaleID]:[Status]],Sales[RevenueCents],-1,Sales[SaleID],1),3)
```

Returns lines S004, S005, S007: the three highest line revenues across **all statuses**, with SaleID breaking ties. Filter Paid first if that is the actual requirement. A top-three-lines query is not top-three-orders.

RAND/RANDBETWEEN/RANDARRAY recalculate and are useful for simulations. Their generated values are not a reproducible sample unless you deliberately capture/freeze a draw or generate it in a seeded external process.

## LET

LET names intermediate expressions within a formula. This improves readability and can avoid repeating an expensive expression.

```excel
=LET(paid,Sales[Status]="Paid",revenue,SUM(FILTER(Sales[RevenueCents],paid,0)),cost,SUM(FILTER(Sales[CostCents],paid,0)),IF(revenue=0,"",(revenue-cost)/revenue))
```

Result: about 52.142857%. This formula explicitly chooses a blank display when paid revenue is zero. Use valid names such as revenue rather than names that collide with cell references or R1C1 conventions.

## LAMBDA and helper functions

A LAMBDA packages a formula as a reusable function. Test it inline first:

```excel
=LAMBDA(revenue,cost,IF(revenue=0,"",(revenue-cost)/revenue))(28000,13400)
```

Then create a workbook name `GrossMargin` in Name Manager whose Refers To formula is:

```excel
=LAMBDA(revenue,cost,IF(revenue=0,"",(revenue-cost)/revenue))
```

Call `=GrossMargin(28000,13400)`. The definition above belongs in Name Manager; entering an uncalled LAMBDA directly in a cell yields #CALC!. LAMBDA is supported in Microsoft 365 and Excel 2024 according to its current documentation, rather than every older perpetual edition. [Microsoft LAMBDA](https://support.microsoft.com/en-us/excel/functions/lambda-function)

Where supported, helpers apply a LAMBDA over arrays:

```excel
=MAP({1,2,3},LAMBDA(value,value^2))
=REDUCE(0,{1,2,3},LAMBDA(total,value,total+value))
=SCAN(0,{1,2,3},LAMBDA(total,value,total+value))
```

Results: `{1,4,9}`, 6, and `{1,3,6}`. REDUCE returns the final accumulator; SCAN returns each intermediate accumulator. BYROW/BYCOL can summarize rows/columns, generally with one result per lambda call. Nested arrays are not a universal supported return structure.

Recursive LAMBDA calls need a termination condition and can hit calculation/recursion limits. Prefer a direct formula, helper function, or transformation pipeline when it makes the logic clearer.

## Empty inputs and performance

FILTER's if_empty argument specifies a fallback value, not a genuine zero-row relational table. Counting a fallback string as if it were a record can produce wrong results. State the empty-case result before wrapping an array expression in ROWS or COUNTA.

Avoid arrays across entire million-row columns when only a bounded Table is needed. Reuse calculations with LET or helper columns, and reserve enough spill space. Test zero records, one record, duplicate keys, ties, and a newly appended row.

## Practice

Build a sorted region list, a paid-only extract, and a reusable margin function. Add a blocking value to a spill area and diagnose #SPILL!, then remove the obstruction. Compare UNIQUE's distinct-values mode with exactly-once mode on OrderID.

Next: [Validation and cleaning](08-data-validation-and-cleaning.md).
