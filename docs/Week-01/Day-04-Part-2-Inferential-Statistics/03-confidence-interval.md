# Confidence Intervals

A p-value tells you whether to reject a hypothesis. A confidence interval tells you what the true value probably looks like. If you had to pick one to report, most statisticians would pick the confidence interval — it carries more information, it is more honest about uncertainty, and it forces you to think about effect size rather than just significance. The fact that most presentations lead with p-values is a historical accident worth correcting.

## Learning Objectives

- State the correct frequentist interpretation of a confidence interval
- Identify the most common misinterpretation (and explain why it is wrong)
- Calculate confidence intervals for means using `scipy.stats.t.interval()`
- Explain what determines interval width and how to make it narrower
- Use confidence intervals to make better decisions than p-values alone allow

---

## The Problem with Point Estimates

Your sample mean is 28.5 minutes. What does that tell you?

Not much on its own. The sample mean is a point estimate — a single number representing your best guess at the population parameter. But it carries no information about how uncertain that guess is. You need an interval.

```python
import numpy as np
from scipy import stats

# Delivery times from a sample of 10 orders
delivery_times = np.array([28, 31, 29, 27, 30, 26, 32, 28, 29, 27])

print(f"Sample mean: {delivery_times.mean():.2f} minutes")
print(f"Sample std:  {delivery_times.std(ddof=1):.2f} minutes")
print(f"Sample size: {len(delivery_times)}")
# Output:
# Sample mean: 28.70 minutes
# Sample std:  1.77 minutes
# Sample size: 10
```

The mean is 28.7. But given only 10 observations, how much do you trust that? This is exactly what the confidence interval answers.

---

## What a Confidence Interval Actually Means

Here is the correct frequentist interpretation, stated precisely:

> If we collected many samples from the same population and calculated a 95% confidence interval from each one, 95% of those intervals would contain the true population parameter.

That is a statement about the procedure, not about any single interval. Any specific interval either contains the true parameter or it does not — there is no probability about it once calculated.

> [!warning] The Most Common Misinterpretation
> "There is a 95% probability that the true parameter lies within this interval."
>
> This statement treats the true parameter as a random variable. In frequentist statistics, the parameter is a fixed (unknown) constant. Only the interval is random. The correct interpretation is about the long-run behavior of the procedure, not about any single interval.
>
> In practice, saying "we are 95% confident the true mean lies between 27.2 and 30.2 minutes" is widely accepted and usually understood. Just do not say "there is a 95% probability" if you want to be technically precise.

---

## Calculating a Confidence Interval for the Mean

The formula uses the t-distribution because we are estimating the population standard deviation from the sample:

```
CI = x̄ ± t*(s / √n)
```

Where t* is the critical value from the t-distribution at the desired confidence level with n-1 degrees of freedom.

```python
import numpy as np
from scipy import stats

delivery_times = np.array([28, 31, 29, 27, 30, 26, 32, 28, 29, 27])

sample_mean = delivery_times.mean()
sample_sem  = stats.sem(delivery_times)  # standard error of the mean = std / sqrt(n)
n           = len(delivery_times)

# 95% confidence interval
ci_95 = stats.t.interval(
    confidence=0.95,
    df=n - 1,
    loc=sample_mean,
    scale=sample_sem
)

print(f"Sample mean:    {sample_mean:.2f}")
print(f"Std error:      {sample_sem:.4f}")
print(f"95% CI:         ({ci_95[0]:.2f}, {ci_95[1]:.2f})")
# Output:
# Sample mean:    28.70
# Std error:      0.5600
# 95% CI:         (27.43, 29.97)
```

Now the estimate is richer: we are 95% confident the true average delivery time falls between 27.4 and 30.0 minutes. A decision-maker can see whether that interval includes the target (30 minutes) and how close it is.

---

## Comparing Confidence Levels

Higher confidence comes at a cost — wider intervals.

```python
import numpy as np
from scipy import stats

data = np.array([28, 31, 29, 27, 30, 26, 32, 28, 29, 27])
mean = data.mean()
sem  = stats.sem(data)
n    = len(data)

for level in [0.90, 0.95, 0.99]:
    ci = stats.t.interval(confidence=level, df=n-1, loc=mean, scale=sem)
    width = ci[1] - ci[0]
    print(f"{int(level*100)}% CI: ({ci[0]:.2f}, {ci[1]:.2f})  width = {width:.2f}")
# Output:
# 90% CI: (27.69, 29.71)  width = 2.02
# 95% CI: (27.43, 29.97)  width = 2.54
# 99% CI: (26.88, 30.52)  width = 3.64
```

A 99% CI is more confident but less precise. The right choice depends on the stakes. In medical contexts, you might accept less precision to have more confidence. In a low-stakes UI experiment, 90% is often sufficient.

---

## What Determines Interval Width

Width is not arbitrary. Three factors control it:

**Sample size (n):** The most powerful lever. Width shrinks as √n. To halve the width, you need four times as many observations.

