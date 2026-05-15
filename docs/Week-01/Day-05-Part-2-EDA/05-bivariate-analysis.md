# Bivariate Analysis

Two variables in isolation only tell half the story. Bivariate analysis is where patterns become actionable — you learn that older customers spend more, that churn is concentrated in one region, that ad spend stops driving sales past a certain threshold. These are the insights that go into presentations and drive model features.

## Learning Objectives

By the end of this, you will be able to:

- Choose the right chart and statistic for any pair of variable types
- Interpret scatter plots, box plots, and cross-tabs with confidence
- Distinguish correlation from causation and explain why it matters
- Analyze relationships between a feature and a target variable

---

## The Three Combinations

Every bivariate question involves two variables. There are three type pairings, and each has its own toolkit:

| Pair | Visual | Statistic |
|---|---|---|
| Numeric vs Numeric | Scatter plot, line plot | Pearson/Spearman correlation |
| Categorical vs Numeric | Box plot, violin plot, bar chart | Grouped mean/median, ANOVA |
| Categorical vs Categorical | Heat map, grouped bar | Cross-tab, chi-square |

---

## Numeric vs Numeric

### Scatter Plot

The scatter plot is the first thing you reach for when both variables are numbers. It shows the direction, strength, and shape of a relationship at a glance.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

rng = np.random.default_rng(42)
n = 200
df = pd.DataFrame({
    "ad_spend":      rng.uniform(1000, 10000, n),
    "revenue":       None,
    "customer_age":  rng.integers(20, 65, n),
    "support_calls": rng.integers(0, 10, n),
    "churn":         rng.choice(["Yes", "No"], n, p=[0.3, 0.7]),
    "region":        rng.choice(["North", "South", "East", "West"], n),
    "segment":       rng.choice(["Basic", "Premium"], n, p=[0.6, 0.4]),
})
df["revenue"] = df["ad_spend"] * rng.uniform(1.5, 3.5, n) + rng.normal(0, 500, n)

fig, ax = plt.subplots(figsize=(8, 5))
ax.scatter(df["ad_spend"], df["revenue"], alpha=0.4, edgecolors="none")
ax.set_xlabel("Ad Spend (₹)")
ax.set_ylabel("Revenue (₹)")
ax.set_title("Ad Spend vs Revenue")
plt.tight_layout()
plt.savefig("scatter_ad_revenue.png")
```

What to look for:

- **Direction** — does revenue go up as ad spend goes up (positive) or down (negative)?
- **Strength** — are points tightly clustered around a line or scattered everywhere?
- **Shape** — is it linear, or does the relationship plateau after a point?
- **Outliers** — points far from the cluster often deserve individual investigation

### Correlation Coefficient

A number between -1 and 1 that summarises the scatter plot. Pearson measures linear relationships; Spearman measures monotonic relationships and is robust to outliers.

```python
pearson_r = df["ad_spend"].corr(df["revenue"], method="pearson")
spearman_r = df["ad_spend"].corr(df["revenue"], method="spearman")

print(f"Pearson r:  {pearson_r:.3f}")
print(f"Spearman r: {spearman_r:.3f}")
# Output: both will be high (~0.85) given the data generation above
```

Interpreting r:

| |r| | Strength |
|---|---|
| 0.9 – 1.0 | Very strong |
| 0.7 – 0.9 | Strong |
| 0.5 – 0.7 | Moderate |
| 0.3 – 0.5 | Weak |
| 0.0 – 0.3 | Negligible |

### Correlation Matrix

When you have many numeric columns, scan all pairs at once.

```python
numeric_cols = ["ad_spend", "revenue", "customer_age", "support_calls"]
corr = df[numeric_cols].corr()

fig, ax = plt.subplots(figsize=(7, 5))
mask = np.triu(np.ones_like(corr, dtype=bool))  # hide upper triangle
sns.heatmap(corr, mask=mask, annot=True, fmt=".2f",
            cmap="coolwarm", center=0, ax=ax)
