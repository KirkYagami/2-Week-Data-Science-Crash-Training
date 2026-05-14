# Matplotlib & Seaborn Cheat Sheet

Matplotlib is the foundation of Python visualization — fine-grained control, every chart element is configurable. Seaborn is built on Matplotlib and adds statistical chart types with sensible defaults for data exploration. Use Matplotlib when you need precise control over layout or output; use Seaborn when you want to communicate data distributions and relationships quickly.

---

## 1. Figure & Axes Setup

### plt.figure() — bare canvas

Use when you need a single plot and want to configure size or resolution before drawing.

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

fig = plt.figure(figsize=(8, 4), dpi=100)   # width x height in inches
plt.plot(x, np.sin(x))
plt.savefig("sine.png", bbox_inches="tight")  # saves to file
```

### plt.subplots() — fig/ax pattern

The preferred pattern in production code. Returns a figure and one or more axes objects, giving you an explicit handle on every element.

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

fig, ax = plt.subplots(figsize=(8, 4), dpi=100)
ax.plot(x, np.cos(x), color="steelblue")
ax.set_title("Cosine Wave")
plt.savefig("cosine.png", bbox_inches="tight")  # saves to file
```

### Multiple axes in one figure

Unpack the axes array when you need side-by-side panels. Each ax is independent.

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

fig, (ax1, ax2) = plt.subplots(nrows=1, ncols=2, figsize=(12, 4))
ax1.plot(x, np.sin(x), color="steelblue")
ax1.set_title("Sine")
ax2.plot(x, np.cos(x), color="coral")
ax2.set_title("Cosine")
plt.tight_layout()
plt.savefig("sine_cosine.png", bbox_inches="tight")  # saves to file
```

---

## 2. Basic Plots

### Line plot — plt.plot()

Line plots show continuous data over a sequence. Use for time series, functions, or any ordered numerical relationship.

```python
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(42)
days = np.arange(1, 31)
revenue = 200 + np.cumsum(rng.normal(10, 30, 30))

fig, ax = plt.subplots(figsize=(9, 4))
ax.plot(days, revenue, color="steelblue", linewidth=2, marker="o", markersize=4)
ax.set_title("Daily Revenue (June)")
ax.set_xlabel("Day")
ax.set_ylabel("Revenue ($)")
plt.savefig("revenue_line.png", bbox_inches="tight")  # saves to file
```

### Scatter plot — plt.scatter()

Use when you want to show the relationship between two continuous variables. Supports per-point color and size encoding.

```python
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(0)
age = rng.integers(22, 60, 80)
salary = 30000 + age * 900 + rng.normal(0, 5000, 80)
experience = rng.integers(1, 15, 80)

fig, ax = plt.subplots(figsize=(8, 5))
scatter = ax.scatter(age, salary, c=experience, cmap="viridis",
                     s=60, alpha=0.7, edgecolors="white", linewidths=0.5)
fig.colorbar(scatter, ax=ax, label="Years of Experience")
ax.set_xlabel("Age")
ax.set_ylabel("Salary ($)")
ax.set_title("Age vs Salary, colored by Experience")
plt.savefig("scatter_salary.png", bbox_inches="tight")  # saves to file
```

### Bar chart — plt.bar() and plt.barh()

Vertical bars for comparing a small number of discrete categories. Horizontal bars (barh) work better when category labels are long.

```python
import matplotlib.pyplot as plt
import numpy as np

categories = ["Python", "SQL", "R", "Scala", "Julia"]
counts = [87, 72, 45, 28, 19]
colors = ["#0D9488" if c == max(counts) else "#94A3B8" for c in counts]

fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# vertical
axes[0].bar(categories, counts, color=colors)
axes[0].set_title("Survey: Primary Language (vertical)")
axes[0].set_ylabel("Respondents")

# horizontal — easier to read long labels
axes[1].barh(categories, counts, color=colors)
axes[1].set_title("Survey: Primary Language (horizontal)")
axes[1].set_xlabel("Respondents")

plt.tight_layout()
plt.savefig("bar_charts.png", bbox_inches="tight")  # saves to file
```

### Histogram — plt.hist()

Shows the distribution of a single continuous variable. Bin count significantly affects interpretation — always try several values.

```python
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(7)
scores = np.concatenate([rng.normal(65, 10, 300), rng.normal(85, 5, 100)])

