# Visualization Best Practices — Clarity, Honesty, and Consistency

A chart that requires explanation has already failed. Bad charts do not just look bad — they obscure insights, mislead stakeholders, and undermine trust in your analysis. This note covers the decisions that separate charts that communicate from charts that confuse: chart selection, color, labeling, data-ink ratio, accessibility, and consistent style across a project.

## Learning Objectives

- Select the right chart type based on the question being answered
- Apply the data-ink ratio principle to remove visual clutter
- Build colorblind-friendly charts using accessible palettes
- Use `rcParams` to set consistent styles across an entire project
- Annotate charts to highlight the specific insight — not the entire dataset
- Avoid the most common chartjunk patterns: 3D, pie charts, dual axes, rainbow colormaps

---

## Start With the Question

Before opening a notebook, write one sentence: *"This chart answers: \_\_\_\_\_\_."*

If you cannot complete that sentence, you are not ready to plot. The chart type follows from the question. The question determines the data transformation. The data transformation determines the axes.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

np.random.seed(42)

# Sample dataset — monthly e-commerce metrics
months = ["Jan","Feb","Mar","Apr","May","Jun",
          "Jul","Aug","Sep","Oct","Nov","Dec"]
revenue      = [82, 91, 104, 118, 127, 143, 151, 148, 139, 162, 175, 198]
orders       = [820, 901, 1040, 1160, 1248, 1410, 1520, 1498, 1370, 1600, 1732, 1960]
return_rate  = [4.2, 3.8, 4.5, 3.1, 3.6, 2.9, 3.2, 3.8, 4.1, 2.7, 3.0, 2.5]

channels = ["Organic Search", "Paid Ads", "Email", "Social", "Direct", "Referral"]
channel_revenue = [340, 218, 175, 96, 142, 89]

np.random.seed(0)
customer_scores = {
    "Loyal":       np.random.normal(82, 10, 200).clip(40, 100),
    "At Risk":     np.random.normal(61, 14, 150).clip(20, 100),
    "New":         np.random.normal(70, 12, 250).clip(20, 100),
}
```

---

## Chart Type Selection

The right chart makes the answer obvious. The wrong chart buries it.

| Question | Right Chart | Wrong Choice |
|---|---|---|
| How did revenue change over 12 months? | Line chart | Bar chart (obscures trend) |
| Which channel drives the most revenue? | Horizontal bar (sorted) | Pie chart |
| How are customer scores distributed? | Histogram or violin | Bar chart of averages |
| Are study hours correlated with scores? | Scatter plot | Line chart |
| How do three groups compare on a metric? | Boxplot or violin | Grouped bar of means only |
| How do values vary across two categories? | Heatmap | 3D bar chart |

> [!warning] Never Use a Pie Chart for More Than 3 Categories
> The human eye cannot accurately compare angles or areas. Pie charts are functionally useless for four or more categories — readers cannot rank slices without reading the labels, which means the visual adds no value over a table. Use a sorted horizontal bar chart instead. It takes the same space and communicates a clear ranking.

```python
# WRONG — pie chart for 6 categories
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].pie(channel_revenue, labels=channels, autopct="%1.0f%%")
axes[0].set_title("Pie Chart — Hard to Rank")

# RIGHT — horizontal bar chart, sorted
sorted_idx = np.argsort(channel_revenue)
sorted_channels = [channels[i] for i in sorted_idx]
sorted_revenue  = [channel_revenue[i] for i in sorted_idx]

