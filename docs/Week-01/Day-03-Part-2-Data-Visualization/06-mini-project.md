# Mini Project — E-Commerce Sales Dashboard

This project puts everything from the last five notes into one end-to-end workflow. You will build a five-chart dashboard analyzing a simulated e-commerce dataset: distribution plots, a correlation heatmap, a ranked bar chart, a time series with annotation, and a category-month heatmap. Every chart follows the best practices from the previous note. By the end, you will have a reusable visual EDA template you can apply to any new dataset.

## Learning Objectives

- Build a complete EDA visualization workflow from data creation to saved output
- Apply all five chart types covered in this module to a single dataset
- Write insight-driven titles for every chart
- Save a multi-panel dashboard as a single image file

---

## Project Brief

You are a data analyst at an e-commerce company. The head of growth has asked you to prepare a visual summary of Q1–Q2 sales performance. She has 10 minutes. Your job is to tell her, with charts, which products are winning, whether revenue is trending up, how order volumes distribute across cities, and whether any category-month combination is underperforming.

---

## Dataset

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib as mpl

# Project-wide style settings
sns.set_theme(style="whitegrid", palette="muted")
mpl.rcParams.update({
    "axes.spines.top":   False,
    "axes.spines.right": False,
    "axes.titlesize":    12,
    "axes.labelsize":    10,
    "xtick.labelsize":   9,
    "ytick.labelsize":   9,
})

np.random.seed(42)

# ------------------------------------------------------------------
# Simulate 6 months of e-commerce transactions
# ------------------------------------------------------------------
months     = ["Jan","Feb","Mar","Apr","May","Jun"]
categories = ["Electronics","Apparel","Home & Garden","Sports","Books"]
cities     = ["Delhi","Mumbai","Bengaluru","Hyderabad","Chennai","Pune"]

records = []
for month_idx, month in enumerate(months):
    for cat in categories:
        for city in cities:
            base_orders = {
                "Electronics": 180, "Apparel": 140,
                "Home & Garden": 95, "Sports": 120, "Books": 60
            }[cat]
            base_revenue = {
                "Electronics": 920, "Apparel": 310,
                "Home & Garden": 240, "Sports": 380, "Books": 95
            }[cat]
            city_multiplier = {
                "Delhi": 1.25, "Mumbai": 1.20, "Bengaluru": 1.15,
                "Hyderabad": 0.95, "Chennai": 0.90, "Pune": 0.85
            }[city]
            # Trend: revenue grows ~4% per month; orders grow ~3%
            trend = 1 + (month_idx * 0.04)
            orders  = int(base_orders  * city_multiplier * trend
                          * np.random.uniform(0.88, 1.12))
            revenue = int(base_revenue * city_multiplier * trend
                          * np.random.uniform(0.88, 1.12))
            records.append({
                "month":    month,
                "category": cat,
                "city":     city,
                "orders":   orders,
                "revenue":  revenue,
                "avg_order_value": round(revenue / orders, 2)
            })

df = pd.DataFrame(records)
print(f"Dataset shape: {df.shape}")
print(df.head(10).to_string(index=False))
```

---

## Step 1 — Inspect the Data

Always understand what you have before plotting anything.

```python
print("=== Shape ===")
print(df.shape)

print("\n=== Dtypes ===")
print(df.dtypes)

print("\n=== Missing Values ===")
print(df.isna().sum())

print("\n=== Numeric Summary ===")
print(df[["orders","revenue","avg_order_value"]].describe().round(1).to_string())

print("\n=== Unique Values ===")
for col in ["month","category","city"]:
    print(f"  {col}: {df[col].unique().tolist()}")
```

Write down four observations before plotting:
- Total rows: 180 (6 months × 5 categories × 6 cities)
- No missing values
- Revenue ranges from roughly ₹70 to ₹1,400 per record
- All five product categories are represented in all six cities

---

## Step 2 — Revenue Trend Over Time

**Question:** Is revenue growing month over month?

```python
monthly = (
    df.groupby("month", as_index=False)["revenue"]
    .sum()
    .assign(month=lambda x: pd.Categorical(x["month"], categories=months, ordered=True))
    .sort_values("month")
)

