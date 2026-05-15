# Distributions

Before you model data, you need to understand its shape. Choosing the wrong distribution assumption is one of the most common sources of silent error in data science — your model trains, your metrics look fine, and your predictions are systematically wrong in ways that only show up when something breaks in production.

## Learning Objectives

- Distinguish between PDF (probability density function) and CDF (cumulative distribution function)
- Recognize and generate normal, binomial, Poisson, and uniform distributions in Python using `scipy.stats` and NumPy
- Apply the 68-95-99.7 rule and compute Z-scores
- Choose the right distribution for a given real-world scenario
- Detect skew and kurtosis numerically and visually

---

## What Is a Distribution?

A distribution describes how probability is spread across possible values. It answers:
- Which values are most likely?
- How spread out are the values?
- Is the data symmetric, or does it have a long tail in one direction?
- Are extreme values common or rare?

### PDF vs CDF

The **probability density function (PDF)** tells you the relative likelihood of a value. For continuous variables, the area under the PDF between two points equals the probability of landing in that range.

The **cumulative distribution function (CDF)** tells you the probability of getting a value less than or equal to x. `CDF(x) = P(X ≤ x)`.

```python
import numpy as np
import pandas as pd
from scipy import stats

# Normal distribution with mean=0, std=1
dist = stats.norm(loc=0, scale=1)

x = 1.0
print(f"PDF at x=1: {dist.pdf(x):.4f}")   # Output: 0.2420 — relative density
print(f"CDF at x=1: {dist.cdf(x):.4f}")   # Output: 0.8413 — P(X ≤ 1) = 84.13%
print(f"P(-1 ≤ X ≤ 1): {dist.cdf(1) - dist.cdf(-1):.4f}")  # Output: 0.6827 — ~68%
```

> [!tip] CDF is what you use in practice
> The CDF directly answers "what fraction of my data falls below this threshold?" That is the question you actually need answered — for percentiles, hypothesis tests, and anomaly detection. The PDF is the mathematical tool underneath it.

---

## Normal Distribution

The normal distribution is the most important distribution in statistics. It is symmetric and bell-shaped, with the mean, median, and mode all at the center.

**Why it appears everywhere:** The Central Limit Theorem says that the average of a large number of independent random variables tends toward a normal distribution, regardless of the original distribution. This is why so many natural phenomena — measurement errors, test scores, biological measurements — approximate normality.

> [!info] Normal Distribution Parameters
> - `loc` = mean (μ) — controls the center
> - `scale` = standard deviation (σ) — controls the width
> - Notation: X ~ N(μ, σ²)

```python
np.random.seed(42)

# Heights of adult males: mean 170cm, std 7cm
heights = pd.Series(stats.norm.rvs(loc=170, scale=7, size=5000))

print(f"Mean:   {heights.mean():.1f} cm")   # Output: ~170.0 cm
print(f"Median: {heights.median():.1f} cm") # Output: ~170.0 cm (close to mean)
print(f"Std:    {heights.std():.1f} cm")    # Output: ~7.0 cm
print(f"Skew:   {heights.skew():.3f}")      # Output: ~0.0 (symmetric)
```

### The 68-95-99.7 Rule

For any normal distribution:

| Range | Probability |
|-------|-------------|
| μ ± 1σ | 68.27% |
| μ ± 2σ | 95.45% |
| μ ± 3σ | 99.73% |

```python
mean_h = 170
std_h  = 7

print(f"68% of heights fall between {mean_h - std_h} and {mean_h + std_h} cm")
print(f"95% of heights fall between {mean_h - 2*std_h} and {mean_h + 2*std_h} cm")
print(f"99.7% of heights fall between {mean_h - 3*std_h} and {mean_h + 3*std_h} cm")
# Output:
# 68% of heights fall between 163 and 177 cm
# 95% of heights fall between 156 and 184 cm
# 99.7% of heights fall between 149 and 191 cm
```

### Z-Score — Standardizing to N(0, 1)

A Z-score measures how many standard deviations a value is from the mean. It converts any normal distribution to the standard normal (mean=0, std=1), which lets you look up probabilities in a table or use `scipy`.

> [!info] Z-Score Formula
> `Z = (x - μ) / σ`

```python
# Is a height of 185 cm unusual?
height_value = 185
z_score = (height_value - mean_h) / std_h

p_above = 1 - stats.norm.cdf(z_score)  # P(X > 185)

print(f"Z-score for 185 cm: {z_score:.2f}")            # Output: 2.14
print(f"P(height > 185 cm): {p_above:.4f}")            # Output: 0.0161 — about 1.6%
print(f"185 cm is in the top {p_above*100:.1f}% of heights")
```

```python
# Bulk standardization in pandas
heights_z = (heights - heights.mean()) / heights.std()

print(f"Standardized mean: {heights_z.mean():.4f}")  # Output: ~0.0
print(f"Standardized std:  {heights_z.std():.4f}")   # Output: ~1.0
```

