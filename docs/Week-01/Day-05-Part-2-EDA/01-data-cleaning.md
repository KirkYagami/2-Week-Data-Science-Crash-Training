# Data Cleaning for EDA

Raw data is never ready to analyze. Every dataset you will encounter in the real world has missing values, type mismatches, inconsistent strings, and silent errors baked in. If you skip cleaning and go straight to charts, you will draw conclusions from noise. The cleaning step is not bureaucratic overhead — it is the foundation everything else stands on.

## Learning Objectives

By the end of this note you will be able to:

- Run a systematic first-pass audit on any new dataset
- Detect and handle missing values with intentionality, not reflexes
- Find and resolve duplicate rows
- Fix data type mismatches that break downstream analysis
- Standardize inconsistent categorical values
- Flag rows that violate business rules

---

## Why Cleaning Comes First

EDA is about asking questions and getting honest answers. Dirty data gives you dishonest answers. A mean age of 47 that includes a row where `age = 999` is not the mean age — it is a corrupted number. A churn rate that counts the same customer ID three times is not the churn rate.

Clean before you explore. Revisit cleaning after you explore. EDA and cleaning are iterative — each pass reveals new issues.

> [!info] The Iterative Reality
> Professional EDA is not linear. You will clean, explore, discover a new issue, go back and clean again, then explore further. Plan for at least two cleaning passes on any real dataset.

---

## Step 0 — Create a Realistic Dirty Dataset

Before diving into techniques, here is the kind of dataset you will actually encounter. All examples in this note use this data.

```python
import pandas as pd
import numpy as np

# Simulate a messy customer dataset
np.random.seed(42)

raw_data = {
    "Customer ID": [101, 102, 103, 104, 105, 106, 107, 108, 102, 109, 110],
    "  Age ": [25, 34, -5, 29, np.nan, 52, 31, 999, 34, 27, 45],
    "City": ["Delhi", "mumbai", "Delhi", "PUNE", "Mumbai ", "delhi", "Pune", "Mumbai", "mumbai", "Chennai", None],
    "Gender": ["Male", "F", "Male", "female", "M", "Female", "Male", "M", "F", "Male", "Female"],
    "Monthly Spend": ["1200", "3400", "800", "N/A", "950", "5100", "2200", "88000", "3400", "1100", "2900"],
    "Signup Date": ["2021-03-15", "2020-11-02", "2022-01-10", "2021-07-19", "2019-05-30",
                    "2023-02-14", "2021-09-01", "2020-08-22", "2020-11-02", "2022-12-05", "2023-06-18"],
    "Churn": ["No", "Yes", "No", "Yes", "No", "No", "Yes", "No", "Yes", "No", "No"]
}

df = pd.DataFrame(raw_data)
print(df.shape)
# Output: (11, 7)

print(df.head())
```

This dataset has: duplicate rows, wrong column name formatting, impossible age values, inconsistent gender strings, a numeric column stored as strings, a `None` city, and a potential outlier in spend. Every issue you will clean below appears here.

---

## Step 1 — The First-Pass Audit

Run these five commands before you do anything else. Read every line of output.

```python
print(df.shape)
# Output: (11, 7)  — 11 rows, 7 columns

print(df.dtypes)
# Output:
# Customer ID        int64
#   Age             float64
# City               object
# Gender             object
# Monthly Spend      object   <-- should be numeric, it's a string
# Signup Date        object   <-- should be datetime, it's a string
# Churn              object

print(df.info())
# Gives you non-null counts per column — missing values appear as count < total rows

print(df.isna().sum())
# Output:
# Customer ID      0
#   Age            1
# City             1
# Gender           0
# Monthly Spend    0
# Signup Date      0
# Churn            0

print(df.describe())
# Numeric summary — mean, std, min, max, quartiles
# If Age min = -5.0, something is wrong
```

> [!tip] Audit Before Touching
> Read the audit output completely before writing a single cleaning line. The audit tells you what problems exist. Then you write targeted fixes, not guesses.

---

## Step 2 — Clean Column Names

