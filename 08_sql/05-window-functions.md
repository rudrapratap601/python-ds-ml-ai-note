# 5. Window Functions: Ranking, Frames, and Time-Series Analysis

> **Goal:** Calculate across related rows while retaining individual rows. This is a central chapter for analytics and SQL interviews.

## Contents

- [OVER, PARTITION BY, and ORDER BY](#over-partition-by-and-order-by)
- [ROW_NUMBER, RANK, and DENSE_RANK](#row_number-rank-and-dense_rank)
- [Top N per group](#top-n-per-group)
- [LAG and LEAD](#lag-and-lead)
- [Running totals and moving averages](#running-totals-and-moving-averages)
- [ROWS, RANGE, GROUPS, and peers](#rows-range-groups-and-peers)
- [FIRST_VALUE, LAST_VALUE, and NTH_VALUE](#first_value-last_value-and-nth_value)
- [Distribution functions and named windows](#distribution-functions-and-named-windows)
- [Filtering, performance, and practice](#filtering-performance-and-practice)

## OVER, PARTITION BY, and ORDER BY

```sql
SELECT employee_id, employee_name, department_id, salary,
       AVG(salary) OVER (PARTITION BY department_id) AS department_average,
       SUM(salary) OVER () AS company_payroll
FROM employees
ORDER BY employee_id;
```

GROUP BY would collapse the rows into groups. A window calculation attaches a result to each row of its input. `OVER ()` uses the entire input; PARTITION BY splits it into independent groups.

ORDER BY inside OVER controls the window calculation's sequence. The final query's ORDER BY controls presentation. One does not replace the other. Windows operate after WHERE/grouping/HAVING; a row filtered out earlier is not available to the window. See the [window tutorial](https://www.postgresql.org/docs/current/tutorial-window.html).

## ROW_NUMBER, RANK, and DENSE_RANK

```sql
SELECT employee_id, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC, employee_id) AS row_num,
       RANK() OVER (ORDER BY salary DESC) AS salary_rank,
       DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_level
FROM employees
ORDER BY salary DESC, employee_id;
```

| Employee | Salary | ROW_NUMBER | RANK | DENSE_RANK |
|---|---:|---:|---:|---:|
| Asha | 120000 | 1 | 1 | 1 |
| Dev | 90000 | 2 | 2 | 2 |
| Mira | 90000 | 3 | 2 | 2 |
| Noor | 70000 | 4 | 4 | 3 |
| Ishan | 70000 | 5 | 4 | 3 |
| Tara | 60000 | 6 | 6 | 4 |

ROW_NUMBER assigns unique positions. RANK preserves ties and leaves gaps. DENSE_RANK preserves ties without gaps. Notice that employee_id is a tie-breaker **only** for ROW_NUMBER; adding it to the rank ordering would stop equal salaries from being peers.

## Top N per group

```sql
WITH ranked AS (
    SELECT employee_id, employee_name, department_id, salary,
           ROW_NUMBER() OVER (
               PARTITION BY department_id
               ORDER BY salary DESC, employee_id
           ) AS position
    FROM employees
)
SELECT employee_id, employee_name, department_id, salary
FROM ranked
WHERE position <= 2
ORDER BY department_id, position;
```

This returns at most two employees per department. DENSE_RANK by salary would return all employees in the top two **salary levels**, possibly more than two rows. State the tie policy before choosing a function.

The outer query is needed because WHERE in the same query level cannot generally refer to a window result. Some warehouse dialects offer QUALIFY; do not assume it exists in every engine.

## LAG and LEAD

```sql
SELECT order_id, customer_id, order_date,
       LAG(order_date) OVER (
           PARTITION BY customer_id ORDER BY order_date, order_id
       ) AS previous_order_date,
       LEAD(order_date) OVER (
           PARTITION BY customer_id ORDER BY order_date, order_id
       ) AS next_order_date
FROM orders
WHERE status = 'paid'
ORDER BY customer_id, order_date, order_id;
```

LAG looks backward; LEAD looks forward. An optional offset and default can be supplied. A default applies when the offset row is absent, not automatically when the row exists and its value is NULL.

Because WHERE filters to paid orders first, the previous row means previous **paid** order. It does not mean the previous calendar day. Time differences require the correct engine-specific date arithmetic.

## Running totals and moving averages

```sql
WITH daily AS (
    SELECT o.order_date, SUM(oi.quantity * oi.unit_price_cents) AS revenue_cents
    FROM orders AS o
    JOIN order_items AS oi ON oi.order_id = o.order_id
    WHERE o.status = 'paid'
    GROUP BY o.order_date
)
SELECT order_date, revenue_cents,
       SUM(revenue_cents) OVER (
           ORDER BY order_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_revenue_cents,
       AVG(revenue_cents) OVER (
           ORDER BY order_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS trailing_three_observed_dates_average
FROM daily
ORDER BY order_date;
```

The final running total is **28,000**. The moving average is over up to three observed date rows, not necessarily three consecutive calendar days. Fill a calendar first or use a supported time-based frame for a true duration window.

Near the beginning, the frame contains fewer than three rows. Decide whether partial windows should be shown or filtered based on `COUNT(*) OVER (...)`.

## ROWS, RANGE, GROUPS, and peers

| Frame unit | Meaning | Example use |
|---|---|---|
| ROWS | Physical row positions in the window ordering | Last 3 records |
| RANGE | Ordering-value range; peers share relevant boundary values | Time/value interval where supported |
| GROUPS | Number of peer groups | Previous distinct ordered-value groups where supported |

Peers have equal values for all expressions in the window ORDER BY. In PostgreSQL and SQLite, an ordered aggregate window without an explicit frame commonly defaults to RANGE from the start through the current peer group. This can make tied values advance together. Dialect/function restrictions differ; explicitly state frames for clarity.

```sql
SELECT employee_id, salary,
       SUM(salary) OVER (ORDER BY salary) AS peer_aware_running_sum,
       SUM(salary) OVER (
           ORDER BY salary, employee_id
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS row_running_sum
FROM employees
ORDER BY salary, employee_id;
```

At the first 70,000 salary, the peer-aware sum already includes both 70,000 employees. The row-based sum with a unique tie-breaker includes only the first one so far.

Common boundaries: UNBOUNDED PRECEDING, `n PRECEDING`, CURRENT ROW, `n FOLLOWING`, and UNBOUNDED FOLLOWING. A frame starting after its end is invalid. GROUPS, interval RANGE offsets, and EXCLUDE clauses are not equally supported across engines. See [SQLite frame semantics](https://www.sqlite.org/windowfunctions.html) and the [dialect guide](12-dialects-and-command-reference.md).

## FIRST_VALUE, LAST_VALUE, and NTH_VALUE

LAST_VALUE means last value **in the frame**, not necessarily last in the partition. Use a full-partition frame when that is intended:

```sql
SELECT employee_id, department_id, salary,
       FIRST_VALUE(salary) OVER (
           PARTITION BY department_id ORDER BY salary DESC, employee_id
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS highest_salary,
       LAST_VALUE(salary) OVER (
           PARTITION BY department_id ORDER BY salary DESC, employee_id
           ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
       ) AS lowest_salary
FROM employees
ORDER BY employee_id;
```

If only a numeric minimum/maximum is required, MIN/MAX over the partition can be simpler. Value functions are useful when you need an associated field ordered by another field. NTH_VALUE selects a position in its frame where supported; SQL Server does not have the same complete value-function set as PostgreSQL.

Null-skipping options such as IGNORE NULLS differ by engine and version; do not paste them across dialects without checking support.

## Distribution functions and named windows

```sql
SELECT employee_id, salary,
       NTILE(3) OVER (ORDER BY salary, employee_id) AS bucket,
       PERCENT_RANK() OVER (ORDER BY salary) AS relative_rank,
       CUME_DIST() OVER (ORDER BY salary) AS cumulative_fraction
FROM employees
ORDER BY salary, employee_id;
```

NTILE divides rows into roughly equal-count buckets, not equal value ranges, and can split equal values across buckets. PERCENT_RANK relates rank to partition size; CUME_DIST gives the fraction of rows at or below the current peer group in ascending order. They are not interchangeable percentile definitions.

**PostgreSQL/SQLite example:** Name a reusable window:

```sql
SELECT order_id, customer_id,
       ROW_NUMBER() OVER customer_history AS position,
       LAG(order_id) OVER customer_history AS previous_order
FROM orders
WINDOW customer_history AS (
    PARTITION BY customer_id ORDER BY order_date, order_id
)
ORDER BY customer_id, position;
```

Named-window support and extension rules vary; recent SQL Server versions require appropriate support/compatibility for their WINDOW clause.

## Filtering, performance, and practice

Windows may require sorting or buffering. Reduce the correct input first, avoid unnecessary distinct orderings, and inspect plans. An index can help supply ordering, but the engine may still sort or choose another plan.

Practice: second-highest distinct salary; latest paid order per customer; each order's share of customer revenue; month-over-month change after monthly aggregation; a seven-day rolling total with missing dates; explain the LAST_VALUE frame trap.

Next: [Functions, dates, and cleaning](06-functions-dates-and-data-cleaning.md).
