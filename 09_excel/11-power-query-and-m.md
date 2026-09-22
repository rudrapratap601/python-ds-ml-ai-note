# 11 · Power Query and the M Language

Power Query records repeatable import and transformation steps. Use it for recurring files, cleanup, joins, reshaping, and summaries instead of repeating manual edits each month. Connector/editor/refresh support differs across Windows, Mac, web, license, and build. [Microsoft platform overview](https://support.microsoft.com/en-us/excel/about-power-query-in-excel)

## Contents

- [Workflow and load choices](#workflow-and-load-choices)
- [Core transformations](#core-transformations)
- [A complete M example](#a-complete-m-example)
- [Append and merge](#append-and-merge)
- [Reshaping and reusable functions](#reshaping-and-reusable-functions)
- [Performance and refresh](#performance-and-refresh)
- [Practice](#practice)

## Workflow and load choices

1. Choose Data → Get Data or From Table/Range.
2. Inspect headers and source types; preserve stable identifiers as Text.
3. Apply named transformations and review their effect on rows and totals.
4. Load to a worksheet Table, create only a connection, or load to the Data Model where supported.
5. Refresh and verify the output against source controls.

Worksheet load is useful for visible detail; connection-only staging avoids unnecessary copies; the Data Model supports relational analysis without putting every record on a sheet. Worksheet row limits still apply to worksheet outputs.

Do not manually edit a query's output as the permanent correction: refresh can overwrite it. Put corrections in the source, a separate mapping/input table, or an explicit transformation step.

## Core transformations

| Operation | Use | Check |
|---|---|---|
| Choose/remove columns | Reduce unnecessary data | Retain required keys and lineage |
| Change type with locale | Parse dates and numbers | Rejects, mixed formats, leading zeros |
| Filter rows | Define scope | Do not silently discard errors or late records |
| Trim/clean/replace | Normalize text | Preserve meaningful distinctions |
| Split/extract | Parse compound fields | Missing/extra delimiters |
| Group By | Aggregate to a declared grain | Key columns and numerator/denominator |
| Unpivot | Convert repeated month columns into rows | Preserve identifier columns |
| Pivot | Turn categories into columns | Multiple records per cell need aggregation |
| Remove duplicates | Enforce a chosen uniqueness rule | Which duplicate survives? |

Applied Steps is an executable transformation sequence. Renaming or removing a referenced column can break later steps. Data previews may profile only a sample; configure whole-dataset profiling when supported and needed.

## A complete M example

Prerequisite: the Sales Table exists and its OrderDate values are real dates. Create a **new blank query**, open Advanced Editor, and replace its code with the following. Name the query `PaidMonthly` and load its result to a separate sheet/table, never back into the input Sales Table.

```powerquery
let
    Source = Excel.CurrentWorkbook(){[Name="Sales"]}[Content],
    Selected = Table.SelectColumns(Source,
        {"SaleID", "OrderDate", "Status", "Quantity", "UnitPriceCents"}),
    Typed = Table.TransformColumnTypes(Selected,
        {{"SaleID", type text}, {"OrderDate", type date},
         {"Status", type text}, {"Quantity", Int64.Type},
         {"UnitPriceCents", Int64.Type}}),
    CleanStatus = Table.TransformColumns(Typed,
        {{"Status", each if _ = null then null else Text.Trim(_), type text}}),
    Paid = Table.SelectRows(CleanStatus, each [Status] = "Paid"),
    Revenue = Table.AddColumn(Paid, "RevenueCents",
        each [Quantity] * [UnitPriceCents], Int64.Type),
    Month = Table.AddColumn(Revenue, "MonthStart",
        each Date.StartOfMonth([OrderDate]), type date),
    Grouped = Table.Group(Month, {"MonthStart"},
        {{"PaidRevenueCents", each List.Sum([RevenueCents]), Int64.Type}}),
    Sorted = Table.Sort(Grouped, {{"MonthStart", Order.Ascending}})
in
    Sorted
```

Expected: 2026-01-01 → 12000; 2026-02-01 → 16000. Selecting only the raw input columns avoids colliding with the worksheet's existing RevenueCents/MonthStart calculated columns.

M is case-sensitive. `let` binds step names and `in` selects the result. Square brackets access fields, curly braces index list/table items, and `each` is shorthand for a one-argument function. M `null` differs from text containing the word null. M is distinct from worksheet formulas and DAX.

## Append and merge

**Append** stacks records with aligned column names, for example monthly files with the same schema. Mismatched columns can create nulls; schema drift needs a policy. Keep source filename and ingest date for traceability.

**Merge** joins tables using selected keys. Left outer preserves left records, inner keeps matches, and left anti exposes missing matches. Set key types consistently and verify uniqueness on the lookup side before expanding nested matches.

If every Sales product should match one Products row, a left merge and expansion should still produce 10 lines and 36000 total revenue. An increased row count indicates duplicate matches; lost rows may indicate an unintended inner join or filter.

Folder combine workflows infer a sample-file transformation. Filter the folder to the intended files first, excluding temporary files and the output workbook itself. Check column names, encoding, sheet/table names, and schema changes across every file.

## Reshaping and reusable functions

A wide budget with Jan, Feb, Mar columns is often easier to analyze after unpivoting into Month/Target rows. Preserve the business identifiers, then convert the unpivoted month into a proper date using explicit year information.

Standalone M function example, entered as a separate blank query:

```powerquery
(value as nullable text) as nullable text =>
    if value = null then null else Text.Upper(Text.Trim(value))
```

Name it `NormalizeCode`, then invoke it in another query's custom column. Do not uppercase case-sensitive IDs without a valid business rule.

Use `try ... otherwise` only for an intentional error policy. Replacing every conversion error with zero conceals bad data. Prefer an exception table that records original value, error reason, and source.

## Performance and refresh

Query folding pushes supported transformations to the source, often a database. It depends on connector and step support; a workbook Table source does not become a SQL query simply because the transformations look relational. Filter/select early where appropriate, avoid unnecessary expensive operations, and inspect available native-query/diagnostic tools. [Microsoft folding concepts](https://learn.microsoft.com/en-us/power-query/query-folding-basics)

Table.Buffer is not a universal acceleration switch: it can consume memory and prevent downstream folding. Measure instead of adding it to every query.

Credentials, privacy levels, source paths, refresh permissions, and background dependencies affect deployment. Do not disable privacy controls simply to make an unexplained combination work. Refreshing in Excel, the web, or an automated flow may support different connectors and authentication paths.

## Practice

Run PaidMonthly and reconcile it with SUMIFS. Merge Sales with Products and prove row count is unchanged. Append a second synthetic month with the same schema, then deliberately rename a source column and document the failure/recovery behavior.

Next: [Data Model and DAX](12-data-model-power-pivot-and-dax.md).
