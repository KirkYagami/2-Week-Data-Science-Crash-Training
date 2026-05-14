# SQL Cheat Sheet for Data Scientists

SQL is the primary language for extracting, transforming, and aggregating data. As a data scientist, you will write SQL before any modeling begins — to explore datasets, build features, and validate hypotheses.

**Schema used throughout this cheat sheet:**

```sql
-- orders(order_id, customer_id, amount, created_at, status)
-- customers(customer_id, name, country, created_at)
-- products(product_id, name, category, price)
```

---

## SELECT Basics

### SELECT and column aliasing

Use `AS` to rename columns in output — critical when column names are ambiguous or when the downstream tool (pandas, BI dashboard) needs clean names.

```sql
SELECT
    order_id,
    customer_id,
    amount,
    amount * 1.1 AS amount_with_tax   -- alias a computed column
FROM orders;
```

### WHERE — filtering rows

`WHERE` runs before aggregation. Use it to reduce the dataset early and keep queries fast.

```sql
SELECT order_id, amount, status
FROM orders
WHERE status = 'completed'         -- only completed orders
  AND amount > 100;                -- with amount above threshold
```

### ORDER BY — sorting results

Sort descending to find top values, ascending to find outliers at the bottom.

```sql
SELECT order_id, customer_id, amount
FROM orders
WHERE status = 'completed'
ORDER BY amount DESC               -- highest-value orders first
LIMIT 10;                          -- take only the top 10
```

### DISTINCT — deduplicate rows

Use `DISTINCT` when you want the unique set of values, not every occurrence.

```sql
SELECT DISTINCT country
FROM customers
ORDER BY country;                  -- all countries we have customers in
```

### LIMIT — cap result size

Always use `LIMIT` when exploring an unfamiliar table. Running unbounded queries on large tables is slow and expensive.

```sql
SELECT *
FROM orders
LIMIT 5;                           -- quick look at structure and values
```

---

## Filtering

### Comparison operators

Standard operators: `=`, `!=` or `<>`, `>`, `<`, `>=`, `<=`.

```sql
SELECT order_id, amount
FROM orders
WHERE amount >= 500               -- orders at or above 500
  AND status != 'cancelled';     -- exclude cancelled
```

### IN and NOT IN

`IN` replaces a chain of `OR` conditions. `NOT IN` excludes a set — but be careful with NULLs (see NULL Handling section).

```sql
SELECT order_id, status
FROM orders
WHERE status IN ('completed', 'shipped');  -- match any value in the list

SELECT order_id, status
FROM orders
WHERE status NOT IN ('cancelled', 'refunded');
```

### BETWEEN

`BETWEEN` is inclusive on both ends. Use it for date ranges and numeric ranges.

```sql
SELECT order_id, amount, created_at
FROM orders
WHERE amount BETWEEN 100 AND 500      -- 100 and 500 are both included
  AND created_at BETWEEN '2024-01-01' AND '2024-12-31';
```

### LIKE — pattern matching

`%` matches any sequence of characters. `_` matches exactly one character. Case sensitivity depends on the database and collation.

```sql
SELECT name, country
FROM customers
WHERE name LIKE 'A%'           -- names starting with A
  AND country LIKE '%land%';  -- country contains "land" (e.g. Finland, Ireland)
```

### IS NULL and IS NOT NULL

Never use `= NULL` — it always returns false. SQL uses `IS NULL` for this comparison.

```sql
SELECT order_id, customer_id
FROM orders
WHERE customer_id IS NULL;     -- orphaned orders with no linked customer

SELECT order_id, amount
FROM orders
WHERE amount IS NOT NULL;      -- exclude rows with missing amount
```

---

## Aggregations

### COUNT — row and value counts

`COUNT(*)` counts all rows. `COUNT(column)` counts non-NULL values in that column. They differ when NULLs are present.

```sql
SELECT
    COUNT(*) AS total_orders,               -- includes rows with NULL fields
    COUNT(customer_id) AS orders_with_customer  -- excludes rows where customer_id IS NULL
FROM orders;
```

### SUM and AVG

Used constantly for revenue totals, averages, and per-group summaries.

