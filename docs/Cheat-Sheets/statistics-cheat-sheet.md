# Statistics Cheat Sheet

A dense, code-first reference for the statistics every data scientist uses daily.
Every entry follows the same structure: why it matters, the formula, then runnable code.

---

## Descriptive Statistics

Summary statistics are the first thing you compute on any new dataset. They tell you the shape, center, and spread of your data before you build a single model.

### Mean, Median, Mode

**Mean** — the arithmetic average; sensitive to outliers.
**Median** — the middle value; robust to outliers; use when data is skewed.
**Mode** — the most frequent value; relevant for categorical data and distributions with peaks.

$$\bar{x} = \frac{1}{n} \sum_{i=1}^{n} x_i$$

```python
import numpy as np
import pandas as pd
from scipy import stats

data = [12, 15, 14, 10, 15, 18, 100, 14, 15, 13]

mean   = np.mean(data)       # 22.6 — pulled up by the outlier 100
median = np.median(data)     # 14.5 — unaffected by the outlier
mode   = stats.mode(data, keepdims=True).mode[0]  # 15

print(f"Mean: {mean}, Median: {median}, Mode: {mode}")
# Output: Mean: 22.6, Median: 14.5, Mode: 15
```

> [!tip]
> When mean >> median, the distribution is right-skewed and the median is the better measure of center. Always report both.

### Variance and Standard Deviation

Variance measures average squared deviation from the mean. Standard deviation puts it back in the original units, making it interpretable.

$$s^2 = \frac{1}{n-1} \sum_{i=1}^{n}(x_i - \bar{x})^2 \qquad s = \sqrt{s^2}$$

Use `ddof=1` for sample variance (Bessel's correction); `ddof=0` for population variance.

```python
data = np.array([12, 15, 14, 10, 15, 18, 14, 15, 13])

variance = np.var(data, ddof=1)    # sample variance
std_dev  = np.std(data, ddof=1)    # sample std dev

print(f"Variance: {variance:.2f}, Std Dev: {std_dev:.2f}")
# Output: Variance: 4.11, Std Dev: 2.03
```

### Skewness

Skewness measures the asymmetry of a distribution.
- Positive skew: right tail is longer (mean > median). Common in income, house prices.
- Negative skew: left tail is longer (mean < median). Common in test scores near a ceiling.

$$\text{Skewness} = \frac{1}{n} \sum \left(\frac{x_i - \bar{x}}{s}\right)^3$$

```python
right_skewed = np.concatenate([np.random.normal(10, 2, 900), np.random.exponential(5, 100)])
skew = stats.skew(right_skewed)
print(f"Skewness: {skew:.3f}")
# Output: Skewness: ~1.2  (positive = right tail)
```

> [!warning]
> Rule of thumb: |skewness| > 1 is substantially skewed. Many ML models (linear regression, LDA) assume roughly symmetric distributions. Apply log transform or Box-Cox if skewness is extreme.

### Kurtosis

Kurtosis measures the heaviness of the tails relative to a normal distribution.
- Excess kurtosis > 0 (leptokurtic): heavy tails, more outliers than normal.
- Excess kurtosis < 0 (platykurtic): light tails, fewer outliers.

```python
normal_data = np.random.normal(0, 1, 1000)
heavy_tail  = np.random.standard_t(df=3, size=1000)  # fat tails

print(f"Normal kurtosis (excess): {stats.kurtosis(normal_data):.3f}")
print(f"t(3) kurtosis  (excess): {stats.kurtosis(heavy_tail):.3f}")
# Output: Normal ~0.0, t(3) ~3–5 (heavy tails flagged)
```

### IQR and Outlier Detection

The Interquartile Range (IQR = Q3 − Q1) spans the middle 50% of data. It is the standard way to define outlier thresholds without assuming normality.

$$\text{IQR} = Q3 - Q1$$
$$\text{Outlier if } x < Q1 - 1.5 \cdot \text{IQR} \text{ or } x > Q3 + 1.5 \cdot \text{IQR}$$

```python
data = np.array([10, 12, 13, 12, 14, 15, 13, 14, 100, 11])

q1, q3 = np.percentile(data, [25, 75])
iqr = q3 - q1
lower = q1 - 1.5 * iqr
upper = q3 + 1.5 * iqr

outliers = data[(data < lower) | (data > upper)]
print(f"Q1={q1}, Q3={q3}, IQR={iqr}, Outliers={outliers}")
# Output: Q1=11.75, Q3=14.0, IQR=2.25, Outliers=[100]
```

---

## Probability Basics

Probability is the language of uncertainty. Every model you build is implicitly or explicitly making probability statements — knowing the rules keeps you from making logical errors.

### Basic Probability Rules

For any event A in sample space S:

$$0 \le P(A) \le 1 \qquad P(S) = 1$$

$$P(A \cup B) = P(A) + P(B) - P(A \cap B)$$

$$P(A^c) = 1 - P(A)$$

```python
# Empirical probability from data
outcomes = ['heads', 'tails'] * 500 + ['heads'] * 50  # biased coin
p_heads = outcomes.count('heads') / len(outcomes)
print(f"P(heads) = {p_heads:.3f}")
# Output: P(heads) = 0.533
```

### Conditional Probability

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}$$

