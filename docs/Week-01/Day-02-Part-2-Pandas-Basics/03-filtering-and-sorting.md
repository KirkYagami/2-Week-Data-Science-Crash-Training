# Filtering and Sorting

Loading data is just the start. Every real analysis is a series of focused questions: "which customers spent over ₹10,000 this quarter?" or "what are the five worst-performing products?" Filtering and sorting are how you ask those questions directly in code. Get fast at this and your analysis loop accelerates dramatically.

## Learning Objectives

- Write boolean index expressions to filter rows by condition
- Combine conditions with `&`, `|`, and `~` correctly
- Use `.loc` for label-based row+column selection in a single expression
- Use `.iloc` for position-based access
- Use `.isin()` for membership filtering
- Use `.query()` for readable filter expressions
- Use `.str` accessor methods for text filtering
- Sort by one or multiple columns with mixed ascending/descending order
- Use `.nlargest()` and `.nsmallest()` for efficient top/bottom selection
- Avoid the `SettingWithCopyWarning` trap

---

## The Sample Dataset

Use this throughout this file. The realistic column names make patterns easier to remember.

```python
import pandas as pd

sales = pd.DataFrame({
    "order_id": [1001, 1002, 1003, 1004, 1005, 1006, 1007, 1008, 1009, 1010],
    "customer": ["Priya", "Rohan", "Amit", "Divya", "Karan", "Neha", "Suresh", "Anjali", "Vikram", "Pooja"],
    "product": ["Laptop", "Mouse", "Keyboard", "Monitor", "Mouse", "Laptop", "Webcam", "Keyboard", "Monitor", "Laptop"],
    "category": ["Electronics", "Accessories", "Accessories", "Electronics", "Accessories",
                 "Electronics", "Accessories", "Accessories", "Electronics", "Electronics"],
    "quantity": [1, 4, 2, 1, 8, 2, 3, 1, 2, 1],
    "unit_price": [74999, 599, 1299, 16999, 599, 74999, 2499, 1299, 16999, 74999],
    "city": ["Delhi", "Mumbai", "Pune", "Delhi", "Bangalore", "Mumbai", "Pune", "Delhi", "Bangalore", "Mumbai"],
    "rating": [4.5, 3.8, 4.2, 4.9, 3.5, 4.7, 4.0, 3.9, 4.6, 4.8]
})

sales["revenue"] = sales["quantity"] * sales["unit_price"]
```

---

## How Boolean Indexing Works

When you write a condition on a DataFrame column, pandas evaluates it element-by-element and returns a Series of `True`/`False` values. Passing that Series back into `[]` keeps only the rows where the value is `True`.

```python
# Step 1: The condition returns a boolean Series
mask = sales["revenue"] > 50000

print(mask)
# Output:
# 0     True
# 1    False
# 2    False
# 3    False
# 4    False
# 5     True
# ...
# dtype: bool

# Step 2: Use the mask to select rows
high_value_orders = sales[mask]

# Or in one expression (more common):
high_value_orders = sales[sales["revenue"] > 50000]

print(high_value_orders[["customer", "product", "revenue"]])
# Output:
#   customer product  revenue
# 0    Priya  Laptop    74999
# 5     Neha  Laptop   149998
# 9    Pooja  Laptop    74999
```

Understanding this two-step process — condition creates mask, mask selects rows — prevents the confusion when combining conditions.

---

## Combining Conditions

Use `&` (AND), `|` (OR), `~` (NOT). Not `and`, `or`, `not`. That is a critical difference.

```python
# AND: both conditions must be true
electronics_high_value = sales[
    (sales["category"] == "Electronics") &
    (sales["revenue"] > 30000)
]

print(electronics_high_value[["customer", "product", "revenue"]])
# Output:
#   customer  product  revenue
# 0    Priya   Laptop    74999
# 5     Neha   Laptop   149998
# 8   Vikram  Monitor    33998
# 9    Pooja   Laptop    74999

# OR: at least one condition must be true
delhi_or_mumbai = sales[
    (sales["city"] == "Delhi") |
    (sales["city"] == "Mumbai")
]

# NOT: invert the condition
not_accessories = sales[~(sales["category"] == "Accessories")]
# Equivalent to:
not_accessories = sales[sales["category"] != "Electronics"]
```

