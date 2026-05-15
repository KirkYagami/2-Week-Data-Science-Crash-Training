# Apply Functions — Custom Logic in Pandas

Pandas gives you vectorized operations, string accessors, date accessors, and a dozen aggregation methods. Use all of them first. `.apply()` is the escape hatch you reach for when nothing else fits — and knowing when to reach for it (versus when you are about to write slow, unreadable code) is what separates a beginner from someone who works with data professionally.

The most expensive consulting engagement I know of involved a data pipeline that processed 5 million rows with `apply(axis=1)` on 12 columns. Runtime: 45 minutes per run. The fix was vectorizing three of those operations. Runtime dropped to 4 minutes. Same result, same code structure, just better tool choices.

## Learning Objectives

By the end of this section you will be able to:

- Describe the performance hierarchy: vectorized > `.map()` > `.apply()` on a Series > `.apply(axis=1)` on a DataFrame
- Use `.map()` for element-wise value substitution on a Series
- Use `.apply()` on a Series for complex single-column transformations
- Use `.apply(axis=1)` for logic that genuinely requires multiple columns in a single row
- Use `np.select()` and `pd.cut()` as vectorized alternatives to most conditional apply calls
- Use `.str` and `.dt` accessors as the correct tools for text and date operations

---

## The Performance Hierarchy

Before writing any transformation, ask which level you are at:

```
1. Vectorized Pandas/NumPy operations   ← always try this first
2. .map() on a Series                   ← for lookup/replacement in one column
3. .apply() on a Series                 ← for complex logic in one column
4. .apply(axis=1) on a DataFrame        ← for logic needing multiple columns per row
```

Each level down is slower. Level 4 can be 10–100x slower than level 1 on large DataFrames, because it abandons C-optimized array operations and iterates row by row in Python.

---

## Sample Dataset

```python
import pandas as pd
import numpy as np

employees = pd.DataFrame({
    "emp_id": [1001, 1002, 1003, 1004, 1005, 1006],
    "name": [" alice sharma ", "BOB MENON", "Charlie Rao", "diana nair", "EVAN PATEL", " fatima khan "],
    "department": ["Sales", "Engineering", "Engineering", "HR", "Sales", "Engineering"],
    "salary": [52000, 95000, 88000, 44000, 61000, 110000],
    "experience_years": [2, 7, 5, 1, 3, 9],
    "performance_rating": [4.1, 4.9, 4.4, 3.7, 4.2, 4.8],
    "region": ["North", "South", "South", "North", "West", "South"]
})
```

---

## Level 1: Vectorized Operations — Always Try This First

Arithmetic between columns and scalar comparisons are vectorized by default. They operate on the entire array at once using C-level loops.

```python
# Revenue calculation — vectorized, fast, readable
employees["annual_bonus_pool"] = employees["salary"] * 0.12

# Boolean mask — vectorized
employees["is_senior"] = employees["experience_years"] >= 5

# Multiple conditions — use & and | with parentheses
employees["is_high_earner"] = (
    (employees["salary"] >= 80000) & (employees["experience_years"] >= 5)
)

print(employees[["name", "salary", "is_senior", "is_high_earner"]])
# Output:
#              name  salary  is_senior  is_high_earner
# 0   alice sharma   52000      False           False
# 1       BOB MENON  95000       True            True
# 2     Charlie Rao  88000       True            True
# ...
```

---

## Level 1 Extended: `np.select()` for Multi-Condition Labels

This is the vectorized equivalent of a multi-branch `if/elif/else`. It handles multiple conditions in one pass over the array.

```python
conditions = [
    employees["salary"] >= 90000,
    employees["salary"] >= 60000,
    employees["salary"] >= 40000
]

labels = ["Band 4 - Principal", "Band 3 - Senior", "Band 2 - Mid"]

employees["salary_band"] = np.select(
    conditions,
    labels,
    default="Band 1 - Junior"
)

print(employees[["name", "salary", "salary_band"]])
# Output:
#              name  salary          salary_band
# 0   alice sharma   52000      Band 2 - Mid
# 1       BOB MENON  95000  Band 4 - Principal
# 2     Charlie Rao  88000  Band 3 - Senior
# 3      diana nair  44000      Band 2 - Mid
# 4      EVAN PATEL  61000      Band 3 - Senior
# 5    fatima khan  110000  Band 4 - Principal
```

