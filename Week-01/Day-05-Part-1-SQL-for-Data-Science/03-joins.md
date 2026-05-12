# 🔗 03 — Joins
## Combining Tables

> [!info] Goal
> Learn how to combine related tables using SQL joins.

---

## Sample Tables

`orders`

| order_id | customer_id | product_id | revenue |
|----------|-------------|------------|---------|
| 101 | 1 | 10 | 150000 |
| 102 | 2 | 11 | 3000 |

`customers`

| customer_id | customer_name | city |
|-------------|---------------|------|
| 1 | Alice | Delhi |
| 2 | Bob | Mumbai |

---

## INNER JOIN

Keeps only matching rows.

```sql
SELECT
  o.order_id,
  c.customer_name,
  c.city,
  o.revenue
FROM orders o
INNER JOIN customers c
  ON o.customer_id = c.customer_id;
```

---

## LEFT JOIN

Keeps all rows from the left table.

```sql
SELECT
  o.order_id,
  c.customer_name,
  o.revenue
FROM orders o
LEFT JOIN customers c
  ON o.customer_id = c.customer_id;
```

Use left joins when you do not want to lose records from the main table.

---

## RIGHT and FULL JOIN

```sql
SELECT *
FROM orders o
RIGHT JOIN customers c
  ON o.customer_id = c.customer_id;
```

```sql
SELECT *
FROM orders o
FULL OUTER JOIN customers c
  ON o.customer_id = c.customer_id;
```

Not every database supports `FULL OUTER JOIN`.

---

## Joining Multiple Tables

```sql
SELECT
  o.order_id,
  c.customer_name,
  p.product_name,
  o.quantity,
  p.price,
  o.quantity * p.price AS revenue
FROM orders o
LEFT JOIN customers c
  ON o.customer_id = c.customer_id
LEFT JOIN products p
  ON o.product_id = p.product_id;
```

---

## Common Join Problems

### Row Explosion

Duplicate keys can multiply rows.

Check key uniqueness before joining.

### Missing Matches

After a left join:

```sql
WHERE c.customer_id IS NULL
```

finds orders without matching customers.

---

## Practice

Write queries to:

- join orders with customers
- join orders with products
- calculate revenue after joining products
- find customers with no orders
- find orders with missing customer details

---

## Interview Questions

**Q1:** Difference between inner and left join?

> Inner keeps matches only; left keeps all left-table rows.

**Q2:** Why can joins duplicate rows?

> Repeated keys can create many-to-many matches.

**Q3:** What join should you use for orders and customer details?

> Usually left join from orders to customers, to keep all orders.

---

## 🔗 What's Next?

➡️ [[04-subqueries]]
