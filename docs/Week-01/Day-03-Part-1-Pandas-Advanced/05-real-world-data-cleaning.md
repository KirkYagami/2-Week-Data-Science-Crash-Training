# 🧼 05 — Real-World Data Cleaning
## From Messy Table to Analysis-Ready Dataset

> [!info] Goal
> Practice a repeatable Pandas cleaning workflow for messy real-world data.

---

## Why Real Data Is Messy

Real datasets commonly contain:

- inconsistent column names
- extra spaces
- mixed capitalization
- numbers stored as text
- dates stored as strings
- missing values
- duplicate rows
- impossible values
- category spelling variations

Good data cleaning is systematic, not random.

---

## Messy Sample Dataset

```python
import pandas as pd
import numpy as np

raw = pd.DataFrame({
    " Customer ID ": [1, 2, 3, 3, 4, 5],
    "Name": [" alice ", "BOB", "Charlie", "Charlie", None, "fatima"],
    "Age": ["25", "unknown", "32", "32", "-5", "41"],
    "City": ["delhi", "Mumbai ", "", "Mumbai", "PUNE", None],
    "Spend": ["1,200", "2500", "", "2500", "-100", "3200"],
    "Signup Date": ["2024-01-10", "not available", "2024/03/05", "2024/03/05", "2024-04-20", "2024-05-01"]
})
```

---

## Step 1 — Inspect First

```python
print(raw.head())
print(raw.shape)
print(raw.info())
print(raw.isna().sum())
```

Before cleaning, understand what is wrong.

---

## Step 2 — Standardize Column Names

```python
df = raw.copy()

df.columns = (
    df.columns
    .str.strip()
    .str.lower()
    .str.replace(" ", "_")
)

print(df.columns)
```

Result:

```text
customer_id, name, age, city, spend, signup_date
```

---

## Step 3 — Remove Duplicate Rows

```python
df = df.drop_duplicates()
```

If duplicates should be based on one key:

```python
df = df.drop_duplicates(subset=["customer_id"], keep="first")
```

---

## Step 4 — Clean Text Columns

```python
df["name"] = df["name"].str.strip().str.title()
df["city"] = df["city"].str.strip().str.title()
```

Convert empty strings to missing:

```python
df = df.replace({"": np.nan, "unknown": np.nan, "not available": np.nan})
```

---

## Step 5 — Convert Numeric Columns

Remove commas before converting:

```python
df["spend"] = df["spend"].str.replace(",", "", regex=False)
df["spend"] = pd.to_numeric(df["spend"], errors="coerce")
```

Convert age:

```python
df["age"] = pd.to_numeric(df["age"], errors="coerce")
```

Invalid values become `NaN`.

---

## Step 6 — Convert Dates

```python
df["signup_date"] = pd.to_datetime(df["signup_date"], errors="coerce")
```

Extract date features if useful:

```python
df["signup_year"] = df["signup_date"].dt.year
df["signup_month"] = df["signup_date"].dt.month
```

---

## Step 7 — Fix Impossible Values

```python
df.loc[df["age"] < 0, "age"] = np.nan
df.loc[df["spend"] < 0, "spend"] = np.nan
```

Domain rules matter. Negative age and negative spend are invalid here.

---

## Step 8 — Handle Missing Values

```python
df["name"] = df["name"].fillna("Unknown")
df["city"] = df["city"].fillna("Unknown")
df["age"] = df["age"].fillna(df["age"].median())
df["spend"] = df["spend"].fillna(df["spend"].median())
```

For important IDs, dropping may be better:

```python
df = df.dropna(subset=["customer_id"])
```

---

## Step 9 — Validate Cleaned Data

```python
print(df.head())
print(df.info())
print(df.isna().sum())
print(df.describe())
```

Check business rules:

```python
print((df["age"] < 0).sum())
print((df["spend"] < 0).sum())
print(df["customer_id"].duplicated().sum())
```

---

## Step 10 — Save Cleaned Data

```python
df.to_csv("customers_clean.csv", index=False)
```

---

## Full Cleaning Pipeline

```python
import pandas as pd
import numpy as np


def clean_customers(raw):
    df = raw.copy()

    df.columns = (
        df.columns
        .str.strip()
        .str.lower()
        .str.replace(" ", "_")
    )

    df = df.drop_duplicates(subset=["customer_id"], keep="first")
    df = df.replace({"": np.nan, "unknown": np.nan, "not available": np.nan})

    df["name"] = df["name"].str.strip().str.title()
    df["city"] = df["city"].str.strip().str.title()

    df["age"] = pd.to_numeric(df["age"], errors="coerce")
    df["spend"] = (
        df["spend"]
        .str.replace(",", "", regex=False)
        .pipe(pd.to_numeric, errors="coerce")
    )
    df["signup_date"] = pd.to_datetime(df["signup_date"], errors="coerce")

    df.loc[df["age"] < 0, "age"] = np.nan
    df.loc[df["spend"] < 0, "spend"] = np.nan

    df["age_was_missing"] = df["age"].isna()
    df["spend_was_missing"] = df["spend"].isna()

    df["name"] = df["name"].fillna("Unknown")
    df["city"] = df["city"].fillna("Unknown")
    df["age"] = df["age"].fillna(df["age"].median())
    df["spend"] = df["spend"].fillna(df["spend"].median())

    return df


cleaned = clean_customers(raw)
print(cleaned)
```

---

## Cleaning Checklist

- [ ] Inspect shape, head, info, and missing values
- [ ] Standardize column names
- [ ] Remove duplicate rows
- [ ] Strip whitespace from text
- [ ] Standardize capitalization
- [ ] Convert placeholder values to `NaN`
- [ ] Convert numeric columns with `pd.to_numeric()`
- [ ] Convert date columns with `pd.to_datetime()`
- [ ] Fix impossible values
- [ ] Handle missing values
- [ ] Validate cleaned result
- [ ] Save cleaned output

---

## Common Mistakes

### Cleaning the Original Without Copying

```python
df = raw.copy()
```

Keep raw data unchanged so you can restart if needed.

### Not Validating After Cleaning

Always check that the cleaning worked.

### Hiding Bad Data Too Early

Sometimes you should report impossible values instead of silently fixing them.

### Using One Cleaning Strategy for Every Column

Names, ages, dates, IDs, and spend values need different logic.

---

## Practice

Using the messy `raw` DataFrame:

- standardize column names
- remove duplicate customers
- clean name and city formatting
- convert age and spend to numeric
- convert signup date to datetime
- replace negative age and spend with missing values
- fill missing city with `"Unknown"`
- fill missing age and spend with medians
- save the cleaned data to CSV

---

## Interview Questions

**Q1:** What is a typical data cleaning workflow?

> Inspect, standardize columns, clean text, convert types, handle duplicates, handle missing values, validate, then save.

**Q2:** Why use `errors="coerce"`?

> It converts invalid parsing values to missing values instead of crashing.

**Q3:** Why keep a raw copy of the data?

> So you can reproduce the cleaning process and recover if a cleaning step goes wrong.

---

## ✅ Key Takeaways

- Real cleaning is a sequence of small, testable steps.
- Always inspect before and after cleaning.
- Convert data types deliberately.
- Use business rules to detect impossible values.
- Wrap repeatable cleaning logic in a function.

---

## 🔗 What's Next?

➡️ [[06-exercises]] — Practice advanced Pandas workflows
