# Window Functions

Window functions are the most powerful feature in SQL that most beginners have never used. GROUP BY forces a trade-off: get aggregated numbers or get row-level detail, but not both. Window functions break that constraint — you can compute a running total, assign a rank, or calculate a per-group average while keeping every individual row in the result. If you have ever exported data to Python just to compute a rank or a lag, window functions will change how you work.

## Learning Objectives

- Understand the OVER() clause and how it defines the window
- Use PARTITION BY to apply calculations within groups
- Use ORDER BY inside OVER() to define row sequence for rankings and running calculations
- Distinguish ROW_NUMBER, RANK, and DENSE_RANK — and know which to use when
- Compute running totals, moving averages, and period-over-period comparisons with LAG/LEAD
- Use NTILE to bucket rows into quantiles
- Understand how window functions interact with WHERE and HAVING

---

## The Schema

```sql
-- orders(order_id, customer_id, product_id, amount, status, created_at)
-- customers(customer_id, name, country, email, created_at)
-- products(product_id, name, category, price)
```

---

## The Core Concept: OVER()

The OVER() clause is what makes a function a window function. It tells the database: "compute this function across a defined set of rows related to the current row, without collapsing them."

```sql
-- GROUP BY version: collapses to one row per country
SELECT country, AVG(amount) AS avg_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY country;

-- Window function version: keeps every order row, adds country average
SELECT
    o.order_id,
    o.amount,
    c.country,
    AVG(o.amount) OVER (PARTITION BY c.country) AS country_avg_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

Output of the window version:

```
order_id | amount | country | country_avg_amount
---------|--------|---------|-------------------
101      | 450    | India   | 612.50
102      | 775    | India   | 612.50
103      | 300    | USA     | 425.00
104      | 550    | USA     | 425.00
```

Every row survives. The country average is attached to each individual order.

---

## PARTITION BY — Applying Windows to Groups

PARTITION BY divides rows into groups for the window calculation. Without PARTITION BY, the entire result set is one window.

```sql
-- Without PARTITION BY: one window = all rows
SELECT
    order_id,
    amount,
    AVG(amount) OVER () AS overall_avg   -- one number for all rows
FROM orders;

-- With PARTITION BY: one window per status
SELECT
    order_id,
    status,
    amount,
    AVG(amount) OVER (PARTITION BY status) AS status_avg_amount
FROM orders;
```

> [!info]
> PARTITION BY in window functions is analogous to GROUP BY in aggregate queries, but it does not collapse rows. Think of it as: "for the current row, define which other rows are in my group."

---

## ORDER BY Inside OVER() — Sequence-Dependent Calculations

Adding ORDER BY to the OVER() clause changes the window from "all rows in the partition" to "all rows in the partition up to and including the current row" (for most functions). This is what enables running totals and rankings.

```sql
-- Running total of revenue ordered by date
SELECT
    order_id,
    created_at,
    amount,
    SUM(amount) OVER (
        ORDER BY created_at
    ) AS running_total
FROM orders
ORDER BY created_at;
```

```
order_id | created_at | amount | running_total
---------|------------|--------|---------------
1001     | 2024-01-03 | 250    | 250
1002     | 2024-01-05 | 400    | 650
1003     | 2024-01-07 | 175    | 825
```

> [!warning]
> Without ORDER BY in OVER(), functions like SUM and AVG look at all rows in the partition — they produce a single value repeated per row (like a GROUP BY). With ORDER BY, they become cumulative — they include only rows up to the current one. The same function, very different results.

---

## Ranking Functions

### ROW_NUMBER

Assigns a unique sequential integer to each row. No ties — every row gets a distinct number.

```sql
-- Rank orders by amount, most expensive first
SELECT
    order_id,
    customer_id,
    amount,
    ROW_NUMBER() OVER (ORDER BY amount DESC) AS row_num
FROM orders;
```

```sql
-- Rank each customer's orders by amount, within each customer
SELECT
    order_id,
    customer_id,
    amount,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY amount DESC
    ) AS order_rank_per_customer
FROM orders;
```

Use ROW_NUMBER when you want exactly one result per group (e.g., the single most recent order per customer) — because it never ties.

### RANK

RANK assigns the same rank to tied rows but skips the following ranks. Positions 1, 2, 2, 4 (position 3 is skipped because two rows tied for second).

```sql
SELECT
    order_id,
    amount,
    RANK() OVER (ORDER BY amount DESC) AS revenue_rank
FROM orders;
```

```
order_id | amount | revenue_rank
---------|--------|-------------
1005     | 900    | 1
1002     | 750    | 2
1008     | 750    | 2           <- tied
1001     | 600    | 4           <- 3 is skipped
```

### DENSE_RANK

Like RANK, but does not skip numbers. Positions 1, 2, 2, 3.

```sql
SELECT
    order_id,
    amount,
    DENSE_RANK() OVER (ORDER BY amount DESC) AS dense_revenue_rank