> [!warning] Z-scores assume normality
> Applying a Z-score transformation is perfectly valid. Interpreting the Z-score as a probability (via the normal CDF) is only valid if the underlying data is approximately normal. Always check with a histogram or `.skew()` before relying on normal-distribution probabilities.

---

## Binomial Distribution

The binomial distribution counts the number of successes in a fixed number of independent yes/no trials, each with the same probability of success.

**Real examples:** number of customers who click an ad out of 1,000 shown to, number of defective items in a batch, number of correct predictions in 50 test cases.

> [!info] Binomial Parameters
> - `n` = number of trials
> - `p` = probability of success on each trial
> - X counts successes: X ~ Binomial(n, p)
> - Mean = n × p
> - Std = √(n × p × (1 - p))

```python
# A/B test: 500 visitors see a new landing page, historic conversion rate is 8%
# What distribution describes the number of conversions?

n = 500
p = 0.08

binom_dist = stats.binom(n=n, p=p)

expected_conversions = n * p
std_conversions      = np.sqrt(n * p * (1 - p))

print(f"Expected conversions: {expected_conversions:.1f}")  # Output: 40.0
print(f"Std of conversions:   {std_conversions:.1f}")       # Output: 6.1

# Probability of getting exactly 40 conversions
print(f"P(X = 40): {binom_dist.pmf(40):.4f}")  # Output: ~0.0650

# Probability of getting 50 or more conversions (unusually high?)
print(f"P(X ≥ 50): {1 - binom_dist.cdf(49):.4f}")  # Output: ~0.0556
```

```python
# Simulate the A/B test 10,000 times
np.random.seed(42)
simulated_conversions = np.random.binomial(n=500, p=0.08, size=10_000)

print(f"Simulated mean: {simulated_conversions.mean():.1f}")   # Output: ~40.0
print(f"Simulated std:  {simulated_conversions.std():.1f}")    # Output: ~6.1
```

> [!tip] Binomial powers A/B testing
> When you run an A/B test on a website, you're comparing two binomial processes — the conversion rate under control and under treatment. Understanding the binomial distribution tells you how much variation to expect by chance, which is the foundation of statistical significance testing.

---

## Poisson Distribution

The Poisson distribution counts events that occur at a constant average rate in a fixed time interval. It applies when events are rare relative to the number of opportunities and when events are independent.

**Real examples:** customer support tickets per hour, server errors per day, transactions per minute, accidents per month at an intersection.

> [!info] Poisson Parameter
> - `λ` (lambda) = average number of events per interval
> - Mean = λ, Variance = λ (they are equal — a useful diagnostic)
> - X ~ Poisson(λ)

```python
# A support team receives an average of 12 tickets per hour
lam = 12
poisson_dist = stats.poisson(mu=lam)

# Probability of exactly 12 tickets in the next hour
print(f"P(X = 12): {poisson_dist.pmf(12):.4f}")  # Output: 0.1144

# Probability of more than 20 tickets (team overwhelmed?)
print(f"P(X > 20): {1 - poisson_dist.cdf(20):.4f}")  # Output: 0.0113 — about 1.1%

# Probability of fewer than 5 tickets (unusually quiet hour?)
print(f"P(X < 5):  {poisson_dist.cdf(4):.4f}")  # Output: 0.0076 — about 0.8%
```

```python
# Simulate ticket arrivals for 1,000 hours
np.random.seed(0)
hourly_tickets = pd.Series(np.random.poisson(lam=12, size=1000))

print(f"Simulated mean:     {hourly_tickets.mean():.2f}")  # Output: ~12.0
print(f"Simulated variance: {hourly_tickets.var():.2f}")   # Output: ~12.0
# Mean ≈ Variance is the signature of a Poisson distribution
```

> [!warning] Poisson assumes events are independent
> If a surge of tickets causes more tickets (customers help each other, or a bug generates cascading errors), events are NOT independent. In that case, the variance will exceed the mean — called overdispersion. Check `variance / mean` on count data. If it is much greater than 1, consider a negative binomial distribution instead.

---

## Uniform Distribution

All values in a range are equally likely. This is the "fair" distribution — no value is more probable than any other.

**Real examples:** random number generation, simulation seeds, arrival times within a time window if you have no other information.

```python
# Uniform distribution between 0 and 1
uniform_dist = stats.uniform(loc=0, scale=1)

print(f"Mean:   {uniform_dist.mean():.3f}")  # Output: 0.500
print(f"Std:    {uniform_dist.std():.3f}")   # Output: 0.289

# Simulate and check
np.random.seed(42)
samples = pd.Series(np.random.uniform(low=0, high=1, size=10000))
print(f"Simulated mean: {samples.mean():.3f}")  # Output: ~0.500
print(f"Simulated std:  {samples.std():.3f}")   # Output: ~0.289
```

