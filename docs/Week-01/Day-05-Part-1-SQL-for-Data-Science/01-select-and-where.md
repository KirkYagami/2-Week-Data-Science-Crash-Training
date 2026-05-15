# SELECT and WHERE

SQL is the native language of data. Before you reach for pandas, someone already loaded that data from a database using a query. Understanding SELECT and WHERE is not optional — it is the foundation everything else is built on.

## Learning Objectives

- Understand the difference between how SQL is *written* and how it is *executed*
- Select specific columns and alias them for readability
- Filter rows precisely using WHERE with multiple conditions
- Handle NULL values correctly (and understand why `= NULL` silently fails)
- Use LIKE, IN, and BETWEEN to write clean, readable filters

---

## The Schema We Use Throughout This Course

Every example uses the same three tables. Memorise the columns — you will see them in every file.

```sql
-- orders(order_id, customer_id, product_id, amount, status, created_at)
-- customers(customer_id, name, country, email, created_at)
-- products(product_id, name, category, price)
```

---

## The Execution Order Problem

This is the single most important thing to understand before writing any SQL. SQL is *written* in this order:

```sql
SELECT   -- 4. which columns to return
FROM     -- 1. which table to read
WHERE    -- 2. which rows to keep
ORDER BY -- 5. how to sort the output
LIMIT    -- 6. how many rows to return
```

But the database *executes* it in a completely different order:

```
FROM → WHERE → SELECT → ORDER BY → LIMIT
```

Why does this matter? Because you cannot use a SELECT alias inside a WHERE clause — WHERE runs before SELECT. This catches everyone at least once.

```sql
-- This FAILS — aliases from SELECT are not visible in WHERE
SELECT amount * 0.9 AS discounted_price
FROM orders
WHERE discounted_price > 100;  -- ERROR: column "discounted_price" does not exist

-- This works — repeat the expression in WHERE
SELECT amount * 0.9 AS discounted_price
FROM orders
WHERE amount * 0.9 > 100;
```

> [!warning]
> The alias you define in SELECT is not available in WHERE, GROUP BY, or HAVING. It is only available in ORDER BY (which runs last). This is one of the most common beginner errors.

---

## Basic SELECT

Selecting all columns with `*` is convenient for exploration, but a bad habit in production. It pulls every column across the network and breaks if someone adds or reorders columns.

```sql
-- Explore: fine during development
SELECT *
FROM orders;

-- Production: be explicit
SELECT order_id, customer_id, amount, status
FROM orders;
```

### Column Aliases

Aliases rename columns in the output. Use them to make results readable.

```sql
SELECT
    order_id,
    amount,
    amount * 0.9          AS discounted_amount,  -- computed column
    created_at            AS order_date           -- rename for clarity
FROM orders;
```

> [!tip]
> Use snake_case for aliases and keep them descriptive. `rev` saves you three characters and costs whoever reads the query 10 seconds of confusion.

### Table Aliases

When joining tables you will need to prefix columns with the table name. Table aliases keep this readable.

```sql
-- Without alias: verbose
SELECT orders.order_id, orders.amount
FROM orders;

-- With alias: clean
SELECT o.order_id, o.amount
FROM orders o;
```

---

## DISTINCT

DISTINCT removes duplicate rows from the result. It applies to the entire row, not just one column.

```sql
-- How many unique countries do our customers come from?
SELECT DISTINCT country
FROM customers;

-- Unique combinations of status and country
SELECT DISTINCT status, country
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

> [!warning]
> `SELECT DISTINCT` scans the full result set to deduplicate. On large tables this is expensive. If you find yourself using DISTINCT often, ask whether your JOIN is creating duplicates upstream — that is usually the real problem.

---

## Filtering with WHERE

WHERE is where the work happens. Every condition evaluates to TRUE, FALSE, or NULL for each row. Only rows where the condition is TRUE are returned.

### Comparison Operators

```sql
-- Exact match
SELECT * FROM orders WHERE status = 'completed';