`np.select()` evaluates conditions in order — the first matching condition wins.

---

## Level 1 Extended: `pd.cut()` for Binning Numeric Ranges

```python
employees["experience_bucket"] = pd.cut(
    employees["experience_years"],
    bins=[0, 2, 5, 10],
    labels=["Junior (0-2y)", "Mid (3-5y)", "Senior (6+y)"],
    include_lowest=True
)

print(employees[["name", "experience_years", "experience_bucket"]])
# Output:
#              name  experience_years experience_bucket
# 0   alice sharma                 2      Junior (0-2y)
# 1       BOB MENON                7         Senior (6+y)
# 2     Charlie Rao                5           Mid (3-5y)
# ...
```

---

## Level 2: `.map()` on a Series — Dictionary Lookups and Simple Substitution

`.map()` applies an element-wise mapping from a dictionary, a Series, or a function. It is the right tool when you are transforming values in a single column using a lookup table.

```python
dept_code_map = {
    "Sales": "SLS",
    "Engineering": "ENG",
    "HR": "HRM"
}

employees["dept_code"] = employees["department"].map(dept_code_map)

print(employees[["name", "department", "dept_code"]])
# Output:
#              name   department dept_code
# 0   alice sharma        Sales       SLS
# 1       BOB MENON  Engineering       ENG
# 2     Charlie Rao  Engineering       ENG
# 3      diana nair           HR       HRM
# ...
```

> [!warning] `.map()` returns NaN for values not in the dictionary
> If a value in the Series has no key in the mapping dictionary, the result is NaN — silently. Always check `result.isna().sum()` after a `.map()` call to make sure every value had a match.

```python
# Example of silent NaN from missing key
incomplete_map = {"Sales": "SLS", "Engineering": "ENG"}  # HR is missing
employees["dept_code_v2"] = employees["department"].map(incomplete_map)
print(employees["dept_code_v2"].isna().sum())
# Output: 1  (the HR row)
```

`.replace()` is an alternative that leaves unmatched values unchanged instead of converting them to NaN:

```python
employees["department"] = employees["department"].replace({
    "Engineering": "Software Engineering"
})
# HR and Sales remain unchanged; only Engineering rows are updated
```

---

## Level 2: `.str` Accessors — The Right Tool for Text

Never use `.apply()` for text cleaning. The `.str` accessor is vectorized and handles NaN automatically.

```python
# Clean the messy name column
employees["name_clean"] = (
    employees["name"]
    .str.strip()       # remove leading/trailing whitespace
    .str.title()       # title case
)

print(employees["name_clean"].tolist())
# Output: ['Alice Sharma', 'Bob Menon', 'Charlie Rao', 'Diana Nair', 'Evan Patel', 'Fatima Khan']
```

```python
# Common .str operations
employees["name_upper"] = employees["name_clean"].str.upper()
employees["name_lower"] = employees["name_clean"].str.lower()
employees["first_name"] = employees["name_clean"].str.split(" ").str[0]
employees["name_length"] = employees["name_clean"].str.len()
employees["starts_with_a"] = employees["name_clean"].str.startswith("A")
employees["has_sharma"] = employees["name_clean"].str.contains("Sharma", na=False)
```

> [!tip] `.str` methods are NaN-safe
> Unlike Python string methods, `.str.strip()`, `.str.upper()` etc. return NaN for NaN inputs rather than crashing. This makes them safe to use on real-world data without wrapping in try/except.

---

## Level 2: `.dt` Accessors — The Right Tool for Dates