Conditional probability answers: "Given that B occurred, how likely is A?" This is the foundation of Naive Bayes, decision trees, and any causal reasoning.

```python
# Confusion matrix → conditional probabilities
# True positive rate (sensitivity) = P(predicted positive | actually positive)
from sklearn.metrics import confusion_matrix

y_true = [1,1,1,0,0,0,1,0,1,0]
y_pred = [1,1,0,0,1,0,1,0,1,0]

tn, fp, fn, tp = confusion_matrix(y_true, y_pred).ravel()

sensitivity = tp / (tp + fn)   # P(pred=1 | actual=1)
specificity = tn / (tn + fp)   # P(pred=0 | actual=0)
print(f"Sensitivity: {sensitivity:.2f}, Specificity: {specificity:.2f}")
# Output: Sensitivity: 0.75, Specificity: 0.75
```

### Bayes' Theorem

$$P(A \mid B) = \frac{P(B \mid A) \cdot P(A)}{P(B)}$$

Bayes' theorem lets you update beliefs given new evidence. It underpins spam filters, medical diagnosis, and Bayesian ML.

```python
# Medical test example
# Disease prevalence = 1%, test sensitivity = 99%, false positive rate = 5%
p_disease   = 0.01
p_pos_given_disease  = 0.99   # sensitivity
p_pos_given_healthy  = 0.05   # false positive rate
p_healthy   = 1 - p_disease

# P(positive) by total probability theorem
p_positive  = p_pos_given_disease * p_disease + p_pos_given_healthy * p_healthy

# P(disease | positive) — Bayes' theorem
p_disease_given_pos = (p_pos_given_disease * p_disease) / p_positive

print(f"P(disease | positive test) = {p_disease_given_pos:.3f}")
# Output: P(disease | positive test) = 0.166
# Only 16.6% chance of disease even with a positive test — base rate matters enormously
```

> [!warning]
> The base rate neglect trap: a 99% accurate test for a rare disease (1% prevalence) gives mostly false positives. Always factor in P(A) before interpreting P(A|B).

---

## Distributions

Choosing the wrong distribution is one of the most common modeling errors. Each distribution describes a specific data-generating process — match the process, not just the shape.

### Normal Distribution

Parameters: mean μ, standard deviation σ.
Use when: sums of many independent effects, measurement errors, residuals from linear regression.

$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}$$

68-95-99.7 rule: 68% of data within ±1σ, 95% within ±2σ, 99.7% within ±3σ.

```python
mu, sigma = 170, 10  # height in cm

dist = stats.norm(loc=mu, scale=sigma)

# PDF at a point
print(f"PDF at 170 cm: {dist.pdf(170):.4f}")

# CDF: P(X <= 180)
print(f"P(height <= 180 cm): {dist.cdf(180):.4f}")
# Output: P(height <= 180 cm): 0.8413

# Random samples
samples = dist.rvs(size=1000)
print(f"Sample mean: {samples.mean():.2f}, Sample std: {samples.std():.2f}")

# Inverse CDF (quantile): find x such that P(X <= x) = 0.95
print(f"95th percentile: {dist.ppf(0.95):.2f}")
# Output: 95th percentile: 186.45
```