axes[1].barh(sorted_channels, sorted_revenue, color="#0D9488", edgecolor="white")
axes[1].set_title("Horizontal Bar — Ranking Is Immediate")
axes[1].set_xlabel("Revenue (₹ thousands)")
axes[1].grid(axis="x", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("pie_vs_bar.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Data-Ink Ratio

Edward Tufte's principle: every drop of ink on a chart should contribute to communicating the data. Ink that does not carry data is chartjunk. Remove it.

Common chartjunk:
- Heavy grid lines
- Colored backgrounds (especially gradient fills)
- 3D effects (always remove — they warp scale)
- Unnecessary borders around plots
- Duplicate legends
- Tick marks on every unit when labels would suffice

```python
months_short = ["J","F","M","A","M","J","J","A","S","O","N","D"]

fig, axes = plt.subplots(1, 2, figsize=(14, 4))

# HIGH chartjunk
axes[0].plot(months, revenue, linewidth=2, color="blue")
axes[0].set_title("Monthly Revenue", fontweight="bold", fontsize=14)
axes[0].set_facecolor("#E8F4FD")
axes[0].grid(True, linewidth=1.5)
axes[0].spines["top"].set_visible(True)
axes[0].spines["right"].set_visible(True)
axes[0].tick_params(axis="both", labelsize=10)
axes[0].set_xlabel("Month", fontsize=12)
axes[0].set_ylabel("Revenue", fontsize=12)

# LOW chartjunk — same data
axes[1].plot(months, revenue, linewidth=2.5, color="#0D9488", marker="o", markersize=4)
axes[1].set_title("Revenue Grew 142% in 12 Months", fontsize=12)
axes[1].spines["top"].set_visible(False)
axes[1].spines["right"].set_visible(False)
axes[1].grid(axis="y", linestyle="--", alpha=0.35)
axes[1].set_xlabel("Month")
axes[1].set_ylabel("Revenue (₹ thousands)")

plt.tight_layout()
plt.savefig("chartjunk_comparison.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Color Accessibility — Colorblind-Friendly Palettes

8% of men and 0.5% of women have some form of color vision deficiency. The most common type (red-green) means your red/green comparison chart is invisible to roughly 1 in 12 male viewers.

Rules for accessible color:
1. Never rely on red vs. green alone to convey meaning
2. Add a secondary encoding: shape, line style, or text label
3. Use palettes designed for colorblind viewers

```python
# Colorblind-friendly palettes in seaborn
print(sns.color_palette("colorblind").as_hex())
# ['#0173b2', '#de8f05', '#029e73', '#d55e00', '#cc78bc', '#ca9161', '#fbafe4', '#949494', '#ece133', '#56b4e9']

# Comparison: default vs. colorblind
fig, axes = plt.subplots(1, 2, figsize=(13, 4))

cohort_means = {k: v.mean() for k, v in customer_scores.items()}
cohort_names = list(cohort_means.keys())
means = list(cohort_means.values())

# Default palette — problematic for red-green colorblind readers
default_colors = ["#e74c3c", "#2ecc71", "#3498db"]
axes[0].bar(cohort_names, means, color=default_colors)
axes[0].set_title("Default Colors (problematic)")
axes[0].set_ylabel("Average Score")
axes[0].set_ylim(0, 100)

# Colorblind-friendly palette
cb_colors = ["#0173b2", "#de8f05", "#029e73"]
axes[1].bar(cohort_names, means, color=cb_colors)
axes[1].set_title("Colorblind-Friendly Palette")
axes[1].set_ylabel("Average Score")
axes[1].set_ylim(0, 100)

plt.tight_layout()
plt.savefig("colorblind_palettes.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!tip] Use `sns.color_palette("colorblind")` as Your Default
> Set it at the top of every analysis: `sns.set_palette("colorblind")`. This costs you nothing and makes your charts accessible to every reader. The palette also looks good — it was designed by researchers to be both accessible and visually distinct.

---

## Always Label Your Axes — No Exceptions

An unlabeled axis is an incomplete sentence. Your reader should never have to guess the unit or scale.

The label must include:
- The variable name (not the column name — a human name)
- The unit in parentheses

```python
fig, axes = plt.subplots(1, 2, figsize=(13, 4))

# WRONG — no units
axes[0].plot(months, revenue)
axes[0].set_title("Monthly Revenue")
axes[0].set_xlabel("month")
axes[0].set_ylabel("revenue")  # what unit? what scale?

# RIGHT — units and human-readable names
axes[1].plot(months, revenue, color="#0D9488", linewidth=2, marker="o", markersize=4)
axes[1].set_title("Monthly Revenue — Consistent Growth Through Year")
axes[1].set_xlabel("Month (2024)")
axes[1].set_ylabel("Revenue (₹ thousands)")
axes[1].spines["top"].set_visible(False)
axes[1].spines["right"].set_visible(False)
axes[1].grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("axis_labels.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Titles That State the Insight

A weak title describes the chart. A strong title states the insight.

| Weak Title | Strong Title |
|---|---|
| "Revenue by Month" | "Revenue Grew 142% Over 12 Months" |
| "Score Distribution" | "Loyal Customers Score 21 Points Higher on Average" |
| "Channel Revenue" | "Organic Search Outperforms All Paid Channels Combined" |

The title is not a label — it is the main finding.

---

## Annotation — Point at What Matters

Annotations let you draw attention to a specific data point without narrating the entire chart. Use them for anomalies, peaks, milestones, and thresholds.

```python
fig, ax = plt.subplots(figsize=(10, 4))

ax.plot(months, revenue, color="#0D9488", linewidth=2.5, marker="o",
        markersize=5, markerfacecolor="white", markeredgewidth=2)

# Annotate the peak month
peak_idx = revenue.index(max(revenue))
ax.annotate(
    f"Peak: ₹{max(revenue)}k",
    xy=(peak_idx, max(revenue)),
    xytext=(peak_idx - 2.5, max(revenue) + 8),
    fontsize=10,
    color="#DC2626",
    arrowprops={"arrowstyle": "->", "color": "#DC2626", "lw": 1.5}
)

# Add a reference line for the annual average
avg = np.mean(revenue)
ax.axhline(avg, linestyle="--", color="gray", linewidth=1, alpha=0.7)
ax.text(11.1, avg + 1, f"Avg: ₹{avg:.0f}k", color="gray", fontsize=9, va="bottom")

ax.set_title("Revenue Peaks in December — Holiday Surge", fontsize=13)
ax.set_xlabel("Month (2024)")
ax.set_ylabel("Revenue (₹ thousands)")
ax.set_xticks(range(len(months)))
ax.set_xticklabels(months)
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)
ax.grid(axis="y", linestyle="--", alpha=0.35)

plt.tight_layout()
plt.savefig("annotated_chart.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Consistent Style Across a Project — `rcParams`

If your notebook has ten charts that all look different, your analysis looks unfinished. Use `rcParams` to set a project-wide style once.

```python
import matplotlib as mpl

# Set once at the top of your project notebook
mpl.rcParams.update({
    "figure.figsize":       (9, 4),
    "figure.dpi":           120,
    "axes.spines.top":      False,
    "axes.spines.right":    False,
    "axes.grid":            True,
    "grid.linestyle":       "--",
    "grid.alpha":           0.4,
    "axes.titlesize":       13,
    "axes.labelsize":       11,
    "xtick.labelsize":      9,
    "ytick.labelsize":      9,
    "legend.framealpha":    0.9,
    "font.family":          "sans-serif",
})

# Every subsequent chart will inherit these defaults
fig, ax = plt.subplots()   # uses (9, 4) figsize and 120 dpi automatically
ax.plot(months, revenue, color="#0D9488", linewidth=2)
ax.set_title("Revenue — rcParams Applied")
ax.set_xlabel("Month")
ax.set_ylabel("Revenue (₹ thousands)")

plt.tight_layout()
plt.savefig("rcparams_demo.png", dpi=150, bbox_inches="tight")  # or plt.show()

# Restore defaults when you are done
# mpl.rcParams.update(mpl.rcParamsDefault)
```

> [!tip] Store Your `rcParams` Block in a Shared Cell
> Put all `rcParams` settings in the first code cell of every project notebook. Name it clearly. This makes the style reproducible and easy to hand off to a teammate.

---

## Before and After — The Full Workflow

```python
# BEFORE: a weak, default chart
import pandas as pd
df_cat = pd.DataFrame({"category": channels, "revenue": channel_revenue})
df_cat.plot(x="category", y="revenue", kind="bar")
plt.show()

# AFTER: a chart that communicates
df_cat = df_cat.sort_values("revenue")

fig, ax = plt.subplots(figsize=(9, 5))

colors = ["#94A3B8"] * len(df_cat)
colors[-1] = "#0D9488"   # highlight the top channel

ax.barh(df_cat["category"], df_cat["revenue"], color=colors, edgecolor="white")

# Add value labels at the end of each bar
for i, (val, name) in enumerate(zip(df_cat["revenue"], df_cat["category"])):
    ax.text(val + 3, i, f"₹{val}k", va="center", fontsize=10)

ax.set_title("Organic Search Drives 33% of Total Revenue", fontsize=13)
ax.set_xlabel("Revenue (₹ thousands)")
ax.set_xlim(0, 400)
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)
ax.grid(axis="x", linestyle="--", alpha=0.35)

plt.tight_layout()
plt.savefig("before_after_chart.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## The Pre-Publication Checklist

Before sharing any chart, run through this list:

- [ ] The title states the insight, not just the topic
- [ ] Both axes are labeled with variable name and unit
- [ ] The y-axis of a bar chart starts at zero
- [ ] Categories in bar charts are sorted by value
- [ ] Colors are meaningful, not decorative
- [ ] The palette is colorblind-friendly
- [ ] No 3D effects
- [ ] No pie chart with more than 3 slices
- [ ] `tight_layout()` has been called — no overlapping elements
- [ ] The chart can be understood without reading surrounding text
- [ ] One written sentence summarizes the main finding

---

## Practice Exercises

**Warm-up:** Take the channel revenue bar chart above and add value labels at the end of each bar using `ax.text()`. The label format should be `"₹{value}k"`.

**Main:** Using the `customer_scores` dictionary, plot overlapping KDE plots for all three groups using `sns.kdeplot()`. Apply the colorblind palette. Add vertical dashed lines for each group's mean. Write a title that states which group scores highest.

**Stretch:** Create a before/after comparison using `plt.subplots(1, 2)`. Left: a default `df.plot()` bar chart of channel revenue. Right: the polished version — sorted, annotated, no top/right spines, meaningful title. Save at 200 DPI.

---

## Interview Questions

**Q: What is the data-ink ratio and how do you improve it?**

??? "Show answer"
    The data-ink ratio is the proportion of a chart's ink that directly represents data vs. decorative or redundant elements. A higher ratio means more signal per unit of visual space. To improve it: remove grid lines or lighten them, remove the top and right spines (`ax.spines["top"].set_visible(False)`), use lighter tick marks, remove background fills, and eliminate anything the reader's eye crosses that does not convey information.

**Q: Why should bar charts always start at zero?**

??? "Show answer"
    The length of a bar encodes the value. If the axis starts at, say, 90, a bar at 100 looks twice as tall as a bar at 95 — a 100% visual difference for a 5% actual difference. Starting at zero maintains the proportional relationship between visual length and actual value. Line charts can use non-zero baselines because the slope (change) carries meaning, not the absolute length from the axis.

**Q: A colleague used red for "good" and green for "bad" in a dashboard. What is the problem?**

??? "Show answer"
    Red-green color blindness (deuteranopia) affects about 8% of men. To a red-green colorblind reader, those two colors may be indistinguishable. Additionally, the convention is the opposite — red typically means danger/bad, green means go/good. Using them in reverse adds cognitive overhead. Fix: use a colorblind-safe palette (e.g., blue and orange), add a secondary encoding (up arrow / down arrow, positive/negative text), and keep conventional color meanings.

**Q: What is the difference between a chart title and an axis label?**

??? "Show answer"
    An axis label identifies the variable and its unit — it is a reference: "Revenue (₹ thousands)". A chart title communicates the main finding — it is a statement: "Revenue Grew 142% Over 12 Months". The title should tell the reader what to notice; the axis labels give them the context to verify it. Titles that only describe the chart type ("Bar Chart of Revenue by Month") waste the most prominent text position on the page.

---

> [!success] Key Takeaways
> - Choose the chart type based on the question, not the data shape.
> - The chart title is the answer. The axis labels are the context.
> - Never use a pie chart for more than 3 categories. Sort bar charts.
> - Use colorblind-safe palettes (`sns.color_palette("colorblind")`) by default.
> - Remove spines, lighten grids, eliminate chartjunk — every non-data element you remove makes the data clearer.
> - Set `rcParams` once to enforce consistent style across your entire project.

---

[[04-heatmaps|Previous: Heatmaps]] | [[06-mini-project|Next: Mini Project — Sales Dashboard]]
