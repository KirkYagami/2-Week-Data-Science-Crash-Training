# 🪆 04 — Subqueries
## Queries Inside Queries

> [!info] Goal
> Learn how to use subqueries for filtering, comparison, and intermediate results.

---

## What is a Subquery?

A subquery is a query nested inside another SQL query.

```sql
SELECT *
FROM orders
WHERE revenue > (
  SELECT AVG(revenue)
  FROM orders
);
```

This finds orders above average revenue.

---

## Subquery in WHERE

```sql
SELECT *
FROM customers
WHERE customer_id IN (
  SELECT customer_id
  FROM orders
  WHERE revenue > 50000
);
```

Finds customers who placed high-revenue orders.

---

## Subquery in FROM

```sql
SELECT
  city,
  AVG(total_revenue) AS avg_customer_revenue
FROM (
  SELECT
    customer_id,
    city,
    SUM(revenue) AS total_revenue
  FROM orders
  GROUP BY customer_id, city
) customer_summary
GROUP BY city;
```

Subqueries in `FROM` create temporary result tables.

---

## Subquery in SELECT

```sql
SELECT
  order_id,
  revenue,
  (SELECT AVG(revenue) FROM orders) AS avg_revenue
FROM orders;
```

This attaches overall average revenue to each row.

---

## Correlated Subquery

A correlated subquery refers to the outer query.

```sql
SELECT *
FROM orders o
WHERE revenue > (
  SELECT AVG(revenue)
  FROM orders
  WHERE city = o.city
);
```

This finds orders above their own city average.

---

## Subquery vs Join

| Use Subquery When | Use Join When |
|-------------------|---------------|
| filtering by an aggregate | combining columns from tables |
| checking existence | enriching rows with lookup data |
| logic is small and readable | relationships are central |

---

## Practice

Write subqueries to:

- find orders above average revenue
- find customers with at least one order
- find cities with above-average total revenue
- find orders above their city average

---

## Interview Questions

**Q1:** What is a subquery?

> A query nested inside another query.

**Q2:** What is a correlated subquery?

> A subquery that depends on values from the outer query.

**Q3:** Are subqueries always better than joins?

> No. Use whichever is clearer and efficient for the problem.

---

## 🔗 What's Next?

➡️ [[05-ctes]]
