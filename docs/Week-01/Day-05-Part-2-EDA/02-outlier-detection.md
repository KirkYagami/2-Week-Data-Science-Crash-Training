# Outlier Detection

Outliers are the values that break your model's assumptions, skew your summary statistics, and make your visualizations unreadable. But here is the thing experienced analysts understand: an outlier in your data might be the most important signal you have. A transaction 50x larger than average is not noise — it might be your best customer or your first fraud case. Never delete outliers without first understanding why they exist.

## Learning Objectives

By the end of this note you will be able to:

- Detect outliers using the IQR method and Z-score method
- Visualize outliers with boxplots and scatter plots
- Distinguish between data entry errors and genuine extreme values
- Choose an appropriate treatment strategy for each type of outlier
- Create outlier flags instead of blindly removing data

---

## What is an Outlier, Really?

An outlier is a value that sits far outside the typical range of a distribution. But "far outside" is context-dependent.

- A salary of $500,000 is an outlier in a dataset of baristas. It is not an outlier in a dataset of Fortune 500 CEOs.
- A transaction of $0.01 is an outlier in a fraud dataset. It might be the most important row you have.
- An age of 2 is impossible for a bank account holder. It is perfectly plausible for a pediatric patient.

Outlier detection is a measurement technique. The decision of what to do with an outlier is a judgment call that requires domain knowledge.

> [!info] Two Types of Outliers
> **Errors:** data entry mistakes, sensor failures, unit mismatches (pounds entered as kilograms), copy-paste issues. These should be corrected or nulled.
> **Real extremes:** genuine values that are unusual but accurate — a VIP customer's spend, a record-breaking sale, a rare medical event. These should be flagged, studied, and often kept.

---

## Build the Dataset

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

np.random.seed(42)

n = 200
data = {
    "customer_id": range(1, n + 1),
    "age": np.clip(np.random.normal(35, 8, n).astype(int), 18, 75),
    "monthly_spend": np.random.exponential(scale=1500, size=n).round(2),
    "order_count": np.random.poisson(lam=5, size=n),
    "session_duration_min": np.random.normal(loc=12, scale=4, size=n).round(1)
}

df = pd.DataFrame(data)

# Inject realistic outliers
df.loc[5, "monthly_spend"] = 95000    # VIP customer or data error?
df.loc[12, "monthly_spend"] = 87500
df.loc[88, "age"] = 150              # Obvious data error
df.loc[103, "order_count"] = 500     # Warehouse account or error?
df.loc[145, "session_duration_min"] = -3.0  # Impossible value

print(df.describe())
```

---

## Method 1 — Visual Detection (Always Start Here)

Visualization is your first tool. Numbers tell you something is wrong. Charts show you what is happening.

### Boxplot

The boxplot is designed specifically to show outliers. Values beyond 1.5 × IQR from the quartiles are plotted as individual points.

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

sns.boxplot(y=df["monthly_spend"], ax=axes[0])
axes[0].set_title("Monthly Spend — Raw")
axes[0].set_ylabel("Spend (₹)")

sns.boxplot(y=df["order_count"], ax=axes[1])
axes[1].set_title("Order Count — Raw")

plt.tight_layout()
plt.show()
```

Reading a boxplot:
- The box covers Q1 to Q3 (the interquartile range)
- The line inside is the median
- Whiskers extend to 1.5 × IQR from each quartile
- Points beyond the whiskers are flagged as potential outliers

### Histogram

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

sns.histplot(df["monthly_spend"], bins=40, kde=True, ax=axes[0])
axes[0].set_title("Monthly Spend Distribution")
axes[0].set_xlabel("Spend (₹)")

sns.histplot(df["session_duration_min"], bins=30, kde=True, ax=axes[1])
axes[1].set_title("Session Duration")
axes[1].axvline(0, color="red", linestyle="--", label="Zero")
axes[1].legend()

plt.tight_layout()
plt.show()
# A long right tail in spend, and a bar at negative values in session duration
```

### Scatter Plot for Paired Outliers

Sometimes a value is only an outlier in combination with another column.

```python
plt.figure(figsize=(8, 5))
sns.scatterplot(data=df, x="order_count", y="monthly_spend", alpha=0.6)
plt.title("Orders vs Spend — Outliers Show Up as Isolated Points")
plt.xlabel("Order Count")
plt.ylabel("Monthly Spend (₹)")
plt.show()
```

> [!tip] Plot First, Calculate Second
> When you see a boxplot flagging 40 outliers, check the histogram. If the distribution is heavily right-skewed, those "outliers" might just be the tail of a lognormal distribution, not actual errors.

---

## Method 2 — IQR Method

The Interquartile Range (IQR) method defines bounds based on the middle 50% of the data. It is robust to the distribution shape, making it more reliable than Z-score for skewed data.

```python
def find_outliers_iqr(series, factor=1.5):
    """
    Returns a boolean mask where True = outlier.
    factor=1.5 is standard. Use factor=3.0 for extreme outliers only.
    """
    q1 = series.quantile(0.25)
    q3 = series.quantile(0.75)
    iqr = q3 - q1
    lower = q1 - factor * iqr
    upper = q3 + factor * iqr
    return (series < lower) | (series > upper), lower, upper

