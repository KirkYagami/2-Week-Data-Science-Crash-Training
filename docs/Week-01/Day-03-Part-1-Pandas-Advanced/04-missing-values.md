# Missing Values — Detecting and Handling Incomplete Data

Missing data is not a nuisance — it is information. A missing age might mean the user skipped the field. A missing transaction timestamp might mean the payment gateway timed out. A missing sensor reading might mean the device was offline. How you handle missing values changes the conclusions you draw. Handle them wrongly and your model or report is built on a lie.

Most people learn two functions — `dropna()` and `fillna()` — and call it done. This section covers the full picture: how to find missing values, how to understand why they are missing, how to decide between dropping and filling, and how to execute each strategy correctly.

## Learning Objectives

By the end of this section you will be able to:

- Use `.isna()`, `.notna()`, and summary patterns to audit missing data in any DataFrame
- Distinguish between NaN/None/NaT and invalid placeholder values like `"unknown"` or `""`
- Choose between dropping and filling based on the nature and volume of missingness
- Apply five filling strategies — constant, mean/median, mode, forward/backward fill, group-based fill — and know when each is appropriate
- Use `.interpolate()` for time-series missingness
- Preserve missingness as a signal using indicator columns
- Avoid the most common missing-value mistakes that corrupt analysis silently

---

## Sample Dataset

```python
import pandas as pd
import numpy as np

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4, 5, 6, 7],
    "name": ["Alice Sharma", "Bob Menon", None, "Diana Nair", "Evan Patel", "Fatima Khan", "Gopal Singh"],
    "age": [25, np.nan, 32, 29, np.nan, 41, 37],
    "city": ["Delhi", "Mumbai", None, "Pune", "Delhi", None, "Mumbai"],
    "annual_spend": [12000, 25000, np.nan, 18000, 0, 32000, np.nan],
    "plan": ["Premium", "Basic", "Premium", np.nan, "Basic", "Premium", "Basic"],
    "signup_date": pd.to_datetime([
        "2024-01-10", None, "2024-03-05", "2024-03-20",
        "2024-04-15", "2024-05-01", "2024-06-12"
    ])
})
```

---

## Step 1: Audit Before You Act

Never fill or drop anything until you understand what is missing and how much.

```python
# Column-level missing count
print(customers.isna().sum())
# Output:
# customer_id     0
# name            1
# age             2
# city            2
# annual_spend    2
# plan            1
# signup_date     1
# dtype: int64

# Missing percentage — often more useful than raw count
missing_pct = (customers.isna().mean() * 100).round(1)
print(missing_pct)
# Output:
# customer_id      0.0
# name            14.3
# age             28.6
# city            28.6
# annual_spend    28.6
# plan            14.3
# signup_date     14.3
# dtype: float64

# Shape context
print(f"DataFrame shape: {customers.shape}")
# Output: DataFrame shape: (7, 7)
```

```python
# Which rows have at least one missing value?
rows_with_any_missing = customers[customers.isna().any(axis=1)]
print(rows_with_any_missing[["customer_id", "name", "age", "city"]])
# Output:
#    customer_id  name   age   city
# 1            2  Bob Menon  NaN  Mumbai
# 2            3       None  32.0    None
# 5            6  Fatima Khan  41.0    None
# 6            7  Gopal Singh  37.0  Mumbai
# (rows 1, 2, 5 — various columns missing)

# Which rows have ALL values missing in a subset?
rows_missing_critical = customers[customers[["name", "city"]].isna().all(axis=1)]
```

---

## Missing vs Invalid: The Distinction That Matters

True missing values are `NaN`, `None`, or `NaT`. But real data has a third category: values that are present but meaningless.

- `age = -5` — present, but physically impossible
- `annual_spend = "unknown"` — present, but not usable as a number
- `city = ""` — present, but an empty string is not a city
- `signup_date = "not available"` — present, but not parseable as a date

These need to be converted to real missing values before any analysis.

