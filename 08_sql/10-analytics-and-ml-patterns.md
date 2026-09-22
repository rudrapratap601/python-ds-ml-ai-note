# SQL for Analytics and Machine Learning

Use the [practice database](00-practice-database.md). Before writing an analytical query, state its **grain**: what does one output row represent? Decide which events qualify, how time is measured, and how missing observations should behave.

## Contents

- [Revenue and growth](#revenue-and-growth)
- [Customer features and percentages](#customer-features-and-percentages)
- [Cohort retention](#cohort-retention)
- [Gaps and islands](#gaps-and-islands)
- [Customers satisfying every requirement](#customers-satisfying-every-requirement)
- [Point-in-time features and leakage](#point-in-time-features-and-leakage)
- [Warehouse and pipeline design](#warehouse-and-pipeline-design)
- [Practice](#practice)

## Revenue and growth

Revenue here means the sum of historical item prices on currently paid orders. It excludes pending/cancelled orders and does not model refunds, tax, shipping, or currency conversion. Define those before applying this pattern to real financial data.

Aggregate to one row per month **before** comparing months. This SQLite example returns January 12,000 cents and February 16,000 cents, with February growth approximately 33.33%.

<!-- dialect: sqlite -->
```sql
WITH monthly AS (
    SELECT strftime('%Y-%m', o.order_date) AS month,
           SUM(i.quantity * i.unit_price_cents) AS revenue_cents
    FROM orders AS o
    JOIN order_items AS i ON i.order_id = o.order_id
    WHERE o.status = 'paid'
    GROUP BY strftime('%Y-%m', o.order_date)
), compared AS (
    SELECT month, revenue_cents,
           LAG(revenue_cents) OVER (ORDER BY month) AS previous_cents
    FROM monthly
)
SELECT month, revenue_cents, previous_cents,
       100.0 * (revenue_cents - previous_cents)
           / NULLIF(previous_cents, 0) AS growth_percent
FROM compared
ORDER BY month;
```

`LAG` finds the previous *available row*. If March has no sales, an April row might compare against February. For calendar-month growth, build a date/calendar table, left join revenue onto every month, and explicitly choose whether missing revenue means zero or unavailable. Avoid presenting missing source data as zero activity.

For PostgreSQL, use `date_trunc('month', o.order_date)::date` to form a month key. Filter a timestamp range before grouping so an index on the original timestamp can remain useful. Date boundaries need an agreed business timezone. [PostgreSQL date functions](https://www.postgresql.org/docs/current/functions-datetime.html)

## Customer features and percentages

Start from customers when customers with no qualifying activity must remain. Aggregate items to order totals first; otherwise an order with two items would count twice.

```sql
WITH order_totals AS (
    SELECT o.order_id, o.customer_id,
           SUM(i.quantity * i.unit_price_cents) AS total_cents
    FROM orders AS o
    JOIN order_items AS i ON i.order_id = o.order_id
    WHERE o.status = 'paid'
    GROUP BY o.order_id, o.customer_id
), customer_totals AS (
    SELECT c.customer_id, c.customer_name,
           COUNT(t.order_id) AS paid_orders,
           COALESCE(SUM(t.total_cents), 0) AS revenue_cents
    FROM customers AS c
    LEFT JOIN order_totals AS t ON t.customer_id = c.customer_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT customer_id, customer_name, paid_orders, revenue_cents,
       1.0 * revenue_cents / NULLIF(paid_orders, 0) AS average_order_cents,
       100.0 * revenue_cents
           / NULLIF(SUM(revenue_cents) OVER (), 0) AS revenue_share_percent
FROM customer_totals
ORDER BY customer_id;
```

Dev contributes 50% of paid revenue. Noor and Ishan have zero paid orders; their average order value is NULL because no denominator exists. `COALESCE` is a business decision, not a universal cleanup step.

Common analytical mistakes:

| Mistake | Better approach |
|---|---|
| Average customer averages to find overall order average | Divide total order revenue by total order count |
| Join two one-to-many tables and sum their amounts | Aggregate each child independently to the parent grain first |
| Use DISTINCT to hide an unexplained duplicate join | Check join keys and expected cardinality |
| Count event rows as people | Deduplicate user/event-period pairs or count distinct users |
| Mix currencies in one SUM | Convert with a documented rate/date or group by currency |

## Cohort retention

A cohort groups entities by a common starting event. This example uses **first paid purchase month**, not signup month. A retained customer has at least one paid purchase in a later month; multiple purchases in a month count once.

<!-- dialect: sqlite -->
```sql
WITH activity AS (
    SELECT DISTINCT customer_id, strftime('%Y-%m', order_date) AS month
    FROM orders
    WHERE status = 'paid'
), first_purchase AS (
    SELECT customer_id, MIN(month) AS cohort_month
    FROM activity
    GROUP BY customer_id
), cohort_sizes AS (
    SELECT cohort_month, COUNT(*) AS cohort_size
    FROM first_purchase
    GROUP BY cohort_month
)
SELECT f.cohort_month, a.month AS activity_month, s.cohort_size,
       COUNT(*) AS active_customers,
       100.0 * COUNT(*) / s.cohort_size AS retention_percent
FROM first_purchase AS f
JOIN activity AS a ON a.customer_id = f.customer_id
JOIN cohort_sizes AS s ON s.cohort_month = f.cohort_month
GROUP BY f.cohort_month, a.month, s.cohort_size
ORDER BY f.cohort_month, a.month;
```

Expected: January cohort size 2 has 2 active customers in January and February. February cohort size 1 has 1 active customer in February. This tiny fixture is for mechanics, not conclusions about business retention.

The query omits months with zero activity. For a complete retention matrix, cross join cohorts to eligible calendar months and left join activity. Future periods are **not yet observed**, rather than zero retention. Compare cohorts at the same age and avoid incomplete current periods.

## Gaps and islands

An island is a consecutive run. For distinct consecutive dates, subtracting the row number from the day number produces a constant within each run.

<!-- dialect: sqlite -->
```sql
WITH activity(day) AS (
    VALUES ('2026-01-01'), ('2026-01-02'), ('2026-01-04'),
           ('2026-01-05'), ('2026-01-06')
), numbered AS (
    SELECT day, julianday(day)
           - ROW_NUMBER() OVER (ORDER BY day) AS island_key
    FROM activity
)
SELECT MIN(day) AS start_day, MAX(day) AS end_day, COUNT(*) AS days
FROM numbered
GROUP BY island_key
ORDER BY start_day;
```

Results: January 1–2 (2 days), January 4–6 (3 days). Deduplicate dates first. For per-user streaks, partition the row number by user and group by user plus island key. For sessionization, use `LAG(timestamp)` to identify a gap beyond a threshold, then a cumulative SUM of the new-session flag; date subtraction and interval syntax depend on the engine.

## Customers satisfying every requirement

Relational division asks “who satisfies all items in this required set?” Double `NOT EXISTS` expresses “there is no required product this customer has not bought.”

```sql
WITH required_products(product_id) AS (VALUES (10), (20))
SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM required_products AS r
    WHERE NOT EXISTS (
        SELECT 1
        FROM orders AS o
        JOIN order_items AS i ON i.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
          AND o.status = 'paid'
          AND i.product_id = r.product_id
    )
)
ORDER BY c.customer_id;
```

Returns Asha and Dev. If the required set is empty, every customer qualifies mathematically; add an `EXISTS` check on the required set if your product requires a nonempty requirement list.

## Point-in-time features and leakage

Training features must contain only information that was available **at prediction time**. Filtering by event date alone may be insufficient: an event could arrive late, a status could be updated later, or a customer attribute could change.

This illustrates the cutoff mechanism using the simple fixture. It is **not** a production point-in-time guarantee because the fixture stores current order status rather than historical status versions.

```sql
WITH prediction_rows(customer_id, cutoff) AS (
    VALUES (1, '2026-02-01'), (2, '2026-02-01'), (3, '2026-02-01')
), order_totals AS (
    SELECT o.order_id, o.customer_id, o.order_date, o.status,
           SUM(i.quantity * i.unit_price_cents) AS total_cents
    FROM orders AS o
    JOIN order_items AS i ON i.order_id = o.order_id
    GROUP BY o.order_id, o.customer_id, o.order_date, o.status
)
SELECT p.customer_id, p.cutoff,
       COUNT(t.order_id) AS historical_paid_orders,
       COALESCE(SUM(t.total_cents), 0) AS historical_revenue_cents
FROM prediction_rows AS p
LEFT JOIN order_totals AS t
    ON t.customer_id = p.customer_id
   AND t.order_date < p.cutoff
   AND t.status = 'paid'
GROUP BY p.customer_id, p.cutoff
ORDER BY p.customer_id;
```

Expected revenues: 6,000, 6,000, and 0. A half-open historical interval excludes events exactly at the prediction cutoff.

For a real feature pipeline:

1. Define the entity and prediction timestamp as the row key.
2. Filter both event time and availability/ingestion time as required by the prediction contract.
3. Join versioned attributes using `valid_from <= cutoff AND cutoff < valid_to`, with an explicit rule for an open-ended `valid_to`.
4. Build labels from a separate future outcome interval; do not join those outcomes into features.
5. Respect label-maturity delays and prevent the same entity leaking across evaluation splits when appropriate.
6. Assert one output row per prediction key, acceptable missingness, and correct temporal boundaries.
7. Version source snapshots, transformations, and timezone assumptions with the model experiment.

## Warehouse and pipeline design

| Concept | Purpose and practical concern |
|---|---|
| Fact table | Events or measurements at a declared grain, such as one order line |
| Dimension table | Descriptive entities, such as customers or products |
| Star schema | Facts joined to dimensions for understandable analytical queries |
| Surrogate key | Warehouse-generated key; retain source/business key for lineage |
| Slowly changing dimension type 1 | Overwrite attributes; loses old values |
| Slowly changing dimension type 2 | Insert attribute versions with validity intervals; prevent overlaps |
| Snapshot fact | State measured periodically, such as daily inventory |
| Incremental load | Process new/changed records; handle late arrivals and updates |
| Idempotent load | Retrying the same input does not create extra logical records |
| Watermark | Progress boundary; event-time-only watermarks can miss late data |

Some measures are additive across all dimensions (units sold); others are only partly additive (inventory across products but not across time); ratios often require recomputing from their numerator and denominator. Keep raw source records and a lineage trail so aggregate errors can be traced.

## Practice

1. Add a calendar month with no orders and make revenue show zero for it.
2. Compute paid revenue per product without excluding products that have never sold.
3. Add a future order and verify that historical features do not change.
4. Explain why changing an old order from pending to paid can still change the illustrative historical features.
5. Build one row per customer containing order count, revenue, and most recent paid order date.

Next: [Practice problems and solutions](11-practice-problems-and-solutions.md).
