# 🧪 07 — Mini Case Study: Customer EDA

> [!info] Goal
> Apply the Week 01 toolchain to a small customer dataset.

---

## Dataset

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4, 5, 6, 7, 8],
    "age": [22, 35, 42, 29, 31, 55, np.nan, 47],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune", "Mumbai", "Delhi", "Pune", None],
    "segment": ["Basic", "Premium", "Premium", "Basic", "Basic", "Premium", "Basic", "Premium"],
    "monthly_spend": [500, 2500, 3200, 700, 650, 4000, 800, 10000],
    "churn": ["Yes", "No", "No", "Yes", "Yes", "No", "Yes", "No"]
})
```

---

## Step 1 — Inspect

```python
print(customers.head())
print(customers.shape)
print(customers.info())
print(customers.isna().sum())
print(customers.describe())
```

---

## Step 2 — Clean

```python
customers["city"] = customers["city"].fillna("Unknown")
customers["age_missing"] = customers["age"].isna()
customers["age"] = customers["age"].fillna(customers["age"].median())
```

---

## Step 3 — Univariate Analysis

```python
sns.histplot(customers["monthly_spend"], bins=6, kde=True)
plt.title("Monthly Spend Distribution")
plt.show()

print(customers["segment"].value_counts())
print(customers["churn"].value_counts())
```

---

## Step 4 — Bivariate Analysis

Spend by churn:

```python
sns.boxplot(data=customers, x="churn", y="monthly_spend")
plt.title("Monthly Spend by Churn")
plt.show()
```

Churn by segment:

```python
print(pd.crosstab(customers["segment"], customers["churn"], normalize="index") * 100)
```

Age vs spend:

```python
sns.scatterplot(data=customers, x="age", y="monthly_spend", hue="churn")
plt.title("Age vs Monthly Spend")
plt.show()
```

---

## Step 5 — Outlier Check

```python
q1 = customers["monthly_spend"].quantile(0.25)
q3 = customers["monthly_spend"].quantile(0.75)
iqr = q3 - q1

lower = q1 - 1.5 * iqr
upper = q3 + 1.5 * iqr

outliers = customers[
    (customers["monthly_spend"] < lower) |
    (customers["monthly_spend"] > upper)
]

print(outliers)
```

---

## Step 6 — Example Insights

- Premium customers appear less likely to churn than Basic customers.
- Monthly spend is right-skewed because one customer spends much more than others.
- Missing age was handled with median imputation and tracked with an indicator.
- Churn analysis should separate Basic and Premium customers.

---

## Final Deliverable

Create a short report with:

- dataset summary
- cleaning steps
- missing value handling
- 3 univariate charts or tables
- 3 bivariate charts or tables
- outlier notes
- 5 insights
- 3 next steps

---

## Interview Questions

**Q1:** What did you learn from this EDA?

> Segment and spend appear related to churn, but the dataset is too small for final conclusions.

**Q2:** What would you do next?

> Collect more data, validate patterns on a larger sample, engineer features, and prepare for modeling.

**Q3:** Why keep an age missing indicator?

> Missingness itself may contain useful signal.

---

## ✅ Week 01 Wrap-Up

You have now practiced:

- Python fundamentals
- NumPy
- Pandas
- data visualization
- statistics
- SQL
- EDA workflow

This is the foundation for Week 02 machine learning.

---

## 🔗 Next

➡️ [[../../Week-02/Day-01-Part-1-Machine-Learning-Basics/00-notes|Week 02 — Machine Learning Basics]]
