# SQL Dialects and Command Reference

SQL is a language family implemented by database engines. PostgreSQL, MySQL, SQL Server, and SQLite share many concepts but differ in types, date arithmetic, generated keys, concurrency, permissions, and optimizer behavior. A client command such as `\dt` is not SQL.

## Contents

- [Dialect comparison](#dialect-comparison)
- [Adapting the practice database](#adapting-the-practice-database)
- [Command families and use cases](#command-families-and-use-cases)
- [Database clients](#database-clients)
- [Debugging checklist](#debugging-checklist)
- [Official references](#official-references)

## Dialect comparison

These are common forms, not a promise that every version supports every feature. The MySQL references target 8.4. Check the version you deploy.

| Task | PostgreSQL | MySQL | SQL Server | SQLite |
|---|---|---|---|---|
| Limit ordered output | `LIMIT 10` | `LIMIT 10` | `TOP (10)` or `OFFSET … FETCH` | `LIMIT 10` |
| Identifier quoting | `"column"` | Backticks by default | `[column]`; double quotes with appropriate setting | Double quotes |
| String literal | Single quotes | Single quotes | Single quotes; `N'…'` for Unicode literals where needed | Single quotes |
| Generated integer key | `GENERATED … AS IDENTITY` | `AUTO_INCREMENT` | `IDENTITY(1,1)` | `INTEGER PRIMARY KEY` rowid behavior |
| Boolean | `boolean` | BOOLEAN is a TINYINT synonym | `bit` | Commonly INTEGER with CHECK |
| Concatenation | `\|\|` or `concat` | `CONCAT` | `CONCAT` or `+` | `\|\|` |
| Recursive CTE | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH` | `WITH RECURSIVE` |
| Upsert | `ON CONFLICT` | `ON DUPLICATE KEY UPDATE` | Design a concurrency-safe update/insert; review MERGE carefully | `ON CONFLICT` |
| Full outer join | Supported | No native FULL OUTER JOIN | Supported | Supported in modern SQLite (3.39+) |
| Plan estimate | `EXPLAIN` | `EXPLAIN` | Estimated execution plan in client | `EXPLAIN QUERY PLAN` |
| Actual plan | `EXPLAIN ANALYZE` executes | `EXPLAIN ANALYZE` executes supported statements | Actual plan executes | Query-plan output is not an actual timing profile |
| Date storage | DATE and timestamp types | DATE and datetime/timestamp types | DATE and datetime types | No dedicated date storage class; choose a convention |
| Stored procedures | Yes | Yes | Yes | No native stored procedures |
| Server roles/GRANT | Yes | Yes | Yes | Embedded DB; file/application access controls |

Do not replace `||` with `+` blindly: NULL behavior and implicit conversion rules can change results. Likewise, engine-specific collations affect case sensitivity, sorting, and uniqueness. [PostgreSQL expressions](https://www.postgresql.org/docs/current/sql-expressions.html), [SQLite SQL](https://www.sqlite.org/lang.html)

### Pagination examples

Always provide an ordering that uniquely orders rows.

**PostgreSQL:**

<!-- dialect: postgresql -->
```sql
SELECT order_id, order_date FROM orders
ORDER BY order_date, order_id LIMIT 3 OFFSET 3;
```

**SQL Server:**

<!-- dialect: sqlserver -->
```sql
SELECT order_id, order_date FROM orders
ORDER BY order_date, order_id
OFFSET 3 ROWS FETCH NEXT 3 ROWS ONLY;
```

For large offsets or frequently changing datasets, see [keyset pagination](07-indexes-and-query-optimization.md). Pagination syntax alone does not provide a consistent snapshot across requests.

### Date arithmetic examples

These independent examples all add seven days to February 1, 2026.

**PostgreSQL:**

<!-- dialect: postgresql -->
```sql
SELECT DATE '2026-02-01' + 7 AS next_date;
```

**MySQL:**

<!-- dialect: mysql -->
```sql
SELECT DATE_ADD('2026-02-01', INTERVAL 7 DAY) AS next_date;
```

**SQL Server:**

<!-- dialect: sqlserver -->
```sql
SELECT DATEADD(day, 7, CAST('2026-02-01' AS date)) AS next_date;
```

**SQLite:**

<!-- dialect: sqlite -->
```sql
SELECT date('2026-02-01', '+7 days') AS next_date;
```

## Adapting the practice database

The fixture supplies IDs explicitly, so generated-key syntax is unnecessary. Read the engine notes in [setup](00-practice-database.md) before running it.

- **SQLite:** enable `PRAGMA foreign_keys = ON` for each connection, before beginning a transaction. Use the ISO date strings shown. Declaring `VARCHAR(100)` does not enforce a 100-character limit in SQLite.
- **PostgreSQL:** omit SQLite PRAGMAs. The basic schema and rows work with the shown standard types and explicitly assigned IDs.
- **MySQL:** use a transactional engine such as InnoDB with foreign-key enforcement. Check server SQL mode; do not rely on permissive grouping or silent coercion. Recursive queries need a version with CTE support.
- **SQL Server:** change the email uniqueness definition as below, use TOP/OFFSET-FETCH in place of LIMIT, omit RECURSIVE, and adapt statement-specific date/concatenation syntax. A CTE following another statement often needs the previous statement terminated; `;WITH` is commonly used defensively.

### SQL Server nullable unique email

The fixture has two NULL emails. SQL Server's ordinary single-column UNIQUE constraint allows only one NULL. **Before creating the fixture there**, change its email declaration to `email VARCHAR(200)` (remove UNIQUE), then create this filtered index after creating customers:

<!-- dialect: sqlserver -->
```sql
CREATE UNIQUE INDEX uq_customers_nonnull_email
ON customers(email)
WHERE email IS NOT NULL;
```

This enforces uniqueness only for known emails. PostgreSQL, MySQL, and SQLite ordinarily permit multiple NULLs under their default single-column unique behavior; specialized options may alter that. [SQL Server unique indexes](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/create-unique-indexes?view=sql-server-ver17)

## Command families and use cases

These categories are study aids; engines differ in transaction behavior and terminology. In particular, do not assume all DDL can be rolled back on every engine.

| Command or clause | When to use it | Important detail |
|---|---|---|
| SELECT | Read or derive rows | Specify needed columns and output grain |
| FROM / JOIN / ON | Combine row sources | Verify keys and multiplicity |
| WHERE | Filter input rows | Only TRUE survives; UNKNOWN does not |
| GROUP BY | Produce one row per group | All selected nongrouped expressions need valid aggregate semantics |
| HAVING | Filter groups | Common for aggregate thresholds |
| ORDER BY | Specify result order | Without it, result order is not guaranteed |
| DISTINCT | Remove duplicate projected rows | Does not repair an incorrect join |
| UNION ALL | Append results including duplicates | Inputs need compatible columns |
| UNION / INTERSECT / EXCEPT | Set combination/comparison | Duplicate and ALL support varies by engine |
| CASE | Conditional expression | Not application-style control flow |
| COALESCE / NULLIF | Handle missing values or zero denominators | Preserve meaningful distinctions |
| EXISTS / NOT EXISTS | Test presence/absence | Useful for semijoins and antijoins |
| WITH | Name query stages | Does not universally force materialization |
| OVER | Define window calculations | Partition, order, and frame have different roles |
| INSERT | Add records | List columns explicitly |
| UPDATE | Change existing records | Check the predicate and affected row count |
| DELETE | Remove matching records | Missing WHERE targets every row |
| CREATE TABLE | Define a relation | Encode keys, constraints, and types |
| ALTER TABLE | Evolve a schema | Locks/rewrite cost and syntax vary |
| DROP | Remove an object | Dependencies and data loss need deliberate handling |
| TRUNCATE | Clear a table where supported | Semantics differ from DELETE; SQLite has no TRUNCATE |
| CREATE INDEX | Add an access path or enforce uniqueness | Adds write/storage cost |
| CREATE VIEW | Name a reusable query | Ordinary views do not store the result |
| BEGIN / START TRANSACTION | Begin an atomic unit | Driver and engine behavior matter |
| COMMIT / ROLLBACK | Persist/discard transaction changes | External side effects may not be reversible |
| SAVEPOINT | Mark a partial rollback position | Syntax varies; not an independent commit |
| GRANT / REVOKE | Manage server permissions | Use least privilege; role inheritance matters |
| EXPLAIN | Inspect the optimizer plan | Actual execution variants execute the query |

Learn syntax through the relevant chapter rather than treating this table as an executable script. Rare administrative and vendor extensions are best located in the official command catalogs below.

## Database clients

Run these in a terminal only after installing/configuring the relevant client and using your own local practice database. Names are examples. Do not embed passwords in shell commands or commit connection strings.

| Client | Example connection | Useful client commands |
|---|---|---|
| SQLite CLI | `sqlite3 practice.db` | `.tables`, `.schema customers`, `.headers on`, `.mode column`, `.quit` |
| PostgreSQL psql | `psql -h localhost -U learner -d practice` | `\dt`, `\d customers`, `\conninfo`, `\q` |
| MySQL CLI | `mysql -h localhost -u learner -p practice` | `SHOW TABLES;`, `DESCRIBE customers;`, `SELECT VERSION();`, `quit` |
| SQL Server sqlcmd | `sqlcmd -S localhost -d practice -E` | `GO` sends a batch; `QUIT` exits; `-E` uses integrated authentication where configured |

`GO`, psql backslash commands, and SQLite dot commands belong to their clients; Python DB drivers generally will not understand them. A GUI SQL editor still sends engine-specific SQL; selecting a different editor does not make queries portable.

Connection failures are usually about server availability, host/port, authentication, TLS, or permissions. Syntax failures after connection are about SQL and dialect. Keep these troubleshooting paths separate.

## Debugging checklist

1. Confirm engine/version, selected database/schema, and session timezone.
2. Read the full error and identify the first failing statement.
3. Check spelling, quoting, aliases, commas, and datatype conversions.
4. Run each CTE independently, examining row counts and keys.
5. Check duplicates at joins and NULLs in filters and aggregates.
6. Add deterministic ordering while comparing expected and actual output.
7. For performance, use representative data and inspect estimates/actuals with an appropriate plan tool.
8. For writes, reproduce on an isolated copy and check constraints, permissions, locks, and affected rows.

## Official references

- [PostgreSQL SQL command catalog](https://www.postgresql.org/docs/current/sql-commands.html)
- [PostgreSQL tutorial](https://www.postgresql.org/docs/current/tutorial.html)
- [MySQL 8.4 SQL statements](https://dev.mysql.com/doc/refman/8.4/en/sql-statements.html)
- [SQL Server Transact-SQL reference](https://learn.microsoft.com/en-us/sql/t-sql/language-reference?view=sql-server-ver17)
- [SQLite SQL language](https://www.sqlite.org/lang.html)
- [SQLite command-line shell](https://www.sqlite.org/cli.html)
- [Python sqlite3](https://docs.python.org/3/library/sqlite3.html)

Return to the [SQL learning index](README.md).
