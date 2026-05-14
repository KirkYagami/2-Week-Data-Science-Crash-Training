# Pandas Cheat Sheet

Pandas is the workhorse of tabular data in Python. This sheet is dense by design — every entry has a why and a runnable example. Use it as a lookup reference while working on projects.

---

## Creating DataFrames

### From a dictionary

When your data already lives in Python as named lists or arrays, a dict is the most natural starting point. Keys become column names.

```python
import pandas as pd
import numpy as np

data = {
    "name": ["Alice", "Bob", "Carol"],
    "age": [32, 25, 29],
    "salary": [90000, 72000, 85000],
}
df = pd.DataFrame(data)
print(df)
# Output:
#     name  age  salary
# 0  Alice   32   90000
# 1    Bob   25   72000
# 2  Carol   29   85000
```

### From a list of dicts

Useful when each record is already a dict — common when parsing JSON API responses or iterating database rows.

```python
records = [
    {"product": "Widget A", "units": 120, "revenue": 2400.0},
    {"product": "Widget B", "units": 85,  "revenue": 1700.0},
    {"product": "Widget C", "units": 200, "revenue": 3800.0},
]
products = pd.DataFrame(records)
# Missing keys in any record become NaN automatically
```

### From CSV or Excel

The most common entry point in practice. Always check `sep`, `encoding`, and `parse_dates` before loading.

```python
# CSV
sales = pd.read_csv("sales_data.csv", parse_dates=["order_date"], encoding="utf-8")

# Excel — specify the sheet name when the file has multiple sheets
budget = pd.read_excel("budget.xlsx", sheet_name="Q1", engine="openpyxl")
```

### From a NumPy array

Use this when you have numerical results from NumPy computations and want to attach column labels.

```python
matrix = np.random.randn(4, 3)
df_np = pd.DataFrame(matrix, columns=["feature_a", "feature_b", "feature_c"])
print(df_np.round(3))
# Each column is named; index defaults to 0, 1, 2, 3
```

### Setting a meaningful index at creation

A named index speeds up lookups and makes merge keys explicit. Set it at creation rather than resetting it later.

```python
df = pd.DataFrame(data).set_index("name")
print(df.loc["Alice"])
# Output:
# age       32
# salary    90000
# Name: Alice, dtype: int64
```

---

## Inspection

### .head() and .tail()

Always call these first after loading. They give you a quick sanity check on column names, dtypes, and whether the load parsed correctly.

```python
df = pd.read_csv("sales_data.csv")
df.head(5)   # first 5 rows
df.tail(3)   # last 3 rows — check for trailing blank rows or footer lines
```

### .info()

Shows dtypes, non-null counts, and memory usage in one call. The fastest way to spot columns that have been read as `object` when they should be numeric or datetime.

```python
df.info()
# Output (example):
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 1000 entries, 0 to 999
# Data columns (total 5 columns):
#  #   Column      Non-Null Count  Dtype
# ---  ------      --------------  -----
#  0   order_id    1000 non-null   int64
#  1   order_date  980 non-null    object   ← should be datetime
#  2   customer    1000 non-null   object
#  3   revenue     995 non-null    float64
#  4   region      1000 non-null   object
```

### .describe()

Summary statistics for numeric columns. Use `include="all"` to include categorical columns too.

```python
df.describe()
# For categoricals:
df.describe(include="all")
# Use percentiles= to see the tails more clearly
df["revenue"].describe(percentiles=[0.01, 0.25, 0.75, 0.99])
```

### .shape, .dtypes, .columns, .index

Quick accessors you reach for constantly.

```python
df.shape       # (1000, 5) — rows, columns
df.dtypes      # Series mapping column name → dtype
df.columns     # Index(['order_id', 'order_date', ...])
df.index       # RangeIndex(start=0, stop=1000, step=1)

# Check one dtype
df["revenue"].dtype   # dtype('float64')
```

