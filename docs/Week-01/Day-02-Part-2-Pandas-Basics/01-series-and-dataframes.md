# Series and DataFrames

Before you can analyze data in pandas, you need to understand what pandas actually stores. The two core objects — `Series` and `DataFrame` — are not just containers. They carry labels, types, and alignment semantics that control everything else you do. Misunderstand them and you will fight the library. Understand them and the library works for you.

## Learning Objectives

- Explain the relationship between a `Series` and a `DataFrame`
- Understand the `Index` as a first-class object, not just row numbers
- Create DataFrames from dictionaries and lists of dicts
- Inspect shape, dtypes, memory usage, and summary info
- Select one column (returns `Series`) vs. multiple columns (returns `DataFrame`) — and know why this matters
- Use `.loc` for label-based access and `.iloc` for position-based access
- Add, modify, rename, and drop columns correctly

---

## What Pandas Is and Why It Exists

NumPy is great at fast numerical operations on homogeneous arrays. Real-world tabular data is not homogeneous — a customer table has names (strings), ages (integers), and signup dates (datetimes) in the same row. NumPy cannot label columns or rows. Pandas solves both problems: it handles mixed types and it makes labels a first-class concept.

```python
import pandas as pd
import numpy as np

print(pd.__version__)
```

---

## The Series: A Labeled 1D Array

A `Series` is a one-dimensional array where every value has a label. That label is called the **index**.

```python
scores = pd.Series([88, 92, 75, 95, 81])

print(scores)
# Output:
# 0    88
# 1    92
# 2    75
# 3    95
# 4    81
# dtype: int64
```

The left column is the index. The right column is the data. Both are first-class — you can access either independently.

```python
print(scores.values)   # array([88, 92, 75, 95, 81])
print(scores.index)    # RangeIndex(start=0, stop=5, step=1)
print(scores.dtype)    # int64
```

### Custom Index Labels

The index does not have to be integers. This is where Series becomes powerful.

```python
exam_scores = pd.Series(
    [88, 92, 75, 95],
    index=["Priya", "Rohan", "Amit", "Divya"]
)

print(exam_scores)
# Output:
# Priya    88
# Rohan    92
# Amit     75
# Divya    95
# dtype: int64

print(exam_scores["Priya"])   # 88
print(exam_scores["Rohan":"Amit"])   # label-based slice, both ends included
```

> [!info] Series is the building block
> Every column in a DataFrame is a Series. Every row you extract from a DataFrame is a Series. The Series is not just a column — it is the fundamental data unit pandas works with internally.

### Vectorized Operations on Series

Series supports the same vectorized operations as NumPy arrays. The index is preserved.

```python
adjusted_scores = exam_scores + 5

print(adjusted_scores)
# Output:
# Priya    93
# Rohan    97
# Amit     80
# Divya    100
# dtype: int64
```

---

## The DataFrame: A Labeled 2D Table

A `DataFrame` is a collection of `Series` that all share the same index. Think of it as a spreadsheet or SQL table — rows have labels (the index), columns have labels (column names), and every column is a `Series`.

```python
employees = pd.DataFrame({
    "name": ["Priya", "Rohan", "Amit", "Divya", "Karan"],
    "department": ["Sales", "Engineering", "Engineering", "HR", "Sales"],
    "salary": [52000, 88000, 79000, 46000, 67000],
    "years_exp": [2, 6, 4, 1, 3]
})

print(employees)
# Output:
#     name   department  salary  years_exp
# 0  Priya        Sales   52000          2
# 1  Rohan  Engineering   88000          6
# 2   Amit  Engineering   79000          4
# 3  Divya           HR   46000          1
# 4  Karan        Sales   67000          3
```

### Creating DataFrames from Different Sources

**From a dictionary of lists** — most common when building test data:

```python
products = pd.DataFrame({
    "product_name": ["Laptop", "Mouse", "Keyboard", "Monitor", "Webcam"],
    "category": ["Electronics", "Accessories", "Accessories", "Electronics", "Accessories"],
    "unit_price": [74999, 599, 1299, 16999, 2499],
    "units_in_stock": [8, 120, 45, 15, 30]
})
```

**From a list of dicts** — common when data comes from APIs or JSON:

```python
records = [
    {"order_id": 1001, "customer": "Neha", "amount": 5200, "status": "shipped"},
    {"order_id": 1002, "customer": "Suresh", "amount": 12400, "status": "pending"},
    {"order_id": 1003, "customer": "Anjali", "amount": 890, "status": "shipped"},
]

orders = pd.DataFrame(records)
print(orders)
# Output:
#    order_id customer  amount   status
# 0      1001     Neha    5200  shipped
# 1      1002   Suresh   12400  pending
# 2      1003   Anjali     890  shipped
```

---

## The Index: More Than Row Numbers

The Index is the most underestimated concept in pandas. Students treat it as background noise — row numbers that pandas auto-assigns. It is actually a first-class object that enables alignment, fast lookup, and time series operations.

```python
print(employees.index)
# RangeIndex(start=0, stop=5, step=1)

print(employees.columns)
# Index(['name', 'department', 'salary', 'years_exp'], dtype='object')
```