fig, ax = plt.subplots(figsize=(8, 4))
ax.hist(scores, bins=30, color="steelblue", edgecolor="white", alpha=0.85)
ax.axvline(scores.mean(), color="coral", linewidth=2, linestyle="--", label=f"Mean: {scores.mean():.1f}")
ax.set_xlabel("Score")
ax.set_ylabel("Frequency")
ax.set_title("Exam Score Distribution")
ax.legend()
plt.savefig("histogram.png", bbox_inches="tight")  # saves to file
```

---

## 3. Labels & Titles

### Title, axis labels, and legend

These are non-negotiable on any chart destined for a presentation or report. Use `ax.*` methods — they give consistent behaviour on multi-panel figures.

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 4 * np.pi, 200)

fig, ax = plt.subplots(figsize=(9, 4))
ax.plot(x, np.sin(x), label="sin(x)", color="steelblue")
ax.plot(x, np.cos(x), label="cos(x)", color="coral", linestyle="--")

ax.set_title("Sine and Cosine", fontsize=14, fontweight="bold", pad=12)
ax.set_xlabel("x (radians)", fontsize=11)
ax.set_ylabel("Amplitude", fontsize=11)
ax.legend(fontsize=10, framealpha=0.5)
plt.savefig("labels_demo.png", bbox_inches="tight")  # saves to file
```

### Custom tick positions and labels

Override automatic ticks when default values are ambiguous or when you need meaningful category labels on a numeric axis.

```python
import matplotlib.pyplot as plt
import numpy as np

months = np.arange(1, 13)
sales = [42, 38, 55, 61, 70, 83, 91, 88, 74, 60, 49, 95]
month_labels = ["Jan", "Feb", "Mar", "Apr", "May", "Jun",
                "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]

fig, ax = plt.subplots(figsize=(10, 4))
ax.bar(months, sales, color="steelblue")
ax.set_xticks(months)
ax.set_xticklabels(month_labels, rotation=0, fontsize=9)
ax.set_yticks(range(0, 110, 10))
ax.set_title("Monthly Sales")
plt.savefig("custom_ticks.png", bbox_inches="tight")  # saves to file
```

---

## 4. Styling

### Colors, linestyles, markers, and alpha

These parameters go directly into `plot()`, `scatter()`, etc. Learn them by name — they make code self-documenting.

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 60)

fig, ax = plt.subplots(figsize=(9, 5))
ax.plot(x, np.sin(x),        color="#0D9488", linestyle="-",  linewidth=2, marker="o", markersize=4, label="solid + circle")
ax.plot(x, np.sin(x) + 0.5,  color="coral",   linestyle="--", linewidth=2, marker="s", markersize=5, label="dashed + square")
ax.plot(x, np.sin(x) + 1.0,  color="orchid",  linestyle=":",  linewidth=2, marker="^", markersize=5, alpha=0.7, label="dotted + triangle, alpha=0.7")
ax.legend()
ax.set_title("Linestyle, Marker, and Alpha Options")
plt.savefig("styles_demo.png", bbox_inches="tight")  # saves to file
```

### plt.style.use() — preset stylesheets

Switch the entire look with one line. Use before creating the figure. `seaborn-v0_8-whitegrid` is clean for reports; `ggplot` mimics R aesthetics.

```python
import matplotlib.pyplot as plt
import numpy as np

# Available: 'ggplot', 'seaborn-v0_8-whitegrid', 'bmh', 'dark_background', 'fivethirtyeight'
plt.style.use("seaborn-v0_8-whitegrid")

rng = np.random.default_rng(1)
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(np.cumsum(rng.normal(0, 1, 200)), linewidth=1.5, color="steelblue")
ax.set_title("Random Walk — seaborn-v0_8-whitegrid style")
plt.savefig("style_demo.png", bbox_inches="tight")  # saves to file

plt.style.use("default")   # reset to Matplotlib default afterward
```

### Grid customization

A subtle grid helps readers trace values. Never let the grid compete with the data.

```python
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(5)
x = np.sort(rng.uniform(0, 10, 50))
y = 2 * x + rng.normal(0, 3, 50)