```python
raw = pd.DataFrame({
    "customer_id": [1, 2, 3, 4, 5],
    "age": ["25", "-5", "unknown", "32", ""],
    "spend": ["1,200", "N/A", "2500", "-100", "missing"],
    "signup_date": ["2024-01-10", "bad date", "2024-03-05", "N/A", "2024-05-01"]
})

# Replace common placeholder strings with NaN
raw = raw.replace({
    "": np.nan,
    "N/A": np.nan,
    "n/a": np.nan,
    "NA": np.nan,
    "unknown": np.nan,
    "missing": np.nan,
    "not available": np.nan,
    "-": np.nan
})

# Or do it at read time
# df = pd.read_csv("data.csv", na_values=["", "N/A", "unknown", "missing", "-"])

# Convert types with coerce — invalid strings become NaN automatically
raw["age"] = pd.to_numeric(raw["age"], errors="coerce")
raw["spend"] = (
    raw["spend"]
    .str.replace(",", "", regex=False)  # remove thousands separator first
    .pipe(pd.to_numeric, errors="coerce")
)
raw["signup_date"] = pd.to_datetime(raw["signup_date"], errors="coerce")

# Fix impossible values
raw.loc[raw["age"] < 0, "age"] = np.nan
raw.loc[raw["spend"] < 0, "spend"] = np.nan

print(raw.isna().sum())
```

> [!warning] `isna()` does not catch invalid strings
> `customers["age"].isna()` returns False for the string `"unknown"` — because `"unknown"` is not NaN, it is a string. You must first convert or replace placeholder strings before relying on `.isna()` to find missing data.

---

## Dropping Missing Values

### Drop rows where any column is missing

```python
clean = customers.dropna()
print(f"Original: {len(customers)} rows -> After dropna(): {len(clean)} rows")
# Output: Original: 7 rows -> After dropna(): 2 rows
# Dropping on any column is aggressive — here it removes 5 of 7 rows
```

Use with caution. On real datasets, `dropna()` with no arguments often destroys most of your data.

### Drop rows missing specific critical columns

```python
# Only drop rows where name is missing — keep the rest
clean = customers.dropna(subset=["name"])
print(f"After dropping missing name: {len(clean)} rows")
# Output: After dropping missing name: 6 rows

# Drop rows missing any of several key columns
clean = customers.dropna(subset=["customer_id", "name"])
```

### Drop with `how` and `thresh`

```python
# how="all": only drop rows where EVERY specified column is missing
customers.dropna(how="all")

# thresh=N: keep rows with at least N non-null values
customers.dropna(thresh=5)  # keep rows with at least 5 non-null values out of 7

# Drop columns that are mostly missing
threshold = int(len(customers) * 0.7)  # column must be 70% complete to keep
customers.dropna(axis=1, thresh=threshold)
```

> [!tip] Use `subset` + `thresh` instead of plain `dropna()`
> Plain `dropna()` is almost never what you want in practice. Define which columns are truly mandatory (`subset=`) and what completeness minimum you accept (`thresh=`). This gives you precise control over what gets dropped.

---

## Filling Missing Values

### Strategy 1: Fill with a constant

```python
# Categorical columns — fill with a label that makes the missing explicit
customers["city"] = customers["city"].fillna("Unknown")
customers["plan"] = customers["plan"].fillna("No Plan")

# Or use the .fillna() dict form to fill multiple columns at once
customers = customers.fillna({
    "city": "Unknown",
    "plan": "No Plan",
    "name": "Unnamed Customer"
})
```

### Strategy 2: Fill numeric columns with mean or median

```python
# Median is more robust than mean when outliers exist
age_median = customers["age"].median()
customers["age"] = customers["age"].fillna(age_median)

# Mean is fine when data is roughly symmetric and outlier-free
spend_mean = customers["annual_spend"].mean()
customers["annual_spend"] = customers["annual_spend"].fillna(spend_mean)

print(f"Age median used for fill: {age_median}")
print(f"Spend mean used for fill: {spend_mean:.0f}")
```