```sql
SELECT
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_order_value,
    MIN(amount) AS smallest_order,
    MAX(amount) AS largest_order
FROM orders
WHERE status = 'completed';
```

### COUNT(DISTINCT) — unique value counts

Use this to count unique customers, products, or any entity — not the number of events.

```sql
SELECT
    COUNT(DISTINCT customer_id) AS unique_customers,  -- how many distinct buyers
    COUNT(*) AS total_orders                          -- how many orders they placed
FROM orders
WHERE status = 'completed';
```

### HAVING — filter on aggregated values

`HAVING` filters after grouping. `WHERE` filters before. You cannot use `WHERE` to filter on a `SUM()` or `COUNT()` result — use `HAVING` instead.

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(amount) AS lifetime_value
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 1000       -- only customers who spent more than 1000 total
ORDER BY lifetime_value DESC;
```

---

## GROUP BY

### Grouping by a single column

The most common pattern in data science: compute a metric broken down by a category.

```sql
SELECT
    status,
    COUNT(*) AS order_count,
    SUM(amount) AS total_amount
FROM orders
GROUP BY status;
-- result: one row per status value, with aggregates
```

### Grouping by multiple columns

Every column in `SELECT` that is not inside an aggregate function must appear in `GROUP BY`.

```sql
SELECT
    c.country,
    o.status,
    COUNT(*) AS order_count,
    AVG(o.amount) AS avg_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.country, o.status     -- one row per country + status combination
ORDER BY c.country, order_count DESC;
```

### Using HAVING with GROUP BY

Filter groups after aggregation — for example, to find high-value segments.

```sql
SELECT
    country,
    COUNT(DISTINCT customer_id) AS customer_count
FROM customers
GROUP BY country
HAVING COUNT(DISTINCT customer_id) >= 100   -- only countries with 100+ customers
ORDER BY customer_count DESC;
```

### Grouping by a computed expression

You can group by a derived value — useful for grouping by year, month, or a bucketed numeric range.

```sql
SELECT
    EXTRACT(YEAR FROM created_at) AS order_year,
    EXTRACT(MONTH FROM created_at) AS order_month,
    COUNT(*) AS order_count,
    SUM(amount) AS monthly_revenue
FROM orders
GROUP BY
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at)
ORDER BY order_year, order_month;
```

---

## JOINs

### INNER JOIN — keep only matching rows

```sql
-- INNER JOIN: only rows that exist in BOTH tables
--
--   orders      customers
--   [ A  ]  +  [ A ] --> A (kept)
--   [ B  ]                  (dropped — no matching customer)
--              [ C ]        (dropped — no matching order)

SELECT
    o.order_id,
    c.name AS customer_name,
    c.country,
    o.amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
-- customers with no orders and orders with no customer are both excluded
```

### LEFT JOIN — keep all rows from the left table

```sql
-- LEFT JOIN: all rows from orders, matched rows from customers (NULL if no match)
--
--   orders      customers
--   [ A  ] --> [ A ] --> A with customer data
--   [ B  ] --> no match --> B with NULLs for customer columns

SELECT
    o.order_id,
    o.amount,
    c.name AS customer_name       -- NULL if no customer record exists
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id;
```

### RIGHT JOIN — keep all rows from the right table

Right joins are rare. Most practitioners convert them to left joins by swapping the table order — it is easier to read.

```sql
-- Equivalent to a LEFT JOIN with tables swapped
SELECT
    o.order_id,
    c.name AS customer_name
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id;
-- all customers appear, even those who have never ordered
```

### FULL OUTER JOIN — keep all rows from both tables

Use when you need to reconcile two datasets and detect what is missing on either side.

```sql
-- FULL OUTER JOIN: all rows from both tables, NULLs where there is no match
SELECT
    o.order_id,
    c.customer_id,
    c.name
FROM orders o
FULL OUTER JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL           -- customers who have never placed an order
   OR c.customer_id IS NULL;       -- orders with no matching customer record
```

---

## Subqueries

### Scalar subquery — single value in SELECT or WHERE

A scalar subquery returns exactly one row and one column. Use it to compare each row against a global metric.

```sql
SELECT
    order_id,
    amount,
    amount - (SELECT AVG(amount) FROM orders) AS deviation_from_mean
    -- subquery runs once, returns a single number