> [!info] Uniform distribution in data science
> The uniform distribution is the starting point for simulation. When you call `np.random.uniform()`, you draw from it. Many sampling procedures convert uniform random numbers into other distributions. It also appears in p-value distributions under the null hypothesis — if the null is true, p-values are uniformly distributed between 0 and 1.

---

## Skewness and Kurtosis

Two numbers that describe the shape of a distribution beyond mean and variance.

**Skewness** measures asymmetry.
- Skew = 0: symmetric (like normal)
- Skew > 0: right-skewed (long right tail — income, house prices)
- Skew < 0: left-skewed (long left tail — easy exam scores)

**Kurtosis** measures tail heaviness relative to a normal distribution.
- Kurtosis = 0 (excess): normal tails
- Kurtosis > 0: heavy tails (more extreme values than normal — financial returns)
- Kurtosis < 0: light tails (fewer extreme values than normal)

```python
np.random.seed(42)

normal_data   = pd.Series(np.random.normal(loc=50, scale=10, size=5000))
right_skewed  = pd.Series(np.random.exponential(scale=10, size=5000))
left_skewed   = pd.Series(100 - np.random.exponential(scale=10, size=5000))

for name, data in [("Normal", normal_data), ("Right-skewed", right_skewed), ("Left-skewed", left_skewed)]:
    print(f"{name:15s} | Mean: {data.mean():5.1f} | Median: {data.median():5.1f} | "
          f"Skew: {data.skew():+.2f} | Kurt: {data.kurtosis():+.2f}")

# Output (approximate):
# Normal          | Mean:  50.1 | Median:  50.1 | Skew: +0.01 | Kurt: -0.01
# Right-skewed    | Mean:  10.1 | Median:   6.9 | Skew: +2.03 | Kurt: +6.13
# Left-skewed     | Mean:  89.9 | Median:  93.1 | Skew: -2.03 | Kurt: +6.13
```

> [!tip] Rule of thumb for skew
> `|skew| < 0.5` — roughly symmetric, mean is fine.
> `0.5 ≤ |skew| < 1` — moderate skew, consider using median.
> `|skew| ≥ 1` — strong skew, median is more appropriate; consider log transformation.

---

## Detecting and Comparing Distributions

Always visualize before modeling. The `.describe()` output hides skew.

```python
np.random.seed(7)

# Simulate customer purchase amounts (right-skewed)
purchase_amounts = pd.Series(np.random.exponential(scale=150, size=2000))

print("=== Summary Statistics ===")
print(purchase_amounts.describe().round(2))
print(f"\nSkew:     {purchase_amounts.skew():.3f}")
print(f"Kurtosis: {purchase_amounts.kurtosis():.3f}")

# Q: Is mean or median more appropriate here?
gap = purchase_amounts.mean() - purchase_amounts.median()
print(f"\nMean - Median gap: {gap:.2f}")
# A large positive gap confirms right skew — report median
```

```python
# Check if data could be normal using the Shapiro-Wilk test (small samples)
from scipy.stats import shapiro

sample = purchase_amounts.sample(50, random_state=42)
stat, p_value = shapiro(sample)
print(f"Shapiro-Wilk p-value: {p_value:.6f}")
# p < 0.05 means we reject the hypothesis of normality
```

---

## Choosing the Right Distribution

| Scenario | Distribution | Key Question |
|----------|-------------|--------------|
| Height, weight, test scores (large samples) | Normal | Is it symmetric? |
| Number of successes in fixed trials | Binomial | Yes/No outcomes? Fixed n? |
| Rare events per unit time or area | Poisson | Events independent? Count data? |
| Random numbers, arrival within a window | Uniform | All outcomes equally likely? |
| Income, house prices, response times | Log-normal | Right-skewed, positive only? |
| Time until failure, customer lifetime | Exponential | Memoryless waiting time? |

> [!warning] Real data rarely fits one distribution perfectly
> The goal is not to find the one true distribution — it is to find a distribution that approximates the data well enough for your purpose. Always check your assumption with a histogram, a Q-Q plot, or a statistical test. Never assume normality without evidence, especially for financial or behavioral data.

---

> [!success] Key Takeaways
> - A distribution describes how probability is spread across values. Know the PDF vs CDF distinction.
> - Normal distribution: symmetric, bell-shaped. Z-scores and the 68-95-99.7 rule apply.
> - Binomial: count of successes in fixed independent trials. Foundation of A/B testing.
> - Poisson: count of events per interval at a constant rate. Mean equals variance.
> - Uniform: all outcomes equally likely. The starting point for random number generation.
> - Skew tells you which direction the tail goes. Mean follows the tail; median stays near the center.
> - Always visualize your data before assuming any distribution.

---

[[03-probability-basics|Back: Probability Basics]] | [[05-statistics-cheat-sheet|Next: Statistics Cheat Sheet]]
