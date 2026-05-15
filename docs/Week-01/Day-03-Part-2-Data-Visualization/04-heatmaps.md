# Heatmaps — Correlation Matrices, Pivot Tables, and Missing Data

A heatmap turns a table of numbers into a visual pattern your brain can read in seconds. A well-built correlation heatmap can reveal that two features you planned to use together are 0.95 correlated — redundant, and a sign to drop one before modeling. A poorly built heatmap with the wrong colormap or no masking is just noise. This note covers how to build heatmaps that actually communicate.

## Learning Objectives

- Build a correlation heatmap and mask the upper triangle to remove redundancy
- Choose the right colormap for the data type (diverging vs. sequential)
- Create a pivot table and visualize it as a heatmap
- Visualize missing value patterns with a boolean heatmap
- Format annotations so numbers are readable at scale

---

## Setup

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="white")
```

---

## Correlation Heatmap — The Most Common Use Case

A correlation matrix is a square table where each cell shows the Pearson correlation between two numeric columns. Values range from −1 (perfect negative correlation) to +1 (perfect positive correlation). Zero means no linear relationship.

You will build this before every ML project. It tells you which features move together, which are redundant, and which have a relationship with your target.

```python
np.random.seed(42)
n = 250

# Simulate a student performance dataset
study_hours = np.random.normal(4.5, 1.5, n).clip(0, 10)
sleep_hours = np.random.normal(7.0, 1.2, n).clip(4, 10)
attendance  = np.random.uniform(55, 100, n)
stress      = np.random.normal(5, 2, n).clip(1, 10)
score       = (study_hours * 5 + attendance * 0.3 - stress * 1.5
               + np.random.normal(30, 8, n)).clip(20, 100)
prev_score  = (score * 0.7 + np.random.normal(0, 10, n)).clip(20, 100)

perf_df = pd.DataFrame({
    "study_hours": study_hours,
    "sleep_hours": sleep_hours,
    "attendance":  attendance,
    "stress":      stress,
    "score":       score,
    "prev_score":  prev_score,
})
```

```python
corr = perf_df.corr()

fig, ax = plt.subplots(figsize=(8, 6))

sns.heatmap(
    corr,
    annot=True,
    fmt=".2f",
    cmap="coolwarm",
    center=0,
    vmin=-1, vmax=1,
    linewidths=0.5,
    linecolor="white",
    ax=ax
)

ax.set_title("Feature Correlation Matrix", fontsize=13, pad=12)

