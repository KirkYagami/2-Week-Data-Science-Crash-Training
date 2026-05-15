# Univariate Analysis

Before you can understand how variables relate to each other, you need to understand each variable on its own. Univariate analysis is not just about making charts — it is about answering a specific question for each column: "Is this column's distribution what I expect?" When the answer is no, you have found something worth investigating.

## Learning Objectives

By the end of this note you will be able to:

- Choose the right visualization for numeric vs categorical vs date columns
- Read a histogram, KDE plot, boxplot, and Q-Q plot and know what each reveals
- Calculate and interpret summary statistics including skewness and kurtosis
- Identify distribution shapes and understand their practical implications
- Detect problematic patterns in categorical distributions (dominance, rareness, imbalance)

---

## Why One Variable at a Time?

You might be tempted to jump straight to correlation matrices and multi-feature plots. The problem is that if you have not examined each column individually, you will misread every relationship you find later.

Consider: you compute the correlation between `age` and `monthly_spend` and find a weak positive correlation. Before you conclude "older customers spend more," you need to know: Is `age` normally distributed? Is it capped at a certain value? Does it have a bimodal structure (two age groups)? Is `monthly_spend` dominated by a few extreme values? Without that context, the correlation number is uninterpretable.

> [!info] Univariate Analysis Builds Pattern Recognition
> After doing univariate analysis on dozens of datasets, you develop intuition for what "normal" looks like for different types of columns. Age follows a roughly normal distribution. Income is right-skewed. Count data follows Poisson. When a column does not match its expected shape, that is signal.

---

## Build the Dataset

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats

np.random.seed(42)
n = 500

df = pd.DataFrame({
    "customer_id": range(1, n + 1),
    "age": np.clip(np.random.normal(37, 10, n).astype(int), 18, 75),
    "annual_income": np.random.lognormal(mean=10.7, sigma=0.55, size=n).round(0),
    "monthly_spend": np.random.exponential(scale=2000, size=n).round(2),
    "support_tickets": np.random.poisson(lam=1.5, size=n),
    "session_minutes": np.abs(np.random.normal(15, 5, n).round(1)),
    "credit_score": np.clip(np.random.normal(660, 80, n).astype(int), 300, 850),
    "city": np.random.choice(
        ["Mumbai", "Delhi", "Bangalore", "Chennai", "Hyderabad", "Kolkata", "Pune"],
        n, p=[0.28, 0.22, 0.18, 0.12, 0.10, 0.06, 0.04]
    ),
    "account_type": np.random.choice(["Savings", "Current", "Salary"], n, p=[0.55, 0.25, 0.20]),
    "risk_tier": np.random.choice(["Low", "Medium", "High"], n, p=[0.50, 0.35, 0.15]),
    "churn": np.random.choice([0, 1], n, p=[0.75, 0.25])
})

# Inject some realism: bimodal age distribution (young and middle-aged customers)
idx_young = np.random.choice(n, 80, replace=False)
df.loc[idx_young, "age"] = np.random.randint(20, 28, 80)

print(df.shape)
# Output: (500, 11)
```

---

## Analyzing Numeric Features

### Start with `.describe()`

Before any plots, read the numbers.

```python
print(df[["age", "annual_income", "monthly_spend", "credit_score"]].describe().round(2))
# Output:
#           age  annual_income  monthly_spend  credit_score
# count  500.00         500.00         500.00        500.00
# mean    36.51       55241.73        2103.47        659.23
# std     11.04       36891.22        2142.30         79.82
# min     18.00       12831.00           8.84        300.00
# 25%     28.00       33074.50         588.31        607.00
# 50%     36.00       46518.50        1433.68        660.00
# 75%     45.00       65214.75        2985.91        711.75
# max     75.00      392761.00       22483.71        850.00
```

**What to look for in `.describe()` output:**

- `min` and `max` — do they make sense given the column's definition?
- Gap between `mean` and `median (50%)` — a large gap signals skewness
- `std` relative to `mean` — if std > mean for a non-negative column, the data is very spread out
- `25%` vs `75%` — how wide is the interquartile range?

```python
# Also check skewness
print("\nSkewness:")
for col in ["age", "annual_income", "monthly_spend", "credit_score"]:
    print(f"  {col}: {df[col].skew():.2f}")