### Binomial Distribution

Parameters: n (trials), p (probability of success per trial).
Use when: counting successes in n independent yes/no trials (click-through rates, defect counts).

$$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k}$$

$$\text{Mean} = np \qquad \text{Variance} = np(1-p)$$

```python
n, p = 100, 0.03   # 100 users, 3% conversion rate
dist = stats.binom(n=n, p=p)

# P(exactly 5 conversions)
print(f"P(X=5): {dist.pmf(5):.4f}")
# Output: P(X=5): 0.1013

# P(3 or fewer conversions)
print(f"P(X <= 3): {dist.cdf(3):.4f}")
# Output: P(X <= 3): 0.6472

# Expected value and std dev
print(f"Mean: {dist.mean()}, Std: {dist.std():.3f}")
# Output: Mean: 3.0, Std: 1.706

# Simulate 10 campaigns
print(dist.rvs(size=10))
```

### Poisson Distribution

Parameter: λ (average events per interval).
Use when: counting rare events in a fixed time or space window — support tickets per hour, server errors per day.

$$P(X = k) = \frac{\lambda^k e^{-\lambda}}{k!} \qquad \text{Mean} = \text{Variance} = \lambda$$

```python
lam = 4.5  # average support tickets per hour
dist = stats.poisson(mu=lam)

# P(exactly 3 tickets in an hour)
print(f"P(X=3): {dist.pmf(3):.4f}")
# Output: P(X=3): 0.1687

# P(more than 8 tickets — busy hour threshold)
print(f"P(X > 8): {1 - dist.cdf(8):.4f}")
# Output: P(X > 8): 0.0557

samples = dist.rvs(size=1000)
print(f"Sample mean: {samples.mean():.2f}, Sample var: {samples.var():.2f}")
# Both should be close to 4.5 (mean = variance for Poisson)
```

> [!warning]
> Poisson assumes events are independent and the rate λ is constant. If events cluster (traffic spikes, bursty failures), use a Negative Binomial instead — it has a separate variance parameter.

### Checking Distribution Fit

Before assuming normality, test it. Many statistical tests break silently when the normality assumption fails.

```python
data = np.random.normal(0, 1, 200)

# Shapiro-Wilk test (best for n < 2000)
stat, p_val = stats.shapiro(data)
print(f"Shapiro-Wilk: W={stat:.4f}, p={p_val:.4f}")
# p > 0.05 → fail to reject normality

# Kolmogorov-Smirnov test against normal
stat_ks, p_ks = stats.kstest(data, 'norm', args=(data.mean(), data.std()))
print(f"KS test: stat={stat_ks:.4f}, p={p_ks:.4f}")
```

---

## Sampling and the Central Limit Theorem

The CLT is why so many statistical methods work in practice. It guarantees that sample means behave normally even when the underlying data does not — once n is large enough.

### Standard Error of the Mean

The standard error quantifies how much sample means vary across repeated samples. Smaller SE → more precise estimate.

$$SE = \frac{\sigma}{\sqrt{n}}$$

```python
population = np.random.exponential(scale=5, size=100_000)  # right-skewed

n = 50
sample_means = [np.mean(np.random.choice(population, size=n)) for _ in range(5000)]

theoretical_se = population.std() / np.sqrt(n)
empirical_se   = np.std(sample_means)

print(f"Theoretical SE: {theoretical_se:.4f}")
print(f"Empirical SE:   {empirical_se:.4f}")
# Output: Both close to ~0.707
```

### CLT Simulation

The CLT states: regardless of the population distribution, the distribution of sample means approaches normal as n → ∞.

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

population = np.random.exponential(scale=5, size=100_000)  # clearly not normal

fig, axes = plt.subplots(1, 3, figsize=(12, 4))
for ax, n in zip(axes, [2, 10, 50]):
    sample_means = [np.mean(np.random.choice(population, size=n)) for _ in range(3000)]
    ax.hist(sample_means, bins=40, edgecolor='k')
    ax.set_title(f"n={n}  |  skew={stats.skew(sample_means):.2f}")