Column names with spaces, mixed case, and leading/trailing whitespace cause silent bugs. `df["  Age "]` is not the same as `df["age"]`. Fix names first so every subsequent line of code is clean.

```python
# Before
print(df.columns.tolist())
# Output: ['Customer ID', '  Age ', 'City', 'Gender', 'Monthly Spend', 'Signup Date', 'Churn']

# Clean all at once
df.columns = (
    df.columns
    .str.strip()            # remove leading/trailing whitespace
    .str.lower()            # lowercase everything
    .str.replace(" ", "_")  # spaces become underscores
    .str.replace(r"[^\w]", "", regex=True)  # drop any other special characters
)

print(df.columns.tolist())
# Output: ['customer_id', 'age', 'city', 'gender', 'monthly_spend', 'signup_date', 'churn']
```

---

## Step 3 — Fix Data Types

The type pandas infers is often wrong. Numeric columns may be stored as strings if even one row contains text. Date columns are almost always read as strings.

```python
# monthly_spend is a string because of "N/A" in the raw data
# errors="coerce" converts unparseable values to NaN instead of crashing
df["monthly_spend"] = pd.to_numeric(df["monthly_spend"], errors="coerce")

print(df["monthly_spend"].dtype)   # Output: float64
print(df["monthly_spend"].isna().sum())  # Output: 1  (the "N/A" row became NaN)

# signup_date should be datetime
df["signup_date"] = pd.to_datetime(df["signup_date"], errors="coerce")

print(df["signup_date"].dtype)   # Output: datetime64[ns]
```

> [!warning] `errors="coerce"` Creates Silent NaNs
> When you use `errors="coerce"` on a column, invalid values become `NaN`. This is usually what you want, but always check `.isna().sum()` after the conversion. If you suddenly have 500 more NaNs than before, your format string was wrong.

---

## Step 4 — Handle Duplicates

Duplicates inflate counts, distort means, and make your churn rates wrong. Always check before analyzing.

```python
print(df.duplicated().sum())
# Output: 1  (customer 102 appears twice)

# Inspect the duplicate rows first — don't delete blindly
print(df[df.duplicated(keep=False)])
#    customer_id   age   city gender  monthly_spend signup_date churn
# 1          102  34.0  mumbai      F         3400.0  2020-11-02   Yes
# 8          102  34.0  mumbai      F         3400.0  2020-11-02   Yes

# Identical rows — safe to drop
df = df.drop_duplicates()

print(df.shape)
# Output: (10, 7)
```

For key-based duplicates where you want to keep only the most recent record per customer:

```python
# Keep the latest record per customer_id
df = df.sort_values("signup_date").drop_duplicates(subset=["customer_id"], keep="last")
```

> [!warning] What Kind of Duplicate?
> A true duplicate (same row, same data) is safe to drop. A customer_id that appears twice with different transactions may not be a duplicate — it may mean two separate orders. Always understand what a row represents before dropping anything.

---

## Step 5 — Missing Value Audit

Missing values are not all the same. A `NaN` in `age` might mean the customer did not provide it. A `NaN` in `churn` might mean the outcome has not happened yet. A `NaN` in `city` after a database join might mean the join failed. The **reason** for missingness determines how you handle it.

```python
# Count and percentage
missing_count = df.isna().sum().sort_values(ascending=False)
missing_pct = (df.isna().mean() * 100).sort_values(ascending=False)

missing_report = pd.DataFrame({
    "missing_count": missing_count,
    "missing_pct": missing_pct.round(1)
})

print(missing_report[missing_report["missing_count"] > 0])
# Output:
#                missing_count  missing_pct
# monthly_spend              1         10.0
# city                       1         10.0
# age                        1         10.0
```

**Decision framework for missing values:**

| Scenario | Recommended Action |
|----------|-------------------|
| Target column has NaN | Drop those rows — you cannot train without a label |
| Missing < 5%, numerical | Impute with median (more robust than mean) |
| Missing < 5%, categorical | Impute with mode or "Unknown" |
| Missing 5–30%, numerical | Impute + create a binary missing indicator column |
| Missing > 30% | Consider dropping the column, or use models that handle NaN natively |
| Missing is informative | Create a missing indicator column, then impute |