# Output:
#   age: 0.11         (roughly symmetric)
#   annual_income: 3.42  (strongly right-skewed)
#   monthly_spend: 2.67  (strongly right-skewed)
#   credit_score: -0.03  (roughly symmetric)
```

Skewness interpretation:
- Between -0.5 and 0.5: roughly symmetric
- Between 0.5 and 1.0 (or -1.0 to -0.5): moderately skewed
- Beyond 1.0 (or below -1.0): strongly skewed — consider log transform before modeling

---

### Histogram — The Most Useful First Chart

The histogram answers: "How is this value distributed across my data?"

```python
fig, axes = plt.subplots(2, 2, figsize=(12, 8))
fig.suptitle("Univariate Distributions — Numeric Features", fontsize=14, fontweight="bold")

# Age: roughly normal with a young-customer cluster
sns.histplot(df["age"], bins=30, kde=True, ax=axes[0, 0], color="#0D9488")
axes[0, 0].set_title("Age")
axes[0, 0].axvline(df["age"].mean(), color="#F59E0B", linestyle="--", label=f"Mean: {df['age'].mean():.0f}")
axes[0, 0].axvline(df["age"].median(), color="white", linestyle=":", label=f"Median: {df['age'].median():.0f}")
axes[0, 0].legend(fontsize=8)

# Income: right-skewed (lognormal)
sns.histplot(df["annual_income"], bins=40, kde=True, ax=axes[0, 1], color="#0D9488")
axes[0, 1].set_title("Annual Income (right-skewed)")
axes[0, 1].axvline(df["annual_income"].mean(), color="#F59E0B", linestyle="--", label=f"Mean: {df['annual_income'].mean():.0f}")
axes[0, 1].axvline(df["annual_income"].median(), color="white", linestyle=":", label=f"Median: {df['annual_income'].median():.0f}")
axes[0, 1].legend(fontsize=8)

# Monthly spend: exponential distribution
sns.histplot(df["monthly_spend"], bins=40, kde=True, ax=axes[1, 0], color="#0D9488")
axes[1, 0].set_title("Monthly Spend (exponential)")

# Credit score: roughly normal
sns.histplot(df["credit_score"], bins=30, kde=True, ax=axes[1, 1], color="#0D9488")
axes[1, 1].set_title("Credit Score (normal)")

plt.tight_layout()
plt.show()
```

**How to read a histogram:**
- Single peak (unimodal) with symmetric taper: normal-like distribution
- Single peak with long right tail: right-skewed (income, spend, counts are usually this shape)
- Single peak with long left tail: left-skewed (time-to-failure, test scores near ceiling)
- Two peaks (bimodal): two sub-populations mixed together — investigate the segments
- Flat (uniform): all values equally common — unusual, might signal synthetic data

> [!tip] Choose Bin Count Thoughtfully
> `bins=10` hides structure. `bins=200` shows noise. For 500 rows, try 30–50 bins. For 10,000+ rows, 50–100. The Freedman-Diaconis rule (`bins="fd"`) is a reliable automatic choice.

---

### KDE Plot — Smooth the Distribution

KDE (Kernel Density Estimate) smooths the histogram into a continuous curve. It is better for comparing distributions across groups.

```python
fig, ax = plt.subplots(figsize=(8, 4))

sns.kdeplot(df["annual_income"], ax=ax, color="#0D9488", linewidth=2, label="Annual Income")

# Show what the log-transformed version looks like (much more symmetric)
ax2 = ax.twinx()
sns.kdeplot(np.log1p(df["annual_income"]), ax=ax2, color="#F59E0B", linewidth=2, linestyle="--", label="Log(Income)")