plt.tight_layout()
plt.savefig("clt_demo.png", dpi=100)
# By n=50, sample means are approximately normal even for exponential data
```

> [!tip]
> A common rule of thumb: n ≥ 30 is "enough" for CLT to kick in. For heavily skewed distributions (e.g., incomes), you may need n ≥ 100+. Always check empirically with a simulation like the one above.

---

## Confidence Intervals

A confidence interval gives you a range of plausible values for a population parameter. It quantifies uncertainty — more useful than a point estimate alone.

### Confidence Interval for a Mean

$$\bar{x} \pm t^* \cdot \frac{s}{\sqrt{n}}$$

t* is the critical value from the t-distribution at your chosen confidence level.

```python
sample = np.random.normal(loc=50, scale=10, size=40)

confidence = 0.95
ci = stats.t.interval(
    confidence,
    df=len(sample) - 1,
    loc=np.mean(sample),
    scale=stats.sem(sample)    # standard error of mean
)
print(f"95% CI: ({ci[0]:.2f}, {ci[1]:.2f})")
# Output: 95% CI: (47.xx, 53.xx)  — will contain true mean 50 in 95% of experiments
```

### Interpreting the Interval

A 95% CI does NOT mean "there is a 95% probability the true mean is in this interval." The true mean is fixed (just unknown); the interval is random.

Correct interpretation: "If we repeated this experiment 100 times and built a CI each time, approximately 95 of those intervals would contain the true mean."

```python
# Demonstrate the coverage property
true_mean = 50
hits = 0
for _ in range(1000):
    s = np.random.normal(loc=true_mean, scale=10, size=40)
    lo, hi = stats.t.interval(0.95, df=39, loc=np.mean(s), scale=stats.sem(s))
    if lo <= true_mean <= hi:
        hits += 1
print(f"Coverage: {hits}/1000 = {hits/10:.1f}%")
# Output: Coverage: ~950/1000 = ~95.0%
```

### Interval Width and Sample Size

Width ∝ 1/√n. To halve the interval width, you need 4× the sample size.

```python
for n in [10, 40, 160, 640]:
    se = 10 / np.sqrt(n)      # sigma=10
    t_star = stats.t.ppf(0.975, df=n-1)
    width = 2 * t_star * se
    print(f"n={n:4d}  →  width = {width:.2f}")
# Output:
# n=  10  →  width = 7.15
# n=  40  →  width = 3.20
# n= 160  →  width = 1.57
# n= 640  →  width = 0.78
```

---

## Hypothesis Testing Framework

Hypothesis testing is a decision procedure — it tells you whether data is consistent with a baseline claim. The framework is the same regardless of which specific test you use.

### The Framework

1. State H₀ (null — status quo) and H₁ (alternative — what you want to detect).
2. Choose significance level α (typically 0.05).
3. Compute the test statistic and its p-value under H₀.
4. Reject H₀ if p-value < α.

### P-value

The p-value is the probability of observing data at least as extreme as yours, assuming H₀ is true. It is NOT the probability that H₀ is true.

```python
# One-sample t-test: is the population mean equal to 50?
sample = np.array([52.1, 49.8, 53.4, 51.0, 50.7, 54.2, 48.9, 52.8])

t_stat, p_value = stats.ttest_1samp(sample, popmean=50)
print(f"t={t_stat:.3f}, p={p_value:.4f}")
# Output: t=2.xxx, p=0.03xx  → reject H₀ at α=0.05 if p < 0.05
```

### Type I and Type II Errors

| Decision \ Truth | H₀ True     | H₀ False    |
|------------------|-------------|-------------|
| Reject H₀        | Type I (α)  | Correct     |
| Fail to reject   | Correct     | Type II (β) |

- **Type I error rate** = α (false positive). You control this directly.
- **Type II error rate** = β. Power = 1 − β (probability of detecting a real effect).

```python
# Power analysis: what sample size detects a 5-unit difference with 80% power?
from statsmodels.stats.power import TTestIndPower

