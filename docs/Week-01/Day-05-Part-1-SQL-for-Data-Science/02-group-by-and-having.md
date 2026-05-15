# GROUP BY and HAVING

Raw rows tell you what happened. Aggregations tell you what it means. GROUP BY is how you turn a table of 10 million order records into a two-line summary that shows which region is growing and which is declining. Every analyst uses it constantly.

## Learning Objectives

- Use aggregate functions (COUNT, SUM, AVG, MIN, MAX) to summarise data
- Understand why GROUP BY is required when you mix raw and aggregated columns
- Filter *before* grouping with WHERE and *after* grouping with HAVING
- Group by multiple columns to build multi-dimensional summaries
- Recognise the non-aggregated column trap that produces misleading results

---

## The Schema

```sql
-- orders(order_id, customer_id, product_id, amount, status, created_at)
-- customers(customer_id, name, country, email, created_at)
-- products(product_id, name, category, price)
```

---

## Aggregate Functions

Aggregate functions collapse many rows into one number. They always ignore NULLs — except `COUNT(*)`, which counts rows regardless.

```sql
-- Overall order metrics
SELECT
    COUNT(*)           AS total_orders,        -- counts every row
    COUNT(amount)      AS orders_with_amount,  -- skips NULLs
    SUM(amount)        AS total_revenue,
    AVG(amount)        AS average_order_value,
    MIN(amount)        AS smallest_order,
    MAX(amount)        AS largest_order
FROM orders;
```

> [!warning]
> `COUNT(*)` and `COUNT(column)` are not the same. `COUNT(amount)` skips rows where amount is NULL. If 10% of your rows have a NULL amount, your `COUNT(amount)` will be 10% lower than `COUNT(*)`. Always decide which one you actually want.

---

## GROUP BY — Aggregating by Category

Without GROUP BY, aggregate functions collapse the entire table into one row. With GROUP BY, they collapse *per group*.

```sql
-- Revenue and order count by status
SELECT
    status,
    COUNT(*)      AS order_count,
    SUM(amount)   AS total_revenue,
    AVG(amount)   AS avg_order_value
FROM orders
GROUP BY status;
```

Output:

```
status      | order_count | total_revenue | avg_order_value
------------|-------------|---------------|----------------
completed   | 4821        | 9,432,000     | 1,956.42
pending     | 1203        | 1,840,000     | 1,529.51
cancelled   | 387         | 643,000       | 1,661.24
```

> [!warning]
> Every column in SELECT that is not inside an aggregate function must appear in GROUP BY. If you select a column without grouping by it, the database either throws an error (PostgreSQL, MySQL with strict mode) or picks an arbitrary value from the group (MySQL without strict mode). The arbitrary-value case is silently wrong — your numbers look plausible but are meaningless.

```sql
-- This fails in PostgreSQL (and should fail everywhere)
SELECT customer_id, name, SUM(amount)
FROM orders
GROUP BY customer_id;
-- ERROR: column "name" must appear in GROUP BY or be used in an aggregate

-- Fix: group by all non-aggregated columns
SELECT customer_id, name, SUM(amount)
FROM orders
GROUP BY customer_id, name;
```

---

## Grouping by Multiple Columns

Add columns to GROUP BY to break down your summary further.

```sql
-- Revenue by country and status
SELECT
    c.country,
    o.status,
    COUNT(*)    AS order_count,
    SUM(amount) AS total_revenue
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.country, o.status
ORDER BY c.country, o.status;
```

```sql
-- Monthly revenue by product category
SELECT
    DATE_TRUNC('month', o.created_at) AS month,
    p.category,
    SUM(o.amount)                      AS monthly_revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY DATE_TRUNC('month', o.created_at), p.category
ORDER BY month, category;
```

> [!tip]
> When grouping by dates, use date truncation functions (`DATE_TRUNC` in PostgreSQL, `DATE_FORMAT` in MySQL, `STRFTIME` in SQLite) to group by month or year. Grouping by a raw datetime column will create one group per unique timestamp, which is almost never what you want.

---

## HAVING — Filtering After Aggregation

WHERE filters rows before aggregation. HAVING filters groups after aggregation. The distinction matters because you cannot use aggregate functions in WHERE.

```sql
-- Find customers who have spent more than 5000 in total
-- (Cannot use WHERE here — SUM does not exist yet at the WHERE stage)
SELECT
    customer_id,
    COUNT(*)    AS order_count,
    SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 5000
ORDER BY total_spent DESC;
```

### WHERE and HAVING in the Same Query

Use WHERE to reduce the rows going *into* the aggregation, and HAVING to filter the groups *after*.

```sql
-- High-value completed orders: only count completed orders,
-- and only return customers who spent more than 5000 in completed orders
SELECT
    customer_id,
    COUNT(*)    AS completed_orders,
    SUM(amount) AS completed_revenue
FROM orders
WHERE status = 'completed'          -- runs first: eliminates non-completed rows
GROUP BY customer_id
HAVING SUM(amount) > 5000          -- runs after: keeps only high-value customers
ORDER BY completed_revenue DESC;
```