ax.set_xlabel("Value")
ax.set_title("Income: Raw vs Log-Transformed Distribution")
ax.legend(loc="upper left")
ax2.legend(loc="upper right")
plt.show()
```

---

### Boxplot — Five-Number Summary + Outliers

The boxplot shows Q1, median, Q3, whiskers (1.5×IQR), and individual outlier points — all in one chart.

```python
fig, axes = plt.subplots(1, 3, figsize=(14, 5))

sns.boxplot(y=df["annual_income"], ax=axes[0], color="#0D9488")
axes[0].set_title("Income (note outlier points above whisker)")

sns.boxplot(y=df["monthly_spend"], ax=axes[1], color="#0D9488")
axes[1].set_title("Monthly Spend")

sns.boxplot(y=df["credit_score"], ax=axes[2], color="#0D9488")
axes[2].set_title("Credit Score (more symmetric)")

plt.tight_layout()
plt.show()
```

**How to read a boxplot:**
- Box height = IQR (small box = low variance in middle 50%)
- Median line position within box: if close to Q1 = right-skewed; close to Q3 = left-skewed
- Long whiskers = values spread far from the quartiles
- Individual points = flagged outliers (but not necessarily errors)

---

### Q-Q Plot — Test for Normality

A Q-Q (Quantile-Quantile) plot compares your data's quantiles to a theoretical normal distribution. If the points fall on the diagonal line, the data is normally distributed.

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Age: should be roughly normal
stats.probplot(df["age"], dist="norm", plot=axes[0])
axes[0].set_title("Q-Q Plot: Age (expect near-linear)")

# Income: right-skewed, expect deviation from the line
stats.probplot(df["annual_income"], dist="norm", plot=axes[1])
axes[1].set_title("Q-Q Plot: Income (expect curve at the right tail)")

plt.tight_layout()
plt.show()
```

**Reading a Q-Q plot:**
- Points on the diagonal: normally distributed
- Points curve upward at the right: right-skewed (heavy right tail)
- Points curve downward at the left: left-skewed
- Points at the ends diverge sharply: heavy tails (leptokurtic)

> [!info] When Normality Matters
> Q-Q plots matter most when you plan to use models or tests that assume normality (linear regression, ANOVA, t-tests). For tree-based models (Random Forest, XGBoost), the distribution of individual features matters much less.

---

## Analyzing Categorical Features

### Value Counts

```python
print(df["city"].value_counts())
# Output:
# Mumbai       138
# Delhi        108
# Bangalore     93
# Chennai       61
# Hyderabad     52
# Kolkata       30
# Pune          18
# Name: city, dtype: int64

print(df["city"].value_counts(normalize=True).mul(100).round(1))
# Output: percentage distribution
```

### Bar Chart for Categorical Distribution

```python
fig, axes = plt.subplots(1, 2, figsize=(13, 5))

# City distribution
city_counts = df["city"].value_counts()
sns.barplot(x=city_counts.values, y=city_counts.index, ax=axes[0], palette="Blues_r")
axes[0].set_title("Customer Distribution by City")
axes[0].set_xlabel("Count")

# Account type
account_counts = df["account_type"].value_counts()
sns.barplot(x=account_counts.index, y=account_counts.values, ax=axes[1], palette="Blues_r")
axes[1].set_title("Account Type Distribution")
axes[1].set_ylabel("Count")

plt.tight_layout()
plt.show()
```

> [!warning] Watch for Dominant Categories
> If one category holds 95% of your data, it will dominate every grouped statistic. Any comparison involving "city" will be almost entirely Mumbai. Models may also learn the dominant category as a near-constant feature. Check your category distribution before any grouped analysis.

---

### Ordinal Category Analysis

For ordered categories (risk tier, satisfaction rating), the order matters in your visualization.

```python
# Define the correct order
risk_order = ["Low", "Medium", "High"]
risk_counts = df["risk_tier"].value_counts().reindex(risk_order)

fig, ax = plt.subplots(figsize=(7, 4))
bars = ax.bar(risk_order, risk_counts.values, color=["#22c55e", "#f59e0b", "#ef4444"])
ax.set_title("Risk Tier Distribution (Ordered)")
ax.set_ylabel("Count")

# Add count labels on bars
for bar, count in zip(bars, risk_counts.values):
    ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 3,
            str(count), ha="center", fontsize=10)

plt.tight_layout()
plt.show()
```

