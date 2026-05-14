# 🧪 06 — Mini Project: Sales Visualization Report

> [!info] Goal
> Use Pandas, Matplotlib, and Seaborn to create a small visual analysis report.

---

## Project Brief

You are given sales data for a small business. Your task is to create a short visualization report that answers:

- How is revenue changing over time?
- Which category generates the most revenue?
- Which city has the most orders?
- What is the relationship between orders and revenue?
- Are there useful patterns in category-month revenue?

---

## Dataset

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="whitegrid")

sales = pd.DataFrame({
    "month": ["Jan", "Jan", "Feb", "Feb", "Mar", "Mar", "Apr", "Apr", "May", "May", "Jun", "Jun"],
    "city": ["Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories"],
    "orders": [120, 80, 150, 95, 135, 110, 170, 130, 190, 145, 220, 160],
    "revenue": [120000, 32000, 150000, 42000, 140000, 50000, 180000, 62000, 210000, 70000, 240000, 85000]
})
```

---

## Step 1 — Inspect the Data

```python
print(sales.head())
print(sales.shape)
print(sales.info())
print(sales.isna().sum())
```

Write down:

- number of rows
- number of columns
- column names
- whether missing values exist

---

## Step 2 — Revenue Trend Over Time

```python
monthly_revenue = sales.groupby("month", as_index=False)["revenue"].sum()

month_order = ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]
monthly_revenue["month"] = pd.Categorical(
    monthly_revenue["month"],
    categories=month_order,
    ordered=True
)
monthly_revenue = monthly_revenue.sort_values("month")

plt.figure(figsize=(8, 4))
sns.lineplot(data=monthly_revenue, x="month", y="revenue", marker="o")
plt.title("Revenue Increased Steadily from January to June")
plt.xlabel("Month")
plt.ylabel("Revenue (INR)")
plt.tight_layout()
plt.show()
```

Insight example:

> Revenue grew month by month, reaching the highest value in June.

---

## Step 3 — Revenue by Category

```python
category_revenue = (
    sales.groupby("category", as_index=False)["revenue"]
    .sum()
    .sort_values("revenue")
)

plt.figure(figsize=(7, 4))
sns.barplot(data=category_revenue, x="revenue", y="category")
plt.title("Electronics Generates More Revenue Than Accessories")
plt.xlabel("Revenue (INR)")
plt.ylabel("Category")
plt.tight_layout()
plt.show()
```

---

## Step 4 — Orders by City

```python
city_orders = (
    sales.groupby("city", as_index=False)["orders"]
    .sum()
    .sort_values("orders", ascending=False)
)

plt.figure(figsize=(7, 4))
sns.barplot(data=city_orders, x="city", y="orders")
plt.title("Total Orders by City")
plt.xlabel("City")
plt.ylabel("Orders")
plt.tight_layout()
plt.show()
```

---

## Step 5 — Orders vs Revenue

```python
plt.figure(figsize=(7, 4))
sns.scatterplot(
    data=sales,
    x="orders",
    y="revenue",
    hue="category",
    style="city",
    s=100
)

plt.title("Orders and Revenue Move Together")
plt.xlabel("Orders")
plt.ylabel("Revenue (INR)")
plt.tight_layout()
plt.show()
```

---

## Step 6 — Category-Month Heatmap

```python
pivot = sales.pivot_table(
    values="revenue",
    index="category",
    columns="month",
    aggfunc="sum"
)

pivot = pivot[month_order]

