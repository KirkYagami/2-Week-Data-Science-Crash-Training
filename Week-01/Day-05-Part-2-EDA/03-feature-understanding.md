# 🧠 03 — Feature Understanding
## Know Your Columns Before Modeling

> [!info] Goal
> Learn how to classify, inspect, and document dataset features.

---

## What is a Feature?

A feature is an input variable used for analysis or modeling.

Examples:

- age
- income
- city
- purchase_count
- signup_date

The target is the variable you want to predict or explain.

---

## Feature Types

| Type | Examples | Common Checks |
|------|----------|---------------|
| Numeric continuous | salary, revenue | distribution, outliers |
| Numeric discrete | order_count | range, zeros |
| Categorical nominal | city, product | value counts |
| Categorical ordinal | low/medium/high | order |
| Date/time | signup_date | range, seasonality |
| Boolean | is_active | class balance |

---

## Inspect Features

```python
df.info()
df.describe()
df.describe(include="object")
df.nunique().sort_values()
```

---

## Cardinality

Cardinality is the number of unique values.

```python
df["city"].nunique()
df["city"].value_counts().head(10)
```

High-cardinality categorical features may need special handling.

---

## Target Awareness

For supervised ML, always identify:

```text
features = inputs
target = output to predict
```

Example:

```python
target = "churn"
features = [col for col in df.columns if col != target]
```

---

## Leakage Check

Data leakage happens when a feature contains information that would not be available at prediction time.

Examples:

- `cancellation_date` used to predict churn
- `final_grade` used to predict pass/fail
- `refund_issued` used to predict fraud before investigation

---

## Feature Notes Template

| Feature | Type | Meaning | Issues | Action |
|---------|------|---------|--------|--------|
| age | numeric | customer age | missing, impossible values | impute median |
| city | categorical | customer city | inconsistent casing | standardize |
| signup_date | date | account creation date | string dtype | convert datetime |

---

## Practice

Choose a dataset and create a feature summary:

- feature name
- type
- missing count
- unique count
- possible issues
- cleaning action

---

## Interview Questions

**Q1:** What is feature leakage?

> Using information in training that would not be available when making real predictions.

**Q2:** What is cardinality?

> Number of unique values in a feature.

**Q3:** Why identify feature types?

> Different feature types need different cleaning, visualization, and modeling strategies.

---

## 🔗 Next

➡️ [[04-univariate-analysis]]
