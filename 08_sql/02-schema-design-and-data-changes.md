# 2. Schema Design, Constraints, and Data Changes

> **Goal:** Define valid data and change it without breaking its meaning. Examples use a disposable practice database.

## Contents

- [Types and keys](#types-and-keys)
- [Constraints](#constraints)
- [Normalization and relationships](#normalization-and-relationships)
- [INSERT, UPDATE, and DELETE](#insert-update-and-delete)
- [Schema changes and migrations](#schema-changes-and-migrations)
- [Upserts and bulk loading](#upserts-and-bulk-loading)
- [Design review and practice](#design-review-and-practice)

## Types and keys

| Data | Common type choice | Check |
|---|---|---|
| Counts/IDs | INTEGER/BIGINT | Range; IDs are not always numeric |
| Exact decimal quantity | DECIMAL/NUMERIC | Precision, scale, rounding, overflow |
| Approximate measurement | REAL/DOUBLE | Floating-point error |
| Text | VARCHAR/TEXT | Length rules, collation, Unicode |
| Calendar date | DATE | No time of day |
| Event instant | Appropriate timestamp type | Timezone and precision semantics |
| Binary payload | Vendor binary type | Large-object storage strategy |
| Boolean | BOOLEAN or dialect equivalent | SQLite/SQL Server representation differs |

A primary key identifies each row. A candidate key is any minimal unique identifier; one is chosen as the primary key. A composite key uses multiple columns. A surrogate key is generated for identity; a natural key comes from the domain. A surrogate key does not remove the need for a UNIQUE constraint on a meaningful natural identifier.

## Constraints

`NOT NULL` requires a value. `UNIQUE` enforces uniqueness according to engine null/collation rules. `CHECK` validates a predicate. `FOREIGN KEY` requires a referenced key or permits NULL if the column is nullable. `DEFAULT` supplies a value when omitted; it is not a general validation rule.

```sql
CREATE TABLE inventory_demo (
    product_id INTEGER PRIMARY KEY,
    stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

INSERT INTO inventory_demo (product_id, stock) VALUES (10, 5);
SELECT product_id, stock FROM inventory_demo;
```

CHECK constraints generally accept an unknown result, so `CHECK (stock >= 0)` alone does not reject NULL. Add `NOT NULL` when required. A self-referencing foreign key for a manager does not by itself prevent cycles.

Foreign keys can specify delete/update actions such as NO ACTION, RESTRICT, CASCADE, or SET NULL, with differences across engines. Choose the action from the business lifecycle, not convenience. Automatic cascades can affect many rows.

Primary/unique constraints commonly create supporting indexes. A foreign key does **not universally** create an index on the referencing columns; inspect your engine and workload. Database constraints protect against all writers, unlike validation in only one application.

## Normalization and relationships

| Concept | Problem avoided | Example |
|---|---|---|
| First normal form | Repeating groups in one logical record | Store order lines as rows, not `product1`, `product2` columns |
| Second normal form | Non-key facts depending on only part of a composite candidate key | In enrollment keyed by `(student_id, course_id)`, student name depends only on student_id |
| Third normal form | Non-key facts depending transitively on the key | Department name belongs in departments |
| BCNF | Nontrivial functional dependencies whose determinants are not superkeys | Analyze more complex overlapping candidate keys |

Functional dependencies express business rules: `product_id → product_name` means a product ID determines its name in the chosen model. Normalization depends on those rules, not just the presence of a numeric ID.

One-to-many relationships place the foreign key on the many side. Many-to-many relationships use a junction table. A one-to-one relationship can be expressed with a foreign key plus uniqueness when that is the intended constraint.

Historical transaction price is a fact about the sale, not simply a duplicate of today's product price. Model time and history before deciding that every repeated value is a normalization mistake.

Denormalization can help measured read workloads, but it introduces synchronization rules. Analytical star schemas use fact tables at a defined grain and descriptive dimensions. Slowly changing dimensions represent attribute history; joining on today's dimension values can misstate past reports.

## INSERT, UPDATE, and DELETE

Explicit column lists protect inserts from column-order assumptions. Multiple VALUES rows or `INSERT ... SELECT` are useful for bulk transformations.

```sql
BEGIN;

INSERT INTO customers (customer_id, customer_name, region, email, signup_date)
VALUES (99, 'Practice User', 'East', 'practice@example.com', '2026-03-01');

UPDATE customers
SET region = 'West'
WHERE customer_id = 99;

SELECT customer_name, region FROM customers WHERE customer_id = 99;

DELETE FROM customers WHERE customer_id = 99;

ROLLBACK;
```

This transaction syntax works in SQLite/PostgreSQL and in MySQL transactional engines. SQL Server uses `BEGIN TRANSACTION`, not bare `BEGIN`, to start a transaction. In real work, use COMMIT only when the intended changes are validated.

Before an UPDATE/DELETE, inspect the same WHERE predicate with SELECT and verify the intended key. Without WHERE, the statement affects all qualifying table rows. Parameterized predicates and affected-row checks belong in application code.

`DELETE` removes rows; `DROP TABLE` removes the table object; `TRUNCATE` is an engine-specific fast whole-table operation with distinct locking, identity, trigger, and transaction behavior. SQLite has no TRUNCATE statement. Never infer rollback behavior across engines from one vendor's example.

## Schema changes and migrations

```sql
ALTER TABLE inventory_demo ADD COLUMN reorder_level INTEGER NOT NULL DEFAULT 0;
```

Supported ALTER operations, online behavior, lock duration, and table rewrites vary. A migration is a versioned transition with preconditions, validation, deployment ordering, and recovery planning.

For a large production table, an “add column and backfill everything” change may lock or overload the system. Consider expand/backfill/validate/contract steps, bounded batches, and compatibility between old and new application versions.

Renaming a column can break code, views, reports, or exports. Dropping an unused-looking index can affect a rare critical query. Inspect dependencies and actual workload before destructive migrations.

## Upserts and bulk loading

An upsert inserts or updates when a specified uniqueness conflict occurs. A uniqueness constraint is essential; an application-level existence check can race with another writer.

**PostgreSQL/SQLite example**, using the practice inventory table:

<!-- dialect: postgres-sqlite -->
```sql
INSERT INTO inventory_demo (product_id, stock)
VALUES (10, 7)
ON CONFLICT (product_id)
DO UPDATE SET stock = excluded.stock;
```

MySQL uses `ON DUPLICATE KEY UPDATE`; SQL Server has different patterns including MERGE with important concurrency/design considerations. SQLite `INSERT OR REPLACE` can delete and reinsert the existing row; it is not a drop-in synonym for an in-place update.

Database loaders such as PostgreSQL COPY, MySQL LOAD DATA, and SQL Server bulk tooling can outperform row-by-row inserts. Load into a staging table when parsing, deduplication, and validation need to precede publication. Check permissions, encoding, schema, null representation, and transaction boundaries.

## Design review and practice

Check table grain, keys, required values, ranges, currency/units, timezone rules, unique/null semantics, deletion behavior, and history requirements. Use exact decimal types or documented minor units for exact monetary amounts; do not assume a floating-point value is exact currency.

Practice: model students and courses with an enrollment table; add a stock constraint; distinguish a historical order price from current catalog price; design a migration that introduces a required column without immediately breaking existing writers.

References: [PostgreSQL constraints](https://www.postgresql.org/docs/current/ddl-constraints.html), [SQLite type model](https://www.sqlite.org/datatype3.html). Next: [Joins and aggregation](03-joins-aggregation-and-set-operations.md).