fig, ax = plt.subplots(figsize=(8, 4))
ax.scatter(x, y, color="steelblue", alpha=0.7, zorder=3)
ax.grid(True, which="major", linestyle="--", linewidth=0.6, color="#CCCCCC", alpha=0.8)
ax.set_axisbelow(True)   # grid stays behind data points
ax.set_title("Grid Behind Data (ax.set_axisbelow)")
plt.savefig("grid_demo.png", bbox_inches="tight")  # saves to file
```

---

## 5. Subplots

### Grid of subplots — plt.subplots(nrows, ncols)

Use a 2-D array of axes when you need a grid. Access individual panels with `axes[row, col]`.

```python
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(99)
fig, axes = plt.subplots(2, 2, figsize=(10, 7))

titles = ["Histogram", "Line", "Scatter", "Bar"]
for i, ax in enumerate(axes.flat):
    data = rng.normal(0, 1, 200)
    if i == 0:
        ax.hist(data, bins=20, color="steelblue")
    elif i == 1:
        ax.plot(data[:50], color="coral")
    elif i == 2:
        ax.scatter(data[:80], rng.normal(0, 1, 80), alpha=0.5, color="orchid")
    else:
        ax.bar(range(6), rng.integers(5, 20, 6), color="#0D9488")
    ax.set_title(titles[i], fontsize=10)

plt.tight_layout()
plt.savefig("subplot_grid.png", bbox_inches="tight")  # saves to file
```

### Shared axes

Share x or y axes across panels so zooming one panel or reading comparisons across panels is consistent.

```python
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(10)
dates = np.arange(100)
series_a = np.cumsum(rng.normal(0, 1, 100))
series_b = np.cumsum(rng.normal(0, 2, 100))

fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(9, 6), sharex=True)
ax1.plot(dates, series_a, color="steelblue")
ax1.set_ylabel("Series A")
ax2.plot(dates, series_b, color="coral")
ax2.set_ylabel("Series B")
ax2.set_xlabel("Time Step")
fig.suptitle("Shared X Axis — same time window", fontsize=13)
plt.tight_layout()
plt.savefig("shared_axes.png", bbox_inches="tight")  # saves to file
```

---

## 6. Annotations

### ax.annotate() — arrow + label

Use to call out a specific data point — a peak, an anomaly, a business event.

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 200)
y = np.sin(x) * np.exp(-0.15 * x)

peak_idx = np.argmax(y)

fig, ax = plt.subplots(figsize=(9, 4))
ax.plot(x, y, color="steelblue", linewidth=2)
ax.annotate(
    f"Peak: ({x[peak_idx]:.2f}, {y[peak_idx]:.2f})",
    xy=(x[peak_idx], y[peak_idx]),
    xytext=(x[peak_idx] + 1.5, y[peak_idx] + 0.1),
    arrowprops=dict(arrowstyle="->", color="coral", lw=1.5),
    fontsize=10, color="coral"
)
ax.set_title("Damped Sine with Peak Annotation")
plt.savefig("annotate.png", bbox_inches="tight")  # saves to file
```

### ax.axhline() and ax.axvline() — reference lines

Horizontal and vertical reference lines are the fastest way to communicate a threshold or event boundary.

```python
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(3)
values = rng.normal(50, 12, 150)

fig, ax = plt.subplots(figsize=(9, 4))
ax.plot(values, color="#94A3B8", linewidth=1, alpha=0.8)
ax.axhline(y=50, color="steelblue", linestyle="--", linewidth=1.5, label="Mean baseline")
ax.axhline(y=74, color="coral",     linestyle="--", linewidth=1.5, label="Alert threshold (74)")
ax.axvline(x=100, color="#F59E0B",  linestyle=":",  linewidth=1.5, label="Event at t=100")
ax.legend(fontsize=9)
ax.set_title("Reference Lines")
plt.savefig("reference_lines.png", bbox_inches="tight")  # saves to file
```

### ax.text() — freeform label

Place text at an arbitrary data coordinate. Useful for labelling bar tops or adding data-driven notes.

```python
import matplotlib.pyplot as plt
import numpy as np

categories = ["Q1", "Q2", "Q3", "Q4"]
values = [42, 58, 51, 73]

fig, ax = plt.subplots(figsize=(7, 4))
bars = ax.bar(categories, values, color="steelblue")
for bar, val in zip(bars, values):
    ax.text(
        bar.get_x() + bar.get_width() / 2,  # center of bar
        bar.get_height() + 0.8,              # just above top
        str(val),
        ha="center", va="bottom", fontsize=10, fontweight="bold"
    )
ax.set_ylim(0, 85)
ax.set_title("Bar Labels with ax.text()")
plt.savefig("bar_labels.png", bbox_inches="tight")  # saves to file
```

