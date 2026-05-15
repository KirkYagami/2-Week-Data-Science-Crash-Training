# JOINs

Real data lives in multiple tables. A JOIN is how you connect them. It is also how you can accidentally turn 10,000 rows into 10 million rows in under a second. Understanding JOINs deeply — not just the syntax, but the mechanics of which rows survive and which disappear — is what separates analysts who can be trusted with production data from those who cannot.

## Learning Objectives

- Understand the mechanics of INNER, LEFT, RIGHT, and FULL OUTER JOINs
- Know which rows survive each join type, and build that intuition visually
- Recognise and prevent the two most dangerous JOIN mistakes: row explosion and silent data loss
- Write multi-table JOINs correctly using table aliases
- Use self-joins and understand when cross joins are appropriate (rarely)
- Debug unexpected row counts after a JOIN

---

## The Schema

```sql
-- orders(order_id, customer_id, product_id, amount, status, created_at)
-- customers(customer_id, name, country, email, created_at)
-- products(product_id, name, category, price)
```

---

## Why Tables Are Separate

Databases split data into multiple tables to avoid redundancy. Instead of storing the customer's name and country in every order row, you store a `customer_id` and look up the customer details when needed. A JOIN re-assembles this split data at query time.

The column used to connect two tables is the **join key**. In our schema, `orders.customer_id` is a foreign key that references `customers.customer_id`.

---

## INNER JOIN — Only Matching Rows

An INNER JOIN returns rows where the join key exists in *both* tables. If an order has a `customer_id` that does not exist in the `customers` table, that order disappears from the result.

```sql
-- Enrich orders with customer name and country
SELECT
    o.order_id,
    o.amount,
    o.status,
    c.name     AS customer_name,
    c.country
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id;
```

**Rows that survive:** only orders where `o.customer_id` has a match in `customers`.

**Mental model:** think of two overlapping circles (a Venn diagram). INNER JOIN returns only the intersection.

> [!warning]
> INNER JOIN silently drops rows with no match. If 5% of your orders have an orphaned `customer_id` (customer was deleted, or a data pipeline bug), those orders disappear without any error. Always check your row count before and after a JOIN. If you expected 50,000 rows and got 47,500, your join key has gaps.

---

## LEFT JOIN — Keep All Left Table Rows

A LEFT JOIN returns every row from the left table, and fills in the right table's columns with NULL where there is no match.

```sql
-- Keep all orders; bring in customer details where available
SELECT
    o.order_id,
    o.amount,
    o.status,
    c.name     AS customer_name,  -- NULL if no matching customer
    c.country                     -- NULL if no matching customer
FROM orders o
LEFT JOIN customers c
    ON o.customer_id = c.customer_id;
```

**Rows that survive:** all rows from `orders`. Rows with no customer match get NULLs in the customer columns.

**When to use LEFT JOIN vs INNER JOIN:** If you want to guarantee that every row from your primary table appears in the result, use LEFT JOIN. Use INNER JOIN only when you are certain both sides are complete.

> [!tip]
> In analytics, LEFT JOIN is almost always the safer choice. Start with LEFT JOIN and switch to INNER JOIN only if you have verified key completeness and want to explicitly exclude unmatched rows.

### Finding Unmatched Rows with LEFT JOIN

The classic anti-join pattern: find rows in the left table that have no match in the right table.

```sql
-- Find customers who have never placed an order
SELECT
    c.customer_id,
    c.name,
    c.email
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;  -- NULL means no match was found
```

**Why this works:** LEFT JOIN gives every customer a row. For customers with no orders, all the `orders` columns are NULL. Filtering on `o.order_id IS NULL` keeps only those customers.

---

## RIGHT JOIN — Keep All Right Table Rows

RIGHT JOIN is the mirror of LEFT JOIN: keep all rows from the right table, fill NULLs for unmatched left table rows. In practice, most analysts just swap the table order and write a LEFT JOIN instead — it is easier to reason about.

```sql
-- Same as the anti-join above, written as RIGHT JOIN
-- (swapping tables makes this a LEFT JOIN)
SELECT
    c.customer_id,
    c.name
FROM orders o
RIGHT JOIN customers c
    ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL;
```

