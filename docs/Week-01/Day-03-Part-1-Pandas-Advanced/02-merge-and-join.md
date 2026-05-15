# Merge and Join — Combining DataFrames

Real data is never in one table. Orders live in one file, customer details in another, product catalog in a third, regional mappings somewhere else. The skill that separates analysts who can answer cross-table questions from those who cannot is knowing how to combine DataFrames correctly — and, more importantly, knowing when a merge has silently gone wrong.

I once debugged a production report that was inflating revenue by 40%. The bug was a merge that produced 2.4 million rows from two tables with 600k and 800k rows respectively, because neither table had unique keys on the join column. The report had been running for three months. Nobody noticed because the absolute numbers looked plausible. The `validate` parameter would have caught it on day one.

## Learning Objectives

By the end of this section you will be able to:

- Choose between `pd.merge()`, `.join()`, and `pd.concat()` based on what the operation needs to do
- Explain all four join types with concrete intuition, not just definitions
- Use `left_on` / `right_on` when key column names differ across DataFrames
- Use the `validate` parameter to catch duplicate-key bugs before they corrupt results
- Diagnose and fix the most common merge problems: row explosion, missing matches, mismatched dtypes, and the dreaded `_x` / `_y` column clutter
- Chain multiple merges to build a fully joined, analysis-ready table

---

## The Three Tools and When to Use Each

| Tool | Use when |
|------|----------|
| `pd.merge()` | Joining on one or more shared key columns — the SQL JOIN equivalent |
| `pd.concat()` | Stacking DataFrames vertically (more rows) or horizontally (more columns) |
| `.join()` | Joining on the DataFrame index — mostly useful after `.set_index()` |

In practice, `pd.merge()` handles 90% of real-world joining. Learn it deeply.

---

## Sample Data

```python
import pandas as pd
import numpy as np

orders = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105],
    "customer_id": [1, 2, 1, 3, 9],   # customer 9 does not exist in customers table
    "product_id": [10, 11, 12, 10, 13],  # product 13 does not exist in products table
    "quantity": [2, 1, 3, 1, 2],
    "order_date": pd.to_datetime(["2024-01-10", "2024-01-12", "2024-01-15", "2024-01-18", "2024-01-20"])
})

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],  # customer 4 has no orders
    "customer_name": ["Alice Sharma", "Bob Menon", "Charlie Rao", "Diana Nair"],
    "city": ["Delhi", "Mumbai", "Pune", "Bangalore"],
    "tier": ["Gold", "Silver", "Gold", "Bronze"]
})

products = pd.DataFrame({
    "product_id": [10, 11, 12],  # product 13 is missing
    "product_name": ["Laptop", "Mouse", "Keyboard"],
    "price": [75000, 600, 1200],
    "category": ["Electronics", "Accessories", "Accessories"]
})
```

---

## `pd.merge()` — SQL-Style Joins

```python
# Basic merge: match rows where customer_id is equal in both tables
orders_with_customers = pd.merge(orders, customers, on="customer_id")
print(orders_with_customers)
# Output:
#    order_id  customer_id  product_id  quantity order_date customer_name    city    tier
# 0       101            1          10         2 2024-01-10  Alice Sharma   Delhi    Gold
# 1       103            1          12         3 2024-01-15  Alice Sharma   Delhi    Gold
# 2       102            2          11         1 2024-01-12     Bob Menon  Mumbai  Silver
# 3       104            3          10         1 2024-01-18   Charlie Rao    Pune    Gold
```

Notice: order 105 (customer 9) is missing from the result. The default join type is `inner`, which only keeps rows that match on both sides.

---

## The Four Join Types

### Inner Join — Only Matching Rows

```python
inner = pd.merge(orders, customers, on="customer_id", how="inner")
print(f"Orders: {len(orders)} rows -> Inner join: {len(inner)} rows")
# Output: Orders: 5 rows -> Inner join: 4 rows
# (order 105 with customer_id=9 is dropped — no match in customers)
```