> [!info]
> The full execution order with GROUP BY is: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`. HAVING is the only clause that runs after GROUP BY, which is why it is the only place where aggregate functions can be used as filters.

### The WHERE vs HAVING Decision

Ask yourself: "Am I filtering individual rows or am I filtering summarised groups?"

| What you want to filter | Use |
|---|---|
| Only look at orders from 2024 | WHERE (row-level filter) |
| Only customers with > 10 orders | HAVING (group-level filter) |
| Only completed orders | WHERE (row-level filter) |
| Only categories with avg price > 50 | HAVING (group-level filter) |

```sql
-- Both together: completed orders in 2024, only high-spending customers
SELECT
    c.name,
    c.country,
    COUNT(*)    AS orders_in_2024,
    SUM(o.amount) AS revenue_in_2024
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'completed'
  AND o.created_at >= '2024-01-01'
  AND o.created_at <  '2025-01-01'
GROUP BY c.customer_id, c.name, c.country
HAVING SUM(o.amount) > 10000
ORDER BY revenue_in_2024 DESC;
```

---

## Useful Patterns

### Count Distinct Values

```sql
-- How many unique customers placed orders each month?
SELECT
    DATE_TRUNC('month', created_at) AS month,
    COUNT(DISTINCT customer_id)     AS unique_customers
FROM orders
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;
```

### Percentage of Total

```sql
-- What share of revenue does each status represent?
SELECT
    status,
    SUM(amount)                               AS status_revenue,
    SUM(amount) * 100.0 / SUM(SUM(amount)) OVER () AS pct_of_total
FROM orders
GROUP BY status;
-- Note: OVER() here is a window function — covered in file 06
```

A simpler approach without window functions:

```sql
WITH totals AS (
    SELECT SUM(amount) AS grand_total FROM orders
)
SELECT
    o.status,
    SUM(o.amount)                             AS status_revenue,
    ROUND(SUM(o.amount) * 100.0 / t.grand_total, 2) AS pct_of_total
FROM orders o
CROSS JOIN totals t
GROUP BY o.status, t.grand_total
ORDER BY status_revenue DESC;
```

### Find Duplicate Records

```sql
-- Find customer_ids that appear more than once in the customers table
-- (should not happen if customer_id is a proper primary key — but check anyway)
SELECT
    email,
    COUNT(*) AS occurrences
FROM customers
GROUP BY email
HAVING COUNT(*) > 1;
```

> [!tip]
> This duplicate-finding pattern — GROUP BY the column(s) that should be unique, HAVING COUNT(*) > 1 — is one of the most useful SQL patterns in data quality work. Run it before every JOIN to verify key uniqueness.

---

## Common Aggregate Functions Reference

| Function | What it does | NULL behaviour |
|---|---|---|
| `COUNT(*)` | Count of all rows | Includes NULLs |
| `COUNT(col)` | Count of non-NULL values | Skips NULLs |
| `COUNT(DISTINCT col)` | Count of unique non-NULL values | Skips NULLs |
| `SUM(col)` | Total | Skips NULLs |
| `AVG(col)` | Mean | Skips NULLs (denominator is non-NULL count) |
| `MIN(col)` | Smallest value | Skips NULLs |
| `MAX(col)` | Largest value | Skips NULLs |

> [!warning]
> `AVG` skips NULLs in both numerator and denominator. `AVG(amount)` on [100, 200, NULL] returns 150, not 100. If NULL should be treated as 0, use `AVG(COALESCE(amount, 0))` instead.

---

## Practice Exercises

**Warm-up**

1. Count the total number of orders in the table.
2. Find total revenue and average order value by `status`.
3. Find the number of orders per product category (join to `products`).

**Main**

4. Find all countries that have generated more than 50,000 in total revenue (join to `customers`).
5. Find customers who placed more than 5 orders. Show their `customer_id` and order count.
6. Find the month with the highest total revenue. Show month and revenue.
7. Find product categories where the average order amount is above the overall average order amount.

**Stretch**

8. Find customers who placed at least 3 completed orders in 2024. Use both WHERE and HAVING.
9. For each country, find the percentage of orders that were cancelled. Show country, cancelled count, total count, and percentage.
10. A table has a `user_id` and `email` column. Write a query to find emails that belong to more than one user_id (potential duplicates). Explain what you would do with the results.

---

## Interview Questions

**Q: What is the difference between WHERE and HAVING?**

WHERE filters individual rows before the database groups them. HAVING filters groups after aggregation. You cannot use aggregate functions like SUM or COUNT in WHERE — those values do not exist yet at that execution stage.

**Q: Why can you use a SELECT alias in ORDER BY but not in WHERE or HAVING?**

Because of execution order: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY. SELECT runs before ORDER BY, so aliases are defined by the time ORDER BY executes. WHERE and HAVING run before SELECT, so the aliases do not exist yet.

**Q: What does COUNT(*) vs COUNT(column) return when there are NULLs?**

`COUNT(*)` counts every row. `COUNT(column)` skips rows where that column is NULL. On a table where 20% of rows have a NULL amount, `COUNT(amount)` will be 20% lower than `COUNT(*)`.

---

> [!success]
> Key takeaway: WHERE filters rows (before grouping), HAVING filters groups (after aggregation). Every non-aggregated column in SELECT must appear in GROUP BY. AVG and SUM silently ignore NULLs.

---

[[01-select-and-where|Previous: SELECT and WHERE]] | [[03-joins|Next: JOINs]]