FROM orders;
```

```
order_id | amount | dense_revenue_rank
---------|--------|-------------------
1005     | 900    | 1
1002     | 750    | 2
1008     | 750    | 2           <- tied
1001     | 600    | 3           <- no gap
```

**When to use which:**

| Use case | Function |
|---|---|
| Select exactly one row per group | ROW_NUMBER |
| Leaderboards, competition rankings | RANK |
| Ordinal position without gaps | DENSE_RANK |
| "Second highest" problems | DENSE_RANK |

```sql
-- Classic interview problem: find the 2nd highest order amount per customer
-- Wrong approach: SELECT MAX(amount) WHERE amount != (SELECT MAX...) -- fragile
-- Correct approach: window function

WITH ranked_orders AS (
    SELECT
        customer_id,
        amount,
        DENSE_RANK() OVER (
            PARTITION BY customer_id
            ORDER BY amount DESC
        ) AS rnk
    FROM orders
)
SELECT customer_id, amount AS second_highest_amount
FROM ranked_orders
WHERE rnk = 2;
```

---

## Filtering Window Function Results

Window functions are computed in SELECT — after WHERE and GROUP BY. You cannot filter on a window function result in the same WHERE clause. Use a CTE or subquery.

```sql
-- This FAILS — window functions cannot be used in WHERE directly
SELECT order_id, amount, ROW_NUMBER() OVER (ORDER BY amount DESC) AS rn
FROM orders
WHERE rn = 1;    -- ERROR: column "rn" does not exist

-- Correct: wrap in a CTE or subquery
WITH ranked AS (
    SELECT
        order_id,
        customer_id,
        amount,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
    FROM orders
)
SELECT order_id, customer_id, amount
FROM ranked
WHERE rn = 1;   -- most recent order per customer
```

---

## Running Totals and Moving Averages

### Running Total

```sql
-- Cumulative revenue by date
SELECT
    DATE_TRUNC('day', created_at) AS order_date,
    SUM(amount)                   AS daily_revenue,
    SUM(SUM(amount)) OVER (
        ORDER BY DATE_TRUNC('day', created_at)
    )                             AS cumulative_revenue
FROM orders
GROUP BY DATE_TRUNC('day', created_at)
ORDER BY order_date;
```

> [!info]
> `SUM(SUM(amount)) OVER (...)` nests an aggregate inside a window function. The inner SUM groups by date; the outer SUM accumulates those daily totals. This pattern appears constantly in revenue analysis.

### Moving Average

Use the ROWS BETWEEN frame specification to define how many preceding rows to include.

```sql
-- 7-day moving average of daily revenue
SELECT
    DATE_TRUNC('day', created_at)  AS order_date,
    SUM(amount)                    AS daily_revenue,
    AVG(SUM(amount)) OVER (
        ORDER BY DATE_TRUNC('day', created_at)
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW   -- 7 rows: 6 before + current
    )                              AS moving_avg_7d
FROM orders
GROUP BY DATE_TRUNC('day', created_at)
ORDER BY order_date;
```

**Frame specification options:**

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- cumulative (running total)
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW          -- 7-period moving window
ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING          -- centered window
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING  -- reverse cumulative
```

---

## LAG and LEAD — Accessing Adjacent Rows

LAG accesses the value from a previous row. LEAD accesses the value from a following row. Both require ORDER BY in the OVER clause to define "previous" and "next".

```sql
-- Month-over-month revenue with absolute and percentage change
WITH monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', created_at) AS month,
        SUM(amount)                     AS monthly_revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', created_at)
)
SELECT
    month,
    monthly_revenue,
    LAG(monthly_revenue) OVER (ORDER BY month) AS prev_month_revenue,
    monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY month) AS abs_change,
    ROUND(
        (monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY month))
        * 100.0
        / NULLIF(LAG(monthly_revenue) OVER (ORDER BY month), 0),
        2
    ) AS pct_change
FROM monthly_revenue
ORDER BY month;
```

> [!tip]
> Use `NULLIF(denominator, 0)` when dividing by LAG values. If the previous month revenue was 0, you get a division by zero error. NULLIF converts 0 to NULL, making the division return NULL instead of crashing.

LAG with a custom offset — compare to 12 months ago for year-over-year:

```sql
-- Year-over-year comparison: this month vs same month last year
WITH monthly AS (
    SELECT
        DATE_TRUNC('month', created_at) AS month,
        SUM(amount)                     AS revenue
    FROM orders
    GROUP BY 1
)
SELECT
    month,
    revenue,
    LAG(revenue, 12) OVER (ORDER BY month) AS same_month_last_year,
    ROUND(
        (revenue - LAG(revenue, 12) OVER (ORDER BY month))
        * 100.0
        / NULLIF(LAG(revenue, 12) OVER (ORDER BY month), 0),
        1
    ) AS yoy_pct_change
FROM monthly
ORDER BY month;
```

---

## NTILE — Bucketing into Quantiles

NTILE(n) divides rows into n groups of approximately equal size and assigns each row a bucket number.