> [!warning] Always wrap each condition in parentheses
> Without parentheses, Python's operator precedence can evaluate `&` before `==`, producing cryptic errors or wrong results.
>
> ```python
> # Wrong — raises TypeError or produces wrong result
> sales[sales["revenue"] > 50000 & sales["rating"] > 4.0]
>
> # Correct — each condition has its own parentheses
> sales[(sales["revenue"] > 50000) & (sales["rating"] > 4.0)]
> ```

> [!warning] Never use Python's `and`, `or`, `not` with pandas
> `and` expects a single boolean value. A pandas boolean Series is not a single boolean. This raises `ValueError: The truth value of a Series is ambiguous`.
>
> ```python
> # Wrong — ValueError
> sales[sales["revenue"] > 50000 and sales["rating"] > 4.0]
>
> # Correct
> sales[(sales["revenue"] > 50000) & (sales["rating"] > 4.0)]
> ```

---

## Filtering by Membership with .isin()

When you want to check if a value is in a list of options, `.isin()` is cleaner than chaining multiple `|` conditions.

```python
# Instead of this:
target_cities = sales[
    (sales["city"] == "Delhi") |
    (sales["city"] == "Mumbai") |
    (sales["city"] == "Pune")
]

# Write this:
target_cities = sales[sales["city"].isin(["Delhi", "Mumbai", "Pune"])]

print(target_cities[["customer", "city", "revenue"]])
# Output:
#   customer    city  revenue
# 0    Priya   Delhi    74999
# 1    Rohan  Mumbai     2396
# 2     Amit    Pune     2598
# 3    Divya   Delhi    16999
# 5     Neha  Mumbai   149998
# 7   Anjali   Delhi     1299
# 9    Pooja  Mumbai    74999

# Exclude a list of values using ~ (NOT):
not_metro = sales[~sales["city"].isin(["Delhi", "Mumbai", "Bangalore"])]
```

---

## .loc — The Right Tool for Simultaneous Row and Column Selection

You have seen `[]` for row filtering. `.loc` is the better tool when you also want to control which columns come back.

```python
# Filter rows by condition AND select specific columns in one expression
high_rated = sales.loc[
    sales["rating"] >= 4.5,
    ["customer", "product", "city", "revenue", "rating"]
]

print(high_rated)
# Output:
#   customer  product    city  revenue  rating
# 0    Priya   Laptop   Delhi    74999     4.5
# 3    Divya  Monitor   Delhi    16999     4.9
# 5     Neha   Laptop  Mumbai   149998     4.7
# 8   Vikram  Monitor Bangalore  33998     4.6
# 9    Pooja   Laptop  Mumbai    74999     4.8

# .loc with row label range and column names
# (rows 2 through 5 by label, two columns)
print(sales.loc[2:5, ["customer", "revenue"]])
# Output:
#   customer  revenue
# 2     Amit     2598
# 3    Divya    16999
# 4    Karan     4792
# 5     Neha   149998
```

> [!info] .loc slice end is inclusive
> `df.loc[2:5]` returns rows with labels 2, 3, 4, and 5 — four rows. This differs from Python list slicing and `.iloc`, where the end is excluded. Both behaviors are consistent within themselves; you need to know which tool you are using.

### .iloc — Position-Based Access

`.iloc` uses integer positions. Both row and column positions are zero-indexed, and slices exclude the endpoint.

```python
# First row, all columns
print(sales.iloc[0])

# Rows 0-2 (positions 0, 1, 2), columns 0-3 (positions 0, 1, 2, 3)
print(sales.iloc[0:3, 0:4])
# Output:
#    order_id customer   product     category
# 0      1001    Priya    Laptop  Electronics
# 1      1002    Rohan     Mouse  Accessories
# 2      1003     Amit  Keyboard  Accessories

# Last row
print(sales.iloc[-1])

# Every other row
print(sales.iloc[::2])
```