-- Not equal (both forms work)
SELECT * FROM orders WHERE status != 'cancelled';
SELECT * FROM orders WHERE status <> 'cancelled';

-- Numeric comparisons
SELECT * FROM orders WHERE amount > 500;
SELECT * FROM orders WHERE amount >= 100 AND amount <= 1000;
```

### Combining Conditions with AND / OR

```sql
-- Both conditions must be true
SELECT *
FROM orders
WHERE status = 'completed'
  AND amount > 1000;

-- Either condition can be true
SELECT *
FROM orders
WHERE country = 'India'
   OR country = 'USA';
```

> [!warning]
> AND binds tighter than OR. Without parentheses, `WHERE a = 1 OR b = 2 AND c = 3` is interpreted as `WHERE a = 1 OR (b = 2 AND c = 3)`. Always add parentheses when mixing AND and OR — your intent will be clear and you will not introduce a silent bug.

```sql
-- Ambiguous — do you mean this?
WHERE status = 'completed' OR status = 'pending' AND amount > 500

-- Clear — use parentheses
WHERE (status = 'completed' OR status = 'pending') AND amount > 500
```

---

## IN — Cleaner than Multiple ORs

```sql
-- This works but is ugly
SELECT *
FROM orders
WHERE status = 'completed'
   OR status = 'pending'
   OR status = 'processing';

-- This is cleaner
SELECT *
FROM orders
WHERE status IN ('completed', 'pending', 'processing');
```

IN also works with a subquery — you will see this pattern in the subqueries file.

```sql
-- Orders from customers in specific countries
SELECT *
FROM orders
WHERE customer_id IN (
    SELECT customer_id
    FROM customers
    WHERE country IN ('India', 'USA', 'Germany')
);
```

---

## BETWEEN — Inclusive Range Checks

BETWEEN includes both endpoints. `BETWEEN 100 AND 500` means `>= 100 AND <= 500`.

```sql
-- Orders with amount between 100 and 500 (inclusive)
SELECT *
FROM orders
WHERE amount BETWEEN 100 AND 500;

-- Date ranges
SELECT *
FROM orders
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';
```

> [!info]
> For date ranges, BETWEEN on a datetime column can be tricky. `BETWEEN '2024-01-01' AND '2024-12-31'` misses records from `2024-12-31 00:00:01` onwards. Prefer `>= '2024-01-01' AND < '2025-01-01'` for datetime columns.

---

## LIKE — Pattern Matching

LIKE matches string patterns. Two wildcards: `%` matches any sequence of characters, `_` matches exactly one character.

```sql
-- Customers whose name starts with 'A'
SELECT * FROM customers WHERE name LIKE 'A%';

-- Customers whose email is from gmail
SELECT * FROM customers WHERE email LIKE '%@gmail.com';

-- Products with exactly 5-character names
SELECT * FROM products WHERE name LIKE '_____';

-- Case-insensitive match (use ILIKE in PostgreSQL)
SELECT * FROM customers WHERE name ILIKE 'alice%';
```

> [!warning]
> A leading wildcard like `LIKE '%text'` forces a full table scan — the database cannot use an index. On a million-row table this can be catastrophic. If you need full-text search, use a proper text search index or engine.

---

## NULL Handling — The Most Important Gotcha in SQL

NULL means "unknown" — not zero, not empty string, not false. It is in a category of its own. Any comparison with NULL returns NULL (not TRUE, not FALSE). This means NULL silently fails filter conditions.

```sql
-- This returns nothing when amount is NULL — not an error, just no match
SELECT * FROM orders WHERE amount = NULL;   -- WRONG, always returns 0 rows

-- This is correct
SELECT * FROM orders WHERE amount IS NULL;

-- Find rows with a value
SELECT * FROM orders WHERE amount IS NOT NULL;
```

The practical danger: if you filter with `WHERE status != 'cancelled'`, rows where status IS NULL are also excluded. The database does not interpret NULL as "not cancelled" — it interprets it as "unknown".

```sql
-- This misses orders where status is NULL
SELECT * FROM orders WHERE status != 'cancelled';

