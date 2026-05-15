# Basic Data Analysis

Loading and filtering data is mechanical. Analysis is where you start asking questions and the data starts giving answers. This file covers the handful of pandas methods you will use in the first five minutes of every exploratory analysis — the commands that tell you what you are dealing with before you commit to any hypothesis.

## Learning Objectives

- Run a structured first-look inspection on any new dataset
- Generate and interpret summary statistics with `.describe()`
- Understand what `.info()` reveals that `.dtypes` does not
- Count frequencies with `.value_counts()` and understand its parameters
- Measure uniqueness with `.nunique()` and `.unique()`
- Detect and quantify missing values systematically
- Compute correlations with `.corr()` and interpret them carefully
- Run grouped aggregations as a preview of advanced `groupby` work
- Build calculated columns to answer derived questions

---

## Why First-Look Analysis Matters

The fastest way to waste time on a data science project is to start modeling before understanding the data. Every experienced analyst has the same reflex: load the file, run inspection commands, understand the shape and quality of the data before writing a single line of analysis code.

A dirty dataset produces wrong answers silently. A type mismatch produces an error you spend an hour debugging. Missing values produce biased aggregations. Knowing this before you start is the difference between confident analysis and guesswork.

---

## The Sample Dataset

```python
import pandas as pd
import numpy as np

# A realistic sales dataset with deliberate imperfections
transactions = pd.DataFrame({
    "order_id": [1001, 1002, 1003, 1004, 1005, 1006, 1007, 1008, 1009, 1010,
                 1011, 1012, 1013, 1014, 1015],
    "customer_name": ["Priya", "Rohan", "Amit", "Divya", "Karan", "Neha", "Suresh",
                      "Anjali", "Vikram", "Pooja", "Meera", "Raj", "Sunita", "Arjun", "Kavya"],
    "product": ["Laptop", "Mouse", "Keyboard", "Monitor", "Mouse", "Laptop", "Webcam",
                "Keyboard", "Monitor", "Laptop", "Mouse", "Webcam", "Laptop", "Keyboard", "Monitor"],
    "category": ["Electronics", "Accessories", "Accessories", "Electronics", "Accessories",
                 "Electronics", "Accessories", "Accessories", "Electronics", "Electronics",
                 "Accessories", "Accessories", "Electronics", "Accessories", "Electronics"],
    "quantity": [1, 4, 2, 1, 8, 2, 3, 1, 2, 1, 6, 2, 1, 3, 1],
    "unit_price": [74999, 599, 1299, 16999, 599, 74999, 2499, 1299, 16999, 74999,
                   599, 2499, 74999, 1299, 16999],
    "city": ["Delhi", "Mumbai", "Pune", "Delhi", "Bangalore", "Mumbai", "Pune", "Delhi",
             "Bangalore", "Mumbai", None, "Delhi", "Mumbai", "Pune", "Bangalore"],
    "customer_rating": [4.5, 3.8, 4.2, 4.9, 3.5, 4.7, 4.0, 3.9, 4.6, 4.8,
                        None, 4.1, 4.3, None, 4.7],
    "return_flag": [False, False, True, False, False, False, True, False, False, False,
                    False, False, True, False, False]
})

transactions["revenue"] = transactions["quantity"] * transactions["unit_price"]
```

---

## The First-Look Inspection Sequence

Run these commands in this order every time you load a new dataset. They answer a structured set of questions.

```python
# 1. How big is it?
print(transactions.shape)
# Output: (15, 10)
# 15 rows, 10 columns

# 2. What does the data look like?
print(transactions.head())

# 3. What are the column names and types?
print(transactions.dtypes)
# Output:
# order_id            int64
# customer_name      object
# product            object
# category           object
# quantity            int64
# unit_price          int64
# city               object
# customer_rating   float64
# return_flag          bool
# revenue             int64

# 4. Are there missing values?
print(transactions.isna().sum())
# Output:
# order_id          0
# customer_name     0
# ...
# city              1
# customer_rating   2
# ...

# 5. Full structural summary
transactions.info()
# Output:
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 15 entries, 0 to 14
# Data columns (total 10 columns):
# ...  Non-Null counts tell you exactly where missing data lives
```