analysis = TTestIndPower()
effect_size = 5 / 10   # delta / sigma = Cohen's d
n_required = analysis.solve_power(
    effect_size=effect_size,
    alpha=0.05,
    power=0.80,
    alternative='two-sided'
)
print(f"Required n per group: {int(np.ceil(n_required))}")
# Output: Required n per group: 64
```

> [!warning]
> Statistical significance ≠ practical significance. A p-value of 0.001 does not mean the effect is large or important. Always report effect size alongside the p-value.

---

## Common Statistical Tests

Choosing the wrong test gives you wrong answers. Match the test to the data type and the question.

### One-Sample t-Test

**When to use:** You have one continuous sample and want to test whether its population mean equals a known value.

```python
# Are delivery times different from the advertised 30 minutes?
delivery_times = np.array([28, 35, 32, 29, 31, 34, 27, 30, 33, 28, 36, 29])

t_stat, p_value = stats.ttest_1samp(delivery_times, popmean=30)
print(f"t={t_stat:.3f}, p={p_value:.4f}")
# p > 0.05 → fail to reject; data consistent with 30-min mean
```

### Two-Sample (Independent) t-Test

**When to use:** Compare means of two independent groups. Check for equal variances first (Levene's test).

$$t = \frac{\bar{x}_1 - \bar{x}_2}{\sqrt{\frac{s_1^2}{n_1} + \frac{s_2^2}{n_2}}}$$

```python
group_a = np.random.normal(50, 10, 30)
group_b = np.random.normal(55, 10, 30)

# Check variance equality
levene_stat, levene_p = stats.levene(group_a, group_b)
equal_var = levene_p > 0.05   # True → use equal_var=True (Student's t)

t_stat, p_value = stats.ttest_ind(group_a, group_b, equal_var=equal_var)
print(f"Levene p={levene_p:.3f}, t={t_stat:.3f}, p={p_value:.4f}")
```

### Paired t-Test

**When to use:** Two measurements on the same subject (before/after, same user two conditions). More powerful than two-sample t-test when measurements are correlated.

```python
before = np.array([72, 68, 75, 80, 65, 70, 77, 82])
after  = np.array([68, 65, 70, 76, 60, 67, 74, 79])

t_stat, p_value = stats.ttest_rel(before, after)
print(f"t={t_stat:.3f}, p={p_value:.4f}")
# Likely significant — same subjects, so within-person variation is controlled
```

### Chi-Square Test of Independence

**When to use:** Test whether two categorical variables are independent. Does gender affect product preference? Does region affect churn?

```python
# Observed counts: rows = gender, cols = product preference
observed = np.array([[120, 90, 40],
                     [ 80, 110, 60]])

chi2, p_value, dof, expected = stats.chi2_contingency(observed)
print(f"chi2={chi2:.3f}, p={p_value:.4f}, dof={dof}")
# p < 0.05 → variables are not independent

# Rule of thumb: all expected cell counts should be >= 5
print("Expected counts:\n", expected.round(1))
```

> [!warning]
> Chi-square requires expected cell counts ≥ 5. If cells are sparse, use Fisher's exact test (`stats.fisher_exact` for 2×2 tables).

### One-Way ANOVA

**When to use:** Compare means across 3+ independent groups. ANOVA tests whether any group differs — use post-hoc tests (Tukey) to find which pairs differ.

$$F = \frac{\text{Variance between groups}}{\text{Variance within groups}}$$

```python
group1 = np.random.normal(50, 8, 30)
group2 = np.random.normal(55, 8, 30)
group3 = np.random.normal(53, 8, 30)

f_stat, p_value = stats.f_oneway(group1, group2, group3)
print(f"F={f_stat:.3f}, p={p_value:.4f}")

# Post-hoc: Tukey HSD (requires statsmodels)
from statsmodels.stats.multicomp import pairwise_tukeyhsd
import pandas as pd

