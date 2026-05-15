# Mean, Median, and Mode

Salary surveys regularly report an "average" salary that almost nobody earns. This is not an accident — it is a deliberate (or careless) choice of the wrong summary statistic. Understanding when the mean lies to you is one of the first skills that separates a data scientist from someone who just runs `.mean()` on everything.

## Learning Objectives

- Compute mean, median, and mode in Python using NumPy and pandas
- Explain why the mean is misleading for skewed data
- Choose the correct measure of central tendency for a given dataset
- Understand weighted mean and when to use it
- Recognize how outliers shift the mean while leaving the median stable

---

## What Is Central Tendency?

Central tendency answers the question: "What is the typical value in this dataset?" There are three common answers, and they give different results depending on the shape of your data.

| Measure | Definition | Best when |
|---------|-----------|-----------|
| Mean | Sum divided by count | Data is symmetric, no extreme outliers |
| Median | Middle value when sorted | Data is skewed or has outliers |
| Mode | Most frequent value | Categorical data, or finding the peak of a distribution |

The right choice is not a matter of preference — it follows from the shape of your data.

---

## Mean — The Arithmetic Average

The mean adds everything up and divides by the number of values.

> [!info] Formula
> `mean = (x₁ + x₂ + ... + xₙ) / n`
> In NumPy: `np.mean(arr)` | In pandas: `series.mean()`

```python
import numpy as np
import pandas as pd

salaries = pd.Series([45000, 48000, 52000, 55000, 60000, 58000, 950000])

print("Mean:  ", salaries.mean())
print("Median:", salaries.median())
# Output:
# Mean:   181142.857...
# Median: 55000.0
```

One CEO salary of 950,000 pulls the mean to 181,000 — a number that describes nobody in this dataset. The median of 55,000 is what a typical employee actually earns.

> [!warning] The mean lies on skewed data
> Income, house prices, customer spend, website response times — these are all right-skewed. Reporting the mean for these variables without also checking the median is a common mistake. In any EDA, always compute both.

### Why the Mean Still Matters

The mean is not wrong — it is the right tool for symmetric data. It is also the foundation of many statistical methods (regression, t-tests) and is what your ML model minimizes when you use mean squared error. Know when to trust it.

```python
# Symmetric data — mean and median are close
exam_scores = pd.Series([68, 72, 74, 75, 76, 77, 79, 81, 83, 85])

print("Mean:  ", exam_scores.mean())    # Output: 77.0
print("Median:", exam_scores.median())  # Output: 76.5
# Close to each other — either is fine here
```

---

## Median — The Middle Value

Sort your data. The median is the value that sits at the exact center. Half the values are below it, half above.

```python
# Odd number of values — middle element
odd_series = pd.Series([10, 20, 30, 40, 50])
print(odd_series.median())  # Output: 30.0

# Even number of values — average of two middle elements
even_series = pd.Series([10, 20, 30, 40])
print(even_series.median())  # Output: 25.0
```

> [!tip] The classic skew test
> Compute `mean - median`. If the result is large and positive, your data is right-skewed. Large and negative means left-skewed. Near zero means roughly symmetric. This one comparison tells you which summary statistic to report.

```python
housing_prices = pd.Series([250000, 275000, 290000, 310000, 320000, 340000, 2500000])

gap = housing_prices.mean() - housing_prices.median()
print(f"Mean:   {housing_prices.mean():,.0f}")    # Output: 612,142
print(f"Median: {housing_prices.median():,.0f}")  # Output: 310,000
print(f"Gap:    {gap:,.0f}")                      # Output: 302,142 — heavily right-skewed
```

---

## Mode — The Most Common Value

The mode is the value that appears most often. It is the only measure of central tendency that works for categorical data.

```python
customer_cities = pd.Series(["Mumbai", "Delhi", "Mumbai", "Bangalore", "Mumbai", "Delhi"])

print(customer_cities.mode())
# Output:
# 0    Mumbai
# dtype: object
```

For numeric data, mode finds the peak of the distribution — useful when you want to know the most popular price point or the most common transaction size.

```python
purchase_amounts = pd.Series([99, 149, 99, 199, 99, 249, 149, 99])
print(purchase_amounts.mode())
# Output:
# 0    99
# dtype: int64
```

