# 🪟 06 — Window Functions
## Row-Level Analytics Without Collapsing Rows

> [!info] Goal
> Learn how window functions calculate rankings, running totals, and group metrics while keeping row-level detail.

---

## Why Window Functions Matter

`GROUP BY` collapses rows. Window functions keep rows.

```sql
SELECT
  order_id,
  city,
  revenue,
  AVG(revenue) OVER (PARTITION BY city) AS city_avg_revenue
FROM orders;
```

This keeps every order and adds city average revenue.

---

## Basic Syntax

```sql
function() OVER (
  PARTITION BY group_column
  ORDER BY sort_column
)
```

---

## Ranking

```sql
SELECT
  order_id,
  city,
  revenue,
  RANK() OVER (ORDER BY revenue DESC) AS revenue_rank
FROM orders;
```

Within each city:

```sql
SELECT
  order_id,
  city,
  revenue,
  RANK() OVER (PARTITION BY city ORDER BY revenue DESC) AS city_rank
FROM orders;
```

---

## ROW_NUMBER, RANK, DENSE_RANK

| Function | Behavior |
|----------|----------|
| `ROW_NUMBER()` | unique sequence, no ties |
| `RANK()` | ties share rank, skips next rank |
| `DENSE_RANK()` | ties share rank, no gaps |

---

## Running Total

```sql
SELECT
  order_date,
  revenue,
  SUM(revenue) OVER (
    ORDER BY order_date
  ) AS running_revenue
FROM orders;
```

---

## Moving Average

```sql
SELECT
  order_date,
  revenue,
  AVG(revenue) OVER (
    ORDER BY order_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ) AS moving_avg_3
FROM orders;
```

---

## LAG and LEAD

```sql
SELECT
  order_date,
  revenue,
  LAG(revenue) OVER (ORDER BY order_date) AS previous_revenue,
  revenue - LAG(revenue) OVER (ORDER BY order_date) AS revenue_change
FROM orders;
```

Use `LEAD()` to access the next row.

---

## Practice

Write queries to:

- rank orders by revenue
- rank orders within each city
- calculate running revenue by date
- calculate city average revenue per order
- calculate previous order revenue using `LAG`

---

## Interview Questions

**Q1:** Difference between GROUP BY and window function?

> GROUP BY collapses rows; window functions keep row-level detail.

**Q2:** What does `PARTITION BY` do?

> It defines groups inside the window calculation.

**Q3:** What is `LAG()` used for?

> Accessing a previous row's value.

---

## 🔗 What's Next?

➡️ [[07-sql-case-studies]]
