# Line, Bar, and Histogram Charts — Choosing the Right Tool

The most common mistake in data visualization is reaching for the wrong chart type. A bar chart where a line chart should be. A histogram when you need a bar chart. A pie chart for six categories. Choosing wrong misleads your audience before you've said a word. This note gives you the framework for picking the right chart — and the code to build it well.

## Learning Objectives

- Identify which chart type fits which kind of question
- Build polished line charts with markers, colors, and twin axes
- Build horizontal and vertical bar charts with error bars and sorted categories
- Build histograms with appropriate bin counts, density normalization, and overlapping distributions
- Avoid the most common mistakes: wrong baseline, unlabeled axes, hidden overplotting

---

## Sample Dataset

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(42)

# Monthly store performance
months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun",
          "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]
revenue   = [82, 91, 104, 118, 127, 143, 151, 148, 139, 162, 175, 198]
units_sold = [410, 455, 520, 590, 635, 715, 755, 740, 695, 810, 875, 990]
return_rate = [4.2, 3.8, 4.5, 3.1, 3.6, 2.9, 3.2, 3.8, 4.1, 2.7, 3.0, 2.5]

# Department revenue with std dev for error bars
departments = ["Electronics", "Apparel", "Home & Garden", "Sports", "Books", "Toys"]
dept_revenue = [342, 218, 175, 196, 89, 143]
dept_std     = [28, 19, 22, 17, 11, 16]

# Score distributions for two student cohorts
cohort_a = np.random.normal(loc=68, scale=12, size=200)
cohort_b = np.random.normal(loc=74, scale=9,  size=200)
cohort_a = np.clip(cohort_a, 0, 100)
cohort_b = np.clip(cohort_b, 0, 100)
```

---

## Line Charts — Show Change Over Time

Use a line chart when the x-axis is ordered (time, sequence, steps) and the relationship between adjacent points matters. Connecting the dots implies continuity — only do it when that implication is true.

> [!info] When a Line Chart Is Wrong
> If your x-axis is categories with no natural order (city names, product types), do not use a line chart. The line implies a trend between adjacent bars that does not exist. Use a bar chart instead.

```python
fig, ax = plt.subplots(figsize=(10, 4))

ax.plot(months, revenue,
        color="#0D9488", linewidth=2.5, marker="o",
        markersize=6, markerfacecolor="white", markeredgewidth=2,
        label="Revenue (thousands)")

# Shade the area under the line
ax.fill_between(range(len(months)), revenue, alpha=0.08, color="#0D9488")

ax.set_title("Revenue Grew 142% Over the Year", fontsize=13, pad=12)
ax.set_xlabel("Month")
ax.set_ylabel("Revenue (₹ thousands)")
ax.set_xticks(range(len(months)))
ax.set_xticklabels(months)
ax.legend()
ax.grid(axis="y", linestyle="--", alpha=0.4)
ax.set_ylim(0, 220)