Use inner join when: you only care about complete records. Be aware that you lose data silently.

### Left Join — All Rows from the Left Table

```python
left = pd.merge(orders, customers, on="customer_id", how="left")
print(f"Orders: {len(orders)} rows -> Left join: {len(left)} rows")
# Output: Orders: 5 rows -> Left join: 5 rows

# Order 105 is preserved, but customer columns are NaN
print(left[left["customer_id"] == 9])
# Output:
#    order_id  customer_id  product_id  quantity order_date customer_name city tier
# 4       105            9          13         2 2024-01-20           NaN  NaN  NaN
```

Left join is the most commonly used type in analysis. You want to keep all your primary records and attach whatever information is available. Missing matches show up as NaN, which you can inspect and handle deliberately.

### Right Join — All Rows from the Right Table

```python
right = pd.merge(orders, customers, on="customer_id", how="right")
print(f"Right join: {len(right)} rows")
# Output: Right join: 5 rows
# (customer 4, Diana, is preserved even though she has no orders)

print(right[right["customer_id"] == 4])
# Output:
#    order_id  customer_id  product_id  quantity order_date customer_name      city    tier
# 4       NaN            4         NaN       NaN        NaT   Diana Nair  Bangalore  Bronze
```

Right joins are rare. Most people write the same result as a left join by swapping the argument order. They are equivalent.

### Outer (Full) Join — All Rows from Both Tables

```python
outer = pd.merge(orders, customers, on="customer_id", how="outer")
print(f"Outer join: {len(outer)} rows")
# Output: Outer join: 6 rows
# (all 5 orders + customer 4 who has no orders)
```

Outer join is useful for auditing — you can see unmatched rows on both sides. Use it to answer "what exists in table A that has no match in table B, and vice versa?"

---

## Visual Intuition

```
orders table:       customers table:
customer_id         customer_id
1                   1
2                   2
1                   3
3                   4  ← no matching order
9  ← no match

inner:  keeps 1,2,1,3          (4 rows — drops order with 9, drops customer 4)
left:   keeps 1,2,1,3,9        (5 rows — NaN for customer 9's details)
right:  keeps 1,2,1,3,4        (5 rows — NaN for customer 4's order details)
outer:  keeps 1,2,1,3,9,4      (6 rows — NaN where no match exists)
```

---

## When Key Column Names Differ

The `on=` parameter only works when both tables have the same column name for the key. When names differ, use `left_on` and `right_on`.

```python
# Suppose orders uses "cust_id" instead of "customer_id"
orders_alt = orders.rename(columns={"customer_id": "cust_id"})

merged = pd.merge(
    orders_alt,
    customers,
    left_on="cust_id",
    right_on="customer_id",
    how="left"
)

# Both "cust_id" and "customer_id" appear in the result — drop the redundant one
merged = merged.drop(columns=["customer_id"])
```

---

## The `validate` Parameter — Your Safety Net

This is the single most important parameter most people never use.

```python
# Tell Pandas what you expect the key relationship to be
pd.merge(
    orders,
    customers,
    on="customer_id",
    how="left",
    validate="many_to_one"  # many orders can share one customer, but each customer_id is unique in customers
)
```

| Validation value | What it checks |
|-----------------|----------------|
| `"one_to_one"` | Key is unique in both tables |
| `"one_to_many"` | Key is unique in the left table |
| `"many_to_one"` | Key is unique in the right table |
| `"many_to_many"` | No uniqueness requirement (allows row explosion) |

If the actual data violates your expectation, Pandas raises `MergeError` immediately — before you compute anything wrong.

```python
# Example of validate catching a problem
customers_with_dupe = pd.concat([
    customers,
    pd.DataFrame({"customer_id": [1], "customer_name": ["Alice Duplicate"],
                  "city": ["Delhi"], "tier": ["Gold"]})
])

try:
    pd.merge(orders, customers_with_dupe, on="customer_id",
             how="left", validate="many_to_one")
except Exception as e:
    print(f"Caught: {e}")
# Output: Caught: Merge keys are not unique in right dataset; not a many-to-one merge
```