FROM orders
WHERE status = 'completed';
```

### IN subquery — filter using a set from another query

Use when you need to filter one table based on values in another table without doing a join.

```sql
SELECT name, country
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM orders
    WHERE status = 'completed'    -- only customers who have completed at least one order
      AND amount > 500
);
```

### Correlated subquery — references the outer query

The subquery runs once per row in the outer query. It is usually slower than a join but sometimes the clearest way to express a condition.

```sql
SELECT
    o.order_id,
    o.customer_id,
    o.amount
FROM orders o
WHERE o.amount > (
    SELECT AVG(o2.amount)
    FROM orders o2
    WHERE o2.customer_id = o.customer_id  -- avg for THIS customer only
);
-- finds orders where the amount exceeds that customer's personal average
```

### Subquery in FROM clause (derived table)

Treat a subquery as a temporary table. This is the foundation of multi-step transformations before CTEs existed.

```sql
SELECT
    country,
    AVG(customer_lifetime_value) AS avg_ltv
FROM (
    SELECT
        c.customer_id,
        c.country,
        SUM(o.amount) AS customer_lifetime_value
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.country
) AS customer_ltv
GROUP BY country
ORDER BY avg_ltv DESC;
```

---

## CTEs

### Basic CTE with WITH

CTEs make complex queries readable by naming intermediate results. They are not faster than subqueries in most databases, but they are far easier to debug and maintain.

```sql
WITH completed_orders AS (
    -- step 1: isolate completed orders
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE status = 'completed'
)
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_spent
FROM completed_orders
GROUP BY customer_id
ORDER BY total_spent DESC;
```

### Chaining multiple CTEs

Each CTE can reference the ones defined before it. This is the SQL equivalent of building a pipeline step by step.

```sql
WITH completed_orders AS (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE status = 'completed'
),
customer_totals AS (
    -- step 2: aggregate per customer, references the first CTE
    SELECT
        customer_id,
        COUNT(*) AS order_count,
        SUM(amount) AS lifetime_value
    FROM completed_orders
    GROUP BY customer_id
),
high_value_customers AS (
    -- step 3: filter to high-value segment, references the second CTE
    SELECT customer_id, lifetime_value
    FROM customer_totals
    WHERE lifetime_value > 1000
)
SELECT
    c.name,
    c.country,
    hvc.lifetime_value
FROM high_value_customers hvc
JOIN customers c ON hvc.customer_id = c.customer_id
ORDER BY hvc.lifetime_value DESC;
```

### Recursive CTE — brief overview

Recursive CTEs are used for hierarchical data (org charts, category trees). They are rare in flat analytical schemas but worth knowing.

```sql
-- Pattern only — not applicable to the flat orders schema
WITH RECURSIVE hierarchy AS (
    -- anchor: starting point (e.g. root node)
    SELECT id, parent_id, name, 1 AS depth
    FROM category
    WHERE parent_id IS NULL

    UNION ALL

    -- recursive: join back to itself until no more children
    SELECT c.id, c.parent_id, c.name, h.depth + 1
    FROM category c
    JOIN hierarchy h ON c.parent_id = h.id
)
SELECT id, name, depth
FROM hierarchy
ORDER BY depth, name;
```

---

## Window Functions

Window functions compute values across a set of rows related to the current row, without collapsing rows the way `GROUP BY` does.

### ROW_NUMBER — unique sequential rank

Use to deduplicate rows (keep the latest record per customer) or to paginate results.

```sql
SELECT
    order_id,
    customer_id,
    amount,
    created_at,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id          -- restart numbering for each customer
        ORDER BY created_at DESC          -- most recent order gets number 1
    ) AS order_rank
FROM orders;
-- filter WHERE order_rank = 1 to get each customer's most recent order
```

### RANK and DENSE_RANK — ranking with ties

`RANK` leaves gaps after ties (1, 2, 2, 4). `DENSE_RANK` does not (1, 2, 2, 3). Use `DENSE_RANK` when the gap would be confusing.

```sql
SELECT
    customer_id,
    amount,
    RANK() OVER (ORDER BY amount DESC) AS rank_with_gaps,
    DENSE_RANK() OVER (ORDER BY amount DESC) AS rank_no_gaps
