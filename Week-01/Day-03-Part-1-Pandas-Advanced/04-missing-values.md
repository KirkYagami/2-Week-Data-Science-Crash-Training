# 🕳️ 04 — Missing Values
## Detecting and Handling Incomplete Data

> [!info] Goal
> Learn how to find, understand, remove, and fill missing values in Pandas.

---

## Why Missing Values Matter

Missing data can:

- break calculations
- distort averages and totals
- reduce model quality
- hide data collection problems
- create misleading analysis

Handling missing values is a core data cleaning skill.

---

## Sample Dataset

```python
import pandas as pd
import numpy as np

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4, 5, 6],
    "name": ["Alice", "Bob", None, "Diana", "Evan", "Fatima"],
    "age": [25, np.nan, 32, 29, np.nan, 41],
    "city": ["Delhi", "Mumbai", None, "Pune", "Delhi", None],
    "spend": [1200, 2500, np.nan, 1800, 0, 3200],
    "signup_date": ["2024-01-10", None, "2024-03-05", "bad date", "2024-04-20", "2024-05-01"]
})
```

---

## Detect Missing Values

```python
customers.isna()
customers.notna()
```

Count missing values by column:

```python
customers.isna().sum()
```

Missing percentage:

```python
customers.isna().mean() * 100
```

Rows with any missing values:

```python
customers[customers.isna().any(axis=1)]
```

Rows where a specific column is missing:

```python
customers[customers["age"].isna()]
```

---

## Missing Values vs Invalid Values

Missing values are usually `NaN`, `None`, or `NaT`.

Invalid values may look present but still be wrong:

- age = `-5`
- spend = `"unknown"`
- date = `"bad date"`
- city = empty string `""`

Convert invalid values to missing values when appropriate.

```python
customers["signup_date"] = pd.to_datetime(
    customers["signup_date"],
    errors="coerce"
)
```

Invalid dates become `NaT`.

---

## Drop Missing Values

### Drop Rows with Any Missing Value

```python
clean = customers.dropna()
```

This can remove many rows, so use carefully.

### Drop Rows Missing Specific Columns

```python
clean = customers.dropna(subset=["name", "city"])
```

### Drop Columns with Too Many Missing Values

```python
clean = customers.dropna(axis=1)
```

More practical:

```python
threshold = len(customers) * 0.5
clean = customers.dropna(axis=1, thresh=threshold)
```

This keeps columns with at least 50% non-missing values.

---

## Fill Missing Values

### Fill with Constant

```python
customers["city"] = customers["city"].fillna("Unknown")
```

### Fill Numeric with Mean or Median

```python
customers["age"] = customers["age"].fillna(customers["age"].median())
customers["spend"] = customers["spend"].fillna(customers["spend"].mean())
```

Median is often safer than mean when outliers exist.

### Fill with Mode

```python
most_common_city = customers["city"].mode()[0]
customers["city"] = customers["city"].fillna(most_common_city)
```

---

## Forward Fill and Backward Fill

Useful for time series data.

```python
df["price"] = df["price"].ffill()
df["price"] = df["price"].bfill()
```

| Method | Meaning |
|--------|---------|
| `ffill()` | Fill using previous valid value |
| `bfill()` | Fill using next valid value |

---

## Group-Based Filling

Fill missing ages using median age by city:

```python
customers["age"] = customers["age"].fillna(
    customers.groupby("city")["age"].transform("median")
)
```

This is better than using one global median when groups differ.

---

## Mark Missingness

Sometimes missingness itself is meaningful.

```python
customers["age_was_missing"] = customers["age"].isna()
```

Create missingness indicator columns before filling values.

---

## Replace Placeholder Missing Values

Datasets often use placeholders:

```python
df = df.replace({
    "": np.nan,
    "NA": np.nan,
    "N/A": np.nan,
    "missing": np.nan,
    "-": np.nan
})
```

When reading CSV:

```python
df = pd.read_csv("data.csv", na_values=["", "NA", "N/A", "missing", "-"])
```

---

## Recommended Workflow

```python
df = pd.read_csv("data.csv", na_values=["", "NA", "N/A", "missing", "-"])

print(df.shape)
print(df.isna().sum())
print(df.isna().mean() * 100)

df["age_missing"] = df["age"].isna()
df["age"] = df["age"].fillna(df["age"].median())
df["city"] = df["city"].fillna("Unknown")

df = df.dropna(subset=["customer_id"])
```

---

## Common Mistakes

### Filling Everything with Zero

Zero is a real value. It is not always the same as missing.

### Dropping Rows Too Early

Dropping all rows with any missing values can destroy useful data.

### Filling Before Investigating

Always inspect missing counts and percentages first.

### Forgetting Invalid Values

Missing checks do not catch invalid strings like `"unknown"` unless you convert them.

---

## Practice

Using the `customers` DataFrame:

- count missing values by column
- calculate missing percentage by column
- show rows where city is missing
- convert `signup_date` to datetime with invalid dates coerced
- create an `age_was_missing` column
- fill missing age with median age
- fill missing city with `"Unknown"`
- drop rows missing `name`

---

## Interview Questions

**Q1:** What is the difference between `isna()` and `notna()`?

> `isna()` returns True for missing values. `notna()` returns True for present values.

**Q2:** When should you use median instead of mean for filling?

> Use median when numeric data has outliers or is skewed.

**Q3:** Why create a missingness indicator?

> Missingness can carry information, and the indicator preserves that signal after filling.

---

## ✅ Key Takeaways

- Always inspect missing values before handling them.
- Use `dropna()` carefully.
- Use `fillna()` with a meaningful strategy.
- Convert placeholder strings to real missing values.
- Use group-based filling when groups have different distributions.
- Preserve missingness indicators when missing data may be meaningful.

---

## 🔗 What's Next?

➡️ [[05-real-world-data-cleaning]] — Combine multiple cleaning skills into a practical workflow
