# Feature Understanding

The biggest mistake analysts make is running statistics on columns they do not understand. Before you calculate a correlation or plot a histogram, you need to know what each column actually means in the real world — what it measures, who collected it, when, and under what conditions. A column named `score` could be a customer satisfaction rating out of 5, a credit risk score out of 850, or a model confidence score between 0 and 1. The number 3.7 means completely different things in each case.

## Learning Objectives

By the end of this note you will be able to:

- Classify every column in a dataset by its data type and role
- Assess cardinality and identify how it affects modeling choices
- Identify the semantic meaning of missing values, not just their count
- Detect features that would cause data leakage in a predictive model
- Build a feature inventory document as a standard analysis artifact

---

## Why Feature Understanding Precedes Analysis

You can run `df.describe()` in 10 seconds. Understanding what those numbers mean takes longer but saves hours of rework. Two practitioners looking at the same dataset without feature understanding will draw different conclusions — and both may be wrong.

When you pick up a new dataset, ask these questions before touching the data:

- What does one row represent? (a customer? a transaction? a day? a product?)
- What is the target variable — what are you trying to understand or predict?
- Who collected this data, when, and how?
- What are the units for each numeric column?
- Are there columns that contain future information (potential leakage)?

> [!quote] On Feature Understanding
> "The data you receive is not the phenomenon itself. It is a measurement of the phenomenon, filtered through whoever built the data pipeline. Understanding the measurement process is inseparable from understanding the data." — Common wisdom in data science teams.

---

## Build the Dataset

```python
import pandas as pd
import numpy as np

np.random.seed(42)
n = 300

df = pd.DataFrame({
    # Identifiers — should never be features
    "customer_id": range(1001, 1001 + n),
    "account_number": [f"ACC{i:05d}" for i in range(n)],

    # Numeric continuous
    "age": np.clip(np.random.normal(38, 10, n).astype(int), 18, 80),
    "annual_income": np.random.lognormal(mean=10.8, sigma=0.6, size=n).round(0),
    "monthly_spend": np.random.exponential(scale=2200, size=n).round(2),
    "credit_score": np.clip(np.random.normal(650, 80, n).astype(int), 300, 850),

    # Numeric discrete
    "num_products": np.random.randint(1, 6, n),
    "support_tickets": np.random.poisson(lam=1.2, size=n),

    # Categorical nominal
    "city": np.random.choice(["Mumbai", "Delhi", "Bangalore", "Chennai", "Hyderabad"], n,
                              p=[0.30, 0.25, 0.20, 0.15, 0.10]),
    "account_type": np.random.choice(["Savings", "Current", "Salary"], n, p=[0.55, 0.25, 0.20]),
    "payment_method": np.random.choice(["UPI", "Card", "Netbanking", "Wallet"], n,
                                        p=[0.40, 0.30, 0.20, 0.10]),

    # Categorical ordinal
    "satisfaction_rating": np.random.choice([1, 2, 3, 4, 5], n, p=[0.05, 0.10, 0.20, 0.40, 0.25]),
    "risk_tier": np.random.choice(["Low", "Medium", "High"], n, p=[0.50, 0.35, 0.15]),

    # Boolean
    "is_premium": np.random.choice([0, 1], n, p=[0.70, 0.30]),
    "has_loan": np.random.choice([0, 1], n, p=[0.60, 0.40]),

    # Date
    "signup_date": pd.date_range("2019-01-01", periods=n, freq="3D"),

    # Target variable
    "churn": np.random.choice([0, 1], n, p=[0.75, 0.25]),
})

# Inject missing values
df.loc[np.random.choice(n, 20, replace=False), "annual_income"] = np.nan
df.loc[np.random.choice(n, 15, replace=False), "credit_score"] = np.nan
df.loc[np.random.choice(n, 8, replace=False), "city"] = np.nan

print(f"Dataset: {df.shape[0]} rows, {df.shape[1]} columns")
```

---

## Feature Type Classification

Getting the type right determines everything downstream — which plots you make, which statistics apply, and which encoding strategy to use before modeling.