FROM orders
WHERE status = 'completed';
```

### LAG and LEAD — access adjacent rows

Use `LAG` to compare a row to the previous row (e.g. month-over-month change). Use `LEAD` to look forward.

```sql
SELECT
    order_id,
    customer_id,
    amount,
    created_at,
    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY created_at
    ) AS previous_order_amount,
    amount - LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY created_at
    ) AS change_from_previous
FROM orders
WHERE status = 'completed';
```

### Running totals with SUM OVER

Cumulative sums are a window function, not a `GROUP BY`. The `ROWS UNBOUNDED PRECEDING` frame clause controls which rows are included.

```sql
SELECT
    order_id,
    customer_id,
    amount,
    created_at,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_per_customer
FROM orders
WHERE status = 'completed';
```

### NTILE — divide rows into buckets

Use to create quartiles, deciles, or any equal-size grouping for segmentation.

```sql
SELECT
    customer_id,
    SUM(amount) AS lifetime_value,
    NTILE(4) OVER (ORDER BY SUM(amount) DESC) AS value_quartile
    -- 1 = top 25%, 4 = bottom 25%
FROM orders
WHERE status = 'completed'
GROUP BY customer_id;
```

---

## String Functions

### UPPER, LOWER, TRIM

Normalize strings before grouping or joining on text columns. Inconsistent casing causes silent mismatches.

```sql
SELECT
    UPPER(country) AS country_upper,      -- 'GERMANY'
    LOWER(name) AS name_lower,            -- 'alice smith'
    TRIM(name) AS name_trimmed            -- removes leading and trailing spaces
FROM customers;
```

### CONCAT — joining strings

Use to build full names, labels, or composite keys.

```sql
SELECT
    CONCAT(name, ' (', country, ')') AS display_label   -- 'Alice Smith (Germany)'
FROM customers;

-- PostgreSQL also supports the || operator:
SELECT name || ' — ' || country AS label
FROM customers;
```

### SUBSTRING — extract part of a string

```sql
SELECT
    name,
    SUBSTRING(name, 1, 3) AS name_prefix  -- first 3 characters (1-indexed in SQL)
FROM customers;

-- Extract domain from an email stored in the name column (example only):
SUBSTRING(name FROM POSITION('@' IN name) + 1)
```

### REPLACE — swap substrings

Use to clean data in place, for example stripping currency symbols or fixing inconsistent codes.

```sql
SELECT
    order_id,
    REPLACE(status, '_', ' ') AS status_clean  -- 'in_progress' -> 'in progress'
FROM orders;
```

### LIKE patterns for data profiling

Using `LIKE` in aggregations helps you profile messy string columns quickly.

```sql
SELECT
    COUNT(*) FILTER (WHERE name LIKE '% %') AS has_space,      -- PostgreSQL syntax
    COUNT(*) FILTER (WHERE name NOT LIKE '% %') AS single_token
FROM customers;
```

---

## Date Functions

Date handling is highly dialect-specific. The examples below use **PostgreSQL** syntax. MySQL equivalents are flagged.

### DATE_TRUNC — round down to a period

Use to aggregate by week, month, or quarter. This is the most useful date function for time-series analysis.

```sql
-- PostgreSQL
SELECT
    DATE_TRUNC('month', created_at) AS month,    -- e.g. 2024-03-01 00:00:00
    COUNT(*) AS order_count,
    SUM(amount) AS monthly_revenue
FROM orders
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;

-- MySQL equivalent:
-- DATE_FORMAT(created_at, '%Y-%m-01')
```

### EXTRACT — pull out a specific field

Use for grouping by hour of day, day of week, or month number.

```sql
-- PostgreSQL
SELECT
    EXTRACT(YEAR FROM created_at) AS yr,
    EXTRACT(MONTH FROM created_at) AS mo,
    EXTRACT(DOW FROM created_at) AS day_of_week   -- 0 = Sunday, 6 = Saturday
FROM orders;

-- MySQL equivalent: YEAR(), MONTH(), DAYOFWEEK()
```

### Date arithmetic — intervals and differences

```sql
-- PostgreSQL: subtract dates to get an interval, or add intervals
SELECT
    order_id,
    created_at,
    created_at + INTERVAL '30 days' AS due_date,   -- add 30 days
    CURRENT_DATE - created_at::date AS days_since_order