### .value_counts()

The fastest way to understand the distribution of a categorical column. More informative than `.describe()` for strings.

```python
df["region"].value_counts()
# Output:
# West      312
# East      289
# South     210
# North     189
# Name: region, dtype: int64

df["region"].value_counts(normalize=True).round(2)  # as proportions
```

---

## Selection

### .loc[] — label-based selection

Use `.loc` when you know the row label (index value) and column name. It is explicit and readable — prefer it over chained bracket access.

```python
# Single row by label
df.loc[42]

# Row range by label, specific columns
df.loc[10:20, ["customer", "revenue"]]

# All rows, specific columns
df.loc[:, "revenue"]
```

### .iloc[] — position-based selection

Use `.iloc` when you need to select by integer position — useful in loops, ML train/test splits, or when the index is non-default.

```python
# First row
df.iloc[0]

# First 5 rows, first 3 columns
df.iloc[:5, :3]

# Last row
df.iloc[-1]

# Every other row
df.iloc[::2]
```

### Boolean filtering

The most common selection pattern in data cleaning. Build a boolean mask, then apply it.

```python
# Single condition
high_revenue = df[df["revenue"] > 5000]

# Multiple conditions — wrap each in parentheses; use & (and), | (or), ~ (not)
west_high = df[(df["region"] == "West") & (df["revenue"] > 5000)]

# Negation
not_west = df[~(df["region"] == "West")]
```

### .query()

More readable than chained boolean masks when you have several conditions. Accepts column names directly without `df["..."]` syntax.

```python
result = df.query("region == 'West' and revenue > 5000")

# Use @ to reference a Python variable inside the query string
threshold = 5000
result = df.query("revenue > @threshold and region != 'North'")
```

### Selecting multiple columns

Returns a DataFrame, not a Series. Common mistake is using `df["col1", "col2"]` (raises KeyError) instead of `df[["col1", "col2"]]`.

```python
subset = df[["customer", "revenue", "region"]]

# Or use .loc for the same result with label slicing
subset = df.loc[:, ["customer", "revenue", "region"]]
```

---

## Filtering & Sorting

### .isin()

Cleaner than chaining multiple `==` conditions with `|`. Use when testing membership in a known list.

```python
target_regions = ["West", "East"]
df_target = df[df["region"].isin(target_regions)]

# Inverse: rows NOT in the list
df_other = df[~df["region"].isin(target_regions)]
```

### .between()

Inclusive range filter on numeric or datetime columns. More readable than `(df["col"] >= a) & (df["col"] <= b)`.

```python
mid_revenue = df[df["revenue"].between(1000, 5000)]

# Works on dates too
import pandas as pd
df_q1 = df[df["order_date"].between("2024-01-01", "2024-03-31")]
```

### .sort_values()

Sort by one or more columns. Always specify `ascending` explicitly when the direction matters for correctness.

```python
# Single column, descending
df_sorted = df.sort_values("revenue", ascending=False)

# Multiple columns: primary sort by region, secondary by revenue desc
df_sorted = df.sort_values(["region", "revenue"], ascending=[True, False])

# Sort and reset the integer index
df_sorted = df.sort_values("revenue", ascending=False).reset_index(drop=True)
```

### .nlargest() / .nsmallest()

Faster than sort + head for finding top-N records on a single column. Skips sorting the rest of the DataFrame.

```python
top5 = df.nlargest(5, "revenue")
bottom5 = df.nsmallest(5, "revenue")

# Keep all ties at the boundary
top5_all = df.nlargest(5, "revenue", keep="all")
```

### Filtering with string patterns

Combine boolean indexing with `.str` methods for text-based filters.

```python
# Rows where customer name starts with "A"
df[df["customer"].str.startswith("A")]

# Case-insensitive contains
df[df["customer"].str.contains("corp", case=False, na=False)]
```

---

## Missing Data

### .isna() / .notna()