> [!warning] After filtering, .loc and .iloc diverge
> After filtering, the index labels stay as they were in the original DataFrame, but the positions reset to 0, 1, 2... relative to the filtered result.
>
> ```python
> high_value = sales[sales["revenue"] > 50000]
>
> # These labels still exist from the original index:
> print(high_value.loc[0])    # Priya's row (label 0)
> print(high_value.loc[5])    # Neha's row (label 5)
>
> # iloc uses position in the filtered result:
> print(high_value.iloc[0])   # Priya (first position in filtered result)
> print(high_value.iloc[1])   # Neha (second position in filtered result)
> print(high_value.iloc[2])   # Pooja (third position)
>
> # This raises KeyError — label 1 is not in the filtered result:
> # high_value.loc[1]
> ```
>
> When you need clean 0-based integer labels after filtering, reset the index:
>
> ```python
> high_value = high_value.reset_index(drop=True)
> ```

---

## .query() — Readable Filter Expressions

For complex filters, `.query()` can be more readable than nested bracket expressions. It accepts a string expression.

```python
# Equivalent to: sales[(sales["category"] == "Electronics") & (sales["rating"] >= 4.5)]
result = sales.query("category == 'Electronics' and rating >= 4.5")

print(result[["customer", "product", "revenue", "rating"]])
# Output:
#   customer  product  revenue  rating
# 0    Priya   Laptop    74999     4.5
# 3    Divya  Monitor    16999     4.9
# 5     Neha   Laptop   149998     4.7
# 9    Pooja   Laptop    74999     4.8

# Reference a Python variable using @
min_revenue = 30000
result = sales.query("revenue > @min_revenue and city == 'Mumbai'")

# Column names with spaces need backticks
# df.query("`unit price` > 1000")
```

> [!tip] When to use .query()
> Use `.query()` when you have many conditions and readability matters. Avoid it for very simple single-condition filters — the overhead of parsing a string is not worth it there. Also avoid it when column names have special characters (beyond spaces, which backticks handle).

---

## Filtering Text Columns with .str

```python
# Customers whose name starts with a specific letter
a_customers = sales[sales["customer"].str.startswith("A")]

# Products containing "key" (case-insensitive)
keyboard_orders = sales[sales["product"].str.contains("key", case=False)]

# Cities in uppercase for comparison
delhi_orders = sales[sales["city"].str.lower() == "delhi"]

print(delhi_orders[["customer", "product", "revenue"]])
# Output:
#   customer  product  revenue
# 0    Priya   Laptop    74999
# 3    Divya  Monitor    16999
# 7   Anjali Keyboard     1299
```

> [!warning] String filters on columns with NaN values
> If a column has missing values, `.str.contains()` and `.str.startswith()` return `NaN` for those rows, not `False`. This means those rows slip through your filter. Fix it with `na=False`:
>
> ```python
> # Safe version that treats NaN as non-matching
> result = sales[sales["customer"].str.contains("r", na=False)]
> ```

---

## Sorting

### Sort by one column

```python
# Ascending (default)
print(sales.sort_values("revenue").head(5))

# Descending
print(sales.sort_values("revenue", ascending=False).head(5))
# Output shows highest revenue orders first
```

### Sort by multiple columns

```python
# Sort by city A-Z, then revenue high-to-low within each city
sorted_sales = sales.sort_values(
    ["city", "revenue"],
    ascending=[True, False]
)

print(sorted_sales[["customer", "city", "revenue"]])
# Output:
#   customer       city  revenue
# 7   Anjali      Delhi     1299
# 3    Divya      Delhi    16999
# 0    Priya      Delhi    74999
# 4    Karan  Bangalore     4792
# 8   Vikram  Bangalore    33998
# ...
```

> [!warning] sort_values does not modify the original DataFrame
> `sales.sort_values("revenue")` returns a new sorted DataFrame. The `sales` variable is unchanged. Assign back if you need the sorted version to persist:
>
> ```python
> sales = sales.sort_values("revenue", ascending=False)
> # Or use ignore_index=True to reset the index in the same step:
> sales = sales.sort_values("revenue", ascending=False, ignore_index=True)
> ```