> [!info] Median vs Mean for imputation
> Median is the safer default for numerical imputation. It is not affected by extreme values. If `annual_spend` has one customer who spent ₹10,00,000 and ten who spent ₹1,000, the mean is driven up by that one outlier. The median is not. Use mean only when you have verified the distribution is roughly symmetric and free of extreme values.

### Strategy 3: Fill with mode (most common value)

```python
most_common_city = customers["city"].mode()[0]
customers["city"] = customers["city"].fillna(most_common_city)

print(f"Mode used for city fill: {most_common_city}")
```

Note the `[0]` — `.mode()` returns a Series because there can be ties. Take the first one.

### Strategy 4: Group-based fill (smarter imputation)

Filling with a global mean ignores structure in the data. If customers in Delhi are systematically younger than customers in Mumbai, using a single global age median for everyone introduces bias.

```python
# Fill missing age with the median age for each city
customers["age"] = customers["age"].fillna(
    customers.groupby("city")["age"].transform("median")
)

# Fill missing spend with the median spend per plan tier
customers["annual_spend"] = customers["annual_spend"].fillna(
    customers.groupby("plan")["annual_spend"].transform("median")
)
```

This is where `.transform()` shines — it produces group-level values aligned to the original index, so you can use it directly in `.fillna()`.

> [!tip] Group-based imputation is almost always better than global imputation
> If your data has meaningful subgroups (by category, region, time period), imputing within groups produces more realistic fill values. The extra one line of code is worth it.

### Strategy 5: Forward fill and backward fill (time series)

When data is ordered by time, missing values often mean "unchanged from the last known value."

```python
sensor_data = pd.DataFrame({
    "timestamp": pd.date_range("2024-01-01", periods=8, freq="H"),
    "temperature": [22.5, np.nan, np.nan, 23.1, np.nan, 22.8, np.nan, 23.5]
})

# Forward fill: carry the last known value forward
sensor_data["temp_ffill"] = sensor_data["temperature"].ffill()

# Backward fill: use the next known value to fill backward
sensor_data["temp_bfill"] = sensor_data["temperature"].bfill()

print(sensor_data[["timestamp", "temperature", "temp_ffill", "temp_bfill"]])
# Output:
#             timestamp  temperature  temp_ffill  temp_bfill
# 0 2024-01-01 00:00:00         22.5        22.5        22.5
# 1 2024-01-01 01:00:00          NaN        22.5        23.1
# 2 2024-01-01 02:00:00          NaN        22.5        23.1
# 3 2024-01-01 03:00:00         23.1        23.1        23.1
# 4 2024-01-01 04:00:00          NaN        23.1        22.8
# ...
```

> [!warning] Only use ffill/bfill on time-ordered data
> Forward fill on an unordered DataFrame propagates arbitrary values. Always sort by timestamp before applying `ffill()` or `bfill()` to time series data.

### Strategy 6: Interpolation (time series)

Interpolation estimates missing values by fitting a line (or curve) between known values. It is better than ffill when the true value is expected to change gradually.

```python
sensor_data["temp_linear"] = sensor_data["temperature"].interpolate(method="linear")

print(sensor_data[["timestamp", "temperature", "temp_linear"]])
# Output:
#             timestamp  temperature  temp_linear
# 0 2024-01-01 00:00:00         22.5    22.500000
# 1 2024-01-01 01:00:00          NaN    22.700000
# 2 2024-01-01 02:00:00          NaN    22.900000
# 3 2024-01-01 03:00:00         23.1    23.100000
# 4 2024-01-01 04:00:00          NaN    22.966667
# ...
```

---

## Preserving Missingness as a Signal

Sometimes the fact that a value was missing is itself informative. A customer who did not fill in their age behaves differently from one who did. Dropping or filling that information before modeling destroys the signal.

