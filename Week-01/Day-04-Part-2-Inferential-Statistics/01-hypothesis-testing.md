# 🧪 01 — Hypothesis Testing
## Making Decisions from Data

> [!info] Goal
> Learn how hypothesis testing helps decide whether an observed result is likely real or due to random chance.

---

## What is Hypothesis Testing?

Hypothesis testing is a statistical framework for testing claims using sample data.

Examples:

- Did a new website design increase conversion?
- Is average delivery time below 30 minutes?
- Are two customer groups spending differently?
- Does a medicine reduce recovery time?

---

## Null and Alternative Hypotheses

| Hypothesis | Meaning |
|------------|---------|
| Null hypothesis `H0` | No effect, no difference, default assumption |
| Alternative hypothesis `H1` | There is an effect or difference |

Example:

```text
H0: New design has same conversion rate as old design.
H1: New design has a different conversion rate.
```

---

## Basic Workflow

1. Define `H0` and `H1`.
2. Choose significance level, usually `alpha = 0.05`.
3. Choose the correct statistical test.
4. Calculate test statistic and p-value.
5. Compare p-value with alpha.
6. Interpret in business language.

---

## Decision Rule

```text
If p-value <= alpha: reject H0
If p-value > alpha: fail to reject H0
```

Important: we say **fail to reject H0**, not "prove H0 true."

---

## Example: One-Sample t-test

Question: is average delivery time different from 30 minutes?

```python
import pandas as pd
from scipy import stats

delivery_times = pd.Series([28, 31, 29, 27, 30, 26, 32, 28, 29, 27])

t_stat, p_value = stats.ttest_1samp(delivery_times, popmean=30)

print(t_stat)
print(p_value)
```

Interpretation:

```python
alpha = 0.05

if p_value <= alpha:
    print("Reject H0")
else:
    print("Fail to reject H0")
```

---

## One-Tailed vs Two-Tailed Tests

| Test Type | Question |
|-----------|----------|
| Two-tailed | Is there any difference? |
| One-tailed | Is it greater or smaller in a specific direction? |

Examples:

```text
Two-tailed: average is different from 30
One-tailed: average is less than 30
```

Use one-tailed tests only when the direction is decided before looking at results.

---

## Type I and Type II Errors

| Error | Meaning |
|-------|---------|
| Type I error | False positive: rejecting true H0 |
| Type II error | False negative: failing to reject false H0 |

`alpha` controls the Type I error risk.

---

## Common Mistakes

- Treating p-value as probability that `H0` is true.
- Saying a test "proves" something.
- Choosing one-tailed tests after seeing the data.
- Ignoring sample size and practical significance.
- Testing without a clear hypothesis.

---

## Practice

Use a one-sample t-test to check if average score differs from 75:

```python
scores = pd.Series([72, 78, 74, 80, 76, 73, 77, 79])
```

Write:

- `H0`
- `H1`
- test result
- interpretation

---

## Interview Questions

**Q1:** What is the null hypothesis?

> The default assumption of no effect or no difference.

**Q2:** What does it mean to reject `H0`?

> The sample provides enough evidence against the null hypothesis at the chosen alpha level.

**Q3:** What is alpha?

> The significance level, commonly 0.05, representing acceptable Type I error risk.

---

## ✅ Key Takeaways

- Hypothesis testing compares evidence against a default assumption.
- `H0` means no effect; `H1` means effect or difference.
- P-value helps decide whether to reject `H0`.
- Statistical significance is not always practical significance.

---

## 🔗 What's Next?

➡️ [[02-p-value]] — Understand p-values clearly