> [!tip] Make `validate` a habit on every merge
> Pass `validate="many_to_one"` or `validate="one_to_one"` on every merge where you expect clean keys. It adds one argument and saves you from debugging corrupt results hours later. The performance cost is negligible.

---

## Duplicate Column Names and the `_x` / `_y` Problem

When both tables have a column with the same name (other than the key column), Pandas appends `_x` and `_y` suffixes.

```python
# Both tables have a "city" column (hypothetically)
orders_with_city = orders.copy()
orders_with_city["city"] = "Order City"

result = pd.merge(orders_with_city, customers, on="customer_id", how="left")
print(result.columns.tolist())
# Output: ['order_id', 'customer_id', 'product_id', 'quantity', 'order_date', 'city_x', 'customer_name', 'city_y', 'tier']
```

`city_x` came from orders, `city_y` from customers. The default `_x`/`_y` is ambiguous and ugly.

```python
# Fix: use explicit suffixes
result = pd.merge(
    orders_with_city,
    customers,
    on="customer_id",
    how="left",
    suffixes=("_order", "_customer")
)
# Now you get city_order and city_customer — far clearer

# Or better: rename the ambiguous column before merging
orders_with_city = orders_with_city.rename(columns={"city": "order_city"})
result = pd.merge(orders_with_city, customers, on="customer_id", how="left")
```

> [!warning] `_x` and `_y` columns are a smell
> If you see `_x` or `_y` in your column names after a merge, something is ambiguous. Either rename columns beforehand or use explicit `suffixes`. Never leave `_x`/`_y` columns in a result you plan to use downstream — they lead to bugs when you accidentally reference the wrong one.

---

## Chaining Multiple Merges

Building a fully joined table from three source tables:

```python
full_orders = (
    orders
    .merge(customers, on="customer_id", how="left", validate="many_to_one")
    .merge(products, on="product_id", how="left", validate="many_to_one")
)

full_orders["revenue"] = full_orders["quantity"] * full_orders["price"]

print(full_orders[["order_id", "customer_name", "product_name", "quantity", "price", "revenue"]])
# Output:
#    order_id customer_name product_name  quantity   price   revenue
# 0       101  Alice Sharma       Laptop         2   75000  150000.0
# 1       102     Bob Menon        Mouse         1     600     600.0
# 2       103  Alice Sharma     Keyboard         3    1200    3600.0
# 3       104   Charlie Rao       Laptop         1   75000   75000.0
# 4       105           NaN          NaN         2     NaN       NaN
```

Order 105 has no customer match (customer_id=9) and no product match (product_id=13), so those columns are NaN. This is correct behavior — it flags data quality issues rather than hiding them.

> [!info] Chaining merges reads naturally
> The chained syntax (`.merge().merge()`) reads like a sentence: "take orders, attach customer info, then attach product info." This is much cleaner than saving intermediate variables. Use it freely.

---

## `pd.concat()` — Stacking DataFrames

`concat()` does not match on keys. It stacks — either more rows (vertically) or more columns (horizontally).

### Stacking rows — combining datasets of the same structure

```python
jan_orders = pd.DataFrame({
    "order_id": [201, 202],
    "product": ["Laptop", "Mouse"],
    "revenue": [150000, 3000]
})

feb_orders = pd.DataFrame({
    "order_id": [301, 302, 303],
    "product": ["Monitor", "Keyboard", "Laptop"],
    "revenue": [30000, 3600, 75000]
})

all_orders = pd.concat([jan_orders, feb_orders], ignore_index=True)
print(all_orders)
# Output:
#    order_id   product  revenue
# 0       201    Laptop   150000
# 1       202     Mouse     3000
# 2       301   Monitor    30000
# 3       302  Keyboard     3600
# 4       303    Laptop    75000
```

