# Statistics Cheat Sheet

A dense reference for Day 4 concepts. Every entry has the formula, the Python one-liner, and when to use it.

---

## Central Tendency

| Measure | Formula | Python | Use When |
|---------|---------|--------|----------|
| Mean | `Σx / n` | `s.mean()` | Symmetric data, no extreme outliers |
| Median | Middle value (sorted) | `s.median()` | Skewed data, outliers present |
| Mode | Most frequent value | `s.mode()[0]` | Categorical data, finding the peak |
| Weighted mean | `Σ(w·x) / Σw` | `np.average(arr, weights=w)` | Values have different importance |

```python
import pandas as pd
import numpy as np

s = pd.Series([22, 25, 29, 35, 40, 95])

print(s.mean())              # 41.0 — pulled up by 95
print(s.median())            # 32.0 — more representative
print(s.mode()[0])           # all unique here; .mode() returns all tied peaks
print(np.average(s, weights=[1,1,1,1,1,3]))  # 3x weight on the last value
```

---

## Diagnosing Skew Quickly

```python
gap = s.mean() - s.median()

# Gap > 0   → right-skewed (long right tail — income, prices)
# Gap < 0   → left-skewed  (long left tail  — easy exam scores)
# Gap ≈ 0   → roughly symmetric

print(s.skew())   # positive = right, negative = left
```

| `skew` value | Shape | Prefer |
|-------------|-------|--------|
| `< -1` or `> 1` | Strongly skewed | Median |
| `-1` to `-0.5` or `0.5` to `1` | Moderately skewed | Median |
| `-0.5` to `0.5` | Roughly symmetric | Mean |

---

## Spread

| Measure | Formula | Python | Use When |
|---------|---------|--------|----------|
| Range | `max - min` | `s.max() - s.min()` | Quick sanity check |
| Variance (sample) | `Σ(x - x̄)² / (n-1)` | `s.var()` | Squared units; input to std |
| Variance (population) | `Σ(x - μ)² / n` | `s.var(ddof=0)` | Full population data |
| Std dev (sample) | `√variance` | `s.std()` | Spread in original units |
| Std dev (population) | `√variance` | `s.std(ddof=0)` | Full population data |
| IQR | `Q3 - Q1` | `s.quantile(.75) - s.quantile(.25)` | Skewed data, outlier-robust |
| Coefficient of variation | `(std / mean) × 100` | `(s.std() / s.mean()) * 100` | Comparing spread across scales |

```python
s = pd.Series([60, 70, 75, 80, 95])

print(f"Range:      {s.max() - s.min()}")
print(f"Variance:   {s.var():.2f}")        # sample (ddof=1 default)
print(f"Std:        {s.std():.2f}")
print(f"IQR:        {s.quantile(.75) - s.quantile(.25):.2f}")
print(f"CV:         {(s.std() / s.mean()) * 100:.1f}%")
```

---

## Pandas vs NumPy — Default Behavior

| Operation | pandas | NumPy |
|-----------|--------|-------|
| Variance default | `ddof=1` (sample) | `ddof=0` (population) |
| Std dev default | `ddof=1` (sample) | `ddof=0` (population) |
| NaN handling | skips NaN by default | propagates NaN by default |

```python
arr = np.array([10, 20, 30, 40, 50])
s   = pd.Series(arr)

print(np.std(arr))         # 14.14 — population std (ddof=0)
print(s.std())             # 15.81 — sample std (ddof=1)
print(np.std(arr, ddof=1)) # 15.81 — force sample in NumPy
```

---

## Outlier Detection — IQR Method

```python
q1  = s.quantile(0.25)
q3  = s.quantile(0.75)
iqr = q3 - q1

lower = q1 - 1.5 * iqr
upper = q3 + 1.5 * iqr

outliers    = s[(s < lower) | (s > upper)]
clean_data  = s[(s >= lower) & (s <= upper)]
```

This is what a box plot uses to draw its whiskers and flag outliers.

---

## Outlier Detection — Z-Score Method

```python
from scipy import stats

z_scores = np.abs(stats.zscore(s))
outliers = s[z_scores > 3]   # values more than 3 std from the mean
```

> [!warning] Use Z-scores only for roughly normal data. Use IQR for skewed data. Mixing them gives wrong results.

---

## Probability Rules