You can set a meaningful index to make row lookup faster and more readable:

```python
employees_indexed = employees.set_index("name")

print(employees_indexed)
# Output:
#        department  salary  years_exp
# name
# Priya       Sales   52000          2
# Rohan Engineering   88000          6
# Amit  Engineering   79000          4
# Divya          HR   46000          1
# Karan       Sales   67000          3

print(employees_indexed.loc["Rohan"])
# Output:
# department    Engineering
# salary              88000
# years_exp               6
# Name: Rohan, dtype: object
```

> [!warning] Index alignment can bite you silently
> When you add two Series or DataFrames, pandas aligns by index first. If the indexes do not match, you get `NaN` where labels differ. This is powerful but surprising if you do not know it is happening.
>
> ```python
> a = pd.Series([1, 2, 3], index=["x", "y", "z"])
> b = pd.Series([10, 20, 30], index=["x", "z", "w"])
>
> print(a + b)
> # Output:
> # w     NaN
> # x    11.0
> # y     NaN
> # z    23.0
> # dtype: float64
> ```

To reset back to default integer index:

```python
employees_reset = employees_indexed.reset_index()
```

---

## Inspecting a DataFrame

The first thing to do after creating or loading a DataFrame is inspect it. Never skip this step.

```python
sales_data = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105, 106, 107, 108],
    "product": ["Laptop", "Mouse", "Keyboard", "Monitor", "Mouse", "Laptop", "Webcam", "Keyboard"],
    "category": ["Electronics", "Accessories", "Accessories", "Electronics",
                 "Accessories", "Electronics", "Accessories", "Accessories"],
    "quantity": [1, 4, 2, 1, 8, 2, 3, 1],
    "unit_price": [74999, 599, 1299, 16999, 599, 74999, 2499, 1299],
    "city": ["Delhi", "Mumbai", "Pune", "Delhi", "Bangalore", "Mumbai", "Pune", "Delhi"]
})

print(sales_data.head())       # first 5 rows (or pass n: head(3))
print(sales_data.tail(3))      # last 3 rows
print(sales_data.shape)        # (8, 6) → 8 rows, 6 columns
print(sales_data.columns.tolist())  # list of column names
print(sales_data.dtypes)       # dtype of each column
```

`.info()` is the single most useful first command — it shows dtypes, non-null counts, and memory in one shot:

```python
sales_data.info()
# Output:
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 8 entries, 0 to 7
# Data columns (total 6 columns):
#  #   Column      Non-Null Count  Dtype
# ---  ------      --------------  -----
#  0   order_id    8 non-null      int64
#  1   product     8 non-null      object
#  2   category    8 non-null      object
#  3   quantity    8 non-null      int64
#  4   unit_price  8 non-null      int64
#  5   city        8 non-null      object
# dtypes: int64(3), object(3)
# memory usage: 512.0+ bytes
```

> [!tip] Check memory on large DataFrames
> For large datasets, `.memory_usage(deep=True)` shows actual memory consumed per column. Object (string) columns consume far more than numeric ones. Converting to the right dtype can cut memory use dramatically.
>
> ```python
> sales_data.memory_usage(deep=True)
> # Output:
> # Index         128
> # order_id       64
> # product       570
> # category      536
> # quantity       64
> # unit_price     64
> # city          501
> # dtype: int64
> ```

---

## Data Types (dtypes)

Pandas assigns a dtype to every column. Getting dtypes right matters because wrong types cause silent errors in analysis.

| Pandas dtype | What it stores | NumPy equivalent |
|---|---|---|
| `int64` | Integers | `np.int64` |
| `float64` | Floating point | `np.float64` |
| `object` | Strings (and mixed types) | N/A |
| `bool` | True/False | `np.bool_` |
| `datetime64[ns]` | Timestamps | `np.datetime64` |
| `category` | Low-cardinality strings | N/A |

> [!warning] "object" dtype does not mean string
> When pandas shows `object` dtype, it usually means strings, but it can mean any Python object. A column with mixed types (integers and strings in the same column) also shows `object`. If analysis on an `object` column fails, check `.unique()` first to see what is actually there.

Cast dtypes when needed:

```python
# String column that should be numeric
sales_data["unit_price"] = pd.to_numeric(sales_data["unit_price"], errors="coerce")

# String column that should be category (saves memory)
sales_data["category"] = sales_data["category"].astype("category")

# String column that should be datetime
# df["order_date"] = pd.to_datetime(df["order_date"])
```

---

## Selecting Columns

This is where a very common mistake lives. Single brackets and double brackets return different types.

### Single column → returns a Series

```python
product_col = sales_data["product"]

print(type(product_col))   # <class 'pandas.core.series.Series'>
print(product_col.head(3))
# Output:
# 0      Laptop
# 1       Mouse
# 2    Keyboard
# Name: product, dtype: object
```

### Multiple columns → returns a DataFrame