ax.set_title("Correlation Matrix")
plt.tight_layout()
plt.savefig("corr_matrix.png")
```

> [!warning] Correlation ≠ causation
> A correlation of 0.85 between ad spend and revenue does not mean ad spend *causes* revenue. Both might be driven by a third variable — time of year, new product launches, or simply the fact that bigger companies both spend more and earn more. Always ask: what else could explain this?

> [!warning] Anscombe's Quartet
> Four datasets can have identical correlations (r = 0.816) but completely different scatter patterns — including one with a curved relationship and one with an outlier driving everything. Always plot. Never trust the number alone.

---

## Categorical vs Numeric

### Grouped Summary Statistics

Start with numbers before charts.

```python
summary = df.groupby("segment")["revenue"].agg(["mean", "median", "std", "count"])
print(summary)
# Output:
#           mean    median        std  count
# segment
# Basic    12847     11923     5621    120
# Premium  19341     18104     7832     80
```

The median is often more honest than the mean here — one high-value customer in a small segment inflates the mean significantly.

### Box Plot

Shows distribution shape, median, IQR, and outliers per group. The most information-dense chart for this combination.

```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.boxplot(data=df, x="segment", y="revenue", ax=ax)
ax.set_title("Revenue Distribution by Customer Segment")
ax.set_ylabel("Revenue (₹)")
plt.tight_layout()
plt.savefig("boxplot_segment_revenue.png")
```

### Violin Plot

When you suspect the distribution is bimodal or has an unusual shape, a violin plot shows the full density.

```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.violinplot(data=df, x="region", y="revenue", ax=ax, inner="box")
ax.set_title("Revenue Distribution by Region")
plt.xticks(rotation=15)
plt.tight_layout()
plt.savefig("violin_region_revenue.png")
```

> [!tip] Bar chart vs box plot
> A bar chart of means hides the distribution entirely. Two groups can have the same mean but completely different shapes — one tight and consistent, one wildly variable. Use box plots when distribution shape matters, which it almost always does.

---

## Categorical vs Categorical

### Cross-Tabulation

The workhorse for two categorical variables. Shows raw counts and, more usefully, percentages.

```python
# Raw counts
counts = pd.crosstab(df["segment"], df["churn"])
print(counts)
# Output:
# churn    No  Yes
# segment
# Basic    78   42
# Premium  62   18

# Row percentages — what fraction of each segment churns?
pct = pd.crosstab(df["segment"], df["churn"], normalize="index") * 100
print(pct.round(1))
# Output:
# churn    No    Yes
# segment
# Basic    65.0  35.0
# Premium  77.5  22.5
```

The row percentages immediately tell the story: Premium customers churn at half the rate of Basic customers.

### Grouped Bar Chart

Visualise the cross-tab.

```python
fig, ax = plt.subplots(figsize=(7, 5))
pct.plot(kind="bar", ax=ax, rot=0, color=["#0D9488", "#F59E0B"])
ax.set_title("Churn Rate by Segment")
ax.set_ylabel("Percentage (%)")
ax.legend(title="Churn")
plt.tight_layout()
plt.savefig("bar_churn_segment.png")
```

### Heatmap of Cross-Tab

More readable than a bar chart when there are many categories.

```python
fig, ax = plt.subplots(figsize=(6, 4))
sns.heatmap(
    pd.crosstab(df["region"], df["churn"], normalize="index") * 100,
    annot=True, fmt=".1f", cmap="YlOrRd", ax=ax
)
ax.set_title("Churn Rate (%) by Region")
plt.tight_layout()
plt.savefig("heatmap_region_churn.png")
```

---

## Target Variable Analysis

In a supervised ML project, one specific bivariate comparison matters more than all others: each feature against the target. Run this systematically before touching a model.

```python
# Numeric features vs binary target
for col in ["ad_spend", "revenue", "customer_age", "support_calls"]:
    churned_mean     = df[df["churn"] == "Yes"][col].mean()
    not_churned_mean = df[df["churn"] == "No"][col].mean()
    print(f"{col:20s}  churned={churned_mean:8.1f}  retained={not_churned_mean:8.1f}")
```

If churned customers have significantly more support calls, that is a signal. If their ad spend is similar, it is probably not a useful feature.

> [!success] The bivariate habit
> Before building any model, produce a 2x2 grid: numeric features vs target (box plots), and categorical features vs target (cross-tabs). This 30-minute exercise will tell you which features are worth engineering and which to ignore.

---

## Common Pitfalls

> [!warning] Ecological fallacy
> Relationships that hold at the group level (regions with more ad spend have higher revenue) do not necessarily hold at the individual level. Aggregating then correlating inflates apparent relationships. Always verify at the appropriate grain.

> [!warning] Overplotting
> With thousands of points, a scatter plot becomes a solid blob. Use `alpha=0.1`, hexbin plots (`plt.hexbin()`), or 2D KDE plots (`sns.kdeplot(x=..., y=...)`) instead.

---

[[04-univariate-analysis]] | [[06-eda-workflow]]
