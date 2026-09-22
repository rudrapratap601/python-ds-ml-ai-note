# 03 · Formula Foundations, Logic, and Errors

A formula begins with `=` and computes a result from constants, references, operators, and functions. Keep business assumptions in labeled inputs and use formulas to express relationships.

## Contents

- [Syntax and arithmetic](#syntax-and-arithmetic)
- [Logical decisions](#logical-decisions)
- [Missing values and counting](#missing-values-and-counting)
- [Errors and diagnosis](#errors-and-diagnosis)
- [Calculation and auditing](#calculation-and-auditing)
- [Practice](#practice)

## Syntax and arithmetic

```excel
=2+3*4
=(2+3)*4
=ROUND(10/3,2)
=100*(1-15%)
="Order "&101
```

Results: 14, 20, 3.33, 85, and `Order 101`. Parentheses make order of operations explicit. Use `*` for multiplication, `/` for division, `^` for powers, and `&` for joining text. Numeric comparisons use `=`, `<>`, `<`, `<=`, `>`, and `>=`.

Excel uses floating-point arithmetic. Tiny residuals such as a value extremely close to zero can appear; compare using an appropriate tolerance or apply a justified rounding rule. Do not round every intermediate result merely to hide errors.

## Logical decisions

```excel
=IF(2>1,"Yes","No")
=IF(AND(10>=1,10<=100),"In range","Out of range")
=IF(OR("Paid"="Paid","Paid"="Pending"),"Active","Closed")
=NOT(2>1)
```

Use IF for a two-way decision, AND when all tests must be true, OR when any test may be true, and NOT to reverse a logical value. Write mutually understandable output labels instead of unexplained codes.

In a Sales calculated column:

```excel
=IF([@Status]="Paid",[@RevenueCents],0)
=IF([@RevenueCents]=0,"",[@ProfitCents]/[@RevenueCents])
```

The second formula leaves undefined margins visually blank. That blank-looking result is text, not a zero margin. Downstream calculations need a deliberate policy.

IFS can replace a long nested IF for ordered conditions, and SWITCH can map a single value to labels where available. A lookup table is usually easier to maintain than dozens of hard-coded business categories. Check newer function support in the [Microsoft catalog](https://support.microsoft.com/en-us/excel/excel-functions-alphabetical).

## Missing values and counting

| Function | What it counts/tests |
|---|---|
| COUNT | Numeric values, including valid date serials in references |
| COUNTA | Nonempty values, including errors and formulas returning empty text |
| COUNTBLANK | Empty cells and formulas returning empty text |
| ISBLANK | A genuinely empty cell, not a formula returning `""` |
| ISNUMBER | Whether a value is numeric |
| ISTEXT | Whether a value is text |

Thus COUNTA and COUNTBLANK are not complements in every situation. A row with formulas can look blank while still containing content. Zero is a real number and should not automatically be treated as missing.

For a population mean, `AVERAGE(range)` ignores empty cells and text in references, but includes numeric zeros. Replacing missing observations with zero changes the question and denominator.

## Errors and diagnosis

| Error/display | Typical cause | First response |
|---|---|---|
| #DIV/0! | Zero or empty denominator | Check denominator and intended missing-value policy |
| #N/A | Lookup has no match | Check keys, types, whitespace, and match mode |
| #VALUE! | Wrong type or incompatible argument | Inspect inputs and array shapes |
| #REF! | Invalid/deleted reference | Restore or rebuild the intended reference |
| #NAME? | Unknown function/name or missing quotes | Check spelling, defined names, and version |
| #NUM! | Invalid numeric domain or failed iteration | Inspect numeric assumptions |
| #SPILL! | Dynamic output range cannot be populated | Inspect obstacles, merged cells, and Table placement |
| #CALC! | Unsupported/empty array calculation | Check formula structure and empty-result handling |
| #### | Often a narrow column; can be unsupported negative date/time display | Widen it, then inspect the actual value |
| Formula shown as text | Text format, leading apostrophe, or Show Formulas mode | Inspect the cell and re-enter after fixing the cause |

```excel
=IFERROR(10/0,"Review denominator")
=IFNA(NA(),"Missing lookup")
```

IFERROR catches many error types; IFNA catches #N/A. Use the narrowest handling that matches your intention. Wrapping an entire model in `IFERROR(...,0)` can make missing references and broken logic look like valid zero activity.

## Calculation and auditing

Automatic calculation updates dependent formulas when inputs change; manual mode can leave results stale. Check calculation mode when debugging and before distribution. Refreshing a data connection, refreshing a PivotTable, and recalculating formulas are related but different operations.

Use Show Formulas, Trace Precedents/Dependents, Evaluate Formula, and the Watch Window where available. For a complex expression, inspect intermediate values in helper cells before compressing it into a single formula.

A circular reference means a calculation depends on itself directly or indirectly. Iterative calculation can model intentional circular systems, but enabling it to silence an accidental circular reference hides the problem. Define convergence and maximum-iteration rules for intentional iterative models.

Audit by changing one input at a time and checking direction and magnitude of the result. Add reconciliation checks, for example all-status revenue = paid + pending + cancelled revenue.

## Practice

Create a divide-by-zero example and fix its meaning rather than only hiding the error. Compare ISBLANK, COUNTA, and COUNTBLANK on an empty cell, zero, and a formula returning `""`. Verify that changing one quantity changes the relevant revenue and summary totals.

Next: [Conditional aggregation](04-aggregation-and-business-metrics.md).
