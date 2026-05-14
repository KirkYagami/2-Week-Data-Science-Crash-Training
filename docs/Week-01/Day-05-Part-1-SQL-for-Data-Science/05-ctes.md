# 🧱 05 — CTEs
## Common Table Expressions

> [!info] Goal
> Learn how CTEs make complex SQL easier to read and maintain.

---

## What is a CTE?

A CTE is a named temporary result used inside a query.

```sql
WITH city_revenue AS (
  SELECT
    city,
    SUM(revenue) AS total_revenue
  FROM orders
  GROUP BY city
)
SELECT *
FROM city_revenue
WHERE total_revenue > 100000;
```

---

## Why Use CTEs?

CTEs help:

- break complex logic into steps
- avoid repeated subqueries
- improve readability
- debug intermediate outputs

---

## Multiple CTEs

```sql
WITH customer_revenue AS (
  SELECT
    customer_id,
    SUM(revenue) AS total_revenue
  FROM orders
  GROUP BY customer_id
),
high_value_customers AS (
  SELECT *
  FROM customer_revenue
  WHERE total_revenue > 100000
)
SELECT *
FROM high_value_customers;
```

---

## CTE with Joins

```sql
WITH customer_revenue AS (
  SELECT
    customer_id,
    SUM(revenue) AS total_revenue
  FROM orders
  GROUP BY customer_id
)
SELECT
  c.customer_name,
  c.city,
  cr.total_revenue
FROM customer_revenue cr
JOIN customers c
  ON cr.customer_id = c.customer_id;
```

---

## CTE vs Subquery

| CTE | Subquery |
|-----|----------|
| Easier to read for multi-step logic | Good for short nested logic |
| Can be reused in same query | Usually inline |
| Better for debugging | Compact |

---

## Practice

Write CTEs to:

- calculate revenue by customer
- filter high-value customers
- join high-value customers to customer details
- calculate monthly revenue then filter top months

---

## Interview Questions

**Q1:** What does `WITH` do?

> It defines a common table expression.

**Q2:** Why use CTEs?

> To make complex queries readable and modular.

**Q3:** Are CTEs permanent tables?

> No. They exist only for the query.

---

## 🔗 What's Next?

➡️ [[06-window-functions]]
