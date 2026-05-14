# 🎯 08 — SQL Interview Questions

---

## Core Questions

**Q1:** What is SQL?

> SQL is a language for querying and managing relational databases.

---

**Q2:** What is the difference between `WHERE` and `HAVING`?

> `WHERE` filters rows before grouping. `HAVING` filters groups after aggregation.

---

**Q3:** What is the difference between `INNER JOIN` and `LEFT JOIN`?

> Inner join returns matching rows only. Left join keeps all rows from the left table.

---

**Q4:** What is a primary key?

> A column or set of columns that uniquely identifies each row.

---

**Q5:** What is a foreign key?

> A column that references a key in another table.

---

**Q6:** Difference between `COUNT(*)` and `COUNT(column)`?

> `COUNT(*)` counts rows. `COUNT(column)` counts non-null values.

---

**Q7:** What is a CTE?

> A named temporary result defined with `WITH` and used in a query.

---

**Q8:** What is a window function?

> A function that performs calculations across related rows while keeping row-level output.

---

**Q9:** Difference between `RANK()` and `DENSE_RANK()`?

> `RANK()` leaves gaps after ties. `DENSE_RANK()` does not.

---

**Q10:** What does `LAG()` do?

> It accesses a previous row's value based on window ordering.

---

## Write SQL

**Q11:** Find total revenue by city.

```sql
SELECT city, SUM(revenue) AS total_revenue
FROM orders
GROUP BY city;
```

---

**Q12:** Find top 5 customers by revenue.

```sql
SELECT customer_id, SUM(revenue) AS total_revenue
FROM orders
GROUP BY customer_id
ORDER BY total_revenue DESC
LIMIT 5;
```

---

**Q13:** Find orders above average revenue.

```sql
SELECT *
FROM orders
WHERE revenue > (
  SELECT AVG(revenue)
  FROM orders
);
```

---

**Q14:** Rank orders by revenue within each city.

```sql
SELECT
  order_id,
  city,
  revenue,
  RANK() OVER (PARTITION BY city ORDER BY revenue DESC) AS city_rank
FROM orders;
```

---

**Q15:** Find customers with no orders.

```sql
SELECT c.*
FROM customers c
LEFT JOIN orders o
  ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

---

## Scenario Questions

**Q16:** A join returns more rows than expected. Why?

> Duplicate keys may have created a many-to-many join.

---

**Q17:** A query has both `WHERE` and `HAVING`. Which runs first?

> `WHERE` filters raw rows before grouping. `HAVING` filters after grouping.

---

**Q18:** Why use CTEs in analytics SQL?

> They make multi-step logic easier to read, debug, and maintain.

---

## Checklist

- [ ] SELECT specific columns
- [ ] filter with WHERE
- [ ] aggregate with GROUP BY
- [ ] filter groups with HAVING
- [ ] join tables
- [ ] use subqueries and CTEs
- [ ] use window functions
- [ ] answer business questions in SQL

---

## 🔗 Next

➡️ [[../Day-05-Part-2-EDA/01-data-cleaning|EDA]]
