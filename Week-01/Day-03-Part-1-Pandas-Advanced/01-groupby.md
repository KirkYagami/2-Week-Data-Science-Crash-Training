# 📊 01 — GroupBy
## Split, Apply, Combine

> [!info] Goal
> Learn how to summarize data by categories using Pandas `groupby()`.

---

## Why GroupBy Matters

Most real analysis questions are grouped questions:

- What is revenue by city?
- What is average salary by department?
- Which product category sells the most?
- How many customers are in each segment?

Pandas `groupby()` helps answer these questions.

---

## Sample Dataset

```python
import pandas as pd

sales = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105, 106, 107, 108],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune", "Mumbai", "Delhi", "Pune", "Mumbai"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories", "Accessories", "Electronics", "Electronics", "Accessories"],
    "product": ["Laptop", "Mouse", "Laptop", "Keyboard", "Mouse", "Monitor", "Monitor", "Keyboard"],
    "quantity": [2, 5, 1, 3, 10, 2, 1, 4],
    "price": [75000, 600, 75000, 1200, 600, 15000, 15000, 1200]
})

sales["revenue"] = sales["quantity"] * sales["price"]
```

---

## Basic GroupBy

```python
sales.groupby("city")["revenue"].sum()
```

This means:

1. Split rows by city.
2. Select the `revenue` column.
3. Sum revenue inside each city group.

---

## Common Aggregations

```python
sales.groupby("category")["revenue"].sum()
sales.groupby("category")["revenue"].mean()
sales.groupby("category")["revenue"].count()
sales.groupby("category")["quantity"].max()
sales.groupby("category")["quantity"].min()
```

Useful aggregation methods:

| Method | Meaning |
|--------|---------|
| `sum()` | Total |
| `mean()` | Average |
| `median()` | Middle value |
| `count()` | Non-missing count |
| `size()` | Row count |
| `min()` | Minimum |
| `max()` | Maximum |
| `std()` | Standard deviation |

---

## Multiple Aggregations

```python
sales.groupby("category")["revenue"].agg(["sum", "mean", "count"])
```

Output columns show each aggregation.

### Named Aggregations

```python
summary = sales.groupby("category").agg(
    total_revenue=("revenue", "sum"),
    average_revenue=("revenue", "mean"),
    total_quantity=("quantity", "sum"),
    order_count=("order_id", "count")
)

print(summary)
```

Named aggregations produce cleaner column names.

---

## Grouping by Multiple Columns

```python
sales.groupby(["city", "category"])["revenue"].sum()
```

This creates a multi-level index.

To convert it back into normal columns:

```python
city_category_revenue = (
    sales.groupby(["city", "category"])["revenue"]
    .sum()
    .reset_index()
)
```

---

## Sorting Grouped Results

```python
sales.groupby("product")["revenue"].sum().sort_values(ascending=False)
```

This answers: which products generated the most revenue?

---

## `size()` vs `count()`

```python
sales.groupby("city").size()
sales.groupby("city")["revenue"].count()
```

| Method | Counts |
|--------|--------|
| `size()` | All rows in each group |
| `count()` | Non-missing values in selected column |

If missing values exist, `size()` and `count()` can differ.

---

## GroupBy with `as_index=False`

```python
summary = sales.groupby("city", as_index=False)["revenue"].sum()
```

This keeps `city` as a normal column instead of turning it into the index.

---

## Transform

`transform()` returns a result with the same number of rows as the original DataFrame.

Example: compare each order to its category average.

```python
sales["category_avg_revenue"] = (
    sales.groupby("category")["revenue"].transform("mean")
)

sales["above_category_avg"] = sales["revenue"] > sales["category_avg_revenue"]
```

Use `transform()` when you need group-level values back on each original row.

---

## Filtering Groups

Keep only cities with total revenue above 100000:

```python
large_cities = sales.groupby("city").filter(
    lambda group: group["revenue"].sum() > 100000
)
```

---

## Common Patterns

### Revenue by Category

```python
sales.groupby("category")["revenue"].sum()
```

### Average Order Value by City

```python
sales.groupby("city")["revenue"].mean()
```

### Top Product by Revenue

```python
sales.groupby("product")["revenue"].sum().sort_values(ascending=False).head(1)
```

### Department Salary Summary

```python
employees.groupby("department").agg(
    avg_salary=("salary", "mean"),
    max_salary=("salary", "max"),
    employee_count=("name", "count")
)
```

---

## Common Mistakes

### Forgetting to Reset Index

```python
summary = sales.groupby("city")["revenue"].sum()
```

This returns a Series with `city` as the index.

Use:

```python
summary = sales.groupby("city", as_index=False)["revenue"].sum()
```

### Grouping Before Creating Needed Columns

```python
sales.groupby("category")["revenue"].sum()
```

This fails if `revenue` does not exist yet.

Create it first:

```python
sales["revenue"] = sales["quantity"] * sales["price"]
```

---

## Practice

Using the `sales` DataFrame:

- calculate total revenue by city
- calculate total revenue by category
- calculate total quantity by product
- find average order value by city
- create a summary with total revenue, average revenue, and order count by category
- find the product with the highest total revenue
- add a column showing each order's city average revenue using `transform()`

---

## Interview Questions

**Q1:** What does `groupby()` do?

> It splits data into groups, applies an operation to each group, and combines the results.

**Q2:** What is the difference between `agg()` and `transform()`?

> `agg()` reduces each group to one row or value. `transform()` returns values aligned with the original DataFrame.

**Q3:** Why use `as_index=False`?

> It keeps group keys as normal columns, which is often easier for further analysis or exporting.

---

## ✅ Key Takeaways

- `groupby()` is essential for category-level analysis.
- Use `agg()` for multiple summary metrics.
- Use `reset_index()` or `as_index=False` for cleaner tabular output.
- Use `transform()` when group-level values should be attached to original rows.
- Sort grouped results to find top categories, products, or cities.

---

## 🔗 What's Next?

➡️ [[02-merge-and-join]] — Combine multiple DataFrames
