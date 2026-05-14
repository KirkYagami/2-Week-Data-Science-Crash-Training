# 🎨 03 — Seaborn Basics
## Statistical Charts with Less Code

> [!info] Goal
> Learn how Seaborn simplifies common statistical visualizations.

---

## What is Seaborn?

**Seaborn** is a Python visualization library built on top of Matplotlib.

It is especially useful for:

- statistical charts
- grouped visualizations
- attractive default styling
- working directly with Pandas DataFrames

---

## Setup

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="whitegrid")
```

---

## Sample Dataset

```python
tips = sns.load_dataset("tips")

print(tips.head())
```

If you do not have internet or bundled datasets available, create a simple dataset:

```python
tips = pd.DataFrame({
    "total_bill": [16.99, 10.34, 21.01, 23.68, 24.59, 25.29, 8.77],
    "tip": [1.01, 1.66, 3.50, 3.31, 3.61, 4.71, 2.00],
    "sex": ["Female", "Male", "Male", "Male", "Female", "Male", "Male"],
    "day": ["Sun", "Sun", "Sun", "Sun", "Sun", "Sun", "Sun"],
    "time": ["Dinner", "Dinner", "Dinner", "Dinner", "Dinner", "Dinner", "Dinner"]
})
```

---

## Scatter Plot

```python
sns.scatterplot(
    data=tips,
    x="total_bill",
    y="tip"
)

plt.title("Total Bill vs Tip")
plt.show()
```

Add color by category:

```python
sns.scatterplot(
    data=tips,
    x="total_bill",
    y="tip",
    hue="sex"
)

plt.show()
```

---

## Line Plot

```python
monthly = pd.DataFrame({
    "month": ["Jan", "Feb", "Mar", "Apr"],
    "revenue": [12000, 15000, 14000, 18000]
})

sns.lineplot(data=monthly, x="month", y="revenue", marker="o")
plt.title("Monthly Revenue")
plt.show()
```

---

## Bar Plot

```python
sns.barplot(data=tips, x="day", y="total_bill")
plt.title("Average Bill by Day")
plt.show()
```

By default, Seaborn bar plots show an aggregate, usually the mean.

---

## Count Plot

Use count plots for category counts.

```python
sns.countplot(data=tips, x="day")
plt.title("Orders by Day")
plt.show()
```

---

## Histogram

```python
sns.histplot(data=tips, x="total_bill", bins=10, kde=True)
plt.title("Distribution of Total Bill")
plt.show()
```

`kde=True` adds a smooth density curve.

---

## Box Plot

```python
sns.boxplot(data=tips, x="day", y="total_bill")
plt.title("Bill Distribution by Day")
plt.show()
```

Box plots are good for comparing distributions across categories.

---

## Pair Plot

```python
sns.pairplot(tips[["total_bill", "tip"]])
plt.show()
```

Pair plots help quickly inspect relationships between numeric columns.

---

## Faceting with `col`

Some Seaborn functions can create multiple charts by category.

```python
sns.relplot(
    data=tips,
    x="total_bill",
    y="tip",
    hue="sex",
    col="time"
)

plt.show()
```

Facets are useful when one chart would become too crowded.

---

## Customizing Seaborn Charts

```python
plt.figure(figsize=(8, 4))
sns.barplot(data=tips, x="day", y="total_bill", hue="sex")
plt.title("Average Bill by Day and Gender")
plt.xlabel("Day")
plt.ylabel("Average Bill")
plt.legend(title="Gender")
plt.show()
```

Seaborn creates the chart, Matplotlib customizes the final details.

---

## Common Seaborn Functions

| Function | Use |
|----------|-----|
| `sns.scatterplot()` | Numeric relationship |
| `sns.lineplot()` | Trend |
| `sns.barplot()` | Aggregated category comparison |
| `sns.countplot()` | Category counts |
| `sns.histplot()` | Distribution |
| `sns.boxplot()` | Spread and outliers |
| `sns.heatmap()` | Matrix/correlation visualization |
| `sns.pairplot()` | Multiple numeric relationships |

---

## Practice

Using `tips`:

- create a scatter plot of total bill vs tip
- color scatter points by sex
- create a histogram of total bill
- create a count plot of day
- create a box plot of total bill by day
- create a bar plot of average tip by day

---

## Interview Questions

**Q1:** What is Seaborn built on top of?

> Matplotlib.

**Q2:** Why use Seaborn instead of only Matplotlib?

> Seaborn creates statistical charts with cleaner defaults and works naturally with Pandas DataFrames.

**Q3:** What is the difference between `barplot()` and `countplot()`?

> `barplot()` shows an aggregated numeric value. `countplot()` counts rows in categories.

---

## ✅ Key Takeaways

- Seaborn is excellent for statistical visualizations.
- It works well with Pandas DataFrames.
- Use `hue` to show categories with color.
- Use Matplotlib functions to customize Seaborn charts.
- Use box plots and histograms to understand distributions.

---

## 🔗 What's Next?

➡️ [[04-heatmaps]] — Visualize correlations and matrices
