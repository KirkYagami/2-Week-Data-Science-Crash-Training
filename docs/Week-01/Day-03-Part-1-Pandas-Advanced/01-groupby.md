# GroupBy — Split, Apply, Combine

GroupBy is the engine behind almost every analytical question worth asking. Revenue by region, average salary by department, order count by customer segment — all of these are groupby questions. If you work with tabular data professionally, you will write groupby code every single day.

## Learning Objectives

By the end of this section you will be able to:

- Explain the split-apply-combine mental model and use it to decompose any aggregation problem
- Use `.groupby().agg()` with named aggregations to produce clean, readable summaries
- Choose between `as_index=False` and `.reset_index()` deliberately
- Use `.transform()` to add group-level statistics back onto the original DataFrame
- Use `.apply()` on groups for custom logic that `.agg()` cannot express
- Explain what happens to NaN values during groupby operations
- Avoid the most common groupby pitfalls that produce wrong answers silently

---

## The Mental Model: Split, Apply, Combine

Every groupby operation has three phases.

**Split:** Pandas divides the DataFrame into separate sub-DataFrames, one per unique group value.

**Apply:** A function runs on each sub-DataFrame independently — sum, mean, custom logic, anything.

**Combine:** The results from each group get assembled back into one output.

```python
import pandas as pd
import numpy as np

sales = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105, 106, 107, 108, 109, 110],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune", "Mumbai", "Delhi", "Pune", "Mumbai", "Delhi", "Pune"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories",
                 "Accessories", "Electronics", "Electronics", "Accessories",
                 "Accessories", "Electronics"],
    "product": ["Laptop", "Mouse", "Laptop", "Keyboard", "Mouse", "Monitor",
                "Monitor", "Keyboard", "USB Hub", "Laptop"],
    "quantity": [2, 5, 1, 3, 10, 2, 1, 4, 6, 1],
    "price": [75000, 600, 75000, 1200, 600, 15000, 15000, 1200, 800, 75000],
    "sales_rep": ["Priya", "Rahul", "Priya", "Anjali", "Rahul", "Priya",
                  "Anjali", "Rahul", "Anjali", "Priya"]
})

sales["revenue"] = sales["quantity"] * sales["price"]
```

The most basic operation: total revenue by city.

```python
city_revenue = sales.groupby("city")["revenue"].sum()
print(city_revenue)
# Output:
# city
# Delhi     247800
# Mumbai     11400
# Pune       91800
# Name: revenue, dtype: int64
```

Three things to notice: the group key (`city`) becomes the index, the output is a Series, and the three phases happened invisibly — Pandas split by city, summed each group, and combined the results.

---

## Common Aggregation Methods

```python
# Single aggregation
sales.groupby("category")["revenue"].sum()
sales.groupby("category")["revenue"].mean()
sales.groupby("category")["revenue"].median()
sales.groupby("category")["quantity"].max()
sales.groupby("category")["quantity"].min()
sales.groupby("category")["revenue"].std()

# Count vs size — these differ when NaNs exist
sales.groupby("city")["revenue"].count()  # non-null values only
sales.groupby("city").size()              # all rows, including NaN rows
```

| Method | What it computes | NaN behavior |
|--------|-----------------|--------------|
| `sum()` | Total | Ignores NaN |
| `mean()` | Average | Ignores NaN |
| `median()` | Middle value | Ignores NaN |
| `count()` | Non-null rows | Excludes NaN |
| `size()` | All rows | Includes NaN rows |
| `std()` | Standard deviation | Ignores NaN |
| `min()` / `max()` | Extremes | Ignores NaN |
| `nunique()` | Distinct count | Ignores NaN |

> [!warning] `size()` vs `count()` is a common source of wrong results
> If your data has missing values in the column you are counting, `count()` will give a smaller number than `size()`. Neither is wrong — they answer different questions. Ask yourself: do I want to count rows in each group, or non-missing values in each group?

---

## Named Aggregations: The Right Way to Build Summaries

The old way of passing a list to `.agg()` creates column names like `(revenue, sum)`, which are annoying to work with downstream. Named aggregations solve this.

```python
category_summary = sales.groupby("category").agg(
    total_revenue=("revenue", "sum"),
    avg_revenue=("revenue", "mean"),
    median_revenue=("revenue", "median"),
    total_quantity=("quantity", "sum"),
    order_count=("order_id", "count"),
    unique_products=("product", "nunique")
)

print(category_summary)
# Output:
#              total_revenue  avg_revenue  median_revenue  total_quantity  order_count  unique_products
# category
# Accessories          18000       3600.0          3000.0              28            5                3
# Electronics         333000      66600.0         75000.0               7            5                3
```

