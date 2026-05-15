# CTEs — Common Table Expressions

When a SQL query grows beyond three or four steps, nested subqueries become unreadable. You end up with six layers of parentheses and no way to test intermediate results. CTEs solve this by letting you name each step and build on it — the same way you would break a complex Python function into smaller functions. Every senior analyst defaults to CTEs. There is a reason.

## Learning Objectives

- Write a CTE using the WITH clause and understand its scope
- Chain multiple CTEs to build multi-step analysis
- Understand when CTEs outperform subqueries on readability and maintainability
- Know the performance trade-offs: when CTEs help and when they do not
- Write a basic recursive CTE to generate sequences or traverse hierarchies

---

## The Schema

```sql
-- orders(order_id, customer_id, product_id, amount, status, created_at)
-- customers(customer_id, name, country, email, created_at)
-- products(product_id, name, category, price)
```

---

## The Problem CTEs Solve

Consider this query without a CTE: find customers whose total spend is above the average customer spend.

```sql
-- Without CTE: nested subqueries, hard to follow
SELECT c.name, c.country, cs.total_spent
FROM customers c
JOIN (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
) cs ON c.customer_id = cs.customer_id
WHERE cs.total_spent > (
    SELECT AVG(total_spent)
    FROM (
        SELECT customer_id, SUM(amount) AS total_spent
        FROM orders
        GROUP BY customer_id
    ) inner_agg
)
ORDER BY cs.total_spent DESC;
```

Now with CTEs:

```sql
-- With CTEs: same logic, reads like a story
WITH customer_spend AS (
    -- Step 1: total spend per customer
    SELECT
        customer_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
),
avg_spend AS (
    -- Step 2: the average of those totals
    SELECT AVG(total_spent) AS avg_customer_spend
    FROM customer_spend
)
-- Step 3: filter and enrich
SELECT
    c.name,
    c.country,
    cs.total_spent
FROM customer_spend cs
JOIN customers c ON cs.customer_id = c.customer_id
CROSS JOIN avg_spend a
WHERE cs.total_spent > a.avg_customer_spend
ORDER BY cs.total_spent DESC;
```

The logic is identical. The CTE version takes five extra lines but can be read top-to-bottom without mentally unwinding nested parentheses.

---

## Basic CTE Syntax

```sql
WITH cte_name AS (
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT *
FROM cte_name;
```

The CTE lives only within the query that defines it. Once the query finishes, the CTE is gone. It is not a table, not a view, not a temp table — it is a named intermediate result.

```sql
-- Which countries had more than 100 orders?
WITH order_counts AS (
    SELECT
        c.country,
        COUNT(*) AS total_orders
    FROM orders o
    JOIN customers c ON o.customer_id = c.customer_id
    GROUP BY c.country
)
SELECT country, total_orders
FROM order_counts
WHERE total_orders > 100
ORDER BY total_orders DESC;
```

---

## Chaining Multiple CTEs

Separate multiple CTEs with a comma. Each CTE can reference the ones defined before it.

```sql
WITH
-- Step 1: revenue and order count per customer
customer_metrics AS (
    SELECT
        customer_id,
        COUNT(*)    AS order_count,
        SUM(amount) AS total_revenue
    FROM orders
    WHERE status = 'completed'
    GROUP BY customer_id
),

-- Step 2: label customers by tier
customer_tiers AS (
    SELECT
        customer_id,
        order_count,
        total_revenue,
        CASE
            WHEN total_revenue >= 10000 THEN 'platinum'
            WHEN total_revenue >= 5000  THEN 'gold'
            WHEN total_revenue >= 1000  THEN 'silver'
            ELSE                             'bronze'
        END AS tier
    FROM customer_metrics
),

-- Step 3: count how many customers are in each tier
tier_summary AS (
    SELECT
        tier,
        COUNT(*)        AS customer_count,
        SUM(total_revenue) AS tier_revenue
    FROM customer_tiers
    GROUP BY tier
)

-- Final output
SELECT
    tier,
    customer_count,
    tier_revenue,
    ROUND(tier_revenue * 100.0 / SUM(tier_revenue) OVER (), 2) AS pct_of_revenue
FROM tier_summary
ORDER BY tier_revenue DESC;
```

> [!tip]
> Name your CTEs after what they contain, not what they do. `customer_metrics` is a better name than `step1` or `agg_query`. The name should describe the shape of data, so the main query reads like: "from customer metrics, label by tier, then summarise by tier."

---

## CTE with Joins

CTEs combine naturally with JOINs for enrichment steps.

```sql
-- Find the top 10 customers by revenue in the last 90 days
-- with their country and most recent order date
WITH recent_orders AS (
    -- Filter first — reduce the data early
    SELECT
        customer_id,
        SUM(amount)  AS revenue_90d,
        COUNT(*)     AS orders_90d,
        MAX(created_at) AS last_order_date
    FROM orders
    WHERE created_at >= CURRENT_DATE - INTERVAL '90 days'
      AND status = 'completed'
    GROUP BY customer_id
)
SELECT
    c.name,
    c.country,
    c.email,
    r.revenue_90d,
    r.orders_90d,
    r.last_order_date
FROM recent_orders r
JOIN customers c ON r.customer_id = c.customer_id
ORDER BY r.revenue_90d DESC
LIMIT 10;
```

---

## CTEs vs Subqueries — When to Use Each

| Situation | Prefer |
|---|---|
| Logic used once, short, simple | Subquery |
| Logic used more than once in the same query | CTE |
| Multi-step analysis (3+ steps) | CTE |
| Need to debug intermediate results | CTE |
| Someone else needs to read this query | CTE |
| Performance-sensitive and optimizer matters | Test both |

