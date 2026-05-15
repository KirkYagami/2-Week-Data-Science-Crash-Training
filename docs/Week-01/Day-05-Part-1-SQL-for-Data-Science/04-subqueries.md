# Subqueries

A subquery lets you use the result of one query as the input to another. They make it possible to express multi-step logic in a single SQL statement. They are also easy to abuse — a poorly written correlated subquery can execute millions of times on a large table. Knowing *when* to use them versus when to reach for a JOIN or CTE is a skill that separates competent from expert SQL.

## Learning Objectives

- Write scalar, table, and correlated subqueries
- Understand where each type can appear: in SELECT, FROM, and WHERE
- Use EXISTS as a performant alternative to IN for existence checks
- Recognise the NULL in NOT IN trap — one of the most dangerous bugs in SQL
- Know when subqueries hurt performance and when a JOIN or CTE is better

---

## The Schema

```sql
-- orders(order_id, customer_id, product_id, amount, status, created_at)
-- customers(customer_id, name, country, email, created_at)
-- products(product_id, name, category, price)
```

---

## What Is a Subquery?

A subquery is a query inside another query, wrapped in parentheses. The outer query uses the result of the inner query.

Three types to know:

| Type | Returns | Used in |
|---|---|---|
| Scalar subquery | One row, one column | SELECT, WHERE, HAVING |
| Table subquery | Multiple rows and columns | FROM clause |
| Correlated subquery | Depends on outer query row | WHERE, HAVING |

---

## Scalar Subqueries — in SELECT and WHERE

A scalar subquery returns exactly one value. It can be used anywhere a single value is expected.

```sql
-- Find all orders above the average order amount
-- The subquery returns one number: the overall average
SELECT
    order_id,
    customer_id,
    amount
FROM orders
WHERE amount > (
    SELECT AVG(amount)
    FROM orders
);
```

The same approach in SELECT — attach a benchmark value to every row:

```sql
-- Show each order with the overall average alongside it
SELECT
    order_id,
    amount,
    (SELECT AVG(amount) FROM orders) AS overall_avg,
    amount - (SELECT AVG(amount) FROM orders) AS deviation_from_avg
FROM orders
ORDER BY deviation_from_avg DESC;
```

> [!warning]
> A scalar subquery must return exactly one row and one column. If your subquery returns multiple rows, the database throws an error. If it returns zero rows, the result is NULL. Both cases are easy to get wrong when the subquery logic has a bug.

---

## Subqueries in FROM — Derived Tables

A subquery in the FROM clause creates a temporary result set that you query against. The database calls this a derived table or inline view.

```sql
-- Find the average of per-customer totals
-- (different from the average of all orders)
SELECT AVG(customer_total) AS avg_customer_lifetime_value
FROM (
    SELECT
        customer_id,
        SUM(amount) AS customer_total
    FROM orders
    GROUP BY customer_id
) customer_summary;
```

You can also filter and join against a derived table:

```sql
-- Find customers whose total spending is above the median customer spend
-- Step 1 (inner): compute per-customer totals
-- Step 2 (outer): filter by the average of those totals
SELECT
    cs.customer_id,
    c.name,
    cs.customer_total
FROM (
    SELECT
        customer_id,
        SUM(amount) AS customer_total
    FROM orders
    GROUP BY customer_id
) cs
JOIN customers c ON cs.customer_id = c.customer_id
WHERE cs.customer_total > (
    SELECT AVG(customer_total)
    FROM (
        SELECT customer_id, SUM(amount) AS customer_total
        FROM orders
        GROUP BY customer_id
    ) inner_summary
)
ORDER BY cs.customer_total DESC;
```

> [!tip]
> When a FROM subquery is getting complex — especially when you need to reference the same derived table more than once — switch to a CTE. The query above is much cleaner written with WITH. See the CTEs file.

---

## Subqueries in WHERE with IN

