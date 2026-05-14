# 📊 04 — Basic Data Analysis
## Turning DataFrames into Insights

> [!info] Goal
> Learn the first analysis commands to run after loading a dataset.

---

## Sample Dataset

```python
import pandas as pd

sales = pd.DataFrame({
    "order_id": [101, 102, 103, 104, 105, 106],
    "product": ["Laptop", "Mouse", "Laptop", "Keyboard", "Mouse", "Monitor"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories", "Accessories", "Electronics"],
    "quantity": [2, 5, 1, 3, 10, 2],
    "price": [75000, 600, 75000, 1200, 600, 15000],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune", "Mumbai", "Bangalore"]
})

sales["revenue"] = sales["quantity"] * sales["price"]
```

---

## First Look at Data

```python
sales.head()
sales.tail()
sales.shape
sales.info()
sales.dtypes
sales.columns
```

These commands answer:

- How many rows and columns are there?
- What are the column names?
- What data types are present?
- Are there missing values?

---

## Summary Statistics

```python
sales.describe()
```

This gives count, mean, standard deviation, min, quartiles, and max for numeric columns.

For all columns:

```python
sales.describe(include="all")
```

---

## Selecting Numeric Columns

```python
numeric_sales = sales.select_dtypes(include="number")
```

Selecting text/object columns:

```python
text_sales = sales.select_dtypes(include="object")
```

---

## Basic Aggregations

```python
sales["revenue"].sum()
sales["revenue"].mean()
sales["revenue"].min()
sales["revenue"].max()
sales["quantity"].median()
sales["quantity"].std()
```

### Multiple Aggregations

```python
sales["revenue"].agg(["sum", "mean", "min", "max"])
```

---

## Counting Values

### Unique Values

```python
sales["product"].unique()
sales["product"].nunique()
```

### Frequency Counts

```python
sales["product"].value_counts()
sales["category"].value_counts()
sales["city"].value_counts()
```

### Percentages

```python
sales["category"].value_counts(normalize=True) * 100
```

---

## Missing Value Checks

```python
sales.isna()
sales.isna().sum()
sales.isna().mean() * 100
```

Missing value handling is covered more deeply in Day 03, but you should always check for missing data early.

---

## Creating New Analysis Columns

```python
sales["revenue"] = sales["quantity"] * sales["price"]
```

Create a label column:

```python
sales["order_size"] = "Small"
sales.loc[sales["revenue"] >= 50000, "order_size"] = "Large"
```

Using `np.where()`:

```python
import numpy as np

sales["high_value"] = np.where(sales["revenue"] >= 50000, "Yes", "No")
```

---

## Simple Business Questions

### Total Revenue

```python
total_revenue = sales["revenue"].sum()
```

### Highest Revenue Order

```python
sales.nlargest(1, "revenue")
```

### Most Common Product

```python
sales["product"].value_counts().head(1)
```

### Electronics Orders Only

```python
electronics = sales[sales["category"] == "Electronics"]
```

### Revenue from Accessories

```python
accessories_revenue = sales.loc[
    sales["category"] == "Accessories",
    "revenue"
].sum()
```

---

## Intro to Grouped Analysis

Advanced `groupby()` is covered in Day 03, but the basic idea is important.

```python
sales.groupby("category")["revenue"].sum()
sales.groupby("city")["revenue"].mean()
sales.groupby("product")["quantity"].sum()
```

Multiple aggregations:

```python
sales.groupby("category")["revenue"].agg(["sum", "mean", "count"])
```

---

## Sorting Analysis Results

```python
sales.groupby("product")["revenue"].sum().sort_values(ascending=False)
```

This answers: which product generated the most revenue?

---

## Basic Analysis Workflow

```python
import pandas as pd

df = pd.read_csv("sales.csv")

print(df.head())
print(df.shape)
print(df.info())
print(df.isna().sum())
print(df.describe())

df["revenue"] = df["quantity"] * df["price"]

print(df["revenue"].sum())
print(df["product"].value_counts())
print(df.groupby("category")["revenue"].sum())
```

---

## Common Mistakes

### Analyzing Before Inspecting

Do not jump straight to calculations. First check shape, types, column names, and missing values.

### Treating Numbers Stored as Text as Numeric

```python
df["salary"].mean()
```

This may fail if `salary` is stored as strings.

Fix:

```python
df["salary"] = pd.to_numeric(df["salary"], errors="coerce")
```

### Forgetting That `value_counts()` Ignores Missing Values by Default

```python
df["city"].value_counts(dropna=False)
```

---

## Practice

Using the `sales` DataFrame:

- print dataset shape
- print summary statistics
- find total revenue
- find average revenue per order
- count orders by city
- count orders by product
- find the top 3 orders by revenue
- calculate total revenue by category
- calculate total quantity sold by product

---

## ✅ Key Takeaways

- `describe()` gives quick numeric summaries.
- `value_counts()` is the fastest way to inspect categories.
- `isna().sum()` checks missing values.
- New columns are created using normal assignment.
- Basic `groupby()` helps answer category-level questions.
- A good analysis starts with inspection before calculation.

---

## 🔗 What's Next?

➡️ [[05-practice-exercises]] — Practice the full Pandas basics workflow