### Top and bottom records with .nlargest() and .nsmallest()

```python
# Top 3 orders by revenue
top_3 = sales.nlargest(3, "revenue")

print(top_3[["customer", "product", "revenue"]])
# Output:
#   customer product  revenue
# 5     Neha  Laptop   149998
# 0    Priya  Laptop    74999
# 9    Pooja  Laptop    74999

# Bottom 3 by rating
worst_rated = sales.nsmallest(3, "rating")

print(worst_rated[["customer", "product", "rating"]])
# Output:
#   customer   product  rating
# 4    Karan     Mouse     3.5
# 1    Rohan     Mouse     3.8
# 7   Anjali  Keyboard     3.9
```

> [!tip] nlargest/nsmallest vs sort + head
> `df.nlargest(5, "col")` is equivalent to `df.sort_values("col", ascending=False).head(5)` but is faster for large DataFrames because it does not sort the entire dataset. Prefer `nlargest`/`nsmallest` when you only need the top/bottom N rows.

---

## The SettingWithCopyWarning

This warning confuses every pandas beginner. It is worth understanding properly.

When you filter a DataFrame, pandas may return either a **view** (a window into the original data) or a **copy** (a separate object). The distinction is not always predictable. If you then try to modify the filtered result, pandas warns you that the modification might not affect the original DataFrame.

```python
# This looks fine but may trigger SettingWithCopyWarning
electronics = sales[sales["category"] == "Electronics"]
electronics["discount"] = 0.10   # SettingWithCopyWarning here
```

The safe pattern is to call `.copy()` explicitly when you intend to modify a filtered subset:

```python
electronics = sales[sales["category"] == "Electronics"].copy()
electronics["discount"] = 0.10   # No warning — we own this DataFrame
```

The other safe pattern is to use `.loc` to modify the original DataFrame directly:

```python
# Add a discount column only for Electronics rows, directly in the original
sales.loc[sales["category"] == "Electronics", "discount"] = 0.10
```

> [!warning] SettingWithCopyWarning is telling you something real
> It is not a style warning. It means your modification may be silently lost. When you see it, do not suppress it. Either use `.copy()` for a clean independent copy, or use `.loc` to modify the original. The chained assignment pattern `df[condition]["col"] = value` is always wrong.

---

## Method Chaining

Pandas methods return DataFrames, which means you can chain them. This produces clean, readable analysis pipelines.

```python
# Find the top 3 Electronics orders by revenue, keeping relevant columns
top_electronics = (
    sales
    .loc[sales["category"] == "Electronics"]
    .sort_values("revenue", ascending=False)
    .head(3)
    [["customer", "product", "city", "revenue", "rating"]]
    .reset_index(drop=True)
)

print(top_electronics)
# Output:
#   customer product    city  revenue  rating
# 0     Neha  Laptop  Mumbai   149998     4.7
# 1    Priya  Laptop   Delhi    74999     4.5
# 2    Pooja  Laptop  Mumbai    74999     4.8
```

Each method returns a new DataFrame, so the chain is safe. Wrap in parentheses so you can put each method on its own line for readability.

---

> [!success] Key takeaways
> - Boolean indexing works by creating a True/False mask, then using that mask to select rows.
> - Use `&`, `|`, `~` for combining conditions — never Python's `and`, `or`, `not`.
> - Always wrap each condition in parentheses when combining.
> - `.loc[row_condition, column_list]` is the cleaner pattern for filtering rows and selecting columns together.
> - `.iloc` uses integer positions; `.loc` uses labels. Know which you are using.
> - `.query()` is readable for complex conditions with many clauses.
> - Use `.copy()` when you intend to modify a filtered subset, to avoid `SettingWithCopyWarning`.
> - `.nlargest()` and `.nsmallest()` are faster than sort + head for top/bottom N.

---

[[02-reading-csv-excel|Previous: Reading CSV and Excel Files]] | [[04-basic-data-analysis|Next: Basic Data Analysis]]
