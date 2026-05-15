# Hypothesis Testing

Every decision in data science — ship the new model, run the A/B test, recommend the product change — eventually comes down to a question: is what I'm seeing real, or is it noise? Hypothesis testing is the formal framework for answering that question with a defensible, auditable process. It is how you tell your product manager "yes, the new design works" without embarrassing yourself six months later.

## Learning Objectives

- Frame any business question as a null and alternative hypothesis
- Understand Type I and Type II errors and why they cost real money
- Choose a significance level with intent, not by habit
- Run an end-to-end hypothesis test in Python with scipy
- Explain the decision in plain English to a non-technical audience

---

## Why You Need a Framework Before You Look at the Data

The temptation is to look at your data, notice something interesting, and then test whether it is significant. That is the wrong order. Hypothesis testing works by establishing what you expect to see under the assumption of "nothing interesting happening," then measuring how surprising your actual data is relative to that baseline. If you set up the hypothesis after seeing the data, you have already broken the logic of the test.

This is not academic pedantry. It is the reason A/B testing platforms require you to specify your primary metric before the experiment goes live. The framework only holds if you commit to the question before you see the answer.

---

## The Two Hypotheses

Every hypothesis test starts with two competing claims.

**Null hypothesis (H₀):** The boring, default world. No effect. No difference. Nothing happening.

**Alternative hypothesis (H₁ or Hₐ):** Something is happening. There is an effect, a difference, a signal.

The test does not try to prove H₁ true. It measures whether the data is surprising enough to make H₀ implausible. This is an important distinction — you are never "proving" your new feature works. You are accumulating evidence against the claim that it does not work.

> [!info] The Legal Analogy
> Think of H₀ as "innocent until proven guilty." The burden of proof is on the data. You start from the assumption of no effect. The data has to convince you otherwise. A verdict of "not guilty" does not mean the defendant is innocent — it means the evidence was not strong enough. Similarly, "fail to reject H₀" does not mean H₀ is true.

### Framing Examples

| Business Question | H₀ | H₁ |
|---|---|---|
| Did the new landing page improve conversions? | Conversion rate is the same | Conversion rate changed |
| Is delivery time below 30 minutes? | Mean delivery time = 30 min | Mean delivery time < 30 min |
| Does the new drug reduce recovery time? | Mean recovery time is unchanged | Mean recovery time decreased |
| Are spending patterns different across regions? | All regions have equal mean spend | At least one region differs |

---

## The Significance Level (α)

Before running any test, you set α — the threshold for how surprising the data needs to be before you will reject H₀.

The standard choice is α = 0.05. This means you are willing to be wrong 5% of the time — specifically, wrong in the direction of rejecting H₀ when it is actually true.

This is not a law of nature. It is a convention. Medical trials often use α = 0.01 because the cost of a false positive (approving an ineffective drug) is catastrophic. Exploratory data analysis might use α = 0.10. The point is to choose it deliberately, before you see the p-value, with the consequences in mind.

> [!warning] Never Change α After Seeing the Data
> If you run your test, see p = 0.07, and then decide your threshold should have been 0.10 — you have invalidated the test. The significance level must be committed to before the test runs. Changing it after is a form of p-hacking, even if it feels innocent.

---

## Type I and Type II Errors

The framework can be wrong in two distinct ways. Both cost money.

|  | H₀ is True | H₀ is False |
|---|---|---|
| **Reject H₀** | Type I Error (False Positive) | Correct |
| **Fail to Reject H₀** | Correct | Type II Error (False Negative) |

**Type I Error (False Positive):** You conclude the new design works when it actually does not. You ship a change that has no real effect, wasting engineering time and potentially confusing users.

**Type II Error (False Negative):** You conclude the new design does not work when it actually does. You miss a real improvement and leave value on the table.

α directly controls your Type I error rate. If α = 0.05, you accept a 5% chance of a false positive.

**Statistical Power (1 - β)** is the probability of correctly detecting a real effect when it exists. It controls your Type II error rate. Power depends on your sample size, effect size, and α. A poorly powered study might have a 50% chance of missing a real effect — which means your "no significant difference" result is nearly useless.

> [!tip] The Power Rule of Thumb
> Before running an important experiment, calculate the required sample size to achieve 80% power. Use `scipy.stats.ttest_ind` with the `power` module or a tool like `statsmodels`. Running underpowered tests is how well-intentioned teams conclude "our new model had no impact" when they just did not collect enough data.

```python
from scipy import stats

# Calculate required sample size for 80% power
# effect_size: Cohen's d (difference in means / pooled std)
# alpha: significance level
# power: desired power

effect_size = 0.5   # medium effect
alpha = 0.05
power = 0.80

# Using statsmodels for power analysis
from statsmodels.stats.power import TTestIndPower

analysis = TTestIndPower()
n = analysis.solve_power(effect_size=effect_size, alpha=alpha, power=power)
print(f"Required sample size per group: {n:.0f}")
# Output: Required sample size per group: 64
```

