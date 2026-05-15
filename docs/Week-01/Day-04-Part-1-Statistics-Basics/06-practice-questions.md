# Practice Questions: Statistics Basics

Work through these in order. Warm-up builds fluency. Main exercises test understanding. Stretch problems require combining multiple concepts — the kind of thinking you need in interviews and on real projects.

---

## Setup

```python
import pandas as pd
import numpy as np
from scipy import stats
```

---

## Level 1 — Warm-Up

These check that you can use the right functions. No trick questions.

### Q1.1 — Basic Summary Statistics

```python
response_times = pd.Series([120, 135, 118, 145, 132, 128, 890, 125, 140, 130])
```

Compute:
- Mean, median, mode
- Variance and standard deviation (sample)
- IQR
- Identify whether the outlier is at the high or low end

??? "Show answer"
    ```python
    print(f"Mean:    {response_times.mean():.1f} ms")      # 216.3 ms — pulled up by 890
    print(f"Median:  {response_times.median():.1f} ms")    # 131.5 ms — more representative
    print(f"Std:     {response_times.std():.1f} ms")       # 237.5 ms — inflated by outlier

    q1  = response_times.quantile(0.25)
    q3  = response_times.quantile(0.75)
    iqr = q3 - q1
    print(f"IQR:     {iqr:.1f} ms")                        # 16.25 ms — typical variation

    print(f"Skew:    {response_times.skew():.2f}")          # large positive — right-skewed
    ```
    The 890ms response time is at the high end. It inflates the mean by ~85ms and triples the std. The median and IQR tell the real story: typical response times are around 131ms with 16ms of normal variation.

---

### Q1.2 — Choosing Mean vs Median

For each dataset below, state whether you would report the mean or median, and why.

a) Daily temperatures in a city over one year (°C)

b) Individual income in a country

c) Number of bugs per sprint over 20 sprints (most sprints have 2-5 bugs, one sprint had 50)

d) Customer ratings (1-5 stars) aggregated by product

??? "Show answer"
    a) **Mean** — temperatures are symmetric throughout the year. The mean temperature is a meaningful summary (e.g., "average January temperature is 12°C").

    b) **Median** — income is strongly right-skewed. The top 1% earn orders of magnitude more than the median. Median income reflects what a typical person earns; mean income does not.

    c) **Median** — the one sprint with 50 bugs is an outlier. Median is more representative of the typical sprint.

    d) **Mean** — star ratings have a known scale (1-5), and the average rating is how most systems present product quality. You could also report the distribution (how many 1-star, 2-star etc.) which is more informative than either.

---

### Q1.3 — Probability Calculations

Calculate the following by hand, then verify with Python:

a) P(drawing an ace from a standard 52-card deck)

b) P(drawing two aces in a row, WITH replacement)

c) P(drawing two aces in a row, WITHOUT replacement)

d) P(at least one head in 4 fair coin flips)

??? "Show answer"
    ```python
    # a) P(ace)
    p_ace = 4 / 52
    print(f"a) P(ace) = {p_ace:.4f}")  # 0.0769

    # b) With replacement — independent events
    p_two_aces_with = (4/52) * (4/52)
    print(f"b) P(two aces, with replacement) = {p_two_aces_with:.4f}")  # 0.0059

    # c) Without replacement — conditional
    p_two_aces_without = (4/52) * (3/51)
    print(f"c) P(two aces, without replacement) = {p_two_aces_without:.4f}")  # 0.0045

    # d) Complement: P(at least one head) = 1 - P(no heads)
    p_at_least_one_head = 1 - (0.5 ** 4)
    print(f"d) P(at least one head in 4 flips) = {p_at_least_one_head:.4f}")  # 0.9375
    ```
    Part d demonstrates the power of the complement rule — it is far easier than counting all outcomes with at least one head.

---

## Level 2 — Main Exercises

These require choosing the right tool, not just applying it.

### Q2.1 — Salary Analysis (The Classic Skew Example)

```python
np.random.seed(42)
dept_salaries = pd.DataFrame({
    "name":       ["Alice", "Bob", "Carol", "David", "Eve", "Frank", "Grace", "CEO"],
    "department": ["Eng", "Eng", "Sales", "Sales", "HR", "Eng", "HR", "Exec"],
    "salary":     [85000, 92000, 58000, 64000, 47000, 110000, 51000, 1800000]
})
```