# Apply to monthly_spend
outlier_mask, lower_bound, upper_bound = find_outliers_iqr(df["monthly_spend"])

print(f"IQR bounds: [{lower_bound:.0f}, {upper_bound:.0f}]")
# Output: IQR bounds: [-1823, 6319]  (lower bound is negative — spend can't be negative)

print(f"Outliers detected: {outlier_mask.sum()}")
# Output: Outliers detected: 8

# Inspect the flagged rows
print(df[outlier_mask][["customer_id", "monthly_spend", "order_count"]].sort_values("monthly_spend", ascending=False))
```

> [!info] Why IQR Works on Skewed Data
> The IQR only uses the 25th and 75th percentiles, so extreme values do not influence the bounds themselves. This makes it suitable for income, revenue, and transaction data — anything that is right-skewed. Z-score, by contrast, uses the mean and standard deviation, which outliers themselves distort.

---

## Method 3 — Z-Score Method

Z-score measures how many standard deviations a value is from the mean. Values beyond ±3 standard deviations are conventionally flagged.

```python
def find_outliers_zscore(series, threshold=3.0):
    """
    Returns boolean mask where True = outlier.
    Assumes the data is roughly normally distributed.
    """
    mean = series.mean()
    std = series.std()
    z_scores = (series - mean) / std
    return z_scores.abs() > threshold, z_scores

# Apply to session_duration_min (closer to normal)
outlier_mask_z, z_scores = find_outliers_zscore(df["session_duration_min"])

print(f"Z-score outliers: {outlier_mask_z.sum()}")
# Output: Z-score outliers: 1  (the -3.0 row)

# Show the extreme values
print(df[outlier_mask_z][["customer_id", "session_duration_min"]])
# customer 145 with -3.0 minutes
```

**Compare the two methods on the same column:**

```python
iqr_mask, _, _ = find_outliers_iqr(df["monthly_spend"])
zscore_mask, _ = find_outliers_zscore(df["monthly_spend"])

print(f"IQR method found: {iqr_mask.sum()} outliers")
print(f"Z-score method found: {zscore_mask.sum()} outliers")

# Often different counts — IQR is more conservative for skewed data
```

> [!warning] Z-Score Fails on Skewed Distributions
> If `monthly_spend` is right-skewed and you use Z-score, the high outliers pull the mean up and inflate the standard deviation. This shrinks the effective Z-score of every value and makes the method miss real outliers. Use IQR for skewed data. Use Z-score only when the distribution is approximately normal.

---

## Method 4 — Percentile Clipping

A practical middle ground for production pipelines: define bounds at the 1st and 99th percentile (or 5th and 95th, depending on tolerance).

```python
p01 = df["monthly_spend"].quantile(0.01)
p99 = df["monthly_spend"].quantile(0.99)

print(f"1st percentile: {p01:.0f}")
print(f"99th percentile: {p99:.0f}")

above_p99 = df[df["monthly_spend"] > p99]
print(f"Rows above 99th percentile: {len(above_p99)}")
```

---

## Treatment Options — The Decision Framework

Once you have identified outliers, you have five options. The choice depends on whether the outlier is an error or a genuine value.

| Treatment | When to Use | Code |
|-----------|------------|------|
| **Keep** | Genuine value, important signal | Do nothing |
| **Remove** | Confirmed data entry error | `df = df[~outlier_mask]` |
| **Cap (Winsorize)** | Genuine but extreme, want to reduce influence | `df["col"].clip(lower, upper)` |
| **Log Transform** | Right-skewed column, outliers are real | `np.log1p(df["col"])` |
| **Flag + Keep** | Unsure — preserve data, create indicator | Add binary column |

```python
# Option 1: Flag outliers without removing them
df["spend_outlier_flag"] = find_outliers_iqr(df["monthly_spend"])[0].astype(int)

# Option 2: Cap at IQR bounds (Winsorization)
_, lower_bound, upper_bound = find_outliers_iqr(df["monthly_spend"])
df["monthly_spend_capped"] = df["monthly_spend"].clip(lower=0, upper=upper_bound)
# Note: lower=0 because negative spend is impossible

# Option 3: Log transform (common for income, spend, count data)
df["monthly_spend_log"] = np.log1p(df["monthly_spend"])

# Option 4: Remove confirmed errors
df = df[df["age"] <= 120]   # age 150 is definitively wrong
df = df[df["session_duration_min"] >= 0]  # negative duration is impossible

