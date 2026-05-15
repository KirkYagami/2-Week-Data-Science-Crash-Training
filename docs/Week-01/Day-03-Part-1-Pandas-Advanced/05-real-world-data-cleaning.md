# Real-World Data Cleaning — From Messy Table to Analysis-Ready Dataset

Every dataset a business gives you is dirty. Not a little dirty. Structurally, pervasively dirty. Column names with spaces and inconsistent casing. Dates stored as strings, sometimes three different formats in the same column. Numbers with commas. Ages of -999 (a programmer's "unknown" placeholder). Duplicate rows because someone exported twice and concatenated the files. Categories spelled three different ways.

The difference between someone who finishes a cleaning job in 30 minutes and someone who spends three hours is having a repeatable, systematic process — not cleverness. This section gives you that process.

## Learning Objectives

By the end of this section you will be able to:

- Execute a full cleaning pipeline in a fixed, reproducible order
- Standardize column names with a single line of chained string operations
- Detect and remove exact duplicates and key-based duplicates
- Standardize inconsistent text categories (case, whitespace, common typos)
- Convert numeric and date columns safely using `errors="coerce"`
- Flag and fix impossible values using domain knowledge
- Detect and remove outliers using IQR and z-score methods
- Downcast numeric dtypes to reduce memory usage
- Build a cleaning function that is reusable across similar datasets

---

## The Messy Dataset

This is realistic. Every problem below is pulled from real data I have seen in production.

```python
import pandas as pd
import numpy as np

raw = pd.DataFrame({
    " Customer ID ": [1, 2, 3, 3, 4, 5, 6],
    "Name": [" alice sharma ", "BOB MENON", "Charlie Rao", "Charlie Rao", None, " FATIMA ", "Gopal Singh"],
    "Age": ["25", "unknown", "32", "32", "-5", "41", "150"],  # "unknown", negative, impossible
    "City": ["delhi", "Mumbai ", "  pune", "Pune", "BANGALORE", None, "mumbai"],
    "Annual Spend": ["1,200", "25000", "", "12000", "-500", "32,000", "N/A"],
    "Plan": ["premium", "BASIC", "Premium", "Premium", "basic", "PREMIUM", "Basic"],
    "Signup Date": ["2024-01-10", "not available", "2024/03/05", "2024/03/05",
                    "2024-04-20", "2024-05-01", "15-06-2024"]
})
```

---

## The Cleaning Order (and Why It Matters)

Order is not arbitrary. Each step produces a cleaner foundation for the next.

```
1. Inspect — understand what you have before touching anything
2. Copy raw data — never mutate the original
3. Standardize column names — everything downstream depends on column names
4. Replace placeholder values — convert "unknown", "N/A", "" to NaN
5. Remove duplicates — before filling missing values, not after
6. Clean text columns — whitespace, case, known typos
7. Convert types — numeric, dates, categories
8. Fix impossible values — domain rules, not statistics
9. Handle missing values — fill or drop
10. Detect and handle outliers — statistical and domain-based
11. Optimize dtypes — reduce memory usage
12. Validate — assert your expectations
13. Save — persist the clean version
```

---

## Step 1: Inspect — Understand Before You Touch

```python
print(raw.shape)
# Output: (7, 7)

print(raw.dtypes)
# Output: everything is object (string) — this is typical of CSV imports

print(raw.head(3))
# Shows the raw values before any cleaning

print(raw.isna().sum())
# Column-level missing count

# Check unique values in categorical-ish columns
print(raw["Plan"].value_counts(dropna=False))
# Output shows "premium", "BASIC", "Premium" etc. — inconsistent casing
```

---

## Step 2: Copy — Protect the Raw Data

```python
df = raw.copy()
```

Always work on a copy. You will make mistakes during cleaning. With a copy, you restart the cleaning pipeline without losing the original. In a production setting, raw data is often immutable by policy — you should simulate that discipline even in notebooks.

---

## Step 3: Standardize Column Names

```python
df.columns = (
    df.columns
    .str.strip()           # remove leading/trailing whitespace: " Customer ID " → "Customer ID"
    .str.lower()           # lowercase: "Customer ID" → "customer id"
    .str.replace(" ", "_", regex=False)  # spaces to underscores: "customer id" → "customer_id"
    .str.replace(r"[^a-z0-9_]", "", regex=True)  # remove any other special characters
)

print(df.columns.tolist())
# Output: ['customer_id', 'name', 'age', 'city', 'annual_spend', 'plan', 'signup_date']
```

Column names should be lowercase, underscore-separated, and free of special characters. This lets you access them with `df.column_name` dot notation and pass them safely to any library.

---

## Step 4: Replace Placeholder Values with NaN

Do this before anything else — otherwise `"unknown"` passes type checks and becomes NaN after type conversion only if you use `errors="coerce"`.

```python
placeholders = ["", "N/A", "n/a", "NA", "unknown", "not available", "none", "-", "null", "NULL"]

df = df.replace(placeholders, np.nan)

print(df.isna().sum())
# Now "unknown" and "not available" are properly NaN
```

---

## Step 5: Remove Duplicates

Do this before filling missing values. If you fill first and then deduplicate, you might keep filled-in rows and drop the original.

```python
print(f"Before deduplication: {len(df)} rows")

# Exact duplicates: every column matches
df = df.drop_duplicates()
print(f"After exact dedup: {len(df)} rows")

# Key-based duplicates: same customer_id (convert to int first for this to work)
df["customer_id"] = pd.to_numeric(df["customer_id"], errors="coerce")
df = df.drop_duplicates(subset=["customer_id"], keep="first")
print(f"After key-based dedup: {len(df)} rows")
# Output: Before: 7, After exact: 6, After key-based: 6 (same, since exact dup was the key dup here)
```

> [!warning] `drop_duplicates()` with no arguments drops complete row duplicates only
> If two rows have the same `customer_id` but slightly different values in other columns (typo in a name, different timestamp), they are not exact duplicates. You need `subset=["customer_id"]` to deduplicate by business key. Decide which record to keep (`keep="first"` or `keep="last"`) based on your data's logic — usually the most recent entry.

---

## Step 6: Clean Text Columns

```python
# Strip whitespace and standardize case for name
df["name"] = df["name"].str.strip().str.title()

# Strip and title-case city — but also fix common variations
df["city"] = df["city"].str.strip().str.title()

# Standardize plan — strip and title case handles "BASIC", " premium ", "Basic"
df["plan"] = df["plan"].str.strip().str.title()

print(df[["name", "city", "plan"]].head(6))
# Output:
#           name       city    plan
# 0  Alice Sharma      Delhi  Premium
# 1     Bob Menon     Mumbai    Basic
# 2   Charlie Rao       Pune  Premium
# 3        None  Bangalore     Basic
# 4       Fatima       None  Premium
# 5  Gopal Singh     Mumbai    Basic
```

### Handling known typos and inconsistent spellings

```python
# After .str.title(), "Bengaluru" and "Bangalore" still differ
# Use .replace() to normalize known variants
city_corrections = {
    "Bengaluru": "Bangalore",
    "New Delhi": "Delhi",
    "Bombay": "Mumbai"
}

df["city"] = df["city"].replace(city_corrections)
```

> [!tip] Always inspect value_counts() after text cleaning
> Run `df["city"].value_counts()` after standardizing. You will often find you still have "Pune " (trailing space survived), "PUNE" (all caps), or "Pune, Maharashtra" (extra detail) that needs another pass.

---

## Step 7: Convert Data Types

### Numeric conversion

```python
# Remove formatting characters before converting
df["annual_spend"] = (
    df["annual_spend"]
    .str.replace(",", "", regex=False)    # remove thousands separator
    .str.replace("₹", "", regex=False)    # remove currency symbol if present
    .pipe(pd.to_numeric, errors="coerce") # convert; invalid → NaN
)

df["age"] = pd.to_numeric(df["age"], errors="coerce")

print(df[["customer_id", "age", "annual_spend"]].dtypes)
# Output:
# customer_id      float64
# age              float64
# annual_spend     float64
```

### Date conversion

```python
# Pandas handles multiple date formats reasonably well with dayfirst=False
df["signup_date"] = pd.to_datetime(df["signup_date"], errors="coerce", dayfirst=False)

# Inspect what became NaT
print(df[df["signup_date"].isna()][["customer_id", "signup_date"]])
# Shows "not available" and any format Pandas could not parse

# Extract useful date features
df["signup_year"] = df["signup_date"].dt.year
df["signup_month"] = df["signup_date"].dt.month
df["signup_quarter"] = df["signup_date"].dt.quarter
```

> [!warning] Date format ambiguity: 05/06/2024 is May 6 or June 5?
> `pd.to_datetime("05/06/2024")` will guess. In Indian data, day-first formats are common. Set `dayfirst=True` if your dates are in DD/MM/YYYY format. When in doubt, look at a sample of dates to figure out which format is used, then specify it explicitly with `format="%d/%m/%Y"`.

### Categorical conversion

```python
# Convert low-cardinality string columns to Categorical for memory savings and ordering
df["plan"] = pd.Categorical(df["plan"], categories=["Basic", "Premium"], ordered=True)
df["city"] = df["city"].astype("category")

print(df["plan"].dtype)   # Output: CategoricalDtype(categories=['Basic', 'Premium'], ordered=True)
```

---

## Step 8: Fix Impossible Values Using Domain Rules

Type conversion makes values numeric, but it cannot catch business-rule violations. That requires domain knowledge.

```python
# Age: human age range is 0–120
print(f"Ages before fix: {df['age'].describe()}")

df.loc[df["age"] < 0, "age"] = np.nan
df.loc[df["age"] > 120, "age"] = np.nan

print(f"Ages after fix: {df['age'].describe()}")
# min is now >= 0, max <= 120

# Spend: negative spend is impossible (returns, credits would be separate)
df.loc[df["annual_spend"] < 0, "annual_spend"] = np.nan

# Validate: customer_id must be positive integer
df.loc[df["customer_id"] <= 0, "customer_id"] = np.nan
```

---

## Step 9: Handle Missing Values

By this step, all your missing values are genuine NaN — not disguised strings. You can now apply strategies with confidence.

```python
# Create indicators for informative missingness before filling
df["age_was_missing"] = df["age"].isna()
df["spend_was_missing"] = df["annual_spend"].isna()

# Fill missing age with median (robust to the outlier we just removed)
df["age"] = df["age"].fillna(df["age"].median())

# Fill missing spend with group median by plan tier
df["annual_spend"] = df["annual_spend"].fillna(
    df.groupby("plan")["annual_spend"].transform("median")
)

# Fill remaining spend NaN (if plan is also missing, group median is NaN)
df["annual_spend"] = df["annual_spend"].fillna(df["annual_spend"].median())

# Fill categorical with "Unknown"
df["city"] = df["city"].fillna("Unknown")
df["name"] = df["name"].fillna("Unknown Customer")

# Drop rows where the primary key is missing — cannot use these rows
df = df.dropna(subset=["customer_id"])
```

---

## Step 10: Detect and Handle Outliers

### IQR method — statistical outlier flagging

```python
def flag_iqr_outliers(series: pd.Series, multiplier: float = 1.5) -> pd.Series:
    """Return a boolean mask: True where value is an IQR outlier."""
    q1 = series.quantile(0.25)
    q3 = series.quantile(0.75)
    iqr = q3 - q1
    lower = q1 - multiplier * iqr
    upper = q3 + multiplier * iqr
    return (series < lower) | (series > upper)


df["age_outlier"] = flag_iqr_outliers(df["age"])
df["spend_outlier"] = flag_iqr_outliers(df["annual_spend"])

print(f"Age outliers: {df['age_outlier'].sum()}")
print(f"Spend outliers: {df['spend_outlier'].sum()}")
```

> [!info] Flagging vs removing outliers
> Flagging is almost always safer than removing. An outlier might be a data entry error — or it might be your most important customer who spent 100x the average. Never silently delete outlier rows. Flag them, investigate them, and then decide with domain knowledge whether to exclude, cap, or keep them.

### Capping outliers (Winsorization)

```python
# Cap outliers at 1st and 99th percentile instead of removing
p01 = df["annual_spend"].quantile(0.01)
p99 = df["annual_spend"].quantile(0.99)

df["annual_spend_capped"] = df["annual_spend"].clip(lower=p01, upper=p99)
```

---

## Step 11: Optimize Memory with Dtype Downcasting

```python
print(f"Memory before: {df.memory_usage(deep=True).sum() / 1024:.1f} KB")

# Downcast integers and floats to smaller types where possible
df["customer_id"] = pd.to_numeric(df["customer_id"], downcast="integer")
df["age"] = pd.to_numeric(df["age"], downcast="integer")

# Float64 to float32 for columns where precision beyond 6 decimals is unnecessary
df["annual_spend"] = df["annual_spend"].astype("float32")

print(f"Memory after: {df.memory_usage(deep=True).sum() / 1024:.1f} KB")
```

On datasets with millions of rows, dtype optimization can cut memory use by 50–70%. An `int64` column that fits in `int16` uses 4x the memory it needs.

---

## Step 12: Validate the Cleaned Data

```python
def validate_customers(df: pd.DataFrame) -> None:
    """Assert business rules on the cleaned DataFrame."""
    errors = []

    if df["customer_id"].isna().any():
        errors.append("customer_id has missing values")

    if df["customer_id"].duplicated().any():
        errors.append(f"customer_id has {df['customer_id'].duplicated().sum()} duplicates")

    if (df["age"].dropna() < 0).any():
        errors.append("age has negative values")

    if (df["age"].dropna() > 120).any():
        errors.append("age has impossible values > 120")

    if (df["annual_spend"].dropna() < 0).any():
        errors.append("annual_spend has negative values")

    if not set(df["plan"].dropna().unique()).issubset({"Basic", "Premium"}):
        errors.append(f"plan has unexpected values: {df['plan'].dropna().unique()}")

    if errors:
        raise ValueError("Validation failed:\n" + "\n".join(f"  - {e}" for e in errors))
    else:
        print("Validation passed. Dataset is clean.")


validate_customers(df)
```

Validation is not optional. It is the test that proves your cleaning worked. In production pipelines, validation failures send alerts rather than silently producing wrong reports.

---

## The Full Pipeline as a Function

```python
import pandas as pd
import numpy as np


def clean_customer_data(raw: pd.DataFrame) -> pd.DataFrame:
    """
    Full cleaning pipeline for customer data.
    Returns a clean DataFrame. Does not mutate the input.
    """
    df = raw.copy()

    # Standardize column names
    df.columns = (
        df.columns.str.strip().str.lower()
        .str.replace(" ", "_", regex=False)
        .str.replace(r"[^a-z0-9_]", "", regex=True)
    )

    # Replace placeholder values
    df = df.replace(["", "N/A", "n/a", "NA", "unknown", "not available", "-"], np.nan)

    # Deduplicate
    df = df.drop_duplicates()
    if "customer_id" in df.columns:
        df["customer_id"] = pd.to_numeric(df["customer_id"], errors="coerce")
        df = df.drop_duplicates(subset=["customer_id"], keep="first")

    # Clean text
    for col in ["name", "city", "plan"]:
        if col in df.columns:
            df[col] = df[col].str.strip().str.title()

    # Convert types
    df["age"] = pd.to_numeric(df["age"], errors="coerce")
    df["annual_spend"] = (
        df["annual_spend"]
        .str.replace(",", "", regex=False)
        .pipe(pd.to_numeric, errors="coerce")
    )
    df["signup_date"] = pd.to_datetime(df["signup_date"], errors="coerce")

    # Fix impossible values
    df.loc[df["age"] < 0, "age"] = np.nan
    df.loc[df["age"] > 120, "age"] = np.nan
    df.loc[df["annual_spend"] < 0, "annual_spend"] = np.nan

    # Missingness indicators
    df["age_was_missing"] = df["age"].isna()
    df["spend_was_missing"] = df["annual_spend"].isna()

    # Fill missing values
    df["age"] = df["age"].fillna(df["age"].median())
    df["annual_spend"] = df["annual_spend"].fillna(df["annual_spend"].median())
    df["city"] = df["city"].fillna("Unknown")
    df["name"] = df["name"].fillna("Unknown Customer")

    # Drop rows missing primary key
    df = df.dropna(subset=["customer_id"])

    # Extract date features
    df["signup_year"] = df["signup_date"].dt.year
    df["signup_month"] = df["signup_date"].dt.month

    return df


cleaned = clean_customer_data(raw)
print(cleaned.head())
print(cleaned.dtypes)
print(cleaned.isna().sum())
```

---

## Cleaning Checklist

Use this to verify you have not skipped a step.

- [ ] Inspected shape, dtypes, head, and missing counts
- [ ] Worked on a copy — raw data is untouched
- [ ] Standardized column names (lowercase, underscored, no special chars)
- [ ] Replaced placeholder strings with NaN
- [ ] Removed exact duplicates
- [ ] Removed key-based duplicates
- [ ] Stripped whitespace from all text columns
- [ ] Standardized casing (`.str.title()` or `.str.lower()`)
- [ ] Fixed known category spelling variations
- [ ] Converted numeric columns with `pd.to_numeric(errors="coerce")`
- [ ] Converted date columns with `pd.to_datetime(errors="coerce")`
- [ ] Applied domain rules to remove impossible values
- [ ] Created missingness indicator columns before filling
- [ ] Filled or dropped missing values with a deliberate strategy
- [ ] Checked for outliers and flagged them
- [ ] Validated the cleaned result against business rules
- [ ] Saved the cleaned output

---

## Common Mistakes

### Cleaning the original DataFrame in place

```python
# Wrong — modifies raw, cannot recover
raw["age"] = pd.to_numeric(raw["age"], errors="coerce")

# Correct — work on a copy
df = raw.copy()
df["age"] = pd.to_numeric(df["age"], errors="coerce")
```

### Filling before deduplicating

If you fill missing values first and then deduplicate, you might discard original rows in favor of filled-in ones. Deduplicate on the original (or replaced-placeholder) data.

### Using one strategy for all missing columns

Names, ages, cities, dates, and IDs all have different semantics. A missing city should be filled with `"Unknown"`. A missing age should be filled with the median. A missing customer ID means the row is unusable and should be dropped. There is no single correct strategy.

### Not validating after cleaning

A cleaning pipeline that does not verify its output is a hypothesis, not a process. Write assertions. Check counts. Inspect the describe() output. Verify that impossible values are gone.

---

> [!success] Key Takeaways
> - Follow a fixed cleaning order — the order prevents steps from undermining each other.
> - Standardize column names first, everything else depends on them.
> - Replace placeholder strings before checking for missing values — `isna()` does not catch `"unknown"`.
> - Deduplicate before filling missing values.
> - Wrap the full pipeline in a function — it becomes a reproducible, testable artifact.
> - Validate the output — cleaning that is not verified is not finished.

---

[[04-missing-values]] — Detect and handle missing data | [[06-exercises]] — Practice advanced Pandas workflows