`ignore_index=True` resets the row index. Without it, you keep the original indices (0,1 from jan, 0,1,2 from feb), which causes duplicate index values.

> [!warning] `pd.concat()` aligns on column names, not position
> If the two DataFrames have different column names, concat produces NaN wherever a column is missing. This is correct behavior but it surprises people. Always inspect `df.isna().sum()` after concatenating DataFrames from different sources.

```python
# If column names differ, you get NaN columns
jan_v2 = jan_orders.rename(columns={"revenue": "total"})
mismatched = pd.concat([jan_v2, feb_orders])
print(mismatched)
# Output has both "revenue" and "total" columns, with NaN where each is absent
```

### Stacking columns — adding columns from another DataFrame

```python
# Useful when two DataFrames have the same rows but different features
features_a = pd.DataFrame({"age": [25, 32, 29], "income": [50000, 80000, 60000]})
features_b = pd.DataFrame({"credit_score": [720, 680, 750], "region": ["North", "South", "West"]})

combined = pd.concat([features_a, features_b], axis=1)
print(combined)
# Output:
#    age  income  credit_score region
# 0   25   50000           720  North
# 1   32   80000           680  South
# 2   29   60000           750   West
```

Only use `axis=1` concat when you are certain the rows correspond to each other. There is no key matching happening here.

---

## `.join()` — Index-Based Joins

`.join()` matches on the DataFrame index, not on a column. It is essentially a shorthand for `pd.merge()` with `left_index=True, right_index=True`.

```python
customers_indexed = customers.set_index("customer_id")
orders_indexed = orders.set_index("customer_id")

result = orders_indexed.join(customers_indexed, how="left")
```

In most real-world work, `pd.merge()` is clearer. Use `.join()` when you deliberately work with indexed data structures.

---

## Diagnosing Merge Problems

### Row explosion — more output rows than expected

```python
# Before any merge, check for duplicate keys
print(f"Duplicate customer_ids in customers: {customers['customer_id'].duplicated().sum()}")
print(f"Orders per customer:\n{orders['customer_id'].value_counts()}")
```

If the right table (the "one" side) has duplicate keys and you do a many-to-one merge, every row in the left table that matches will be duplicated for each match on the right. This multiplies rows silently.

```python
# Always validate
pd.merge(orders, customers, on="customer_id", how="left", validate="many_to_one")
# Raises MergeError if customers has duplicate customer_ids
```

### Missing matches after a left join

```python
full = pd.merge(orders, customers, on="customer_id", how="left")
unmatched = full[full["customer_name"].isna()]
print(f"Orders with no customer match: {len(unmatched)}")
print(unmatched[["order_id", "customer_id"]])
```

### Mismatched key types

```python
# Integer vs string is the most common cause of zero matches after a merge
print(orders["customer_id"].dtype)    # int64
print(customers["customer_id"].dtype)  # may be object (string) if read from CSV

# Fix by converting to the same type
orders["customer_id"] = orders["customer_id"].astype(int)
customers["customer_id"] = customers["customer_id"].astype(int)
```

> [!warning] Merging integer keys against string keys produces zero matches silently
> `1 == "1"` is False in Python and in Pandas. If your key columns have mismatched dtypes, the merge will succeed without error but produce zero matched rows. The tell-tale sign is that all columns from the right table are NaN after the merge. Always check `.dtype` on both key columns before merging.

---

> [!success] Key Takeaways
> - `pd.merge()` is your primary tool for SQL-style joins. `pd.concat()` stacks. `.join()` is index-based.
> - The four join types determine which unmatched rows survive: inner drops both sides, left keeps left, right keeps right, outer keeps all.
> - Always pass `validate=` on merges where keys should be unique on one or both sides.
> - `_x` / `_y` column names are a warning sign — rename columns or use explicit `suffixes`.
> - After any merge, check row count and inspect NaN values before computing on the result.

---

[[01-groupby]] — Split, Apply, Combine | [[03-apply-functions]] — Apply custom logic to rows and columns