Returns a boolean DataFrame or Series. Use `.sum()` to count missing values per column — this is the first thing to check in any EDA.

```python
df.isna().sum()
# Output:
# order_id       0
# order_date    20
# customer       0
# revenue        5
# region         0
# dtype: int64

# Percentage missing
(df.isna().sum() / len(df) * 100).round(2)

# Rows with any missing value
df[df.isna().any(axis=1)]
```

### .dropna()

Remove rows or columns with missing data. Be deliberate — dropping too aggressively can introduce bias.

```python
# Drop rows with any NaN
df_clean = df.dropna()

# Drop rows only if a specific column is null
df_clean = df.dropna(subset=["revenue"])

# Drop columns that are entirely null
df_clean = df.dropna(axis=1, how="all")

# Keep rows with at least N non-null values
df_clean = df.dropna(thresh=4)
```

### .fillna()

Fill missing values with a scalar, a dict of per-column values, or a method.

```python
# Fill all NaNs with zero
df["revenue"] = df["revenue"].fillna(0)

# Fill with column mean
df["revenue"] = df["revenue"].fillna(df["revenue"].mean())

# Per-column fill using a dict
df = df.fillna({"revenue": 0, "region": "Unknown"})

# Forward fill (carry last valid observation forward) — common in time series
df["price"] = df["price"].fillna(method="ffill")
```

### .interpolate()

Better than `.fillna(mean)` for time-series gaps — it estimates values based on neighboring points rather than a global statistic.

```python
# Linear interpolation (default)
df["temperature"] = df["temperature"].interpolate(method="linear")

# Polynomial — useful when the signal has curvature
df["temperature"] = df["temperature"].interpolate(method="polynomial", order=2)
```

### Detecting patterns in missingness

Before filling, check whether missing values are random or systematic — systematic patterns often mean a data pipeline problem.

```python
# Are nulls in revenue correlated with region?
df.groupby("region")["revenue"].apply(lambda x: x.isna().sum())
```

---

## String Operations

### .str.lower() / .str.upper() / .str.strip()

Normalize strings before merging or grouping. Mismatched cases and trailing whitespace are silent killers in joins.

```python
df["customer"] = df["customer"].str.lower().str.strip()
df["region"] = df["region"].str.title()  # Title Case
```

### .str.contains()

Pattern matching using regex or literal strings. Always pass `na=False` to avoid NaN propagation.

```python
# Rows where customer contains "tech" (case-insensitive)
tech_customers = df[df["customer"].str.contains("tech", case=False, na=False)]

# Regex: starts with digit
df[df["order_id_str"].str.contains(r"^\d", na=False)]
```

### .str.split()

Split a column on a delimiter. Use `expand=True` to get a DataFrame of columns instead of a Series of lists.

```python
# "Alice Smith" → two columns: first_name, last_name
name_parts = df["full_name"].str.split(" ", expand=True)
name_parts.columns = ["first_name", "last_name"]
df = pd.concat([df, name_parts], axis=1)
```

### .str.replace()

Substitute patterns using regex or literal strings. Use regex=False when replacing literal characters to avoid accidental pattern matches.

```python
# Remove dollar signs and commas before casting to float
df["price"] = df["price"].str.replace(r"[\$,]", "", regex=True).astype(float)

# Normalize inconsistent spellings
df["region"] = df["region"].str.replace("Sth", "South", regex=False)
```

### .str.extract()

Pull captured groups out of a regex into new columns. Useful for parsing structured strings like timestamps, product codes, or log entries.

```python
# Extract area code from phone numbers like "(415) 555-1234"
df["area_code"] = df["phone"].str.extract(r"\((\d{3})\)")

# Multiple capture groups → multiple columns
df[["year", "month", "day"]] = df["date_str"].str.extract(r"(\d{4})-(\d{2})-(\d{2})")
```

---

## GroupBy & Aggregation

### Basic .groupby()