---

## 7. Saving

### plt.savefig() — raster output

PNG at 150–300 dpi covers most use cases. Use `bbox_inches="tight"` to avoid clipping labels.

```python
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(np.sin(np.linspace(0, 10, 300)), color="steelblue")
ax.set_title("Saving as PNG")

plt.savefig("output.png", dpi=150, bbox_inches="tight")  # saves to file
# dpi=300 for print; dpi=96 for web-only
```

### plt.savefig() — vector output (SVG/PDF)

SVG and PDF are resolution-independent. Use for publications, slides, or dashboards that need to scale.

```python
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(6, 4))
x = np.linspace(0, 10, 200)
ax.plot(x, np.sin(x), color="#0D9488", linewidth=2)
ax.set_title("Vector Export")

plt.savefig("output.svg", bbox_inches="tight")   # saves to file — SVG for web/slides
plt.savefig("output.pdf", bbox_inches="tight")   # saves to file — PDF for LaTeX / print
```

---

## 8. Distribution Plots (Seaborn)

### sns.histplot() — histogram with optional KDE overlay

The Seaborn version of a histogram adds a KDE curve in one parameter and handles `hue` grouping automatically.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(42)
data = {
    "score": np.concatenate([rng.normal(65, 10, 300), rng.normal(85, 7, 200)]),
    "group": ["A"] * 300 + ["B"] * 200
}

fig, ax = plt.subplots(figsize=(9, 4))
sns.histplot(data=data, x="score", hue="group", bins=30, kde=True,
             alpha=0.5, palette=["steelblue", "coral"], ax=ax)
ax.set_title("Score Distribution by Group")
plt.savefig("sns_histplot.png", bbox_inches="tight")  # saves to file
```

### sns.kdeplot() — density curves

KDE plots smooth out the histogram. Use when you have many observations and want to compare distribution shapes cleanly.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

rng = np.random.default_rng(11)
fig, ax = plt.subplots(figsize=(8, 4))

for label, loc, scale in [("Low risk", 40, 8), ("Med risk", 60, 12), ("High risk", 80, 15)]:
    vals = rng.normal(loc, scale, 500)
    sns.kdeplot(vals, fill=True, alpha=0.35, label=label, ax=ax)

ax.set_xlabel("Predicted Score")
ax.set_title("KDE: Risk Tier Score Distributions")
ax.legend()
plt.savefig("sns_kdeplot.png", bbox_inches="tight")  # saves to file
```

### sns.boxplot() and sns.violinplot()

Boxplot shows median, IQR, and outliers at a glance. Violinplot adds the full density shape — more informative when you have enough data (n > 50 per group).

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(7)
df = pd.DataFrame({
    "department": np.repeat(["Engineering", "Sales", "Marketing", "Support"], 120),
    "salary": np.concatenate([
        rng.normal(95000, 18000, 120),
        rng.normal(72000, 22000, 120),
        rng.normal(80000, 15000, 120),
        rng.normal(60000, 12000, 120),
    ])
})

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(13, 5))
sns.boxplot(data=df, x="department", y="salary", palette="Set2", ax=ax1)
ax1.set_title("Box Plot — Salary by Department")
sns.violinplot(data=df, x="department", y="salary", palette="Set2", inner="quartile", ax=ax2)
ax2.set_title("Violin Plot — same data")
plt.tight_layout()
plt.savefig("sns_box_violin.png", bbox_inches="tight")  # saves to file
```

---

## 9. Categorical Plots (Seaborn)

### sns.barplot() — mean with confidence intervals

Seaborn's barplot shows the mean and bootstrapped 95% CI automatically. Use it when the uncertainty around the mean matters.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(20)
df = pd.DataFrame({
    "channel": np.repeat(["Email", "Social", "Paid Search", "Organic"], 80),
    "conversion_rate": np.concatenate([
        rng.beta(3, 30, 80),
        rng.beta(2, 40, 80),
        rng.beta(5, 25, 80),
        rng.beta(4, 20, 80),
    ])
})

fig, ax = plt.subplots(figsize=(8, 4))
sns.barplot(data=df, x="channel", y="conversion_rate", palette="muted",
            capsize=0.1, ax=ax)
ax.set_ylabel("Conversion Rate")
ax.set_title("Mean Conversion Rate by Channel (95% CI)")
plt.savefig("sns_barplot.png", bbox_inches="tight")  # saves to file
```

