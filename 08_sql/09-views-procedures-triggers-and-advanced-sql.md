# 9. Views, Procedures, Triggers, JSON, and Advanced Database Features

> **Goal:** Know what each database feature provides and when its maintenance cost is justified. Engine-specific examples are explicitly labeled.

## Contents

- [Views and materialized views](#views-and-materialized-views)
- [Temporary tables and CTEs](#temporary-tables-and-ctes)
- [Functions and procedures](#functions-and-procedures)
- [Triggers](#triggers)
- [JSON and semi-structured data](#json-and-semi-structured-data)
- [Partitioning and analytical models](#partitioning-and-analytical-models)
- [Advanced feature map](#advanced-feature-map)
- [Practice](#practice)

## Views and materialized views

A normal view stores a query definition, not a frozen result. It provides a reusable interface but does not guarantee an execution-speed improvement.

```sql
CREATE VIEW paid_order_totals AS
SELECT o.order_id, o.customer_id, o.order_date,
       SUM(oi.quantity * oi.unit_price_cents) AS total_cents
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
WHERE o.status = 'paid'
GROUP BY o.order_id, o.customer_id, o.order_date;

SELECT order_id, total_cents
FROM paid_order_totals
ORDER BY order_id;
```

The view establishes order grain once. It excludes orders with no lines and includes only the current paid status. Neither decision is a universal business rule; document them.

A materialized view stores results and needs a refresh strategy. **PostgreSQL example**, after creating the ordinary view above:

<!-- dialect: postgresql -->
```sql
CREATE MATERIALIZED VIEW daily_paid_revenue AS
SELECT order_date, SUM(total_cents) AS revenue_cents
FROM paid_order_totals
GROUP BY order_date;

CREATE UNIQUE INDEX idx_daily_paid_revenue_date
ON daily_paid_revenue (order_date);

REFRESH MATERIALIZED VIEW daily_paid_revenue;
```

Refresh cost, locks, freshness, and concurrent-refresh requirements matter. PostgreSQL's concurrent refresh has prerequisites including a suitable unique index and a populated view. SQL Server indexed views use different rules; MySQL and SQLite do not provide this same native materialized-view interface.

## Temporary tables and CTEs

| Object | Lifetime | Good use |
|---|---|---|
| CTE | One statement | Name logical query stages |
| Derived table | One query expression | Local inline transformation |
| Temporary table | Session/transaction rules vary | Reuse or index a substantial intermediate result |
| View | Persistent definition | Shared logical interface |
| Materialized summary | Persistent stored results | Repeated expensive reads with controlled freshness |

Temporary objects can have statistics and indexes where supported, but also allocate storage and need cleanup. Connection pooling can make session lifetime longer than one request. An application must not accidentally reuse another request's stale temporary state.

## Functions and procedures

A function can participate in expressions or return a table depending on the engine. A procedure is invoked as an operation and may support transaction control under specific rules. Syntax and capabilities vary widely.

**PostgreSQL SQL-language function:**

<!-- dialect: postgresql -->
```sql
CREATE FUNCTION line_total_cents(qty INTEGER, unit_price INTEGER)
RETURNS BIGINT
LANGUAGE SQL
IMMUTABLE
STRICT
AS $$
    SELECT qty::BIGINT * unit_price::BIGINT;
$$;

SELECT line_total_cents(3, 1000);
```

IMMUTABLE promises the same result for the same inputs independently of database state; do not apply it to a lookup or time-dependent function. STRICT means NULL input yields NULL without calling the body. The cast widens before multiplication, but BIGINT still has a finite range.

**PostgreSQL procedure:**

<!-- dialect: postgresql -->
```sql
CREATE PROCEDURE rename_customer(p_customer_id INTEGER, p_name TEXT)
LANGUAGE SQL
AS $$
    UPDATE customers
    SET customer_name = p_name
    WHERE customer_id = p_customer_id;
$$;

-- Example call, intentionally rolled back:
BEGIN;
CALL rename_customer(4, 'Noor Updated');
ROLLBACK;
```

This minimal procedure relies on existing constraints and does not report a missing ID; production APIs should define validation, affected-row behavior, authorization, and error semantics. Stored code can reduce repeated logic, but can also hide expensive row-by-row work and complicate deployment.

SQLite does not have equivalent SQL-defined stored procedures; applications can register user-defined functions through supported APIs. Those functions execute in the application process and need their own correctness/performance review.

## Triggers

A trigger runs automatically for specified database events. It can support auditing or invariants that simpler constraints cannot express, but hidden side effects can surprise application developers.

**SQLite example:** Log real status changes in the practice database.

<!-- dialect: sqlite -->
```sql
CREATE TABLE order_status_audit_demo (
    order_id INTEGER NOT NULL,
    old_status TEXT NOT NULL,
    new_status TEXT NOT NULL
);

CREATE TRIGGER audit_order_status_demo
AFTER UPDATE OF status ON orders
WHEN OLD.status IS NOT NEW.status
BEGIN
    INSERT INTO order_status_audit_demo (order_id, old_status, new_status)
    VALUES (NEW.order_id, OLD.status, NEW.status);
END;

BEGIN;
UPDATE orders SET status = 'paid' WHERE order_id = 106;
SELECT order_id, old_status, new_status FROM order_status_audit_demo;
ROLLBACK;
```

The query sees `(106, pending, paid)` inside the transaction; rollback undoes both the row change and this audit insertion. A durable security audit may need additional controls and information, such as actor/time and append-only permissions.

Row-level versus statement-level triggers, recursion, ordering, and OLD/NEW access differ by engine. Prefer a constraint when it expresses the rule directly. Do not send irreversible external messages from a trigger without a carefully designed delivery pattern; an outbox table is often a better boundary.

## JSON and semi-structured data

JSON is useful for genuinely variable structure, but putting every field into a blob can make types, keys, joins, and constraints harder to enforce.

**PostgreSQL:**

<!-- dialect: postgresql -->
```sql
SELECT '{"model":"baseline","metrics":{"accuracy":0.82}}'::jsonb
           ->> 'model' AS model_name,
       ('{"metrics":{"accuracy":0.82}}'::jsonb
           #>> '{metrics,accuracy}')::NUMERIC AS accuracy;
```

`->` returns JSON; `->>` returns text in PostgreSQL. Distinguish missing keys, JSON null, SQL NULL, and invalid values before casting. JSON types/operators/indexes are highly dialect-specific.

Normalize stable relational keys and frequently constrained fields. Use JSON for flexible attributes when that fits the contract, and index the actual query expressions when needed. Arrays and text search have similarly engine-specific semantics and indexing choices.

## Partitioning and analytical models

**PostgreSQL partitioned-table shape:**

<!-- dialect: postgresql -->
```sql
CREATE TABLE event_archive_demo (
    event_id BIGINT NOT NULL,
    event_date DATE NOT NULL,
    event_type TEXT NOT NULL
) PARTITION BY RANGE (event_date);

CREATE TABLE event_archive_2026_01_demo
PARTITION OF event_archive_demo
FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

Rows outside available partitions need a matching future/default partition or fail. Global uniqueness rules, indexes, retention, and partition count require planning. Partitioning is distinct from sharding across servers and replication across copies.

For analytical systems, define fact-table grain, dimension keys, slowly changing history, and late-arriving records. Columnar storage can accelerate scans of selected columns, while row-oriented transactional systems optimize a different mix of reads/writes. Choose based on workload rather than assuming one layout wins universally.

## Advanced feature map

| Feature | What to learn next |
|---|---|
| Full-text search | Tokenization, language configuration, ranking, indexes |
| Spatial SQL | Coordinate systems, geometry/geography, spatial predicates |
| Generated/computed columns | Determinism, storage, indexing, write cost |
| Row-level security | Policy composition, ownership/bypass behavior, testing |
| CDC | Change ordering, deletes, schema evolution, replay/idempotency |
| Replication | Lag, failover, consistency, conflict policy |
| Bulk ingestion | Staging, validation, batch size, error recovery |
| Ordered-set aggregates | Percentiles and supported dialect syntax |
| Query federation | Pushdown, network cost, cross-source consistency |

## Practice

Create a reusable order-total view; define a freshness policy for a daily summary; compare a trigger with a CHECK constraint; extract a typed JSON field while handling missing values; design a month-partition retention plan.

References: [PostgreSQL CREATE VIEW](https://www.postgresql.org/docs/current/sql-createview.html), [functions](https://www.postgresql.org/docs/current/sql-createfunction.html), [SQLite triggers](https://www.sqlite.org/lang_createtrigger.html). Next: [Analytics and ML patterns](10-analytics-and-ml-patterns.md).
