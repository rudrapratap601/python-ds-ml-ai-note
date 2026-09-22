# 10 · Charts, Dashboards, and Reporting

A dashboard should help someone answer a question or make a decision. Begin with the audience, metric definitions, comparison period, and required action; choose charts afterward.

## Contents

- [Choose a chart](#choose-a-chart)
- [Build an accurate chart](#build-an-accurate-chart)
- [A sales dashboard layout](#a-sales-dashboard-layout)
- [Interaction and accessibility](#interaction-and-accessibility)
- [Publishing checklist](#publishing-checklist)
- [Practice](#practice)

## Choose a chart

| Question | Suitable starting point | Watch for |
|---|---|---|
| Compare categories | Sorted bar or column | Long labels, too many categories |
| Show change over time | Line | Missing periods, uneven intervals, incorrect date sorting |
| Compare two numeric variables | XY scatter | Confounding and outliers; correlation is not causation |
| Show distribution | Histogram or box-and-whisker | Bin choice, sample size, excluded missing values |
| Explain movement from start to finish | Waterfall | Correct intermediate totals and signs |
| Compare composition | Stacked bar or 100% stacked bar | Hard-to-compare internal segments |
| Show a small share-of-whole breakdown | Bar, or carefully limited pie | Tiny slices and changing denominators |
| Show a compact trend in a table | Sparkline | Consistent axes for meaningful comparisons |

Avoid 3-D effects that distort area and depth. A secondary axis can be useful, but unrelated scales can create a false visual relationship; label both axes and consider separate aligned charts.

## Build an accurate chart

1. Create a clean summary table with clear headers and units.
2. Select it and choose Insert → the appropriate chart type.
3. Verify categories, series, date treatment, and aggregation.
4. Give the chart a title explaining the metric and population.
5. Set number formats, axis limits, labels, and a readable legend.
6. Test filtering, appended data, blanks, and zero values.

Bar/column length normally needs a zero baseline to represent magnitude honestly. A line chart may use a narrower scale for change, but make the axis visible and avoid exaggerating insignificant variation. Use an XY scatter chart for numeric x-values; a category line chart can space labels equally even when numbers are uneven.

Do not chart numbers stored as text, sort month names alphabetically, or include a grand total as though it were another category. Define whether blanks become gaps, zeros, or connected lines; each tells a different story.

## A sales dashboard layout

Use [the fixture](00-practice-workbook.md) and define the default population as Paid sales across January–February 2026.

| Area | Content | Baseline |
|---|---|---|
| Header | Report title, period, source, successful refresh time | Clearly labeled |
| KPI cards | Paid revenue, paid orders, gross margin | 28000 cents; 6; 52.142857% |
| Trend | Monthly revenue versus target | 12000/10000; 16000/15000 |
| Breakdown | Paid revenue by category | Books 6000; Learning 12000; Tools 10000 |
| Detail | Region totals and variance | North 14000; South 14000 |
| Controls | Region/category slicers or dropdowns | Visible selected state |
| Footer | Definitions and exceptions | Zero-denominator and missing-data policy |

Convert cents to currency units explicitly in the chart source if desired: 28000 cents becomes 280 units. Keep the displayed unit consistent across cards, charts, and tables.

Budget is only at monthly grain. If a Region slicer changes actuals but leaves the full-company budget unchanged, comparing them is misleading unless the dashboard explicitly explains it. Either disable that comparison for partial populations or provide a valid allocated budget.

## Interaction and accessibility

Use a limited, consistent palette. Pair color with text, signs, or markers so red/green differences are not the only information. Use readable font sizes, descriptive chart titles, and alt text where supported.

Reserve space for longer labels and growing spill/PivotTable outputs. Keep input controls visually distinct from calculated outputs. Protect formulas against accidental edits when appropriate, while understanding that worksheet protection is not confidentiality.

Display empty selections explicitly. A card left over from the previous calculation can be worse than an error. Avoid a dashboard dependent on the active sheet, an untracked manual filter, or a hidden local file path.

## Publishing checklist

- Reconcile cards, charts, and detail totals under the same filters.
- Confirm the refresh completed and the requested reporting period is complete.
- Check both the full population and a small/empty selection.
- Inspect hidden data, query connections, external links, and private source details.
- Review at the expected zoom and in the intended PDF/print layout.
- Include metric definitions, ownership, and instructions for refreshing.

## Practice

Build the layout above using either formulas or PivotTables. Then select North and explain why company-level budget must not silently be interpreted as North's target. Ask a reader to identify the current period and units without opening the source sheets.

Next: [Power Query](11-power-query-and-m.md).