### sns.countplot() — frequency of categories

Use countplot when you have a raw categorical column and want frequency counts without manually calling `.value_counts()`.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(33)
df = pd.DataFrame({
    "plan": rng.choice(["Free", "Starter", "Pro", "Enterprise"], 500, p=[0.5, 0.25, 0.15, 0.10])
})

fig, ax = plt.subplots(figsize=(7, 4))
sns.countplot(data=df, x="plan",
              order=["Free", "Starter", "Pro", "Enterprise"],
              palette="Blues_d", ax=ax)
ax.set_title("User Count by Plan")
ax.set_ylabel("Count")
plt.savefig("sns_countplot.png", bbox_inches="tight")  # saves to file
```

### sns.stripplot() and sns.swarmplot()

Show every individual data point alongside a categorical grouping. Stripplot adds jitter; swarmplot avoids overlap by pushing points apart. Use when n < 500 and you want to show the raw spread.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(44)
df = pd.DataFrame({
    "team": np.repeat(["Alpha", "Beta", "Gamma"], 60),
    "tasks_completed": np.concatenate([
        rng.poisson(18, 60), rng.poisson(22, 60), rng.poisson(15, 60)
    ])
})

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
sns.stripplot(data=df, x="team", y="tasks_completed", palette="Set1",
              jitter=True, alpha=0.5, size=5, ax=ax1)
ax1.set_title("Strip Plot (jitter)")
sns.swarmplot(data=df, x="team", y="tasks_completed", palette="Set1",
              size=4, ax=ax2)
ax2.set_title("Swarm Plot (no overlap)")
plt.tight_layout()
plt.savefig("sns_strip_swarm.png", bbox_inches="tight")  # saves to file
```

---

## 10. Relationship Plots (Seaborn)

### sns.scatterplot() — two continuous variables with encoding

Extends `plt.scatter` with direct `hue`, `size`, and `style` parameters mapped from DataFrame columns.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(55)
n = 120
df = pd.DataFrame({
    "hours_studied": rng.uniform(0, 10, n),
    "exam_score":    rng.uniform(40, 100, n),
    "attended_tutoring": rng.choice(["Yes", "No"], n),
    "study_group_size":  rng.integers(1, 6, n),
})

fig, ax = plt.subplots(figsize=(9, 5))
sns.scatterplot(data=df, x="hours_studied", y="exam_score",
                hue="attended_tutoring", size="study_group_size",
                palette={"Yes": "#0D9488", "No": "coral"},
                sizes=(30, 160), alpha=0.75, ax=ax)
ax.set_title("Hours Studied vs Exam Score")
plt.savefig("sns_scatter.png", bbox_inches="tight")  # saves to file
```

### sns.lineplot() — aggregated lines over time

Groups by `hue` and draws mean ± CI bands automatically. Ideal for comparing time series across cohorts without manual aggregation.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(66)
weeks = list(range(1, 13)) * 3
cohort = ["Cohort A"] * 12 + ["Cohort B"] * 12 + ["Cohort C"] * 12
base_vals = {"Cohort A": 0.70, "Cohort B": 0.55, "Cohort C": 0.80}

records = []
for _ in range(5):   # 5 replications per cohort per week (simulate repeated measures)
    for c, weeks_range in [("Cohort A", 12), ("Cohort B", 12), ("Cohort C", 12)]:
        for w in range(1, weeks_range + 1):
            records.append({"week": w, "cohort": c,
                            "retention": base_vals[c] - 0.03 * w + rng.normal(0, 0.05)})

df = pd.DataFrame(records)

fig, ax = plt.subplots(figsize=(9, 4))
sns.lineplot(data=df, x="week", y="retention", hue="cohort",
             palette="Set2", linewidth=2, ax=ax)
ax.set_title("Weekly Retention by Cohort (mean ± 95% CI)")
ax.set_ylabel("Retention Rate")
plt.savefig("sns_lineplot.png", bbox_inches="tight")  # saves to file
```

### sns.regplot() and sns.lmplot()