```python
# Track that age was missing (missingness can be a signal)
df["age_was_missing"] = df["age"].isna().astype(int)

# Impute age with median
df["age"] = df["age"].fillna(df["age"].median())

# Impute city with "Unknown"
df["city"] = df["city"].fillna("Unknown")

# Impute monthly_spend with median (the N/A row became NaN during type coercion)
df["monthly_spend"] = df["monthly_spend"].fillna(df["monthly_spend"].median())

print(df.isna().sum())
# Output: all zeros — no more missing values
```

> [!warning] Never Drop All NaN Rows Reflexively
> `df.dropna()` is the most dangerous one-liner in EDA. If you have 10 columns and each has 5% missing, `dropna()` can delete 40% of your dataset. Dropping rows should be your last resort, not your first move.

---

## Step 6 — Standardize Categorical Values

String columns are almost always inconsistent. Humans type things differently. Data comes from multiple systems. The same value exists as "mumbai", "Mumbai", "MUMBAI", and "Mumbai " (trailing space) in real datasets.

```python
print(df["city"].value_counts())
# Output (before cleaning):
# mumbai      3  <-- "mumbai" and "Mumbai " are the same city
# Delhi       2
# PUNE        2
# Mumbai      1
# delhi       1
# Pune        1

# Fix: strip whitespace, then title-case
df["city"] = df["city"].str.strip().str.title()

print(df["city"].value_counts())
# Output (after cleaning):
# Mumbai    4
# Delhi     3
# Pune      3
```

For controlled vocabularies (gender, status codes), use explicit replacement:

```python
print(df["gender"].value_counts())
# Output: Male 5, F 3, female 2, M 1

gender_map = {
    "M": "Male",
    "F": "Female",
    "male": "Male",
    "female": "Female",
    "MALE": "Male",
    "FEMALE": "Female"
}

df["gender"] = df["gender"].str.strip().replace(gender_map)

print(df["gender"].value_counts())
# Output: Male 6, Female 4
```

> [!tip] Use `.str.strip()` Before Every String Operation
> Leading and trailing whitespace is the most common cause of "why doesn't my filter work?" bugs. Make it a habit to strip before grouping, replacing, or comparing.

---

## Step 7 — Validate Business Rules

Business rules are constraints that come from domain knowledge, not from the data itself. The data cannot tell you that a negative age is wrong — you know that. Apply these constraints explicitly.

```python
print(df["age"].describe())
# Output: min = -5.0, max = 999.0 — both are impossible

# Flag impossible values
impossible_age = df[(df["age"] < 0) | (df["age"] > 120)]
print(impossible_age[["customer_id", "age"]])
# Output:
#    customer_id    age
# 2          103   -5.0
# 7          108  999.0

# Set impossible values to NaN, then re-impute
df.loc[(df["age"] < 0) | (df["age"] > 120), "age"] = np.nan
df["age"] = df["age"].fillna(df["age"].median())

print(df["age"].describe())
# Output: min = 25.0, max = 52.0 — sensible range
```

```python
# Check for negative spend (refunds might be valid, but investigate)
print(df[df["monthly_spend"] < 0])  # Empty — none in this case

# Check for extreme outliers using the 99th percentile rule
p99 = df["monthly_spend"].quantile(0.99)
suspicious = df[df["monthly_spend"] > p99]
print(suspicious[["customer_id", "monthly_spend"]])
# Output: customer 108 with spend = 88000 — 25x the next highest
```

> [!info] Outliers and Business Rules Are Different
> A negative age violates a business rule — it is definitively wrong. A spend of 88000 might be an outlier, but it might also be a real high-value customer. Handle rule violations as errors. Handle outliers separately with domain judgment.

---

## The Data Cleaning Checklist

Run through this list on every new dataset:

- [ ] **Shape** — how many rows, how many columns?
- [ ] **Column names** — strip, lowercase, underscores
- [ ] **Data types** — does each column's type match its content?
- [ ] **Duplicates** — full duplicates and key-based duplicates
- [ ] **Missing values** — count, percentage, pattern, then decide
- [ ] **Categorical consistency** — strip, case-normalize, explicit mapping
- [ ] **Business rule violations** — impossible values based on domain knowledge
- [ ] **Date ranges** — are dates within a plausible range?
- [ ] **Numeric ranges** — are values within expected bounds?
- [ ] **Document every decision** — note what you changed and why