A common use case: filter rows based on a list returned by another query.

```sql
-- Find customers who have placed at least one completed order
SELECT customer_id, name, country
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM orders
    WHERE status = 'completed'
);
```

```sql
-- Find products that have generated more than 10,000 in total revenue
SELECT product_id, name, category, price
FROM products
WHERE product_id IN (
    SELECT product_id
    FROM orders
    GROUP BY product_id
    HAVING SUM(amount) > 10000
);
```

---

## The NULL in NOT IN Trap

This is one of the most dangerous bugs in SQL. It causes the query to silently return no rows — without any error.

**Setup:** you want customers who have NOT placed any orders.

```sql
-- This LOOKS correct but is BROKEN if orders.customer_id contains any NULLs
SELECT customer_id, name
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM orders
);
```

**Why it fails:** SQL evaluates `NOT IN` by checking `customer_id != each value in the list`. If the list contains even one NULL, then every comparison against NULL returns NULL (not TRUE or FALSE). SQL requires all comparisons to be TRUE for NOT IN to pass. With a NULL in the list, the entire expression evaluates to NULL for every row — and NULL rows are excluded. You get zero results.

```sql
-- Reproduce the bug
SELECT 1 WHERE 5 NOT IN (1, 2, NULL);  -- returns 0 rows

-- Demonstrate the fix
SELECT customer_id, name
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM orders WHERE customer_id IS NOT NULL  -- exclude NULLs
);
```

The cleanest fix is to use NOT EXISTS or a LEFT JOIN anti-join pattern instead:

```sql
-- Safe: NOT EXISTS does not have the NULL problem
SELECT c.customer_id, c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- Also safe: anti-join pattern
SELECT c.customer_id, c.name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

> [!warning]
> Before using `NOT IN` with a subquery, always ask: can the subquery return NULL? If yes, add `WHERE column IS NOT NULL` to the subquery, or switch to `NOT EXISTS`. This is one of the most common SQL bugs in production code.

---

## Correlated Subqueries

A correlated subquery references a column from the outer query. The database executes the inner query once *for each row* of the outer query.

```sql
-- Find orders where the amount is above that customer's own average
-- The inner query uses 'o.customer_id' from the outer query
SELECT
    o.order_id,
    o.customer_id,
    o.amount
FROM orders o
WHERE o.amount > (
    SELECT AVG(o2.amount)
    FROM orders o2
    WHERE o2.customer_id = o.customer_id  -- correlated: ties to outer row
);
```

Another example — find each customer's most recent order:

```sql
-- Find the most recent order for each customer
SELECT
    o.order_id,
    o.customer_id,
    o.amount,
    o.created_at
FROM orders o
WHERE o.created_at = (
    SELECT MAX(o2.created_at)
    FROM orders o2
    WHERE o2.customer_id = o.customer_id  -- correlated
);
```

> [!warning]
> Correlated subqueries execute once per row of the outer query. On a table with 1 million rows, the subquery runs 1 million times. This is catastrophically slow for large datasets. Almost every correlated subquery can be rewritten as a window function or a JOIN — which the database can execute in a single pass. The window function equivalent of the query above: `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC)`.

---

## EXISTS vs IN — Performance and Semantics

EXISTS checks whether the subquery returns at least one row. It short-circuits as soon as it finds a match — it does not collect all the results.

```sql
-- Using IN: collects all customer_ids from orders, then checks membership
SELECT c.customer_id, c.name
FROM customers c
WHERE c.customer_id IN (
    SELECT customer_id FROM orders WHERE status = 'completed'
);

