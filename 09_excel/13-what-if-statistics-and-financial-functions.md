# 13 · What-If Analysis, Statistics, and Financial Functions

Use Excel to make assumptions explicit, explore sensitivity, and summarize observations. Distinguish arithmetic examples from a statistically justified model or a real financial decision.

## Contents

- [Scenario modeling](#scenario-modeling)
- [Goal Seek and Data Tables](#goal-seek-and-data-tables)
- [Solver](#solver)
- [Descriptive statistics](#descriptive-statistics)
- [Regression and forecasting](#regression-and-forecasting)
- [Financial-function mechanics](#financial-function-mechanics)
- [Practice](#practice)

## Scenario modeling

On a new WhatIf sheet, enter the following layout. Amounts here are fictional currency units, not the sales fixture's cents.

| Cell | Meaning | Value/formula |
|---|---|---|
| B2 | Unit selling price | 50 |
| B3 | Unit variable cost | 30 |
| B4 | Units sold | 100 |
| B5 | Fixed cost | 2000 |
| B7 | Profit | `=(B2-B3)*B4-B5` |

Profit is zero. Break-even units = fixed cost / contribution per unit = 100, assuming positive contribution and no other constraints. Real cost curves may include capacity limits, tiered costs, and taxes.

Keep assumptions separate from outputs. A scenario table can hold Base, Low Demand, and High Cost inputs; Scenario Manager is another option where available. A scenario is a defined possibility, not automatically a probability forecast.

## Goal Seek and Data Tables

**Goal Seek:** Data → What-If Analysis → Goal Seek. Set B7 to 1000 by changing B4. The required quantity is 150 under this linear model. Goal Seek changes one input to meet one formula target; it does not impose a full set of inequality/integer constraints.

**One-variable Data Table:** put candidate prices 40, 50, 60 in D3:D5 and `=$B$7` in E2, leaving D2 blank. Select D2:E5, open What-If Analysis → Data Table, and set Column input cell to B2. Restore B4 to 100 first. Expected profits: -1000, 0, 1000.

For a two-variable table, place one set of assumptions along a row and another down a column, put the output formula at their corner, and identify the correct row/column input cells. This is a **what-if Data Table**, not an Excel Table made with Ctrl+T. Large Data Tables can be calculation-heavy. [Microsoft what-if overview](https://support.microsoft.com/en-us/excel/introduction-to-what-if-analysis)

## Solver

Solver optimizes an objective by changing several decision cells under constraints. Enable the Solver add-in where available. State the model mathematically before using the dialog.

Example: maximize `3*x + 5*y` subject to `2*x + y <= 10`, `x + 3*y <= 15`, and nonnegative integer x/y.

| Cell | Meaning | Value/formula |
|---|---|---|
| B10 | x | 0 initially |
| B11 | y | 0 initially |
| B13 | Objective | `=3*B10+5*B11` |
| B14 | Resource 1 use | `=2*B10+B11` |
| B15 | Resource 2 use | `=B10+3*B11` |

Maximize B13 by changing B10:B11; add B14<=10, B15<=15, B10:B11>=0, and integer constraints. Choose the appropriate linear method (Simplex LP for this model). Expected optimum: x=3, y=4, objective=29.

GRG Nonlinear targets smooth nonlinear models; Evolutionary can address some nonsmooth/discrete models. A success message does not prove a globally optimal result for arbitrary nonlinear problems. Review feasibility, bounds, initial values, tolerance, and the selected method.

## Descriptive statistics

In a separate Stats sheet put 1,2,3,4,5 in A2:A6.

```excel
=AVERAGE(A2:A6)
=MEDIAN(A2:A6)
=VAR.S(A2:A6)
=VAR.P(A2:A6)
=STDEV.S(A2:A6)
=QUARTILE.INC(A2:A6,1)
=QUARTILE.INC(A2:A6,3)
```

Expected: 3, 3, 2.5, 2, approximately 1.581139, 2, and 4. Sample variance divides by n−1; population variance divides by n. Choose based on what the observations represent, not whichever output is smaller.

PERCENTILE.INC/EXC and QUARTILE.INC/EXC use different interpolation conventions. Document the convention when reconciling with Python, SQL, or another report. Histograms depend on bin edges; outliers need investigation, not automatic deletion.

The Analysis ToolPak can provide descriptive statistics, regression, ANOVA, and other procedures where supported. Check assumptions, labels, ranges, missingness, and independence. A p-value is not the probability that a hypothesis is true, and repeated testing can inflate false positives.

## Regression and forecasting

On Stats, put 3,5,7,9,11 in B2:B6 alongside x=1..5 in A2:A6.

```excel
=SLOPE(B2:B6,A2:A6)
=INTERCEPT(B2:B6,A2:A6)
=CORREL(A2:A6,B2:B6)
=FORECAST.LINEAR(6,B2:B6,A2:A6)
```

Results: slope 2, intercept 1, correlation 1, and forecast 13. This intentionally perfect sequence illustrates syntax; it provides no evidence of general predictive performance.

LINEST can return regression coefficients and additional statistics. For real data, examine residuals, outliers, correlated errors, feature leakage, and performance on held-out observations. Correlation does not establish causation.

Time-series forecasts need sufficient regularly spaced history and a treatment for missing periods. FORECAST.ETS/Forecast Sheet availability varies; trend and seasonality should be validated out of sample. Two months of the sales fixture are inadequate for estimating annual seasonality.

## Financial-function mechanics

| Function | Purpose | Important inputs |
|---|---|---|
| PV / FV | Present/future value | Rate per period, number of periods, payment timing |
| PMT | Constant periodic payment | Rate, periods, principal, end/beginning timing |
| NPER / RATE | Solve period count/rate | Consistent cash-flow signs and time units |
| NPV | Discount equally spaced future cash flows | Initial time-zero outflow is normally added separately |
| XNPV | Discount dated cash flows | Actual dates and consistent signs |
| IRR / XIRR | Rate making modeled net value zero | Multiple/no solutions can occur |

For a purely illustrative 10000-unit principal, nominal annual rate 12% paid monthly over 12 months:

```excel
=PMT(12%/12,12,10000)
```

The result is negative because payments are outflows relative to a positive principal receipt. Dividing a nominal annual rate by 12 is not the same as converting an effective annual rate, which would use `(1+annual_rate)^(1/12)-1`.

For an initial outflow of 1000 and year-end inflows of 600 in years 1 and 2:

```excel
=NPV(10%,600,600)-1000
```

Result: approximately 41.322314. Rates here are made-up teaching inputs, not current quotes or advice. Verify cash-flow timing and compounding before interpreting a real model.

## Practice

Reproduce Goal Seek's 150 units and Solver's objective 29. Explain why the two-variable scenario grid is not a probability distribution. Compare sample/population standard deviation, and calculate NPV manually to check timing.

Next: [VBA](14-vba-and-macros.md).