Split-apply-combine: group rows by one or more columns, then apply an aggregation. The most common pattern in exploratory analysis.

```python
# Total revenue per region
df.groupby("region")["revenue"].sum()

# Multiple aggregations on one column
df.groupby("region")["revenue"].agg(["sum", "mean", "count"])

# Group by multiple columns
df.groupby(["region", "product"])["revenue"].sum()
```

### Named aggregations with .agg()

Produce clean output column names in a single call instead of renaming afterwards. Requires pandas >= 0.25.

```python
summary = df.groupby("region").agg(
    total_revenue=("revenue", "sum"),
    avg_revenue=("revenue", "mean"),
    order_count=("order_id", "count"),
    max_order=("revenue", "max"),
)
print(summary)
# Output:
#         total_revenue  avg_revenue  order_count  max_order
# region
# East         ...            ...          ...        ...
```

### .transform()

Returns a Series aligned to the original DataFrame's index — essential for adding group-level statistics as new columns without collapsing rows.

```python
# Add a column showing each region's total revenue alongside each row
df["region_total"] = df.groupby("region")["revenue"].transform("sum")

# Z-score within group (normalize each group independently)
df["revenue_zscore"] = df.groupby("region")["revenue"].transform(
    lambda x: (x - x.mean()) / x.std()
)
```

### .apply() on groups

Use when built-in aggregations are not flexible enough. The function receives a sub-DataFrame for each group.

```python
def top_two_customers(group):
    return group.nlargest(2, "revenue")

top_customers = df.groupby("region").apply(top_two_customers).reset_index(drop=True)
```

### .nunique() and .size()

Counting distinct values within groups is common in customer analytics. `.size()` counts all rows; `.count()` skips NaN; `.nunique()` counts distinct non-null values.

```python
# Distinct customers per region
df.groupby("region")["customer"].nunique()

# Number of orders per region (including NaN rows)
df.groupby("region").size()
```

---

## Merging & Joining

### pd.merge() — the standard join

Works like SQL joins. Prefer `pd.merge()` over `.join()` unless both DataFrames share the same index.

```python
orders = pd.DataFrame({
    "order_id": [1, 2, 3, 4],
    "customer_id": [101, 102, 101, 103],
    "revenue": [500, 800, 300, 150],
})
customers = pd.DataFrame({
    "customer_id": [101, 102, 104],
    "name": ["Alice", "Bob", "Dave"],
    "region": ["West", "East", "West"],
})

# Inner join — only rows with matching keys on both sides
result = pd.merge(orders, customers, on="customer_id", how="inner")

# Left join — all orders, NaN for unmatched customer info
result = pd.merge(orders, customers, on="customer_id", how="left")
```

### Merge on different column names

When the key column has different names in each DataFrame, use `left_on` and `right_on`.

```python
result = pd.merge(
    orders,
    customers,
    left_on="customer_id",
    right_on="cust_id",
    how="inner",
)
# Drop the redundant key column after the merge
result = result.drop(columns="cust_id")
```

### pd.concat()

Stack DataFrames vertically (axis=0) or horizontally (axis=1). Use for combining datasets that share the same schema — for example, monthly exports stacked into an annual view.

```python
# Vertical stack (same columns)
jan = pd.read_csv("jan_sales.csv")
feb = pd.read_csv("feb_sales.csv")
all_sales = pd.concat([jan, feb], axis=0, ignore_index=True)

# Horizontal stack (same rows, different columns)
df_combined = pd.concat([df_features, df_labels], axis=1)
```

### Merge types cheat summary

```python
# Inner:  rows present in BOTH DataFrames
# Left:   all rows from left, NaN for missing right-side data
# Right:  all rows from right, NaN for missing left-side data
# Outer:  all rows from both, NaN wherever there is no match

result = pd.merge(df_a, df_b, on="key", how="inner")
result = pd.merge(df_a, df_b, on="key", how="left")
result = pd.merge(df_a, df_b, on="key", how="right")
result = pd.merge(df_a, df_b, on="key", how="outer")

# Validate the merge — catch unexpected duplicates
result = pd.merge(df_a, df_b, on="key", how="left", validate="many_to_one")
```