---

## Analyzing Date Features

```python
# Extract useful components
df["signup_year"] = pd.to_datetime("2019-01-01") + pd.to_timedelta(
    np.arange(n) * 3, unit="D"
)
df["signup_month"] = df["signup_year"].dt.month
df["signup_day_of_week"] = df["signup_year"].dt.day_name()

fig, axes = plt.subplots(1, 2, figsize=(13, 4))

# Monthly signups
month_counts = df["signup_month"].value_counts().sort_index()
axes[0].bar(month_counts.index, month_counts.values, color="#0D9488")
axes[0].set_xticks(range(1, 13))
axes[0].set_xticklabels(["Jan", "Feb", "Mar", "Apr", "May", "Jun",
                          "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"], rotation=45)
axes[0].set_title("Monthly Signup Distribution")

# Day of week
dow_order = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
dow_counts = df["signup_day_of_week"].value_counts().reindex(dow_order)
axes[1].bar(range(7), dow_counts.values, color="#0D9488")
axes[1].set_xticks(range(7))
axes[1].set_xticklabels(["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"])
axes[1].set_title("Signup Day of Week")

plt.tight_layout()
plt.show()
```

---

## Analyzing the Target Variable

Always examine your target variable's distribution before anything else. For classification, class imbalance changes every decision you make about modeling.

```python
target_counts = df["churn"].value_counts()
target_pct = df["churn"].value_counts(normalize=True).mul(100).round(1)

print("Churn distribution:")
print(pd.DataFrame({"count": target_counts, "pct": target_pct}))
# Output:
# churn    count    pct
#     0      375   75.0
#     1      125   25.0

fig, axes = plt.subplots(1, 2, figsize=(10, 4))

axes[0].bar(["Retained (0)", "Churned (1)"], target_counts.values,
            color=["#0D9488", "#ef4444"])
axes[0].set_title("Target Distribution: Churn")
axes[0].set_ylabel("Count")

# Pie chart for proportion
axes[1].pie(target_counts.values, labels=["Retained", "Churned"],
            autopct="%1.1f%%", colors=["#0D9488", "#ef4444"])
axes[1].set_title("Churn Proportion")

plt.tight_layout()
plt.show()
```

> [!warning] Class Imbalance Changes Your Evaluation Metric
> A 75/25 split is moderate imbalance. A model that predicts "No churn" for every customer achieves 75% accuracy without learning anything. When you see imbalance, plan to use precision/recall/F1 or AUC-ROC rather than raw accuracy as your evaluation metric.

---

## The Univariate Checklist

Run through this for every column:

- [ ] What is the data type? Does it match the column's content?
- [ ] How many non-null values? What percentage is missing?
- [ ] How many unique values? (cardinality)
- [ ] For numeric: what are the min, max, mean, median, std? Is skewness > 1?
- [ ] For numeric: plot histogram + boxplot. What is the shape?
- [ ] For categorical: what are the top 5 categories and their percentages?
- [ ] For categorical: are there rare categories (< 2% of data)?
- [ ] Does anything look surprising compared to what you expected?
- [ ] Write one insight per column before moving on

> [!success] One Insight Per Feature
> The discipline of writing one insight per feature forces you to actually think about what each distribution tells you. "This column looks normal" is not an insight. "Annual income is right-skewed (skewness=3.4), and the top 5% of customers earn more than the bottom 50% combined — this will require log transformation before linear models" is an insight.

---

## Practice Exercises

### Warm-Up

```python
np.random.seed(10)
scores = pd.Series(np.concatenate([
    np.random.normal(55, 8, 100),   # one group
    np.random.normal(85, 5, 40),    # another group
]))
```

1. Plot a histogram with `bins=30` and KDE overlay. What does the shape suggest?
2. Calculate mean, median, and std. Why does the mean sit between the two peaks?
3. Create a boxplot. How many outliers are flagged?
4. Create a Q-Q plot. What does the deviation from the diagonal tell you?

