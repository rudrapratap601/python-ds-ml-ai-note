# 15 · Office Scripts and Automation Choices

Office Scripts automate Excel through a TypeScript-based API. They can integrate with Power Automate where the account, licensing, tenant policies, and platform support it. Check for the Automate tab and consult the current requirements rather than assuming all Excel installations include the feature. [Microsoft overview](https://learn.microsoft.com/en-us/office/dev/scripts/overview/excel)

## Contents

- [Choose the right tool](#choose-the-right-tool)
- [Script structure](#script-structure)
- [A complete summary script](#a-complete-summary-script)
- [Reliable automated workflows](#reliable-automated-workflows)
- [Performance and practice](#performance-and-practice)

## Choose the right tool

| Need | Starting point |
|---|---|
| Live calculation from cell inputs | Formulas or LAMBDA |
| Repeatable import/clean/merge | Power Query |
| Relational aggregation | Data Model/DAX |
| Desktop Excel object automation or legacy integration | VBA |
| Workbook automation with supported cloud-flow integration | Office Scripts |
| Large external analysis or ML pipeline | Python/SQL outside Excel |

Power Query transforms data; Office Scripts operate on the workbook through APIs. Neither is a universal replacement for the other. [Microsoft comparison](https://learn.microsoft.com/en-us/office/dev/scripts/resources/power-query-differences)

## Script structure

An Office Script exposes `main(workbook: ExcelScript.Workbook, ...)`. Use typed arrays and explicit worksheet/Table names. A recorded action sequence is a starting point, not a guarantee of robustness when rows, headers, or active selections change.

Office Scripts are not arbitrary Node.js programs: do not assume a filesystem, installed npm packages, desktop COM APIs, or all browser APIs. Network access and execution constraints depend on how the script runs.

## A complete summary script

Prerequisite: the baseline Table named Sales. Create a script through Automate → New Script and replace the starter code below. It creates or reuses ScriptReport and **overwrites A1:B5 there**. It does not edit source records.

```typescript
function main(workbook: ExcelScript.Workbook) {
  const table = workbook.getTable("Sales");
  if (!table) throw new Error("Missing Sales table.");
  if (table.getRowCount() === 0) throw new Error("Sales has no data rows.");

  const headers = table.getHeaderRowRange().getValues()[0].map(value => String(value));
  const index = (name: string): number => {
    const position = headers.indexOf(name);
    if (position < 0) throw new Error(`Missing column: ${name}`);
    return position;
  };
  const status = index("Status");
  const quantity = index("Quantity");
  const price = index("UnitPriceCents");
  const order = index("OrderID");
  const rows = table.getRangeBetweenHeaderAndTotal().getValues();
  const orders = new Set<string>();
  let revenue = 0;
  let paidLines = 0;

  for (const row of rows) {
    if (row[status] !== "Paid") continue;
    const q = row[quantity];
    const p = row[price];
    const orderID = row[order];
    if (typeof q !== "number" || typeof p !== "number" ||
        !Number.isInteger(q) || q <= 0 || !Number.isInteger(p) || p < 0) {
      throw new Error("Paid rows need positive integer quantities and nonnegative integer cent prices.");
    }
    if (typeof orderID !== "number" || !Number.isInteger(orderID)) {
      throw new Error("The practice OrderID must be an integer.");
    }
    revenue += q * p;
    paidLines += 1;
    orders.add(String(orderID));
  }

  let report = workbook.getWorksheet("ScriptReport");
  if (!report) report = workbook.addWorksheet("ScriptReport");
  report.getRange("A1:B5").setValues([
    ["Metric", "Value"],
    ["Paid revenue cents", revenue],
    ["Paid lines", paidLines],
    ["Paid orders", orders.size],
    ["Generated at (UTC)", new Date().toISOString()]
  ]);
  report.getRange("A1:B1").getFormat().getFont().setBold(true);
  report.getRange("A1:B5").getFormat().autofitColumns();
}
```

Expected: revenue 28000, lines 8, orders 6. The timestamp records when this summary was generated; it does **not** prove an external data source was refreshed then. Save the script and test repeated runs on a practice copy.

## Reliable automated workflows

For a scheduled flow, document the trigger, workbook location, permitted input changes, script arguments, output, and failure notification. This chapter describes the design; it does not create a schedule or send any data.

Make repeated execution safe. Replacing a designated report range is naturally repeatable; blindly appending the same transaction on every retry creates duplicates. Use business keys or run IDs for append workflows and define how partial failures are recovered.

Avoid concurrent writers to the same workbook. Choose a controlled sequence: acquire/prepare input, complete supported refresh operations, compute, validate, then publish. Not every refresh API behaves identically in an interactive script and a Power Automate run; test that environment explicitly.

Do not hard-code credentials or depend on ActiveWorksheet, a selected range, or one user's local file path. Pass validated parameters and fail clearly when expected sheets or headers are missing.

## Performance and practice

Read a range once, process values in memory, and write in batches. Repeated workbook API reads inside loops can be expensive. Avoid logging large private datasets. Define upper bounds for record count and execution time for the intended workflow.

Practice: run the summary twice, add a legitimate line to an existing order, and confirm line count increases while distinct order count stays unchanged. Try a text quantity on a copy and verify the script rejects it rather than silently coercing it.

Next: [Python and the ML workflow](16-python-sql-and-ml-workflows.md).
