# 05 · Lookups, Matching, and Reconciliation

A lookup maps a key to a value. Before choosing a function, decide whether keys are unique, whether the match must be exact, and what should happen when a key is absent or repeated.

## Contents

- [XLOOKUP](#xlookup)
- [INDEX and MATCH](#index-and-match)
- [VLOOKUP and HLOOKUP](#vlookup-and-hlookup)
- [Approximate matching](#approximate-matching)
- [Multiple matches and criteria](#multiple-matches-and-criteria)
- [Reconciliation and practice](#reconciliation-and-practice)

## XLOOKUP

```excel
=XLOOKUP("P20",Products[ProductID],Products[Product],"Missing product")
=XLOOKUP("P99",Products[ProductID],Products[ListPriceCents],"Missing product")
```

Results: `Python course` and `Missing product`. XLOOKUP defaults to an exact match and can return a column on either side of the key. It returns one matching position unless the return array spans several columns.

Core syntax: `XLOOKUP(lookup_value,lookup_array,return_array,[if_not_found],[match_mode],[search_mode])`. Match modes include exact (0), exact/next smaller (-1), exact/next larger (1), and wildcard (2). Search mode -1 searches from last to first. Binary search modes require correctly sorted lookup arrays; otherwise results can be invalid.

XLOOKUP is unavailable in Excel 2016/2019; use INDEX/MATCH or VLOOKUP there. Check actual feature availability, not just the broad Applies To banner on a support page. [Microsoft XLOOKUP documentation](https://support.microsoft.com/en-us/excel/functions/xlookup-function)

Do not replace historical Sales prices with today's Products list prices. A lookup may identify a discrepancy without proving which value should be changed.

## INDEX and MATCH

```excel
=INDEX(Products[Product],MATCH("P20",Products[ProductID],0))
```

MATCH with 0 finds an exact position; INDEX returns a value at that position. This returns `Python course` and works in many older Excel versions. Missing keys produce #N/A; IFNA can handle an expected missing match without concealing unrelated errors.

For a two-way lookup, use one MATCH for the row and another for the column. For example, if B2:D4 contains a pricing matrix, A2:A4 product names, B1:D1 region labels, F2 a product, and G2 a region:

```excel
=INDEX(B2:D4,MATCH(F2,A2:A4,0),MATCH(G2,B1:D1,0))
```

This is a separate miniature layout, not the Sales fixture. Both headers need unambiguous unique labels.

## VLOOKUP and HLOOKUP

```excel
=VLOOKUP("P20",Products[[ProductID]:[ListPriceCents]],4,FALSE)
```

Result: 3000. VLOOKUP searches the first column of its range and returns an indexed column to its right. FALSE requests exact matching. Omitting the last argument defaults to approximate behavior and can produce plausible but wrong results for IDs.

VLOOKUP's numeric column index can become wrong when the lookup range changes. INDEX/MATCH or XLOOKUP makes the returned column explicit. HLOOKUP uses the analogous horizontal layout; it is useful for some legacy models but not a reason to store growing transaction records across columns.

## Approximate matching

Use the optional `Tiers` table from [setup](00-practice-workbook.md). LowerBound contains 0, 5000, 10000, and Discount contains 0%, 5%, 10%.

```excel
=XLOOKUP(7500,Tiers[LowerBound],Tiers[Discount],"Below range",-1)
=VLOOKUP(7500,Tiers[[LowerBound]:[Discount]],2,TRUE)
```

Both return 5%. VLOOKUP approximate matching requires ascending thresholds. XLOOKUP's default linear search with next-smaller matching does not have that same sorting requirement, but sorted unique thresholds are much easier to audit; binary search does require sort order.

Test values below the first threshold, exactly on each boundary, just above it, and above the last. Define whether brackets include their lower or upper boundary. Progressive taxation or tiered billing may apply different rates to different portions, so a single lookup rate multiplied by the entire amount may be the wrong model.

## Multiple matches and criteria

XLOOKUP returns one matching record, not the sum of all matches. Use SUMIFS for totals, FILTER for every matching row, or Power Query merge for relational joins.

Modern Excel example returning all paid North sales:

```excel
=FILTER(Sales[[SaleID]:[Status]],(Sales[Region]="North")*(Sales[Status]="Paid"),"No matches")
```

Place this outside a Table with enough empty space. It returns five lines. For a multiple-criteria single lookup, a Boolean lookup array can work in modern Excel, but uniqueness must still be checked. Concatenating arbitrary fields without an unambiguous separator can create key collisions.

A merge can multiply rows if the right-side key has duplicates. A lookup returning the first match can hide the same underlying data-quality problem. Validate keys before either operation.

## Reconciliation and practice

In a Sales helper column:

```excel
=COUNTIF(Products[ProductID],[@ProductID])
```

Every row should return 1: zero means an unmatched key; greater than one means an ambiguous catalog. Clean whitespace and types deliberately. Text 0010 may be a different identifier from numeric 10; do not strip meaningful zeros to force a match.

Practice: find P40's price, flag P99 as missing, compare historical and catalog prices, and intentionally duplicate a Products key to observe the difference between first-match lookup and a merge. Restore the baseline afterward.

Next: [Text, dates, and time](06-text-dates-and-times.md).