> [!warning] Multiple modes
> `series.mode()` returns all modes if there are ties. Always check `len(series.mode())` before assuming a single peak. A dataset with two modes (bimodal) often signals two distinct subgroups — worth investigating.

```python
bimodal = pd.Series([1, 1, 2, 3, 4, 5, 5])
print(bimodal.mode())
# Output:
# 0    1
# 1    5
# dtype: int64
# Two modes — suggests two clusters in the data
```

---

## Weighted Mean

The standard mean treats every value equally. The weighted mean lets you give some values more importance — used in GPA calculations, index construction, and recommendation systems.

> [!info] Weighted Mean Formula
> `weighted_mean = sum(value × weight) / sum(weights)`

```python
# A student's grades across courses with different credit hours
grades  = np.array([85, 90, 78, 92])
credits = np.array([4,  3,  4,  2])

weighted_mean = np.average(grades, weights=credits)
simple_mean   = np.mean(grades)

print(f"Simple mean:   {simple_mean:.2f}")    # Output: 86.25
print(f"Weighted mean: {weighted_mean:.2f}")  # Output: 84.69
# The 4-credit course with 78 drags the weighted mean below the simple mean
```

---

## Real-World Example: Salary Data

This is the canonical example. Run this and internalize the numbers.

```python
import pandas as pd
import numpy as np

np.random.seed(42)

# Simulate a small company: 20 employees + 1 founder
employee_salaries = np.random.normal(loc=60000, scale=8000, size=20)
founder_salary    = np.array([2_500_000])

all_salaries = pd.Series(np.concatenate([employee_salaries, founder_salary]))

print(f"Mean salary:   ${all_salaries.mean():>12,.0f}")
print(f"Median salary: ${all_salaries.median():>12,.0f}")
print(f"Mode salary:   ${all_salaries.mode()[0]:>12,.0f}")  # Will be approximate
print(f"Min:           ${all_salaries.min():>12,.0f}")
print(f"Max:           ${all_salaries.max():>12,.0f}")

# Output (approximate):
# Mean salary:   $    179,412
# Median salary: $     59,711
# Mode salary:   ~  varies
# Min:           $     45,083
# Max:           $  2,500,000
```

The mean of $179K describes none of the 20 employees and misrepresents the company's compensation. The median of $59K is what most people actually earn. This gap is why public salary discussions almost always use median — and why some companies prefer to report mean.

---

## Computing All Three in a DataFrame

```python
df = pd.DataFrame({
    "employee_id":  range(1, 8),
    "department":   ["Engineering", "Engineering", "Sales", "Sales", "HR", "Engineering", "Sales"],
    "annual_salary": [70000, 85000, 55000, 60000, 48000, 95000, 52000]
})

# Per-column summaries
print(df["annual_salary"].mean())    # Output: 66428.57
print(df["annual_salary"].median())  # Output: 60000.0
print(df["department"].mode()[0])    # Output: Engineering

# Group-level summaries — more useful in practice
print(df.groupby("department")["annual_salary"].agg(["mean", "median"]))
# Output:
#               mean  median
# department
# Engineering  83333   85000
# HR           48000   48000
# Sales        55667   55000
```

> [!tip] Always group before summarizing
> A company-wide average salary hides the fact that engineering earns 70% more than HR. Always break your summary statistics down by relevant groups — you will find the real story there.

---

## Missing Values

Pandas skips `NaN` by default when computing these statistics. This is usually what you want, but be aware.

```python
data_with_gaps = pd.Series([100, 200, np.nan, 400, 500])

print(data_with_gaps.mean())    # Output: 300.0 (ignores NaN, computes over 4 values)
print(data_with_gaps.count())   # Output: 4 (not 5)
```

> [!warning] NaN changes your denominator silently
> If you have 30% missing values and compute the mean, you are computing the mean of the 70% that remain. Whether that is a valid estimate depends on WHY the values are missing. Never report a mean without checking `.isnull().sum()` first.

---

> [!success] Key Takeaways
> - Mean is sensitive to outliers. Use it on symmetric data.
> - Median is robust. Use it when data is skewed or has outliers.
> - Mode works on categories and tells you the most popular value.
> - The gap between mean and median is your first diagnostic for skew.
> - Weighted mean matters when values have different levels of importance.
> - Always check for missing values before computing any summary statistic.

---

[[00-agenda|Back to Day 4 Agenda]] | [[02-variance-and-std|Next: Variance and Standard Deviation]]
