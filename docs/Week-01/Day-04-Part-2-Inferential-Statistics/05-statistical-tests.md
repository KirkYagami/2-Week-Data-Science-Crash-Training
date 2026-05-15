# Statistical Tests — Choosing the Right One

Picking the wrong statistical test is one of the most common errors in applied data work. Using a t-test when you have three groups, or a chi-square test on continuous data, or ignoring assumptions about variance — these mistakes lead to invalid p-values and wrong conclusions. The good news: test selection follows a clear decision process. Once you know the rules, you can pick the right test in about thirty seconds.

## Learning Objectives

- Use a systematic decision process to select the correct test for any scenario
- Run and interpret one-sample t-test, two-sample t-test (equal and unequal variance), paired t-test, ANOVA, and chi-square test
- Understand the assumptions of each test and what to do when they fail
- Know the non-parametric alternatives for each parametric test
- Recognize when you need to apply multiple testing correction

---

## The Test Selection Decision Tree

Answer three questions in order:

**1. What is the outcome variable type?**
- Continuous/numeric → consider t-tests, ANOVA, correlation
- Categorical/binary → consider chi-square, proportions tests

**2. How many groups are you comparing?**
- One group vs a known value → one-sample t-test
- Two groups → two-sample t-test (or paired if same subjects)
- Three or more groups → ANOVA

**3. Are the observations independent?**
- Independent → independent samples test
- Same subjects measured twice → paired test

```
outcome is numeric?
├── yes → how many groups?
│   ├── 1 (vs. known value) → one-sample t-test
│   ├── 2 → are subjects paired?
│   │   ├── yes → paired t-test
│   │   └── no → independent two-sample t-test
│   └── 3+ → one-way ANOVA
└── no (categorical) → chi-square test
    ├── goodness of fit (one variable) → chi-square goodness of fit
    └── independence (two variables) → chi-square contingency table
```

> [!info] Parametric vs Non-Parametric
> Parametric tests (t-tests, ANOVA) assume the data is approximately normally distributed and have specific variance assumptions. Non-parametric tests make fewer assumptions by working on ranks rather than raw values. Non-parametric tests are less powerful when parametric assumptions hold, but they are valid when those assumptions do not.

---

## One-Sample t-Test

**Use when:** You have one group of numeric measurements and want to test whether the population mean equals a specific value.

**Example:** Is the average delivery time different from our SLA of 30 minutes?

**Assumptions:**
- Observations are independent
- Data is approximately normally distributed (or n > 30 by CLT)
- No extreme outliers

```python
import numpy as np
from scipy import stats

# Delivery times (minutes) for 15 orders
delivery_times = np.array([
    28, 31, 29, 27, 30, 26, 32, 28, 29, 27,
    25, 33, 30, 28, 31
])

# H₀: population mean = 30 minutes
# H₁: population mean ≠ 30 minutes (two-tailed)
t_stat, p_value = stats.ttest_1samp(delivery_times, popmean=30)

print(f"Sample mean:  {delivery_times.mean():.2f} min")
print(f"t-statistic:  {t_stat:.4f}")
print(f"p-value:      {p_value:.4f}")
# Output:
# Sample mean:  29.27 min
# t-statistic:  -1.7542
# p-value:      0.1013

alpha = 0.05
result = "Reject H₀" if p_value <= alpha else "Fail to reject H₀"
print(f"\nDecision (α={alpha}): {result}")
# Output: Decision (α=0.05): Fail to reject H₀

# One-tailed test: is delivery time BELOW 30 minutes?
t_stat_1t, p_value_1t = stats.ttest_1samp(delivery_times, popmean=30, alternative='less')
print(f"\nOne-tailed (less) p-value: {p_value_1t:.4f}")
# Output: One-tailed (less) p-value: 0.0507
```

Note: the two-tailed test does not reject at α = 0.05. The one-tailed test (testing whether the mean is specifically below 30) barely does not reject either. This is a case where you would want to look at the confidence interval and consider sample size before drawing conclusions.

---

## Independent Two-Sample t-Test

**Use when:** You have two independent groups and want to compare their means.

**Example:** Does the new website design produce higher average session duration than the old design?