### Detecting merge quality

Always check the row count and NaN counts after a merge to confirm it behaved as expected.

```python
print(f"Left rows: {len(orders)}, Merged rows: {len(result)}")
result.isna().sum()  # NaNs in right-side columns reveal unmatched left keys
```

---

## Reshaping

### .pivot_table()

The go-to for cross-tabular summaries. Think of it as a configurable GroupBy that produces a 2D grid. More flexible than `.pivot()` because it handles duplicate entries.

```python
pivot = df.pivot_table(
    values="revenue",
    index="region",
    columns="product",
    aggfunc="sum",
    fill_value=0,
)
# Output: regions as rows, products as columns, total revenue in cells
```

### .melt()

The inverse of pivot — converts wide format to long format. Use before feeding data into seaborn or scikit-learn, which expect long format.

```python
# Wide: one column per month
wide = pd.DataFrame({
    "region": ["East", "West"],
    "Jan": [1000, 1500],
    "Feb": [1100, 1600],
    "Mar": [900, 1400],
})

long = pd.melt(wide, id_vars=["region"], var_name="month", value_name="revenue")
print(long)
# Output:
#   region month  revenue
# 0   East   Jan     1000
# 1   West   Jan     1500
# 2   East   Feb     1100
# ...
```

### .stack() / .unstack()

Rotate column level to row index (stack) or row index to column level (unstack). Most useful when working with MultiIndex DataFrames.

```python
# Stack: move the innermost column level into the row index
stacked = pivot.stack()

# Unstack: move the innermost row index level into columns
unstacked = stacked.unstack()
```

### pd.crosstab()

Quick frequency table between two categorical variables. A shorthand for `pivot_table` with `aggfunc="count"` — useful in EDA to check for class imbalance or co-occurrence patterns.

```python
ct = pd.crosstab(df["region"], df["product"])
# Normalized to show row percentages
ct_pct = pd.crosstab(df["region"], df["product"], normalize="index").round(2)
```

### .pivot() — simple reshape without aggregation

Use `.pivot()` (not `.pivot_table()`) when there are no duplicate index-column pairs to aggregate.

```python
# Reshape log data: one row per (date, metric) pair → one column per metric
df_wide = df_long.pivot(index="date", columns="metric", values="value")
```

---

## Apply & Map

### .apply() on a Series

Applies a function element-wise to a Series. Use for transformations that cannot be expressed as vectorized NumPy operations.

```python
# Custom categorization
def revenue_tier(x):
    if x >= 10000:
        return "High"
    elif x >= 5000:
        return "Medium"
    return "Low"

df["tier"] = df["revenue"].apply(revenue_tier)

# Lambda for simple one-liners
df["revenue_k"] = df["revenue"].apply(lambda x: round(x / 1000, 1))
```

### .apply() on a DataFrame (axis=1)

Applies a function row-wise. Slower than vectorized operations — only use when you genuinely need multiple columns to compute the result.

```python
def flag_row(row):
    if row["revenue"] > 5000 and row["region"] == "West":
        return "Priority"
    return "Standard"

df["flag"] = df.apply(flag_row, axis=1)
```

### .map() on a Series

Maps values using a dict or function. The idiomatic way to recode or relabel a categorical column.

```python
region_codes = {"West": "W", "East": "E", "North": "N", "South": "S"}
df["region_code"] = df["region"].map(region_codes)
# Values not in the dict become NaN — use .fillna() if needed
```

### Prefer vectorized operations over .apply()

`.apply()` is a Python loop under the hood — it is 10–100x slower than equivalent NumPy or Pandas vectorized operations. Use it only when no vectorized alternative exists.