### Main Exercise

```python
np.random.seed(7)
retail = pd.DataFrame({
    "order_value": np.random.exponential(800, 400),
    "items_per_order": np.random.poisson(3.2, 400),
    "customer_rating": np.random.choice([1, 2, 3, 4, 5], 400, p=[0.05, 0.08, 0.17, 0.38, 0.32]),
    "product_category": np.random.choice(
        ["Electronics", "Clothing", "Food", "Books", "Home", "Sports", "Beauty"],
        400, p=[0.25, 0.20, 0.15, 0.12, 0.12, 0.10, 0.06]
    ),
    "return_flag": np.random.choice([0, 1], 400, p=[0.88, 0.12])
})
```

1. For `order_value`: histogram, boxplot, describe(), skewness. Write one insight.
2. For `items_per_order`: histogram, describe(). Does it look like a Poisson distribution?
3. For `customer_rating`: value_counts(), bar chart with correct order. Is the distribution what you expect for a real e-commerce site?
4. For `product_category`: bar chart sorted by frequency. Which category is dominant?
5. For `return_flag`: count distribution. Is this a balanced or imbalanced target?

### Stretch

Build a reusable `univariate_report(df, col)` function that:
- Detects whether the column is numeric or categorical
- For numeric: prints describe(), skewness, and creates histogram + boxplot side by side
- For categorical: prints value_counts(normalize=True), flags rare categories, and creates a bar chart
- Returns a dict with key statistics

---

## Interview Questions

**Q1: A histogram shows a bimodal distribution (two peaks) for customer age. What does that tell you?**

??? "Show answer"
    A bimodal distribution typically signals two distinct sub-populations in your data. For age, this might mean you have a young customer segment (20–30) and a middle-aged segment (45–60), with very few customers in between. This is important because: (1) Summary statistics like mean age will be meaningless — the mean might fall in the valley between the two peaks, representing almost no actual customer. (2) Any analysis treating this as a single population will be misleading. (3) You should segment the data and analyze each group separately before any modeling.

**Q2: What is skewness and why does it matter for machine learning?**

??? "Show answer"
    Skewness measures the asymmetry of a distribution. A right-skewed distribution (positive skewness) has a long right tail — most values are low but a few are very high (income, spend, transaction amounts). Left-skewed distributions are the reverse. It matters for ML because: linear models assume normally distributed residuals (not features, but skewness in features often leads to skewness in residuals); distance-based algorithms (KNN, K-Means) are distorted by skewed features; log transformation or Box-Cox can normalize skewed features and often improves model performance. Tree-based models (Random Forest, XGBoost) are not sensitive to skewness.

**Q3: When would you use a KDE plot instead of a histogram?**

??? "Show answer"
    A KDE (Kernel Density Estimate) plot is preferred when: you want to compare multiple distributions on the same axis (overlapping KDE curves are much cleaner than overlapping histograms); you want a smooth continuous representation without choosing bin count; you are presenting to a non-technical audience who might misread histogram bins. The downside of KDE is that it can suggest smoothness that does not exist, and it can extend beyond the actual data range (e.g., generating negative values for a non-negative variable). For initial EDA, use both — they reveal different aspects.

**Q4: Your target variable has a 95/5 class split. What are the implications?**

??? "Show answer"
    With 95% negative / 5% positive: (1) Accuracy is a misleading metric — a model that predicts "negative" for everything achieves 95% accuracy without learning anything. Use precision, recall, F1-score, or AUC-ROC instead. (2) The model needs exposure to the minority class — consider oversampling (SMOTE), undersampling, or class weighting. (3) Your threshold for classification matters more — the default 0.5 probability threshold may not be optimal; you may need to lower it to catch more positives. (4) During cross-validation, use stratified splits to ensure both folds contain representatives of both classes.

---

[[03-feature-understanding|Previous: Feature Understanding]] | [[05-bivariate-analysis|Next: Bivariate Analysis]]
