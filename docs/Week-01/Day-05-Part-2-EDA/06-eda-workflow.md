# EDA Workflow

EDA is not random exploration. The people who get the most out of it follow a systematic process — the same checklist every time, adapted to the data. Without structure, you end up making 40 charts, writing zero insights, and missing the missing values that will break your model three days later.

This file gives you that structure.

## Learning Objectives

By the end of this, you will be able to:

- Run a systematic EDA on any tabular dataset in under an hour
- Distinguish data quality issues from genuine analytical signals
- Produce written insights, not just charts
- Know exactly what to hand off to the modelling phase

---

## The EDA Checklist

Print this and use it every time.

### Phase 1 — First Contact (5 minutes)

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("your_data.csv")

# Shape and column names
print(df.shape)          # (rows, cols)
print(df.columns.tolist())
print(df.dtypes)

# First look at the data
df.head(10)
```

Questions to answer:

- What does each row represent? (a customer? a transaction? a day?)
- What is the target variable?
- Are there columns you can immediately identify as IDs or leakage candidates?

### Phase 2 — Data Quality Audit (10 minutes)

```python
# Missing values — absolute and percentage
missing = pd.DataFrame({
    "count": df.isna().sum(),
    "pct":   (df.isna().mean() * 100).round(1)
}).query("count > 0").sort_values("pct", ascending=False)
print(missing)

# Duplicates
n_dupes = df.duplicated().sum()
print(f"Duplicate rows: {n_dupes} ({n_dupes/len(df)*100:.1f}%)")

# Type issues — columns that look numeric but are stored as object
print(df.select_dtypes("object").describe())
```

For each column with missing values, ask: **why is it missing?** Is the absence meaningful (no purchase = no revenue), random (sensor glitch), or structural (field didn't exist before a certain date)?

> [!warning] Never start by dropping NaNs
> `df.dropna()` is the most common destructive mistake in EDA. You might discard 30% of your data based on one column with one missing value. Understand the missingness pattern before deciding how to handle it.

### Phase 3 — Univariate Analysis (15 minutes)

Analyse each column individually. You are looking for distribution shape, outliers, cardinality, and anything unexpected.

```python
# Numeric columns
print(df.describe(percentiles=[0.01, 0.25, 0.5, 0.75, 0.99]))

# Categorical columns
for col in df.select_dtypes("object").columns:
    print(f"\n{col}: {df[col].nunique()} unique values")
    print(df[col].value_counts().head(10))
