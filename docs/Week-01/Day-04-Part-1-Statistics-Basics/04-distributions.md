# 🔔 04 — Distributions
## Understanding Data Shapes

> [!info] Goal
> Learn common distributions and why data shape affects analysis.

---

## What is a Distribution?

A **distribution** describes how values are spread across possible outcomes.

It answers:

- Which values are common?
- Which values are rare?
- Is the data symmetric?
- Are there outliers?
- Is the data skewed?

---

## Visualizing Distribution

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

values = np.random.normal(loc=50, scale=10, size=1000)

plt.hist(values, bins=30, edgecolor="black")
plt.title("Distribution")
plt.xlabel("Value")
plt.ylabel("Frequency")
plt.show()
```

---

## Normal Distribution

The normal distribution is bell-shaped and symmetric.

```python
normal = np.random.normal(loc=0, scale=1, size=1000)
```

Properties:

- mean, median, and mode are near the center
- many values near the average
- fewer values far from the average

Common examples:

- measurement errors
- test scores in large populations
- standardized features

---

## Skewed Distribution

### Right-Skewed

Long tail on the right.

Examples:

- income
- house prices
- customer spend

```python
right_skewed = np.random.exponential(scale=2, size=1000)
```

For right-skewed data, median is often better than mean.

### Left-Skewed

Long tail on the left.

Example:

- scores on an easy exam where most people score high

---

## Uniform Distribution

All values are roughly equally likely.

```python
uniform = np.random.uniform(low=0, high=1, size=1000)
```

Example:

- random number generator output

---

## Binomial Distribution

Counts successes in repeated yes/no trials.

```python
binomial = np.random.binomial(n=10, p=0.5, size=1000)
```

Examples:

- number of heads in 10 coin flips
- number of conversions from 100 visitors
- number of churned customers in a sample

---

## Distribution Checks in Pandas

```python
df["salary"].describe()
df["salary"].hist(bins=30)
df["salary"].skew()
```

Positive skew means right-skewed. Negative skew means left-skewed.

---

## Why Distribution Shape Matters

Distribution affects:

- choice of mean vs median
- outlier detection
- statistical test selection
- feature transformations
- model assumptions

---

## Common Mistakes

- Assuming every variable is normally distributed.
- Using mean for heavily skewed data.
- Ignoring outliers visible in a histogram.
- Reporting summary statistics without checking shape.

---

## Practice

Tasks:

- generate normal data and plot histogram
- generate exponential data and plot histogram
- calculate mean and median for both
- compare how skew changes mean vs median

---

## Interview Questions

**Q1:** What is a normal distribution?

> A symmetric bell-shaped distribution where values cluster around the mean.

**Q2:** What is right skew?

> A distribution with a long tail toward higher values.

**Q3:** Why does distribution shape matter?

> It affects summary statistics, outlier handling, transformations, and statistical tests.

---

## ✅ Key Takeaways

- A distribution shows how values are spread.
- Normal distributions are symmetric.
- Skewed distributions have long tails.
- Median is often safer for skewed data.
- Always visualize important numeric variables.

---

## 🔗 What's Next?

➡️ [[05-statistics-cheat-sheet]] — Review the core formulas and commands
