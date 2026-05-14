# 🔥 04 — Heatmaps
## Visualizing Tables, Matrices, and Correlations

> [!info] Goal
> Learn how to use heatmaps to show patterns in numeric matrices and correlation tables.

---

## What is a Heatmap?

A **heatmap** uses color intensity to represent numeric values.

Heatmaps are useful for:

- correlation matrices
- missing value patterns
- sales by month and category
- confusion matrices
- pivot tables

---

## Setup

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

---

## Basic Heatmap

```python
data = pd.DataFrame({
    "Math": [80, 90, 75],
    "Science": [85, 88, 92],
    "English": [78, 82, 80]
}, index=["Alice", "Bob", "Charlie"])

sns.heatmap(data, annot=True, cmap="Blues")
plt.title("Student Scores")
plt.show()
```

`annot=True` prints values inside cells.

---

## Correlation Heatmap

```python
df = pd.DataFrame({
    "study_hours": [2, 3, 4, 5, 6],
    "attendance": [70, 75, 80, 85, 90],
    "score": [60, 65, 72, 78, 85],
    "sleep_hours": [6, 7, 7, 8, 8]
})

corr = df.corr(numeric_only=True)

sns.heatmap(corr, annot=True, cmap="coolwarm", center=0)
plt.title("Correlation Matrix")
plt.show()
```

Correlation values range from `-1` to `1`.

| Value | Meaning |
|-------|---------|
| Near `1` | Strong positive relationship |
| Near `0` | Weak/no linear relationship |
| Near `-1` | Strong negative relationship |

---

## Pivot Table Heatmap

```python
sales = pd.DataFrame({
    "month": ["Jan", "Jan", "Feb", "Feb", "Mar", "Mar"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories"],
    "revenue": [150000, 9000, 180000, 12000, 210000, 15000]
})

pivot = sales.pivot_table(
    values="revenue",
    index="category",
    columns="month",
    aggfunc="sum"
)

sns.heatmap(pivot, annot=True, fmt=".0f", cmap="YlGnBu")
plt.title("Revenue by Category and Month")
plt.show()
```

Pivot table heatmaps are excellent for business summaries.

---

## Missing Value Heatmap

```python
customers = pd.DataFrame({
    "name": ["Alice", "Bob", None, "Diana"],
    "age": [25, None, 32, 29],
    "city": ["Delhi", "Mumbai", None, "Pune"]
})

sns.heatmap(customers.isna(), cbar=False, cmap="Reds")
plt.title("Missing Value Pattern")
plt.show()
```

This helps reveal where missing values are concentrated.

---

## Heatmap Options

```python
sns.heatmap(
    corr,
    annot=True,
    cmap="coolwarm",
    center=0,
    linewidths=0.5,
    fmt=".2f"
)

plt.show()
```

| Option | Purpose |
|--------|---------|
| `annot=True` | Show values in cells |
| `fmt=".2f"` | Format values to 2 decimals |
| `cmap="coolwarm"` | Color palette |
| `center=0` | Center colors around zero |
| `linewidths=0.5` | Add grid lines |
| `cbar=False` | Hide color bar |

---

## Mask Upper Triangle

For correlation matrices, the top and bottom triangles repeat the same information.

```python
import numpy as np

mask = np.triu(np.ones_like(corr, dtype=bool))

sns.heatmap(
    corr,
    mask=mask,
    annot=True,
    cmap="coolwarm",
    center=0
)

plt.title("Correlation Matrix")
plt.show()
```

---

## Common Mistakes

### Using Heatmaps for Too Many Categories

If there are hundreds of rows or columns, the heatmap becomes unreadable.

### Forgetting to Format Large Numbers

Use `fmt=".0f"` for whole numbers or `fmt=".2f"` for decimals.

### Overinterpreting Correlation

Correlation does not prove causation.

---

## Practice

Tasks:

- create a correlation heatmap for a numeric DataFrame
- create a pivot table of revenue by category and month
- visualize the pivot table as a heatmap
- create a missing value heatmap
- mask the upper triangle of a correlation heatmap

---

## Interview Questions

**Q1:** What does a heatmap show?

> It shows numeric values using color intensity.

**Q2:** Why use `center=0` for correlation heatmaps?

> It makes positive and negative correlations visually balanced around zero.

**Q3:** What does `annot=True` do?

> It displays the numeric value inside each heatmap cell.

---

## ✅ Key Takeaways

- Heatmaps are useful for matrices, pivot tables, and correlations.
- Use `annot=True` for readability.
- Use `cmap` and `center` thoughtfully.
- Correlation heatmaps help identify relationships between numeric variables.
- Missing value heatmaps help diagnose data quality.

---

## 🔗 What's Next?

➡️ [[05-visualization-best-practices]] — Make charts clearer and more honest