```

Automated distribution plots:

```python
numeric_cols = df.select_dtypes(include=np.number).columns.tolist()
n = len(numeric_cols)
fig, axes = plt.subplots(nrows=(n + 2) // 3, ncols=3, figsize=(15, 4 * ((n + 2) // 3)))
axes = axes.flatten()

for i, col in enumerate(numeric_cols):
    sns.histplot(df[col].dropna(), kde=True, ax=axes[i])
    axes[i].set_title(col)

for j in range(i + 1, len(axes)):
    axes[j].set_visible(False)

plt.tight_layout()
plt.savefig("univariate_distributions.png")
```

What to note per column:

- Is it normally distributed, skewed, bimodal?
- Are there impossible values (negative age, 200% completion rate)?
- What is the range? Does it match your domain expectations?

### Phase 4 — Bivariate Analysis (15 minutes)

Focus on relationships between features and the target. This is where feature engineering ideas come from.

```python
target = "churn"   # replace with your actual target

# Numeric features vs target
numeric_feats = [c for c in df.select_dtypes(include=np.number).columns if c != target]
n = len(numeric_feats)
fig, axes = plt.subplots(nrows=(n + 2) // 3, ncols=3, figsize=(15, 4 * ((n + 2) // 3)))
axes = axes.flatten()

for i, col in enumerate(numeric_feats):
    sns.boxplot(data=df, x=target, y=col, ax=axes[i])
    axes[i].set_title(f"{col} vs {target}")

for j in range(i + 1, len(axes)):
    axes[j].set_visible(False)

plt.tight_layout()
plt.savefig("bivariate_target.png")

# Categorical features vs target
cat_feats = [c for c in df.select_dtypes("object").columns if c != target]
for col in cat_feats:
    ct = pd.crosstab(df[col], df[target], normalize="index") * 100
    print(f"\n{col} vs {target}:")
    print(ct.round(1))
```

### Phase 5 — Correlation and Multicollinearity (5 minutes)

```python
corr = df[numeric_feats].corr()

fig, ax = plt.subplots(figsize=(10, 8))
mask = np.triu(np.ones_like(corr, dtype=bool))
sns.heatmap(corr, mask=mask, annot=True, fmt=".2f",
            cmap="coolwarm", center=0, ax=ax, annot_kws={"size": 8})
ax.set_title("Feature Correlation Matrix")
plt.tight_layout()
plt.savefig("correlation_matrix.png")
```

Flag pairs with |r| > 0.85 — highly correlated features cause problems in linear models and confuse feature importance. You will need to decide which one to keep.

### Phase 6 — Outlier Scan (5 minutes)

```python
def flag_outliers_iqr(series):
    q1, q3 = series.quantile(0.25), series.quantile(0.75)
    iqr = q3 - q1
    lower, upper = q1 - 1.5 * iqr, q3 + 1.5 * iqr
    return ((series < lower) | (series > upper)).sum()

outlier_counts = {col: flag_outliers_iqr(df[col]) for col in numeric_feats}
print(pd.Series(outlier_counts).sort_values(ascending=False))
```

For each column with significant outliers, look at the actual rows — not just the count.

---

## Writing Insights, Not Just Charts

Every chart should produce a written finding. Use this template:

```
Observation:   [What the data shows, quantified]
Interpretation: [Why this might be happening]
Action:         [What to do with this in modelling/cleaning]
```

Examples:

```
Observation:   Premium customers churn at 22% vs 35% for Basic customers.
Interpretation: Higher investment/engagement in the product may reduce churn risk.
Action:         Include segment as a feature. Consider a segment-specific model.

Observation:   Revenue has 8 outliers above ₹50,000 (IQR upper fence: ₹31,200).
Interpretation: Likely enterprise clients — a genuine different customer type.
Action:         Do not remove. Add a binary flag is_enterprise for the model.

Observation:   support_calls and churn_rate have a Pearson r of 0.71.
Interpretation: Dissatisfied customers call more before leaving.
Action:         Strong candidate for a model feature. Consider interaction with recency.
```

> [!tip] Quantity of insights
> A good EDA on a business dataset should produce at least 10 written insights — not 10 charts. If you have 40 charts and 2 sentences, you did visualisation, not analysis.

---

## EDA Deliverable Checklist

Before handing off to modelling, confirm you have:

- [ ] Documented what each row represents
- [ ] Identified the target variable
- [ ] Logged all missing value patterns and your handling decision for each
- [ ] Noted all duplicate rows and whether they were removed
- [ ] Flagged impossible/suspicious values
- [ ] Produced distribution plots for all numeric features
- [ ] Produced value_counts for all categorical features
- [ ] Analysed each feature against the target
- [ ] Identified highly correlated feature pairs
- [ ] Flagged outliers and decided: error, genuine extreme, or separate segment
- [ ] Written at least 8 insights in the Observation / Interpretation / Action format
- [ ] Listed 3+ feature engineering ideas that came out of the analysis

---

## Common EDA Mistakes

> [!warning] Plotting without a question
> "Let me just see what this looks like" produces nothing useful. Before every chart, state the question it answers. "Does ad spend predict revenue?" "Do churned customers make more support calls?"

> [!warning] Doing EDA once
> EDA is not a one-time step. After feature engineering, run it again. After merging in new data, run it again. After a model performs unexpectedly, run it again. The data changes, and your understanding of it should too.

> [!warning] Reporting outliers without investigating
> Flagging outliers and then dropping them is not analysis — it is data loss. Investigate each outlier. Sometimes they are the most interesting rows in the dataset.

---

[[05-bivariate-analysis]] | [[07-mini-case-study]]
