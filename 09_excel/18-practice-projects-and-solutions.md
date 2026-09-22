# 18 · Practice Problems, Worked Answers, and Projects

Use the [shared workbook](00-practice-workbook.md). Try each problem before reading its answer. State the population, grain, and units before selecting a function.

## Contents

- [Beginner exercises](#beginner-exercises)
- [Intermediate exercises](#intermediate-exercises)
- [Advanced exercises](#advanced-exercises)
- [Portfolio projects](#portfolio-projects)
- [Interview review](#interview-review)

## Beginner exercises

### 1. Calculate line revenue

In Sales[RevenueCents]:

```excel
=[@Quantity]*[@UnitPriceCents]
```

S001 revenue is 2000 cents. All-status total is 36000. Do not substitute a catalog's current price for the historical line price.

### 2. Count paid lines

```excel
=COUNTIF(Sales[Status],"Paid")
```

Answer: 8. This is not the number of paid orders.

### 3. Find P40's catalog price

```excel
=XLOOKUP("P40",Products[ProductID],Products[ListPriceCents],"Missing")
```

Answer: 2000 cents, even though P40 has no sales. Older alternative: `=INDEX(Products[ListPriceCents],MATCH("P40",Products[ProductID],0))`.

### 4. Compute paid revenue

```excel
=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid")
```

Answer: 28000 cents. A formatting change alone will not convert this into 280 currency units; divide by 100 for that presentation.

## Intermediate exercises

### 5. Compute February paid revenue

```excel
=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid",Sales[OrderDate],">="&DATE(2026,2,1),Sales[OrderDate],"<"&DATE(2026,3,1))
```

Answer: 16000 cents. January is 12000; month-over-month growth is `(16000-12000)/12000`, approximately 33.333333%. These are two observed months, not a forecast model.

### 6. Compute paid gross margin

```excel
=SUMIFS(Sales[ProfitCents],Sales[Status],"Paid")/SUMIFS(Sales[RevenueCents],Sales[Status],"Paid")
```

Answer: 14600/28000, approximately 52.142857%. Averaging individual line margin percentages does not generally give the correct overall margin.

### 7. Count distinct paid orders

```excel
=IF(COUNTIF(Sales[Status],"Paid")=0,0,ROWS(UNIQUE(FILTER(Sales[OrderID],Sales[Status]="Paid"))))
```

Answer: 6. For versions without dynamic arrays, use a Data Model distinct count or a paid-only deduplicated order table. Average paid order value is 28000/6, approximately 4666.666667 cents.

### 8. Return paid North lines

```excel
=FILTER(Sales[SaleID],(Sales[Region]="North")*(Sales[Status]="Paid"),"No matches")
```

Answer: S001, S002, S003, S009, S010 in source order. Their combined revenue is 14000 cents. FILTER returns rows; XLOOKUP would ordinarily return one match.

### 9. Build a monthly target report

Use Budget[MonthStart] as report month keys. In a new Budget calculated column named ActualCents:

```excel
=SUMIFS(Sales[RevenueCents],Sales[Status],"Paid",Sales[OrderDate],">="&[@MonthStart],Sales[OrderDate],"<"&EDATE([@MonthStart],1))
```

Then compute ActualCents−TargetCents and ActualCents/TargetCents−1 with a denominator policy. Expected January variance: 2000 and 20%; February: 1000 and approximately 6.666667%. Overall: 3000 and 12%, not the mean of the monthly percentages.

## Advanced exercises

### 10. Prove a merge preserves the data grain

Left-merge Sales to Products by ProductID and expand the descriptive columns. Expected: 10 rows and all-status revenue 36000. Duplicate P20 in Products on a copy and observe row multiplication. Repair the key rather than dividing the wrong total by a convenient constant.

### 11. Reconcile three implementations

Build monthly paid revenue using SUMIFS, a PivotTable, and the M query from [Power Query](11-power-query-and-m.md). Each should return January 12000 and February 16000. If one differs, inspect date types, Status filters, source ranges, and refresh state.

### 12. Test model filter context

Create the [paid DAX measures](12-data-model-power-pivot-and-dax.md). Use Products[Category] on rows and verify Books 6000, Learning 12000, Tools 10000. Apply Pending-only Status filtering and explain why KEEPFILTERS prevents the measure from replacing that selection with Paid.

### 13. Solve the what-if model

Use the layouts in [analysis](13-what-if-statistics-and-financial-functions.md). Goal Seek needs 150 units to earn 1000 profit under the stated price/cost assumptions. Solver's constrained product mix is x=3, y=4 with objective 29. Check the constraints yourself: 2×3+4=10 and 3+3×4=15.

### 14. Make automation repeatable

Run either the VBA or Office Script summary twice. Expected output remains one report, not duplicated source records. Rename a required column and verify the workflow fails visibly. A report-generation timestamp is not proof that an upstream connection refreshed.

### 15. Design edge-case checks

On a copy, introduce zero paid rows, an unknown ProductID, a duplicate SaleID, a text quantity, an invalid status, and a blocked spill range. For each case record expected behavior: reject, report an exception, or return a defined empty result. A generic IFERROR(...,0) is not an adequate answer to all six cases.

## Portfolio projects

### Project A: Sales dashboard

Deliver a documented workbook with controlled inputs, monthly actual/target results, category/region charts, filter controls, and quality checks. Record metric definitions, refresh procedure, supported Excel version, and a screenshot/PDF intended for readers. Reconcile every displayed total to the fixture before expanding the data.

### Project B: Monthly file consolidation

Combine several synthetic monthly CSVs through Power Query. Preserve source filenames, enforce a schema, quarantine invalid rows, detect duplicate line IDs, and demonstrate that a new month requires refresh rather than manual copy/paste.

### Project C: Inventory planning

Model products, receipts, and issues at declared grains. Calculate on-hand quantities, reorder flags, and supplier lead-time assumptions. Add validation and a Solver example only after defining capacity and demand constraints. Distinguish a stock snapshot from a sum of snapshots across time.

### Project D: ML prediction review

Import predictions generated by an external Python pipeline. Join by stable entity IDs, track prediction timestamps and model versions, and summarize errors by group without leaking target outcomes into training features. Keep model training/evaluation code and source snapshots reproducible outside ad hoc workbook edits.

## Interview review

| Question | Expected explanation |
|---|---|
| Table versus range? | Structure, expanding references, calculated columns, and source management |
| Relative versus absolute reference? | Which row/column shifts when copied |
| VLOOKUP exact-match argument? | FALSE; omitted argument uses approximate matching |
| Why are orders double counted? | Source is at line grain and OrderID repeats |
| SUMIFS versus FILTER? | Aggregate qualifying rows versus return qualifying rows |
| PivotTable versus Power Query? | Interactive summary versus repeatable transformation |
| M versus DAX? | Data preparation language versus model calculations/filter context |
| Why can a grand total differ from summed row percentages? | Ratios recompute from their own total numerator/denominator |
| VBA versus Office Scripts? | Different APIs, execution environments, and deployment capabilities |
| Does hiding a sheet secure data? | No; it changes presentation, not access rights |

Next: [Function and shortcut reference](19-reference-shortcuts-and-compatibility.md).