```sql
-- Divide customers into 4 quartiles by total spend
WITH customer_spend AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spend
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_spend,
    NTILE(4) OVER (ORDER BY total_spend DESC) AS spend_quartile
    -- 1 = top 25%, 4 = bottom 25%
FROM customer_spend
ORDER BY total_spend DESC;
```

```sql
-- Find what threshold separates the top 10% of orders
WITH order_percentiles AS (
    SELECT
        order_id,
        amount,
        NTILE(10) OVER (ORDER BY amount DESC) AS decile
    FROM orders
)
SELECT MIN(amount) AS top_10pct_threshold
FROM order_percentiles
WHERE decile = 1;
```

---

## FIRST_VALUE and LAST_VALUE

Return the first or last value in the window partition, useful for "what was the opening or closing value."

```sql
-- For each order, show the first and most recent order amount for that customer
SELECT
    order_id,
    customer_id,
    amount,
    created_at,
    FIRST_VALUE(amount) OVER (
        PARTITION BY customer_id
        ORDER BY created_at
    ) AS first_order_amount,
    LAST_VALUE(amount) OVER (
        PARTITION BY customer_id
        ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS latest_order_amount
FROM orders;
```

> [!warning]
> LAST_VALUE requires an explicit frame of `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`. Without it, the default frame is "rows from the start of the partition to the current row," so LAST_VALUE returns the current row's value rather than the last row in the partition.

---

## Complete Example: Sales Performance Dashboard

One query that an analyst might actually need — combining rankings, running totals, and LAG.

```sql
WITH daily_sales AS (
    SELECT
        DATE_TRUNC('day', created_at)  AS sale_date,
        COUNT(*)                        AS orders,
        SUM(amount)                     AS revenue
    FROM orders
    WHERE status = 'completed'
      AND created_at >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY 1
),
enriched AS (
    SELECT
        sale_date,
        orders,
        revenue,
        -- Running total
        SUM(revenue) OVER (ORDER BY sale_date)
            AS cumulative_revenue,
        -- 7-day moving average
        AVG(revenue) OVER (
            ORDER BY sale_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        )                               AS revenue_ma_7d,
        -- Day-over-day change
        revenue - LAG(revenue) OVER (ORDER BY sale_date)
            AS daily_change,
        -- Rank this day among all days in the period
        RANK() OVER (ORDER BY revenue DESC)
            AS revenue_rank
    FROM daily_sales
)
SELECT *
FROM enriched
ORDER BY sale_date;
```

---

## Practice Exercises

**Warm-up**

1. Add a column to each order row showing the average order amount for that customer (across all their orders). Do not collapse rows.
2. Rank all orders by amount descending. Use ROW_NUMBER, RANK, and DENSE_RANK in the same query and compare the results where amounts are equal.
3. Calculate the running total of revenue ordered by `created_at`.

**Main**

4. Find each customer's most recent order. Return one row per customer with the order details.
5. Calculate month-over-month revenue change (absolute and percentage) for the last 12 months.
6. Divide all customers into 5 equal groups by total spend. Show the spend range for each quintile.
7. For each order, show how many days have passed since that customer's previous order.

**Stretch**

8. Find all orders where the amount is more than 2x the 7-day moving average at the time of the order. These could indicate anomalies or promotional spikes.
9. Calculate a 3-month rolling retention rate: for each month, what percentage of customers from 3 months ago placed an order this month?
10. Explain why window functions cannot be used in a WHERE clause, and demonstrate the correct pattern for filtering on a window function result.

---

## Interview Questions

**Q: What is the difference between GROUP BY and a window function?**

GROUP BY collapses rows — you lose row-level detail in exchange for aggregate values. Window functions compute aggregate or ranking values while keeping every individual row. Use GROUP BY when you want one row per group. Use window functions when you want the aggregated value alongside the original rows.

**Q: What is the difference between ROW_NUMBER, RANK, and DENSE_RANK?**

All three assign numbers based on ORDER BY. ROW_NUMBER assigns a unique sequential number — no ties possible. RANK assigns the same number to ties but skips subsequent numbers (1, 2, 2, 4). DENSE_RANK assigns the same number to ties without skipping (1, 2, 2, 3). Use ROW_NUMBER to select exactly one row per group. Use DENSE_RANK for "nth highest" problems.

**Q: What does PARTITION BY do in a window function?**

PARTITION BY divides the result set into groups. The window function is applied independently within each partition. Without PARTITION BY, the entire result set is one window.

**Q: Why does LAST_VALUE sometimes return the current row instead of the last row in the partition?**

Because the default window frame for ORDER BY is `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. LAST_VALUE with this frame returns the last row seen so far — which is always the current row. Add `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` to get the true last row in the partition.

---

> [!success]
> Key takeaway: Window functions compute across rows without collapsing them — the OVER() clause defines which rows participate. ROW_NUMBER for deduplication, RANK/DENSE_RANK for rankings, LAG/LEAD for time comparisons, SUM OVER for running totals. Filter on window results using a CTE, not a WHERE clause.

---

[[05-ctes|Previous: CTEs]] | [[07-sql-case-studies|Next: SQL Case Studies]]