> [!info]
> RIGHT JOIN is rarely used in practice. When you see one, it usually means the developer wrote the tables in the wrong order and then added RIGHT to compensate. Rewrite it as a LEFT JOIN with the tables swapped — the intent will be much clearer.

---

## FULL OUTER JOIN — Keep Everything

FULL OUTER JOIN returns all rows from both tables, with NULLs where there is no match on either side.

```sql
-- Show all customers and all orders, including those with no match on either side
SELECT
    c.customer_id,
    c.name,
    o.order_id,
    o.amount
FROM customers c
FULL OUTER JOIN orders o
    ON c.customer_id = o.customer_id;
```

**Use cases:** data reconciliation (finding mismatches between two datasets), or building a complete picture when either side may have gaps.

> [!info]
> FULL OUTER JOIN is not supported in all databases. MySQL does not support it natively — simulate it with a LEFT JOIN UNION ALL RIGHT JOIN WHERE left IS NULL.

---

## Joining Three Tables

Each JOIN adds one table to the result. Chain them by adding JOIN clauses.

```sql
-- Full order details: combine orders, customers, and products
SELECT
    o.order_id,
    o.created_at,
    o.amount,
    o.status,
    c.name       AS customer_name,
    c.country,
    p.name       AS product_name,
    p.category,
    p.price      AS list_price
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
LEFT JOIN products p  ON o.product_id  = p.product_id
ORDER BY o.created_at DESC;
```

> [!tip]
> When joining multiple tables, always use table aliases (`o`, `c`, `p`) and prefix every column with its alias. When a query returns confusing results, removing one JOIN at a time helps you isolate which join is causing the problem.

---

## The Row Explosion Problem

This is the most dangerous JOIN mistake. It happens silently and the results look plausible.

**Scenario:** the `orders` table has 10,000 rows. A data bug means the `products` table has two rows with the same `product_id`. When you join on `product_id`, every order row matches two product rows. Your result: 20,000 rows. Nothing errors. No warning.

```sql
-- Check for duplicate keys BEFORE joining
-- If this returns any rows, your join will create duplicates
SELECT product_id, COUNT(*) AS occurrences
FROM products
GROUP BY product_id
HAVING COUNT(*) > 1;
```

```sql
-- Also check the row count before and after
SELECT COUNT(*) FROM orders;                    -- before join

SELECT COUNT(*) FROM orders o
JOIN products p ON o.product_id = p.product_id; -- after join — should match
```

> [!warning]
> Always test your JOINs on a small dataset first — a bad JOIN can return millions of rows in seconds, and crash downstream code or pipelines. On large tables, verify key uniqueness before you run the full query.

---

## JOIN on Multiple Conditions

You can join on multiple conditions using AND in the ON clause.

```sql
-- Join on two conditions: customer_id AND matching status
-- (contrived example, but the syntax is important)
SELECT o.order_id, o.amount, c.name
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
    AND o.status = 'completed';  -- additional filter in the ON clause
```

> [!info]
> Putting a filter in the ON clause of a LEFT JOIN behaves differently from putting it in the WHERE clause. In ON, it filters which right-table rows participate in the match, but still keeps all left-table rows (with NULLs). In WHERE, it eliminates left-table rows that did not match — effectively converting a LEFT JOIN into an INNER JOIN.

```sql
-- These two queries return DIFFERENT results

-- Filter in ON: keeps all customers, shows NULL for non-completed orders
SELECT c.name, o.order_id, o.amount
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
    AND o.status = 'completed';

-- Filter in WHERE: only customers who have completed orders
SELECT c.name, o.order_id, o.amount
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.status = 'completed';
```

---

## Self-Join

A self-join joins a table to itself. Useful when rows in the same table have a relationship to each other.

```sql
-- Find pairs of customers from the same country
-- (each pair listed once, alphabetically)
SELECT
    a.name    AS customer_a,
    b.name    AS customer_b,
    a.country
FROM customers a
JOIN customers b
    ON a.country = b.country
    AND a.customer_id < b.customer_id   -- prevents duplicates and self-pairing
ORDER BY a.country, a.name;
```

---

## CROSS JOIN — The Dangerous One

A CROSS JOIN produces every combination of rows from both tables (the Cartesian product). 100 rows × 100 rows = 10,000 rows. 1,000 rows × 1,000 rows = 1 million rows.