```python
order_data = pd.DataFrame({
    "order_id": [1, 2, 3, 4],
    "order_date": pd.to_datetime(["2024-01-15", "2024-03-22", "2024-07-08", "2024-11-30"]),
    "delivery_date": pd.to_datetime(["2024-01-18", "2024-03-25", "2024-07-12", "2024-12-05"])
})

order_data["order_year"] = order_data["order_date"].dt.year
order_data["order_month"] = order_data["order_date"].dt.month
order_data["order_quarter"] = order_data["order_date"].dt.quarter
order_data["day_of_week"] = order_data["order_date"].dt.day_name()
order_data["is_weekend"] = order_data["order_date"].dt.dayofweek >= 5
order_data["delivery_days"] = (order_data["delivery_date"] - order_data["order_date"]).dt.days

print(order_data[["order_id", "order_month", "day_of_week", "delivery_days"]])
# Output:
#    order_id  order_month day_of_week  delivery_days
# 0         1            1      Monday              3
# 1         2            3      Friday              3
# 2         3            7      Monday              4
# 3         4           11    Saturday              5
```

---

## Level 3: `.apply()` on a Series — Complex Single-Column Logic

Use `.apply()` on a Series when the transformation for one column is too complex for a lambda or a simple vectorized expression, but does not need values from other columns.

```python
def performance_tier(rating: float) -> str:
    """Classify employee performance rating into human-readable tier."""
    if pd.isna(rating):
        return "Unrated"
    if rating >= 4.7:
        return "Exceptional"
    if rating >= 4.3:
        return "Strong"
    if rating >= 3.8:
        return "Meets Expectations"
    return "Below Expectations"


employees["performance_tier"] = employees["performance_rating"].apply(performance_tier)

print(employees[["name_clean", "performance_rating", "performance_tier"]])
# Output:
#       name_clean  performance_rating     performance_tier
# 0   Alice Sharma                 4.1  Meets Expectations
# 1      Bob Menon                 4.9          Exceptional
# 2    Charlie Rao                 4.4               Strong
# 3     Diana Nair                 3.7   Below Expectations
# 4     Evan Patel                 4.2  Meets Expectations
# 5   Fatima Khan                 4.8          Exceptional
```

Using a named function (not a lambda) is better here because:
- The function has a docstring
- The logic is readable even when complex
- It can be tested independently

Lambda is fine for simple one-liners:

```python
# Lambda: acceptable for simple transformations
employees["salary_k"] = employees["salary"].apply(lambda s: f"₹{s/1000:.0f}K")

print(employees["salary_k"].tolist())
# Output: ['₹52K', '₹95K', '₹88K', '₹44K', '₹61K', '₹110K']
```

---

## Level 4: `.apply(axis=1)` — When Logic Genuinely Needs Multiple Columns

This is the slowest option. Only use it when the decision depends on values from multiple columns in the same row, and no vectorized alternative exists.

```python
def compute_bonus(row: pd.Series) -> float:
    """
    Bonus logic: base 10% of salary.
    Top performers (rating >= 4.7) get an extra flat bonus.
    Senior employees (7+ years) get an additional seniority top-up.
    """
    base_bonus = row["salary"] * 0.10

    if row["performance_rating"] >= 4.7:
        base_bonus += 15000

    if row["experience_years"] >= 7:
        base_bonus += 8000

    return base_bonus


employees["annual_bonus"] = employees.apply(compute_bonus, axis=1)

print(employees[["name_clean", "salary", "performance_rating", "experience_years", "annual_bonus"]])
# Output:
#       name_clean  salary  performance_rating  experience_years  annual_bonus
# 0   Alice Sharma   52000                 4.1                 2        5200.0
# 1      Bob Menon   95000                 4.9                 7       32500.0
# 2    Charlie Rao   88000                 4.4                 5        8800.0
# 3     Diana Nair   44000                 3.7                 1        4400.0
# 4     Evan Patel   61000                 4.2                 3        6100.0
# 5   Fatima Khan  110000                 4.8                 9       34000.0
```