Tasks:
1. Calculate mean and median salary for the full company
2. Calculate mean and median salary per department
3. Identify which department's numbers are most misleading if you use the mean
4. Calculate the CV (coefficient of variation) per department
5. State which two numbers you would report in a press release and why

??? "Show answer"
    ```python
    # 1. Company-wide
    overall = dept_salaries["salary"]
    print(f"Company mean:   ${overall.mean():>10,.0f}")    # $288,375 — distorted
    print(f"Company median: ${overall.median():>10,.0f}")  # $74,500  — representative

    # 2. Per department
    dept_summary = dept_salaries.groupby("department")["salary"].agg(["mean", "median"])
    print(dept_summary)

    # 3. The Exec department has one person (the CEO at $1.8M)
    # Any department with the CEO skews the numbers. In this case,
    # "Exec" mean = $1,800,000 which is just the CEO's salary — pointless.
    # For multi-person departments: Engineering ($95,667 mean vs $92,000 median)
    # shows mild right skew from Frank at $110K.

    # 4. CV per department
    cv = dept_salaries.groupby("department")["salary"].apply(
        lambda x: (x.std() / x.mean()) * 100 if len(x) > 1 else float('nan')
    )
    print("\nCoefficient of Variation (%)")
    print(cv.round(1))

    # 5. In a press release:
    # Report MEDIAN salary ($74,500) — it represents what most employees earn.
    # Report the distribution by department — it reveals pay equity between teams.
    # The CEO's $1.8M is public knowledge anyway; hiding it in an average is misleading.
    ```

---

### Q2.2 — Applying Bayes' Theorem to a Business Problem

Your fraud detection model has the following properties:
- 0.5% of transactions are fraudulent (base rate)
- The model correctly identifies fraud 92% of the time (sensitivity)
- The model incorrectly flags legitimate transactions 3% of the time (false positive rate)

A transaction is flagged by the model. What is the probability it is actually fraudulent?

Then: if you improve the model's sensitivity to 99% while keeping the false positive rate at 3%, how much does the posterior probability change?

??? "Show answer"
    ```python
    def fraud_posterior(sensitivity, fpr, prevalence=0.005):
        p_fraud     = prevalence
        p_not_fraud = 1 - prevalence

        # Total probability of being flagged
        p_flagged = (sensitivity * p_fraud) + (fpr * p_not_fraud)

        # Bayes: P(fraud | flagged)
        p_fraud_given_flagged = (sensitivity * p_fraud) / p_flagged
        return p_fraud_given_flagged

    p_v1 = fraud_posterior(sensitivity=0.92, fpr=0.03)
    p_v2 = fraud_posterior(sensitivity=0.99, fpr=0.03)

    print(f"Model v1 (92% sensitivity): P(fraud | flagged) = {p_v1:.4f} ({p_v1*100:.1f}%)")
    print(f"Model v2 (99% sensitivity): P(fraud | flagged) = {p_v2:.4f} ({p_v2*100:.1f}%)")
    # Output:
    # Model v1 (92% sensitivity): P(fraud | flagged) = 0.1330 (13.3%)
    # Model v2 (99% sensitivity): P(fraud | flagged) = 0.1420 (14.2%)
    ```

    Even with 92% sensitivity, only 13.3% of flagged transactions are actually fraudulent. This seems low but is correct — because fraud is rare (0.5%), the 3% false positive rate on the 99.5% of legitimate transactions produces many more false alarms than the 92% catch rate catches real fraud.

    The key insight: **reducing the false positive rate from 3% to 1% would improve the posterior far more than improving sensitivity from 92% to 99%**. Try it:

    ```python
    p_v3 = fraud_posterior(sensitivity=0.92, fpr=0.01)
    print(f"Model v3 (1% FPR): P(fraud | flagged) = {p_v3:.4f} ({p_v3*100:.1f}%)")
    # Output: P(fraud | flagged) = 0.3172 (31.7%)
    ```

---

### Q2.3 — Distribution Identification

For each scenario, identify which distribution best describes the random variable, state its parameters, and compute one meaningful probability.

a) A call center handles an average of 8 calls per hour. What is the probability of handling more than 12 calls in the next hour?