```python
# Slow: apply
df["revenue_log"] = df["revenue"].apply(np.log)

# Fast: vectorized (same result)
df["revenue_log"] = np.log(df["revenue"])

# Slow: apply for string operations
df["customer_upper"] = df["customer"].apply(str.upper)

# Fast: .str accessor
df["customer_upper"] = df["customer"].str.upper()
```

### DataFrame.map() — element-wise on a DataFrame

Replaces the deprecated `.applymap()`. Applies a function to every element in a DataFrame. Use for formatting or type coercion across the whole table.

```python
# Format all floats to 2 decimal places
df_display = df.select_dtypes(include="float").map(lambda x: f"{x:.2f}")

# Replace values using a dict across all columns
df_recoded = df.map(lambda x: x.strip() if isinstance(x, str) else x)
```

---

## DateTime

### pd.to_datetime()

Convert a string or numeric column to a proper datetime dtype. Always do this after loading — leaving dates as strings breaks every time-based operation.

```python
df["order_date"] = pd.to_datetime(df["order_date"])

# Non-standard format — specify explicitly
df["order_date"] = pd.to_datetime(df["order_date"], format="%d/%m/%Y")

# Unix timestamps (seconds since epoch)
df["event_time"] = pd.to_datetime(df["epoch_seconds"], unit="s")

# Coerce unparseable values to NaT instead of raising an error
df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")
```

### .dt accessor

Access datetime components without looping. Use these to create features for ML models or to filter by time period.

```python
df["year"] = df["order_date"].dt.year
df["month"] = df["order_date"].dt.month
df["day_of_week"] = df["order_date"].dt.day_name()   # "Monday", "Tuesday", ...
df["quarter"] = df["order_date"].dt.quarter
df["is_weekend"] = df["order_date"].dt.dayofweek >= 5

# Time components
df["hour"] = df["event_time"].dt.hour
df["minute"] = df["event_time"].dt.minute
```

### Date arithmetic

Pandas datetime arithmetic returns `Timedelta` objects — subtract two datetime columns to get duration.

```python
df["delivery_days"] = (df["delivery_date"] - df["order_date"]).dt.days

# Add or subtract fixed durations
df["followup_date"] = df["order_date"] + pd.Timedelta(days=30)

# Business day offset
from pandas.tseries.offsets import BDay
df["next_biz_day"] = df["order_date"] + BDay(1)
```

### Resampling

Aggregate a time-indexed DataFrame to a different frequency — the time-series equivalent of GroupBy.

```python
# Set the datetime column as the index first
df = df.set_index("order_date")

# Monthly total revenue
monthly = df["revenue"].resample("ME").sum()

# Weekly mean, forward-fill missing weeks
weekly = df["revenue"].resample("W").mean().fillna(method="ffill")

# Resample with multiple aggregations
monthly_summary = df["revenue"].resample("ME").agg(["sum", "mean", "count"])
```

### Filtering by date range

```python
# Using string slicing on a DatetimeIndex
df_q1 = df.loc["2024-01":"2024-03"]

# Or with boolean mask on a regular column
df_q1 = df[df["order_date"].between("2024-01-01", "2024-03-31")]
```

---

## Window Functions

### .rolling()

Compute statistics over a sliding window — the standard tool for smoothing noisy time series and computing moving averages.

```python
# 7-day rolling mean
df["revenue_7d_avg"] = df["revenue"].rolling(window=7).mean()

# Rolling standard deviation (volatility)
df["revenue_7d_std"] = df["revenue"].rolling(window=7).std()

# Minimum window size before computing (avoids NaN-heavy start)
df["revenue_7d_avg"] = df["revenue"].rolling(window=7, min_periods=3).mean()
```

### .expanding()

Like `.rolling()` but the window grows with each row — starts at the first observation and includes all preceding data. Use for cumulative statistics.