The syntax is `output_column_name=("source_column", "aggregation_function")`. Each named argument becomes one output column. This is the pattern you should default to — it produces a clean DataFrame that is ready for further analysis or export.

---

## Grouping by Multiple Keys

Multi-key groupby answers questions like: what is the revenue breakdown by city and category?

```python
city_category = sales.groupby(["city", "category"]).agg(
    total_revenue=("revenue", "sum"),
    order_count=("order_id", "count")
)

print(city_category)
# Output:
#                           total_revenue  order_count
# city   category
# Delhi  Accessories              4800            1
#        Electronics            243000            3
# Mumbai Accessories            11400            4
#        Electronics                0            0
# Pune   Accessories              3600            1
#        Electronics             88200            2
```

The result has a **MultiIndex** — both `city` and `category` are part of the index.

```python
# Convert to flat columns for easier downstream use
city_category_flat = city_category.reset_index()
print(city_category_flat.columns)
# Output: Index(['city', 'category', 'total_revenue', 'order_count'], dtype='object')
```

> [!tip] Always reset the index when you need to use group keys as filter conditions
> After a groupby, the group keys live in the index. If you forget to call `.reset_index()` and then try to filter on `df["city"]`, you will get a KeyError because `city` is the index, not a column. Use `reset_index()` or pass `as_index=False` to keep group keys as columns from the start.

---

## `as_index=False` — Keeping Group Keys as Columns

Two ways to get the same result:

```python
# Option A: reset_index after
summary_a = (
    sales.groupby("city")["revenue"]
    .sum()
    .reset_index()
)

# Option B: as_index=False during groupby
summary_b = sales.groupby("city", as_index=False)["revenue"].sum()

print(summary_a)
# Output:
#      city  revenue
# 0   Delhi   247800
# 1  Mumbai    11400
# 2    Pune    91800
```

Both produce the same result. Use `as_index=False` when you know upfront that you want columns — it saves you from writing `.reset_index()` every time.

> [!info] Why this matters in practice
> When you pass a grouped result into another merge or into a plotting function, it almost always needs to be a flat DataFrame with regular columns. Getting into the habit of `reset_index()` saves you cryptic errors downstream.

---

## `.transform()` — The Underrated One

`.agg()` reduces each group to one row. `.transform()` does something different: it computes group-level statistics but returns values aligned to the **original DataFrame's index** — same number of rows, same order, no reduction.

This is useful when you want to add a group summary back onto each original row.

**Use case:** flag every order that is above-average for its category.

```python
# Without transform, you would need a merge to get this back on the original rows
sales["category_avg_revenue"] = (
    sales.groupby("category")["revenue"].transform("mean")
)

sales["above_category_avg"] = sales["revenue"] > sales["category_avg_revenue"]

print(sales[["order_id", "category", "revenue", "category_avg_revenue", "above_category_avg"]])
# Output:
#    order_id     category  revenue  category_avg_revenue  above_category_avg
# 0       101  Electronics   150000               66600.0                True
# 1       102  Accessories     3000                3600.0               False
# 2       103  Electronics    75000               66600.0                True
# ...
```

**Another use case:** compute each rep's share of total city revenue.

```python
sales["city_total_revenue"] = sales.groupby("city")["revenue"].transform("sum")
sales["city_revenue_share"] = (sales["revenue"] / sales["city_total_revenue"] * 100).round(1)

print(sales[["order_id", "city", "revenue", "city_revenue_share"]].head(6))
```

> [!tip] `.transform()` replaces a merge
> Before discovering `.transform()`, many people do a groupby, save the result, then merge it back onto the original. That works but requires three steps. `.transform()` does the same thing in one line — and it preserves the original index alignment automatically.

You can pass any string aggregation name or a function to `.transform()`:

```python
# These all work
sales.groupby("category")["revenue"].transform("mean")
sales.groupby("category")["revenue"].transform("sum")
sales.groupby("category")["revenue"].transform("rank")
sales.groupby("category")["revenue"].transform(lambda x: (x - x.mean()) / x.std())  # z-score within group
```

---

## `.apply()` on Groups — When You Need Custom Logic

`.apply()` on a GroupBy object lets you run an arbitrary function on each sub-DataFrame. Use it when `.agg()` and `.transform()` cannot express what you need.

**Use case:** return the top-revenue order from each city.