`regplot` draws a linear fit with confidence band on a single axes. `lmplot` wraps it for faceting across categorical columns.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(77)
n = 100
df = pd.DataFrame({
    "ad_spend": rng.uniform(500, 5000, n),
    "revenue":  rng.uniform(500, 5000, n) * 2.3 + rng.normal(0, 2000, n),
    "region":   rng.choice(["North", "South"], n)
})

# regplot — single axes
fig, ax = plt.subplots(figsize=(7, 4))
sns.regplot(data=df, x="ad_spend", y="revenue",
            scatter_kws={"alpha": 0.5, "color": "steelblue"},
            line_kws={"color": "coral", "linewidth": 2}, ax=ax)
ax.set_title("Ad Spend vs Revenue with Linear Fit")
plt.savefig("sns_regplot.png", bbox_inches="tight")  # saves to file

# lmplot — faceted by region (creates its own figure)
g = sns.lmplot(data=df, x="ad_spend", y="revenue", col="region",
               palette=["steelblue", "coral"], height=4, aspect=1.1)
g.figure.suptitle("Ad Spend vs Revenue by Region", y=1.02)
g.savefig("sns_lmplot.png", bbox_inches="tight")  # saves to file
```

---

## 11. Matrix Plots (Seaborn)

### sns.heatmap() — annotated correlation matrix

The most common use case: display a correlation matrix with values annotated in each cell.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(88)
df = pd.DataFrame(rng.normal(0, 1, (150, 6)),
                  columns=["Age", "Income", "Spend", "Tenure", "Logins", "Churn_Score"])

corr = df.corr()

fig, ax = plt.subplots(figsize=(8, 6))
sns.heatmap(corr, annot=True, fmt=".2f", cmap="coolwarm",
            vmin=-1, vmax=1, center=0,
            linewidths=0.5, linecolor="#2B2B2B",
            ax=ax)
ax.set_title("Correlation Matrix")
plt.savefig("sns_heatmap.png", bbox_inches="tight")  # saves to file
```

### sns.heatmap() — pivot table / frequency matrix

Heatmaps also visualize any 2-D numeric table, not just correlations.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(89)
df = pd.DataFrame({
    "day":     rng.choice(["Mon", "Tue", "Wed", "Thu", "Fri"], 500),
    "hour":    rng.choice(range(9, 18), 500),
    "tickets": rng.integers(1, 10, 500)
})

pivot = df.pivot_table(values="tickets", index="day", columns="hour", aggfunc="sum")
day_order = ["Mon", "Tue", "Wed", "Thu", "Fri"]
pivot = pivot.reindex(day_order)

fig, ax = plt.subplots(figsize=(11, 4))
sns.heatmap(pivot, annot=True, fmt=".0f", cmap="YlOrRd", linewidths=0.3, ax=ax)
ax.set_title("Support Tickets by Day and Hour")
plt.savefig("sns_pivot_heatmap.png", bbox_inches="tight")  # saves to file
```

### sns.clustermap() — hierarchically clustered heatmap

Reorders rows and columns by similarity (hierarchical clustering). Use when you don't know the natural groupings in your feature set.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(90)
# Simulate 3 latent customer segments
data = np.vstack([
    rng.multivariate_normal([5, 2, 8, 1], np.eye(4) * 0.5, 20),
    rng.multivariate_normal([1, 8, 2, 7], np.eye(4) * 0.5, 20),
    rng.multivariate_normal([6, 5, 3, 9], np.eye(4) * 0.5, 20),
])
df = pd.DataFrame(data, columns=["Recency", "Frequency", "Monetary", "Support_Calls"])

g = sns.clustermap(df, standard_scale=1, cmap="vlag",
                   figsize=(8, 8), dendrogram_ratio=0.15)
g.figure.suptitle("Clustered Customer Feature Map", y=1.01)
g.savefig("sns_clustermap.png", bbox_inches="tight")  # saves to file
```

---

## 12. Pair & Grid Plots (Seaborn)

### sns.pairplot() — all-vs-all scatterplot matrix

Shows pairwise relationships across all numeric columns in one call. Diagonal shows per-variable distribution. Expensive on large datasets — subsample if n > 5000.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(100)
n = 200
df = pd.DataFrame({
    "height_cm": rng.normal(170, 10, n),
    "weight_kg": rng.normal(70, 12, n),
    "age":       rng.integers(20, 60, n),
    "bmi":       rng.normal(23, 4, n),
    "group":     rng.choice(["Active", "Sedentary"], n)
})

