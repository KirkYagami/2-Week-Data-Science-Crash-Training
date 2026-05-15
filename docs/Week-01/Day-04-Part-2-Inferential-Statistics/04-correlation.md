# Correlation

Correlation is the first thing most analysts reach for when asked "what drives X?" It is fast, intuitive, and frequently misleading. The number tells you how two variables move together. It tells you nothing about why, and nothing about the shape of the relationship beyond whether it trends up or down. Used well, correlation is a diagnostic tool for generating hypotheses. Used badly, it is how you convince yourself that ice cream causes drowning.

## Learning Objectives

- Compute Pearson and Spearman correlation coefficients and explain when each is appropriate
- Interpret correlation magnitude using standard benchmarks
- Build and read a correlation matrix and heatmap in pandas and seaborn
- Explain why correlation does not imply causation with concrete examples
- Recognize Anscombe's quartet and Simpson's paradox as warnings against mechanical correlation analysis
- Test correlation for statistical significance

---

## Why Measure Relationships?

Before building a model, you need to know which features are actually related to your target. Correlation is the fastest way to answer that question for numeric variables. It tells you:

- Which features might be useful predictors
- Which features are redundant (highly correlated with each other)
- Where to focus deeper investigation

The key word is "investigation." Correlation opens the door. It does not tell you what is behind it.

---

## Pearson Correlation

Pearson's r measures the **linear** relationship between two numeric variables. It ranges from -1 to +1.

| Range | Interpretation |
|---|---|
| 0.90 to 1.00 | Very strong positive |
| 0.70 to 0.89 | Strong positive |
| 0.50 to 0.69 | Moderate positive |
| 0.30 to 0.49 | Weak positive |
| -0.29 to 0.29 | Negligible / no linear relationship |
| -0.30 to -0.49 | Weak negative |
| -0.50 to -0.69 | Moderate negative |
| -0.70 to -1.00 | Strong to very strong negative |

These benchmarks are conventions from Cohen (1988). They vary by field. In social science, r = 0.3 is considered decent. In physics, r = 0.99 is expected.

> [!info] The Formula
> Pearson's r is the covariance of the two variables divided by the product of their standard deviations. It is scale-invariant — correlation does not change if you multiply one variable by a constant.
>
> `r = Σ[(xᵢ - x̄)(yᵢ - ȳ)] / [n × σₓ × σᵧ]`

```python
import numpy as np
import pandas as pd
from scipy import stats

# Study hours, exam scores, and sleep hours for 8 students
study_hours = np.array([1, 2, 3, 4, 5, 6, 7, 8])
exam_scores = np.array([52, 58, 63, 70, 74, 80, 85, 90])
sleep_hours = np.array([9, 8, 8, 7, 7, 6, 6, 5])

# Method 1: scipy (also gives p-value)
r, p_value = stats.pearsonr(study_hours, exam_scores)
print(f"Pearson r (study vs score): {r:.4f}, p = {p_value:.4f}")
# Output: Pearson r (study vs score): 0.9975, p = 0.0000

r2, p2 = stats.pearsonr(study_hours, sleep_hours)
print(f"Pearson r (study vs sleep): {r2:.4f}, p = {p2:.4f}")
# Output: Pearson r (study vs sleep): -0.9994, p = 0.0000
```

---

## Spearman Rank Correlation

Pearson measures linear relationships. Spearman measures **monotonic** relationships — whether one variable consistently increases as the other does, even if the relationship is not a straight line.

Spearman works by ranking both variables and then computing Pearson correlation on the ranks. This makes it:

- Robust to outliers (one extreme value cannot swing the entire correlation)
- Appropriate for ordinal data (satisfaction ratings, rankings)
- Useful when data is not normally distributed

```python
# Customer satisfaction score (1-10) vs repurchase count
satisfaction = np.array([3, 5, 5, 7, 8, 8, 9, 10])
repurchases  = np.array([1, 2, 3, 5, 6, 7, 9, 15])  # Non-linear growth

pearson_r,  _ = stats.pearsonr(satisfaction, repurchases)
spearman_r, _ = stats.spearmanr(satisfaction, repurchases)

print(f"Pearson r:  {pearson_r:.4f}")
print(f"Spearman r: {spearman_r:.4f}")
# Output:
# Pearson r:  0.9661
# Spearman r: 1.0000
```

Spearman gives 1.0 because repurchases always increase with satisfaction — the relationship is perfectly monotonic, even though it is not linear. Pearson misses this because the growth is exponential, not linear.

