# 🔗 02 — Merge and Join
## Combining DataFrames

> [!info] Goal
> Learn how to combine related datasets using `merge()`, `join()`, and `concat()`.

---

## Why Combining Data Matters

Real data is often split across multiple tables:

- orders in one file
- customer details in another
- product details in another
- region mappings somewhere else

To analyze properly, you need to combine these tables.

---

## Sample Data

```python
import pandas as pd

orders = pd.DataFrame({
    "order_id": [101, 102, 103, 104],
    "customer_id": [1, 2, 1, 3],
    "product_id": [10, 11, 12, 10],
    "quantity": [2, 1, 3, 1]
})

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "customer_name": ["Alice", "Bob", "Charlie", "Diana"],
    "city": ["Delhi", "Mumbai", "Pune", "Bangalore"]
})

products = pd.DataFrame({
    "product_id": [10, 11, 12],
    "product_name": ["Laptop", "Mouse", "Keyboard"],
    "price": [75000, 600, 1200]
})
```

---

## `pd.merge()`

`merge()` combines DataFrames using one or more matching columns.

```python
orders_with_customers = pd.merge(
    orders,
    customers,
    on="customer_id"
)
```

This is similar to SQL joins.

---

## Join Types

| Join Type | Keeps |
|-----------|-------|
| `inner` | Only matching rows in both DataFrames |
| `left` | All rows from left DataFrame |
| `right` | All rows from right DataFrame |
| `outer` | All rows from both DataFrames |

### Inner Join

```python
pd.merge(orders, customers, on="customer_id", how="inner")
```

Keeps only matching customer IDs.

### Left Join

```python
pd.merge(orders, customers, on="customer_id", how="left")
```

Keeps all orders, even if customer details are missing.

### Outer Join

```python
pd.merge(orders, customers, on="customer_id", how="outer")
```

Keeps every customer and every order.

---

## Different Column Names

Sometimes keys have different names.

```python
orders_alt = orders.rename(columns={"customer_id": "cust_id"})

merged = pd.merge(
    orders_alt,
    customers,
    left_on="cust_id",
    right_on="customer_id",
    how="left"
)
```

---

## Merging Multiple Tables

```python
full_orders = (
    orders
    .merge(customers, on="customer_id", how="left")
    .merge(products, on="product_id", how="left")
)

full_orders["revenue"] = full_orders["quantity"] * full_orders["price"]
```

This creates one analysis-ready table.

---

## Handling Duplicate Column Names

```python
pd.merge(
    left_df,
    right_df,
    on="id",
    suffixes=("_left", "_right")
)
```

Use `suffixes` when both DataFrames contain columns with the same name.

---

## Validate Merge Expectations

```python
pd.merge(
    orders,
    customers,
    on="customer_id",
    how="left",
    validate="many_to_one"
)
```

Common validations:

| Validation | Meaning |
|------------|---------|
| `one_to_one` | Each key appears once in both |
| `one_to_many` | Left key once, right key multiple |
| `many_to_one` | Left key multiple, right key once |
| `many_to_many` | Keys can repeat on both sides |

This helps catch accidental duplicate joins.

---

## `concat()`

Use `concat()` to stack DataFrames.

### Stack Rows

```python
january = pd.DataFrame({"order_id": [1, 2], "revenue": [1000, 2000]})
february = pd.DataFrame({"order_id": [3, 4], "revenue": [1500, 2500]})

all_orders = pd.concat([january, february], ignore_index=True)
```

### Stack Columns

```python
pd.concat([df1, df2], axis=1)
```

---

## `join()`

`join()` is mostly used for joining on indexes.

```python
left = customers.set_index("customer_id")
right = orders.set_index("customer_id")

result = right.join(left, how="left")
```

For beginner and intermediate work, `merge()` is usually clearer.

---

## Common Problems

### Unexpected Row Explosion

If both tables have duplicate keys, a merge can create more rows than expected.

Check duplicates:

```python
customers["customer_id"].duplicated().sum()
orders["customer_id"].duplicated().sum()
```

Use `validate=...` to catch this early.

### Missing Matches

After a left join, missing values may mean the right table did not have matching keys.

```python
merged[merged["customer_name"].isna()]
```

### Mismatched Key Types

```python
orders["customer_id"].dtype
customers["customer_id"].dtype
```

If one is integer and one is text, convert before merging.

```python
orders["customer_id"] = orders["customer_id"].astype(str)
customers["customer_id"] = customers["customer_id"].astype(str)
```

---

## Practice

Using the sample data:

- merge orders with customers
- merge orders with products
- create a full order table with customer and product details
- calculate revenue
- find revenue by city
- try an outer join and inspect missing values
- use `validate="many_to_one"` in a merge

---

## Interview Questions

**Q1:** What is the difference between `merge()` and `concat()`?

> `merge()` combines tables by matching keys. `concat()` stacks tables along rows or columns.

**Q2:** What is a left join?

> A left join keeps all rows from the left DataFrame and adds matching values from the right DataFrame.

**Q3:** Why can merges create more rows than expected?

> Duplicate keys on both sides can create many-to-many matches, multiplying rows.

---

## ✅ Key Takeaways

- Use `merge()` for SQL-style joins.
- Use `concat()` for stacking DataFrames.
- Use `join()` mostly for index-based joins.
- Always check key columns before merging.
- Use `validate` to catch bad merge assumptions.
- After merging, inspect missing values and row counts.

---

## 🔗 What's Next?

➡️ [[03-apply-functions]] — Apply custom logic to rows and columns
