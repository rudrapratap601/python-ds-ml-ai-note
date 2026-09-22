# 3. Joins, Aggregation, and Set Operations

> **Goal:** Combine tables without multiplying facts accidentally, then summarize at the intended grain. Load the [practice data](00-practice-database.md) first.

## Contents

- [Join types](#join-types)
- [Outer joins and filter placement](#outer-joins-and-filter-placement)
- [Cardinality and fanout](#cardinality-and-fanout)
- [GROUP BY and HAVING](#group-by-and-having)
- [Conditional aggregation](#conditional-aggregation)
- [Set operations](#set-operations)
- [Grouping sets and reporting totals](#grouping-sets-and-reporting-totals)
- [Practice](#practice)

## Join types

| Join | Result | Typical use |
|---|---|---|
| INNER | Matching combinations | Orders with their customers |
| LEFT | All left rows plus matches; unmatched right columns become NULL | Customers including those without orders |
| RIGHT | Symmetric to LEFT | Often clearer to reverse tables and use LEFT |
| FULL OUTER | Matched and unmatched rows from both sides | Reconciliation |
| CROSS | Every left/right pair | Small grids or scenario combinations |
| Self join | One table under two aliases | Employee and manager |

Support for RIGHT/FULL joins varies by engine/version; MySQL 8.4 has no native FULL OUTER JOIN. A join condition can express equality or another relationship, such as a range. Without the intended condition, a large Cartesian result can appear.

```sql
SELECT o.order_id, c.customer_name, o.status
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id
ORDER BY o.order_id;

SELECT e.employee_name, m.employee_name AS manager_name
FROM employees AS e
LEFT JOIN employees AS m ON m.employee_id = e.manager_id
ORDER BY e.employee_id;
```

Aliases disambiguate repeated column names. Qualify join keys explicitly. Avoid NATURAL JOIN in stable application interfaces because adding another same-named column can silently change the join condition.

## Outer joins and filter placement

To preserve every customer while counting paid orders, filter the right side in ON:

```sql
SELECT c.customer_id, c.customer_name, COUNT(o.order_id) AS paid_orders
FROM customers AS c
LEFT JOIN orders AS o
  ON o.customer_id = c.customer_id
 AND o.status = 'paid'
GROUP BY c.customer_id, c.customer_name
ORDER BY c.customer_id;
```

Expected counts: **3, 2, 1, 0, 0**. Moving `o.status = 'paid'` into WHERE would discard unmatched NULL-extended rows and lose customers with no paid order.

`COUNT(*)` counts every joined result row, including the unmatched placeholder row. `COUNT(o.order_id)` counts non-null order IDs, producing zero for unmatched customers.

ON determines matching; WHERE filters the resulting rows. They are not interchangeable for outer joins. To find missing matches, use a non-nullable right-side key with `WHERE o.order_id IS NULL`, or use NOT EXISTS.

## Cardinality and fanout

Joining one order to two lines produces two rows. If the same order also joins to three payments, a naive join across both child tables may produce six rows. Summing order-level or payment-level amounts then overcounts.

Aggregate each child table to the intended parent grain before joining independent one-to-many relationships:

```sql
WITH order_totals AS (
    SELECT order_id, SUM(quantity * unit_price_cents) AS total_cents
    FROM order_items
    GROUP BY order_id
)
SELECT o.order_id, c.customer_name, t.total_cents
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id
JOIN order_totals AS t ON t.order_id = o.order_id
WHERE o.status = 'paid'
ORDER BY o.order_id;
```

This has one row per paid order. The CTE is explained in the next chapter. An INNER join here excludes any order without lines; use LEFT plus an explicit missing-total policy if such orders should appear.

Do not fix fanout with `SUM(DISTINCT amount)`: two legitimate orders can have the same amount. Deduplicate by the actual key/relationship, not merely by equal numeric values.

## GROUP BY and HAVING

```sql
SELECT c.region,
       COUNT(DISTINCT o.order_id) AS paid_orders,
       SUM(oi.quantity * oi.unit_price_cents) AS revenue_cents
FROM customers AS c
JOIN orders AS o ON o.customer_id = c.customer_id
JOIN order_items AS oi ON oi.order_id = o.order_id
WHERE o.status = 'paid'
GROUP BY c.region
HAVING SUM(oi.quantity * oi.unit_price_cents) >= 10000
ORDER BY c.region;
```

Expected: North has 4 paid orders and 14,000 cents; South has 2 and 14,000 cents. Counting distinct order IDs is appropriate because the input is at line grain, while summing line revenue is already at its correct grain.

| Function | Missing-data behavior to remember |
|---|---|
| `COUNT(*)` | Counts rows |
| `COUNT(column)` | Counts non-null values |
| `COUNT(DISTINCT column)` | Counts distinct non-null values |
| `SUM`, `AVG`, `MIN`, `MAX` | Generally ignore NULL inputs |

Most aggregates except COUNT return NULL for an empty input. `AVG(x)` excludes NULL observations; `AVG(COALESCE(x, 0))` changes the denominator and meaning. For portable queries, include every selected nonaggregated expression in GROUP BY rather than relying on vendor-specific allowances.

WHERE filters input rows before grouping; HAVING filters groups after aggregation. Use WHERE for ordinary row predicates so the intention is clear and fewer rows need later processing.

## Conditional aggregation

```sql
SELECT customer_id,
       SUM(CASE WHEN status = 'paid' THEN 1 ELSE 0 END) AS paid_count,
       SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END) AS pending_count,
       SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled_count
FROM orders
GROUP BY customer_id
ORDER BY customer_id;
```

This only includes customers present in orders. Use an outer join if the report must include customers with no orders.

`COUNT(CASE WHEN condition THEN 1 END)` counts matching rows. Adding `ELSE 0` to that COUNT would count nonmatching rows too, because zero is non-null. PostgreSQL/SQLite also support aggregate `FILTER (WHERE ...)`; CASE is more broadly portable.

## Set operations

```sql
SELECT customer_id FROM orders WHERE status = 'paid'
UNION
SELECT customer_id FROM orders WHERE status = 'pending'
ORDER BY customer_id;

SELECT customer_id FROM customers
EXCEPT
SELECT customer_id FROM orders
ORDER BY customer_id;
```

The first returns IDs 1–4. The second returns 5. UNION removes duplicate complete rows; UNION ALL preserves them and avoids the requirement to deduplicate. INTERSECT returns shared rows; EXCEPT returns rows in the left result but not the right.

Inputs need the same number of columns and compatible types, aligned by position, not by matching aliases. Parentheses and operator precedence matter in mixed set expressions. ALL variants and supported syntax differ across engines; consult the [dialect chapter](12-dialects-and-command-reference.md).

## Grouping sets and reporting totals

**PostgreSQL example:** Produce regional/status subtotals and a grand total without manually repeating the query.

<!-- dialect: postgresql -->
```sql
SELECT c.region, o.status, COUNT(*) AS orders,
       GROUPING(c.region) AS region_is_aggregate,
       GROUPING(o.status) AS status_is_aggregate
FROM customers AS c
JOIN orders AS o ON o.customer_id = c.customer_id
GROUP BY GROUPING SETS ((c.region, o.status), (c.region), ())
ORDER BY region_is_aggregate, c.region, status_is_aggregate, o.status;
```

ROLLUP creates hierarchical subtotals; CUBE creates all grouping combinations. GROUPING distinguishes a placeholder NULL created by aggregation from a genuine NULL key. Availability differs; these are not SQLite features.

## Practice

Find never-ordered products; count all customers by paid-order count including zero; show employees without a manager; calculate average paid order value by aggregating order totals before averaging; explain how joining two child tables can multiply rows.

Reference: [Table expressions and grouping](https://www.postgresql.org/docs/current/queries-table-expressions.html). Next: [Subqueries and CTEs](04-subqueries-ctes-and-recursion.md).