```python
def top_order(group_df):
    return group_df.nlargest(1, "revenue")

top_orders_by_city = sales.groupby("city").apply(top_order)
print(top_orders_by_city[["order_id", "city", "product", "revenue"]])
# Output:
#              order_id    city   product  revenue
# city
# Delhi   0        101   Delhi    Laptop   150000
# Mumbai  7        108  Mumbai  Keyboard     4800
# Pune    9        110    Pune    Laptop    75000
```

> [!warning] `.apply()` on groups is powerful but slow on large DataFrames
> Pandas has to iterate over each group and call your function separately. On a DataFrame with millions of rows and thousands of groups, this can be painfully slow. If you can express the same logic with `.agg()`, `.transform()`, or vectorized operations, do that instead. Save `.apply()` on groups for cases where no vectorized alternative exists.

---

## `filter()` — Removing Entire Groups

`.filter()` keeps or removes entire groups based on a boolean function. It is not the same as filtering rows.

```python
# Keep only cities where total revenue exceeds 100,000
high_revenue_cities = sales.groupby("city").filter(
    lambda group: group["revenue"].sum() > 100000
)

print(high_revenue_cities["city"].unique())
# Output: ['Delhi' 'Pune']
```

The result retains the original row structure — it is not summarized.

---

## NaN Behavior in GroupBy

By default, Pandas **excludes** NaN values from group keys. A row whose group key is NaN does not appear in any group.

```python
sales_with_gap = sales.copy()
sales_with_gap.loc[2, "city"] = None  # make one city NaN

# The NaN row is silently excluded
print(sales_with_gap.groupby("city")["revenue"].sum())
# Note: order_id 103's row (Delhi, NaN city) does not appear in any group
```

```python
# To include NaN as its own group, use dropna=False
print(sales_with_gap.groupby("city", dropna=False)["revenue"].sum())
# Output includes a NaN group
```

> [!warning] Silent exclusion of NaN group keys can skew totals
> If you are computing a grand total by summing all groups, and some rows have NaN keys, those rows are silently excluded. Your group totals will not add up to the DataFrame total. Always check `df["key_column"].isna().sum()` before running a groupby that needs to account for all rows.

---

## Sorting Grouped Results

```python
# Top products by revenue
top_products = (
    sales.groupby("product")["revenue"]
    .sum()
    .sort_values(ascending=False)
)

print(top_products)
# Output:
# product
# Laptop      375000
# Monitor      30000
# Keyboard      8400
# ...
```

---

## Common Pitfalls

### Pitfall 1: Forgetting `reset_index()` before filtering or merging

```python
city_rev = sales.groupby("city")["revenue"].sum()
# city is the INDEX, not a column

# This will KeyError:
# city_rev[city_rev["city"] == "Delhi"]

# Correct:
city_rev = city_rev.reset_index()
city_rev[city_rev["city"] == "Delhi"]
```

### Pitfall 2: Using `.agg()` when you meant `.transform()`

If you want the group-level value on every original row (for example, to compute a ratio), you need `.transform()`, not `.agg()`. `.agg()` reduces the DataFrame — you cannot assign its output back to a column directly.

```python
# Wrong — this will not align properly:
# sales["cat_total"] = sales.groupby("category")["revenue"].agg("sum")

# Correct:
sales["cat_total"] = sales.groupby("category")["revenue"].transform("sum")
```

### Pitfall 3: Grouping before creating the column you need

```python
# This fails if "revenue" does not exist yet
sales.groupby("category")["revenue"].sum()

# Always create derived columns first
sales["revenue"] = sales["quantity"] * sales["price"]
sales.groupby("category")["revenue"].sum()
```

### Pitfall 4: The `observed` parameter with Categorical columns

If you have Categorical dtype columns and do not pass `observed=True`, Pandas will include groups for every category level — even ones with zero rows. This creates phantom rows in your summary.

```python
sales["category"] = pd.Categorical(
    sales["category"],
    categories=["Electronics", "Accessories", "Software"]  # Software doesn't exist in data
)

# Bad: includes a Software row with NaN/zero values
print(sales.groupby("category")["revenue"].sum())

# Good: only shows groups that actually exist in the data
print(sales.groupby("category", observed=True)["revenue"].sum())
```

---

> [!success] Key Takeaways
> - GroupBy follows split-apply-combine. Once you see it that way, every aggregation problem becomes straightforward.
> - Use named aggregations with `.agg()` — they produce clean column names and readable code.
> - Use `.transform()` when you need group-level values attached to original rows. It replaces the groupby-then-merge pattern.
> - Always check `reset_index()` / `as_index=False` so group keys come back as columns.
> - NaN group keys are silently excluded by default — use `dropna=False` if you need them.

---

[[02-merge-and-join]] — Combine multiple DataFrames with merge, join, and concat
