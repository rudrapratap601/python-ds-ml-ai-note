# 04 · Aggregation and Business Metrics

Use the [Sales table](00-practice-workbook.md). Every metric needs a definition: which records qualify, what one row represents, what units it uses, and what its denominator means.

## Contents

- [Basic summaries](#basic-summaries)
- [Conditional summaries](#conditional-summaries)
- [Dates and criteria](#dates-and-criteria)
- [Weighted metrics and ratios](#weighted-metrics-and-ratios)
- [Distinct orders and targets](#distinct-orders-and-targets)
- [Practice](#practice)

## Basic summaries

```excel
=SUM(Sales[RevenueCents])
=SUM(Sales[Quantity])
=MIN(Sales[UnitPriceCents])
=MAX(Sales[UnitPriceCents])
=AVERAGE(Sales[UnitPriceCents])
```

Expected results: 36000, 14, 1000, 5000, and 3000. The last result is a **line-weighted mean of unit prices**, not a quantity-weighted price and not an average order value.

MEDIAN returns the middle of sorted numeric observations; LARGE/SMALL return an ordered observation; RANK.EQ assigns ranks with ties. Each answers a different question from SUM or AVERAGE.

## Conditional summaries

```excel
=SUMIF(Sales[Status],"Paid",Sales[RevenueCents])
=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid",Sales[Region],"North")
=COUNTIF(Sales[Status],"Paid")
=COUNTIFS(Sales[Status],"Paid",Sales[Region],"North")
=AVERAGEIFS(Sales[RevenueCents],Sales[Status],"Paid")
```

Expected: 28000, 14000, 8, 5, and 3500. The average is per **paid line**. Orders with more lines contribute more observations; average paid order value is 28000/6 instead.

SUMIF places its sum range last; SUMIFS places it first. SUMIFS/COUNTIFS criteria pairs are joined by AND. For OR across mutually exclusive statuses, add two SUMIFS results. If conditions can overlap, adding them can double count.

Criteria `*` and `?` match text patterns; `~` escapes a wildcard. A not-equal-to-Cancelled criterion excludes that literal label but does not prove that every remaining row is a valid paid sale.

## Dates and criteria

```excel
=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid",Sales[OrderDate],">="&DATE(2026,1,1),Sales[OrderDate],"<"&DATE(2026,2,1))
```

Result: 12000. Concatenate comparison operators to cell values or DATE expressions. Do not put a reference inside quotes and expect it to be evaluated.

Use half-open intervals: greater than or equal to the start, less than the next period's start. An end-date condition using `<=` on midnight can exclude later timestamps on the final day. Grouping by a text month such as Jan also mixes different years unless year is represented.

With a month-start date in Analysis!A2, use:

```excel
=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid",Sales[OrderDate],">="&A2,Sales[OrderDate],"<"&EDATE(A2,1))
```

## Weighted metrics and ratios

```excel
=SUMPRODUCT(Sales[Quantity],Sales[UnitPriceCents])
=SUMPRODUCT((Sales[Status]="Paid")*Sales[Quantity]*Sales[UnitPriceCents])
=SUMIFS(Sales[ProfitCents],Sales[Status],"Paid")/SUMIFS(Sales[RevenueCents],Sales[Status],"Paid")
```

Expected: 36000, 28000, and about 52.142857%. The Boolean condition in SUMPRODUCT becomes 1 or 0 through multiplication. All arrays must align in length; text/errors inside arithmetic need proper cleaning.

Gross margin = profit / revenue. Markup = profit / cost. These are different measures. Overall margin is total profit divided by total revenue, **not** the ordinary average of line margins. Likewise, a weighted score is SUMPRODUCT(values,weights)/SUM(weights), with an explicit rule for missing observations and zero total weight.

For a zero denominator, decide whether the result is blank, not applicable, or a meaningful default; do not silently display 0% as if it were measured performance.

## Distinct orders and targets

Modern Excel with FILTER and UNIQUE:

```excel
=ROWS(UNIQUE(FILTER(Sales[OrderID],Sales[Status]="Paid")))
```

Result: 6 on the fixture. If there are no paid records, FILTER here returns an error; adding empty text as its fallback and blindly counting rows would incorrectly count one. For a model that permits no paid records:

```excel
=IF(COUNTIF(Sales[Status],"Paid")=0,0,ROWS(UNIQUE(FILTER(Sales[OrderID],Sales[Status]="Paid"))))
```

Older-version options include a deduplicated paid-order list using Power Query or a Data Model distinct-count measure. A regular COUNT of OrderID is a line count in this dataset.

Create a monthly report with MonthStart, ActualCents, TargetCents, VarianceCents, and VariancePercent. Actual minus target is positive when revenue exceeds this target. For cost metrics, a favorable variance may have the opposite sign.

```excel
=SUM(Sales[PaidRevenueCents])-SUM(Budget[TargetCents])
=SUM(Sales[PaidRevenueCents])/SUM(Budget[TargetCents])-1
```

Results: 3000 cents and 12%. Sum the totals first; averaging January and February variance percentages would answer a different question.

## Practice

Calculate paid revenue by category and verify 6000, 12000, and 10000. Produce January/February actual-versus-target rows and explain why averaging line values cannot recover average order value.

Next: [Lookups](05-lookups-and-matching.md).