**Variability (s):** High variance in your data means high uncertainty in the estimate. You cannot shrink this without changing what you measure.

**Confidence level:** Higher confidence = wider interval. This is the direct tradeoff.

```python
import numpy as np
from scipy import stats

# Demonstrating the effect of sample size
np.random.seed(42)
population = np.random.normal(loc=50, scale=10, size=100000)

for n in [10, 50, 100, 500, 1000]:
    sample = np.random.choice(population, size=n)
    ci = stats.t.interval(0.95, df=n-1, loc=sample.mean(), scale=stats.sem(sample))
    width = ci[1] - ci[0]
    print(f"n={n:5d}: CI = ({ci[0]:.2f}, {ci[1]:.2f}), width = {width:.2f}")
# Output:
# n=   10: CI = (46.35, 57.19), width = 10.84
# n=   50: CI = (47.94, 52.65), width = 4.71
# n=  100: CI = (48.47, 52.04), width = 3.57
# n=  500: CI = (49.55, 51.29), width = 1.74
# n= 1000: CI = (49.64, 50.74), width = 1.10
```

The pattern is clear. More data = more precision. This is the fundamental law of statistical estimation.

---

## Confidence Intervals vs P-Values

| | P-Value | Confidence Interval |
|---|---|---|
| Answers | Is there evidence against H₀? | What is the plausible range of the effect? |
| Reports effect size? | No | Yes (the width and location tell you magnitude) |
| Conveys uncertainty? | Not directly | Yes — wide CI = more uncertain |
| Decision-making | Binary: reject or not | Richer: is the entire CI above the threshold? |

Consider this scenario: your A/B test shows a lift of 0.5% with p = 0.03. Do you ship?

With a p-value alone: "Yes, it's significant."

With a confidence interval: check whether the entire CI is above your minimum detectable effect. If the CI is [0.1%, 0.9%], the lower bound might be too small to justify the engineering cost.

> [!tip] The Confidence Interval as a Decision Tool
> In A/B testing, ask: "Does the entire confidence interval lie above zero (or above my minimum business threshold)?" If the CI includes zero, you cannot rule out that the effect is zero — even if the p-value crossed your threshold. If the CI is entirely above your minimum detectable effect, you have strong evidence to ship.

---

## Confidence Interval for Proportions

For conversion rates and binary outcomes:

```python
import numpy as np
from scipy import stats

# A/B test: 1200 users in treatment, 78 converted
n_treatment = 1200
conversions = 78

# Using the Wilson score interval (recommended for proportions)
from statsmodels.stats.proportion import proportion_confint

ci = proportion_confint(count=conversions, nobs=n_treatment, alpha=0.05, method='wilson')
rate = conversions / n_treatment

print(f"Observed conversion rate: {rate:.4f} ({rate*100:.2f}%)")
print(f"95% CI (Wilson):          ({ci[0]*100:.2f}%, {ci[1]*100:.2f}%)")
# Output:
# Observed conversion rate: 0.0650 (6.50%)
# 95% CI (Wilson):          (5.25%, 8.02%)
```

> [!info] Why Wilson, Not Normal Approximation?
> The standard "p ± z√(p(1-p)/n)" formula behaves poorly when the proportion is near 0 or 1, or when n is small. The Wilson score interval stays within [0, 1] and has better coverage properties. Use it for proportions in practice.

---

## Practice Exercises

**Warm-up:** Calculate 90%, 95%, and 99% confidence intervals for this dataset. Plot the three intervals side-by-side.

```python
import numpy as np
scores = np.array([72, 78, 74, 80, 76, 73, 77, 79, 68, 82, 75, 71])
# Calculate mean, standard error, and three CIs
# Print which confidence levels exclude the value 70
```

**Main:** Your company's previous average NPS score was 42. You surveyed 30 customers and got the following scores. Build a 95% CI and determine whether 42 falls outside it.

```python
nps_scores = [45, 38, 51, 44, 39, 47, 52, 41, 43, 49,
              46, 37, 50, 42, 48, 44, 53, 40, 45, 47,
              41, 38, 52, 46, 44, 39, 50, 43, 47, 45]
```

**Stretch:** Demonstrate the frequentist interpretation of confidence intervals by running a simulation: draw 100 samples of size 30 from a known population, calculate a 95% CI for each, and count how many contain the true mean. It should be close to 95.

---

> [!success] Key Takeaways
> - A 95% CI means: 95% of intervals constructed this way will contain the true parameter. It does not mean this specific interval has a 95% probability of containing it.
> - Wider intervals = more uncertainty. Caused by small samples, high variance, or high confidence level.
> - The most powerful way to narrow a CI is to increase sample size.
> - Confidence intervals are often more useful than p-values because they show magnitude and uncertainty, not just significance.
> - For proportions, use the Wilson score interval instead of the normal approximation.

---

[[02-p-value|Previous: P-Values]] | [[04-correlation|Next: Correlation]]
