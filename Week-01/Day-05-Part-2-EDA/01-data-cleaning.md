# 🧼 01 — Data Cleaning for EDA
## Preparing Data Before Exploration

> [!info] Goal
> Learn the first cleaning checks to run before exploratory data analysis.

---

## Why Cleaning Comes First

EDA on messy data leads to wrong conclusions.

Before charts and statistics, check:

- column names
- data types
- missing values
- duplicates
- impossible values
- inconsistent categories

---

## Starter Workflow

```python
import pandas as pd
import numpy as np

df = pd.read_csv("data.csv")

print(df.head())
print(df.shape)
print(df.info())
print(df.isna().sum())
print(df.duplicated().sum())
```

---

## Clean Column Names

```python
df.columns = (
    df.columns
    .str.strip()
    .str.lower()
    .str.replace(" ", "_")
)
```

---

## Handle Duplicates

```python
df = df.drop_duplicates()
```

For key-based duplicates:

```python
df = df.drop_duplicates(subset=["customer_id"], keep="first")
```

---

## Missing Values

```python
missing = df.isna().sum().sort_values(ascending=False)
missing_pct = df.isna().mean().sort_values(ascending=False) * 100
```

Common handling:

```python
df["city"] = df["city"].fillna("Unknown")
df["age"] = df["age"].fillna(df["age"].median())
df = df.dropna(subset=["target"])
```

---

## Fix Data Types

```python
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")
df["date"] = pd.to_datetime(df["date"], errors="coerce")
```

---

## Clean Text Categories

```python
df["city"] = df["city"].str.strip().str.title()

df["gender"] = df["gender"].replace({
    "M": "Male",
    "F": "Female",
    "male": "Male",
    "female": "Female"
})
```

---

## Validate Business Rules

```python
df.loc[df["age"] < 0, "age"] = np.nan
df.loc[df["revenue"] < 0, "revenue"] = np.nan
```

Use domain knowledge. Negative revenue may be valid for refunds, but negative age is not.

---

## Practice

Given a raw customer dataset:

- inspect shape and info
- clean column names
- check duplicates
- check missing values
- convert numeric/date columns
- standardize text categories
- validate impossible values

---

## Interview Questions

**Q1:** What are the first checks in EDA?

> Shape, head, info, missing values, duplicates, and data types.

**Q2:** Why clean column names?

> Consistent names make analysis easier and reduce coding errors.

**Q3:** Should you always drop missing values?

> No. Decide based on column importance, missing percentage, and business meaning.

---

## 🔗 Next

➡️ [[02-outlier-detection]]