plt.tight_layout()
plt.savefig("correlation_heatmap.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!warning] Choosing the Right Colormap
> - **Diverging** (`"coolwarm"`, `"RdBu"`, `"PiYG"`) — use when values have a meaningful center point (zero). Correlation matrices always use diverging. Set `center=0`.
> - **Sequential** (`"Blues"`, `"YlGnBu"`, `"Greens"`) — use when values go from low to high with no meaningful zero (sales totals, counts, frequencies). Never use sequential for correlation — negative correlations will be invisible.
> - **Never use `"jet"` or `"rainbow"`** — they have perceptual discontinuities that introduce false patterns and are not accessible to colorblind readers.

---

## Masking the Upper Triangle

A correlation matrix is symmetric — the value at `[i, j]` equals the value at `[j, i]`. Displaying both triangles doubles the visual information without adding insight. Masking the upper triangle removes the redundancy and makes the matrix easier to scan.

```python
import numpy as np

mask = np.triu(np.ones_like(corr, dtype=bool))

fig, ax = plt.subplots(figsize=(8, 6))

sns.heatmap(
    corr,
    mask=mask,
    annot=True,
    fmt=".2f",
    cmap="coolwarm",
    center=0,
    vmin=-1, vmax=1,
    linewidths=0.5,
    linecolor="white",
    square=True,
    ax=ax
)

ax.set_title("Lower-Triangle Correlation Matrix", fontsize=13, pad=12)

plt.tight_layout()
plt.savefig("correlation_lower_triangle.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!tip] `square=True` Makes the Cells Uniform
> Adding `square=True` forces each cell to be a perfect square regardless of figure dimensions. This makes correlation matrices look cleaner and prevents a stretched, distorted appearance.

---

## Reading the Correlation Matrix

```python
# Find the highest-correlation pairs (excluding self-correlation)
corr_pairs = (
    corr.where(np.tril(np.ones(corr.shape), k=-1).astype(bool))
    .stack()
    .reset_index()
)
corr_pairs.columns = ["feature_1", "feature_2", "correlation"]
corr_pairs = corr_pairs.reindex(
    corr_pairs["correlation"].abs().sort_values(ascending=False).index
)

print(corr_pairs.head(8).to_string(index=False))
# score and prev_score will show the highest correlation
# study_hours and score will also rank high
```

| Value | Meaning |
|---|---|
| 0.9 – 1.0 | Very strong positive — features may be redundant |
| 0.5 – 0.9 | Strong positive — worth investigating |
| 0.0 – 0.5 | Weak to moderate positive |
| −0.5 – 0.0 | Weak to moderate negative |
| −0.9 – −0.5 | Strong negative — one rises as the other falls |
| Below −0.9 | Very strong negative — potentially redundant |

> [!warning] Correlation Is Not Causation
> A high correlation between `score` and `prev_score` does not mean previous scores cause current scores — it means they share common causes (student ability, study habits). Correlation identifies relationships worth investigating. It does not explain them.

---

## Pivot Table Heatmap — Cross-Tabulated Data

A pivot table summarizes a numeric column across two categorical dimensions. Visualizing it as a heatmap makes the pattern visible across all combinations at once.

```python
# Simulate monthly sales by product category and region
months     = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"]
categories = ["Electronics","Apparel","Home & Garden","Sports","Books"]
regions    = ["North","South","East","West"]

np.random.seed(7)
records = []
for month in months:
    for cat in categories:
        for region in regions:
            base = {"Electronics": 180, "Apparel": 95, "Home & Garden": 110,
                    "Sports": 130, "Books": 55}[cat]
            records.append({
                "month": month, "category": cat, "region": region,
                "revenue": int(base * np.random.uniform(0.85, 1.25))
            })

sales_df = pd.DataFrame(records)
```

```python
# Pivot: category × month
pivot_month = sales_df.pivot_table(
    values="revenue",
    index="category",
    columns="month",
    aggfunc="sum"
)
pivot_month = pivot_month[months]   # enforce calendar order

fig, ax = plt.subplots(figsize=(14, 5))

sns.heatmap(
    pivot_month,
    annot=True,
    fmt=".0f",
    cmap="YlGnBu",
    linewidths=0.4,
    linecolor="white",
    ax=ax
)

ax.set_title("Total Revenue by Category and Month (₹ thousands)", fontsize=13, pad=12)
ax.set_xlabel("Month")
ax.set_ylabel("Category")
ax.tick_params(axis="x", rotation=0)

plt.tight_layout()
plt.savefig("pivot_heatmap.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

```python
# Pivot: category × region
pivot_region = sales_df.pivot_table(
    values="revenue",
    index="category",
    columns="region",
    aggfunc="sum"
)

fig, ax = plt.subplots(figsize=(8, 5))

sns.heatmap(
    pivot_region,
    annot=True,
    fmt=".0f",
    cmap="Blues",
    linewidths=0.5,
    linecolor="white",
    ax=ax
)

ax.set_title("Revenue by Category and Region", fontsize=13, pad=12)
ax.set_xlabel("Region")
ax.set_ylabel("Category")

plt.tight_layout()
plt.savefig("pivot_region_heatmap.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

---

## Missing Value Heatmap

Before modeling, you need to know where missing values are clustered. A heatmap of `df.isna()` (which returns a boolean DataFrame) makes the pattern immediately visible.

```python
# Simulate a dataset with realistic missing value patterns
np.random.seed(0)
n_rows = 80

messy_df = pd.DataFrame({
    "customer_id":    range(n_rows),
    "age":            np.where(np.random.rand(n_rows) < 0.12, np.nan,
                               np.random.randint(18, 65, n_rows).astype(float)),
    "income":         np.where(np.random.rand(n_rows) < 0.20, np.nan,
                               np.random.randint(25000, 120000, n_rows).astype(float)),
    "credit_score":   np.where(np.random.rand(n_rows) < 0.08, np.nan,
                               np.random.randint(500, 850, n_rows).astype(float)),
    "loan_amount":    np.where(np.random.rand(n_rows) < 0.25, np.nan,
                               np.random.randint(5000, 50000, n_rows).astype(float)),
    "employment_yrs": np.where(np.random.rand(n_rows) < 0.15, np.nan,
                               np.random.randint(0, 30, n_rows).astype(float)),
    "city":           np.where(np.random.rand(n_rows) < 0.05, np.nan,
                               np.random.choice(["Delhi","Mumbai","Bengaluru"], n_rows)),
})

fig, ax = plt.subplots(figsize=(10, 5))

sns.heatmap(
    messy_df.isna(),
    cbar=False,
    cmap="Reds",
    yticklabels=False,
    ax=ax
)

ax.set_title("Missing Value Pattern — Yellow = Missing", fontsize=13, pad=12)
ax.set_xlabel("Column")
ax.set_ylabel("Row")

# Annotate with percentage missing per column
for i, col in enumerate(messy_df.columns):
    pct = messy_df[col].isna().mean() * 100
    if pct > 0:
        ax.text(i + 0.5, -2, f"{pct:.0f}%", ha="center", va="top",
                fontsize=9, color="#DC2626")

plt.tight_layout()
plt.savefig("missing_values_heatmap.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

> [!info] Patterns in Missing Data Matter
> - **Missing at Random (MAR)** — scattered randomly. Imputation is usually safe.
> - **Missing Not at Random (MNAR)** — income missing for high earners who refuse to disclose. Imputation will introduce bias.
> - **Structurally Missing** — entire columns missing for certain groups. A heatmap immediately reveals this.

---

## Full `sns.heatmap()` Parameter Reference

```python
sns.heatmap(
    data,                 # 2D array or DataFrame
    annot=True,           # display values in cells
    fmt=".2f",            # number format: ".2f" decimal, ".0f" integer, "d" integer (no decimal)
    cmap="coolwarm",      # colormap name
    center=0,             # value to center colormap at
    vmin=-1, vmax=1,      # explicit colormap limits (prevents auto-scaling)
    linewidths=0.5,       # line thickness between cells
    linecolor="white",    # line color between cells
    mask=mask,            # boolean mask — True cells are not plotted
    square=True,          # force square cells
    cbar=True,            # show/hide the color bar
    cbar_kws={"shrink": 0.8},  # colorbar size
    xticklabels=True,     # show x-axis labels
    yticklabels=True,     # show y-axis labels
    ax=ax                 # target axes (always specify this)
)
```

> [!warning] Always Set `vmin` and `vmax` for Correlation Heatmaps
> Without `vmin=-1, vmax=1`, seaborn scales the colormap to the actual range of your data. If your highest correlation is 0.6 and lowest is −0.3, the color at 0.6 will look like "maximum positive" even though it is not. Setting `vmin=-1, vmax=1` keeps the color scale anchored to the true meaning of the data.

---

## Annotation Formatting

Large numbers in heatmap cells are hard to read. Use `fmt` and optionally scale values before plotting.

```python
# Scale revenue to millions before plotting
pivot_scaled = pivot_month / 1000  # now in thousands

fig, ax = plt.subplots(figsize=(14, 5))

sns.heatmap(
    pivot_scaled,
    annot=True,
    fmt=".1f",           # one decimal place
    cmap="YlGnBu",
    linewidths=0.4,
    linecolor="white",
    ax=ax
)

ax.set_title("Revenue by Category and Month (₹ thousands)", fontsize=13, pad=12)

plt.tight_layout()
plt.savefig("pivot_scaled.png", dpi=150, bbox_inches="tight")  # or plt.show()
```

**`fmt` values:**

| fmt | Output |
|---|---|
| `".2f"` | `0.73` |
| `".0f"` | `73` |
| `"d"` | `73` (integer only — will fail on float data) |
| `".1e"` | `7.3e+01` (scientific notation) |
| `".1%"` | `73.2%` |

---

## Practice Exercises

**Warm-up:** Using `perf_df`, compute the correlation matrix. Plot it as a full (unmasked) heatmap with `cmap="RdBu"`, `center=0`, `annot=True`, and `fmt=".2f"`.

**Main:** Plot the same correlation matrix with the upper triangle masked. Add `square=True`. Set `vmin=-1, vmax=1`. Give it a meaningful title. Save at 150 DPI.

**Stretch:** Using `sales_df`, create a pivot table of mean revenue by `region` (rows) and `category` (columns). Plot it as a heatmap using `cmap="Blues"`. Annotate with formatted integers. Underneath the heatmap code, print the top 3 category-region combinations by revenue.

---

## Interview Questions

**Q: Why do you mask the upper triangle of a correlation heatmap?**

??? "Show answer"
    A correlation matrix is symmetric — the value at row i, column j is identical to the value at row j, column i. Displaying both triangles doubles the visual content without adding information. Masking the upper (or lower) triangle reduces clutter, removes duplication, and makes the matrix faster to read. Use `np.triu(np.ones_like(corr, dtype=bool))` to create the mask.

**Q: When should you use a diverging colormap vs. a sequential colormap?**

??? "Show answer"
    Use a diverging colormap (`"coolwarm"`, `"RdBu"`) when the data has a meaningful midpoint and values go in two directions from it — correlation (centered at 0) is the classic example. Use a sequential colormap (`"Blues"`, `"YlGnBu"`) when values go from low to high with no meaningful center — sales totals, counts, frequencies. Using sequential for correlation makes negative correlations look identical to small positive ones, which is misleading.

**Q: What does `center=0` do in `sns.heatmap()`?**

??? "Show answer"
    It anchors the midpoint of the colormap to the value 0. Without it, the colormap center is placed at the midpoint of the data's actual range, which can make near-zero correlations appear strongly positive or negative depending on the data range. Setting `center=0` ensures that zero correlation always maps to the neutral color at the center of the diverging colormap.

**Q: How would you use `pivot_table` to prepare data for a heatmap?**

??? "Show answer"
    `pivot_table` reshapes a long-format DataFrame into a 2D matrix. You specify `index` (row labels), `columns` (column labels), `values` (the number to aggregate), and `aggfunc` (the aggregation: `"sum"`, `"mean"`, `"count"`, etc.). The resulting DataFrame has one row per unique index value and one column per unique column value — exactly the shape `sns.heatmap()` expects.

---

> [!success] Key Takeaways
> - Always mask the upper triangle of a correlation heatmap — it adds no information.
> - Use `cmap="coolwarm"` with `center=0, vmin=-1, vmax=1` for correlation. Use sequential colormaps for totals and counts.
> - Never use `jet` or `rainbow` — they introduce perceptual artifacts.
> - `pivot_table()` is the standard way to prepare cross-tabulated data for a heatmap.
> - Missing value heatmaps reveal whether data is missing randomly or systematically — critical knowledge before imputation.

---

[[03-seaborn-basics|Previous: Seaborn Basics]] | [[05-visualization-best-practices|Next: Visualization Best Practices]]
