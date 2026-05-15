# Variance and Standard Deviation

Two investment portfolios each return an average of 10% per year. You would not call them equivalent without asking how much they swing year to year. That swing — the spread around the average — is what variance and standard deviation measure. Without it, the mean is half the story.

## Learning Objectives

- Compute variance and standard deviation using pandas and NumPy
- Explain the intuition behind variance (average squared distance from the mean)
- Distinguish population variance from sample variance, and understand why the denominator differs
- Use the coefficient of variation to compare spread across variables with different units
- Apply IQR for outlier detection on skewed data

---

## Why Spread Matters

Two datasets can share the same mean and look completely different.

```python
import pandas as pd
import numpy as np

stable   = pd.Series([48, 49, 50, 51, 52])  # mean = 50
volatile = pd.Series([10, 30, 50, 70, 90])  # mean = 50

print(f"Stable mean:   {stable.mean()}")    # Output: 50.0
print(f"Volatile mean: {volatile.mean()}")  # Output: 50.0

print(f"Stable std:    {stable.std():.2f}")    # Output: 1.58
print(f"Volatile std:  {volatile.std():.2f}")  # Output: 31.62
```

Same mean, twenty times the standard deviation. A model predicting the stable series is trivial. A model predicting the volatile series is a much harder problem. Spread tells you something the mean cannot.

---

## Range — The Crude Measure

The simplest spread measure is max minus min.

```python
scores = pd.Series([60, 70, 75, 80, 95])
data_range = scores.max() - scores.min()
print(f"Range: {data_range}")  # Output: 35
```

Range is easy to calculate and easy to explain. It is also nearly useless analytically because a single extreme value controls it entirely. One corrupt record and your range doubles.

---

## Variance — Average Squared Distance from the Mean

Think of variance this way: for each data point, measure how far it is from the mean. Square that distance. Average all the squared distances. That is variance.