FROM orders;

-- MySQL equivalent:
-- DATE_ADD(created_at, INTERVAL 30 DAY)
-- DATEDIFF(CURRENT_DATE, created_at)
```

### DATE_DIFF — difference between two dates

```sql
-- PostgreSQL (using subtraction — returns an interval)
SELECT
    customer_id,
    MIN(created_at) AS first_order,
    MAX(created_at) AS last_order,
    MAX(created_at)::date - MIN(created_at)::date AS days_as_customer
FROM orders
GROUP BY customer_id;

-- MySQL: DATEDIFF(MAX(created_at), MIN(created_at))
-- BigQuery: DATE_DIFF(MAX(created_at), MIN(created_at), DAY)
```

### CURRENT_DATE and NOW

```sql
SELECT
    order_id,
    created_at,
    CURRENT_DATE AS today,                             -- date only, no time
    NOW() AS current_timestamp,                        -- date + time
    CURRENT_DATE - created_at::date AS age_in_days
FROM orders
WHERE created_at >= CURRENT_DATE - INTERVAL '90 days'; -- last 90 days
```

---

## CASE WHEN

### Simple CASE — match a column to fixed values

Use when you want to recode a categorical column into new labels.

```sql
SELECT
    order_id,
    status,
    CASE status
        WHEN 'completed' THEN 'Success'
        WHEN 'shipped'   THEN 'In Transit'
        WHEN 'cancelled' THEN 'Lost'
        ELSE 'Other'                         -- always include ELSE to handle unknowns
    END AS status_label
FROM orders;
```

### Searched CASE — condition-based logic

More flexible: each WHEN clause is a full boolean expression. Use for bucketing numeric values.

```sql
SELECT
    order_id,
    amount,
    CASE
        WHEN amount < 50  THEN 'low'
        WHEN amount < 200 THEN 'medium'
        WHEN amount < 500 THEN 'high'
        ELSE 'enterprise'
    END AS order_tier
FROM orders;
```

### CASE inside an aggregation — conditional counts

This is the SQL equivalent of a pivot table. It avoids multiple self-joins.

```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders,
    COUNT(CASE WHEN status = 'completed' THEN 1 END) AS completed_orders,
    COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled_orders,
    SUM(CASE WHEN status = 'completed' THEN amount ELSE 0 END) AS revenue_from_completed
FROM orders
GROUP BY customer_id;
```

### CASE in ORDER BY — custom sort order

Use when alphabetical or numeric order does not reflect business priority.

```sql
SELECT order_id, status, amount
FROM orders
ORDER BY
    CASE status
        WHEN 'cancelled' THEN 1   -- cancelled first for review
        WHEN 'shipped'   THEN 2
        WHEN 'completed' THEN 3
        ELSE 4
    END,
    amount DESC;
```

---

## NULL Handling

### COALESCE — return the first non-NULL value

`COALESCE` is the standard way to substitute a default value for NULLs. It accepts any number of arguments and returns the first non-NULL one.

```sql
SELECT
    order_id,
    COALESCE(customer_id, -1) AS customer_id_clean,   -- replace NULL with -1
    COALESCE(amount, 0) AS amount_clean                -- treat missing amount as 0
FROM orders;
```

### NULLIF — return NULL when two values are equal

Use to avoid division-by-zero errors and to blank out sentinel values (like -1 or 'N/A') before aggregation.

```sql
SELECT
    order_id,
    amount,
    -- if total_count is 0, NULLIF returns NULL instead, preventing division by zero
    amount / NULLIF(total_count, 0) AS amount_per_item
FROM orders;

-- clean up a column where 'N/A' was used instead of NULL
SELECT NULLIF(country, 'N/A') AS country_clean
FROM customers;
```

### NULL in aggregations — what gets silently excluded

`SUM`, `AVG`, `MIN`, `MAX`, and `COUNT(column)` all ignore NULLs. `COUNT(*)` does not. This discrepancy causes subtle bugs.

```sql
SELECT
    COUNT(*)          AS total_rows,           -- includes NULL amounts
    COUNT(amount)     AS non_null_amounts,     -- excludes NULL amounts
    SUM(amount)       AS total,                -- NULLs treated as if absent, not zero
    AVG(amount)       AS average,              -- denominator = non-NULL row count only
    AVG(COALESCE(amount, 0)) AS average_with_zero_fill  -- treat NULL as 0