| Rule | Formula | Code |
|------|---------|------|
| Complement | `P(not A) = 1 - P(A)` | `1 - p_a` |
| AND (independent) | `P(A ∩ B) = P(A) × P(B)` | `p_a * p_b` |
| OR (general) | `P(A ∪ B) = P(A) + P(B) - P(A ∩ B)` | `p_a + p_b - p_ab` |
| Conditional | `P(A\|B) = P(A ∩ B) / P(B)` | `p_ab / p_b` |
| Bayes | `P(A\|B) = P(B\|A) × P(A) / P(B)` | see below |

```python
# Bayes' theorem
p_a             = 0.01    # prior: P(disease)
p_b_given_a     = 0.95    # likelihood: P(positive | disease)
p_b_given_not_a = 0.05    # false positive rate

p_b = p_b_given_a * p_a + p_b_given_not_a * (1 - p_a)
p_a_given_b = (p_b_given_a * p_a) / p_b

print(f"P(disease | positive test) = {p_a_given_b:.4f}")  # 0.1610
```

---

## Distributions Quick Reference

| Distribution | Parameters | Mean | Variance | Use Case |
|-------------|-----------|------|----------|----------|
| Normal | μ, σ | μ | σ² | Symmetric measurements |
| Binomial | n, p | np | np(1-p) | Count of successes in n trials |
| Poisson | λ | λ | λ | Events per unit time/area |
| Uniform | a, b | (a+b)/2 | (b-a)²/12 | All outcomes equally likely |

### Generate and query distributions with scipy.stats

```python
from scipy import stats

# Normal
n = stats.norm(loc=70, scale=10)       # mean=70, std=10
print(n.pdf(70))                        # density at 70
print(n.cdf(80))                        # P(X ≤ 80) ≈ 0.841
print(n.ppf(0.95))                      # 95th percentile ≈ 86.4

# Binomial
b = stats.binom(n=100, p=0.3)
print(b.pmf(30))                        # P(X = 30)
print(b.cdf(25))                        # P(X ≤ 25)

# Poisson
p = stats.poisson(mu=5)
print(p.pmf(5))                         # P(X = 5)
print(1 - p.cdf(9))                     # P(X > 9)
```

---

## The 68-95-99.7 Rule (Normal Only)

```python
mu, sigma = 100, 15

print(f"68%: {mu - sigma:.0f} to {mu + sigma:.0f}")     # 85 to 115
print(f"95%: {mu - 2*sigma:.0f} to {mu + 2*sigma:.0f}") # 70 to 130
print(f"99.7%: {mu - 3*sigma:.0f} to {mu + 3*sigma:.0f}") # 55 to 145
```

---

## Z-Score

```python
# Standardize a single value
z = (value - mean) / std

# Standardize a full series
z_series = (s - s.mean()) / s.std()

# Look up the probability (for normal data)
from scipy.stats import norm
p_below = norm.cdf(z)   # P(X ≤ value)
p_above = 1 - p_below   # P(X > value)
```

---

## Comprehensive Data Summary

```python
def full_summary(s, name="Series"):
    q1, q3 = s.quantile(0.25), s.quantile(0.75)
    print(f"\n=== {name} ===")
    print(f"  Count:   {s.count()}   | Missing: {s.isnull().sum()}")
    print(f"  Mean:    {s.mean():.3f}  | Median: {s.median():.3f}")
    print(f"  Std:     {s.std():.3f}  | IQR:    {q3 - q1:.3f}")
    print(f"  Min:     {s.min():.3f}  | Max:    {s.max():.3f}")
    print(f"  Skew:    {s.skew():.3f}  | Kurt:   {s.kurtosis():.3f}")

full_summary(pd.Series([22, 25, 30, 35, 200]), "Customer Ages")
```

---

## Choosing Mean vs Median — Decision Rule

```python
def recommend_central_tendency(s):
    skew = abs(s.skew())
    if skew > 1:
        return "Use MEDIAN — strongly skewed data"
    elif skew > 0.5:
        return "Use MEDIAN — moderately skewed; also report mean for context"
    else:
        return "Use MEAN — roughly symmetric data"

print(recommend_central_tendency(pd.Series([100, 120, 130, 125, 1000])))
# Output: Use MEDIAN — strongly skewed data
```

---

[[04-distributions|Back: Distributions]] | [[06-practice-questions|Next: Practice Questions]]