```python
# Quick type summary
print(df.dtypes)
print("\n")
print(df.describe())           # numeric columns only
print(df.describe(include="object"))  # string columns only
```

| Feature Type | Description | Examples | Visualization | Modeling Treatment |
|---|---|---|---|---|
| Numeric continuous | Can take any value in a range | `age`, `income`, `spend` | Histogram, boxplot, KDE | Use directly (may need scaling) |
| Numeric discrete | Integers with meaningful gaps | `num_products`, `count` | Bar chart, histogram | Use directly or treat as categorical |
| Categorical nominal | Named categories, no order | `city`, `payment_method` | Bar chart, count plot | One-hot encoding |
| Categorical ordinal | Ordered categories | `satisfaction_rating`, `risk_tier` | Bar chart (ordered) | Ordinal encoding |
| Boolean / binary | True/False, 0/1 | `is_premium`, `has_loan` | Count plot | Use directly |
| Date/Time | Temporal values | `signup_date` | Line chart, seasonality | Extract components |
| Identifier | Unique key, not a feature | `customer_id`, `account_number` | — | Drop before modeling |
| Target | What you are predicting | `churn` | Distribution plot | Never use as a feature |

```python
# Classify columns by type
identifiers = ["customer_id", "account_number"]
target = "churn"
numeric_continuous = ["age", "annual_income", "monthly_spend", "credit_score"]
numeric_discrete = ["num_products", "support_tickets"]
categorical_nominal = ["city", "account_type", "payment_method"]
categorical_ordinal = ["risk_tier"]  # satisfaction_rating is discrete numeric here
boolean_cols = ["is_premium", "has_loan"]
datetime_cols = ["signup_date"]

print(f"Features to use in modeling: {len(numeric_continuous + numeric_discrete + categorical_nominal + categorical_ordinal + boolean_cols + datetime_cols)}")
print(f"Columns to drop: {identifiers}")
```

---

## Cardinality Analysis

Cardinality is the number of unique values in a column. It drives encoding decisions and flags anomalies.

```python
cardinality = df.nunique().sort_values(ascending=False)
print(cardinality)
# Output:
# customer_id        300  — identifier (all unique)
# account_number     300  — identifier (all unique)
# signup_date        300  — dates are often all unique
# annual_income      275  — high (continuous numeric)
# monthly_spend      300  — high (continuous numeric)
# age                 55  — moderate (discrete numeric range)
# credit_score       350  — high (but bounded 300-850)
# city                 5  — low (nominal categorical)
# payment_method       4  — low (nominal categorical)
# account_type         3  — low (nominal categorical)
# risk_tier            3  — low (ordinal categorical)
# num_products         5  — very low (discrete)
# satisfaction_rating  5  — very low (discrete/ordinal)
# support_tickets      7  — very low (discrete)
# is_premium           2  — binary
# has_loan             2  — binary
# churn                2  — binary (target)
```

**Cardinality thresholds (practical rules):**

```python
for col in df.columns:
    n_unique = df[col].nunique()
    pct_unique = n_unique / len(df) * 100

    if pct_unique > 90 and df[col].dtype == "object":
        print(f"WARNING: {col} — {n_unique} unique ({pct_unique:.0f}%) — likely an identifier, not a feature")
    elif n_unique == 1:
        print(f"WARNING: {col} — only 1 unique value — constant column, no predictive value")
    elif n_unique > 50 and df[col].dtype == "object":
        print(f"INFO: {col} — high cardinality ({n_unique}) — consider target encoding or embedding")
```

> [!warning] High-Cardinality Categoricals Break One-Hot Encoding
> A column with 500 unique city names will produce 500 binary columns when one-hot encoded. This creates a sparse matrix, increases model training time dramatically, and causes overfitting on rare categories. For high-cardinality categoricals, use target encoding, frequency encoding, or reduce cardinality by grouping rare categories into "Other".

---

## Inspecting Each Feature

For each column, run a quick diagnostic before including it in any analysis.

### Numeric Columns