> [!tip] Make this sequence a habit
> Put these five checks at the top of every new analysis notebook. The cost is 30 seconds. The benefit is catching problems before they produce wrong answers two hours later.

---

## .describe() — Numeric Summary Statistics

```python
print(transactions.describe())
# Output:
#          order_id   quantity  unit_price  customer_rating       revenue
# count   15.000000  15.000000   15.000000        13.000000     15.000000
# mean  1008.000000   2.533333   23352.333          4.261538  59426.933333
# std      4.472136   1.959591   30027.811          0.409832  75208.987...
# min   1001.000000   1.000000     599.000          3.500000    599.000000
# 25%   1004.500000   1.000000     599.000          3.900000   1299.000000
# 50%   1008.000000   2.000000    1299.000          4.300000  16999.000000
# 75%   1011.500000   3.000000   16999.000          4.700000  74999.000000
# max   1015.000000   8.000000   74999.000          4.900000  149998.000000
```

What each statistic tells you:

| Stat | What to look for |
|---|---|
| `count` | If it is less than the row count, that column has missing values |
| `mean` | The average — sensitive to outliers |
| `std` | How spread out values are — high std relative to mean means high variance |
| `min` / `max` | Catch obviously wrong values (negative prices, impossible ages) |
| `25%` / `75%` | The middle 50% of data — more outlier-resistant than mean |
| `50%` | The median — compare to mean; if they differ much, distribution is skewed |

Include non-numeric columns:

```python
# Include object and bool columns too
print(transactions.describe(include="all"))
```

Numeric columns only:

```python
print(transactions.describe(include=np.number))
```

---

## .info() — The Structural Snapshot

`.info()` is different from `.describe()`. It is not about statistics — it is about structure.

```python
transactions.info()
# Output:
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 15 entries, 0 to 14
# Data columns (total 10 columns):
#  #   Column           Non-Null Count  Dtype
# ---  ------           --------------  -----
#  0   order_id         15 non-null     int64
#  1   customer_name    15 non-null     object
#  2   product          15 non-null     object
#  3   category         15 non-null     object
#  4   quantity         15 non-null     int64
#  5   unit_price       15 non-null     int64
#  6   city             14 non-null     object     ← 1 missing
#  7   customer_rating  13 non-null     float64    ← 2 missing
#  8   return_flag      15 non-null     bool
#  9   revenue          15 non-null     int64
# dtypes: bool(1), float64(1), int64(4), object(4)
# memory usage: 1.4+ KB
```

The `Non-Null Count` column tells you immediately which columns have missing values without needing `isna().sum()`. For large datasets, `.info()` is the fastest diagnostic you have.

---

## Missing Values — Detection and Quantification

Detecting missing values is a prerequisite to deciding what to do about them. Do not handle what you have not measured.

```python
# Count missing values per column
print(transactions.isna().sum())
# Output:
# order_id           0
# customer_name      0
# product            0
# category           0
# quantity           0
# unit_price         0
# city               1
# customer_rating    2
# return_flag        0
# revenue            0

# Percentage missing per column
missing_pct = transactions.isna().mean() * 100
print(missing_pct.round(1))
# Output:
# city               6.7
# customer_rating   13.3
# (others are 0.0)

# Total missing values in the entire DataFrame
print(transactions.isna().sum().sum())
# Output: 3

# Which rows have any missing value?
rows_with_missing = transactions[transactions.isna().any(axis=1)]
print(rows_with_missing[["order_id", "customer_name", "city", "customer_rating"]])
# Output shows rows where city or customer_rating is NaN
```