-- Using EXISTS: for each customer, checks if any completed order exists
-- Stops searching after finding the first match
SELECT c.customer_id, c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.status = 'completed'
);
```

**When to use EXISTS vs IN:**

- EXISTS is better when the subquery result set is large — it short-circuits.
- IN is simpler to read and performs well when the subquery result set is small.
- EXISTS does not have the NULL problem that NOT IN has.
- NOT EXISTS is almost always the right choice over NOT IN.

> [!tip]
> The `SELECT 1` inside EXISTS is a convention — the database does not actually retrieve those values, only whether a row exists. Some developers write `SELECT *` or `SELECT column_name` — all equivalent. `SELECT 1` signals clearly that you only care about existence.

---

## When to Use Subqueries vs Joins vs CTEs

| Situation | Better approach |
|---|---|
| One-time filter based on aggregate | Subquery in WHERE |
| Need columns from the other table | JOIN |
| Complex multi-step logic | CTE |
| Checking existence only | EXISTS |
| Reusing the same intermediate result | CTE |
| Query needs to be readable by others | CTE |
| Simple inline transformation | Subquery in FROM |

```sql
-- Subquery approach: find products in top revenue categories
SELECT product_id, name, category
FROM products
WHERE category IN (
    SELECT category
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY category
    HAVING SUM(o.amount) > 50000
);

-- JOIN approach: more explicit, easier to extend
SELECT DISTINCT p.product_id, p.name, p.category
FROM products p
JOIN orders o ON p.product_id = o.product_id
JOIN (
    SELECT category, SUM(o2.amount) AS category_revenue
    FROM orders o2
    JOIN products p2 ON o2.product_id = p2.product_id
    GROUP BY p2.category
    HAVING SUM(o2.amount) > 50000
) top_cats ON p.category = top_cats.category;
```

The second version is more complex here. This is a case for a CTE — see the next file.

---

## Practice Exercises

**Warm-up**

1. Find all orders with an amount above the overall average order amount.
2. Find all customers who have placed at least one order.
3. Find all customers who have never placed an order (use NOT EXISTS, not NOT IN).

**Main**

4. Find all products whose price is above the average price in their category. Use a correlated subquery.
5. Find the most recent order for each customer using a correlated subquery.
6. Find customers whose total spend is above the average total spend across all customers (two levels of aggregation).
7. Rewrite question 4 without a correlated subquery — use a JOIN to a derived table instead. Which version is clearer?

**Stretch**

8. Demonstrate the NULL in NOT IN bug: create a scenario (using the orders table) where NOT IN returns zero rows because of a NULL, and then fix it with NOT EXISTS.
9. A correlated subquery runs in 45 seconds on a 500k-row table. Rewrite it using a window function that runs in 2 seconds. (Use the "find orders above their own customer's average" example.)
10. Explain the difference in what EXISTS and COUNT return when used for existence checks. When would you use one over the other?

---

## Interview Questions

**Q: What is a correlated subquery and what is its performance risk?**

A correlated subquery references a column from the outer query, so the database executes it once per row of the outer query. On a table with N rows, the inner query runs N times. For large tables this is extremely slow. The fix is usually a window function or a JOIN.

**Q: Why is NOT IN dangerous when the subquery might return NULLs?**

SQL's three-valued logic means `value != NULL` evaluates to NULL, not TRUE or FALSE. NOT IN requires all comparisons to be non-NULL for the condition to pass. If the subquery returns any NULL, every row evaluates to NULL and the entire NOT IN clause matches nothing. Use NOT EXISTS or add `WHERE column IS NOT NULL` to the subquery.

**Q: What is the difference between IN and EXISTS?**

IN collects all values from the subquery into a list, then checks membership. EXISTS checks whether the subquery returns at least one row and short-circuits on the first match. EXISTS scales better for large subquery result sets and does not have the NULL problem. For NOT conditions, NOT EXISTS is almost always preferable to NOT IN.

---

> [!success]
> Key takeaway: Subqueries are powerful but can be slow — correlated subqueries run once per outer row. The NULL in NOT IN trap silently returns no results. NOT EXISTS is the safe default for anti-join patterns.

---

[[03-joins|Previous: JOINs]] | [[05-ctes|Next: CTEs]]
