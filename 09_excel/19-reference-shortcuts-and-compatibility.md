# 19 · Function Reference, Shortcuts, and Compatibility

Use this as a navigation aid after learning the concepts. For complete argument syntax and version markers, consult the [official alphabetical function catalog](https://support.microsoft.com/en-us/excel/excel-functions-alphabetical). No static note can exhaust every Excel function, connector, add-in, and platform variation.

## Contents

- [Function families](#function-families)
- [Common formula patterns](#common-formula-patterns)
- [Windows desktop shortcuts](#windows-desktop-shortcuts)
- [Feature compatibility](#feature-compatibility)
- [Which tool should I use](#which-tool-should-i-use)
- [Official learning references](#official-learning-references)

## Function families

| Family | Functions to learn | When useful |
|---|---|---|
| Arithmetic and rounding | SUM, PRODUCT, ABS, ROUND, ROUNDUP, ROUNDDOWN, INT, MOD, MROUND | Totals, rounding policies, remainders |
| Counts and summaries | COUNT, COUNTA, COUNTBLANK, MIN, MAX, AVERAGE, MEDIAN | Basic inspection and descriptive metrics |
| Conditional aggregation | SUMIF/SUMIFS, COUNTIF/COUNTIFS, AVERAGEIF/AVERAGEIFS, MINIFS/MAXIFS | Metrics by criteria |
| Logical | IF, IFS, AND, OR, NOT, SWITCH | Decisions and classifications |
| Errors and inspection | IFERROR, IFNA, ISERROR, ISNA, ISNUMBER, ISTEXT, ISBLANK, ISFORMULA | Controlled error handling and diagnostics |
| Lookup | XLOOKUP, XMATCH, INDEX, MATCH, VLOOKUP, HLOOKUP | Key/value retrieval and position matching |
| Text cleanup | TRIM, CLEAN, SUBSTITUTE, REPLACE, UPPER, LOWER, PROPER | Normalize defined text patterns |
| Text extraction | LEFT, RIGHT, MID, LEN, FIND, SEARCH, TEXTBEFORE, TEXTAFTER, TEXTSPLIT | Parse fields with known structure |
| Text assembly/conversion | CONCAT, TEXTJOIN, TEXT, VALUE, NUMBERVALUE | Presentation and explicit parsing |
| Dates | DATE, YEAR, MONTH, DAY, EDATE, EOMONTH, TODAY | Calendar boundaries and attributes |
| Times/workdays | TIME, HOUR, MINUTE, SECOND, NOW, WORKDAY, NETWORKDAYS, INTL variants | Duration and work-calendar calculations |
| Dynamic arrays | FILTER, SORT, SORTBY, UNIQUE, SEQUENCE, TRANSPOSE | Growing extracts and lists |
| Array reshaping | TAKE, DROP, CHOOSECOLS, CHOOSEROWS, HSTACK, VSTACK, TOCOL, TOROW | Modern array layouts |
| Reusable formulas | LET, LAMBDA, MAP, REDUCE, SCAN, BYROW, BYCOL | Named intermediate calculations and function composition |
| Visible-row calculations | SUBTOTAL, AGGREGATE | Reports involving filters/errors with selected behavior |
| Weighted/array arithmetic | SUMPRODUCT, MMULT | Weighted metrics and matrix operations |
| Statistical | STDEV.S/P, VAR.S/P, PERCENTILE.INC/EXC, QUARTILE.INC/EXC, RANK.EQ, LARGE, SMALL | Distribution and ranking |
| Relationships/forecast | CORREL, COVARIANCE.S/P, SLOPE, INTERCEPT, LINEST, FORECAST.LINEAR | Statistical modeling mechanics |
| Financial | PV, FV, PMT, NPER, RATE, NPV, XNPV, IRR, XIRR | Time-value calculations with stated cash-flow assumptions |
| Reference/metadata | ROW, COLUMN, ROWS, COLUMNS, ADDRESS, FORMULATEXT | Layout and auditing |
| Cube/model queries | CUBEVALUE, CUBEMEMBER | Reports querying compatible model/cube connections |

Some functions are version-specific. Their inclusion here is a study map, not a claim that an older workbook can calculate them. Learn argument order, types, missing-value behavior, matching rules, and edge cases instead of memorizing names alone.

## Common formula patterns

These patterns use the fixture unless an input cell is explicitly named. Enter structured current-row formulas inside the relevant Table.

| Need | Formula/pattern |
|---|---|
| Paid revenue | `=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid")` |
| Paid line count | `=COUNTIF(Sales[Status],"Paid")` |
| Stable catalog lookup | `=XLOOKUP("P20",Products[ProductID],Products[Product],"Missing")` |
| First day of a row's month | `=DATE(YEAR([@OrderDate]),MONTH([@OrderDate]),1)` |
| Visible revenue total | `=SUBTOTAL(109,Sales[RevenueCents])` |
| Alphabetical unique regions | `=SORT(UNIQUE(Sales[Region]))` |
| Check one row's SaleID uniqueness | `=COUNTIF(Sales[SaleID],[@SaleID])=1` |
| Parse explicit separators | `=NUMBERVALUE("1.234,50",",",".")` |
| Guard an ordinary ratio using A2/B2 | `=IF(B2=0,"",A2/B2)` |

A formula returning empty text may look blank while still being a nonempty formula cell. Keep that distinction in mind when counting, exporting, or plotting.

## Windows desktop shortcuts

These are common Windows Excel defaults. Mac, browser, keyboard layout, accessibility settings, and laptop Fn behavior can differ; use the Ribbon command when a shortcut is intercepted.

| Shortcut | Action |
|---|---|
| Ctrl+N / Ctrl+O / Ctrl+S | New / open / save workbook |
| Ctrl+Z / Ctrl+Y | Undo / redo or repeat where available |
| Ctrl+C / Ctrl+X / Ctrl+V | Copy / cut / paste |
| Ctrl+Alt+V | Paste Special |
| Ctrl+F / Ctrl+H | Find / replace |
| F2 | Edit active cell |
| F4 while editing a reference | Cycle relative/absolute reference forms |
| Ctrl+1 | Format Cells |
| Ctrl+T | Create a Table |
| Ctrl+Shift+L | Toggle filter controls |
| Ctrl+Arrow | Navigate to a region edge |
| Ctrl+Shift+Arrow | Extend selection to a region edge |
| Ctrl+Home / Ctrl+End | Start / remembered last used cell |
| Ctrl+Page Up / Ctrl+Page Down | Switch worksheets |
| Ctrl+D / Ctrl+R | Fill down / right |
| Alt+= | AutoSum |
| Ctrl+; | Insert current date as a static value |
| Ctrl+Shift+; | Insert current time as a static value |
| Alt+Enter | New line inside a cell |
| F9 | Recalculate changed formulas/dependencies in open workbooks |
| Shift+F9 | Calculate active worksheet |
| Ctrl+Alt+F9 | Force recalculation of all formulas in open workbooks |
| Alt+F11 | Open VBA editor |

F9 **while editing a selected formula expression** can evaluate/replace that expression in the edit buffer; use Esc if you are only inspecting it and want to avoid saving a constant in place of the formula. Context matters.

## Feature compatibility

| Feature | What to verify |
|---|---|
| Tables, ordinary formulas, basic charts | Broad support, but exact UI and platform behavior differ |
| XLOOKUP | Not available in Excel 2016/2019; use an older lookup pattern if required |
| Dynamic arrays | Target function support, spill space, and older-workbook behavior |
| LET/LAMBDA and newer helpers | Check each function's version marker individually |
| Power Query | Platform, connector, editor, authentication, and refresh support |
| Power Pivot/Data Model authoring | Edition and platform; Windows desktop is the main full-authoring environment |
| VBA | Desktop; no macro execution in Excel on the web; platform-specific APIs differ |
| Office Scripts | Qualifying account/platform, Automate availability, admin policy, flow support |
| Python in Excel | License, channel, platform, managed runtime, and available libraries |
| Solver/Analysis ToolPak | Add-in installation and platform support |
| Coauthoring/version history | Storage service, file format, account, and feature constraints |

Function names and argument separators can be localized. Decimal separators, dates, CSV delimiters, and array constants also vary. Avoid ambiguous date strings and document the expected source locale.

For a shared workbook, test with the **oldest intended recipient environment**, not merely the author's current Microsoft 365 installation. `_xlfn.` or #NAME? can indicate an unsupported function; updating formatting will not fix missing function support.

## Which tool should I use

- Need a value that responds immediately to an input? Use formulas.
- Need an interactive grouped summary? Start with a PivotTable.
- Need to repeat import/cleanup/join steps? Use Power Query.
- Need related tables and reusable analytical measures? Use the Data Model/DAX.
- Need to automate workbook actions? Compare VBA and Office Scripts deployment requirements.
- Need large-scale transformations, database invariants, or ML training? Use SQL/Python and return an appropriate report to Excel.

## Official learning references

- [Excel function catalog](https://support.microsoft.com/en-us/excel/excel-functions-alphabetical)
- [Excel import and analysis help](https://support.microsoft.com/en-US/Excel/import-and-analyze-data)
- [Power Query help](https://support.microsoft.com/en-us/excel/power-query-for-excel-help)
- [Power Query M reference](https://learn.microsoft.com/en-us/powerquery-m/)
- [DAX reference](https://learn.microsoft.com/en-us/dax/)
- [VBA language reference](https://learn.microsoft.com/en-us/office/vba/api/overview/language-reference)
- [Office Scripts documentation](https://learn.microsoft.com/en-us/office/dev/scripts/overview/excel)
- [Python in Excel availability](https://support.microsoft.com/en-us/excel/python/python-in-excel-availability)

Return to the [Excel learning index](README.md).