> [!warning] `apply(axis=1)` is row-by-row Python — it does not scale
> On 100 rows, `apply(axis=1)` feels instant. On 1 million rows, it can take minutes. The logic above can be rewritten with vectorized operations:

```python
# Vectorized equivalent — same result, much faster
base = employees["salary"] * 0.10
top_performer_bonus = np.where(employees["performance_rating"] >= 4.7, 15000, 0)
seniority_bonus = np.where(employees["experience_years"] >= 7, 8000, 0)
employees["annual_bonus_fast"] = base + top_performer_bonus + seniority_bonus

# Verify they match
print((employees["annual_bonus"] == employees["annual_bonus_fast"]).all())
# Output: True
```

The vectorized version processes all rows simultaneously in NumPy arrays. Use `np.where()` for single conditions, `np.select()` for multiple conditions.

---

## Applying Functions Across the Whole DataFrame

`DataFrame.apply()` without `axis=1` runs column-wise (default `axis=0`). This is useful for summarizing all numeric columns at once.

```python
numeric_cols = employees[["salary", "experience_years", "performance_rating"]]

print(numeric_cols.apply("mean"))
# Output:
# salary               75000.0
# experience_years         4.5
# performance_rating       4.35

print(numeric_cols.apply(lambda col: col.max() - col.min()))
# Output:
# salary               66000.0
# experience_years         8.0
# performance_rating       1.2
```

---

## Decision Guide: Which Tool to Use

| Transformation | Best tool | Reason |
|---------------|-----------|--------|
| Arithmetic between columns | Vectorized `+`, `*`, etc. | Fastest, most readable |
| Classify by one column's ranges | `pd.cut()` or `np.select()` | Vectorized |
| Map values to other values | `.map(dict)` | Vectorized lookup |
| Replace specific values | `.replace(dict)` | Vectorized, NaN-safe |
| String operations (strip, upper, split) | `.str.*` | Vectorized, NaN-safe |
| Date extraction (year, month, weekday) | `.dt.*` | Vectorized, NaN-safe |
| Complex logic, one column | `.apply(func)` on Series | Acceptable |
| Complex logic requiring multiple columns | `.apply(func, axis=1)` | Last resort |

---

## Common Mistakes

### Using `.apply()` for arithmetic that vectorizes

```python
# Slow — Python loop per row
employees["bonus_wrong"] = employees.apply(lambda r: r["salary"] * 0.10, axis=1)

# Fast — C-level array operation
employees["bonus_right"] = employees["salary"] * 0.10
```

### Forgetting `axis=1` for row-wise apply

```python
# Without axis=1, the function receives each COLUMN as a Series
employees.apply(compute_bonus)         # crashes or gives wrong result

# With axis=1, the function receives each ROW as a Series
employees.apply(compute_bonus, axis=1)  # correct
```

### Using `.apply()` for string operations

```python
# Slow
employees["name_upper"] = employees["name"].apply(str.upper)

# Fast
employees["name_upper"] = employees["name"].str.upper()
```

### Using `.apply()` on `.map()` candidates

```python
# Slow — applies a dict lookup function row by row
employees["dept_code"] = employees["department"].apply(
    lambda d: {"Sales": "SLS", "Engineering": "ENG", "HR": "HRM"}.get(d)
)

# Fast — .map() does the same thing vectorized
employees["dept_code"] = employees["department"].map(
    {"Sales": "SLS", "Engineering": "ENG", "HR": "HRM"}
)
```

---

> [!success] Key Takeaways
> - Vectorized operations are your first choice. They are fast, concise, and handle edge cases well.
> - `.map()` is the right tool for dictionary substitution on a single column.
> - `.str` and `.dt` accessors handle text and date transformations without apply.
> - `np.select()` replaces most multi-condition `apply(axis=1)` calls with vectorized code.
> - `.apply(axis=1)` is powerful and sometimes necessary — but it is slow at scale. Always ask if a vectorized alternative exists.

---

[[02-merge-and-join]] — Combining DataFrames | [[04-missing-values]] — Detect and handle missing data