```python
# Cumulative mean up to each point in time
df["cumulative_avg"] = df["revenue"].expanding().mean()

# Running maximum
df["running_max"] = df["revenue"].expanding().max()

# Running total (same as .cumsum() but more flexible)
df["cumulative_revenue"] = df["revenue"].expanding().sum()
```

### .ewm() — exponentially weighted moving average

More responsive to recent data than a simple rolling mean. Standard in finance and anomaly detection. `span` controls how quickly older values decay.

```python
# Span of 7 ≈ 7-period half-life
df["revenue_ema"] = df["revenue"].ewm(span=7, adjust=False).mean()

# Compare: 7-day simple vs exponential moving average
df["sma_7"] = df["revenue"].rolling(7).mean()
df["ema_7"] = df["revenue"].ewm(span=7).mean()
```

### Rolling with a custom function

When built-in aggregations are insufficient, pass a function to `.rolling().apply()`. Note: this is slower than built-in methods.

```python
import numpy as np

# 30-day rolling Sharpe ratio (mean / std)
def sharpe(returns):
    return returns.mean() / returns.std() if returns.std() != 0 else 0

df["rolling_sharpe"] = df["daily_return"].rolling(30).apply(sharpe, raw=True)
```

---

## Saving & Loading

### .to_csv() / pd.read_csv()

The universal interchange format. Use `index=False` when saving unless the index carries meaningful information — otherwise you get an unnamed index column on reload.

```python
# Save
df.to_csv("output.csv", index=False, encoding="utf-8")

# Load with common options
df = pd.read_csv(
    "output.csv",
    parse_dates=["order_date"],
    dtype={"order_id": str},      # force string to preserve leading zeros
    encoding="utf-8",
    na_values=["N/A", "-", ""],   # additional strings to treat as NaN
)
```

### .read_excel() / .to_excel()

Use `openpyxl` for `.xlsx`. Specify `sheet_name=None` to load all sheets at once as a dict of DataFrames.

```python
# Load one sheet
df = pd.read_excel("report.xlsx", sheet_name="Sales", engine="openpyxl")

# Load all sheets
all_sheets = pd.read_excel("report.xlsx", sheet_name=None, engine="openpyxl")
# all_sheets is a dict: {"Sales": df1, "Returns": df2, ...}

# Save multiple sheets
with pd.ExcelWriter("summary.xlsx", engine="openpyxl") as writer:
    df_sales.to_excel(writer, sheet_name="Sales", index=False)
    df_returns.to_excel(writer, sheet_name="Returns", index=False)
```

### .to_parquet() / .read_parquet()

Parquet is the preferred format for large datasets. It preserves dtypes (including datetime and categoricals), compresses well, and reads far faster than CSV. Use it for any file you will read more than once.

```python
# Save
df.to_parquet("sales.parquet", index=False, compression="snappy")

# Load
df = pd.read_parquet("sales.parquet")

# Load only specific columns — Parquet supports column pruning
df = pd.read_parquet("sales.parquet", columns=["order_date", "revenue"])
```

### Reading chunked CSVs

When a CSV is too large to fit in memory, read it in chunks and process incrementally.

```python
chunk_size = 100_000
results = []

for chunk in pd.read_csv("huge_file.csv", chunksize=chunk_size):
    # Process or filter each chunk
    filtered = chunk[chunk["revenue"] > 1000]
    results.append(filtered)

df = pd.concat(results, ignore_index=True)
```

### Dtype optimization on load

Specifying dtypes at load time reduces memory consumption, sometimes by 50–80%, especially on large files with many repeated string columns.

```python
dtype_map = {
    "region": "category",      # repeated strings → category saves memory
    "product": "category",
    "revenue": "float32",      # float64 by default; float32 often sufficient
    "units": "int32",
}
df = pd.read_csv("sales_data.csv", dtype=dtype_map, parse_dates=["order_date"])
df.info(memory_usage="deep")   # confirm the reduction
```