> [!success] Clean Data is an Investment
> Every hour spent cleaning data saves three hours of debugging wrong conclusions. Analysts who skip cleaning spend their time arguing with their own outputs.

---

## Practice Exercises

### Warm-Up

```python
import pandas as pd
import numpy as np

raw = pd.DataFrame({
    "Name": ["Alice", "Bob", "alice", "  Charlie  ", "Bob"],
    "Score": ["88", "92", "75", "N/A", "92"],
    "Grade": ["A", "a", "B", "b", "a"]
})
```

1. Clean the `Name` column so "Alice", "alice", and "  Charlie  " all become consistent.
2. Convert `Score` to numeric. How many NaN values appear?
3. Standardize `Grade` to uppercase. How many unique grades remain?
4. Find and remove duplicate rows.

### Main Exercise

```python
transactions = pd.DataFrame({
    "txn_id": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    "customer_id": [201, 202, 203, 201, 204, 205, 202, 206, 207, 207],
    "amount": [250.0, 1800.0, -99.0, 300.0, np.nan, 4200.0, 1800.0, 75.0, 950.0, 950.0],
    "payment_method": ["Card", "cash", "Card", "CASH", "card", "Card", "cash", "Card", "Cash", "Cash"],
    "date": ["2023-01-10", "2023-01-11", "2023-01-12", "invalid_date",
             "2023-01-14", "2023-01-15", "2023-01-11", "2023-01-17", "2023-01-18", "2023-01-18"]
})
```

Write a cleaning pipeline that:
1. Detects and removes fully duplicate rows
2. Converts `date` to datetime (coercing errors) and reports how many failed
3. Standardizes `payment_method` to title case
4. Flags negative amounts as invalid and sets them to `NaN`
5. Imputes the missing amount with the column median
6. Produces a final `.info()` showing no remaining issues

### Stretch

Add a function `audit_dataframe(df)` that accepts any DataFrame and prints:
- Shape
- Column types
- Missing count and percentage per column
- Duplicate count
- Min/max for each numeric column

---

## Interview Questions

**Q1: What is the difference between `df.dropna()` and handling missing values intentionally?**

??? "Show answer"
    `df.dropna()` removes every row that contains at least one NaN, regardless of which column or how important that row is. Intentional handling means you examine each column separately, understand why values are missing, and choose the appropriate strategy: imputation for some columns, indicator flags for others, and dropping only rows where the target or a critical key is missing. Reflexive dropping can silently remove 30–50% of a dataset.

**Q2: You have a column where 40% of values are missing. What do you do?**

??? "Show answer"
    First, investigate why it's missing — is there a pattern (all missing for one segment, or one time period)? If the missingness is random, consider dropping the column. If missingness is informative (e.g., customers who didn't fill out income are a specific segment), create a binary indicator column and either drop the original or impute it. Never just drop the column without checking whether missingness itself is a signal.

**Q3: How would you detect that a numeric column is actually stored as strings?**

??? "Show answer"
    `df.dtypes` will show `object` for a column that should be numeric. You can also check `df.info()` which shows the non-null count and dtype together. To confirm, try `pd.to_numeric(df["column"], errors="coerce")` — if this produces NaNs where the original had non-missing values, those values were non-numeric strings.

**Q4: What is the danger of cleaning data before understanding the problem domain?**

??? "Show answer"
    You might delete data that is correct. A transaction amount of -500 might look wrong but is a valid refund. An age of 2 might look impossible for a customer but makes sense for a child's account in a family plan. A missing value in a medical dataset might mean "not tested" vs "test result was normal" — those are very different things. Domain understanding should precede cleaning decisions.

---

[[../Day-05-Part-1-Statistics/06-statistical-testing|Previous: Statistical Testing]] | [[02-outlier-detection|Next: Outlier Detection]]