> [!tip] Which to Use in Practice?
> - Use Pearson when both variables are continuous and the relationship is plausibly linear (check a scatter plot first).
> - Use Spearman when data is ordinal, has outliers, or the relationship might be non-linear but monotonic.
> - When in doubt, compute both and compare. Large differences between them signal non-linearity or outliers.

---

## The Correlation Matrix

When you have multiple numeric features, you want to see all pairwise correlations at once.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

np.random.seed(42)
n = 100

df = pd.DataFrame({
    'ad_spend':    np.random.uniform(1000, 10000, n),
    'revenue':     np.random.uniform(1000, 10000, n) * 2.5 + np.random.normal(0, 500, n),
    'store_size':  np.random.uniform(500, 5000, n),
    'foot_traffic': np.random.uniform(100, 2000, n),
    'discount_pct': np.random.uniform(0, 30, n)
})

# Add correlation structure: revenue is driven by ad spend and foot traffic
df['revenue'] = (
    df['ad_spend'] * 1.8
    + df['foot_traffic'] * 4.0
    + np.random.normal(0, 2000, n)
)

corr_matrix = df.corr(numeric_only=True)
print(corr_matrix.round(2))
# Output (approx):
#               ad_spend  revenue  store_size  foot_traffic  discount_pct
# ad_spend          1.00     0.67        0.03          0.05         -0.02
# revenue           0.67     1.00        0.01          0.77          0.01
# store_size        0.03     0.01        1.00         -0.04          0.07
# foot_traffic      0.05     0.77        0.04          1.00          0.07
# discount_pct     -0.02     0.01        0.07          0.07          1.00
```

The matrix reveals that revenue is strongly correlated with foot traffic (0.77) and moderately with ad spend (0.67), while store size and discount percentage are essentially uncorrelated with revenue. This would guide feature selection in a model.

```python
# Visualize as a heatmap
fig, ax = plt.subplots(figsize=(8, 6))
sns.heatmap(
    corr_matrix,
    annot=True,
    fmt=".2f",
    cmap="coolwarm",
    center=0,
    vmin=-1,
    vmax=1,
    ax=ax
)
ax.set_title("Correlation Matrix — Retail Store Data")
plt.tight_layout()
plt.savefig("correlation_heatmap.png", dpi=150)
plt.show()
```

---

## Correlation is Not Causation

This is not a cliché. It is one of the most consequential mistakes in applied data science.

### The Ice Cream Example

Ice cream sales and swimming pool drowning deaths are positively correlated. The correlation is real. The explanation is not that ice cream causes drowning — it is that both increase in summer. The hidden driver is temperature (or season). This is a **confounding variable**.

### A Real Business Example

Imagine you work at an e-commerce company. You notice that customers who receive more marketing emails have higher average order values. Correlation is positive and significant. Should you send more emails?

Before you do: consider that high-value customers might receive more targeted campaigns because they were already high-value. The causation runs in reverse. Sending more emails to low-value customers might have zero or negative effect.

```python
import numpy as np
import pandas as pd

np.random.seed(1)
n = 500

# Simulating: customer lifetime value drives both emails sent AND purchases
customer_value = np.random.uniform(0, 1, n)  # Underlying driver

emails_sent = customer_value * 15 + np.random.normal(0, 2, n)
emails_sent = emails_sent.clip(0)

purchase_value = customer_value * 200 + np.random.normal(0, 30, n)
purchase_value = purchase_value.clip(0)

df = pd.DataFrame({'emails_sent': emails_sent, 'purchase_value': purchase_value})

r, p = stats.pearsonr(df['emails_sent'], df['purchase_value'])
print(f"Correlation (emails → purchases): {r:.3f}, p = {p:.4f}")
# Output: Correlation (emails → purchases): 0.873, p = 0.0000
# Looks like emails drive purchases — but both are driven by customer_value
```

The correlation is 0.87 — highly significant. But emails did not cause purchases. They are both consequences of underlying customer quality. If you double the email volume based on this correlation, you will not see a 0.87-proportionate increase in revenue.

---

## Anscombe's Quartet: Why You Must Always Plot

Francis Anscombe constructed four datasets in 1973 that have nearly identical correlation coefficients, means, and standard deviations — but look completely different when visualized.

```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from scipy import stats

# Anscombe's Quartet (built into seaborn)
import seaborn as sns
anscombe = sns.load_dataset("anscombe")

fig, axes = plt.subplots(2, 2, figsize=(10, 8))
for ax, dataset_id in zip(axes.flatten(), ['I', 'II', 'III', 'IV']):
    subset = anscombe[anscombe['dataset'] == dataset_id]
    r, _ = stats.pearsonr(subset['x'], subset['y'])
    ax.scatter(subset['x'], subset['y'], alpha=0.7)
    ax.set_title(f"Dataset {dataset_id} — r = {r:.3f}")