plt.tight_layout()
plt.savefig("revenue_trend.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### Multiple Lines

```python
fig, ax = plt.subplots(figsize=(10, 4))

ax.plot(months, revenue, color="#0D9488", linewidth=2.5, marker="o",
        markersize=5, label="Revenue (₹ thousands)")
ax.plot(months, [u / 5 for u in units_sold], color="#F59E0B", linewidth=2,
        linestyle="--", marker="s", markersize=5, label="Units Sold (÷5)")

ax.set_title("Revenue vs. Units Sold — Scaling Tracks Well")
ax.set_xlabel("Month")
ax.set_ylabel("Value")
ax.set_xticks(range(len(months)))
ax.set_xticklabels(months)
ax.legend(framealpha=0.9)
ax.grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("revenue_vs_units.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### Twin Axes — Two Different Scales

When two metrics have very different units (revenue in thousands vs. return rate as a percentage), a twin axis lets you overlay them without distortion.

```python
fig, ax1 = plt.subplots(figsize=(10, 4))

color_rev  = "#0D9488"
color_rate = "#DC2626"

ax1.plot(months, revenue, color=color_rev, linewidth=2.5, marker="o",
         markersize=5, label="Revenue")
ax1.set_xlabel("Month")
ax1.set_ylabel("Revenue (₹ thousands)", color=color_rev)
ax1.tick_params(axis="y", labelcolor=color_rev)
ax1.set_xticks(range(len(months)))
ax1.set_xticklabels(months)

ax2 = ax1.twinx()   # share the x-axis, new y-axis on the right
ax2.plot(months, return_rate, color=color_rate, linewidth=2, linestyle="--",
         marker="^", markersize=5, label="Return Rate")
ax2.set_ylabel("Return Rate (%)", color=color_rate)
ax2.tick_params(axis="y", labelcolor=color_rate)

ax1.set_title("Revenue Rising as Return Rate Falls")

# Combine legends from both axes
lines1, labels1 = ax1.get_legend_handles_labels()
lines2, labels2 = ax2.get_legend_handles_labels()
ax1.legend(lines1 + lines2, labels1 + labels2, loc="upper left")

plt.tight_layout()
plt.savefig("twin_axes.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!warning] Twin Axes Can Mislead
> A dual-axis chart lets you align any two datasets to look correlated. Your audience may infer a relationship that does not exist. Use twin axes only when the two metrics genuinely interact and label each axis with its unit clearly.

---

## Bar Charts — Compare Categories

Use a bar chart to compare discrete, unordered categories. The length of each bar is what carries meaning — so the baseline must always start at zero. A bar chart with a non-zero baseline is a lie.

### Vertical Bar Chart

```python
fig, ax = plt.subplots(figsize=(9, 4))

x = range(len(departments))
bars = ax.bar(x, dept_revenue,
              color="#2563EB", edgecolor="white", linewidth=0.8,
              yerr=dept_std, capsize=4, error_kw={"ecolor": "#374151", "elinewidth": 1.5})

ax.set_xticks(x)
ax.set_xticklabels(departments, rotation=15, ha="right")
ax.set_title("Electronics Leads Revenue by a Wide Margin")
ax.set_xlabel("Department")
ax.set_ylabel("Revenue (₹ thousands)")
ax.set_ylim(0, 400)
ax.grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("dept_revenue_bar.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### Horizontal Bar Chart — Preferred for Long Labels

When category names are long, flip the chart. Horizontal bars are also better for ranking — the eye reads top-to-bottom naturally.

```python
# Sort ascending so the largest bar appears at the top
sorted_idx = sorted(range(len(dept_revenue)), key=lambda i: dept_revenue[i])
sorted_depts   = [departments[i] for i in sorted_idx]
sorted_revenue = [dept_revenue[i] for i in sorted_idx]
sorted_std     = [dept_std[i] for i in sorted_idx]

fig, ax = plt.subplots(figsize=(8, 5))

ax.barh(sorted_depts, sorted_revenue,
        color="#0D9488", edgecolor="white",
        xerr=sorted_std, capsize=4,
        error_kw={"ecolor": "#374151", "elinewidth": 1.5})

ax.set_title("Department Revenue Ranking")
ax.set_xlabel("Revenue (₹ thousands)")
ax.set_xlim(0, 400)
ax.grid(axis="x", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("dept_revenue_hbar.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!tip] Sort Your Bar Charts
> An unsorted bar chart forces the reader to scan every bar to find the largest. Sort descending (or ascending for horizontal). The ranking is usually the insight.

### Grouped Bar Chart

```python
categories = ["Q1", "Q2", "Q3", "Q4"]
product_a  = [42, 58, 51, 67]
product_b  = [35, 49, 62, 71]

x = np.arange(len(categories))
width = 0.35

fig, ax = plt.subplots(figsize=(8, 4))

ax.bar(x - width/2, product_a, width, label="Product A", color="#2563EB")
ax.bar(x + width/2, product_b, width, label="Product B", color="#F59E0B")

ax.set_title("Product B Closed the Gap in H2")
ax.set_xlabel("Quarter")
ax.set_ylabel("Units Sold")
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend()
ax.set_ylim(0, 85)
ax.grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("grouped_bar.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Histograms — Understand Distributions

A histogram is not a bar chart. A bar chart shows one value per category. A histogram groups a continuous variable into bins and counts how many data points fall in each bin. The shape of the histogram tells you about the distribution — whether it's symmetric, skewed, bimodal, or has outliers.

> [!info] Choosing Bin Count
> Too few bins and you lose shape. Too many bins and noise looks like structure. There is no universal rule. Try 10–30 bins for most datasets and adjust. Alternatively, use `bins="auto"` and let NumPy pick.

```python
fig, ax = plt.subplots(figsize=(8, 4))

ax.hist(cohort_a, bins=20, color="#2563EB", alpha=0.6, edgecolor="white",
        label="Cohort A")

ax.set_title("Distribution of Exam Scores — Cohort A")
ax.set_xlabel("Score")
ax.set_ylabel("Number of Students")
ax.legend()
ax.grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("histogram_basic.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### Overlapping Distributions

Use `alpha` (transparency) to compare two distributions in the same histogram.

```python
fig, ax = plt.subplots(figsize=(9, 4))

ax.hist(cohort_a, bins=25, color="#2563EB", alpha=0.55, edgecolor="white",
        label=f"Cohort A  (mean={cohort_a.mean():.1f})")
ax.hist(cohort_b, bins=25, color="#F59E0B", alpha=0.55, edgecolor="white",
        label=f"Cohort B  (mean={cohort_b.mean():.1f})")

ax.axvline(cohort_a.mean(), color="#1D4ED8", linestyle="--", linewidth=1.5)
ax.axvline(cohort_b.mean(), color="#D97706", linestyle="--", linewidth=1.5)

ax.set_title("Cohort B Scores Higher on Average")
ax.set_xlabel("Exam Score")
ax.set_ylabel("Number of Students")
ax.legend()
ax.grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("histogram_overlap.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### Density Histogram (Normalized)

When two groups have different sample sizes, raw counts are unfair. Normalize with `density=True` to compare shapes.

```python
fig, ax = plt.subplots(figsize=(9, 4))

ax.hist(cohort_a, bins=25, density=True, color="#2563EB", alpha=0.55,
        edgecolor="white", label="Cohort A (density)")
ax.hist(cohort_b, bins=25, density=True, color="#F59E0B", alpha=0.55,
        edgecolor="white", label="Cohort B (density)")

ax.set_title("Score Distributions Normalized for Comparison")
ax.set_xlabel("Exam Score")
ax.set_ylabel("Probability Density")
ax.legend()
ax.grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("histogram_density.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

### Cumulative Histogram

A cumulative histogram answers "what fraction of students scored below X?"

```python
fig, ax = plt.subplots(figsize=(9, 4))

ax.hist(cohort_a, bins=30, cumulative=True, density=True,
        color="#2563EB", alpha=0.7, edgecolor="white", label="Cohort A")
ax.hist(cohort_b, bins=30, cumulative=True, density=True,
        color="#F59E0B", alpha=0.7, edgecolor="white", label="Cohort B")

ax.axhline(0.5, color="gray", linestyle=":", linewidth=1.2, label="50th percentile")

ax.set_title("Cumulative Score Distribution")
ax.set_xlabel("Score")
ax.set_ylabel("Fraction of Students")
ax.legend()
ax.grid(linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("histogram_cumulative.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Chart Selection Reference

| Question | Best Chart |
|---|---|
| How did revenue change each month? | Line chart |
| Which department generates the most revenue? | Horizontal bar chart (sorted) |
| How are exam scores distributed? | Histogram |
| Do two groups have different distributions? | Overlapping histogram or violin plot |
| How did two products compare each quarter? | Grouped bar chart |
| How does a metric with one unit compare to a metric with a different unit over time? | Line chart with twin axes |

---

## Practice Exercises

**Warm-up:** Using the `months` and `revenue` arrays above, build a line chart. Add a horizontal reference line at the annual average revenue using `ax.axhline()`. Label it "Annual Average" in the legend.

**Main:** Build a horizontal bar chart of department revenue sorted from lowest to highest. Add value labels at the end of each bar using `ax.text()`. Set a meaningful title that names the winning department.

**Stretch:** Simulate scores for three student cohorts (n=150 each, with means 65, 72, and 79). Plot all three as overlapping density histograms. Add vertical dashed lines for each mean. Add a legend that includes the mean value in the label.

---

## Interview Questions

**Q: Why must a bar chart's y-axis start at zero?**

??? "Show answer"
    The length of a bar encodes its value. If the baseline is not zero, the visual length of the bar no longer represents the actual value — a bar showing 102 looks dramatically longer than one showing 100, even though the difference is 2%. Starting at zero maintains the proportional relationship between bar lengths and the values they represent.

**Q: When should you use a histogram instead of a bar chart?**

??? "Show answer"
    Use a histogram when your variable is continuous and you want to understand its distribution — the shape, center, spread, and presence of outliers. Bar charts compare distinct, named categories. Histograms show frequency across binned ranges of a numeric variable. The key difference: bar chart bars have gaps between them (categories are discrete), histogram bars touch (the range is continuous).

**Q: What does `density=True` do in `ax.hist()`?**

??? "Show answer"
    It normalizes the histogram so that the area of all bars sums to 1, turning counts into probability densities. This is useful when comparing two groups with different sample sizes — raw counts would favor the larger group visually even if the distributions have the same shape.

**Q: What is the risk of using twin axes in a line chart?**

??? "Show answer"
    Twin axes let you scale two independent y-axes independently, which means you can make any two time series appear correlated by choosing convenient scales. The visual alignment suggests a relationship that may not be meaningful. Use twin axes only when the two metrics genuinely interact, and always label both axes with units so the reader can verify the scale.

---

> [!success] Key Takeaways
> - Line charts are for ordered, continuous x-axes (time, sequence). Never connect unordered categories with a line.
> - Bar charts must start at zero. Always sort the categories by value.
> - Histograms show distributions, not comparisons. Adjust bins until the shape is clear.
> - Use `density=True` when comparing groups of different sizes.
> - Twin axes are powerful but easy to misuse — label units clearly.

---

[[01-matplotlib-basics|Previous: Matplotlib Basics]] | [[03-seaborn-basics|Next: Seaborn Basics]]