b) A quality control process samples 200 items. The defect rate is 2%. What is the probability of finding more than 6 defective items?

c) A standardized test has a mean score of 500 and a standard deviation of 100. What fraction of test-takers score above 650?

??? "Show answer"
    ```python
    # a) Poisson — events per time interval, lambda = 8
    from scipy import stats

    poisson_calls = stats.poisson(mu=8)
    p_more_than_12 = 1 - poisson_calls.cdf(12)
    print(f"a) P(calls > 12) = {p_more_than_12:.4f}")   # 0.0638 — about 6.4%

    # b) Binomial — fixed trials, constant p
    binom_defects = stats.binom(n=200, p=0.02)
    p_more_than_6 = 1 - binom_defects.cdf(6)
    print(f"b) P(defects > 6) = {p_more_than_6:.4f}")  # 0.0694 — about 6.9%

    # c) Normal — symmetric measurement, use Z-score
    z = (650 - 500) / 100  # z = 1.5
    p_above_650 = 1 - stats.norm.cdf(z)
    print(f"c) P(score > 650) = {p_above_650:.4f}")     # 0.0668 — about 6.7%
    print(f"   Z-score = {z:.1f} (1.5 standard deviations above mean)")
    ```

---

## Level 3 — Stretch Problems

These require synthesizing multiple concepts, interpreting results, and making a recommendation.

### Q3.1 — Full EDA on a Real-Shaped Dataset

```python
np.random.seed(7)

# Simulated e-commerce data: 500 customer orders
n = 500
orders = pd.DataFrame({
    "order_value":   np.concatenate([
                         np.random.normal(150, 40, 400),    # typical orders
                         np.random.normal(800, 100, 100)    # premium orders
                     ]),
    "items_in_cart": np.random.poisson(lam=3.5, size=n),
    "days_since_last_order": np.random.exponential(scale=45, size=n),
    "customer_tier": np.random.choice(["Bronze", "Silver", "Gold"], size=n,
                                       p=[0.6, 0.3, 0.1])
})
orders["order_value"] = orders["order_value"].clip(lower=10)
```

Complete the following analysis:

1. For each numeric column, compute mean, median, std, IQR, and skew. Decide whether to report mean or median for each.
2. Compute the probability that a randomly selected order has a value above $300.
3. Identify outliers in `order_value` using the IQR method. How many are there? Are they real customers or data errors?
4. Compute the 90th, 95th, and 99th percentile of `order_value`. What does this tell you about high-value customers?
5. Group by `customer_tier` and compute median order value per tier. Does the ranking match your expectation?

??? "Show answer"
    ```python
    import pandas as pd
    import numpy as np
    from scipy import stats as scipy_stats

    np.random.seed(7)
    n = 500
    orders = pd.DataFrame({
        "order_value":   np.concatenate([
                             np.random.normal(150, 40, 400),
                             np.random.normal(800, 100, 100)
                         ]),
        "items_in_cart": np.random.poisson(lam=3.5, size=n),
        "days_since_last_order": np.random.exponential(scale=45, size=n),
        "customer_tier": np.random.choice(["Bronze", "Silver", "Gold"], size=n,
                                           p=[0.6, 0.3, 0.1])
    })
    orders["order_value"] = orders["order_value"].clip(lower=10)

    # 1. Summary per column
    for col in ["order_value", "items_in_cart", "days_since_last_order"]:
        s = orders[col]
        q1, q3 = s.quantile(0.25), s.quantile(0.75)
        skew = s.skew()
        recommendation = "Median" if abs(skew) > 0.5 else "Mean"
        print(f"\n{col}")
        print(f"  Mean: {s.mean():.1f} | Median: {s.median():.1f} | "
              f"Std: {s.std():.1f} | IQR: {q3-q1:.1f} | Skew: {skew:.2f}")
        print(f"  Report: {recommendation}")

    # 2. P(order_value > 300)
    p_above_300 = (orders["order_value"] > 300).mean()
    print(f"\nP(order > $300): {p_above_300:.4f}  ({p_above_300*100:.1f}%)")

    # 3. Outlier detection
    ov = orders["order_value"]
    q1, q3 = ov.quantile(0.25), ov.quantile(0.75)
    iqr = q3 - q1
    lower, upper = q1 - 1.5 * iqr, q3 + 1.5 * iqr
    outliers = ov[(ov < lower) | (ov > upper)]
    print(f"\nOutliers: {len(outliers)} records outside [{lower:.1f}, {upper:.1f}]")
    print(f"  Min outlier: ${outliers.min():.1f} | Max outlier: ${outliers.max():.1f}")
    # These are the premium segment orders — real customers, not errors.
    # The IQR method flags them because it is tuned for the typical segment.

    # 4. High-value customer thresholds
    print("\nOrder value percentiles:")
    for p in [0.90, 0.95, 0.99]:
        print(f"  {p*100:.0f}th percentile: ${ov.quantile(p):.0f}")
    # The 99th percentile is a VIP threshold — handle these customers differently.

    # 5. Tier analysis
    print("\nMedian order value by tier:")
    print(orders.groupby("customer_tier")["order_value"].median().sort_values(ascending=False))
    # Gold should be highest; if not, the tier labels may not be tracking actual spend
    ```

