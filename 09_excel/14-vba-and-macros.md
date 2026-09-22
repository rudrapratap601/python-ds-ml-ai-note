# 14 · VBA and Macros

VBA automates desktop Office workflows. Use it when a repeatable task needs workbook interaction that formulas or Power Query do not provide. Start with a small, explicit operation and test on a copy.

## Contents

- [Setup and the object model](#setup-and-the-object-model)
- [Language essentials](#language-essentials)
- [A complete reporting macro](#a-complete-reporting-macro)
- [Error handling and performance](#error-handling-and-performance)
- [Events, functions, and deployment](#events-functions-and-deployment)
- [Practice](#practice)

## Setup and the object model

Save a practice copy as `.xlsm`; `.xlsx` does not preserve VBA. Enable the Developer tab through Excel settings where needed. On Windows, Alt+F11 opens the VBA editor; Insert → Module adds a standard module. The macro recorder is useful for discovering object-model calls, but recorded code often hard-codes selections and ranges.

| Object | Example |
|---|---|
| Application | Excel-wide settings |
| Workbook | ThisWorkbook: the workbook containing the code |
| Worksheet | ThisWorkbook.Worksheets("Sales") |
| Range | A rectangular cell area |
| ListObject | An Excel Table |
| ListColumn | A column within that Table |

ActiveWorkbook and ActiveSheet depend on the user's current focus. Prefer explicit workbook/sheet references. ThisWorkbook is appropriate when the code is stored in the workbook it should operate on; an add-in or PERSONAL.XLSB macro needs a deliberately selected target instead.

## Language essentials

Use `Option Explicit` to require variable declarations. Declare row counters as Long, text as String, and numeric totals using types suitable for the domain. Variant arrays are useful for bulk reads/writes.

Sub procedures perform actions; Functions return a value. If/Else branches, For/For Each loops, and Select Case express control flow. `Set` assigns an object reference. A VBA array can have different lower bounds; inspect LBound/UBound instead of assuming zero.

Avoid `.Select` and `.Activate` for ordinary data work. `Worksheets("Analysis").Range("A1").Value = "Report"` expresses the target directly.

## A complete reporting macro

Prerequisite: the baseline Sales sheet contains the Table named Sales. The macro creates or reuses a worksheet named VBAReport and **overwrites only A1:B4 there**. Paste into a standard module in the practice workbook and run BuildPaidSummary.

```vb
Option Explicit

Public Sub BuildPaidSummary()
    Dim source As ListObject
    Dim report As Worksheet
    Dim data As Variant
    Dim output(1 To 4, 1 To 2) As Variant
    Dim rowIndex As Long
    Dim statusCol As Long, quantityCol As Long
    Dim priceCol As Long, costCol As Long
    Dim paidLines As Long
    Dim revenue As Double, cost As Double
    Dim oldScreenUpdating As Boolean
    Dim failureMessage As String

    oldScreenUpdating = Application.ScreenUpdating
    On Error GoTo Failed
    Application.ScreenUpdating = False

    Set source = ThisWorkbook.Worksheets("Sales").ListObjects("Sales")
    If source.DataBodyRange Is Nothing Then
        Err.Raise vbObjectError + 1, , "Sales has no data rows."
    End If
    statusCol = source.ListColumns("Status").Index
    quantityCol = source.ListColumns("Quantity").Index
    priceCol = source.ListColumns("UnitPriceCents").Index
    costCol = source.ListColumns("UnitCostCents").Index
    data = source.DataBodyRange.Value2

    For rowIndex = 1 To UBound(data, 1)
        If data(rowIndex, statusCol) = "Paid" Then
            If Not IsNumeric(data(rowIndex, quantityCol)) Or _
               Not IsNumeric(data(rowIndex, priceCol)) Or _
               Not IsNumeric(data(rowIndex, costCol)) Then
                Err.Raise vbObjectError + 2, , "A paid row contains nonnumeric inputs."
            End If
            revenue = revenue + CDbl(data(rowIndex, quantityCol)) * CDbl(data(rowIndex, priceCol))
            cost = cost + CDbl(data(rowIndex, quantityCol)) * CDbl(data(rowIndex, costCol))
            paidLines = paidLines + 1
        End If
    Next rowIndex

    On Error Resume Next
    Set report = ThisWorkbook.Worksheets("VBAReport")
    Err.Clear
    On Error GoTo Failed
    If report Is Nothing Then
        Set report = ThisWorkbook.Worksheets.Add(After:=ThisWorkbook.Worksheets(ThisWorkbook.Worksheets.Count))
        report.Name = "VBAReport"
    End If

    output(1, 1) = "Metric": output(1, 2) = "Value"
    output(2, 1) = "Paid revenue cents": output(2, 2) = revenue
    output(3, 1) = "Paid profit cents": output(3, 2) = revenue - cost
    output(4, 1) = "Paid lines": output(4, 2) = paidLines
    report.Range("A1:B4").Value2 = output
    report.Range("A1:B1").Font.Bold = True
    report.Columns("A:B").AutoFit
    Application.ScreenUpdating = oldScreenUpdating
    Exit Sub

Failed:
    failureMessage = Err.Description
    Application.ScreenUpdating = oldScreenUpdating
    MsgBox "Report failed: " & failureMessage, vbExclamation
End Sub
```

Expected metrics: 28000, 14600, and 8. This is a teaching macro for valid fixture data, not a complete schema validator. Add explicit range/integer/status checks if accepting uncontrolled input. No external files are read or sent.

## Error handling and performance

Read data into an array, calculate in memory, and write a whole result block. Repeated cell-by-cell access can be slow. Avoid selecting cells merely to read them.

If changing Application.Calculation, EnableEvents, ScreenUpdating, or DisplayAlerts, save the prior state and restore it on success **and** failure. Do not always force a preferred global setting on exit; the user may have deliberately chosen another state.

On Error Resume Next suppresses errors; use it only for a narrowly scoped, expected condition, then inspect/reset error handling. A broad handler that keeps going after failed writes can produce a partially correct report.

## Events, functions, and deployment

Worksheet_Change reacts to edits, while calculation events address recalculation. Event handlers that write cells can trigger themselves; guard against recursion and restore EnableEvents even after errors. Place worksheet events in the correct worksheet module, not a standard module.

Worksheet user-defined functions should compute results rather than modify unrelated cells or application state. Nonvolatile UDF dependencies should be passed explicitly. Use LAMBDA when a pure worksheet function can express the task more simply.

VBA macros are desktop-oriented and do not run in Excel on the web. Some Windows-specific APIs, COM automation, and controls do not work on Mac. Macro trust policies, signing, and deployment rules matter; do not tell recipients to enable every macro globally. [Microsoft VBA/Office Scripts comparison](https://learn.microsoft.com/en-us/office/dev/scripts/resources/vba-differences)

## Practice

Run the macro twice and verify it updates the same report. Rename an input header on a copy and verify failure is visible and ScreenUpdating restored. Add a quantity change and reconcile the regenerated report with worksheet formulas.

Next: [Office Scripts and automation choices](15-office-scripts-and-automation.md).