```python
def inspect_numeric(df, col):
    s = df[col]
    print(f"\n--- {col} ---")
    print(f"  Type:     {s.dtype}")
    print(f"  Non-null: {s.notna().sum()} / {len(s)} ({s.notna().mean()*100:.1f}%)")
    print(f"  Mean:     {s.mean():.2f}")
    print(f"  Median:   {s.median():.2f}")
    print(f"  Std:      {s.std():.2f}")
    print(f"  Min:      {s.min():.2f}")
    print(f"  Max:      {s.max():.2f}")
    print(f"  Skewness: {s.skew():.2f}")
    print(f"  Zeros:    {(s == 0).sum()}")

for col in numeric_continuous:
    inspect_numeric(df, col)
```

### Categorical Columns

```python
def inspect_categorical(df, col, top_n=10):
    s = df[col]
    print(f"\n--- {col} ---")
    print(f"  Type:      {s.dtype}")
    print(f"  Non-null:  {s.notna().sum()} / {len(s)}")
    print(f"  Unique:    {s.nunique()}")
    print(f"\n  Value counts (top {top_n}):")
    vc = s.value_counts(dropna=False)
    print(vc.head(top_n).to_string())

    rare = vc[vc < 5]
    if len(rare) > 0:
        print(f"\n  Rare categories (count < 5): {len(rare)}")
        print(rare.to_string())

for col in categorical_nominal:
    inspect_categorical(df, col)
```

---

## The Semantic Meaning of Missing Values

This is the most underrated concept in feature understanding. `NaN` is not a single concept — it has different meanings depending on context, and each meaning calls for a different response.

```python
# Scenario analysis for missing values
missing_semantics = {
    "annual_income": {
        "why_missing": "Customer did not disclose income during onboarding",
        "meaning": "Income is private — missingness may correlate with income level",
        "action": "Create missing indicator + impute with median",
        "risk": "Imputing may obscure a segment of privacy-sensitive customers"
    },
    "credit_score": {
        "why_missing": "No credit history (new to credit, or foreign national)",
        "meaning": "Missing means 'no credit history' — very different from average credit",
        "action": "Create missing indicator, do NOT impute with mean/median",
        "risk": "Imputing with mean makes no-history customers look like average-history customers"
    },
    "city": {
        "why_missing": "Address not provided or database join failed",
        "meaning": "Ambiguous — could be either reason",
        "action": "Investigate join logic; fill with 'Unknown' if truly missing",
        "risk": "Low — city is likely not the primary driver of any model"
    }
}

for col, info in missing_semantics.items():
    print(f"\n{col}:")
    for k, v in info.items():
        print(f"  {k}: {v}")
```

> [!warning] Missing Credit Score Does Not Mean Average Credit
> If you impute a missing credit score with the column median (say, 650), you are telling your model that customers with no credit history look the same as customers with established, average credit. These are completely different risk profiles. The correct approach is to create a `credit_score_missing` indicator and either use a model that handles NaN natively or fill with a sentinel value like -1 that signals "no data" rather than a real score.

---

## Rare Category Detection

Rare categories cause problems in both analysis and modeling. A category that appears in only 3 rows can produce unreliable statistics and will almost certainly appear in training but not validation splits, causing model errors.

```python
def detect_rare_categories(df, col, threshold=0.02):
    """Flag categories that make up less than `threshold` fraction of data."""
    vc = df[col].value_counts(normalize=True)
    rare = vc[vc < threshold]
    print(f"\n{col} — rare categories (< {threshold*100:.0f}% of data):")
    if len(rare) == 0:
        print("  None")
    else:
        print(rare.to_string())
    return rare.index.tolist()

rare_cities = detect_rare_categories(df, "city", threshold=0.05)
rare_payments = detect_rare_categories(df, "payment_method", threshold=0.05)
```

```python
# Group rare categories into "Other"
def group_rare_categories(series, threshold=0.02, other_label="Other"):
    vc = series.value_counts(normalize=True)
    rare_cats = vc[vc < threshold].index
    return series.replace(rare_cats, other_label)

# Apply if needed
df["payment_method_clean"] = group_rare_categories(df["payment_method"], threshold=0.05)
print(df["payment_method_clean"].value_counts())
```

---

## Leakage Detection

Data leakage means a feature contains information that would not be available at prediction time. It inflates model performance during development and causes the model to fail completely in production. Leakage is one of the most common sources of "our model worked in testing but failed in the real world."

