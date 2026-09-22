# 01 · Excel Fundamentals

Excel combines a grid, calculation engine, analytical tools, and automation interfaces. Begin by understanding how data is stored and how a workbook is organized before memorizing functions.

## Contents

- [Interface and terminology](#interface-and-terminology)
- [Entering and navigating data](#entering-and-navigating-data)
- [Data types and precision](#data-types-and-precision)
- [Files and saving](#files-and-saving)
- [Workbook organization](#workbook-organization)
- [Sharing and printing](#sharing-and-printing)
- [Practice](#practice)

## Interface and terminology

| Term | Meaning |
|---|---|
| Workbook | The complete Excel document |
| Worksheet | One grid/tab within a workbook |
| Cell | A row/column intersection such as B3 |
| Range | A group of cells such as B3:D12 |
| Active cell | The cell receiving input |
| Name Box | Displays/selects addresses or defined names |
| Formula Bar | Shows the active cell's stored value or formula |
| Ribbon | Commands grouped into tabs such as Home, Insert, Data, Formulas |
| Status Bar | Quick summaries and application state |
| Table | A structured dataset with headers, filtering, and expandable references |

Rows use numbers; columns use letters. `A1:C5` is a rectangle, `A:A` is an entire column, and `'Sales Data'!A1` refers to a sheet whose name contains a space. Names containing an apostrophe need it escaped by doubling inside the quoted sheet name.

## Entering and navigating data

- Enter confirms and moves down; Tab moves across; Esc cancels an edit.
- Double-click or F2 edits a cell. Selecting a cell and typing replaces its contents.
- Dragging the fill handle copies or extends a series; check whether Excel copied values, advanced dates, or adjusted formulas.
- Paste Special can paste values, formulas, formats, or transpose. Paste Values deliberately removes formula logic.
- Find locates text or formulas; Replace can alter many records at once. Inspect scope and match settings first.
- Freeze Panes keeps headers visible while scrolling; it does not protect cells.

Use Ctrl+Arrow to navigate contiguous regions on Windows. Blank rows can stop navigation unexpectedly. Ctrl+End goes to the remembered used range, which can extend beyond visible data because of formatting.

## Data types and precision

| Value kind | Example | Common trap |
|---|---|---|
| Number | 1250 | A number imported as text may not aggregate as expected |
| Text | P10 or a postal code | Leading zeros can disappear during automatic conversion |
| Logical | TRUE/FALSE | Text `"TRUE"` is not always treated as a logical value |
| Date/time | An Excel serial with a date/time format | A display resembling a date can still be text |
| Error | #N/A or #DIV/0! | An error is not a blank or zero |
| Empty cell | No stored value | A formula returning `""` is not truly empty |

Excel worksheets support 1,048,576 rows and 16,384 columns, and numeric precision is limited to 15 significant digits. Store long identifiers, account numbers, and postal codes as **text before import**; changing the format afterward cannot restore digits already lost. [Microsoft specifications](https://support.microsoft.com/en-gb/excel/excel-specifications-and-limits)

Displaying two decimal places does not round the underlying value. Use ROUND when a business calculation requires actual rounding. Avoid enabling “precision as displayed” casually: it changes stored numeric precision.

Dates are serial values; time is represented by a fraction of a day. Workbooks can use different date systems, so copied dates can shift if those settings differ. See [dates and text](06-text-dates-and-times.md).

## Files and saving

| Format | Use | Limitation |
|---|---|---|
| .xlsx | Standard workbook | Does not preserve VBA macros |
| .xlsm | Macro-enabled workbook | Recipients need an appropriate macro trust policy |
| .xlsb | Binary workbook | Compatibility/tool support differs; can contain macros |
| .xltx / .xltm | Reusable templates | Macro-enabled template uses .xltm |
| .csv | Interchange of one flat table | No workbook formatting, multiple sheets, charts, or reusable formula model |
| .pdf | Fixed presentation for readers | Not the editable analytical source |

Save meaningful versions before major edits. Autosave and version history depend on storage location and application setup; they do not replace a deliberate backup/recovery policy. A CSV export preserves values for one sheet and may reinterpret dates or identifiers when reopened.

## Workbook organization

Use a small number of sheets with distinct purposes: instructions, raw inputs, cleaned data, calculations, and outputs. Avoid hundreds of almost identical monthly sheets; one table with a date column is usually easier to analyze.

Keep raw records rectangular: one header row, one variable per column, one record per row, consistent types. Put titles, notes, and totals outside the source data rectangle. Avoid merged cells in datasets.

Document owner, purpose, sources, units, assumptions, update frequency, and supported Excel version. Mark input cells consistently and explain the convention. Keep formulas separate from manually supplied parameters.

## Sharing and printing

Before sharing, inspect formulas, hidden sheets/rows, external links, named ranges, query connections, and sensitive cached data. Hidden cells are not access control. Comments support discussion; notes can hold annotations, but critical assumptions belong in visible documentation too.

For printing: set print area, orientation, margins, page breaks, repeated header rows, and scaling. “Fit all rows on one page” can make a large report unreadable. Preview the PDF or print layout before distributing it. Add a report date, units, and page numbering when useful.

## Practice

Create the [shared workbook](00-practice-workbook.md), freeze its header row, save a baseline copy, and identify which columns are text, dates, or numbers. Enter `00123` as text and verify that copying it to a CSV and reopening can require an explicit text import.

Next: [Tables, formatting, and references](02-tables-formatting-and-references.md).