> [!info]
> In most databases (PostgreSQL, SQL Server, BigQuery), the query optimizer can inline a CTE and optimise it as if it were a subquery. In PostgreSQL specifically, a CTE was historically treated as an optimisation fence — it materialised the result and prevented certain optimisations. This changed in PostgreSQL 12. In older systems, a complex CTE may be slower than an equivalent subquery.

```sql
-- The subquery version (fine for simple cases)
SELECT *
FROM (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
) summary
WHERE total_spent > 5000;

-- The CTE version (preferred for readability and when logic gets complex)
WITH summary AS (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM summary
WHERE total_spent > 5000;
```

---

## Recursive CTEs

A recursive CTE references itself. It is used for hierarchical data (org charts, category trees) or generating sequences.

The structure is always the same: a non-recursive base case UNION ALL a recursive case that references the CTE name.

### Generating a Date Series

```sql
-- Generate a series of dates for the last 30 days
-- (useful for filling in zero-revenue days in a time series)
WITH RECURSIVE date_series AS (
    -- Base case: start date
    SELECT CURRENT_DATE - INTERVAL '29 days' AS dt

    UNION ALL

    -- Recursive case: add one day at a time
    SELECT dt + INTERVAL '1 day'
    FROM date_series
    WHERE dt < CURRENT_DATE
)
SELECT dt FROM date_series ORDER BY dt;
```

### Traversing a Hierarchy

```sql
-- Imagine a categories table with parent_id referencing itself:
-- categories(category_id, name, parent_id)

-- Find all subcategories under 'Electronics' (any depth)
WITH RECURSIVE category_tree AS (
    -- Base case: start at the root category
    SELECT category_id, name, parent_id, 0 AS depth
    FROM categories
    WHERE name = 'Electronics'

    UNION ALL

    -- Recursive case: find children of current nodes
    SELECT c.category_id, c.name, c.parent_id, ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT category_id, name, depth
FROM category_tree
ORDER BY depth, name;
```

> [!warning]
> Recursive CTEs require a termination condition — the WHERE clause in the recursive case. Without it, the query runs indefinitely. Most databases have a maximum recursion depth (e.g., 100 in SQL Server) that will throw an error before it runs forever, but always include a clear termination condition.

---

## Debugging with CTEs

One underrated benefit of CTEs: you can test each step individually by temporarily replacing the final SELECT.

```sql
WITH
customer_spend AS (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
),
high_value AS (
    SELECT * FROM customer_spend WHERE total_spent > 5000
)

-- Debug: temporarily replace the final SELECT to inspect an intermediate CTE
-- SELECT * FROM customer_spend LIMIT 20;   -- uncomment to inspect step 1
-- SELECT * FROM high_value LIMIT 20;       -- uncomment to inspect step 2

-- Production query:
SELECT c.name, c.country, hv.total_spent
FROM high_value hv
JOIN customers c ON hv.customer_id = c.customer_id
ORDER BY hv.total_spent DESC;
```

> [!tip]
> When debugging a multi-CTE query, comment out the final SELECT and point it at an intermediate CTE to verify the data at that step. This is much faster than trying to infer what went wrong from the final output.

---

## Practice Exercises

**Warm-up**

1. Write a CTE that calculates total revenue and order count per customer. Then query it to find customers with more than 3 orders.
2. Write a CTE that finds the top product in each category by revenue (most expensive product by total orders). Use a second CTE to filter only rank 1.
3. Rewrite the following subquery as a CTE: `SELECT * FROM (SELECT customer_id, AVG(amount) AS avg_amt FROM orders GROUP BY customer_id) t WHERE avg_amt > 200`.

**Main**

4. Write a two-CTE query: first calculate monthly revenue, then calculate the month-over-month change using LAG.
5. Build a customer tier classification using a CTE (bronze/silver/gold/platinum based on total spend). Then in a second CTE, count customers and total revenue per tier.
6. Find the top 5 countries by revenue in the last 30 days. Use a CTE to filter recent orders first, then join to customers and aggregate.

**Stretch**

7. Using a recursive CTE, generate a sequence of integers from 1 to 50.
8. A query has four nested subqueries. Rewrite it using CTEs and explain why the CTE version is easier to maintain.
9. Explain when a CTE might actually be *slower* than a subquery and what you would do about it.

---

## Interview Questions

**Q: What is a CTE and how is it different from a subquery?**

A CTE is a named temporary result set defined with the WITH clause, scoped to the current query. A subquery is an inline query nested inside another. Both produce the same result, but CTEs are reusable within the same query, easier to read, and easier to debug. A subquery that appears more than once should become a CTE.

**Q: Can a CTE reference another CTE?**

Yes. In a WITH block with multiple CTEs, each CTE can reference the CTEs defined before it. This is what enables the "steps" pattern — each CTE builds on the previous one.

**Q: What is a recursive CTE and when would you use it?**

A recursive CTE calls itself in a UNION ALL pattern. It has a base case (starting rows) and a recursive case (each iteration adds more rows). Use it for hierarchical data (org charts, category trees) or generating sequences (date ranges, integer series).

**Q: Are CTEs always more performant than subqueries?**

No. In most modern databases the query optimizer treats them equivalently and may inline the CTE. In PostgreSQL before version 12, CTEs were materialised (computed once and stored), which could hurt or help performance depending on whether the optimizer would have made a better plan without that constraint. Always test on your specific database and data volume if performance is critical.

---

> [!success]
> Key takeaway: Use CTEs when your query has more than two steps, when the same intermediate result is used more than once, or when someone else will need to read and maintain the query. Name CTEs after what they contain, not what they do.

---

[[04-subqueries|Previous: Subqueries]] | [[06-window-functions|Next: Window Functions]]
