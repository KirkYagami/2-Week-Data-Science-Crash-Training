# Seaborn Basics — Statistical Charts Without the Boilerplate

Matplotlib gives you full control. Seaborn gives you statistical charts in two lines. The trade-off is worth it for EDA: seaborn handles aggregation, confidence intervals, grouping by color, and sensible defaults out of the box. The catch is understanding when you have a figure-level function and when you have an axes-level function — getting this wrong causes hours of confusion.

## Learning Objectives

- Explain the difference between figure-level and axes-level seaborn functions
- Use `hue` to split data by a categorical variable
- Build distribution charts: `histplot`, `kdeplot`, `boxplot`, `violinplot`
- Build relational charts: `scatterplot`, `lineplot`
- Build categorical summary charts: `barplot`, `countplot`
- Combine seaborn charts with matplotlib customization

---

## Setup

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="whitegrid", palette="muted")
```

> [!tip] Set the Theme Once, at the Top
> `sns.set_theme()` sets global defaults for every chart in the session. Call it once at the top of your notebook. Common styles: `"whitegrid"` (clean with horizontal guides), `"darkgrid"`, `"ticks"` (minimal), `"white"` (no grid).

---

## Realistic Dataset

```python
np.random.seed(42)
n = 300

df = pd.DataFrame({
    "study_hours": np.random.normal(4.5, 1.8, n).clip(0, 10),
    "sleep_hours": np.random.normal(7.0, 1.2, n).clip(4, 10),
    "score":       np.random.normal(68, 14, n).clip(20, 100),
    "attendance":  np.random.uniform(50, 100, n),
    "cohort":      np.random.choice(["Morning", "Evening", "Weekend"], n),
    "passed":      np.random.choice([True, False], n, p=[0.72, 0.28]),
})

# Make score positively correlated with study hours
df["score"] = (df["score"] + df["study_hours"] * 3.5).clip(20, 100)
```

---

## The Most Important Concept: Figure-Level vs. Axes-Level

This is where almost everyone gets confused. Seaborn has two kinds of functions.

**Axes-level functions** (`sns.scatterplot()`, `sns.histplot()`, `sns.boxplot()`, etc.) draw onto a single `ax`. They work exactly like matplotlib — you can pass an `ax=` argument, combine them with other plots, and use all your matplotlib customization.

**Figure-level functions** (`sns.relplot()`, `sns.displot()`, `sns.catplot()`) create their own figure. They return a `FacetGrid` object, not a matplotlib axes. You cannot pass `ax=` to them. They are designed for faceting (splitting data into subplots by a column value).

> [!warning] Never Mix Figure-Level Functions With `plt.subplots()`
> This is the most common seaborn mistake. Calling `sns.relplot()` inside a `plt.subplots()` block creates a separate figure and ignores your axes entirely. If you need to place a chart inside a subplot grid, use the axes-level equivalent: `sns.scatterplot(ax=ax)` instead of `sns.relplot()`.

```python
# WRONG — relplot ignores the ax you created
fig, ax = plt.subplots()
sns.relplot(data=df, x="study_hours", y="score")   # creates its OWN figure

