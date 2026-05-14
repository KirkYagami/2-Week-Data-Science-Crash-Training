# 📈 01 — Matplotlib Basics
## Your First Python Charts

> [!info] Goal
> Learn the basic Matplotlib workflow: create a figure, plot data, label the chart, customize it, and save it.

---

## Why Visualization Matters

Data visualization helps you:

- detect trends
- compare categories
- find outliers
- explain results to others
- validate assumptions before modeling

In Data Science, a good chart often reveals what a table hides.

---

## Setup

```python
import matplotlib.pyplot as plt
```

Common notebook setting:

```python
%matplotlib inline
```

---

## Basic Line Plot

```python
days = [1, 2, 3, 4, 5]
sales = [100, 130, 120, 160, 180]

plt.plot(days, sales)
plt.show()
```

`plt.show()` displays the chart.

---

## Add Title and Labels

```python
plt.plot(days, sales)
plt.title("Daily Sales")
plt.xlabel("Day")
plt.ylabel("Sales")
plt.show()
```

Always label charts clearly.

---

## Figure Size

```python
plt.figure(figsize=(8, 4))
plt.plot(days, sales)
plt.title("Daily Sales")
plt.xlabel("Day")
plt.ylabel("Sales")
plt.show()
```

`figsize=(width, height)` is measured in inches.

---

## Styling Lines

```python
plt.plot(
    days,
    sales,
    color="green",
    marker="o",
    linestyle="--",
    linewidth=2
)

plt.title("Daily Sales")
plt.xlabel("Day")
plt.ylabel("Sales")
plt.show()
```

Useful options:

| Option | Example |
|--------|---------|
| `color` | `"red"`, `"blue"`, `"green"` |
| `marker` | `"o"`, `"s"`, `"x"` |
| `linestyle` | `"-"`, `"--"`, `":"` |
| `linewidth` | `2`, `3` |

---

## Multiple Lines

```python
week = [1, 2, 3, 4, 5]
online_sales = [100, 130, 120, 160, 180]
store_sales = [90, 110, 115, 140, 150]

plt.plot(week, online_sales, marker="o", label="Online")
plt.plot(week, store_sales, marker="s", label="Store")

plt.title("Online vs Store Sales")
plt.xlabel("Day")
plt.ylabel("Sales")
plt.legend()
plt.show()
```

Use `label` and `legend()` when showing multiple series.

---

## Grid

```python
plt.plot(days, sales, marker="o")
plt.title("Daily Sales")
plt.xlabel("Day")
plt.ylabel("Sales")
plt.grid(True)
plt.show()
```

Grid lines can make values easier to read.

---

## Subplots

Subplots let you show multiple charts in one figure.

```python
fig, axes = plt.subplots(1, 2, figsize=(10, 4))

axes[0].plot(days, sales, marker="o")
axes[0].set_title("Sales")
axes[0].set_xlabel("Day")
axes[0].set_ylabel("Sales")

profit = [20, 25, 22, 35, 40]
axes[1].plot(days, profit, marker="s", color="orange")
axes[1].set_title("Profit")
axes[1].set_xlabel("Day")
axes[1].set_ylabel("Profit")

plt.tight_layout()
plt.show()
```

This is the object-oriented Matplotlib style and is preferred for larger plots.

---

## Plotting from Pandas

```python
import pandas as pd

df = pd.DataFrame({
    "day": [1, 2, 3, 4, 5],
    "sales": [100, 130, 120, 160, 180]
})

plt.plot(df["day"], df["sales"], marker="o")
plt.title("Daily Sales")
plt.xlabel("Day")
plt.ylabel("Sales")
plt.show()
```

Pandas also has built-in plotting:

```python
df.plot(x="day", y="sales", kind="line", marker="o")
plt.show()
```

---

## Save a Figure

```python
plt.plot(days, sales)
plt.title("Daily Sales")
plt.xlabel("Day")
plt.ylabel("Sales")
plt.savefig("daily_sales.png", dpi=300, bbox_inches="tight")
plt.show()
```

Useful options:

- `dpi=300` for high quality
- `bbox_inches="tight"` to avoid cutting labels

---

## Common Mistakes

### Forgetting `plt.show()`

In scripts, charts may not display unless you call:

```python
plt.show()
```

### Missing Labels

A chart without labels is hard to interpret.

### Too Many Lines in One Chart

If a chart becomes crowded, use subplots or filter the data.

---

## Practice

Create a line chart showing monthly revenue:

```python
months = ["Jan", "Feb", "Mar", "Apr", "May"]
revenue = [12000, 15000, 14000, 18000, 21000]
```

Tasks:

- create a line chart
- add markers
- add title and axis labels
- add grid
- save the figure as `monthly_revenue.png`

---

## Interview Questions

**Q1:** What is Matplotlib?

> Matplotlib is a Python library for creating static, animated, and interactive visualizations.

**Q2:** Why use `plt.figure(figsize=...)`?

> It controls the size of the chart, making it easier to read.

**Q3:** What does `plt.legend()` do?

> It displays labels for plotted series.

---

## ✅ Key Takeaways

- Use `plt.plot()` for line charts.
- Always add titles and axis labels.
- Use legends for multiple series.
- Use subplots for multiple related charts.
- Save charts with `plt.savefig()`.

---

## 🔗 What's Next?

➡️ [[02-line-bar-histogram]] — Learn the most common chart types
