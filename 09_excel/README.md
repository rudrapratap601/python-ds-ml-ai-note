# Excel: Beginner to Advanced

A structured set of notes for learning Excel as a spreadsheet user, data analyst, and Python/ML practitioner. Chapters include explanations, formulas, worked examples, pitfalls, exercises, and official references. Start with the shared dataset so the same numbers can be checked across formulas, PivotTables, Power Query, DAX, and automation.

## Learning path

| Chapter | Coverage |
|---|---|
| [00 · Practice workbook](00-practice-workbook.md) | Copyable sales/product data, table setup, calculated columns, targets, expected totals |
| [01 · Fundamentals](01-workbook-fundamentals.md) | Interface, navigation, types, files, organization, printing, sharing |
| [02 · Tables and references](02-tables-formatting-and-references.md) | Structured references, absolute/mixed references, names, formats, sorting/filtering |
| [03 · Formula foundations](03-formula-foundations-and-errors.md) | Arithmetic, logic, missing values, errors, calculation, auditing |
| [04 · Aggregation](04-aggregation-and-business-metrics.md) | SUMIFS/COUNTIFS, date criteria, weighted metrics, margins, distinct counts, variance |
| [05 · Lookups](05-lookups-and-matching.md) | XLOOKUP, INDEX/MATCH, VLOOKUP, exact/approximate matching, duplicate keys |
| [06 · Text and dates](06-text-dates-and-times.md) | Cleanup, extraction, conversion, dates, time, workdays, locale/timezone traps |
| [07 · Modern formulas](07-dynamic-arrays-let-and-lambda.md) | FILTER/SORT/UNIQUE, spills, array helpers, LET, LAMBDA, MAP/REDUCE/SCAN |
| [08 · Data quality](08-data-validation-and-cleaning.md) | Dropdowns, validation, duplicates, missing data, import/export checks |
| [09 · PivotTables](09-pivottables-and-pivotcharts.md) | Grouping, aggregation, distinct counts, dates, percentages, slicers, refresh |
| [10 · Charts and dashboards](10-charts-and-dashboards.md) | Chart choice, metric cards, actual/target reports, interaction, accessibility |
| [11 · Power Query and M](11-power-query-and-m.md) | Import, types, transformations, append/merge, unpivot, M code, folding, refresh |
| [12 · Data Model and DAX](12-data-model-power-pivot-and-dax.md) | Relationships, star schema, measures, context, CALCULATE, calendar/YTD |
| [13 · Analysis](13-what-if-statistics-and-financial-functions.md) | Goal Seek, scenarios, Data Tables, Solver, statistics, regression, cash-flow functions |
| [14 · VBA](14-vba-and-macros.md) | Object model, complete macro, arrays, error recovery, events, deployment |
| [15 · Office Scripts](15-office-scripts-and-automation.md) | TypeScript workbook API, complete script, repeatable workflows, automation choices |
| [16 · Python, SQL, and ML](16-python-sql-and-ml-workflows.md) | pandas reporting, file-library limits, Python in Excel, reproducibility, leakage |
| [17 · Reliability](17-performance-auditing-and-protection.md) | Performance, audits, protection, privacy, collaboration, versioning |
| [18 · Practice and projects](18-practice-projects-and-solutions.md) | Fifteen worked exercises, four portfolio projects, interview questions |
| [19 · Quick reference](19-reference-shortcuts-and-compatibility.md) | Function families, formula patterns, Windows shortcuts, version/platform guide |

## Suggested routes

- **New to Excel:** 00–06, then 08–10. Practice each topic before adding automation.
- **Data analysis:** add 07, 11–13, and 17–18. Focus on table grain, keys, refresh, and reconciliation.
- **Automation:** learn Tables/formulas first, then 11, 14–17. Choose a tool based on where the workbook must run.
- **ML journey:** emphasize types, data quality, Power Query/SQL, temporal feature availability, Python, and reproducible source snapshots.

## Conventions and requirements

- Notes use English function names, comma argument separators, and mostly Windows desktop menu paths. Adapt for your locale/platform.
- Modern functions and advanced tools have version/license requirements; consult Chapter 19 and linked official documentation.
- Money in the shared Sales fixture uses integer cents. Statistical/what-if chapters explicitly identify separate miniature datasets.
- Each line in an `excel` code block is an independent formula unless its placement is otherwise explained. Put current-row formulas inside the relevant Table and spill formulas outside Tables.
- M, DAX, VBA, TypeScript, and Python blocks run in different editors/runtimes; they are not worksheet formulas.
- The notes include data and instructions to build a practice workbook. They do not include a prebuilt .xlsx file.

## How to practice

Save a baseline copy after setup. Predict outputs before entering formulas, compare against the provided answers, and test missing records, duplicates, ties, empty selections, and newly appended rows. Use a practice copy for transformations and scripts that modify a workbook.

Do not treat a lack of visible errors as proof of correctness. Reconcile a metric using an independent method and record its population, grain, denominator, units, and refresh state.

## Validation and limits

Checked on 2026-09-21: all 181 local links/heading references resolved, 142 formula instances passed static delimiter checks, both Python code blocks parsed, and 26 independent dataset/arithmetic checks matched the expected answers. Static checks do not evaluate Excel formulas. The workbook formulas, M, DAX, VBA, and Office Scripts examples were not executed in their native applications; the Python file-I/O example was not run against a saved practice workbook. Follow the stated prerequisites and verify results in your target environment.

These notes provide broad beginner-to-advanced coverage rather than reproducing every vendor function or connector. The reference chapter links official catalogs for deeper specialization.

Return to the [repository index](../README.md).
