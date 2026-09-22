# 09 · PivotTables, PivotCharts, and Slicers

PivotTables summarize a table without writing a separate formula for every group. They are especially useful for exploring categorical patterns and checking formula-based reports.

## Contents

- [Build the first PivotTable](#build-the-first-pivottable)
- [Aggregation and grain](#aggregation-and-grain)
- [Dates and percentages](#dates-and-percentages)
- [Slicers and PivotCharts](#slicers-and-pivotcharts)
- [Refresh and report stability](#refresh-and-report-stability)
- [Practice](#practice)

## Build the first PivotTable

1. Click inside the Sales Table and choose Insert → PivotTable.
2. Confirm the source is `Sales`, then place the report on a new worksheet.
3. Put Region in Rows and RevenueCents in Values.
4. Put Status in Filters and select Paid.
5. Open Value Field Settings and confirm **Sum**, then apply an integer number format.

Expected: North 14000, South 14000, total 28000. West has no paid sales and may be absent unless you configure empty-item display. A missing row is not automatically a numerical zero.

Rows define vertical categories; Columns define horizontal categories; Values define calculations; Filters define the included population. Move Category into Columns for a region-by-category cross-tab, then reconcile its grand total.

## Aggregation and grain

| Value setting | Meaning on this fixture |
|---|---|
| Sum of RevenueCents | Total line revenue in the filter context |
| Count of SaleID | Number of nonempty sales lines |
| Count of OrderID | Also a line count, because OrderID repeats |
| Average of RevenueCents | Average line revenue |
| Distinct Count of OrderID | Unique orders, using a Data Model-backed PivotTable where available |

For paid sales: Count of OrderID is 8; distinct paid order count is 6. Distinct Count requires the appropriate Data Model workflow, not just renaming a Count field. If unavailable, deduplicate at order grain in Power Query first.

If Excel defaults to Count for a field you expected to sum, inspect its types, blanks, and errors. Changing the label to Revenue does not change the aggregation.

Classic PivotTable calculated fields have special aggregation semantics and limitations. For robust reusable ratios such as total profit/total revenue, use a carefully verified external formula or a [DAX measure](12-data-model-power-pivot-and-dax.md). Do not average percentages merely because the field contains percentages.

## Dates and percentages

Put MonthStart in Rows and Sum of RevenueCents in Values, keeping Paid selected. Expected January 12000 and February 16000. Real date fields can be grouped by years/months, but text dates and errors can block grouping. Including years avoids combining January across different years.

Show Values As can express percent of grand total, percent of row/column total, running total, or difference from a chosen base item. Read the denominator and base-item settings; a percentage is meaningful only relative to its defined population.

For North/South paid revenue, each is 50% of the paid grand total. If a slicer reduces the population, that grand total can change. A percentage of the current selection differs from a percentage of the unfiltered company total.

## Slicers and PivotCharts

Insert a slicer for Region or Category and a timeline for a supported date field. Use Report Connections to attach a slicer to the intended compatible PivotTables. Similar-looking slicers do not automatically control every chart.

A PivotChart inherits PivotTable filters and aggregation. Choose a bar/column chart for categories or a line chart for ordered periods. Label money units and whether the metric includes only Paid records. Check what happens when the user selects no matching records or several categories.

## Refresh and report stability

A PivotTable usually summarizes cached data until refreshed. Adding source rows is not enough: use Refresh and verify the source includes those rows. Tables make the source range expandable; a fixed A1:L11 source may miss new rows.

Refresh All can involve connections and PivotTables with different dependencies/background behavior. Wait for source queries to finish, then verify downstream totals; one button press is not proof that every result is current. Record a successful refresh timestamp from the workflow, rather than using NOW as a false claim of data freshness.

Keep dashboard formulas independent of accidental PivotTable cell positions. GETPIVOTDATA can retrieve values by field/item labels and is often more stable than a reference such as B7. Generate it by typing `=` and clicking a PivotTable value, then inspect the resulting field names.

Drill-through can expose the detail rows behind a result. Sharing a summary workbook may still share its source/cache details; inspect the actual file before distribution.

## Practice

Build paid revenue by region/category, monthly paid revenue, and paid line count. Add slicers, connect them, and reconcile totals with SUMIFS. Append a test record, refresh, check the result, then restore the baseline.

Reference: [Microsoft importing and analyzing data](https://support.microsoft.com/en-US/Excel/import-and-analyze-data). Next: [Charts and dashboards](10-charts-and-dashboards.md).
