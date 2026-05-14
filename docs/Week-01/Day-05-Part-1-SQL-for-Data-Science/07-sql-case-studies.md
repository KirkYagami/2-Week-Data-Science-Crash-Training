# 🧪 07 — SQL Case Studies
## Practice Business Questions

> [!info] Goal
> Apply SQL concepts to realistic Data Science questions.

---

## Case Study 1 — Revenue Dashboard

Questions:

- total revenue
- revenue by city
- revenue by category
- top 5 orders
- monthly revenue trend

```sql
SELECT SUM(revenue) AS total_revenue
FROM orders;
```

```sql
SELECT city, SUM(revenue) AS total_revenue
FROM orders
GROUP BY city
ORDER BY total_revenue DESC;
```

```sql
SELECT category, SUM(revenue) AS total_revenue
FROM orders
GROUP BY category
ORDER BY total_revenue DESC;
```

---

## Case Study 2 — High-Value Customers

```sql
WITH customer_revenue AS (
  SELECT
    customer_id,
    SUM(revenue) AS total_revenue,
    COUNT(*) AS order_count
  FROM orders
  GROUP BY customer_id
)
SELECT *
FROM customer_revenue
WHERE total_revenue > 100000
ORDER BY total_revenue DESC;
```

---

## Case Study 3 — Customer Details with Revenue

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
  ON cr.customer_id = c.customer_id
ORDER BY cr.total_revenue DESC;
```

---

## Case Study 4 — Top Product per Category

```sql
WITH product_revenue AS (
  SELECT
    category,
    product_id,
    SUM(revenue) AS total_revenue
  FROM orders
  GROUP BY category, product_id
),
ranked AS (
  SELECT
    *,
    RANK() OVER (
      PARTITION BY category
      ORDER BY total_revenue DESC
    ) AS rnk
  FROM product_revenue
)
SELECT *
FROM ranked
WHERE rnk = 1;
```

---

## Case Study 5 — Month-over-Month Revenue

```sql
WITH monthly AS (
  SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(revenue) AS monthly_revenue
  FROM orders
  GROUP BY DATE_TRUNC('month', order_date)
)
SELECT
  month,
  monthly_revenue,
  monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY month) AS revenue_change
FROM monthly
ORDER BY month;
```

Note: date functions vary by database.

---

## Practice

Answer:

- Which city has the highest revenue?
- Which customers ordered more than 3 times?
- Which category has highest average order value?
- Which product ranks first in each category?
- What is cumulative revenue over time?

---

## ✅ Key Takeaways

- Translate business questions into SQL steps.
- Use CTEs for multi-step analysis.
- Use joins to add details.
- Use window functions for rankings and trends.
- Always sort results for readability.

---

## 🔗 What's Next?

➡️ [[08-sql-interview-questions]]