```python
subset = sales_data[["product", "quantity", "unit_price"]]

print(type(subset))   # <class 'pandas.core.frame.DataFrame'>
print(subset.head(3))
# Output:
#     product  quantity  unit_price
# 0    Laptop         1       74999
# 1     Mouse         4         599
# 2  Keyboard         2        1299
```

> [!warning] The double-bracket trap
> `df["product", "city"]` is wrong — it passes a tuple as the key and raises a `KeyError`. You need `df[["product", "city"]]` with the list inside the brackets.

The reason this matters: many methods behave differently on a Series vs. a DataFrame. Knowing which you have prevents puzzling errors downstream.

---

## .loc and .iloc — Label vs. Position

These two accessors are the source of more beginner confusion than almost anything else. Here is the mental model:

- `.loc` — think **L**abel. You give it row labels and column labels.
- `.iloc` — think **I**nteger. You give it row positions and column positions.

```python
# .loc: by label
print(employees.loc[0])          # row with label 0
print(employees.loc[0:2])        # rows with labels 0, 1, 2 (BOTH ENDS INCLUSIVE)
print(employees.loc[0, "salary"])          # row label 0, column named "salary"
print(employees.loc[0:2, ["name", "salary"]])  # rows 0-2, two columns

# .iloc: by position
print(employees.iloc[0])          # first row (position 0)
print(employees.iloc[0:2])        # first two rows (end position EXCLUDED)
print(employees.iloc[0, 2])       # row 0, column at position 2
print(employees.iloc[0:3, 0:2])   # first 3 rows, first 2 columns
```

> [!warning] .loc slicing includes the endpoint; .iloc does not
> `df.loc[0:3]` returns 4 rows (0, 1, 2, 3). `df.iloc[0:3]` returns 3 rows (0, 1, 2). This mirrors Python list slicing for `.iloc` but not for `.loc`. Get this wrong and you select one row too many without noticing.

When is this distinction critical? When you reset the index, filter rows, or sort — the integer index labels and the positional order can diverge.

```python
filtered = employees[employees["salary"] > 60000]
print(filtered)
# Output:
#     name   department  salary  years_exp
# 1  Rohan  Engineering   88000          6
# 2   Amit  Engineering   79000          4
# 4  Karan        Sales   67000          3

# .loc uses the label (original index):
print(filtered.loc[1])     # works, returns Rohan's row

# .iloc uses the position (0 = first remaining row):
print(filtered.iloc[0])    # returns Rohan's row too, but via position
print(filtered.iloc[1])    # returns Amit's row (second position), not label 2
```

> [!tip] Use .loc for most work
> When in doubt, use `.loc`. It is explicit about what you want. Reserve `.iloc` for when you genuinely need positional access — iterating in a loop, slicing by position, or integrating with numpy operations.

---

## Adding, Modifying, and Dropping Columns

### Add a calculated column

```python
sales_data["revenue"] = sales_data["quantity"] * sales_data["unit_price"]

print(sales_data[["product", "quantity", "unit_price", "revenue"]].head())
# Output:
#     product  quantity  unit_price  revenue
# 0    Laptop         1       74999    74999
# 1     Mouse         4         599     2396
# 2  Keyboard         2        1299     2598
# 3   Monitor         1       16999    16999
# 4     Mouse         8         599     4792
```

### Modify an existing column

```python
# Apply a discount
sales_data["discounted_price"] = sales_data["unit_price"] * 0.90
```

### Add a conditional column with np.where

```python
sales_data["order_size"] = np.where(
    sales_data["revenue"] >= 50000, "large", "small"
)
```

### Drop columns

```python
sales_data = sales_data.drop(columns=["discounted_price"])
```

> [!warning] Most pandas methods do not modify in place
> `df.drop(columns=["col"])` returns a new DataFrame. The original `df` is unchanged. Either assign back — `df = df.drop(...)` — or use `inplace=True`. Most experienced users avoid `inplace=True` because it can cause issues with method chaining and `SettingWithCopyWarning`. Assigning back is cleaner.

### Rename columns

```python
employees = employees.rename(columns={
    "years_exp": "years_experience",
    "department": "dept"
})
```

---

## Series vs. DataFrame: Summary

| Feature | Series | DataFrame |
|---|---|---|
| Dimensions | 1D | 2D |
| Has an index | Yes | Yes (for rows) |
| Has column names | No | Yes |
| Select from DataFrame | `df["col"]` | `df[["col1", "col2"]]` |
| Typical use | Single column of data | Full dataset |
| Operations return | Usually Series | Usually DataFrame |

---

> [!success] Key takeaways
> - A `Series` is a labeled 1D array. A `DataFrame` is a table of Series sharing the same index.
> - The Index is not just row numbers — it is a first-class object that enables alignment and fast lookup.
> - `df["col"]` returns a `Series`. `df[["col1", "col2"]]` returns a `DataFrame`. This difference matters.
> - `.loc` uses labels (both ends inclusive for slices). `.iloc` uses integer positions (end excluded).
> - Most methods return a new object. Assign the result back or nothing changes.

---

[[00-agenda|Previous: Agenda]] | [[02-reading-csv-excel|Next: Reading CSV and Excel Files]]
