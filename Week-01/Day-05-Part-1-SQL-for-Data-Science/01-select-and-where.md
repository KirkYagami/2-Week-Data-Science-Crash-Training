# 🗄️ 01 — SELECT and WHERE
## Querying Rows and Columns

> [!info] Goal
> Learn the basic SQL query pattern used to retrieve, filter, and sort data.

---

## Sample Table: `orders`

| order_id | customer_id | city | category | revenue | order_date |
|----------|-------------|------|----------|---------|------------|
| 101 | 1 | Delhi | Electronics | 150000 | 2025-01-05 |
| 102 | 2 | Mumbai | Accessories | 3000 | 2025-01-07 |
| 103 | 1 | Delhi | Electronics | 75000 | 2025-02-01 |
| 104 | 3 | Pune | Accessories | 3600 | 2025-02-10 |

---

## Basic SELECT

```sql
SELECT *
FROM orders;
```

Select specific columns:

```sql
SELECT order_id, city, revenue
FROM orders;
```

---

## Filter with WHERE

```sql
SELECT *
FROM orders
WHERE city = 'Delhi';
```

Numeric filter:

```sql
SELECT *
FROM orders
WHERE revenue > 50000;
```

---

## Multiple Conditions

```sql
SELECT *
FROM orders
WHERE city = 'Delhi'
  AND revenue > 50000;
```

```sql
SELECT *
FROM orders
WHERE city = 'Delhi'
   OR city = 'Mumbai';
```

---

## IN, BETWEEN, LIKE

```sql
SELECT *
FROM orders
WHERE city IN ('Delhi', 'Mumbai');
```

```sql
SELECT *
FROM orders
WHERE revenue BETWEEN 10000 AND 100000;
```

```sql
SELECT *
FROM orders
WHERE category LIKE 'Elect%';
```

---

## NULL Checks

```sql
SELECT *
FROM orders
WHERE revenue IS NULL;
```

```sql
SELECT *
FROM orders
WHERE revenue IS NOT NULL;
```

Do not use `= NULL`.

---

## Sorting and Limiting

```sql
SELECT *
FROM orders
ORDER BY revenue DESC;
```

```sql
SELECT *
FROM orders
ORDER BY revenue DESC
LIMIT 5;
```

---

## Query Order

SQL is written like:

```sql
SELECT columns
FROM table
WHERE filters
ORDER BY column
LIMIT n;
```

Logical processing order is closer to:

```text
FROM → WHERE → SELECT → ORDER BY → LIMIT
```

---

## Practice

Write queries to:

- select all orders
- select only city and revenue
- find Delhi orders
- find revenue greater than 50000
- find orders from Delhi or Mumbai
- find top 3 orders by revenue

---

## Interview Questions

**Q1:** What does `WHERE` do?

> It filters rows before results are returned.

**Q2:** Difference between `WHERE` and `HAVING`?

> `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation.

**Q3:** How do you check NULL?

> Use `IS NULL` or `IS NOT NULL`.

---

## 🔗 What's Next?

➡️ [[02-group-by-and-having]]
