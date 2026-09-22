# SQL Practice Problems and Worked Solutions

Load the [practice database](00-practice-database.md). Try each problem before reading its solution. Explain the intended output grain and NULL behavior as well as the syntax. Every solution below is a read-only query against the original fixture.

## Contents

- [Beginner problems](#beginner-problems)
- [Intermediate problems](#intermediate-problems)
- [Advanced problems](#advanced-problems)
- [Interview reasoning](#interview-reasoning)
- [A complete practice project](#a-complete-practice-project)

## Beginner problems

### 1. Find customers with no email

Use `IS NULL`, not `= NULL`. Expected customer IDs: 4, 5.

```sql
SELECT customer_id, customer_name
FROM customers
WHERE email IS NULL
ORDER BY customer_id;
```

### 2. Find paid orders in January 2026

A half-open range also works cleanly when a date column later becomes a timestamp. Expected IDs: 101, 102, 103.

```sql
SELECT order_id, customer_id, order_date
FROM orders
WHERE status = 'paid'
  AND order_date >= '2026-01-01'
  AND order_date < '2026-02-01'
ORDER BY order_id;
```

### 3. Count orders by status

Expected: cancelled 1, paid 6, pending 1.

```sql
SELECT status, COUNT(*) AS order_count
FROM orders
GROUP BY status
ORDER BY status;
```

### 4. Find customers with at least two paid orders

Filter individual rows with WHERE, then groups with HAVING. Expected: Asha 3, Dev 2.

```sql
SELECT customer_id, COUNT(*) AS paid_order_count
FROM orders
WHERE status = 'paid'
GROUP BY customer_id
HAVING COUNT(*) >= 2
ORDER BY customer_id;
```

## Intermediate problems

### 5. Find customers with no orders of any status

Expected: Ishan (5). Noor does have a pending order, so does not qualify.

```sql
SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1 FROM orders AS o WHERE o.customer_id = c.customer_id
)
ORDER BY c.customer_id;
```

### 6. Find products that have never appeared on an order

Expected: SQL workbook (40). To ask “never sold,” define which statuses qualify inside the subquery instead.

```sql
SELECT p.product_id, p.product_name
FROM products AS p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items AS i WHERE i.product_id = p.product_id
)
ORDER BY p.product_id;
```

### 7. Calculate average paid order value

Aggregate to order grain before averaging. Expected: 28,000 / 6 = approximately 4,666.67 cents. Averaging individual line amounts answers a different question.

```sql
WITH totals AS (
    SELECT o.order_id, SUM(i.quantity * i.unit_price_cents) AS cents
    FROM orders AS o
    JOIN order_items AS i ON i.order_id = o.order_id
    WHERE o.status = 'paid'
    GROUP BY o.order_id
)
SELECT AVG(1.0 * cents) AS average_order_cents
FROM totals;
```

### 8. Show paid revenue for every product

Expected product/revenue pairs: (10, 6000), (20, 12000), (30, 10000), (40, 0). Qualifying order filtering happens before the LEFT JOIN.

```sql
WITH sold AS (
    SELECT i.product_id, SUM(i.quantity * i.unit_price_cents) AS cents
    FROM order_items AS i
    JOIN orders AS o ON o.order_id = i.order_id
    WHERE o.status = 'paid'
    GROUP BY i.product_id
)
SELECT p.product_id, p.product_name, COALESCE(s.cents, 0) AS revenue_cents
FROM products AS p
LEFT JOIN sold AS s ON s.product_id = p.product_id
ORDER BY p.product_id;
```

## Advanced problems

### 9. Return everybody earning the second-highest distinct salary

Expected: Dev and Mira, both 90,000. `ROW_NUMBER() = 2` would return only one person, while `RANK()` can skip rank numbers. DENSE_RANK represents distinct salary levels.

```sql
WITH ranked AS (
    SELECT employee_id, employee_name, salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_level
    FROM employees
)
SELECT employee_id, employee_name, salary
FROM ranked
WHERE salary_level = 2
ORDER BY employee_id;
```

### 10. Find each customer's latest paid order

Break date ties by order ID for deterministic results. Expected: customer 1 order 108, customer 2 order 105, customer 3 order 107. Customers without paid orders are absent; left join this result to customers if all customers are required.

```sql
WITH ranked AS (
    SELECT order_id, customer_id, order_date,
           ROW_NUMBER() OVER (
               PARTITION BY customer_id
               ORDER BY order_date DESC, order_id DESC
           ) AS rn
    FROM orders
    WHERE status = 'paid'
)
SELECT customer_id, order_id, order_date
FROM ranked
WHERE rn = 1
ORDER BY customer_id;
```

### 11. Find employees paid above their department's average

Expected: Asha (1), Mira (3). An aggregate window retains individual employees while computing their department average.

```sql
WITH compared AS (
    SELECT employee_id, employee_name, salary,
           AVG(1.0 * salary) OVER (
               PARTITION BY department_id
           ) AS department_average
    FROM employees
)
SELECT employee_id, employee_name, salary, department_average
FROM compared
WHERE salary > department_average
ORDER BY employee_id;
```

### 12. Produce daily revenue and running revenue

Expected daily/cumulative cents: Jan 2: 5000/5000; Jan 3: 6000/11000; Jan 5: 1000/12000; Feb 1: 8000/20000; Feb 5: 8000/28000.

```sql
WITH daily AS (
    SELECT o.order_date, SUM(i.quantity * i.unit_price_cents) AS cents
    FROM orders AS o
    JOIN order_items AS i ON i.order_id = o.order_id
    WHERE o.status = 'paid'
    GROUP BY o.order_date
)
SELECT order_date, cents,
       SUM(cents) OVER (
           ORDER BY order_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS cumulative_cents
FROM daily
ORDER BY order_date;
```

### 13. Return employees managed directly or indirectly by Mira

The anchor is Mira herself. Recursive expansion visits reports, and the final filter removes the anchor. Expected: Noor and Ishan at depth 1. The depth bound prevents unlimited expansion in bad data, but a proper production hierarchy should detect cycles explicitly.

```sql
WITH RECURSIVE reports AS (
    SELECT employee_id, employee_name, 0 AS depth
    FROM employees WHERE employee_id = 3
    UNION ALL
    SELECT e.employee_id, e.employee_name, r.depth + 1
    FROM employees AS e
    JOIN reports AS r ON e.manager_id = r.employee_id
    WHERE r.depth < 20
)
SELECT employee_id, employee_name, depth
FROM reports
WHERE depth > 0
ORDER BY depth, employee_id;
```

SQL Server omits the RECURSIVE keyword and has different recursion-limit syntax; see the [dialect reference](12-dialects-and-command-reference.md).

## Interview reasoning

| Question | What a strong answer includes |
|---|---|
| WHERE versus HAVING? | Row filtering before grouping versus group filtering after aggregation |
| COUNT(*) versus COUNT(column)? | All rows versus non-NULL values; outer joins introduce NULL-extended rows |
| Why can NOT IN return no rows? | A NULL in its input can make the predicate UNKNOWN for nonmatching values |
| JOIN versus EXISTS? | JOIN combines matching rows; EXISTS tests membership without multiplying outer rows |
| GROUP BY versus windows? | Grouping reduces rows; windows annotate rows in partitions |
| Why is a query slow despite an index? | Low selectivity, wrong leading keys, expressions/casts, stale estimates, large output, or a cheaper scan |
| Can LIMIT alone give a stable page? | No; use deterministic ordering, and account for concurrent changes |
| Does a CTE cache results? | Engine/version/plan dependent; it is primarily a query-structuring construct |
| Can a foreign key prevent all bad business data? | No; use checks, appropriate uniqueness, transactions, and domain validation too |
| Is a larger isolation level always better? | It changes anomalies and contention/retry behavior; choose for the invariant |
| Is EXPLAIN ANALYZE safe on an UPDATE? | It executes the statement; use an isolated environment and understand side effects |

When optimizing an interview query, explain correctness first. State data volume, distribution, keys, and available indexes before claiming one formulation is faster.

## A complete practice project

Build a small retail reporting database with customers, products, orders, order lines, payments, and refunds. Write down table grain, money units, statuses, keys, and timezones.

1. Add constraints and insert edge cases: duplicate emails, cancelled orders, ties, missing optional fields, and customers without activity.
2. Implement revenue, order value, repeat purchase, cohort retention, and product reports with explicit definitions.
3. Add parameterized application queries and an atomic multi-step write.
4. Generate enough synthetic data to compare plans before and after a justified index.
5. Create a read-only reporting role in a disposable server and test its permissions.
6. Add assertions for row counts, unique report keys, reconciliation totals, and date cutoffs.
7. Record engine/version, schema changes, query plans, and measured timings in Git. Do not commit credentials or customer data.

Next: [Dialects and command reference](12-dialects-and-command-reference.md).
