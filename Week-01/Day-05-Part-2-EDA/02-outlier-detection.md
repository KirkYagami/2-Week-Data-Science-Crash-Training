# 🚩 02 — Outlier Detection
## Finding Unusual Values

> [!info] Goal
> Learn how to detect, inspect, and handle outliers during EDA.

---

## What is an Outlier?

An outlier is a value that is unusually far from the rest of the data.

Examples:

- age = 150
- salary = 10000000
- order quantity = 0
- transaction amount = 50x larger than usual

Outliers can be errors or meaningful rare events.

---

## Visual Detection

```python
import matplotlib.pyplot as plt
import seaborn as sns

sns.boxplot(x=df["revenue"])
plt.show()

sns.histplot(df["revenue"], bins=30)
plt.show()
```

---

## IQR Method

```python
q1 = df["revenue"].quantile(0.25)
q3 = df["revenue"].quantile(0.75)
iqr = q3 - q1

lower = q1 - 1.5 * iqr
upper = q3 + 1.5 * iqr

outliers = df[(df["revenue"] < lower) | (df["revenue"] > upper)]
```

---

## Z-Score Method

```python
z = (df["revenue"] - df["revenue"].mean()) / df["revenue"].std()
outliers = df[z.abs() > 3]
```

Z-score works best when data is roughly normal.

---

## Handling Outliers

Options:

- keep them
- remove them
- cap/winsorize them
- transform the column
- investigate manually
- create an outlier flag

```python
df["revenue_outlier"] = (df["revenue"] < lower) | (df["revenue"] > upper)
df["revenue_capped"] = df["revenue"].clip(lower, upper)
```

---

## Common Mistakes

- Removing outliers automatically.
- Treating rare valid values as errors.
- Using z-score on heavily skewed data.
- Not documenting outlier decisions.

---

## Practice

Use:

```python
sales = pd.Series([100, 120, 130, 125, 140, 1000])
```

Tasks:

- plot histogram
- calculate IQR bounds
- detect outliers
- cap outliers
- explain whether to keep or remove

---

## Interview Questions

**Q1:** What is an outlier?

> A value unusually far from most other values.

**Q2:** What is the IQR rule?

> Values below Q1 - 1.5*IQR or above Q3 + 1.5*IQR are potential outliers.

**Q3:** Should all outliers be removed?

> No. First determine whether they are errors or valid rare events.

---

## 🔗 Next

➡️ [[03-feature-understanding]]
