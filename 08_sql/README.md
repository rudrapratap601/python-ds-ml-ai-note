# SQL: Beginner to Advanced

This section teaches SQL through a shared retail and employee database, with explanations, executable examples, common mistakes, and worked exercises. Start with the practice database, then follow the numbered chapters. You can also use the table below as a topic index.

## Learning path

| Chapter | What you will learn |
|---|---|
| [00 · Practice database](00-practice-database.md) | Setup, tables, sample data, relationships, expected totals, engine differences |
| [01 · SELECT, filtering, and NULL](01-select-filtering-and-null.md) | Query order, projections, predicates, sorting, DISTINCT, CASE, three-valued logic |
| [02 · Schema design and data changes](02-schema-design-and-data-changes.md) | Datatypes, keys, constraints, normalization, INSERT, UPDATE, DELETE, schema changes, upserts |
| [03 · Joins, aggregation, and sets](03-joins-aggregation-and-set-operations.md) | Inner/outer/self/cross joins, cardinality, GROUP BY, HAVING, set operations |
| [04 · Subqueries, CTEs, and recursion](04-subqueries-ctes-and-recursion.md) | Scalar and correlated queries, EXISTS, IN/NULL traps, reusable stages, recursive hierarchies |
| [05 · Window functions](05-window-functions.md) | Ranking, top-N, LAG/LEAD, running totals, moving averages, frames, FIRST/LAST_VALUE |
| [06 · Functions, dates, and cleaning](06-functions-dates-and-data-cleaning.md) | Strings, numbers, casting, date ranges, timezones, deduplication, data-quality checks |
| [07 · Indexes and optimization](07-indexes-and-query-optimization.md) | B-tree/composite/partial/covering indexes, EXPLAIN, selectivity, statistics, query rewrites, pagination |
| [08 · Transactions, concurrency, and security](08-transactions-concurrency-and-security.md) | ACID, isolation, locks, deadlocks, retries, optimistic concurrency, parameterized Python queries, roles |
| [09 · Advanced database objects](09-views-procedures-triggers-and-advanced-sql.md) | Views, materialized views, functions, procedures, triggers, JSON, partitioning, migrations |
| [10 · Analytics and ML patterns](10-analytics-and-ml-patterns.md) | Revenue, cohorts, gaps and islands, relational division, point-in-time features, warehouses |
| [11 · Practice and solutions](11-practice-problems-and-solutions.md) | Thirteen worked problems, expected results, interview reasoning, end-to-end project |
| [12 · Dialects and command reference](12-dialects-and-command-reference.md) | PostgreSQL/MySQL/SQL Server/SQLite differences, command purposes, client usage, reference catalogs |

## How to study

1. Create an isolated practice database using Chapter 00. SQLite is the lowest-setup route; Python's standard library includes a SQLite interface.
2. Run each query, predict its result, and compare it with the explanation. Copy only SQL code into a SQL editor, not Markdown formatting.
3. Use a fresh copy of the fixture when starting a chapter. Some chapters create demonstration tables, indexes, views, or triggers, so rerunning their DDL unchanged can produce “already exists” errors.
4. Read the dialect label immediately above a code block. PostgreSQL, MySQL, SQL Server, and SQLite examples are not interchangeable scripts.
5. Try the exercises before the solutions. Add missing values, duplicate matches, ties, empty groups, and date-boundary events to test your understanding.
6. Read performance plans on larger representative data. A tiny teaching fixture cannot prove that an index makes a production workload faster.

## Conventions

- Prices are integer cents, avoiding accidental floating-point money examples. Real systems also need currency, rounding, tax, refunds, and overflow policies.
- Dates are ISO-formatted. Most introductory queries use a broadly shared SQL subset; LIMIT, recursive syntax, functions, and advanced features need the dialect adaptations described in Chapter 12.
- Output order is guaranteed only when the outer query has ORDER BY. Window ordering defines the calculation, not necessarily the final presentation.
- NULL means missing/unknown, not zero or an empty string. Examples deliberately include missing emails and customers without orders.
- The examples use current order status. A historical ML pipeline needs versioned/available-at data, as discussed in Chapter 10.
- Client commands and database administration examples are references, not instructions to change a live database.

## Example validation

Checked on 2026-09-21 using SQLite 3.53.1 in fresh in-memory databases: 119 SQL statements, two self-contained Python examples, all thirteen worked solutions, analytical totals, and rollback behavior. All 127 local links and heading references resolved. The 21 PostgreSQL/MySQL/SQL Server-specific code blocks are reference examples and were not executed on those servers; consult the linked vendor documentation and test on your target engine.

## What to master first

For analytics: Chapters 00–06, then 10–11. For application development: add constraints, transactions, parameter binding, and migrations. For performance work: master correctness and join cardinality before Chapter 07. For ML: pay special attention to output grain, temporal joins, leakage, and reproducibility.

This is a broad learning guide, not an exhaustive catalog of every vendor extension. Each chapter links official documentation for deeper details. Return to the [repository index](../README.md).