> [!warning] Missing values affect aggregations silently
> Most pandas aggregations like `.mean()`, `.sum()`, and `.std()` skip `NaN` values by default. This means `transactions["customer_rating"].mean()` computes the mean of the 13 non-null values, not all 15. The count in the denominator is 13. This is usually what you want, but you need to know it is happening.
>
> ```python
> # Check: what is the mean including missing?
> # count shows 13, not 15
> print(transactions["customer_rating"].describe())
>
> # Force NaN to count (for diagnostics)
> print(transactions["customer_rating"].count())    # 13 (non-null count)
> print(len(transactions["customer_rating"]))       # 15 (total)
> ```

---

## .value_counts() — Frequency Distribution

`.value_counts()` is the fastest way to understand what is in a categorical column.

```python
# Count how many times each product appears
print(transactions["product"].value_counts())
# Output:
# product
# Laptop      4
# Keyboard    3
# Mouse       3
# Monitor     3
# Webcam      2
# Name: count, dtype: int64

# As percentages
print(transactions["product"].value_counts(normalize=True).round(3) * 100)
# Output:
# product
# Laptop      26.7
# Keyboard    20.0
# Mouse       20.0
# Monitor     20.0
# Webcam      13.3

# Include NaN values in the count (default is to exclude them)
print(transactions["city"].value_counts(dropna=False))
# Output shows NaN as a separate entry

# Sort by index (alphabetical) instead of by count
print(transactions["city"].value_counts().sort_index())
```

> [!warning] value_counts() excludes NaN by default
> If you need to see how many missing values a column has as part of the frequency table, pass `dropna=False`. Otherwise missing values disappear silently from the output.

---

## .nunique() and .unique() — Cardinality

```python
# How many distinct values does each column have?
print(transactions.nunique())
# Output:
# order_id           15
# customer_name      15
# product             5
# category            2
# quantity            7
# unit_price          5
# city                4    (one row has NaN, which is not counted)
# customer_rating    11
# return_flag         2
# revenue            11

# What are the distinct values in a specific column?
print(transactions["product"].unique())
# Output: ['Laptop' 'Mouse' 'Keyboard' 'Monitor' 'Webcam']

print(transactions["category"].unique())
# Output: ['Electronics' 'Accessories']

# Count of distinct values in a single column
print(transactions["city"].nunique())    # 4 (NaN not counted)
print(transactions["city"].nunique(dropna=False))   # 5 (NaN counted as one value)
```

> [!tip] High nunique on an object column is a smell
> If `customer_id` has 15 unique values in 15 rows — expected. If `city` has 14 unique values in 15 rows, something is wrong. Inconsistent capitalization ("Delhi" vs "delhi" vs "DELHI") inflates cardinality artificially. Check `.unique()` when cardinality seems off.

---

## Basic Aggregations

```python
# Single aggregations
print(transactions["revenue"].sum())        # total revenue
print(transactions["revenue"].mean())       # average order value
print(transactions["revenue"].median())     # median order value
print(transactions["revenue"].max())        # largest single order
print(transactions["revenue"].min())        # smallest single order
print(transactions["quantity"].std())       # spread in order sizes

# Multiple aggregations in one call
print(transactions["revenue"].agg(["sum", "mean", "median", "std", "min", "max"]))
# Output:
# sum       891404.000000
# mean       59426.933333
# median     16999.000000
# std        75208.987...
# min          599.000000
# max       149998.000000
```

---

## Correlation — .corr()

Correlation measures how strongly two numeric variables move together. Values range from -1 (perfect negative) to +1 (perfect positive). Values near 0 mean no linear relationship.

```python
# Correlation matrix of all numeric columns
corr_matrix = transactions[["quantity", "unit_price", "customer_rating", "revenue"]].corr()

print(corr_matrix.round(2))
# Output:
#                  quantity  unit_price  customer_rating  revenue
# quantity             1.00       -0.45             0.07     0.46
# unit_price          -0.45        1.00             0.19     0.94
# customer_rating      0.07        0.19             1.00     0.23
# revenue              0.46        0.94             0.23     1.00
```