---

### Q3.2 — Detecting a Broken Process

You monitor a manufacturing line. The process produces widgets, and the weight should be normally distributed with mean 500g and standard deviation 5g. You collect 30 samples.

```python
np.random.seed(99)
# The process drifted — the mean shifted to 504g without anyone noticing
actual_weights = pd.Series(np.random.normal(loc=504, scale=5, size=30))
```

1. Calculate the Z-score for the sample mean relative to the process specification (μ=500, σ=5).
2. Calculate P(sample mean ≥ observed mean | process is working correctly).
   Hint: the standard error of the mean is `σ / √n`.
3. At what sample mean would you become 95% confident the process has drifted?
4. How many samples would you need to detect a 2g shift with 95% confidence?

??? "Show answer"
    ```python
    n         = 30
    spec_mean = 500
    spec_std  = 5
    sample_mean = actual_weights.mean()
    se          = spec_std / np.sqrt(n)   # standard error of the mean

    # 1. Z-score of the sample mean
    z = (sample_mean - spec_mean) / se
    print(f"Sample mean: {sample_mean:.2f}g")
    print(f"Standard error: {se:.3f}g")
    print(f"Z-score: {z:.3f}")

    # 2. P(mean >= observed | process OK)
    # This is the p-value for a one-sided test
    p_value = 1 - scipy_stats.norm.cdf(z)
    print(f"P(mean >= {sample_mean:.2f}g | process OK) = {p_value:.4f}")
    # If p < 0.05, we have evidence the process has drifted

    # 3. Critical value at 95% confidence (one-sided)
    z_critical = scipy_stats.norm.ppf(0.95)
    critical_mean = spec_mean + z_critical * se
    print(f"\n95% detection threshold: {critical_mean:.2f}g")
    print(f"Any sample mean above {critical_mean:.1f}g triggers an alert")

    # 4. Sample size to detect a 2g shift with 95% confidence
    # The shift is 2g. SE = 5 / sqrt(n). We need Z_shift >= 1.645 (95% one-sided).
    # Z = shift / (sigma / sqrt(n))  => n = (z_critical * sigma / shift)^2
    shift = 2
    n_needed = ((z_critical * spec_std) / shift) ** 2
    print(f"\nSamples needed to detect {shift}g shift: {np.ceil(n_needed):.0f}")
    ```

    The Z-score tells you whether a shift this large would be surprising if the process were actually working correctly. The lower the p-value, the stronger your evidence that something changed. This is the logic behind statistical process control and hypothesis testing.

---

> [!success] Self-Check
> - [ ] I can compute mean, median, std, IQR for a series and choose which to report based on skew
> - [ ] I can detect outliers using both the IQR method and Z-score method
> - [ ] I can apply the complement, AND, OR, and conditional probability rules
> - [ ] I can compute a posterior probability using Bayes' theorem and explain the result in plain language
> - [ ] I can identify the correct distribution for a scenario (Normal / Binomial / Poisson) and compute probabilities with scipy.stats
> - [ ] I can compute a Z-score and use it to find the probability of an observed value

---

[[05-statistics-cheat-sheet|Back: Statistics Cheat Sheet]] | [[../Day-04-Part-2-Inferential-Statistics/01-hypothesis-testing|Next: Inferential Statistics]]