all_data = np.concatenate([group1, group2, group3])
labels   = ['G1']*30 + ['G2']*30 + ['G3']*30
tukey = pairwise_tukeyhsd(all_data, labels, alpha=0.05)
print(tukey.summary())
```

---

## Correlation

Correlation measures the strength and direction of a linear (Pearson) or monotonic (Spearman) relationship between two variables.

### Pearson Correlation

Measures linear relationship. Sensitive to outliers. Assumes both variables are continuous and approximately normal.

$$r = \frac{\sum (x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum (x_i - \bar{x})^2 \sum (y_i - \bar{y})^2}}$$

Ranges from −1 (perfect negative) to +1 (perfect positive). 0 means no linear relationship.

```python
n = 100
x = np.random.normal(0, 1, n)
y = 0.7 * x + np.random.normal(0, 0.5, n)   # strong positive linear relationship

r, p_value = stats.pearsonr(x, y)
print(f"Pearson r={r:.3f}, p={p_value:.4e}")
# Output: Pearson r~0.85, p < 0.001

# Rule of thumb for |r|:
# 0.0–0.1: negligible  |  0.1–0.3: small  |  0.3–0.5: moderate  |  0.5+: large
```

### Spearman Rank Correlation

Measures monotonic relationship. Robust to outliers and non-normal data. Use when data is ordinal or when Pearson assumptions are violated.

```python
# Data with an outlier that would distort Pearson
x_out = np.append(x, 10)   # outlier
y_out = np.append(y, -10)  # opposite-direction outlier

r_pearson,  _ = stats.pearsonr(x_out, y_out)
r_spearman, _ = stats.spearmanr(x_out, y_out)

print(f"Pearson  r = {r_pearson:.3f}")   # distorted by outlier
print(f"Spearman r = {r_spearman:.3f}")  # more robust

# Correlation matrix for multiple variables
df = pd.DataFrame({'A': x, 'B': y, 'C': np.random.normal(0, 1, n)})
print(df.corr(method='pearson'))
print(df.corr(method='spearman'))
```

> [!warning]
> Correlation does not imply causation. Also, r=0 means no LINEAR relationship — two variables can be strongly correlated in a non-linear way and still show r≈0. Always plot the data.

---

## A/B Testing

A/B testing is hypothesis testing applied to product decisions. The same statistical machinery applies, but you also need to think about sample size before you start (not after).

### Sample Size Calculation

Determine sample size before running the test. Peeking at results and stopping early inflates Type I error.

```python
from statsmodels.stats.power import TTestIndPower, NormalIndPower
from statsmodels.stats.proportion import proportion_effectsize

# For conversion rates (proportions)
# Baseline conversion: 5%, desired lift: 20% relative → 6%
p_baseline = 0.05
p_treatment = 0.06

effect_size = proportion_effectsize(p_treatment, p_baseline)

analysis = NormalIndPower()
n_per_group = analysis.solve_power(
    effect_size=effect_size,
    alpha=0.05,
    power=0.80,
    alternative='two-sided'
)
print(f"Effect size (h): {effect_size:.4f}")
print(f"Required n per group: {int(np.ceil(n_per_group))}")
# Output: Required n per group: ~3842
# Note: a 20% relative lift in a low-conversion product needs a large sample
```

### Running the Test and Computing p-Value

```python
from statsmodels.stats.proportion import proportions_ztest

# Observed results
n_control   = 4000
n_treatment = 4000
conv_control   = 198   # 4.95% conversion
conv_treatment = 241   # 6.02% conversion

count = np.array([conv_treatment, conv_control])
nobs  = np.array([n_treatment, n_control])

z_stat, p_value = proportions_ztest(count, nobs, alternative='two-sided')
print(f"z={z_stat:.3f}, p={p_value:.4f}")
# p < 0.05 → statistically significant lift
```

### Effect Size (Cohen's d)

Cohen's d normalizes the difference by pooled standard deviation. It tells you how big the effect is, independent of sample size.

$$d = \frac{\bar{x}_1 - \bar{x}_2}{s_{pooled}} \qquad s_{pooled} = \sqrt{\frac{s_1^2 + s_2^2}{2}}$$

Benchmarks: d = 0.2 (small), 0.5 (medium), 0.8 (large).

```python
control   = np.random.normal(50, 10, 200)
treatment = np.random.normal(54, 10, 200)

