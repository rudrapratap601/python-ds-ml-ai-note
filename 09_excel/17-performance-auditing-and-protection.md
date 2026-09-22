# 17 · Performance, Auditing, Protection, and Collaboration

A workbook is useful only if its results can be understood, reproduced, refreshed, and shared appropriately. Reliability depends on both calculation design and operational habits.

## Contents

- [Find the real bottleneck](#find-the-real-bottleneck)
- [Formula and model performance](#formula-and-model-performance)
- [Audit a workbook](#audit-a-workbook)
- [Protection and privacy](#protection-and-privacy)
- [Collaboration and versioning](#collaboration-and-versioning)
- [Delivery checklist](#delivery-checklist)

## Find the real bottleneck

Separate workbook opening, source refresh, formula recalculation, PivotTable refresh, macro execution, and saving. Measure each phase on representative data. A slow network query will not be fixed by changing cell colors; a million-cell volatile array will not be fixed by renaming a sheet.

Record a baseline: file size, row counts, calculation mode, refresh time, important formulas, and hardware/platform. Change one material factor and compare both speed and results.

## Formula and model performance

| Issue | Candidate improvement | Tradeoff |
|---|---|---|
| Whole-column array calculations | Use bounded Tables/ranges | Ensure future rows remain included |
| Repeated expensive expressions | LET/helper columns | Extra named intermediates need documentation |
| Heavy volatile formulas | Replace unnecessary INDIRECT/OFFSET or repeated NOW/RAND usage | Preserve required dynamic behavior |
| Many external workbook links | Consolidate a controlled input query | Define refresh/credential ownership |
| Repeated manual cleanup | Power Query | Requires a stable transformation contract |
| Huge flat lookup model | Relational model or upstream SQL | More modeling knowledge required |
| Cell-by-cell automation | Bulk arrays and batch writes | Validate the exact write range |
| Excess formatting/rules | Restrict to the real data/report area | Keep future expansion intentional |

Volatile functions can recalculate more often than their apparent input changes suggest, affecting dependent formulas too. Not every slow formula is volatile; inspect dependencies and actual data size.

Manual calculation is a workflow choice, not a correctness fix. Before publishing, deliberately recalculate, complete refreshes, and inspect control totals. Restoring automatic calculation after a macro is not always correct if the user originally chose manual; restore the prior state.

For larger models, remove unused columns, avoid unnecessary high-cardinality strings, use appropriate types, and aggregate at the source when detail is not needed. Worksheet row limits are not a target size for every formula. Move computation to SQL/Python when that better suits the workload.

## Audit a workbook

1. Identify its purpose, inputs, outputs, owner, and assumptions.
2. Inspect formulas and named ranges for broken/external references.
3. Compare calculated-column formulas; a pasted constant may have replaced one row's formula.
4. Verify dates, money units, signs, types, lookup uniqueness, and aggregation grain.
5. Trace the important outputs back to source data and parameters.
6. Run boundary cases: zero rows, zero denominator, duplicate keys, missing matches, ties, and an extra row.
7. Reconcile independent implementations, such as SUMIFS against a PivotTable or Power Query group.
8. Record checks with expected results and investigate failures before hiding them.

Use Evaluate Formula and Trace Precedents where available. Maintain an exception count alongside success metrics. A dashboard with a correct grand total can still have incorrect category allocation.

## Protection and privacy

| Mechanism | What it helps with | What it does not provide |
|---|---|---|
| Locked cells + Protect Sheet | Prevent accidental edits to designated cells | Strong confidentiality/access control |
| Workbook structure protection | Restrict sheet-structure changes | Encryption of all visible and hidden data |
| File encryption/password to open | Protect supported encrypted file access | Recovery if a necessary password is lost |
| Cloud sharing permissions | Control access through the storage service | Removal of copies already downloaded |
| Hidden rows/sheets | Simplify presentation | Secrecy |

To protect formula cells from accidental edits, unlock intended input cells first, then protect the sheet with the required allowed operations. Check filtering, sorting, and Table expansion afterward. Microsoft explicitly distinguishes worksheet protection from a security feature. [Protect a worksheet](https://support.microsoft.com/en-us/excel/protect-a-worksheet)

Before distribution, inspect hidden data, PivotTable caches, query results, external links, comments, names, and document properties. Sharing a summary chart inside a workbook can still expose the underlying records.

## Collaboration and versioning

Use a shared location and clear ownership when coauthoring is supported. Version history helps recover earlier states, but concurrent changes to formulas, macros, and queries still need review. Comments are helpful for questions; accepted business logic belongs in the model documentation.

Store small source CSVs, SQL/M/Python/VBA exports, data dictionaries, and Markdown notes in Git where appropriate. Excel workbook binaries are harder to review and merge than text. Avoid committing private data, secrets, or huge generated files. See the repository's version-control notes for workflow concepts.

Version names should distinguish source snapshots from report outputs. Keep a reproducible baseline and record the engine/version and refresh parameters. A file called final_final2 is not a reliable provenance trail.

## Delivery checklist

- The intended recipient's Excel version supports required functions and features.
- Inputs, outputs, units, metric population, and zero/missing policies are documented.
- Source refresh completed and critical calculations/control totals pass.
- Formulas and data ranges include new rows without overwriting inputs.
- Permissions, external dependencies, and sensitive content have been inspected.
- A reader can reproduce the report and recover the source baseline.

Next: [Practice projects and solutions](18-practice-projects-and-solutions.md).
