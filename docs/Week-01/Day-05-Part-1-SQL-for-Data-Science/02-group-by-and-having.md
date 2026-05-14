# 📊 02 — GROUP BY and HAVING
## Aggregating Data

> [!info] Goal
> Learn how to summarize rows into group-level metrics.

---

## Basic Aggregations

```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

```sql
SELECT
  SUM(revenue) AS total_revenue,
  AVG(revenue) AS avg_revenue,
  MIN(revenue) AS min_revenue,
  MAX(revenue) AS max_revenue
FROM orders;
```

---

## GROUP BY

Revenue by city:

```sql
SELECT
  city,
  SUM(revenue) AS total_revenue
FROM orders
GROUP BY city;
```

Orders by category:

```sql
SELECT
  category,
  COUNT(*) AS order_count
FROM orders
GROUP BY category;
```

---

## Multiple Group Columns

```sql
SELECT
  city,
  category,
  SUM(revenue) AS total_revenue
FROM orders
GROUP BY city, category;
```

---

## HAVING

`HAVING` filters aggregated groups.

```sql
SELECT
  city,
  SUM(revenue) AS total_revenue
FROM orders
GROUP BY city
HAVING SUM(revenue) > 100000;
```

---

## WHERE vs HAVING

```sql
SELECT
  city,
  SUM(revenue) AS total_revenue
FROM orders
WHERE order_date >= '2025-01-01'
GROUP BY city
HAVING SUM(revenue) > 100000;
```

| Clause | Filters |
|--------|---------|
| `WHERE` | raw rows before grouping |
| `HAVING` | groups after aggregation |

---

## Common Aggregates

| Function | Meaning |
|----------|---------|
| `COUNT(*)` | count rows |
| `COUNT(column)` | count non-null values |
| `SUM()` | total |
| `AVG()` | average |
| `MIN()` | minimum |
| `MAX()` | maximum |

---

## Practice

Write queries for:

- total orders
- total revenue
- revenue by city
- orders by category
- average revenue by city
- cities with revenue greater than 100000

---

## Interview Questions

**Q1:** Why do we use `GROUP BY`?

> To aggregate rows into category-level summaries.

**Q2:** Difference between `COUNT(*)` and `COUNT(column)`?

> `COUNT(*)` counts rows; `COUNT(column)` counts non-null values.

**Q3:** Can `WHERE` use aggregate functions?

> No. Use `HAVING` for aggregate filters.

---

## 🔗 What's Next?

➡️ [[03-joins]]