Interpreting this:
- `unit_price` and `revenue` correlate at 0.94 — very strong positive relationship. Makes sense: higher price means higher revenue.
- `quantity` and `unit_price` correlate at -0.45 — moderate negative relationship. People buy more of cheaper items.
- `customer_rating` and `revenue` correlate at 0.23 — weak relationship. Rating does not strongly predict order size.

> [!warning] Correlation is not causation
> Two variables can correlate strongly for many reasons: one causes the other, both are caused by a third variable, or it is coincidence. Never conclude causation from correlation alone.

> [!info] Pearson vs. Spearman
> `.corr()` computes Pearson correlation by default — it measures linear relationships. If your data has outliers or non-linear patterns, Spearman correlation (`.corr(method="spearman")`) is more robust.

---

## Creating Analysis Columns

Derived columns let you answer questions that the raw data cannot answer directly.

```python
# Revenue per unit
transactions["revenue_per_unit"] = transactions["revenue"] / transactions["quantity"]

# Order value tier
transactions["order_tier"] = pd.cut(
    transactions["revenue"],
    bins=[0, 5000, 50000, float("inf")],
    labels=["low", "medium", "high"]
)

print(transactions["order_tier"].value_counts())
# Output:
# order_tier
# low       7
# medium    4
# high      4

# Binary flag using np.where
transactions["is_high_value"] = np.where(transactions["revenue"] >= 50000, True, False)
```

---

## Grouped Analysis — A Preview

Full `groupby` is covered in Day 03. But even at the basics level, grouped aggregation is essential.

```python
# Total revenue by category
category_revenue = transactions.groupby("category")["revenue"].sum()
print(category_revenue)
# Output:
# category
# Accessories    21087
# Electronics   870317
# Name: revenue, dtype: int64

# Average rating by product
product_rating = transactions.groupby("product")["customer_rating"].mean().round(2)
print(product_rating)
# Output:
# product
# Keyboard    4.07
# Laptop      4.58
# Monitor     4.73
# Mouse       3.77
# Webcam      4.05

# Multiple aggregations by city
city_summary = transactions.groupby("city").agg(
    total_revenue=("revenue", "sum"),
    order_count=("order_id", "count"),
    avg_rating=("customer_rating", "mean")
).round(2)

print(city_summary)
# Output:
#             total_revenue  order_count  avg_rating
# city
# Bangalore          85999            3        4.27
# Delhi              93296            4        4.37
# Mumbai            562994            4        4.45
# Pune               74997            3        4.30
```

---

## A Complete First-Look Workflow

```python
import pandas as pd
import numpy as np

# Assume we just loaded a dataset
df = transactions.copy()

print("=" * 50)
print("DATASET OVERVIEW")
print("=" * 50)
print(f"Shape: {df.shape[0]} rows, {df.shape[1]} columns")
print()

print("Column types:")
print(df.dtypes)
print()

print("Missing values:")
missing = df.isna().sum()
print(missing[missing > 0])
print()

print("Numeric summary:")
print(df.describe().round(2))
print()

print("Category distributions:")
for col in df.select_dtypes(include="object").columns:
    print(f"\n{col} ({df[col].nunique()} unique values):")
    print(df[col].value_counts().head(5))
```

---

> [!success] Key takeaways
> - `.describe()` gives numeric distribution statistics. `count` less than total rows means missing values.
> - `.info()` gives structural summary: types, non-null counts, and memory in one command.
> - `.value_counts()` is the fastest way to understand categorical columns. Use `dropna=False` to see NaN counts.
> - `.isna().sum()` detects missing values per column. `.isna().mean() * 100` gives percentages.
> - `.corr()` measures linear relationships between numeric columns. Pearson by default.
> - Never start analysis without running the inspection sequence first.

---

[[03-filtering-and-sorting|Previous: Filtering and Sorting]] | [[05-practice-exercises|Next: Practice Exercises]]