-- This is what you probably want
SELECT * FROM orders WHERE status != 'cancelled' OR status IS NULL;
```

> [!warning]
> NULL in NOT IN is one of the most dangerous traps in SQL. If a subquery returns any NULL value, `NOT IN` returns no rows at all — silently. This will be covered in depth in the subqueries file. For now: always check whether your subquery can return NULLs before using NOT IN.

---

## ORDER BY and LIMIT

ORDER BY sorts the result. LIMIT caps the number of rows returned.

```sql
-- Top 10 orders by amount, most expensive first
SELECT order_id, customer_id, amount, status
FROM orders
ORDER BY amount DESC
LIMIT 10;

-- Sort by multiple columns: country first, then amount descending
SELECT o.order_id, c.country, o.amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
ORDER BY c.country ASC, o.amount DESC;
```

> [!tip]
> Always pair LIMIT with ORDER BY. `SELECT * FROM orders LIMIT 10` without ORDER BY returns 10 rows in undefined order — the database picks whichever rows it finds first. This is non-deterministic and will give you different results across runs.

---

## Putting It Together

A realistic query using everything from this file:

```sql
-- Find the 20 largest completed orders placed in 2024
-- from customers in India or Germany
SELECT
    o.order_id,
    c.name           AS customer_name,
    c.country,
    o.amount,
    o.status,
    o.created_at     AS order_date
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'completed'
  AND o.created_at >= '2024-01-01'
  AND o.created_at <  '2025-01-01'
  AND c.country IN ('India', 'Germany')
ORDER BY o.amount DESC
LIMIT 20;
```

---

## Practice Exercises

**Warm-up**

1. Select `order_id`, `amount`, and `status` from `orders`. Alias `amount` as `order_value`.
2. Find all orders where `status` is `'cancelled'`.
3. Find all customers from `'USA'`.

**Main**

4. Find all orders with an amount between 200 and 800, placed after 2024-06-01.
5. Find all products in the `'Electronics'` or `'Clothing'` category, sorted by price descending.
6. Find customers whose email ends with `'@company.com'`.
7. Find orders where `status` is NOT `'cancelled'` — make sure you handle NULLs correctly.

**Stretch**

8. Find the 5 most recent orders where the amount is above the number 500. Write this without a subquery.
9. A teammate wrote `WHERE amount = NULL` and got zero results. Explain exactly why, and fix the query.
10. Write a query that returns `order_id`, `amount`, and a column called `amount_bucket` that shows the literal string `'high'` if amount > 1000, `'medium'` if between 200 and 1000, and `'low'` otherwise — using only features covered in this file. (Hint: you cannot use CASE yet — think about whether this is actually possible with just WHERE and what that tells you about CASE's value.)

---

## Interview Questions

**Q: What is SQL execution order and why does it matter?**

The database executes `FROM` first, then `WHERE`, then `SELECT`, then `ORDER BY`, then `LIMIT`. This is why you cannot reference a SELECT alias in a WHERE clause — WHERE runs before SELECT has defined the alias.

**Q: How do you check for NULL values in SQL?**

Use `IS NULL` or `IS NOT NULL`. The `= NULL` comparison always evaluates to NULL (not TRUE), so it never matches any rows.

**Q: What is the difference between IN and multiple OR conditions?**

Functionally equivalent for fixed lists. IN is more readable and scales better. Multiple OR becomes unmanageable beyond three values. IN also pairs naturally with subqueries.

**Q: Why can LIKE with a leading wildcard hurt performance?**

A pattern like `LIKE '%text'` cannot use a B-tree index because the database cannot skip ahead — it must scan every row. A trailing wildcard like `LIKE 'text%'` can use an index because the database knows where in the index to start.

---

> [!success]
> Key takeaway: SQL execution order (FROM → WHERE → SELECT → ORDER BY) is counterintuitive but governs everything. NULL is not a value — it is the absence of a value — and treating it like one is the source of many silent bugs.

---

[[02-group-by-and-having|Next: GROUP BY and HAVING]]
