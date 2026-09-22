# 06 · Text, Dates, Times, and Conversion

Imported data often fails because types or encodings are wrong, not because the formula is complicated. Preserve the raw input while developing a cleanup rule.

## Contents

- [Cleaning and extracting text](#cleaning-and-extracting-text)
- [Splitting and joining](#splitting-and-joining)
- [Conversion and identifiers](#conversion-and-identifiers)
- [Dates and periods](#dates-and-periods)
- [Time and workdays](#time-and-workdays)
- [Practice](#practice)

## Cleaning and extracting text

```excel
=TRIM("  Asha   Rao  ")
=LEFT("P20-Learning",3)
=RIGHT("P20-Learning",8)
=MID("S001",2,3)
=LEN("Excel")
=SUBSTITUTE("North-West","-"," ")
```

Results: `Asha Rao`, `P20`, `Learning`, `001`, 5, and `North West`. TRIM removes ordinary leading/trailing spaces and reduces repeated ordinary spaces inside text. It does not remove every Unicode whitespace character.

A common imported nonbreaking-space cleanup is `=TRIM(SUBSTITUTE(A2,CHAR(160)," "))`. CLEAN removes certain control characters, not all invisible Unicode characters. Inspect the actual problematic character rather than adding endless replacements blindly.

FIND is case-sensitive; SEARCH is generally case-insensitive and supports wildcards. UPPER/LOWER change case; PROPER changes word capitalization but can damage intentional brand names or identifiers. EXACT can test case-sensitive text equality.

## Splitting and joining

Modern functions, where supported:

```excel
=TEXTBEFORE("P20-Learning","-")
=TEXTAFTER("P20-Learning","-")
=TEXTSPLIT("North,South,West",",")
=TEXTJOIN(", ",TRUE,"North","","West")
```

TEXTSPLIT spills, so enter it outside a Table. Older alternatives include Text to Columns, LEFT/MID/FIND, or Power Query Split Column. TEXTBEFORE/TEXTAFTER/TEXTSPLIT require a newer function set than many legacy installations; see the [function catalog](https://support.microsoft.com/en-us/excel/excel-functions-alphabetical).

Text to Columns can overwrite neighboring cells if its destination is occupied. Flash Fill learns a pattern from examples, but its output is static: source edits do not automatically recompute those values. Test exceptions before treating an inferred pattern as a reliable transformation.

## Conversion and identifiers

VALUE parses a text number using locale-dependent conventions. NUMBERVALUE lets you state separators, for example `=NUMBERVALUE("1.234,50",",",".")` gives 1234.5. Ambiguous values like `1,234` need a source-format contract.

TEXT formats a number into **text**: `=TEXT(DATE(2026,2,1),"yyyy-mm")` returns `2026-02`. Use it for presentation, not as a substitute for a true date key in a model.

Keep IDs as text when leading zeros, long digit sequences, or letters are meaningful. Formatting a numeric ID with `00000` only changes its appearance; exporting or referencing it may expose the underlying shorter number.

## Dates and periods

```excel
=DATE(2026,2,1)
=YEAR(DATE(2026,2,5))
=MONTH(DATE(2026,2,5))
=DAY(DATE(2026,2,5))
=EOMONTH(DATE(2026,2,5),0)
=EDATE(DATE(2026,1,31),1)
```

The last two return February 28, 2026. Format date results as dates. DATE constructs a date from numeric components and normalizes out-of-range components; it is not, by itself, a strict validator of user-entered day/month combinations.

For an ISO text date in A2 with exactly `yyyy-mm-dd`, `=DATE(VALUE(LEFT(A2,4)),VALUE(MID(A2,6,2)),VALUE(RIGHT(A2,2)))` parses components, but still needs validation if impossible dates must be rejected. Power Query type conversion with an explicit locale is often preferable for repeated imports.

TODAY returns the current date and NOW a current date/time when recalculated. They are not permanent entry timestamps. Use a controlled ReportDate input for reproducible reports and a real logged timestamp for an audit record.

Excel supports 1900 and 1904 date systems; dates between the systems differ by 1462 days. The 1900 system also retains a historical fictitious February 29, 1900 for compatibility. Avoid reasoning about early serials as though they perfectly match the Gregorian calendar. [Microsoft date systems](https://support.microsoft.com/en-us/office/date-systems-in-excel-e7fe7167-48a9-4b96-bb53-5612a800b487)

## Time and workdays

```excel
=TIME(9,30,0)
=DATE(2026,2,2)-DATE(2026,2,1)
=NETWORKDAYS(DATE(2026,2,2),DATE(2026,2,6))
=WORKDAY(DATE(2026,2,6),1)
```

Results represent 09:30, 1 day, 5 workdays, and February 9, 2026. NETWORKDAYS counts eligible start/end dates inclusively. Supply a holiday range when appropriate; INTL variants support custom weekends.

Subtract actual date-time values for elapsed duration and multiply by 24 for hours. Use `[h]:mm` to display durations over 24 hours. For clock times alone, MOD(end-start,1) can handle an assumed single overnight crossing, but it silently assumes a duration shorter than 24 hours.

Ordinary Excel date serials do not carry a timezone. A local wall-clock timestamp can be ambiguous during daylight-saving changes. Preserve UTC/offset/timezone information during imports and perform timezone conversions in an appropriate data pipeline.

## Practice

Clean a name containing a nonbreaking space, split a product code, calculate a next-month boundary, and compare a text month label with a real month-start date. Explain why NOW is unsuitable for recording an immutable payment timestamp.

Next: [Dynamic arrays, LET, and LAMBDA](07-dynamic-arrays-let-and-lambda.md).