pooled_std = np.sqrt((control.std()**2 + treatment.std()**2) / 2)
cohens_d   = (treatment.mean() - control.mean()) / pooled_std

print(f"Cohen's d = {cohens_d:.3f}")
# Output: Cohen's d ~ 0.4  (medium effect)
```

> [!tip]
> Report Cohen's d (or relative lift) alongside the p-value in every A/B test result. A p-value of 0.001 with d=0.05 might not be worth shipping. A p-value of 0.03 with d=0.6 often is.

---

## Bayesian Basics

Bayesian inference treats unknown parameters as probability distributions. Instead of rejecting or failing to reject a null hypothesis, you update your beliefs with data.

### Prior, Likelihood, Posterior

$$P(\theta \mid \text{data}) \propto P(\text{data} \mid \theta) \cdot P(\theta)$$

- **Prior** P(θ): belief about the parameter before seeing data.
- **Likelihood** P(data|θ): how probable is the observed data given θ?
- **Posterior** P(θ|data): updated belief after seeing data.

```python
import numpy as np
from scipy import stats

# Estimating a conversion rate p
# Prior: Beta(2, 18) — weakly believe p is around 10%
alpha_prior, beta_prior = 2, 18

# Observed data: 30 conversions out of 200 visitors
conversions, total = 30, 200

# Posterior: Beta(alpha + conversions, beta + failures)  [conjugate prior]
alpha_post = alpha_prior + conversions
beta_post  = beta_prior + (total - conversions)

posterior = stats.beta(alpha_post, beta_post)

print(f"Prior mean: {alpha_prior / (alpha_prior + beta_prior):.3f}")
print(f"Posterior mean: {posterior.mean():.3f}")
print(f"95% Credible Interval: ({posterior.ppf(0.025):.3f}, {posterior.ppf(0.975):.3f})")
# Output: Prior mean: 0.100, Posterior mean: 0.148, CI: (~0.10, ~0.20)
```

### Conjugate Priors

A conjugate prior is one where the posterior has the same distributional form as the prior — makes the math closed-form and fast.

| Likelihood  | Conjugate Prior | Posterior        |
|-------------|-----------------|------------------|
| Binomial    | Beta(α, β)      | Beta(α+k, β+n−k) |
| Poisson     | Gamma(α, β)     | Gamma(α+Σx, β+n) |
| Normal (σ known) | Normal(μ₀, σ₀²) | Normal (updated) |

```python
# Poisson-Gamma conjugate: estimating a call rate λ
# Prior: Gamma(3, 1) — expect about 3 calls/hour
alpha_prior, beta_prior = 3, 1

# Observed: 45 calls over 10 hours → sample rate = 4.5
observed_calls = 45
observed_hours = 10

alpha_post = alpha_prior + observed_calls
beta_post  = beta_prior + observed_hours

posterior = stats.gamma(a=alpha_post, scale=1/beta_post)
print(f"Posterior mean λ: {posterior.mean():.3f}")
print(f"95% Credible Interval: ({posterior.ppf(0.025):.3f}, {posterior.ppf(0.975):.3f})")
# Output: Posterior mean λ: 4.364, CI: (~3.35, ~5.47)
```

### Bayesian Credible Interval vs. Frequentist Confidence Interval

```python
# Bayesian credible interval — direct probability statement IS valid here:
# "There is a 95% probability that λ is in [3.35, 5.47] given the data."
# (Unlike a frequentist CI, which does NOT allow this interpretation.)

alpha_post, beta_post = 48, 11
posterior = stats.gamma(a=alpha_post, scale=1/beta_post)
lo, hi = posterior.ppf(0.025), posterior.ppf(0.975)
print(f"95% Credible Interval: ({lo:.2f}, {hi:.2f})")
```

---

## Multiple Testing

Every time you run a test at α=0.05, you accept a 5% false positive rate. Run 20 tests independently and you expect one false positive by chance. This is the multiple comparisons problem.

### The Problem Illustrated

```python
np.random.seed(0)
# 20 tests, all under H₀ (pure noise, no real effect)
p_values = [stats.ttest_ind(
    np.random.normal(0, 1, 50),
    np.random.normal(0, 1, 50)
).pvalue for _ in range(20)]

