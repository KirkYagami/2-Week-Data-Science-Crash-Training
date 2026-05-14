# 🧩 03 — Apply Functions
## Custom Logic in Pandas

> [!info] Goal
> Learn when to use vectorized operations, `.map()`, `.apply()`, and row-wise functions.

---

## Start with Vectorization

Before using `apply()`, ask: can this be done with vectorized Pandas operations?

```python
df["revenue"] = df["quantity"] * df["price"]
```

This is faster and clearer than row-wise `apply()`.

---

## Sample Dataset

```python
import pandas as pd

employees = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie", "Diana"],
    "department": ["Sales", "Tech", "Tech", "HR"],
    "salary": [50000, 80000, 75000, 45000],
    "experience": [2, 5, 4, 1],
    "rating": [4.2, 4.8, 4.5, 3.9]
})
```

---

## `map()` for Series Transformations

Use `.map()` to transform values in one Series.

```python
department_codes = {
    "Sales": "S",
    "Tech": "T",
    "HR": "H"
}

employees["department_code"] = employees["department"].map(department_codes)
```

If a value is not found in the dictionary, the result is `NaN`.

---

## `replace()` for Value Replacement

```python
employees["department"] = employees["department"].replace({
    "Tech": "Technology",
    "HR": "Human Resources"
})
```

Use `replace()` when you want to rewrite values.

---

## `apply()` on a Series

```python
def salary_level(salary):
    if salary >= 75000:
        return "High"
    if salary >= 50000:
        return "Medium"
    return "Low"


employees["salary_level"] = employees["salary"].apply(salary_level)
```

This applies the function to each value in the `salary` column.

---

## `apply()` with Lambda

```python
employees["name_length"] = employees["name"].apply(lambda name: len(name))
```

Use lambda for simple one-line logic. Use a named function when logic grows.

---

## Row-Wise `apply()`

Use `axis=1` to apply a function to each row.

```python
def performance_label(row):
    if row["rating"] >= 4.5 and row["experience"] >= 4:
        return "Top Performer"
    if row["rating"] >= 4.0:
        return "Good"
    return "Needs Support"


employees["performance"] = employees.apply(performance_label, axis=1)
```

> [!warning] Performance
> Row-wise `apply(axis=1)` is convenient but slower than vectorized operations. Use it when the logic is genuinely row-dependent and not easy to vectorize.

---

## Vectorized Alternative with `np.select()`

```python
import numpy as np

conditions = [
    (employees["rating"] >= 4.5) & (employees["experience"] >= 4),
    employees["rating"] >= 4.0
]

choices = ["Top Performer", "Good"]

employees["performance"] = np.select(
    conditions,
    choices,
    default="Needs Support"
)
```

This is often faster and cleaner for multiple conditional labels.

---

## String Methods

Prefer vectorized `.str` methods for text columns.

```python
employees["name_upper"] = employees["name"].str.upper()
employees["name_clean"] = employees["name"].str.strip().str.title()
employees["starts_with_a"] = employees["name"].str.startswith("A")
```

---

## Date Methods

```python
df["order_date"] = pd.to_datetime(df["order_date"])

df["year"] = df["order_date"].dt.year
df["month"] = df["order_date"].dt.month
df["day_name"] = df["order_date"].dt.day_name()
```

Prefer `.dt` methods over custom date functions.

---

## Applying Functions to Multiple Columns

```python
def bonus(row):
    base_bonus = row["salary"] * 0.10
    if row["rating"] >= 4.5:
        return base_bonus + 5000
    return base_bonus


employees["bonus"] = employees.apply(bonus, axis=1)
```

---

## `apply()` Across Columns

```python
numeric = employees[["salary", "experience", "rating"]]

numeric.apply("mean")
numeric.apply("max")
```

By default, `DataFrame.apply()` works column-wise with `axis=0`.

---

## Common Decision Guide

| Need | Prefer |
|------|--------|
| Arithmetic between columns | Vectorized operation |
| Replace exact values | `replace()` |
| Map categories to labels | `map()` |
| Text cleaning | `.str` methods |
| Date extraction | `.dt` methods |
| Multiple conditional labels | `np.select()` |
| Complex row logic | `apply(axis=1)` |

---

## Common Mistakes

### Using Apply When Vectorization Is Enough

```python
# Slower
df["revenue"] = df.apply(lambda row: row["quantity"] * row["price"], axis=1)

# Better
df["revenue"] = df["quantity"] * df["price"]
```

### Forgetting `axis=1`

```python
df.apply(function)
```

This applies the function column-wise, not row-wise.

Use:

```python
df.apply(function, axis=1)
```

---

## Practice

Using the `employees` DataFrame:

- create a salary level column
- map departments to department codes
- create a performance label using `apply(axis=1)`
- rewrite the performance label using `np.select()`
- clean names using `.str.strip().str.title()`
- create a bonus column from salary and rating

---

## Interview Questions

**Q1:** When should you avoid `apply()`?

> Avoid it when a vectorized Pandas or NumPy operation can solve the problem.

**Q2:** What does `axis=1` mean in `apply()`?

> It applies the function row by row.

**Q3:** What is the difference between `map()` and `apply()`?

> `map()` works on a Series and is commonly used for value mapping. `apply()` can run custom functions on Series or DataFrames.

---

## ✅ Key Takeaways

- Prefer vectorized operations first.
- Use `map()` for simple category mapping.
- Use `replace()` for rewriting values.
- Use `.str` and `.dt` accessors for text and date features.
- Use `apply(axis=1)` for complex row-wise logic.
- Use `np.select()` for efficient multi-condition labels.

---

## 🔗 What's Next?

➡️ [[04-missing-values]] — Detect and handle missing data