print(f"Rows remaining after removing impossible values: {len(df)}")
```

> [!warning] Capping Changes Your Data
> Winsorization (clipping) replaces extreme values with the boundary value. If you cap spend at ₹6,000, then a customer who spent ₹95,000 and one who spent ₹6,000 look identical in your model. That may be acceptable. It may also make your model blind to your highest-value customers. Know what you are sacrificing.

---

## Documenting Outlier Decisions

In professional settings, every outlier decision should be documented. Here is a minimal tracking pattern:

```python
outlier_log = []

# Log each decision
outlier_log.append({
    "column": "age",
    "method": "business_rule",
    "threshold": "age > 120",
    "rows_affected": int((df["age"] > 120).sum()),
    "action": "set to NaN, re-imputed with median",
    "reason": "biologically impossible"
})

outlier_log.append({
    "column": "monthly_spend",
    "method": "IQR_1.5",
    "threshold": f"< {lower_bound:.0f} or > {upper_bound:.0f}",
    "rows_affected": int(iqr_mask.sum()),
    "action": "flagged, original values preserved",
    "reason": "may be genuine VIP customers, needs business validation"
})

outlier_df = pd.DataFrame(outlier_log)
print(outlier_df)
```

> [!success] The Golden Rule
> Never delete outliers silently. Either document why you removed them, or keep them and flag them. Future you — or a colleague — will need to understand every decision you made.

---

## Practice Exercises

### Warm-Up

```python
import numpy as np
sales = np.array([200, 215, 198, 210, 205, 220, 195, 3500, 208, 212, 199])
```

1. Calculate Q1, Q3, and IQR manually.
2. Compute the IQR bounds.
3. Which value is flagged as an outlier?
4. Calculate the Z-score for each value. Does Z-score also flag it?

### Main Exercise

```python
np.random.seed(7)
employees = pd.DataFrame({
    "employee_id": range(1, 151),
    "salary": np.random.normal(60000, 15000, 150).round(0),
    "years_exp": np.random.randint(0, 20, 150),
    "performance_score": np.random.normal(75, 10, 150).round(1)
})

# Inject issues
employees.loc[10, "salary"] = 850000   # Executive? Error?
employees.loc[25, "salary"] = -5000    # Impossible
employees.loc[60, "performance_score"] = 200  # Max is 100
employees.loc[90, "years_exp"] = -2   # Impossible
```

1. Use the IQR method to detect outliers in `salary`. Print the flagged rows.
2. Apply business rules: salary must be positive, score must be between 0 and 100, experience must be non-negative. Set violations to NaN.
3. For `salary`, compare how many outliers IQR detects vs Z-score.
4. Create a `salary_capped` column that caps at the 95th percentile.
5. Create a log of every outlier decision you made.

### Stretch

Write a function `outlier_report(df, columns)` that:
- Accepts a DataFrame and a list of column names
- For each column, applies both IQR and Z-score
- Returns a summary DataFrame showing: column name, IQR outlier count, Z-score outlier count, min outlier value, max outlier value
- Highlights any column where the two methods disagree by more than 50%

---

## Interview Questions

**Q1: What is the difference between the IQR and Z-score methods for outlier detection?**

??? "Show answer"
    IQR uses the 25th and 75th percentiles to define bounds. Because it uses percentiles rather than the mean and standard deviation, it is robust to the shape of the distribution and not influenced by the outliers themselves. Z-score uses the mean and standard deviation — both of which are distorted by outliers. Z-score is appropriate when the data is approximately normally distributed. IQR is preferred for skewed distributions like income, spend, and count data.

**Q2: You find that 15% of your rows are flagged as outliers by the IQR method. What do you do?**

??? "Show answer"
    Do not remove them. First, plot the distribution — if the data is right-skewed (exponential, lognormal), the IQR method will flag the natural tail as "outliers" even though they are legitimate. Consider whether a log transformation would help. Then inspect the flagged rows: are they errors (impossible values) or genuine extremes? For genuine extremes, flag them with a binary column and keep the data. For confirmed errors, set to NaN and document the decision. Removing 15% of your data is a drastic step that requires explicit justification.

**Q3: What is Winsorization and when should you use it?**

??? "Show answer"
    Winsorization caps values at a specified percentile or bound. Values above the upper bound are set to the upper bound; values below the lower bound are set to the lower bound. It is useful when: you need outliers to not influence a model's coefficients, but you do not want to lose those rows entirely. Use it when you are confident the extreme values are genuine but want to reduce their influence. Avoid it when the extreme values carry the most important signal (e.g., fraud detection, VIP customer analysis).

**Q4: How would you handle an outlier in your test set that does not appear in your training set?**

??? "Show answer"
    This is a real production problem. If you trained on capped data, your model has never seen values above the cap — and a test outlier will create unpredictable behavior. The solution is to apply the same capping/transformation pipeline to test data using parameters calculated from training data only. If you capped training salary at ₹6,000, you cap test salary at ₹6,000 too. Never recalculate outlier bounds on test data.

---

[[01-data-cleaning|Previous: Data Cleaning]] | [[03-feature-understanding|Next: Feature Understanding]]
