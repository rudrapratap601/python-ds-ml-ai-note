# 8. Transactions, Concurrency, Security, and Python

> **Goal:** Keep data correct when operations fail or multiple clients act at once. Engine-specific behavior must be tested on the database you actually use.

## Contents

- [ACID and transaction boundaries](#acid-and-transaction-boundaries)
- [Commit, rollback, and savepoints](#commit-rollback-and-savepoints)
- [Isolation and anomalies](#isolation-and-anomalies)
- [Locks, deadlocks, and optimistic updates](#locks-deadlocks-and-optimistic-updates)
- [Parameterized SQL in Python](#parameterized-sql-in-python)
- [A transactional transfer example](#a-transactional-transfer-example)
- [Permissions and operational security](#permissions-and-operational-security)
- [Practice](#practice)

## ACID and transaction boundaries

| Property | Meaning | What it does not imply |
|---|---|---|
| Atomicity | A transaction's changes succeed or roll back as a unit | External emails/files are automatically rolled back |
| Consistency | Enforced invariants remain valid | The database knows every unstated business rule |
| Isolation | Concurrent transactions obey an isolation model | Every isolation level acts like serial execution |
| Durability | Committed changes persist under the configured guarantees | Backups and disaster recovery are unnecessary |

Autocommit and implicit transaction behavior differ across databases and drivers. Know when your client starts a transaction and when it commits. Keep transactions short and avoid waiting for user input while holding locks.

## Commit, rollback, and savepoints

**SQLite/PostgreSQL example:**

```sql
BEGIN;
UPDATE customers SET region = 'East' WHERE customer_id = 4;
SAVEPOINT before_second_change;
UPDATE customers SET region = 'South' WHERE customer_id = 4;
ROLLBACK TO SAVEPOINT before_second_change;
SELECT customer_name, region FROM customers WHERE customer_id = 4;
ROLLBACK;
```

Inside the transaction the query returns Noor/East. The final rollback restores Noor/West. A savepoint lets you undo part of a transaction; it is not necessarily an independent nested transaction that can commit permanently on its own.

Use COMMIT when all required operations succeed. After a statement error, PostgreSQL generally leaves the transaction aborted until rollback or recovery to a savepoint; other engines can behave differently. Do not simply catch an exception and continue issuing writes without understanding the connection state.

## Isolation and anomalies

| Anomaly | Example |
|---|---|
| Dirty read | Read another transaction's uncommitted update |
| Non-repeatable read | Read the same row twice and observe another committed update |
| Phantom | Repeat a predicate and observe a changed set of rows |
| Lost update | Two clients overwrite each other's work based on stale reads |
| Write skew | Concurrent transactions change different rows but jointly violate a cross-row invariant |

The standard names Read Uncommitted, Read Committed, Repeatable Read, and Serializable describe increasing constraints, but implementations differ. MVCC/snapshot behavior is not identical to locking-based behavior. PostgreSQL treats Read Uncommitted as Read Committed; its Repeatable Read prevents phantom reads but can still allow serialization anomalies. Serializable transactions can fail with a serialization error and need a complete retry. See [PostgreSQL isolation](https://www.postgresql.org/docs/current/transaction-iso.html).

Do not assume SQLite, MySQL InnoDB, and SQL Server share defaults or semantics. SQLite has a different concurrency model with a single writer at a time; WAL can improve reader/writer coexistence but does not create unlimited concurrent writers.

Snapshot consistency does not by itself enforce every business rule. Model invariants with constraints, appropriate locks, or serializable logic and retries where needed.

## Locks, deadlocks, and optimistic updates

**PostgreSQL:** Lock selected existing rows before an operation requiring that lock:

<!-- dialect: postgresql -->
```sql
BEGIN;
SELECT order_id, status
FROM orders
WHERE order_id = 106
FOR UPDATE;
-- Validate and perform the intended transition here.
ROLLBACK;
```

Locking one row does not automatically protect an absent row or every related predicate. Lock scope, index choice, and isolation affect correctness. Acquire multiple locks in a consistent order when possible; keep the transaction small.

A deadlock is a cycle of incompatible waits. Databases typically abort one participant. Retry the complete intended transaction with a bounded policy, and ensure external side effects are not duplicated.

An optimistic update checks the version you previously read:

```sql
CREATE TABLE versioned_note_demo (
    note_id INTEGER PRIMARY KEY,
    body VARCHAR(200) NOT NULL,
    version INTEGER NOT NULL
);
INSERT INTO versioned_note_demo (note_id, body, version) VALUES (1, 'Draft', 1);

UPDATE versioned_note_demo
SET body = 'Reviewed', version = version + 1
WHERE note_id = 1 AND version = 1;

SELECT note_id, body, version FROM versioned_note_demo;
```

The application must check that exactly one row changed. Zero means the expected version was absent or stale; do not silently report success. A version column prevents this specific stale-write pattern, not every cross-row anomaly.

## Parameterized SQL in Python

Use the driver's parameter API for data values. Do not construct SQL by interpolating user input into strings.

```python
import sqlite3

connection = sqlite3.connect(":memory:")
try:
    connection.execute("CREATE TABLE people (id INTEGER PRIMARY KEY, name TEXT NOT NULL)")
    connection.executemany("INSERT INTO people (id, name) VALUES (?, ?)", [(1, "Asha"), (2, "Dev")])
    name = "Asha"
    rows = connection.execute("SELECT id FROM people WHERE name = ?", (name,)).fetchall()
    assert rows == [(1,)]
finally:
    connection.close()
```

SQLite uses `?` or named parameters. Other drivers may use `%s`, `:name`, or different conventions; a placeholder is not Python string formatting. Parameters generally represent values, not table names, column names, or ASC/DESC keywords. Use a strict allowlist and driver-supported identifier composition for dynamic identifiers.

`LIKE ?` is injection-safe for the supplied value, but `%` and `_` inside that value still have pattern meaning. Escape them when the requirement is a literal search.

Connection pooling reuses connections; ensure transaction/session state is reset appropriately. Fetch large results in batches or use supported server-side streaming rather than assuming `fetchall()` is always safe for memory. See the [Python sqlite3 documentation](https://docs.python.org/3/library/sqlite3.html).

## A transactional transfer example

This self-contained SQLite example checks both debit and credit row counts. It uses the default Python sqlite3 transaction behavior with the connection context manager; applications using another autocommit mode must adapt transaction control explicitly.

```python
import sqlite3


def transfer(connection, source, destination, amount):
    if source == destination:
        raise ValueError("accounts must differ")
    if isinstance(amount, bool) or not isinstance(amount, int) or amount <= 0:
        raise ValueError("amount must be positive integer cents")
    with connection:
        debit = connection.execute(
            "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?",
            (amount, source, amount),
        )
        if debit.rowcount != 1:
            raise ValueError("missing source or insufficient funds")
        credit = connection.execute(
            "UPDATE accounts SET balance = balance + ? WHERE id = ?",
            (amount, destination),
        )
        if credit.rowcount != 1:
            raise ValueError("missing destination")


connection = sqlite3.connect(":memory:")
try:
    connection.execute("CREATE TABLE accounts (id INTEGER PRIMARY KEY, balance INTEGER NOT NULL CHECK (balance >= 0))")
    with connection:
        connection.executemany("INSERT INTO accounts VALUES (?, ?)", [(1, 5000), (2, 3000)])
    transfer(connection, 1, 2, 1000)
    assert connection.execute("SELECT balance FROM accounts ORDER BY id").fetchall() == [(4000,), (4000,)]
    try:
        transfer(connection, 1, 99, 1000)
    except ValueError:
        pass
    assert connection.execute("SELECT balance FROM accounts ORDER BY id").fetchall() == [(4000,), (4000,)]
finally:
    connection.close()
```

The context manager commits or rolls back under this driver mode; it does not close the connection. This is a teaching example, not a complete payment system: production systems also need idempotency, audit records, numeric range rules, authorization, concurrency handling, and reconciliation.

## Permissions and operational security

Use least-privilege application roles, protected credentials, encrypted connections where required, and separate administrative access. Do not grant a reporting user schema-changing rights simply to fix a failing query.

**PostgreSQL example**, assuming `report_reader` already exists:

<!-- dialect: postgresql -->
```sql
GRANT USAGE ON SCHEMA public TO report_reader;
GRANT SELECT ON customers, orders, order_items TO report_reader;
REVOKE UPDATE ON orders FROM report_reader;
```

Revoking one grant may not remove permissions inherited through another role. Views and row-level security need deliberate ownership and policy design; a view is not automatically a complete privacy boundary. SQLite normally relies on process/file access rather than server roles and GRANT.

A backup is useful only if it can be restored. Test restores, define retention/recovery goals, and avoid logging credentials or full private queries unnecessarily. Read replicas can lag, so they may not immediately reflect a just-committed write.

## Practice

Demonstrate rollback after a failed second operation; simulate two clients using a version check; distinguish a lock wait from a deadlock; write a parameterized search; explain why a transaction cannot undo an email sent by application code.

Next: [Advanced database objects](09-views-procedures-triggers-and-advanced-sql.md).