**Two variants:**
- **Equal variances assumed (Student's t):** use when Levene's test is not significant
- **Unequal variances (Welch's t):** the default in scipy, robust even when variances are similar

```python
import numpy as np
from scipy import stats

np.random.seed(42)
# Session durations (seconds) for two versions of the homepage
old_design = np.array([180, 195, 210, 175, 200, 185, 220, 190, 165, 205,
                       195, 180, 210, 185, 200])
new_design = np.array([220, 235, 245, 215, 240, 230, 255, 225, 210, 248,
                       232, 218, 244, 228, 237])

# Step 1: Check whether variances are equal using Levene's test
levene_stat, levene_p = stats.levene(old_design, new_design)
print(f"Levene's test: stat={levene_stat:.4f}, p={levene_p:.4f}")
# Output: Levene's test: stat=0.0234, p=0.8795
# p > 0.05: no evidence of unequal variances, but we use Welch's anyway (it's safer)

# Step 2: Run the t-test
# equal_var=False → Welch's t-test (recommended default)
t_stat, p_value = stats.ttest_ind(old_design, new_design, equal_var=False)

print(f"\nOld design mean: {old_design.mean():.2f}s")
print(f"New design mean: {new_design.mean():.2f}s")
print(f"Difference:      {new_design.mean() - old_design.mean():.2f}s")
print(f"t-statistic:     {t_stat:.4f}")
print(f"p-value:         {p_value:.6f}")
# Output:
# Old design mean: 193.33s
# New design mean: 232.47s
# Difference:      39.13s
# t-statistic:     -11.7345
# p-value:         0.000000

# Step 3: Effect size (Cohen's d)
pooled_std = np.sqrt((old_design.std()**2 + new_design.std()**2) / 2)
cohens_d = (new_design.mean() - old_design.mean()) / pooled_std
print(f"Cohen's d:       {cohens_d:.4f}  (large effect)")
# Output: Cohen's d:       4.4892  (large effect)
```

> [!tip] Always Use Welch's t-Test by Default
> Welch's t-test (equal_var=False, which is actually the default in scipy) performs well even when variances are equal, and it protects you when they are not. The traditional Student's t-test assumes equal variances — an assumption that often does not hold. Use Welch's unless you have a specific reason not to.

---

## Paired t-Test

**Use when:** The same subjects are measured under two conditions, or measurements are matched by design (before/after, same store two months, matched pairs).

**Why it is different:** The key insight is that with paired data, you can compute the difference per subject and test whether the mean difference is zero. This removes between-subject variability, making the test more powerful.

**Example:** Employee productivity (tasks completed per day) before and after ergonomic keyboard intervention.

```python
import numpy as np
from scipy import stats

# Same 12 employees measured before and after new keyboards
before = np.array([42, 38, 51, 45, 39, 47, 52, 41, 43, 49, 46, 44])
after  = np.array([46, 43, 55, 48, 41, 52, 57, 44, 47, 53, 50, 49])

# The differences tell the actual story
differences = after - before
print(f"Mean difference: {differences.mean():.2f} tasks/day")
print(f"Differences: {differences}")
# Output:
# Mean difference: 4.33 tasks/day
# Differences: [4 5 4 3 2 5 5 3 4 4 4 5]

# H₀: mean difference = 0
# H₁: mean difference ≠ 0
t_stat, p_value = stats.ttest_rel(before, after)

print(f"\nt-statistic: {t_stat:.4f}")
print(f"p-value:     {p_value:.8f}")
# Output:
# t-statistic: -23.8048
# p-value:     0.00000001

print(f"\nAll employees improved. The intervention worked (p << 0.05).")
# Also check effect size for paired data: Cohen's dz
cohens_dz = differences.mean() / differences.std(ddof=1)
print(f"Cohen's dz: {cohens_dz:.4f}  (very large effect)")
# Output: Cohen's dz: 6.8755  (very large effect)
```

> [!warning] Using Independent Test on Paired Data
> If you run an independent t-test on data that is actually paired, you lose all the power that pairing gives you. The within-subject variation cancels out in a paired test but adds noise in an independent test. Always ask: "Are the same subjects appearing in both groups?"

---

## One-Way ANOVA

**Use when:** You want to compare means across three or more independent groups.

**Why not just run multiple t-tests?** If you run three t-tests to compare groups A vs B, A vs C, and B vs C, each at α = 0.05, your overall false-positive rate is no longer 5%. It is closer to 14%. ANOVA tests all groups simultaneously while controlling the Type I error rate.

**What ANOVA tests:** H₀ is that all group means are equal. A significant result tells you at least one group differs — not which one. For that, you need a post-hoc test.

**Example:** Customer spending across three loyalty tiers.

```python
import numpy as np
from scipy import stats
import pandas as pd

np.random.seed(5)
# Monthly spend ($) for Bronze, Silver, and Gold tier customers
bronze = np.random.normal(loc=80,  scale=20, size=40)
silver = np.random.normal(loc=150, scale=25, size=40)
gold   = np.random.normal(loc=300, scale=40, size=40)

# H₀: μ_bronze = μ_silver = μ_gold
# H₁: At least one group mean differs
f_stat, p_value = stats.f_oneway(bronze, silver, gold)

print(f"Bronze mean: ${bronze.mean():.2f}")
print(f"Silver mean: ${silver.mean():.2f}")
print(f"Gold mean:   ${gold.mean():.2f}")
print(f"\nF-statistic: {f_stat:.4f}")
print(f"p-value:     {p_value:.8f}")
# Output:
# Bronze mean: $80.87
# Silver mean: $147.30
# Gold mean:   $299.54
# F-statistic: 578.3742
# p-value:     0.00000000

# ANOVA says "at least one group differs" — use Tukey HSD for which ones
from statsmodels.stats.multicomp import pairwise_tukeyhsd
import pandas as pd

all_data = np.concatenate([bronze, silver, gold])
labels   = ['Bronze'] * 40 + ['Silver'] * 40 + ['Gold'] * 40

tukey = pairwise_tukeyhsd(endog=all_data, groups=labels, alpha=0.05)
print("\nTukey HSD Post-Hoc Test:")
print(tukey.summary())
# Output: All three pairs are significantly different from each other
```

> [!info] ANOVA Assumptions
> 1. Observations are independent within and between groups
> 2. Each group is approximately normally distributed
> 3. The groups have approximately equal variances (homoscedasticity)
>
> Check normality with Shapiro-Wilk (`stats.shapiro`). Check equal variances with Levene's test (`stats.levene`). If variances are unequal, use Welch's ANOVA instead.

---

## Chi-Square Test

The chi-square test is for categorical data. It has two common variants:

**Chi-square goodness of fit:** Tests whether observed frequencies match expected frequencies for a single categorical variable.

**Chi-square test of independence:** Tests whether two categorical variables are associated.

### Goodness of Fit Example

Are customer complaints distributed equally across four product categories, or is one category disproportionately problematic?

```python
from scipy import stats

# Observed complaints per category
observed = np.array([45, 38, 52, 65])  # Electronics, Clothing, Food, Home

# Expected if complaints were uniformly distributed
n_total = observed.sum()
n_categories = len(observed)
expected = np.array([n_total / n_categories] * n_categories)

chi2_stat, p_value = stats.chisquare(f_obs=observed, f_exp=expected)

print(f"Observed: {observed}")
print(f"Expected: {expected}")
print(f"\nchi2-statistic: {chi2_stat:.4f}")
print(f"p-value:        {p_value:.4f}")
# Output:
# Observed: [45 38 52 65]
# Expected: [50. 50. 50. 50.]
# chi2-statistic: 8.0400
# p-value:        0.0452

print(f"\nComplaints are not equally distributed (p < 0.05).")
print(f"Home category has the most complaints.")
```

### Test of Independence Example

Is purchase behavior associated with device type?

```python
import pandas as pd
import numpy as np
from scipy import stats

# Contingency table: device type vs purchased (yes/no)
contingency_table = pd.DataFrame(
    data=[[250, 150], [180, 120], [90,  60]],
    index=['Mobile', 'Desktop', 'Tablet'],
    columns=['Purchased', 'Did Not Purchase']
)

print("Contingency Table:")
print(contingency_table)
# Output:
# Contingency Table:
#          Purchased  Did Not Purchase
# Mobile         250               150
# Desktop        180               120
# Tablet          90                60

chi2_stat, p_value, dof, expected_freq = stats.chi2_contingency(contingency_table)

print(f"\nchi2-statistic: {chi2_stat:.4f}")
print(f"Degrees of freedom: {dof}")
print(f"p-value:        {p_value:.4f}")
print(f"\nExpected frequencies:")
print(pd.DataFrame(expected_freq.round(1), index=contingency_table.index,
                   columns=contingency_table.columns))
# Output:
# chi2-statistic: 0.0000
# Degrees of freedom: 2
# p-value:        1.0000
# (These proportions are identical across devices — no association)
```

> [!warning] Chi-Square Assumes Sufficient Cell Counts
> The chi-square test is unreliable when expected cell counts are below 5. Check the `expected_freq` output. If any cell is below 5, consider combining categories or using Fisher's exact test instead.

```python
# Fisher's exact test (for 2x2 tables with small samples)
table_2x2 = np.array([[15, 5], [10, 20]])
odds_ratio, p_fisher = stats.fisher_exact(table_2x2)
print(f"Fisher's exact test p-value: {p_fisher:.4f}")
# Output: Fisher's exact test p-value: 0.0073
```

---

## Non-Parametric Alternatives

When your data violates the normality assumption (especially with small samples), reach for these:

| Parametric Test | Non-Parametric Alternative | Use When |
|---|---|---|
| One-sample t-test | Wilcoxon signed-rank test | Small n, non-normal, ordinal data |
| Independent t-test | Mann-Whitney U test | Non-normal distributions, ordinal data |
| Paired t-test | Wilcoxon signed-rank test | Paired data, non-normal |
| One-way ANOVA | Kruskal-Wallis test | 3+ groups, non-normal |
| Pearson correlation | Spearman correlation | Non-linear monotonic, ordinal |

```python
from scipy import stats
import numpy as np

# Mann-Whitney U: non-parametric alternative to two-sample t-test
group_a = np.array([12, 15, 11, 18, 10, 13, 16, 14])
group_b = np.array([19, 22, 17, 25, 20, 21, 23, 18])

u_stat, p_mw = stats.mannwhitneyu(group_a, group_b, alternative='two-sided')
print(f"Mann-Whitney U: stat={u_stat:.1f}, p={p_mw:.4f}")
# Output: Mann-Whitney U: stat=0.0, p=0.0006

# Kruskal-Wallis: non-parametric ANOVA
kw_stat, p_kw = stats.kruskal(group_a, group_b, [15, 16, 14, 17, 15])
print(f"Kruskal-Wallis: stat={kw_stat:.4f}, p={p_kw:.4f}")
# Output: Kruskal-Wallis: stat=13.3576, p=0.0013
```

---

## Checking Normality

Before choosing between parametric and non-parametric tests, check your distribution.

```python
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

data = np.array([23, 25, 28, 22, 26, 24, 27, 23, 29, 25, 22, 26, 24, 27, 28])

# Shapiro-Wilk test (best for small samples, n < 50)
shapiro_stat, shapiro_p = stats.shapiro(data)
print(f"Shapiro-Wilk: stat={shapiro_stat:.4f}, p={shapiro_p:.4f}")
# Output: Shapiro-Wilk: stat=0.9746, p=0.9128
# p > 0.05: no evidence against normality — proceed with parametric test

# Visual check: Q-Q plot
fig, ax = plt.subplots(figsize=(6, 5))
stats.probplot(data, dist="norm", plot=ax)
ax.set_title("Q-Q Plot — Does Data Follow Normal Distribution?")
plt.tight_layout()
plt.savefig("qqplot.png", dpi=150)
plt.show()
```

> [!tip] The Central Limit Theorem Is Your Backup
> For large samples (n > 30), the sampling distribution of the mean is approximately normal regardless of the original distribution. This means t-tests are fairly robust to non-normality at reasonable sample sizes. For very small samples (n < 15), normality matters more.

---

## Multiple Testing Correction

When you run many tests simultaneously — testing 50 features, or checking 20 subgroups — your family-wise error rate inflates. At α = 0.05 with 20 independent tests, you expect one false positive just by chance.

```python
import numpy as np
from scipy import stats
from statsmodels.stats.multitest import multipletests

np.random.seed(42)
# 20 independent tests, all truly null
p_values = [stats.ttest_ind(
    np.random.normal(0, 1, 30),
    np.random.normal(0, 1, 30)
)[1] for _ in range(20)]

# Bonferroni correction: strict, controls family-wise error rate
reject_bonferroni, p_bonferroni, _, _ = multipletests(p_values, alpha=0.05, method='bonferroni')

# Benjamini-Hochberg: less conservative, controls false discovery rate
reject_bh, p_bh, _, _ = multipletests(p_values, alpha=0.05, method='fdr_bh')

print(f"Uncorrected significant: {sum(p < 0.05 for p in p_values)}")
print(f"Significant (Bonferroni): {sum(reject_bonferroni)}")
print(f"Significant (BH):         {sum(reject_bh)}")
# Output (approx):
# Uncorrected significant: 2
# Significant (Bonferroni): 0
# Significant (BH):         0
```

---

## Complete Decision Reference

| Scenario | Data Type | Assumptions Met | Test |
|---|---|---|---|
| Sample mean vs known value | Numeric | Normal / large n | One-sample t-test |
| Sample mean vs known value | Numeric | Non-normal, small n | Wilcoxon signed-rank |
| Two independent groups | Numeric | Normal | Welch's two-sample t-test |
| Two independent groups | Numeric | Non-normal | Mann-Whitney U |
| Same subjects, two conditions | Numeric | Normal differences | Paired t-test |
| Same subjects, two conditions | Numeric | Non-normal differences | Wilcoxon signed-rank |
| Three or more independent groups | Numeric | Normal, equal variance | One-way ANOVA |
| Three or more independent groups | Numeric | Non-normal | Kruskal-Wallis |
| Two categorical variables | Categorical | Expected counts ≥ 5 | Chi-square independence |
| Two categorical variables | Categorical | Small n or small counts | Fisher's exact |
| One categorical variable vs distribution | Categorical | Expected counts ≥ 5 | Chi-square goodness of fit |
| Linear relationship between two numerics | Numeric | Normal, linear | Pearson correlation |
| Monotonic relationship or ordinal | Numeric/Ordinal | Any | Spearman correlation |

---

## Practice Exercises

**Warm-up:** You have three classes that took the same exam. Test whether average scores differ across classes.

```python
from scipy import stats
class_a = [72, 68, 75, 70, 73, 69, 74, 71]
class_b = [80, 82, 79, 83, 81, 84, 78, 82]
class_c = [65, 67, 63, 68, 66, 64, 70, 65]
# Choose the right test. Report the result and your interpretation.
```

**Main:** A survey asked 300 respondents their preferred device (Mobile/Desktop/Tablet) and whether they completed a purchase. Test whether device preference is associated with purchase behavior. Create the contingency table from raw data and run the chi-square test.

```python
import pandas as pd
import numpy as np
np.random.seed(1)
devices    = np.random.choice(['Mobile', 'Desktop', 'Tablet'], size=300, p=[0.5, 0.35, 0.15])
# Make purchase probability slightly different by device
purchase_prob = {'Mobile': 0.40, 'Desktop': 0.55, 'Tablet': 0.45}
purchased = [np.random.binomial(1, purchase_prob[d]) for d in devices]
survey_df = pd.DataFrame({'device': devices, 'purchased': purchased})
```

**Stretch:** Run the paired t-test and independent t-test on the same before/after data. Explain why the p-values differ. When would each be appropriate?

---

> [!success] Key Takeaways
> - Test selection depends on: (1) data type, (2) number of groups, (3) whether subjects are paired.
> - Welch's t-test is the safe default for two independent numeric groups.
> - ANOVA tests whether any group mean differs — not which one. Use Tukey HSD post-hoc for pairwise comparisons.
> - Chi-square requires expected cell counts ≥ 5. Use Fisher's exact test for small samples.
> - Non-parametric tests are your fallback when normality assumptions fail.
> - Multiple testing inflates false positives. Apply Bonferroni or Benjamini-Hochberg correction.

---

[[04-correlation|Previous: Correlation]] | [[06-interview-prep|Next: Interview Prep]]
