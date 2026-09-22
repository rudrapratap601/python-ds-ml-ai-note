# 7. Indexes, Execution Plans, and Query Optimization

> **Goal:** Improve a correct query using measured evidence. A faster wrong result is still wrong. The tiny practice database demonstrates syntax, not realistic performance benchmarks.

## Contents

- [A disciplined tuning process](#a-disciplined-tuning-process)
- [How indexes help](#how-indexes-help)
- [Composite and covering indexes](#composite-and-covering-indexes)
- [Sargable predicates](#sargable-predicates)
- [Reading execution plans](#reading-execution-plans)
- [Joins, sorts, and statistics](#joins-sorts-and-statistics)
- [Query rewrites](#query-rewrites)
- [Pagination](#pagination)
- [Partitioning, caching, and operational limits](#partitioning-caching-and-operational-limits)
- [Checklist and practice](#checklist-and-practice)

## A disciplined tuning process

1. State the required rows, grain, ordering, and latency target.
2. Verify correctness, including NULLs, duplicates, ties, and empty results.
3. Reproduce with realistic data volume, distribution, parameters, and concurrency.
4. Inspect the plan and actual execution metrics where safe.
5. Identify the largest avoidable work: reads, repeated loops, sorts, spills, joins, or transfers.
6. Change one meaningful factor and compare both output and resource cost.
7. Test representative parameter values, not only the easiest case.

Measure end-to-end time as well as server execution: network transfer, client deserialization, and application loops can dominate a fast SQL plan.

## How indexes help

An index is an additional structure that can accelerate lookup, joining, uniqueness checks, and ordering. It costs storage, write maintenance, cache space, and operational effort.

```sql
CREATE INDEX idx_orders_customer_date
ON orders (customer_id, order_date, order_id);
```

B-tree indexes are common for equality/range/order workloads. Vendor-specific alternatives include hash, inverted/full-text, GIN/GiST, BRIN, spatial, and columnstore structures. Choose based on supported operators and data layout, not the name alone.

An index scan is not always better than a sequential/table scan. Reading a large fraction of a small table may be cheaper without an index. Low selectivity, stale statistics, and random access costs matter.

## Composite and covering indexes

For `(customer_id, order_date, order_id)`, a common useful shape is customer equality followed by a date range and stable ordering:

```sql
SELECT order_id, order_date
FROM orders
WHERE customer_id = 1
  AND order_date >= '2026-01-01'
  AND order_date < '2026-03-01'
ORDER BY order_date, order_id;
```

Leading-column constraints often determine how effectively a multicolumn B-tree narrows the scan. Later columns can still support filtering, coverage, or engine-specific skip-scan strategies, so “an index is never usable without the first column” is too absolute. See [PostgreSQL multicolumn indexes](https://www.postgresql.org/docs/current/indexes-multicolumn.html).

Column order should reflect equality/range/order requirements and actual workloads. “Put the most selective column first” is not a complete universal design rule.

A covering index contains the values needed by a query. PostgreSQL/SQL Server support INCLUDE columns in suitable indexes; other engines cover through key/storage design. PostgreSQL index-only scans also depend on visibility information, not merely the selected columns existing in the index.

**PostgreSQL example:**

<!-- dialect: postgresql -->
```sql
CREATE INDEX idx_paid_orders_lookup
ON orders (customer_id, order_date, order_id)
INCLUDE (status)
WHERE status = 'paid';
```

A partial/filtered index covers only rows matching a predicate. The planner must establish that the query can use it; parameterization and predicate form can matter. Support and syntax differ by engine. Do not create every example index on a production table without measuring write cost and overlap.

## Sargable predicates

“Sargable” means a predicate can be used effectively as a search condition for an available access path. Avoid needlessly transforming an indexed column when a direct range expresses the same result.

**PostgreSQL comparison:**

<!-- dialect: postgresql -->
```sql
-- Often harder to satisfy with an ordinary index on order_date:
SELECT order_id FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2026;

-- Direct range on the stored column:
SELECT order_id FROM orders
WHERE order_date >= DATE '2026-01-01'
  AND order_date < DATE '2027-01-01';
```

Expression indexes/computed columns can support appropriate transformed predicates. The aim is not to ban functions everywhere; it is to align predicates with indexed expressions and correct type semantics.

Other concerns include implicit casts on join/filter columns, leading-wildcard text search, collation mismatches, and optional-filter patterns such as `parameter IS NULL OR column = parameter`. Prefix LIKE searches may be indexable under suitable engine/collation rules; arbitrary substring search may need specialized indexing.

## Reading execution plans

**SQLite:**

<!-- dialect: sqlite -->
```sql
EXPLAIN QUERY PLAN
SELECT order_id, order_date
FROM orders
WHERE customer_id = 1
ORDER BY order_date, order_id;
```

**PostgreSQL:**

<!-- dialect: postgresql -->
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, order_date
FROM orders
WHERE customer_id = 1
ORDER BY order_date, order_id;
```

Plain EXPLAIN usually plans without executing the query. **EXPLAIN ANALYZE executes it**, including modifications when used on DML. Do not run it on a write statement against real data merely to inspect a plan. Transaction rollback does not undo every possible external effect or sequence advancement. See [EXPLAIN](https://www.postgresql.org/docs/current/sql-explain.html).

| Plan detail | What to ask |
|---|---|
| Estimated versus actual rows | Did the planner misunderstand cardinality? |
| Scan type and filter | How many rows/pages were examined and discarded? |
| Loop count | Is a small inner operation repeated many times? |
| Sort/hash memory and spills | Did work exceed available memory? |
| Buffer/cache reads | Is the workload reading cached or physical data? |
| Join type | Is it suitable for input sizes and predicates? |
| Planning versus execution time | Is planning overhead significant for this workload? |

PostgreSQL cost units are planner estimates, not milliseconds. Actual rows/time can be shown per loop; account for loops and avoid simply summing nested inclusive timings. Different engines report plans and waits differently. The [plan-reading guide](https://www.postgresql.org/docs/current/using-explain.html) explains PostgreSQL's output.

## Joins, sorts, and statistics

- **Nested loop:** Can be excellent for a small outer input plus selective indexed lookups; expensive with many repeated large inner scans.
- **Hash join:** Often effective for equality joins with larger inputs; building/spilling the hash table has a cost.
- **Merge join:** Uses suitably ordered inputs; required sorts may dominate.

Available algorithms vary; SQLite does not implement the same complete planner strategy set as PostgreSQL. SQL is declarative, so textual join order does not normally force physical join order.

Statistics estimate distributions and relationships. After major data changes, appropriate statistics maintenance can improve plans. Correlated columns can defeat independent estimates; some engines support extended statistics. Parameter-sensitive plans may work well for one value and poorly for another.

Use hints only after understanding the problem and the maintenance cost. A forced plan may age badly as data changes.

## Query rewrites

| Symptom | Candidate improvement | Correctness condition |
|---|---|---|
| Join only to test existence | EXISTS | No right-side output or multiplicity required |
| Many-to-many fanout | Preaggregate each child | Preserve intended grain and filters |
| Duplicate elimination not needed | UNION ALL | Duplicates are allowed/desired |
| Repeated scalar calculations | Shared aggregation/window/join | Preserve NULL and empty-input behavior |
| Huge returned table | Select columns and filter appropriately | Do not discard required facts |
| One database request per row | Set-based query or batch | Preserve ordering/error semantics |
| CTE assumed to improve performance | Inspect materialization/inlining | Same result under actual engine behavior |

Do not remove DISTINCT, reorder outer-join filters, replace NOT EXISTS with NOT IN, or change window frames solely for speed without proving equivalence.

## Pagination

OFFSET pagination can require skipping many earlier rows and can shift when concurrent inserts occur. Keyset pagination continues after the last stable ordered key:

```sql
SELECT order_id, order_date
FROM orders
WHERE order_date > '2026-01-05'
   OR (order_date = '2026-01-05' AND order_id > 104)
ORDER BY order_date, order_id
LIMIT 3;
```

Expected IDs: **105, 106, 107**. The cursor must carry both date and ID. An index matching the order can help. Descending order reverses comparisons. Nullable keys, changing sort values, and a required consistent snapshot need explicit handling.

## Partitioning, caching, and operational limits

Partitioning splits a logical table by a key such as date. Partition pruning can avoid irrelevant partitions, but partitioning does not replace all indexes or improve every query. Too many partitions add overhead, and uniqueness/foreign-key support varies.

Materialized summaries or caches can accelerate repeated analysis at the cost of freshness and invalidation. Define when results become stale and how they refresh. Batch writes, avoid long idle transactions, and consider lock contention before adding memory or indexes.

Increasing per-operation memory globally can multiply consumption across operators and concurrent sessions. A query's isolated benchmark does not prove production safety or capacity.

## Checklist and practice

Check result equivalence, representative parameters, row estimates, scans, join fanout, sort/spill behavior, redundant indexes, write overhead, and end-to-end latency. Keep before/after plans and measurements.

Practice: explain why the tiny sample may choose a table scan; design an index for customer/date filtering; compare offset and keyset pagination on larger disposable data; identify an implicit cast; rewrite an existence join without multiplying rows.

Next: [Transactions and security](08-transactions-concurrency-and-security.md).