```python
# Common leakage patterns to watch for
leakage_examples = {
    "churn_date": "Date the customer churned — you only have this after churn happened. Cannot use to predict churn.",
    "refund_issued": "Refunds happen after a complaint. Cannot use to predict whether a complaint will be filed.",
    "final_account_balance": "Balance at account closure. Only known after the fact.",
    "customer_lifetime_value": "Calculated using all future transactions — not available at the time of prediction.",
    "support_resolution_time": "Total resolution time is only known after the ticket is closed.",
}

# Safe features for predicting churn (features known BEFORE churn decision)
safe_features = [
    "age", "annual_income", "monthly_spend", "credit_score",
    "num_products", "support_tickets",  # tickets opened (not resolved)
    "account_type", "payment_method", "is_premium", "has_loan",
    "satisfaction_rating", "risk_tier",
    "signup_date"  # can engineer tenure from this
]

# Check for potential leakage
target_col = "churn"
for col in df.columns:
    if col == target_col:
        continue
    corr = df[col].corr(df[target_col]) if df[col].dtype != "object" else None
    if corr and abs(corr) > 0.8:
        print(f"WARNING: {col} has correlation {corr:.2f} with target — investigate for leakage")
```

> [!warning] High Correlation with Target is a Red Flag
> If a feature has correlation > 0.9 with your target in the training data, ask: "Would I have this value at prediction time?" If the answer is "only after the outcome is known," it is leakage. Perfect models built on leaky data are worthless in production.

---

## The Feature Inventory

Build this table for every new project. It takes 30 minutes and saves days.

```python
feature_inventory = pd.DataFrame({
    "feature": [
        "age", "annual_income", "monthly_spend", "credit_score",
        "num_products", "support_tickets",
        "city", "account_type", "payment_method",
        "satisfaction_rating", "risk_tier",
        "is_premium", "has_loan", "signup_date"
    ],
    "type": [
        "numeric_continuous", "numeric_continuous", "numeric_continuous", "numeric_continuous",
        "numeric_discrete", "numeric_discrete",
        "categorical_nominal", "categorical_nominal", "categorical_nominal",
        "categorical_ordinal", "categorical_ordinal",
        "boolean", "boolean", "datetime"
    ],
    "missing_count": [df[c].isna().sum() for c in [
        "age", "annual_income", "monthly_spend", "credit_score",
        "num_products", "support_tickets",
        "city", "account_type", "payment_method",
        "satisfaction_rating", "risk_tier",
        "is_premium", "has_loan", "signup_date"
    ]],
    "cardinality": [df[c].nunique() for c in [
        "age", "annual_income", "monthly_spend", "credit_score",
        "num_products", "support_tickets",
        "city", "account_type", "payment_method",
        "satisfaction_rating", "risk_tier",
        "is_premium", "has_loan", "signup_date"
    ]],
    "leakage_risk": [
        "No", "No", "No", "No",
        "No", "No",
        "No", "No", "No",
        "Low", "No",
        "No", "No", "No"
    ],
    "action": [
        "Use directly", "Impute median + flag", "Use directly", "Impute median + flag",
        "Use directly", "Use directly",
        "Impute Unknown + encode", "Encode", "Encode",
        "Treat as ordinal", "Ordinal encode",
        "Use directly", "Use directly", "Extract tenure"
    ]
})

print(feature_inventory.to_string(index=False))
```

> [!success] The Feature Inventory is a Communication Tool
> Share this table with business stakeholders. They often know which columns are unreliable, which were recently added to the pipeline, and which contain known quality issues. That context changes your analysis decisions.

---

## Practice Exercises

### Warm-Up

Given `df.dtypes` output below, classify each feature as: identifier, numeric continuous, numeric discrete, categorical nominal, categorical ordinal, boolean, datetime, or target.

```
customer_id         int64
order_date          object
product_category    object
quantity_ordered    int64
unit_price          float64
total_revenue       float64
region              object
return_flag         int64   (0 or 1)
priority_level      object  ("Low", "Medium", "High", "Critical")
revenue_target      float64
```

