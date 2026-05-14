# ✅ 05 — Visualization Best Practices
## Make Charts Clear, Honest, and Useful

> [!info] Goal
> Learn how to choose better charts, reduce clutter, and communicate insights responsibly.

---

## The Purpose of a Chart

A chart should help someone understand data faster.

Before plotting, ask:

- What question am I answering?
- Who is the audience?
- What should they notice first?
- Is a chart better than a table here?

---

## Choose the Right Chart

| Goal | Good Choice |
|------|-------------|
| Show trend over time | Line chart |
| Compare categories | Bar chart |
| Show distribution | Histogram or box plot |
| Show relationship | Scatter plot |
| Show correlation matrix | Heatmap |
| Show part-to-whole | Bar chart, stacked bar with care |

Avoid choosing charts only because they look fancy.

---

## Always Add Context

Good charts usually include:

- clear title
- x-axis label
- y-axis label
- units
- legend when needed
- readable tick labels

```python
plt.title("Monthly Revenue in 2025")
plt.xlabel("Month")
plt.ylabel("Revenue (INR)")
```

---

## Keep It Simple

Remove anything that does not help understanding:

- unnecessary 3D effects
- too many colors
- too many grid lines
- crowded labels
- decorative styling

Simple charts are easier to trust.

---

## Sort Bar Charts

Sorted bar charts are easier to read.

```python
category_revenue = df.groupby("category")["revenue"].sum().sort_values()

plt.barh(category_revenue.index, category_revenue.values)
plt.title("Revenue by Category")
plt.xlabel("Revenue")
plt.show()
```

Horizontal bars often work better for category labels.

---

## Use Color Carefully

Use color to communicate meaning, not decoration.

Good uses:

- highlight one important category
- separate groups
- show high vs low values
- indicate positive vs negative values

Bad uses:

- random rainbow colors
- too many categories
- colors that make labels unreadable

---

## Avoid Misleading Axes

Bar charts should usually start at zero.

```python
plt.ylim(0, max_value)
```

Line charts can sometimes use a non-zero axis, but be transparent and careful.

---

## Handle Outliers Honestly

Outliers can compress the rest of a chart.

Options:

- show the outlier and mention it
- use a log scale when appropriate
- use a box plot
- create a separate zoomed chart

Do not silently remove outliers without explaining why.

---

## Make Text Readable

```python
plt.xticks(rotation=45)
plt.tight_layout()
```

Use rotation when category labels overlap.

For larger charts:

```python
plt.figure(figsize=(10, 5))
```

---

## Use Annotations Sparingly

```python
plt.annotate(
    "Highest revenue",
    xy=("May", 21000),
    xytext=("Apr", 23000),
    arrowprops={"arrowstyle": "->"}
)
```

Annotations are powerful when they highlight a real insight.

---

## Common Bad Chart Choices

### Pie Charts with Many Categories

Use a bar chart instead.

### 3D Charts

They usually distort perception.

### Dual Axis Charts

They can be confusing. Use subplots if possible.

### Too Many Lines

If many lines overlap, filter, group, or use small multiples.

---

## A Good Visualization Workflow

1. Start with a question.
2. Prepare the data.
3. Choose the simplest chart that answers the question.
4. Add labels and units.
5. Remove clutter.
6. Check if the chart can be misread.
7. Write the insight in one sentence.

---

## Example: Before and After

### Weak Chart

```python
df.groupby("category")["revenue"].sum().plot(kind="bar")
plt.show()
```

### Better Chart

```python
category_revenue = (
    df.groupby("category")["revenue"]
    .sum()
    .sort_values()
)

plt.figure(figsize=(8, 4))
plt.barh(category_revenue.index, category_revenue.values, color="steelblue")
plt.title("Electronics Drives the Highest Revenue")
plt.xlabel("Revenue (INR)")
plt.ylabel("Category")
plt.tight_layout()
plt.show()
```

The second chart is sorted, labeled, sized, and has a useful title.

---

## Practice

Take any chart from previous notes and improve it:

- add a clearer title
- add units
- rotate labels if needed
- sort categories
- remove unnecessary styling
- use a better chart type if appropriate
- write one sentence explaining the insight

---

## Interview Questions

**Q1:** What makes a visualization effective?

> It answers a clear question accurately, with minimal clutter and enough context for the audience.

**Q2:** Why are 3D charts usually discouraged?

> They distort perception and make values harder to compare.

**Q3:** Why should bar charts usually start at zero?

> A non-zero baseline can exaggerate differences between categories.

---

## ✅ Key Takeaways

- Choose charts based on the question.
- Label titles, axes, units, and legends clearly.
- Prefer simple charts over decorative charts.
- Sort categories when comparison matters.
- Use color intentionally.
- Avoid misleading scales and unexplained data removal.

---

## 🔗 What's Next?

➡️ [[06-mini-project]] — Build a small visualization report