```sql
-- Generate all combinations of customers and products
-- Result: (number of customers) × (number of products) rows
SELECT c.name, p.name AS product
FROM customers c
CROSS JOIN products p;
```

**When CROSS JOIN is legitimate:** when you genuinely need every combination — for example, generating a complete grid of dates × regions for a dashboard that needs to show zero values for empty periods.

> [!warning]
> An accidental CROSS JOIN — caused by forgetting the ON clause — is one of the easiest ways to take down a database. `SELECT * FROM orders, customers` (old-style join syntax) is a CROSS JOIN if no WHERE condition links the tables. Modern JOIN syntax makes this error impossible because ON is mandatory.

---

## ON vs USING

When the join column has the same name in both tables, USING is a cleaner shorthand.

```sql
-- These are equivalent
SELECT o.order_id, c.name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- USING syntax (only works if column name is identical in both tables)
SELECT order_id, name
FROM orders
JOIN customers USING (customer_id);
```

---

## Debugging a JOIN — Step by Step

When a JOIN gives unexpected results, work through this checklist:

```sql
-- 1. Count rows in each source table
SELECT COUNT(*) FROM orders;     -- e.g., 50,000
SELECT COUNT(*) FROM customers;  -- e.g., 12,000

-- 2. Check for duplicate join keys
SELECT customer_id, COUNT(*)
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;             -- should return 0 rows

-- 3. Run the join and count
SELECT COUNT(*)
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
-- If > 50,000: duplicates in customers
-- If < 50,000: some orders have no matching customer

-- 4. Find the unmatched orders
SELECT COUNT(*)
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

---

## Practice Exercises

**Warm-up**

1. Join `orders` to `customers` and return `order_id`, `amount`, customer `name`, and `country`.
2. Join `orders` to `products` and return `order_id`, `amount`, product `name`, and `category`.
3. Find all customers who have never placed an order.

**Main**

4. Find the total revenue per product category. Join `orders` to `products`.
5. Find the top 10 customers by total amount spent. Include their `name` and `country`.
6. Join all three tables and return a complete order detail view. Show only completed orders.
7. Find products that have never been ordered.

**Stretch**

8. Before running a join between `orders` and `customers`, write the query that checks whether `customer_id` is unique in `customers`. Explain what you would do if duplicates were found.
9. Explain in plain English why putting a filter condition in the ON clause of a LEFT JOIN gives a different result than putting it in the WHERE clause. Write a query demonstrating both.
10. Using a self-join on `customers`, find all pairs of customers from the same country. Avoid listing the same pair twice and avoid pairing a customer with themselves.

---

## Interview Questions

**Q: What is the difference between INNER JOIN and LEFT JOIN?**

INNER JOIN returns only rows with a match in both tables. Rows with no match on either side are dropped. LEFT JOIN returns all rows from the left table; right-table columns are NULL where there is no match. In analytics, LEFT JOIN is usually safer because it does not silently drop data.

**Q: Why might a JOIN return more rows than the left table?**

Duplicate join keys in the right table. If customer_id 42 appears three times in the customers table and customer 42 has placed 100 orders, the result will have 300 rows for that customer (100 × 3). Always verify key uniqueness before joining.

**Q: How do you find rows that exist in one table but not another?**

Use a LEFT JOIN from the first table to the second, then filter `WHERE second_table.key IS NULL`. This is called an anti-join. Alternatively, use `NOT EXISTS` with a correlated subquery.

**Q: What is a CROSS JOIN and when is it intentional?**

A CROSS JOIN produces every combination of rows from both tables. The result set size is the product of both table sizes. It is intentional when you need a Cartesian product — for example, combining every date with every product to create a complete reporting grid. It is dangerous when accidental — always use explicit JOIN syntax with an ON clause to prevent it.

---

> [!success]
> Key takeaway: INNER JOIN drops unmatched rows silently — verify row counts before and after. LEFT JOIN keeps all left-table rows and NULLs the rest. Duplicate join keys cause row explosion — check uniqueness first.

---

[[02-group-by-and-having|Previous: GROUP BY and HAVING]] | [[04-subqueries|Next: Subqueries]]
