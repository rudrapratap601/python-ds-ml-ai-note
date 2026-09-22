# 6. Functions, Dates, and Data Cleaning

> **Goal:** Transform data without silently changing its meaning. Function names and type-conversion rules are among the least portable parts of SQL.

## Contents

- [Numeric and text functions](#numeric-and-text-functions)
- [Casting and missing values](#casting-and-missing-values)
- [Dates, timestamps, and timezones](#dates-timestamps-and-timezones)
- [Date arithmetic by dialect](#date-arithmetic-by-dialect)
- [Cleaning and deduplication](#cleaning-and-deduplication)
- [Quality checks and practice](#quality-checks-and-practice)

## Numeric and text functions

```sql
SELECT product_name, ABS(price_cents - 2000) AS distance_from_2000,
       ROUND(price_cents / 100.0, 2) AS price_units
FROM products
ORDER BY product_id;

SELECT customer_id, UPPER(customer_name) AS display_name,
       LOWER(TRIM(email)) AS normalized_email
FROM customers
ORDER BY customer_id;
```

| Task | Common tools | Portability concern |
|---|---|---|
| Absolute/rounding | ABS, ROUND, CEIL/CEILING, FLOOR | Function names, numeric result types, tie rounding |
| Case/whitespace | LOWER, UPPER, TRIM | Unicode/collation and what counts as whitespace |
| Substring | SUBSTRING or SUBSTR | Syntax and starting positions |
| Text length | CHAR_LENGTH, LENGTH, LEN | Bytes versus characters; trailing spaces |
| Concatenation | `\|\|`, CONCAT, SQL Server `+` | NULL behavior and engine settings |
| Replacement | REPLACE | Literal versus regex operations |
| Pattern extraction | Regex functions | Highly dialect-specific |

Do not treat a formatted numeric string as a number for later sorting/aggregation. Keep numeric values numeric until presentation. Rounding each row before summing can differ from rounding the final sum; choose the required rule explicitly.

## Casting and missing values

```sql
SELECT customer_id,
       COALESCE(NULLIF(TRIM(email), ''), 'missing') AS email_display
FROM customers
ORDER BY customer_id;

SELECT CAST(3 AS DECIMAL(10, 2)) / 2.0 AS fractional_value;
```

CAST changes the type, but invalid input behavior varies. PostgreSQL generally rejects invalid casts; SQLite may coerce unexpected text into a numeric value such as zero. SQL Server has TRY_CAST/TRY_CONVERT for a nullable failed conversion; other engines have different safe-cast facilities.

Do not silently turn failed conversions into plausible data. Stage the raw value, parse into a separate typed field, and report rejects. A database's permissive conversion is not a data-quality policy.

Empty string, whitespace-only text, NULL, numeric zero, and the literal string `'NULL'` are distinct until you deliberately normalize them. Some other engines, notably Oracle, have different empty-string behavior; these notes do not assume it.

## Dates, timestamps, and timezones

Separate these concepts:

- A calendar date, such as a birthday.
- A local wall-clock time, which may be ambiguous during daylight-saving transitions.
- An instant on a global timeline.
- A duration or calendar interval, such as one month.

Adding one month is not the same as adding 30 days. A day can have a different number of local hours around daylight-saving changes. Use the source timezone and the reporting timezone deliberately.

PostgreSQL `timestamp with time zone` represents an instant and displays it according to session timezone; it does not preserve the original named timezone label. SQL Server `timestamp` means rowversion, **not** a date/time field. MySQL TIMESTAMP and DATETIME have different conversion/range semantics. SQLite has no dedicated native date storage class.

For time filtering, use typed parameters or correctly typed literals and half-open ranges:

```sql
SELECT order_id, order_date
FROM orders
WHERE order_date >= '2026-02-01'
  AND order_date < '2026-03-01'
ORDER BY order_date, order_id;
```

This returns 105, 106, 107, 108. With timestamps, calculate those boundaries in the intended timezone instead of assuming a date string always means UTC midnight.

## Date arithmetic by dialect

**PostgreSQL:**

<!-- dialect: postgresql -->
```sql
SELECT DATE '2026-02-01' + INTERVAL '7 days' AS next_week,
       DATE_TRUNC('month', TIMESTAMP '2026-02-05 12:00:00') AS month_start,
       EXTRACT(YEAR FROM DATE '2026-02-05') AS year_number;
```

**MySQL:**

<!-- dialect: mysql -->
```sql
SELECT DATE_ADD('2026-02-01', INTERVAL 7 DAY) AS next_week,
       DATE_FORMAT('2026-02-05', '%Y-%m-01') AS month_start_text,
       YEAR('2026-02-05') AS year_number;
```

**SQL Server:**

<!-- dialect: sqlserver -->
```sql
SELECT DATEADD(day, 7, CAST('2026-02-01' AS date)) AS next_week,
       DATEFROMPARTS(2026, 2, 1) AS month_start,
       YEAR(CAST('2026-02-05' AS date)) AS year_number;
```

**SQLite:**

<!-- dialect: sqlite -->
```sql
SELECT date('2026-02-01', '+7 days') AS next_week,
       date('2026-02-05', 'start of month') AS month_start,
       strftime('%Y', '2026-02-05') AS year_text;
```

The outputs need not have identical types. MySQL DATE_FORMAT and SQLite strftime return text in these examples. SQL Server DATEDIFF counts specified boundary crossings, which is not always the same as a continuous elapsed-duration calculation.

A calendar table is often preferable for business-day logic, holidays, fiscal periods, and zero-filled reporting. Store the reporting rules instead of rebuilding them inconsistently in each dashboard.

## Cleaning and deduplication

Keep raw staging data separate from the cleaned model. Define the duplicate key and winner rule first: duplicate entire rows, duplicate business IDs, and multiple legitimate events are different problems.

The following self-contained CTE models repeated deliveries of the same event:

```sql
WITH raw_events AS (
    SELECT 1 AS ingest_id, 'event-a' AS event_key, '2026-02-01 10:00:00' AS received_at
    UNION ALL SELECT 2, 'event-a', '2026-02-01 10:05:00'
    UNION ALL SELECT 3, 'event-b', '2026-02-01 10:06:00'
), ranked AS (
    SELECT ingest_id, event_key, received_at,
           ROW_NUMBER() OVER (
               PARTITION BY event_key ORDER BY received_at DESC, ingest_id DESC
           ) AS position
    FROM raw_events
)
SELECT ingest_id, event_key, received_at
FROM ranked
WHERE position = 1
ORDER BY event_key;
```

Expected winners: ingest IDs **2 and 3**. This is a read-only cleaned view, not an irreversible delete. A later deletion or merge should be separately scoped, transactionally handled, and tested against concurrent writes.

## Quality checks and practice

```sql
SELECT email, COUNT(*) AS occurrences
FROM customers
WHERE email IS NOT NULL
GROUP BY email
HAVING COUNT(*) > 1;

SELECT o.order_id
FROM orders AS o
WHERE NOT EXISTS (
    SELECT 1 FROM order_items AS oi WHERE oi.order_id = o.order_id
)
ORDER BY o.order_id;
```

Both return no rows in the baseline. Check uniqueness, required values, allowed ranges, foreign-key completeness, freshness, and expected row-count changes. Do not call a dataset clean merely because the query executes.

Practice: standardize blank emails while keeping an audit field; compare month grouping across dialects; deduplicate with a deterministic tie-breaker; validate an imported date without silently replacing failures; design a calendar table with business days.

References: [PostgreSQL date functions](https://www.postgresql.org/docs/current/functions-datetime.html), [SQLite date functions](https://www.sqlite.org/lang_datefunc.html). Next: [Optimization](07-indexes-and-query-optimization.md).