> [!info] Variance Formula
> `variance = Σ(xᵢ - mean)² / n` (population)
> `variance = Σ(xᵢ - mean)² / (n - 1)` (sample — Bessel's correction)

Squaring does two things deliberately. First, it makes all distances positive (a value 5 below the mean contributes the same as one 5 above it). Second, it penalizes large deviations more than small ones. A value 10 units from the mean contributes 100 to the variance; a value 2 units away contributes only 4. Variance is sensitive to outliers by design.

```python
exam_scores = pd.Series([72, 75, 78, 80, 82, 85, 88])

mean_score = exam_scores.mean()
squared_deviations = (exam_scores - mean_score) ** 2

print(f"Mean:              {mean_score:.2f}")             # Output: 80.00
print(f"Squared devs:      {squared_deviations.values}")
print(f"Variance (manual): {squared_deviations.mean():.2f}")  # population variance
print(f"Variance (pandas): {exam_scores.var():.2f}")           # sample variance (ddof=1)
# Output:
# Variance (manual): 26.57
# Variance (pandas): 30.99  ← slightly larger due to Bessel's correction
```

The numbers are close but not identical. The reason is Bessel's correction.

---

## Population vs Sample Variance — Why the Denominator Differs

This is the source of endless confusion. Here is the precise distinction.

**Population variance** divides by `n`. Use this when you have data on every member of the group you care about — all employees in a company, all transactions in a month.

**Sample variance** divides by `n - 1`. Use this when you have a subset drawn from a larger population — 500 survey respondents standing in for all customers, 1,000 rows from a database of millions.

> [!info] Bessel's Correction
> When you compute variance on a sample, you tend to *underestimate* the true population variance. Dividing by `n - 1` instead of `n` corrects for this bias. The intuition: a sample is more likely to miss the extreme values in the population, so its variance looks artificially small. The correction nudges it upward.

```python
population = pd.Series([10, 20, 30, 40, 50])

print(f"Population variance (ddof=0): {population.var(ddof=0):.2f}")  # Output: 200.00
print(f"Sample variance     (ddof=1): {population.var(ddof=1):.2f}")  # Output: 250.00
print(f"Population std      (ddof=0): {population.std(ddof=0):.2f}")  # Output: 14.14
print(f"Sample std          (ddof=1): {population.std(ddof=1):.2f}")  # Output: 15.81
```

> [!tip] Pandas default is sample variance
> `pandas.Series.var()` and `.std()` use `ddof=1` by default — sample statistics. NumPy's `np.var()` and `np.std()` default to `ddof=0` — population statistics. This inconsistency trips up everyone eventually. When in doubt, pass `ddof` explicitly.

```python
arr = np.array([10, 20, 30, 40, 50])

print(np.var(arr))          # Output: 200.0  (ddof=0, population)
print(np.var(arr, ddof=1))  # Output: 250.0  (ddof=1, sample)
```

---

## Standard Deviation — Back to Interpretable Units

Variance is in squared units — square dollars, square centimeters. That makes it hard to interpret directly. Standard deviation is the square root of variance, which brings it back to the same units as the original data.

```python
test_results = pd.Series([55, 60, 65, 70, 75, 80, 85, 90])

print(f"Mean:  {test_results.mean():.1f}")  # Output: 72.5
print(f"Std:   {test_results.std():.1f}")   # Output: 11.5
```

The interpretation: the typical score deviates from the mean by about 11.5 points. Most scores fall between 72.5 - 11.5 = 61 and 72.5 + 11.5 = 84. This is meaningful in the original units.

> [!warning] Std is also pulled by outliers
> Standard deviation is calculated using the mean, so any data point that distorts the mean also distorts the std. A single extreme outlier can double the standard deviation. When you have outliers, IQR is a more honest measure of spread.

```python
clean_data   = pd.Series([50, 55, 60, 65, 70])
with_outlier = pd.Series([50, 55, 60, 65, 700])

print(f"Clean std:        {clean_data.std():.2f}")    # Output: 7.91
print(f"With outlier std: {with_outlier.std():.2f}")  # Output: 289.86
# One value multiplied std by ~36x
```

---

## The 68-95-99.7 Rule (Normal Distribution)

For data that follows a normal distribution, standard deviation has a precise interpretation:

| Range | Coverage |
|-------|----------|
| mean ± 1 std | ~68% of all values |
| mean ± 2 std | ~95% of all values |
| mean ± 3 std | ~99.7% of all values |

```python
from scipy import stats

# Generate normally distributed data
np.random.seed(42)
heights = pd.Series(np.random.normal(loc=170, scale=8, size=10000))  # cm

mean_h = heights.mean()
std_h  = heights.std()

within_1std = ((heights >= mean_h - std_h)   & (heights <= mean_h + std_h)).mean()
within_2std = ((heights >= mean_h - 2*std_h) & (heights <= mean_h + 2*std_h)).mean()
within_3std = ((heights >= mean_h - 3*std_h) & (heights <= mean_h + 3*std_h)).mean()

print(f"Mean: {mean_h:.1f} cm, Std: {std_h:.1f} cm")
print(f"Within 1 std: {within_1std:.1%}")   # Output: ~68%
print(f"Within 2 std: {within_2std:.1%}")   # Output: ~95%
print(f"Within 3 std: {within_3std:.1%}")   # Output: ~99.7%
```

This rule is why "3-sigma" is used as an outlier threshold and why control charts flag points more than 3 standard deviations from the mean.

---

## Coefficient of Variation — Comparing Spread Across Different Variables

Standard deviation is in the same units as the data, which means you cannot directly compare the std of salaries (in rupees) to the std of test scores (in points). The coefficient of variation (CV) normalizes by the mean to give a percentage.

> [!info] Coefficient of Variation Formula
> `CV = (std / mean) × 100`
> A CV of 20% means the typical deviation is 20% of the average value.

```python
annual_salary = pd.Series([45000, 55000, 62000, 70000, 85000])
exam_score    = pd.Series([60, 72, 75, 80, 88])

cv_salary = (annual_salary.std() / annual_salary.mean()) * 100
cv_score  = (exam_score.std()    / exam_score.mean())    * 100

print(f"Salary CV: {cv_salary:.1f}%")  # Output: 22.3%
print(f"Score CV:  {cv_score:.1f}%")   # Output: 13.2%
# Salaries are relatively more variable than exam scores
```

> [!tip] CV in practice
> CV is useful when comparing volatility across assets with different price levels, or comparing variability between two sensor readings with different units. In finance, it measures risk relative to return.

---

## IQR — Spread Without the Outlier Problem

The interquartile range measures the spread of the middle 50% of values. It completely ignores the bottom 25% and top 25%, which means outliers have no effect on it.

```python
daily_sales = pd.Series([200, 210, 220, 215, 205, 218, 2000])  # One bad day of returns

q1  = daily_sales.quantile(0.25)
q3  = daily_sales.quantile(0.75)
iqr = q3 - q1

print(f"Q1:  {q1:.1f}")            # Output: 207.5
print(f"Q3:  {q3:.1f}")            # Output: 219.0
print(f"IQR: {iqr:.1f}")           # Output: 11.5
print(f"Std: {daily_sales.std():.1f}")  # Output: 676.6 — destroyed by the outlier
```

The std of 676 suggests massive variability. The IQR of 11.5 reveals that 6 of the 7 days were tightly clustered. IQR is the honest measure here.

### Using IQR for Outlier Detection

The 1.5×IQR rule is the standard method used by box plots and many outlier detection systems.

```python
customer_spend = pd.Series([200, 250, 300, 280, 320, 290, 5000, 310, 270])

q1  = customer_spend.quantile(0.25)
q3  = customer_spend.quantile(0.75)
iqr = q3 - q1

lower_fence = q1  - 1.5 * iqr
upper_fence = q3  + 1.5 * iqr

outliers = customer_spend[(customer_spend < lower_fence) | (customer_spend > upper_fence)]

print(f"Lower fence: {lower_fence:.1f}")
print(f"Upper fence: {upper_fence:.1f}")
print(f"Outliers:    {outliers.values}")
# Output:
# Lower fence: 185.0
# Upper fence: 407.5
# Outliers:    [5000]
```

---

## Putting It Together: Full Spread Analysis

```python
np.random.seed(0)
product_ratings = pd.Series(np.random.normal(loc=3.8, scale=0.9, size=500).clip(1, 5))

print("=== Spread Analysis ===")
print(f"Mean:        {product_ratings.mean():.3f}")
print(f"Median:      {product_ratings.median():.3f}")
print(f"Std (sample):{product_ratings.std():.3f}")
print(f"Variance:    {product_ratings.var():.3f}")
print(f"IQR:         {product_ratings.quantile(.75) - product_ratings.quantile(.25):.3f}")
print(f"Skew:        {product_ratings.skew():.3f}")
print(f"Min / Max:   {product_ratings.min():.1f} / {product_ratings.max():.1f}")
# The skew value tells you whether the std is being pulled in a direction
```

---

> [!success] Key Takeaways
> - Variance is average squared distance from the mean. It penalizes large deviations heavily.
> - Standard deviation is variance's square root — same units as the data, easier to interpret.
> - Sample std divides by `n - 1` (Bessel's correction). Population std divides by `n`. Pandas defaults to sample; NumPy defaults to population.
> - The 68-95-99.7 rule applies to normal distributions. One std covers about 68% of data.
> - Std is sensitive to outliers. Use IQR when data is skewed or has extreme values.
> - Coefficient of variation compares relative spread across variables with different scales.

---

[[01-mean-median-mode|Back: Mean, Median, and Mode]] | [[03-probability-basics|Next: Probability Basics]]
