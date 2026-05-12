# 📊 02 — Line, Bar, and Histogram Charts
## Choosing the Right Basic Plot

> [!info] Goal
> Learn when and how to use line charts, bar charts, histograms, and scatter plots.

---

## Sample Dataset

```python
import pandas as pd
import matplotlib.pyplot as plt

sales = pd.DataFrame({
    "month": ["Jan", "Feb", "Mar", "Apr", "May", "Jun"],
    "revenue": [12000, 15000, 14000, 18000, 21000, 24000],
    "orders": [120, 150, 135, 170, 190, 220],
    "category": ["A", "B", "A", "C", "B", "C"]
})
```

---

## Line Chart

Use a line chart for trends over time.

```python
plt.figure(figsize=(8, 4))
plt.plot(sales["month"], sales["revenue"], marker="o")
plt.title("Monthly Revenue Trend")
plt.xlabel("Month")
plt.ylabel("Revenue")
plt.grid(True)
plt.show()
```

Best for:

- time series
- trend tracking
- continuous ordered data

---

## Bar Chart

Use a bar chart to compare categories.

```python
plt.figure(figsize=(8, 4))
plt.bar(sales["month"], sales["orders"])
plt.title("Orders by Month")
plt.xlabel("Month")
plt.ylabel("Orders")
plt.show()
```

Best for:

- category comparison
- counts
- totals by group

---

## Horizontal Bar Chart

Horizontal bars work well for long category names.

```python
category_revenue = sales.groupby("category")["revenue"].sum()

plt.figure(figsize=(7, 4))
plt.barh(category_revenue.index, category_revenue.values)
plt.title("Revenue by Category")
plt.xlabel("Revenue")
plt.ylabel("Category")
plt.show()
```

---

## Histogram

Use a histogram to understand the distribution of a numeric variable.

```python
scores = [55, 60, 62, 70, 72, 75, 78, 80, 82, 85, 90, 95]

plt.figure(figsize=(7, 4))
plt.hist(scores, bins=5, edgecolor="black")
plt.title("Score Distribution")
plt.xlabel("Score")
plt.ylabel("Frequency")
plt.show()
```

Best for:

- distributions
- skewness
- spread
- outlier detection

---

## Scatter Plot

Use a scatter plot to study the relationship between two numeric variables.

```python
plt.figure(figsize=(7, 4))
plt.scatter(sales["orders"], sales["revenue"])
plt.title("Orders vs Revenue")
plt.xlabel("Orders")
plt.ylabel("Revenue")
plt.show()
```

Best for:

- correlation
- clusters
- outliers
- numeric relationships

---

## Box Plot

Use a box plot to summarize spread and outliers.

```python
plt.figure(figsize=(6, 4))
plt.boxplot(scores)
plt.title("Score Spread")
plt.ylabel("Score")
plt.show()
```

Box plots show median, quartiles, and possible outliers.

---

## Chart Selection Guide

| Question | Chart |
|----------|-------|
| How does revenue change over time? | Line chart |
| Which category has highest revenue? | Bar chart |
| What is the distribution of scores? | Histogram |
| Are two numeric variables related? | Scatter plot |
| Are there outliers? | Box plot |

---

## Pandas Plot Shortcut

```python
sales.plot(x="month", y="revenue", kind="line", marker="o")
plt.show()
```

```python
sales.plot(x="month", y="orders", kind="bar")
plt.show()
```

```python
sales["revenue"].plot(kind="hist", bins=5)
plt.show()
```

Pandas plotting is convenient for quick exploration.

---

## Subplots for Comparison

```python
fig, axes = plt.subplots(1, 2, figsize=(10, 4))

axes[0].plot(sales["month"], sales["revenue"], marker="o")
axes[0].set_title("Revenue")

axes[1].bar(sales["month"], sales["orders"])
axes[1].set_title("Orders")

plt.tight_layout()
plt.show()
```

---

## Practice

Using the `sales` DataFrame:

- create a line chart of revenue by month
- create a bar chart of orders by month
- create a histogram of revenue
- create a scatter plot of orders vs revenue
- create a horizontal bar chart of total revenue by category

---

## Interview Questions

**Q1:** When should you use a line chart?

> Use a line chart to show trends over ordered data, especially time.

**Q2:** What does a histogram show?

> It shows the distribution of a numeric variable.

**Q3:** What chart helps show relationship between two numeric variables?

> A scatter plot.

---

## ✅ Key Takeaways

- Line charts show trends.
- Bar charts compare categories.
- Histograms show distributions.
- Scatter plots show numeric relationships.
- Box plots help detect spread and outliers.

---

## 🔗 What's Next?

➡️ [[03-seaborn-basics]] — Create cleaner statistical charts faster