### Main Exercise

```python
np.random.seed(99)
loans = pd.DataFrame({
    "loan_id": range(5001, 5201),
    "applicant_age": np.random.randint(22, 65, 200),
    "loan_amount": np.random.lognormal(11, 0.5, 200).round(0),
    "interest_rate": np.random.uniform(8.5, 22.0, 200).round(2),
    "employment_type": np.random.choice(["Salaried", "Self-Employed", "Unemployed", "Contract"], 200, p=[0.55, 0.25, 0.10, 0.10]),
    "credit_history_years": np.random.choice([0, 1, 2, 3, 5, 7, 10, 15], 200),
    "loan_status": np.random.choice(["Approved", "Rejected", "Pending"], 200, p=[0.60, 0.30, 0.10]),
    "default": np.random.choice([0, 1], 200, p=[0.82, 0.18]),
    "default_date": [None] * 164 + [f"2023-{m:02d}-01" for m in np.random.randint(1, 13, 36)]
})

loans.loc[np.random.choice(200, 25), "credit_history_years"] = np.nan
```

1. Classify every column using the feature type table from this note.
2. Identify any columns that would cause data leakage if you tried to predict `default`.
3. Run `inspect_categorical` and `inspect_numeric` on every column.
4. Detect rare categories in `employment_type`.
5. Build a feature inventory table for this dataset.

### Stretch

Write a function `auto_classify_features(df, target_col, id_cols)` that automatically classifies each column as:
- identifier (in `id_cols`)
- target (matches `target_col`)
- binary (only 2 unique non-null values)
- low-cardinality categorical (object dtype, <= 20 unique)
- high-cardinality categorical (object dtype, > 20 unique)
- numeric continuous (float, or int with > 20 unique values)
- numeric discrete (int, <= 20 unique)
- datetime (datetime dtype)

Return a DataFrame with columns: `feature`, `classified_type`, `missing_pct`, `unique_count`.

---

## Interview Questions

**Q1: What is data leakage and how do you detect it?**

??? "Show answer"
    Data leakage occurs when a feature contains information that would not be available at the time a model makes a prediction in production. It causes inflated performance during development and failure in production. Detection methods: check whether each feature is derived from the outcome variable; check whether the feature has suspiciously high correlation with the target; ask "would I have this value before the event I am predicting?" For time-series data, check whether future information leaks into past time windows.

**Q2: Why does cardinality matter for categorical variables?**

??? "Show answer"
    High-cardinality categorical columns (hundreds or thousands of unique values) cause problems for one-hot encoding — you end up with hundreds of sparse binary columns, which slows training, increases memory usage, and causes overfitting on rare categories. High-cardinality also means rare categories appear in very few rows, making statistics on them unreliable. Solutions: frequency encoding, target encoding, grouping rare categories into "Other", or using embedding layers in neural networks.

**Q3: A column has 35% missing values. What questions do you ask before deciding what to do?**

??? "Show answer"
    First: Is there a pattern to the missingness? (Is it missing for a specific time period, a specific segment, or randomly?) Second: What does missing mean semantically? ("No data collected" vs "the event has not occurred" vs "the customer chose not to answer" are all different.) Third: Is this column important for the model? Fourth: Is the missingness itself a signal? If yes, create an indicator column. If the column is critical and missingness is not random, investigate the data pipeline — there may be a collection bug. If missingness is random and the column is important, impute + flag.

**Q4: What is the difference between a numeric discrete and a categorical ordinal feature?**

??? "Show answer"
    Numeric discrete features are counts or integers where the values are genuinely numeric (e.g., `number_of_products` = 3 means three products, and 3 is exactly twice 1.5). Categorical ordinal features have categories that are ordered but where the intervals between them are not meaningful or equal (e.g., `satisfaction` = "High" is greater than "Low", but "High" is not "twice" "Low"). The treatment differs: numeric discrete can often be used directly in models; categorical ordinal should be encoded with care — ordinal encoding preserves order, but one-hot encoding loses it.

---

[[02-outlier-detection|Previous: Outlier Detection]] | [[04-univariate-analysis|Next: Univariate Analysis]]
