# 4. Subqueries, CTEs, EXISTS, and Recursion

> **Goal:** Decompose queries while preserving row grain, null behavior, and execution efficiency.

## Contents

- [Subquery shapes](#subquery-shapes)
- [Scalar and derived-table subqueries](#scalar-and-derived-table-subqueries)
- [Correlated subqueries](#correlated-subqueries)
- [EXISTS, IN, and the NOT IN trap](#exists-in-and-the-not-in-trap)
- [Common table expressions](#common-table-expressions)
- [Recursive CTEs](#recursive-ctes)
- [Lateral joins and choosing a form](#lateral-joins-and-choosing-a-form)

## Subquery shapes

| Form | Result shape | Common location |
|---|---|---|
| Scalar | One value, with zero rows typically giving NULL | SELECT expression or comparison |
| Column/list | One column with multiple rows | IN |
| Table/derived table | Multiple columns and rows | FROM |
| Correlated | References an outer query | EXISTS, scalar expression, filtering |

A scalar subquery should produce at most one row. PostgreSQL and many engines raise an error if it returns more; SQLite can select the first row instead, so do not use SQLite's permissiveness as a portable contract.

## Scalar and derived-table subqueries

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY employee_id;
```

Returns Asha, Dev, and Mira. The aggregate subquery produces a single overall average.

```sql
SELECT totals.order_id, totals.total_cents
FROM (
    SELECT order_id, SUM(quantity * unit_price_cents) AS total_cents
    FROM order_items
    GROUP BY order_id
) AS totals
WHERE totals.total_cents >= 5000
ORDER BY totals.order_id;
```

A derived table lets an outer query filter a computed result. Give it a descriptive alias. This example includes every order status because no status filter was requested.

## Correlated subqueries

```sql
SELECT e.employee_id, e.employee_name, e.salary
FROM employees AS e
WHERE e.salary > (
    SELECT AVG(peer.salary)
    FROM employees AS peer
    WHERE peer.department_id = e.department_id
)
ORDER BY e.employee_id;
```

The inner expression depends on the outer employee's department. Expected IDs: **1 and 3**. Correlation describes semantics, not necessarily a literal execution of the subquery from scratch for every row; optimizers may decorrelate or otherwise transform it.

Still inspect actual plans: repeated nested execution can be expensive on some queries. A grouped join or window expression may be clearer, but is not automatically faster for every data distribution.

## EXISTS, IN, and the NOT IN trap

```sql
SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.customer_id AND o.status = 'paid'
)
ORDER BY c.customer_id;

SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.customer_id AND o.status = 'paid'
)
ORDER BY c.customer_id;
```

EXISTS tests whether any qualifying row exists. It does not multiply the outer row when several matches exist. The first query returns 1, 2, 3; the second returns 4, 5.

IN is natural when testing membership in a known list or one-column query. `NOT IN` behaves unexpectedly if the right side includes NULL: for a nonmatching value, the overall result can be unknown instead of true.

```sql
SELECT employee_id
FROM employees
WHERE employee_id NOT IN (SELECT manager_id FROM employees)
ORDER BY employee_id;
```

This returns **no rows** because the manager list includes NULL. Use NOT EXISTS for “employees who manage nobody”:

```sql
SELECT e.employee_id
FROM employees AS e
WHERE NOT EXISTS (
    SELECT 1 FROM employees AS child WHERE child.manager_id = e.employee_id
)
ORDER BY e.employee_id;
```

Expected: **2, 4, 5, 6**. Filtering NULLs from the NOT IN subquery can also work when that matches the intended null semantics.

`ANY`/`ALL` comparisons are available in some engines: `x > ALL(subquery)` is not universally equivalent to `x > MAX(...)`, particularly with empty inputs and NULLs. Check semantics before rewriting.

## Common table expressions

A CTE names a query expression for one statement. It improves decomposition; it is not a persistent table or a guarantee of caching/materialization.

```sql
WITH paid_order_totals AS (
    SELECT o.order_id, o.customer_id,
           SUM(oi.quantity * oi.unit_price_cents) AS total_cents
    FROM orders AS o
    JOIN order_items AS oi ON oi.order_id = o.order_id
    WHERE o.status = 'paid'
    GROUP BY o.order_id, o.customer_id
), customer_totals AS (
    SELECT customer_id, COUNT(*) AS paid_orders, SUM(total_cents) AS total_cents
    FROM paid_order_totals
    GROUP BY customer_id
)
SELECT c.customer_name, t.paid_orders, t.total_cents
FROM customer_totals AS t
JOIN customers AS c ON c.customer_id = t.customer_id
ORDER BY t.total_cents DESC, c.customer_id;
```

Expected: Dev `(2, 14000)`, Asha `(3, 9000)`, Mira `(1, 5000)`. Naming each intermediate grain makes double counting easier to detect.

PostgreSQL can fold suitable CTEs into the parent query or materialize them depending on their shape/use; `MATERIALIZED` and `NOT MATERIALIZED` offer controls in supported cases. Other engines make different choices. Refer to [CTE materialization](https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-CTE-MATERIALIZATION) rather than assuming every CTE is an optimization barrier or a speedup.

## Recursive CTEs

A recursive query has an anchor, a recursive member, and a termination condition. It is useful for trees, dependency paths, and bounded sequences.

**SQLite/PostgreSQL/MySQL syntax**; SQL Server omits the `RECURSIVE` keyword and has its own recursion-limit options.

```sql
WITH RECURSIVE hierarchy AS (
    SELECT employee_id, employee_name, manager_id, 0 AS depth
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT child.employee_id, child.employee_name, child.manager_id,
           parent.depth + 1
    FROM employees AS child
    JOIN hierarchy AS parent ON child.manager_id = parent.employee_id
    WHERE parent.depth < 20
)
SELECT employee_id, employee_name, depth
FROM hierarchy
ORDER BY depth, employee_id;
```

The maximum depth is a teaching safeguard, not proof the data is acyclic. It can truncate a valid deep tree. General graphs need visited-path/cycle detection appropriate to the engine. Disconnected cycles with no root will not be reached by this anchor.

Changing UNION ALL to UNION only deduplicates complete rows; it does not stop all cycles when values such as depth keep changing. PostgreSQL supports specialized SEARCH/CYCLE syntax; other databases need explicit path tracking. Recursive evaluation order does not guarantee final display order, so sort explicitly.

## Lateral joins and choosing a form

**PostgreSQL:** A LATERAL subquery can reference earlier FROM items. This returns each customer's latest order while preserving customers with none:

<!-- dialect: postgresql -->
```sql
SELECT c.customer_id, recent.order_id, recent.order_date
FROM customers AS c
LEFT JOIN LATERAL (
    SELECT o.order_id, o.order_date
    FROM orders AS o
    WHERE o.customer_id = c.customer_id
    ORDER BY o.order_date DESC, o.order_id DESC
    LIMIT 1
) AS recent ON TRUE
ORDER BY c.customer_id;
```

SQL Server uses APPLY patterns for related tasks. A window-function solution is another option. Choose based on readability, selective indexed lookups, and the measured plan.

Use EXISTS for existence, joins for needed matching columns, scalar subqueries for true scalar results, CTEs for named stages, and recursive CTEs for recursive relationships. No form is inherently fastest in all cases.

Practice: find above-department-average salaries in two ways; find customers who bought every product in a chosen set using double NOT EXISTS; trace a hierarchy; explain the null behavior of NOT IN.

Next: [Window functions](05-window-functions.md).