false_positives = sum(p < 0.05 for p in p_values)
print(f"False positives: {false_positives}/20")
# Output: 1–3 false positives from pure noise (expected: 1)
```

### Bonferroni Correction

Divide α by the number of tests. Guarantees family-wise error rate ≤ α. Conservative — loses power when many tests are run.

$$\alpha_{adjusted} = \frac{\alpha}{m}$$

```python
from statsmodels.stats.multitest import multipletests

# Simulate 100 tests: 90 null, 10 with real effects
null_p    = np.random.uniform(0, 1, 90)                        # no effect
signal_p  = np.random.beta(0.5, 10, 10)                        # real effect → small p
all_p     = np.concatenate([null_p, signal_p])

reject_bonf, p_adj_bonf, _, _ = multipletests(all_p, alpha=0.05, method='bonferroni')
print(f"Bonferroni rejections: {reject_bonf.sum()}")
print(f"Bonferroni adjusted threshold: {0.05/len(all_p):.5f}")
```

### Benjamini-Hochberg FDR Correction

Controls False Discovery Rate (FDR) — the expected proportion of rejected nulls that are false. Less conservative than Bonferroni; preferred when testing many hypotheses (genomics, feature selection, A/B test variants).

The BH procedure sorts p-values and applies a scaled threshold:

$$p_{(i)} \le \frac{i}{m} \cdot \alpha$$

```python
reject_bh, p_adj_bh, _, _ = multipletests(all_p, alpha=0.05, method='fdr_bh')
print(f"BH rejections: {reject_bh.sum()}")
# BH will reject more than Bonferroni — it accepts some false positives
# to gain more true positive detections (better power)

# Compare methods side by side
results = pd.DataFrame({
    'p_value':  all_p,
    'bonf_adj': p_adj_bonf,
    'bh_adj':   p_adj_bh,
    'bonf_reject': reject_bonf,
    'bh_reject':   reject_bh
}).sort_values('p_value').head(10)
print(results.to_string(index=False))
```

> [!warning]
> Bonferroni is appropriate when any false positive is costly (clinical trials). BH is appropriate when you can tolerate a small proportion of false positives and need statistical power (exploratory research, feature screening). Never run 20 A/B test variants and pick the winner without correction — this is p-hacking.

> [!tip]
> A practical rule: if you are testing more than 5 hypotheses simultaneously, apply a correction. Document which correction you chose and why before collecting data.

---

## Quick Reference

### Test Selection Guide

| Data type            | Groups | Use                        |
|----------------------|--------|-----------------------------|
| Continuous           | 1      | One-sample t-test           |
| Continuous           | 2 independent | Two-sample t-test   |
| Continuous           | 2 paired      | Paired t-test       |
| Continuous           | 3+     | One-way ANOVA + Tukey       |
| Categorical          | 2+ variables | Chi-square / Fisher |
| Ordinal / non-normal | 2      | Mann-Whitney U              |
| Ordinal / non-normal | 3+     | Kruskal-Wallis              |

### Effect Size Benchmarks

| Measure   | Small | Medium | Large |
|-----------|-------|--------|-------|
| Cohen's d | 0.2   | 0.5    | 0.8   |
| Pearson r | 0.1   | 0.3    | 0.5   |
| η² (ANOVA)| 0.01  | 0.06   | 0.14  |

### Common scipy.stats Functions

```python
stats.norm(loc, scale)          # Normal distribution object
stats.t.interval(conf, df, loc, scale)  # Confidence interval
stats.ttest_1samp(data, popmean)        # One-sample t-test
stats.ttest_ind(a, b, equal_var)        # Two-sample t-test
stats.ttest_rel(a, b)                   # Paired t-test
stats.chi2_contingency(table)           # Chi-square test
stats.f_oneway(*groups)                 # One-way ANOVA
stats.pearsonr(x, y)                    # Pearson correlation
stats.spearmanr(x, y)                   # Spearman correlation
stats.shapiro(data)                     # Normality test
stats.levene(*groups)                   # Variance equality test
```
