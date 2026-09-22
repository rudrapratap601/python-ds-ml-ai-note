# 1. SELECT, Filtering, Sorting, and NULL

> **Prerequisite:** Load the [practice database](00-practice-database.md). SQL describes the result you want; the database chooses a physical execution strategy.

## Contents

- [SQL vocabulary](#sql-vocabulary)
- [Selecting columns and expressions](#selecting-columns-and-expressions)
- [Filtering rows](#filtering-rows)
- [Three-valued logic and NULL](#three-valued-logic-and-null)
- [Ordering and limiting](#ordering-and-limiting)
- [CASE and conditional expressions](#case-and-conditional-expressions)
- [Logical query order](#logical-query-order)
- [Common mistakes and practice](#common-mistakes-and-practice)

## SQL vocabulary

| Category | Common statements | Purpose |
|---|---|---|
| Queries, often called DQL | `SELECT` | Retrieve data |
| Data manipulation, DML | `INSERT`, `UPDATE`, `DELETE`, dialect-specific `MERGE` | Change rows |
| Data definition, DDL | `CREATE`, `ALTER`, `DROP` | Define database objects |
| Transaction control | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Coordinate changes |
| Access control, DCL | `GRANT`, `REVOKE` | Manage privileges where supported |

These categories are learning vocabulary, not completely separate language engines. SQL is standardized, but vendors differ in functions, types, syntax, and transaction behavior.

Use single quotes for string literals. Identifier quoting varies; standard double quotes identify names, while MySQL commonly uses backticks unless configured otherwise. Prefer simple unquoted names such as `customer_id` to reduce portability issues.

`--` starts a line comment; `/* ... */` encloses a block comment. A semicolon terminates a statement. SQL keywords are commonly written uppercase for readability; that convention does not determine how data strings compare.

## Selecting columns and expressions

```sql
SELECT customer_id, customer_name, region
FROM customers
ORDER BY customer_id;

SELECT product_name, price_cents,
       price_cents / 100.0 AS price_units
FROM products
ORDER BY product_id;
```

`AS` gives an output column a readable alias. `SELECT *` is convenient for exploration but makes application outputs depend on future schema changes and can read unnecessary columns. Choose explicit columns at stable interfaces.

Arithmetic result types depend on operand types and engine rules. Integer division can truncate in some engines; use a suitable decimal cast or literal when fractional results are required. `100.0` is useful in these examples but does not guarantee identical precision rules on every database.

## Filtering rows

```sql
SELECT order_id, order_date
FROM orders
WHERE status = 'paid'
  AND order_date >= '2026-01-01'
  AND order_date < '2026-02-01'
ORDER BY order_date, order_id;
```

Expected order IDs: **101, 103, 102**. A half-open interval includes the start and excludes the end. It works naturally for timestamp ranges as well as dates when the boundary types/timezones are correct.

| Operator | Example | Meaning |
|---|---|---|
| `=`, `<>` | `status <> 'cancelled'` | Equal / not equal |
| `<`, `<=`, `>`, `>=` | `price_cents >= 2000` | Compare |
| `AND`, `OR`, `NOT` | `(a OR b) AND c` | Boolean logic |
| `IN` | `region IN ('North', 'West')` | Membership |
| `BETWEEN` | `price_cents BETWEEN 1000 AND 3000` | Inclusive at both ends |
| `LIKE` | `customer_name LIKE 'A%'` | Pattern matching |
| `IS NULL` | `email IS NULL` | Missing marker test |

`AND` binds more tightly than `OR`; add parentheses to express intent. `LIKE` uses `%` for any sequence and `_` for one character. Case sensitivity and collation differ by database/configuration. Escaping literal wildcard characters requires a deliberate `ESCAPE` convention.

```sql
SELECT customer_name
FROM customers
WHERE region IN ('North', 'West')
ORDER BY customer_id;

SELECT customer_name
FROM customers
WHERE customer_name LIKE 'A%'
ORDER BY customer_id;
```

## Three-valued logic and NULL

NULL represents missing/unknown/not-applicable information according to your schema's meaning. It is not a numeric zero, empty string, or ordinary value.

Comparisons such as `NULL = NULL` and `region <> 'North'` when region is NULL evaluate to **unknown**, not true. `WHERE` retains only true rows; false and unknown are both excluded.

```sql
SELECT customer_name
FROM customers
WHERE region IS NULL;

SELECT customer_name
FROM customers
WHERE region <> 'North' OR region IS NULL
ORDER BY customer_id;
```

The first query returns Ishan. The second returns Dev, Noor, and Ishan. `WHERE region = NULL` is not the correct null test.

| Expression | Result |
|---|---|
| `TRUE AND UNKNOWN` | UNKNOWN |
| `FALSE AND UNKNOWN` | FALSE |
| `TRUE OR UNKNOWN` | TRUE |
| `FALSE OR UNKNOWN` | UNKNOWN |
| `NOT UNKNOWN` | UNKNOWN |

`COALESCE(a, b, ...)` returns the first non-null argument. `NULLIF(a, b)` returns NULL if the two values compare equal, otherwise a. Choose compatible result types.

```sql
SELECT customer_name, COALESCE(region, 'Unspecified') AS display_region
FROM customers
ORDER BY customer_id;

SELECT 10.0 / NULLIF(0, 0) AS ratio;
```

The ratio is NULL instead of dividing by zero. This encodes “undefined,” not a valid zero ratio. Do not replace missing values until you know their business meaning.

## Ordering and limiting

Without `ORDER BY`, result order is not guaranteed, even if it looked stable yesterday. Tie-breakers make top-N results deterministic.

```sql
SELECT employee_id, employee_name, salary
FROM employees
ORDER BY salary DESC, employee_id
LIMIT 3;
```

`LIMIT` works in SQLite, PostgreSQL, and MySQL; SQL Server uses `TOP` or ordered `OFFSET ... FETCH`. The query returns Asha, Dev, and Mira. It chooses three rows, not three distinct salary levels.

Default null placement varies. An explicit portable approach is `ORDER BY CASE WHEN region IS NULL THEN 1 ELSE 0 END, region, customer_id`. Some engines support `NULLS FIRST/LAST` directly.

`DISTINCT` removes duplicate complete selected rows. It does not mean “choose one arbitrary row per customer” and should not hide an incorrect join.

## CASE and conditional expressions

```sql
SELECT product_name,
       CASE
           WHEN price_cents < 2000 THEN 'low'
           WHEN price_cents < 4000 THEN 'medium'
           ELSE 'high'
       END AS price_band
FROM products
ORDER BY product_id;
```

A searched CASE tests conditions in order. A simple CASE compares one expression with alternatives. Without `ELSE`, unmatched cases yield NULL. Compatible output types are required; do not mix numeric values and arbitrary text in one result column without a deliberate type design.

Do not rely on CASE as a universal guard against every planning-time or aggregate evaluation error. For arithmetic safety, encode validity in the expression itself, such as `NULLIF` for a denominator.

## Logical query order

```text
FROM / JOIN
→ WHERE
→ GROUP BY and aggregate calculations
→ HAVING
→ window calculations
→ SELECT output / DISTINCT
→ ORDER BY
→ row limit / offset
```

This is a conceptual model, not a required physical plan. It explains why a SELECT alias is generally unavailable in the same query's WHERE, and why a window result usually needs an outer query before filtering. Vendor alias rules can be more permissive; avoid relying on that for portable queries.

## Common mistakes and practice

Avoid `= NULL`, ambiguous AND/OR logic, unstable top-N queries, accidental integer division, timestamp ranges ending at midnight, and assuming case comparison rules are identical everywhere.

Practice: list paid February orders; select customers with no email; create a price band; return the two most expensive products with stable ordering; explain why `region <> 'North'` excludes Ishan.

Reference: [PostgreSQL query structure](https://www.postgresql.org/docs/current/queries.html). Next: [Schema design and data changes](02-schema-design-and-data-changes.md).