The fix: create an indicator column before you fill.

```python
# Create indicators BEFORE filling
customers["age_was_missing"] = customers["age"].isna()
customers["spend_was_missing"] = customers["annual_spend"].isna()
customers["city_was_missing"] = customers["city"].isna()

# Now fill
customers["age"] = customers["age"].fillna(customers["age"].median())
customers["annual_spend"] = customers["annual_spend"].fillna(customers["annual_spend"].median())
customers["city"] = customers["city"].fillna("Unknown")
```

In machine learning, these indicator columns often improve model performance for free. In analysis, they let you segment "customers who gave us their age" vs "customers who did not."

---

## When to Drop vs When to Fill

This requires judgment, not a formula. The framework:

| Situation | Recommendation |
|-----------|---------------|
| Missing value in a key/ID column | Drop the row — you cannot reliably attribute the data |
| Column is >80% missing | Drop the column — mostly noise |
| Column is <5% missing, numeric | Fill with median/mean |
| Column is <5% missing, categorical | Fill with mode or "Unknown" |
| Missing correlates with another column | Group-based fill |
| Time-ordered data | ffill, bfill, or interpolate |
| Missingness is itself meaningful | Create indicator, then fill |

> [!warning] Never fill with zero unless zero is a valid business value
> Filling `annual_spend` with 0 makes those customers look like they spent nothing. If the value is genuinely missing (unknown), that is very different from "confirmed zero spend." Filling with zero corrupts the mean, median, and any model that uses the column.

---

## NA Propagation Rules

```python
# NaN is contagious in arithmetic
print(np.nan + 5)       # Output: nan
print(np.nan > 5)       # Output: False (comparison with NaN is always False)
print(np.nan == np.nan) # Output: False — this is why isna() exists

# Pandas aggregation functions skip NaN by default
s = pd.Series([1, 2, np.nan, 4, 5])
print(s.sum())    # Output: 12.0 (skips NaN)
print(s.mean())   # Output: 3.0  (mean of 1,2,4,5)
print(s.count())  # Output: 4    (excludes NaN)
```

> [!info] `.sum()` on an all-NaN column returns 0, not NaN
> This surprises many people. `pd.Series([np.nan, np.nan]).sum()` returns `0.0`, not NaN. If you are checking whether an aggregation failed due to all-missing data, use `.count()` (which returns 0) or `.isna().all()` rather than checking if the sum is NaN.

---

## Full Audit Workflow

```python
def audit_missing(df: pd.DataFrame) -> pd.DataFrame:
    """Return a summary of missing data for each column."""
    total = len(df)
    missing_count = df.isna().sum()
    missing_pct = (missing_count / total * 100).round(1)
    dtypes = df.dtypes

    audit = pd.DataFrame({
        "missing_count": missing_count,
        "missing_pct": missing_pct,
        "dtype": dtypes
    })

    return audit.sort_values("missing_pct", ascending=False)


print(audit_missing(customers))
# Output:
#               missing_count  missing_pct   dtype
# age                       2         28.6   float64
# city                      2         28.6    object
# annual_spend              2         28.6   float64
# name                      1         14.3    object
# plan                      1         14.3    object
# signup_date               1         14.3  datetime64[ns]
# customer_id               0          0.0     int64
```

---

> [!success] Key Takeaways
> - Audit before acting: always check missing counts and percentages by column before touching any data.
> - `.isna()` only catches true NaN/None/NaT — convert placeholder strings first.
> - Prefer `dropna(subset=["critical_column"])` over plain `dropna()`, which destroys too much data.
> - Use group-based filling (`transform("median")`) when subgroups have different distributions.
> - Create indicator columns (`col_was_missing`) before filling when missingness carries information.
> - Never fill numeric missing values with 0 unless zero is a valid, meaningful value.

---

[[03-apply-functions]] — Apply custom logic to rows and columns | [[05-real-world-data-cleaning]] — Combine cleaning skills into a full pipeline
