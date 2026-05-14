# 🧱 01 — Series and DataFrames
## The Two Core Pandas Objects

> [!info] Goal
> Learn the difference between a `Series` and a `DataFrame`, then practice creating, inspecting, and selecting data.

---

## What is Pandas?

**Pandas** is a Python library for working with structured data.

It is especially useful for:

- reading CSV and Excel files
- cleaning messy datasets
- filtering rows
- selecting columns
- grouping and summarizing data
- preparing data for visualization and machine learning

The standard import is:

```python
import pandas as pd
```

---

## Series

A **Series** is a one-dimensional labeled array.

Think of it as a single column.

```python
import pandas as pd

scores = pd.Series([85, 90, 78, 92])

print(scores)
```

Output:

```text
0    85
1    90
2    78
3    92
dtype: int64
```

The left side is the **index**. The right side is the **value**.

### Series with Custom Index

```python
scores = pd.Series(
    [85, 90, 78],
    index=["Alice", "Bob", "Charlie"]
)

print(scores)
print(scores["Alice"])
```

Output:

```text
Alice      85
Bob        90
Charlie    78
dtype: int64
85
```

---

## DataFrame

A **DataFrame** is a two-dimensional table with rows and columns.

Think of it as an Excel sheet or SQL table.

```python
students = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "age": [24, 27, 22],
    "score": [85, 90, 78]
})

print(students)
```

Output:

```text
      name  age  score
0    Alice   24     85
1      Bob   27     90
2  Charlie   22     78
```

---

## Creating DataFrames

### From a Dictionary of Lists

```python
df = pd.DataFrame({
    "product": ["Laptop", "Mouse", "Keyboard"],
    "price": [75000, 600, 1200],
    "quantity": [2, 5, 3]
})
```

This is the most common beginner-friendly way.

### From a List of Dictionaries

```python
records = [
    {"name": "Alice", "department": "Sales", "salary": 50000},
    {"name": "Bob", "department": "Tech", "salary": 80000},
    {"name": "Charlie", "department": "HR", "salary": 45000},
]

df = pd.DataFrame(records)
```

This format is common when data comes from APIs or JSON files.

---

## Inspecting a DataFrame

```python
print(df.head())       # first 5 rows
print(df.tail())       # last 5 rows
print(df.shape)        # rows, columns
print(df.columns)      # column names
print(df.index)        # row index
print(df.dtypes)       # column data types
print(df.info())       # compact summary
```

### Example

```python
employees = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie", "Diana"],
    "department": ["Sales", "Tech", "Tech", "HR"],
    "salary": [50000, 80000, 75000, 45000],
    "experience": [2, 5, 4, 1]
})

print(employees.head(2))
print(employees.shape)
print(employees.dtypes)
```

---

## Selecting Columns

### Single Column

```python
employees["name"]
```

This returns a `Series`.

### Multiple Columns

```python
employees[["name", "salary"]]
```

This returns a `DataFrame`.

> [!important] Double Brackets
> Use double brackets when selecting multiple columns: `df[["col1", "col2"]]`.

---

## Selecting Rows

### By Label with `.loc`

```python
employees.loc[0]
employees.loc[0:2]
employees.loc[0:2, ["name", "salary"]]
```

`.loc` uses labels. With integer labels, the end label is included.

### By Position with `.iloc`

```python
employees.iloc[0]
employees.iloc[0:2]
employees.iloc[0:2, 0:2]
```

`.iloc` uses integer position. The end position is excluded, like normal Python slicing.

---

## Add, Modify, and Drop Columns

### Add a Column

```python
employees["bonus"] = employees["salary"] * 0.10
```

### Modify a Column

```python
employees["salary"] = employees["salary"] + 5000
```

### Drop a Column

```python
employees = employees.drop(columns=["bonus"])
```

---

## Rename Columns

```python
employees = employees.rename(columns={
    "name": "employee_name",
    "salary": "monthly_salary"
})
```

---

## Pandas vs NumPy

| Feature | NumPy | Pandas |
|---------|-------|--------|
| Main object | `ndarray` | `Series`, `DataFrame` |
| Best for | Numerical arrays | Tabular datasets |
| Labels | No column labels | Row and column labels |
| Mixed types | Limited | Common |
| Missing values | Supported but lower-level | Built-in workflows |

---

## Common Beginner Mistakes

### Selecting Multiple Columns Incorrectly

```python
# Wrong
df["name", "salary"]

# Correct
df[["name", "salary"]]
```

### Confusing `.loc` and `.iloc`

```python
df.loc[0]    # label 0
df.iloc[0]   # first row by position
```

### Forgetting Assignment

```python
df.drop(columns=["salary"])
```

The line above returns a new DataFrame but does not change `df`.

Use:

```python
df = df.drop(columns=["salary"])
```

---

## Practice

Create this DataFrame:

```python
products = pd.DataFrame({
    "product": ["Laptop", "Mouse", "Keyboard", "Monitor"],
    "category": ["Electronics", "Accessories", "Accessories", "Electronics"],
    "price": [75000, 600, 1200, 15000],
    "stock": [5, 50, 30, 10]
})
```

Tasks:

- print the first 2 rows
- print the shape
- select only the `product` column
- select `product` and `price`
- add a new column called `inventory_value`
- rename `stock` to `units_available`

---

## ✅ Key Takeaways

- A `Series` is one labeled column.
- A `DataFrame` is a labeled table.
- Use `head()`, `shape`, `columns`, `dtypes`, and `info()` to inspect data.
- Use `df["col"]` for one column.
- Use `df[["col1", "col2"]]` for multiple columns.
- Use `.loc` for labels and `.iloc` for positions.
- Most Pandas methods return a new object unless you assign the result.

---

## 🔗 What's Next?

➡️ [[02-reading-csv-excel]] — Load real files into Pandas