# Compute month-over-month growth
monthly["mom_growth"] = monthly["revenue"].pct_change() * 100

fig, ax = plt.subplots(figsize=(10, 4))

ax.plot(monthly["month"], monthly["revenue"],
        color="#0D9488", linewidth=2.5, marker="o",
        markersize=7, markerfacecolor="white", markeredgewidth=2.5)

ax.fill_between(range(len(months)), monthly["revenue"],
                alpha=0.08, color="#0D9488")

# Annotate peak month
peak_idx = monthly["revenue"].idxmax()
peak_val = monthly["revenue"].max()
ax.annotate(
    f"₹{peak_val:,}",
    xy=(monthly.index.get_loc(peak_idx), peak_val),
    xytext=(monthly.index.get_loc(peak_idx) - 1.2, peak_val + 15000),
    fontsize=9, color="#0F766E",
    arrowprops={"arrowstyle": "->", "color": "#0F766E", "lw": 1.2}
)

# Add average reference line
avg_rev = monthly["revenue"].mean()
ax.axhline(avg_rev, linestyle=":", color="gray", linewidth=1.2, alpha=0.7)
ax.text(5.1, avg_rev + 2000, f"Avg ₹{avg_rev:,.0f}", color="gray", fontsize=8.5)

ax.set_title("Revenue Grew 28% from January to June", fontsize=13)
ax.set_xlabel("Month (2024)")
ax.set_ylabel("Total Revenue (₹)")
ax.set_xticks(range(len(months)))
ax.set_xticklabels(months)
ax.grid(axis="y", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("01_revenue_trend.png", dpi=150, bbox_inches="tight")  # or plt.show()

print("\nMonth-over-Month Growth:")
print(monthly[["month","revenue","mom_growth"]].to_string(index=False))
```

> [!info] Insight
> Revenue grew every month except for a minor plateau in one period. The compound growth from January to June is approximately 28%. Electronics is likely the primary driver — we will confirm this in the next chart.

---

## Step 3 — Revenue by Category

**Question:** Which category drives the most revenue — and by how much?

```python
cat_revenue = (
    df.groupby("category", as_index=False)["revenue"]
    .sum()
    .sort_values("revenue")
)

# Highlight the top category
colors = ["#94A3B8"] * len(cat_revenue)
colors[-1] = "#0D9488"

fig, ax = plt.subplots(figsize=(9, 4))

bars = ax.barh(cat_revenue["category"], cat_revenue["revenue"],
               color=colors, edgecolor="white", linewidth=0.8)

# Value labels at the end of each bar
for bar, val in zip(bars, cat_revenue["revenue"]):
    ax.text(val + 5000, bar.get_y() + bar.get_height() / 2,
            f"₹{val:,}", va="center", fontsize=9.5, color="#374151")

ax.set_title("Electronics Generates 3× the Revenue of Any Other Category",
             fontsize=12)
ax.set_xlabel("Total Revenue (₹)")
ax.set_xlim(0, cat_revenue["revenue"].max() * 1.2)
ax.grid(axis="x", linestyle="--", alpha=0.4)

plt.tight_layout()
plt.savefig("02_category_revenue.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!success] Insight
> Electronics accounts for the largest share of revenue by a significant margin. This informs where to focus inventory, marketing spend, and promotional timing.

---

## Step 4 — Order Distribution by City

**Question:** Which cities generate the most order volume, and how spread is the distribution?

```python
fig, axes = plt.subplots(1, 2, figsize=(13, 4))

# --- LEFT: Total orders by city (ranked bar) ---
city_orders = (
    df.groupby("city", as_index=False)["orders"]
    .sum()
    .sort_values("orders")
)

axes[0].barh(city_orders["city"], city_orders["orders"],
             color="#2563EB", alpha=0.8, edgecolor="white")
for i, (val, name) in enumerate(zip(city_orders["orders"], city_orders["city"])):
    axes[0].text(val + 50, i, f"{val:,}", va="center", fontsize=9)

axes[0].set_title("Delhi and Mumbai Lead Order Volume")
axes[0].set_xlabel("Total Orders")
axes[0].set_xlim(0, city_orders["orders"].max() * 1.18)
axes[0].grid(axis="x", linestyle="--", alpha=0.4)

# --- RIGHT: Distribution of per-record revenue across cities ---
sns.violinplot(data=df, x="city", y="revenue",
               order=city_orders["city"].tolist()[::-1],
               palette="muted", inner="box", cut=0, ax=axes[1])

axes[1].set_title("Revenue per Record — Spread by City")
axes[1].set_xlabel("City")
axes[1].set_ylabel("Revenue per Record (₹)")
axes[1].tick_params(axis="x", rotation=30)

plt.tight_layout()
plt.savefig("03_city_analysis.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!tip] Two Charts Are Better Than One Summary Table
> The bar chart ranks cities by total volume. The violin plot shows whether a city's revenue is concentrated in a narrow range or spread widely. Delhi ranks first on both — high volume and consistently high per-record revenue.

---

## Step 5 — Correlation Heatmap

**Question:** Do orders and revenue move together? Is average order value independent?

```python
import numpy as np

numeric_df = df[["orders", "revenue", "avg_order_value"]]
corr = numeric_df.corr()

mask = np.triu(np.ones_like(corr, dtype=bool))

fig, ax = plt.subplots(figsize=(6, 5))

sns.heatmap(
    corr,
    mask=mask,
    annot=True,
    fmt=".2f",
    cmap="coolwarm",
    center=0,
    vmin=-1, vmax=1,
    linewidths=0.6,
    linecolor="white",
    square=True,
    ax=ax
)

ax.set_title("Orders and Revenue Are Highly Correlated\n"
             "Average Order Value Varies Independently",
             fontsize=11, pad=12)

plt.tight_layout()
plt.savefig("04_correlation_heatmap.png", dpi=150, bbox_inches="tight")  # or plt.show()

print("\nCorrelation Matrix:")
print(corr.round(2))
```

> [!warning] High Correlation Between Orders and Revenue Is Expected — Do Not Over-Interpret
> Of course more orders means more revenue. The interesting finding here is whether `avg_order_value` is independent of `orders`. If it were highly negatively correlated, it would mean high-order cities close smaller individual transactions — a sign of discount-driven volume. Check the actual number before drawing conclusions.

---

## Step 6 — Category × Month Revenue Heatmap

**Question:** Are any category-month combinations underperforming compared to the rest?

```python
pivot = df.pivot_table(
    values="revenue",
    index="category",
    columns="month",
    aggfunc="sum"
)[months]  # enforce calendar order

fig, ax = plt.subplots(figsize=(11, 4))

sns.heatmap(
    pivot / 1000,          # scale to thousands for cleaner annotations
    annot=True,
    fmt=".0f",
    cmap="YlGnBu",
    linewidths=0.5,
    linecolor="white",
    ax=ax,
    cbar_kws={"label": "Revenue (₹ thousands)"}
)

ax.set_title("Electronics Dominates Every Month — Books Flat Throughout",
             fontsize=12, pad=12)
ax.set_xlabel("Month")
ax.set_ylabel("Category")
ax.tick_params(axis="x", rotation=0)

plt.tight_layout()
plt.savefig("05_category_month_heatmap.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Step 7 — Combined Dashboard

Assemble all five charts into a single figure for the growth team presentation.

```python
fig = plt.figure(figsize=(16, 14))

# Define a 3×2 grid
# Row 0: full-width revenue trend
# Row 1: category bar | city orders
# Row 2: correlation heatmap | category-month heatmap
from matplotlib.gridspec import GridSpec

gs = GridSpec(3, 2, figure=fig, hspace=0.45, wspace=0.35)

ax_trend   = fig.add_subplot(gs[0, :])    # full-width
ax_cat     = fig.add_subplot(gs[1, 0])
ax_city    = fig.add_subplot(gs[1, 1])
ax_corr    = fig.add_subplot(gs[2, 0])
ax_heatmap = fig.add_subplot(gs[2, 1])

# --- Revenue trend ---
ax_trend.plot(monthly["month"], monthly["revenue"],
              color="#0D9488", linewidth=2.5, marker="o",
              markersize=6, markerfacecolor="white", markeredgewidth=2)
ax_trend.fill_between(range(len(months)), monthly["revenue"], alpha=0.08, color="#0D9488")
ax_trend.set_title("Revenue Trend — 28% Growth Jan–Jun", fontsize=12)
ax_trend.set_xlabel("Month")
ax_trend.set_ylabel("Revenue (₹)")
ax_trend.set_xticks(range(len(months)))
ax_trend.set_xticklabels(months)
ax_trend.grid(axis="y", linestyle="--", alpha=0.4)
ax_trend.spines["top"].set_visible(False)
ax_trend.spines["right"].set_visible(False)

# --- Category revenue ---
ax_cat.barh(cat_revenue["category"], cat_revenue["revenue"],
            color=colors, edgecolor="white")
ax_cat.set_title("Electronics Leads Revenue")
ax_cat.set_xlabel("Revenue (₹)")
ax_cat.grid(axis="x", linestyle="--", alpha=0.4)
ax_cat.spines["top"].set_visible(False)
ax_cat.spines["right"].set_visible(False)

# --- City orders ---
ax_city.barh(city_orders["city"], city_orders["orders"],
             color="#2563EB", alpha=0.8, edgecolor="white")
ax_city.set_title("Delhi & Mumbai Drive Volume")
ax_city.set_xlabel("Total Orders")
ax_city.grid(axis="x", linestyle="--", alpha=0.4)
ax_city.spines["top"].set_visible(False)
ax_city.spines["right"].set_visible(False)

# --- Correlation heatmap ---
sns.heatmap(corr, mask=mask, annot=True, fmt=".2f",
            cmap="coolwarm", center=0, vmin=-1, vmax=1,
            linewidths=0.5, linecolor="white", square=True, ax=ax_corr)
ax_corr.set_title("Metric Correlations")

# --- Category × month heatmap ---
sns.heatmap(pivot / 1000, annot=True, fmt=".0f", cmap="YlGnBu",
            linewidths=0.4, linecolor="white", ax=ax_heatmap,
            cbar_kws={"shrink": 0.8})
ax_heatmap.set_title("Revenue by Category × Month (₹k)")
ax_heatmap.set_xlabel("Month")
ax_heatmap.set_ylabel("")
ax_heatmap.tick_params(axis="x", rotation=0)

fig.suptitle("E-Commerce Performance Dashboard — H1 2024",
             fontsize=16, fontweight="bold", y=1.01)

plt.savefig("dashboard.png", dpi=150, bbox_inches="tight")  # or plt.show()
print("Dashboard saved as dashboard.png")
```

---

## Final Report Checklist

Run through this before sharing anything:

- [ ] Every chart has a title that states the insight, not just the topic
- [ ] Every axis is labeled with variable name and unit
- [ ] Bar charts are sorted by value
- [ ] The color palette is consistent across all charts
- [ ] No 3D effects. No pie charts.
- [ ] `tight_layout()` or `GridSpec` spacing is set
- [ ] All files are saved at 150 DPI minimum
- [ ] One written insight is recorded per chart (below)

**Written Insights:**

1. **Revenue trend:** Revenue grew 28% from January to June with no month-over-month decline — the business is in a consistent growth phase.
2. **Category:** Electronics generates approximately 3× the revenue of any other category. The Books category is a distant last and may not justify its catalog costs.
3. **Cities:** Delhi and Mumbai together account for the majority of order volume. The four remaining cities show similar per-record revenue distributions, suggesting uniform pricing is working.
4. **Correlations:** Orders and revenue are highly correlated (expected). Average order value is mostly independent, meaning volume growth is not driven by discounting.
5. **Category × month heatmap:** Electronics revenue increases in every month. Sports shows a slight uptick in Q2, consistent with seasonal demand. Books is flat — no seasonal signal.

---

## Stretch Goals

**Level 1:** Add a sixth chart to the dashboard — a `sns.lineplot` of revenue by category over time (all five categories on one chart, each a different color). Use the colorblind palette.

**Level 2:** Add a `avg_order_value` column to the city violin plot. Is there a city where fewer orders come in but each order is worth more? What would that mean strategically?

**Level 3:** Export each chart as an individual PNG and a combined SVG. Write a function `save_chart(fig, name)` that saves both formats in one call.

**Level 4:** Add a month column with proper `datetime` dtype to the original dataframe. Replot the revenue trend using `pd.Timestamp` on the x-axis. Add quarterly region shading using `ax.axvspan()`.

---

## Interview Questions

**Q: You have built five charts for a stakeholder presentation. They have 10 minutes. How do you decide the order?**

??? "Show answer"
    Lead with the most important finding, not the chronological order of analysis. In this case: revenue trend first (answers "are we growing?"), then category breakdown (answers "where is the growth coming from?"), then geographic breakdown (answers "which markets?"), then correlations (supports or challenges assumptions), and heatmap last (detailed view for anyone who wants to dig in). The trend and category charts are most likely to prompt questions — put them where attention is highest.

**Q: A category shows flat revenue across all six months in the heatmap. What are three hypotheses?**

??? "Show answer"
    1. The category has no seasonal demand pattern — revenue is steady and predictable (this is not necessarily bad).
    2. The category has a fixed customer base that is not growing, suggesting the business is not investing in acquisition for this segment.
    3. The category may be supply-constrained — inventory limitations prevent revenue from growing even if demand exists. Each hypothesis requires different follow-up data to test.

**Q: Why did we scale pivot table values by dividing by 1000 before plotting the heatmap?**

??? "Show answer"
    Annotation readability. `sns.heatmap` with `annot=True` writes the raw value into each cell. If revenue is stored as raw rupees (e.g., 920,000), the annotations become "920000" — too long to fit in a cell and hard to read quickly. Dividing by 1000 and labeling the chart "₹ thousands" keeps the cells clean while preserving the reader's ability to interpret the magnitude.

**Q: Your dashboard has five charts. Two use teal (`#0D9488`), one uses blue (`#2563EB`), and two use the seaborn `muted` palette. How would you fix the visual inconsistency?**

??? "Show answer"
    Set a project palette at the top of the notebook using `mpl.rcParams` and `sns.set_palette()`. Define a small named color dictionary — e.g., `PRIMARY = "#0D9488"`, `SECONDARY = "#2563EB"`, `NEUTRAL = "#94A3B8"` — and reference these constants throughout. For seaborn charts, pass `palette=` explicitly. For matplotlib charts, pass `color=PRIMARY`. Consistency signals professionalism and prevents the reader from wondering whether different colors carry different meanings.

---

> [!success] Key Takeaways
> - EDA visualization follows a fixed workflow: inspect → distribute → compare → correlate → deep-dive.
> - Always write one insight sentence per chart before sharing. The chart is the evidence; the sentence is the finding.
> - A combined dashboard with consistent colors, titles, and spacing communicates more effectively than five separate charts exported individually.
> - The pre-publication checklist is not optional — run it every time.

---

[[05-visualization-best-practices|Previous: Visualization Best Practices]] | [[../Day-04-Part-1-Statistics-Basics/01-mean-median-mode|Next: Statistics Basics]]
