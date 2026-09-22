# 12 · Data Model, Power Pivot, and DAX

The Data Model relates tables and evaluates measures across filters. Power Pivot provides modeling tools in supported Excel editions, especially Windows desktop; do not assume the full authoring experience exists on Mac or web. Check your edition and deployment. [Microsoft Power Query/Power Pivot overview](https://support.microsoft.com/en-us/excel/learn-to-use-power-query-and-power-pivot-in-excel)

## Contents

- [Design a star schema](#design-a-star-schema)
- [Load and relate the fixture](#load-and-relate-the-fixture)
- [Columns versus measures](#columns-versus-measures)
- [Core DAX measures](#core-dax-measures)
- [Context and CALCULATE](#context-and-calculate)
- [Calendar and time intelligence](#calendar-and-time-intelligence)
- [Model quality and practice](#model-quality-and-practice)

## Design a star schema

| Object | Grain | Example |
|---|---|---|
| Fact table | One event or measurement | Sales: one order line |
| Product dimension | One product | Products: unique ProductID |
| Date dimension | One calendar day | Calendar: unique Date |
| Budget fact | One month/company target | Budget: one MonthStart |

Dimensions describe facts and filter them through relationships. A one-to-many relationship needs unique, compatible keys on the one side. Repeated ProductID in Sales is expected; repeated ProductID in Products is a modeling problem.

Do not join raw sales lines to monthly budget and then sum the repeated budget values. Different grains require a compatible dimension/filter design, not a lookup that duplicates a target on every transaction.

## Load and relate the fixture

1. Load Sales and Products into the Data Model using the available Power Query or Power Pivot workflow.
2. Ensure ProductID is Text in both tables, with no missing/orphan keys and unique Products keys.
3. Create Products[ProductID] → Sales[ProductID] as a one-to-many relationship.
4. Build a PivotTable from the workbook Data Model.
5. Use category labels from **Products**, not a redundant Sales category column, when practicing dimension filtering.

An unmatched fact key can appear under a blank/unknown grouping depending on the model. Investigate it; hiding the blank category is not a repair.

## Columns versus measures

A calculated column produces one stored value per model row at refresh/recalculation. A measure evaluates when queried under the current filters. Use columns for stable row attributes needed for grouping/relationships; use measures for reusable totals and ratios.

Row context means a current row is available, such as inside an iterator or calculated column. Filter context means the set of rows visible because of slicers, report axes, relationships, and explicit filters. Neither is identical to a worksheet cell reference.

## Core DAX measures

The examples use Power Pivot calculation-area notation `Name := expression`. In a New Measure dialog, put the name in its own field and enter only the expression beginning with `=`. Create measures in this order:

```dax
Revenue Cents := SUMX(Sales, Sales[Quantity] * Sales[UnitPriceCents])

Cost Cents := SUMX(Sales, Sales[Quantity] * Sales[UnitCostCents])

Paid Revenue Cents :=
    CALCULATE([Revenue Cents], KEEPFILTERS(Sales[Status] = "Paid"))

Paid Cost Cents :=
    CALCULATE([Cost Cents], KEEPFILTERS(Sales[Status] = "Paid"))

Paid Profit Cents := [Paid Revenue Cents] - [Paid Cost Cents]

Paid Margin := DIVIDE([Paid Profit Cents], [Paid Revenue Cents])

Paid Orders :=
    CALCULATE(DISTINCTCOUNT(Sales[OrderID]), KEEPFILTERS(Sales[Status] = "Paid"))

Average Paid Order Cents := DIVIDE([Paid Revenue Cents], [Paid Orders])
```

At the unfiltered grand total: paid revenue 28000, paid cost 13400, paid profit 14600, margin approximately 52.142857%, and 6 paid orders. SUMX iterates rows and sums each calculated expression; SUM aggregates one existing column. DIVIDE handles a zero/blank denominator with a blank result by default.

These measures calculate from the raw line columns, so worksheet calculated columns are not prerequisites for their arithmetic. DAX BLANK semantics differ from worksheet empty text and SQL NULL; do not assume every comparison behaves identically.

## Context and CALCULATE

CALCULATE changes filter context. A direct condition on a column generally replaces an existing filter on that column; KEEPFILTERS intersects it instead. The paid measures deliberately use KEEPFILTERS: if the user selects only Pending, they should not secretly show Paid results.

To compute a category's share of paid revenue with the Products dimension:

```dax
Category Share :=
    DIVIDE(
        [Paid Revenue Cents],
        CALCULATE([Paid Revenue Cents], ALL(Products[Category]))
    )
```

With only Products[Category] on rows and no other filters, Books contributes 6000/28000, Learning 12000/28000, Tools 10000/28000. ALL here removes the category filter but preserves other filters. A product-specific slicer or a category field taken from Sales can change the denominator behavior; test the exact report layout.

Grand totals recalculate a measure in total context; they do not necessarily add visible row values. This is correct for ratios and distinct counts. A customer who appears in two product categories must not be counted twice in a distinct customer grand total.

CALCULATE also performs context transition from row context where applicable. Learn this before writing nested iterators or calculated columns containing aggregate expressions. Prefer a simple explicit model over a deeply nested measure that hides its population.

## Calendar and time intelligence

Create a full, gap-free Calendar table using a separate Power Query blank query:

```powerquery
let
    Dates = List.Dates(#date(2026, 1, 1), 365, #duration(1, 0, 0, 0)),
    AsTable = Table.FromList(Dates, Splitter.SplitByNothing(), {"Date"}),
    Typed = Table.TransformColumnTypes(AsTable, {{"Date", type date}}),
    Year = Table.AddColumn(Typed, "Year", each Date.Year([Date]), Int64.Type),
    Month = Table.AddColumn(Year, "MonthStart", each Date.StartOfMonth([Date]), type date)
in
    Month
```

Name it Calendar, load to the model, mark it as a Date Table using Date where supported, and relate Calendar[Date] to Sales[OrderDate]. Extend it to all complete years needed for the model. Date keys must match at day grain; timestamps with nonmidnight times need a separate date key.

```dax
Paid Revenue YTD :=
    TOTALYTD([Paid Revenue Cents], 'Calendar'[Date])
```

Put Calendar[MonthStart] on report rows. January YTD is 12000; February YTD is 28000. A model containing only 2026 cannot provide a meaningful prior-year comparison without 2025 data and calendar coverage.

For Budget, introduce a unique month dimension related to both Sales[MonthStart] and Budget[MonthStart], or design another explicit monthly comparison strategy. A repeated Calendar[MonthStart] column cannot serve as the unique side of that relationship. Avoid ambiguous duplicate filter paths.

## Model quality and practice

Remove unused high-cardinality text columns from the model when appropriate, use efficient types, document measures, and hide technical keys from ordinary report users. File size and memory can become limiting before theoretical row capacity.

Reconcile measures against worksheet formulas, inspect blanks/orphans, and test filters individually and in combination. A result that matches only the grand total may still be wrong by region or product.

Practice: build paid revenue by Products[Category], compare distinct orders with line counts, test a Pending-only filter, and add Calendar month rows for YTD.

Reference: [DAX in Power Pivot](https://support.microsoft.com/en-us/excel/data-analysis-expressions-dax-in-power-pivot). Next: [What-if analysis and statistics](13-what-if-statistics-and-financial-functions.md).