# RIGHT — axes-level function respects your ax
fig, ax = plt.subplots(figsize=(7, 4))
sns.scatterplot(data=df, x="study_hours", y="score", ax=ax)
ax.set_title("Study Hours vs. Score")
plt.tight_layout()
plt.savefig("scatter_oo.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

**Quick reference:**

| Category | Figure-Level | Axes-Level Equivalent |
|---|---|---|
| Relational | `relplot` | `scatterplot`, `lineplot` |
| Distribution | `displot` | `histplot`, `kdeplot`, `ecdfplot` |
| Categorical | `catplot` | `boxplot`, `violinplot`, `barplot`, `countplot`, `stripplot` |

---

## Distribution Charts

### `histplot` — Histogram with Optional KDE

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# Basic histogram
sns.histplot(data=df, x="score", bins=25, ax=axes[0],
             color="#0D9488", edgecolor="white")
axes[0].set_title("Score Distribution")
axes[0].set_xlabel("Exam Score")
axes[0].set_ylabel("Count")

# Histogram with KDE overlay, split by cohort
sns.histplot(data=df, x="score", hue="cohort", kde=True,
             bins=20, alpha=0.45, ax=axes[1])
axes[1].set_title("Score Distribution by Cohort")
axes[1].set_xlabel("Exam Score")
axes[1].set_ylabel("Count")

plt.tight_layout()
plt.savefig("histplot.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!info] KDE — Kernel Density Estimate
> `kde=True` overlays a smoothed curve representing the probability density. It makes the overall shape easier to read than a jagged histogram, especially when comparing multiple groups. The KDE is estimated from the data — it's not assuming a normal distribution.

### `kdeplot` — Smooth Density Only

```python
fig, ax = plt.subplots(figsize=(8, 4))

for cohort, color in zip(["Morning", "Evening", "Weekend"],
                          ["#0D9488", "#F59E0B", "#6366F1"]):
    subset = df[df["cohort"] == cohort]["score"]
    sns.kdeplot(subset, ax=ax, label=cohort, color=color,
                linewidth=2, fill=True, alpha=0.15)

ax.set_title("Score Distributions Are Similar Across Cohorts")
ax.set_xlabel("Exam Score")
ax.set_ylabel("Density")
ax.legend(title="Cohort")

plt.tight_layout()
plt.savefig("kdeplot.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### `boxplot` — Spread and Outliers

```python
fig, ax = plt.subplots(figsize=(8, 4))

sns.boxplot(data=df, x="cohort", y="score", ax=ax,
            palette="muted", order=["Morning", "Evening", "Weekend"])

ax.set_title("Score Distribution by Cohort")
ax.set_xlabel("Cohort")
ax.set_ylabel("Exam Score")

plt.tight_layout()
plt.savefig("boxplot.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

A boxplot shows: median (center line), interquartile range (box), whiskers (1.5 × IQR), and individual outliers as dots.

### `violinplot` — Distribution Shape + Box

A violin plot combines a KDE with a boxplot. It shows the full distribution shape, not just the summary statistics. Use it when the shape matters — a bimodal distribution looks like a normal distribution in a boxplot.

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

sns.boxplot(data=df, x="cohort", y="score", ax=axes[0], palette="muted")
axes[0].set_title("Boxplot (hides shape)")
axes[0].set_xlabel("Cohort")
axes[0].set_ylabel("Score")

sns.violinplot(data=df, x="cohort", y="score", ax=axes[1],
               palette="muted", inner="box", cut=0)
axes[1].set_title("Violin (shows shape)")
axes[1].set_xlabel("Cohort")
axes[1].set_ylabel("Score")

plt.tight_layout()
plt.savefig("violin_vs_box.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Relational Charts

### `scatterplot` — Relationship Between Two Numeric Variables

```python
fig, ax = plt.subplots(figsize=(8, 5))

sns.scatterplot(data=df, x="study_hours", y="score",
                hue="cohort", style="passed",
                alpha=0.65, s=50, ax=ax)

ax.set_title("More Study Hours → Higher Scores")
ax.set_xlabel("Study Hours per Day")
ax.set_ylabel("Exam Score")
ax.legend(title="Cohort / Passed", bbox_to_anchor=(1.02, 1), loc="upper left")

plt.tight_layout()
plt.savefig("scatterplot.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!warning] Overplotting in Scatter Plots
> When you have thousands of points, scatter plots become solid blobs. Solutions: set `alpha=0.2` (transparency), use `sns.kdeplot(kind="hex")` for 2D density, or bin and aggregate before plotting. A scatter plot with 10,000 fully opaque dots tells you nothing.

### `lineplot` — Trend Over Time with Confidence Interval

`sns.lineplot` automatically computes the mean and 95% confidence interval when you have multiple observations per x value.

```python
# Simulate daily signups with multiple readings per day
np.random.seed(0)
days = np.repeat(np.arange(1, 31), 5)
signups = (days * 1.8 + np.random.normal(0, 4, len(days))).clip(0)

time_df = pd.DataFrame({"day": days, "signups": signups,
                         "channel": np.random.choice(["Organic", "Paid"], len(days))})

fig, ax = plt.subplots(figsize=(10, 4))

sns.lineplot(data=time_df, x="day", y="signups",
             hue="channel", ax=ax, linewidth=2)

ax.set_title("Daily Signups by Acquisition Channel (shaded = 95% CI)")
ax.set_xlabel("Day of Month")
ax.set_ylabel("Signups")
ax.legend(title="Channel")

plt.tight_layout()
plt.savefig("lineplot_ci.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Categorical Summary Charts

### `barplot` — Aggregated Mean with Confidence Interval

`sns.barplot` computes the mean by default and draws error bars representing the 95% CI. It is not a count chart — it's an estimation chart.

```python
fig, ax = plt.subplots(figsize=(8, 4))

sns.barplot(data=df, x="cohort", y="score",
            hue="passed", ax=ax,
            order=["Morning", "Evening", "Weekend"])

ax.set_title("Average Score by Cohort and Pass Status")
ax.set_xlabel("Cohort")
ax.set_ylabel("Average Score")
ax.legend(title="Passed")

plt.tight_layout()
plt.savefig("barplot.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### `countplot` — Count of Observations Per Category

```python
fig, ax = plt.subplots(figsize=(7, 4))

sns.countplot(data=df, x="cohort", hue="passed",
              order=["Morning", "Evening", "Weekend"], ax=ax)

ax.set_title("Student Count by Cohort and Pass Status")
ax.set_xlabel("Cohort")
ax.set_ylabel("Number of Students")
ax.legend(title="Passed")

plt.tight_layout()
plt.savefig("countplot.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Figure-Level Functions — Faceting

Use `sns.relplot()`, `sns.displot()`, or `sns.catplot()` when you want to split data into separate subplots by a column. Do not use them when you just want a single chart.

```python
g = sns.relplot(
    data=df,
    x="study_hours", y="score",
    col="cohort",
    hue="passed",
    alpha=0.6, s=40,
    height=4, aspect=1.1
)

g.set_titles("Cohort: {col_name}")
g.set_axis_labels("Study Hours", "Score")
g.figure.suptitle("Score vs. Study Hours by Cohort", y=1.03)

g.figure.savefig("facet_scatter.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## The `hue` Parameter

`hue` is seaborn's most powerful parameter. Pass it a categorical column and seaborn automatically:
- assigns a different color to each category
- adds a legend
- keeps all other chart properties consistent

```python
fig, axes = plt.subplots(1, 3, figsize=(14, 4))

# hue on scatter
sns.scatterplot(data=df, x="attendance", y="score",
                hue="cohort", alpha=0.5, ax=axes[0])
axes[0].set_title("Scatter with hue")

# hue on histogram
sns.histplot(data=df, x="score", hue="cohort",
             bins=20, alpha=0.5, ax=axes[1])
axes[1].set_title("Histogram with hue")

# hue on box
sns.boxplot(data=df, x="cohort", y="score",
            hue="passed", ax=axes[2])
axes[2].set_title("Boxplot with hue")

plt.tight_layout()
plt.savefig("hue_examples.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Combining Seaborn and Matplotlib

Seaborn creates the chart. Matplotlib adjusts the details. This is the standard workflow.

```python
fig, ax = plt.subplots(figsize=(9, 5))

sns.scatterplot(data=df, x="study_hours", y="score",
                hue="cohort", alpha=0.5, s=50, ax=ax)

# Add a regression line manually
m, b = np.polyfit(df["study_hours"], df["score"], 1)
x_line = np.linspace(df["study_hours"].min(), df["study_hours"].max(), 100)
ax.plot(x_line, m * x_line + b, color="black", linewidth=2,
        linestyle="--", label=f"Trend: y={m:.1f}x+{b:.1f}")

# Matplotlib customization
ax.set_title("Study Hours Predict Scores Across All Cohorts", fontsize=13)
ax.set_xlabel("Study Hours per Day", fontsize=11)
ax.set_ylabel("Exam Score", fontsize=11)
ax.legend(title="Cohort", framealpha=0.9)
ax.annotate("Each point = one student", xy=(0.02, 0.95),
            xycoords="axes fraction", fontsize=9, color="gray")

plt.tight_layout()
plt.savefig("seaborn_matplotlib_combo.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Practice Exercises

**Warm-up:** Using the `df` dataset, create a `histplot` of `study_hours` with `kde=True`. Split by `cohort` using `hue`. Give it a title and axis labels.

**Main:** Create a 1×2 subplot. Left: a `violinplot` of `score` by `cohort`. Right: a `barplot` of mean `score` by `cohort` split by `passed` using `hue`. Use `tight_layout()`.

**Stretch:** Create a `pairplot` of the numeric columns `study_hours`, `sleep_hours`, `score`, and `attendance`, colored by `cohort`. Observe which pairs show the strongest visual correlation. Write a one-sentence interpretation for each.

---

## Interview Questions

**Q: What is the difference between figure-level and axes-level seaborn functions?**

??? "Show answer"
    Axes-level functions (e.g., `scatterplot`, `histplot`, `boxplot`) draw onto a single matplotlib `Axes` object. You can pass `ax=ax` to place them in a subplot grid and use matplotlib to customize them. Figure-level functions (e.g., `relplot`, `displot`, `catplot`) create their own figure and return a `FacetGrid`. They are designed for faceting across subplots by a column value. You cannot pass `ax=` to a figure-level function.

**Q: What does `hue` do in seaborn?**

??? "Show answer"
    `hue` maps a categorical variable to color. Seaborn assigns a distinct color to each category, plots each group separately, and automatically generates a legend. It is the simplest way to add a third dimension to a two-axis chart.

**Q: When would you choose a violin plot over a box plot?**

??? "Show answer"
    When the shape of the distribution matters. A box plot summarizes five statistics (min, Q1, median, Q3, max). A bimodal distribution — two separate clusters of values — looks identical to a normal distribution in a box plot. A violin plot shows the full KDE of the distribution, so you can see bimodality, skewness, and gaps that a box plot hides.

**Q: `sns.barplot()` shows confidence interval error bars by default. What does this mean?**

??? "Show answer"
    `sns.barplot()` computes the mean of the y variable for each x category, then uses bootstrapping to estimate the 95% confidence interval around each mean. The error bar shows this interval — a taller error bar means more uncertainty (higher variance or fewer data points). It is different from a simple bar chart, which shows a raw value with no uncertainty.

---

> [!success] Key Takeaways
> - Axes-level functions (`scatterplot`, `histplot`, `boxplot`, etc.) work inside `plt.subplots()` grids. Figure-level functions (`relplot`, `displot`, `catplot`) create their own figure — never mix them with `plt.subplots()`.
> - `hue` is seaborn's most powerful feature for exploratory analysis — use it to reveal group-level patterns.
> - `violinplot` shows distribution shape. `boxplot` shows summary statistics. Use violin when shape matters.
> - Seaborn creates the chart. Matplotlib customizes it. Combine both freely.

---

[[02-line-bar-histogram|Previous: Line, Bar, and Histogram Charts]] | [[04-heatmaps|Next: Heatmaps]]