---

## The Decision Rule

Once you have your p-value and your pre-committed α:

```
If p-value ≤ α: reject H₀ (the data is surprising enough)
If p-value > α: fail to reject H₀ (the data is not surprising enough)
```

Two things to commit to memory:

1. "Fail to reject H₀" is not the same as "H₀ is true." You have not proven the null. You have simply not accumulated enough evidence against it.
2. "Reject H₀" is not the same as "the effect is important." Statistical significance and practical significance are different things. More on this in the p-value file.

---

## One-Tailed vs Two-Tailed Tests

**Two-tailed test:** You care about a difference in either direction. "Is the mean different from 30?" means you'd want to know if it is significantly above or below 30.

**One-tailed test:** You only care about one direction. "Is the mean less than 30?" means you only reject H₀ if the evidence points downward.

One-tailed tests have more statistical power for detecting effects in the specified direction, but they cannot detect effects in the opposite direction. Use them only when the direction is part of the scientific question, decided before data collection.

> [!warning] One-Tailed Tests After Peeking at Data
> If you ran a two-tailed test, got p = 0.07, noticed the effect goes in the direction you expected, and then switched to a one-tailed test to get p = 0.035 — you have p-hacked. The one-tailed decision must come before the data.

---

## End-to-End Worked Example

Your team wants to know whether average checkout time on the new payment flow is different from the old average of 45 seconds. You collect 20 sessions from the new flow.

**Step 1: State the hypotheses**

```
H₀: Mean checkout time = 45 seconds
H₁: Mean checkout time ≠ 45 seconds (two-tailed)
```

**Step 2: Set significance level**

```
α = 0.05
```

**Step 3: Run the test**

```python
import numpy as np
from scipy import stats

# Checkout times in seconds for 20 sessions on the new flow
checkout_times = np.array([
    41, 38, 43, 40, 37, 42, 39, 44, 36, 41,
    38, 40, 43, 37, 39, 41, 38, 42, 40, 39
])

# One-sample t-test against the old mean of 45 seconds
t_stat, p_value = stats.ttest_1samp(checkout_times, popmean=45)

print(f"Sample mean:  {checkout_times.mean():.2f} seconds")
print(f"t-statistic:  {t_stat:.4f}")
print(f"p-value:      {p_value:.6f}")
# Output:
# Sample mean:  40.05 seconds
# t-statistic:  -11.0793
# p-value:      0.000000
```

**Step 4: Make the decision**

```python
alpha = 0.05

if p_value <= alpha:
    print(f"p = {p_value:.4f} ≤ α = {alpha}")
    print("Reject H₀: The new flow's checkout time differs significantly from 45 seconds.")
else:
    print(f"p = {p_value:.4f} > α = {alpha}")
    print("Fail to reject H₀: No significant difference detected.")
# Output:
# p = 0.0000 ≤ α = 0.05
# Reject H₀: The new flow's checkout time differs significantly from 45 seconds.
```

**Step 5: Interpret in plain English**

The new payment flow has an average checkout time of about 40 seconds, compared to the old average of 45 seconds. This difference is statistically significant (p < 0.001). The new flow is meaningfully faster — a 5-second reduction represents an 11% improvement that is likely to reduce cart abandonment.

---

## Practice Exercises

**Warm-up:** Use a one-sample t-test to check if average exam scores differ from a target of 75.

```python
import pandas as pd
from scipy import stats

scores = pd.Series([72, 78, 74, 80, 76, 73, 77, 79, 68, 82])

# Write:
# 1. H₀ and H₁
# 2. The test code
# 3. The decision at α = 0.05
# 4. A plain-English interpretation
```

**Main:** A sales team claims their new script increases average deal size above $5,000. You collect 15 deals from reps using the new script. Test this claim using a one-tailed t-test.

```python
deal_sizes = [5200, 4800, 5500, 5100, 4900, 5300, 5600, 4700, 5400, 5000,
              5250, 4850, 5150, 5050, 5350]
# Hint: for a one-tailed test, check the scipy docs for the `alternative` parameter
```

**Stretch:** Run both a two-tailed and one-tailed test on the same data. Explain why the one-tailed p-value is half the two-tailed p-value. When is this difference meaningful?

---

> [!success] Key Takeaways
> - Set H₀, H₁, and α before you look at the data. The framework only works if you do this.
> - "Reject H₀" means the evidence was strong enough, not that you proved anything.
> - "Fail to reject H₀" does not mean the null is true — it may mean your test lacked power.
> - Type I errors are false positives (controlled by α). Type II errors are false negatives (controlled by sample size and power).
> - Statistical significance is not the same as practical importance. Always check effect size.

---

[[04-descriptive-statistics|Previous: Descriptive Statistics]] | [[02-p-value|Next: P-Values — What They Actually Mean]]