FROM orders;
```

### NOT IN with NULLs — a silent trap

If the subquery in `NOT IN` returns any NULL, the entire result is empty. This is one of the most common SQL bugs.

```sql
-- DANGEROUS: if ANY customer_id in the subquery is NULL, this returns 0 rows
SELECT * FROM orders
WHERE customer_id NOT IN (SELECT customer_id FROM customers);

-- SAFE: use NOT EXISTS instead, or filter NULLs from the subquery
SELECT * FROM orders o
WHERE NOT EXISTS (
    SELECT 1
    FROM customers c
    WHERE c.customer_id = o.customer_id
);

-- Also safe: filter NULLs explicitly
SELECT * FROM orders
WHERE customer_id NOT IN (
    SELECT customer_id FROM customers WHERE customer_id IS NOT NULL
);
```

---

## Performance Tips

### Avoid SELECT *

Fetching all columns forces the database to read more data from disk and transfers more bytes across the network. Name only the columns you need.

```sql
-- slow and wasteful on wide tables
SELECT * FROM orders;

-- intentional and efficient
SELECT order_id, customer_id, amount, status
FROM orders
WHERE status = 'completed';
```

### Filter early — push WHERE clauses down

The earlier you filter, the fewer rows travel through the rest of the query. In CTEs and subqueries, filter inside the CTE, not after.

```sql
-- better: filter inside the CTE so subsequent joins operate on fewer rows
WITH recent_orders AS (
    SELECT order_id, customer_id, amount
    FROM orders
    WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'   -- filter here
      AND status = 'completed'
)
SELECT c.country, SUM(r.amount)
FROM recent_orders r
JOIN customers c ON r.customer_id = c.customer_id
GROUP BY c.country;
```

### Avoid functions on indexed columns in WHERE

Wrapping an indexed column in a function prevents the database from using the index, causing a full table scan.

```sql
-- slow: function on the column prevents index use
SELECT * FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024;

-- fast: range condition on the raw column lets the index work
SELECT * FROM orders
WHERE created_at >= '2024-01-01'
  AND created_at < '2025-01-01';
```

### Use indexes on JOIN and filter columns

Indexes on `customer_id` in both `orders` and `customers` make joins faster. You do not create indexes in a query, but you can check whether they exist.

```sql
-- Check existing indexes in PostgreSQL
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'orders';

-- A join on an unindexed column causes a sequential scan on one or both tables
-- If you own the schema, add indexes on foreign keys and frequent filter columns:
-- CREATE INDEX idx_orders_customer_id ON orders(customer_id);
-- CREATE INDEX idx_orders_status ON orders(status);
-- CREATE INDEX idx_orders_created_at ON orders(created_at);
```

### EXPLAIN — read the query plan

Run `EXPLAIN` (or `EXPLAIN ANALYZE` to execute and time it) before optimizing. Guessing at performance is slower than reading the plan.

```sql
-- PostgreSQL: show the execution plan without running the query
EXPLAIN
SELECT c.country, SUM(o.amount)
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'completed'
GROUP BY c.country;

-- PostgreSQL: run the query and show actual timing and row counts
EXPLAIN ANALYZE
SELECT c.country, SUM(o.amount)
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'completed'
GROUP BY c.country;

-- Look for: Seq Scan on large tables (bad), Hash Join vs Nested Loop, high row estimates vs actuals
```

### Avoid correlated subqueries on large tables

A correlated subquery runs once per row in the outer query. On a million-row table, that is a million subquery executions. Rewrite as a join or a window function.

```sql
-- slow: correlated subquery runs once per order row
SELECT order_id, amount,
    (SELECT AVG(amount) FROM orders o2 WHERE o2.customer_id = o.customer_id) AS customer_avg
FROM orders o;

-- fast: window function computes all customer averages in one pass
SELECT
    order_id,
    amount,
    AVG(amount) OVER (PARTITION BY customer_id) AS customer_avg
FROM orders;
```