plt.figure(figsize=(8, 3))
sns.heatmap(pivot, annot=True, fmt=".0f", cmap="YlGnBu")
plt.title("Revenue by Category and Month")
plt.xlabel("Month")
plt.ylabel("Category")
plt.tight_layout()
plt.show()
```

---

## Step 7 — Save Figures

```python
plt.figure(figsize=(8, 4))
sns.lineplot(data=monthly_revenue, x="month", y="revenue", marker="o")
plt.title("Revenue Increased Steadily from January to June")
plt.xlabel("Month")
plt.ylabel("Revenue (INR)")
plt.tight_layout()
plt.savefig("revenue_trend.png", dpi=300, bbox_inches="tight")
plt.show()
```

Repeat this pattern for other charts if needed.

---

## Final Report Checklist

Your report should include:

- [ ] data inspection summary
- [ ] line chart for revenue trend
- [ ] bar chart for revenue by category
- [ ] bar chart for orders by city
- [ ] scatter plot for orders vs revenue
- [ ] heatmap for category-month revenue
- [ ] one written insight per chart

---

## Stretch Goals

- Add a chart showing average revenue per order.
- Add city as a filter by creating separate charts for Delhi and Mumbai.
- Save all charts as image files.
- Create a single figure with multiple subplots.
- Export summary tables to CSV.

---

## One-File Solution Template

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="whitegrid")

sales = pd.DataFrame({
    "month": ["Jan", "Jan", "Feb", "Feb", "Mar", "Mar", "Apr", "Apr", "May", "May", "Jun", "Jun"],
    "city": ["Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai", "Delhi", "Mumbai"],
    "category": ["Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories", "Electronics", "Accessories"],
    "orders": [120, 80, 150, 95, 135, 110, 170, 130, 190, 145, 220, 160],
    "revenue": [120000, 32000, 150000, 42000, 140000, 50000, 180000, 62000, 210000, 70000, 240000, 85000]
})

month_order = ["Jan", "Feb", "Mar", "Apr", "May", "Jun"]

monthly_revenue = sales.groupby("month", as_index=False)["revenue"].sum()
monthly_revenue["month"] = pd.Categorical(monthly_revenue["month"], categories=month_order, ordered=True)
monthly_revenue = monthly_revenue.sort_values("month")

plt.figure(figsize=(8, 4))
sns.lineplot(data=monthly_revenue, x="month", y="revenue", marker="o")
plt.title("Revenue Increased Steadily from January to June")
plt.xlabel("Month")
plt.ylabel("Revenue (INR)")
plt.tight_layout()
plt.show()

category_revenue = sales.groupby("category", as_index=False)["revenue"].sum().sort_values("revenue")

plt.figure(figsize=(7, 4))
sns.barplot(data=category_revenue, x="revenue", y="category")
plt.title("Electronics Generates More Revenue Than Accessories")
plt.xlabel("Revenue (INR)")
plt.ylabel("Category")
plt.tight_layout()
plt.show()

city_orders = sales.groupby("city", as_index=False)["orders"].sum().sort_values("orders", ascending=False)

plt.figure(figsize=(7, 4))
sns.barplot(data=city_orders, x="city", y="orders")
plt.title("Total Orders by City")
plt.xlabel("City")
plt.ylabel("Orders")
plt.tight_layout()
plt.show()

plt.figure(figsize=(7, 4))
sns.scatterplot(data=sales, x="orders", y="revenue", hue="category", style="city", s=100)
plt.title("Orders and Revenue Move Together")
plt.xlabel("Orders")
plt.ylabel("Revenue (INR)")
plt.tight_layout()
plt.show()

pivot = sales.pivot_table(values="revenue", index="category", columns="month", aggfunc="sum")
pivot = pivot[month_order]

plt.figure(figsize=(8, 3))
sns.heatmap(pivot, annot=True, fmt=".0f", cmap="YlGnBu")
plt.title("Revenue by Category and Month")
plt.xlabel("Month")
plt.ylabel("Category")
plt.tight_layout()
plt.show()
```

---

## Interview Questions

**Q1:** What should a visualization report include?

> Clear charts, concise insights, labels, units, and enough context for the audience to understand the finding.

**Q2:** Why write insights below charts?

> The chart shows evidence, but the insight explains what the audience should take away.

**Q3:** Why sort months manually?

> Text months may sort alphabetically. A categorical order ensures the timeline appears correctly.

---

## ✅ Key Takeaways

- A visualization report should answer specific questions.
- Prepare grouped summary tables before plotting.
- Use chart titles that communicate insights.
- Use line, bar, scatter, and heatmap charts for different questions.
- Save important figures for reports and presentations.

---

## 🔗 Navigation

| Previous | Next |
|----------|------|
| [[05-visualization-best-practices]] | [[../Day-04-Part-1-Statistics-Basics/01-mean-median-mode|Statistics Basics]] |

---

*Tags: #data-visualization #matplotlib #seaborn #mini-project #eda*