g = sns.pairplot(df, hue="group", palette={"Active": "#0D9488", "Sedentary": "coral"},
                 diag_kind="kde", plot_kws={"alpha": 0.5, "s": 20})
g.figure.suptitle("Pairplot — Health Metrics by Activity Level", y=1.01)
g.savefig("sns_pairplot.png", bbox_inches="tight")  # saves to file
```

### sns.FacetGrid() — custom multi-panel plots

`FacetGrid` lets you build any plot type across a row/col/hue facet structure. Use it when `pairplot` or `lmplot` don't give you enough control.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

rng = np.random.default_rng(101)
df = pd.DataFrame({
    "score":   rng.normal(70, 15, 600),
    "subject": np.repeat(["Math", "Science", "English"], 200),
    "term":    np.tile(["Fall", "Spring", "Summer"], 200)
})

g = sns.FacetGrid(df, col="subject", row="term",
                  height=3, aspect=1.1, sharey=True)
g.map(sns.histplot, "score", bins=15, color="steelblue", kde=True)
g.set_axis_labels("Score", "Count")
g.set_titles(col_template="{col_name}", row_template="{row_name}")
g.figure.suptitle("Score Distribution by Subject and Term", y=1.02)
g.savefig("sns_facetgrid.png", bbox_inches="tight")  # saves to file
```

---

## 13. Themes & Palettes (Seaborn)

### sns.set_theme() — global style and context

`set_theme` replaces the older `set_style` / `set_context` pair. `context` controls scale: `paper` (smallest), `notebook` (default), `talk`, `poster`.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# Options for style: "darkgrid", "whitegrid", "dark", "white", "ticks"
# Options for context: "paper", "notebook", "talk", "poster"
sns.set_theme(style="whitegrid", context="notebook", font_scale=1.1)

rng = np.random.default_rng(200)
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(np.cumsum(rng.normal(0, 1, 200)), color="#0D9488", linewidth=2)
ax.set_title("whitegrid style, notebook context")
plt.savefig("sns_set_theme.png", bbox_inches="tight")  # saves to file

sns.reset_defaults()   # restore Matplotlib defaults afterward
```

### sns.set_palette() and built-in palettes

Seaborn palettes fall into three categories. Pick the right type for your data.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# Qualitative (categorical, unordered):  "Set1", "Set2", "tab10", "Paired"
# Sequential (low to high):              "Blues", "viridis", "YlOrRd", "rocket"
# Diverging (centered on zero):          "coolwarm", "RdBu", "vlag", "icefire"

fig, axes = plt.subplots(3, 1, figsize=(9, 5))
palette_examples = [
    ("tab10",    "Qualitative — tab10"),
    ("rocket",   "Sequential — rocket"),
    ("coolwarm", "Diverging — coolwarm"),
]
for ax, (pal, title) in zip(axes, palette_examples):
    colors = sns.color_palette(pal, 10)
    for i, c in enumerate(colors):
        ax.barh([0], [1], left=[i], color=c, height=0.8)
    ax.set_xlim(0, 10)
    ax.set_yticks([])
    ax.set_xticks([])
    ax.set_ylabel(title, fontsize=9, rotation=0, labelpad=145, va="center")

plt.suptitle("Palette Types", fontsize=11, y=1.02)
plt.tight_layout()
plt.savefig("sns_palettes.png", bbox_inches="tight")  # saves to file
```

### Custom palette and passing it to any plot

Pin an exact palette for a project so all charts share a consistent color language.

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

brand_palette = ["#0D9488", "#F59E0B", "#94A3B8", "#E11D48", "#7C3AED"]
sns.set_palette(brand_palette)

rng = np.random.default_rng(300)
df = pd.DataFrame({
    "region": np.repeat(["North", "South", "East", "West", "Central"], 80),
    "revenue": np.concatenate([rng.normal(m, 15000, 80)
                                for m in [90000, 70000, 85000, 60000, 75000]])
})

fig, ax = plt.subplots(figsize=(9, 4))
sns.boxplot(data=df, x="region", y="revenue", ax=ax)
ax.set_title("Revenue by Region — Brand Palette")
plt.savefig("sns_brand_palette.png", bbox_inches="tight")  # saves to file

sns.reset_defaults()
```