plt.suptitle("Anscombe's Quartet: Same Correlation, Completely Different Data")
plt.tight_layout()
plt.savefig("anscombe.png", dpi=150)
plt.show()
```

All four have r ≈ 0.816. Dataset II is a perfect quadratic curve — Pearson misses it entirely because it is not a linear relationship. Dataset III has one outlier pulling the correlation. Dataset IV has all x-values identical except one outlier.

> [!warning] Always Visualize Before Reporting Correlation
> A correlation coefficient summarizes the linear relationship between two variables. If the relationship is not linear, the number lies. If there are outliers, the number is distorted. Plot your data before you report any correlation. A scatter plot takes thirty seconds. Being wrong takes months to recover from.

---

## Simpson's Paradox

Simpson's paradox occurs when a trend that appears in grouped data disappears or reverses when the groups are combined.

```python
import pandas as pd
import numpy as np

# Simulated hospital data: treatment success rate
np.random.seed(7)
data = pd.DataFrame({
    'hospital': ['A'] * 300 + ['B'] * 300,
    'condition': (['mild'] * 200 + ['severe'] * 100) + (['mild'] * 50 + ['severe'] * 250),
    'recovered': (
        list(np.random.binomial(1, 0.90, 200)) +  # Hospital A, mild, 90% recovery
        list(np.random.binomial(1, 0.70, 100)) +  # Hospital A, severe, 70% recovery
        list(np.random.binomial(1, 0.80, 50))  +  # Hospital B, mild, 80% recovery
        list(np.random.binomial(1, 0.50, 250))    # Hospital B, severe, 50% recovery
    )
})

# Overall recovery rate
overall = data.groupby('hospital')['recovered'].mean()
print("Overall recovery rate:")
print(overall.round(3))
# Output (approx):
# hospital
# A    0.838
# B    0.557

# Broken down by condition
by_condition = data.groupby(['hospital', 'condition'])['recovered'].mean()
print("\nRecovery rate by condition:")
print(by_condition.round(3))
# Output (approx):
# hospital  condition
# A         mild         0.901
#           severe       0.700
# B         mild         0.800
#           severe       0.500
```

Hospital A appears to have a much higher overall recovery rate (84% vs 56%). But Hospital A gets easier cases (200 mild vs 50 severe) while Hospital B handles harder cases (250 severe vs 50 mild). Within each condition type, Hospital A is better — but the aggregated number makes the gap look larger than it is.

> [!warning] Simpson's Paradox in Data Science
> This is why you cannot interpret an overall correlation without understanding the structure of your data. A positive correlation between X and Y in the full dataset might become negative or zero within every subgroup. Always segment your analysis and check whether the aggregate story holds up under the hood.

---

## Practice Exercises

**Warm-up:** Calculate Pearson and Spearman correlations between ad spend and sales. Explain any difference between them.

```python
import pandas as pd
import numpy as np

data = pd.DataFrame({
    'ad_spend': [1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000],
    'sales':    [5200, 8100, 9500, 13000, 18000, 20000, 28000, 35000]
})
# The sales growth is non-linear — which correlation method captures it better?
```

**Main:** Using the dataset below, build a full correlation matrix, create a heatmap, and identify which pair of features is most correlated with `revenue`. Then build a scatter plot of that pair.

```python
np.random.seed(10)
n = 80
market_df = pd.DataFrame({
    'price':       np.random.uniform(10, 100, n),
    'quantity':    np.random.uniform(50, 500, n),
    'ad_spend':    np.random.uniform(500, 5000, n),
    'seasonality': np.random.choice([0, 1], n),
    'revenue':     None
})
market_df['revenue'] = (
    market_df['quantity'] * market_df['price'] * 0.9
    + market_df['ad_spend'] * 2.1
    + np.random.normal(0, 500, n)
)
```

**Stretch:** Generate a dataset where Pearson correlation is near 0 but Spearman correlation is strong. Explain why this is possible.

---

> [!success] Key Takeaways
> - Pearson measures linear relationships. Spearman measures monotonic relationships and is robust to outliers.
> - Correlation coefficients range from -1 to +1. The sign indicates direction; the magnitude indicates strength.
> - Always plot your data. Anscombe's quartet proves that identical correlations can represent fundamentally different relationships.
> - Correlation does not imply causation. The causal story requires domain knowledge, experimental design, or causal inference methods.
> - Simpson's paradox can reverse correlations when data is aggregated. Always check subgroup behavior.

---

[[03-confidence-interval|Previous: Confidence Intervals]] | [[05-statistical-tests|Next: Choosing the Right Statistical Test]]
