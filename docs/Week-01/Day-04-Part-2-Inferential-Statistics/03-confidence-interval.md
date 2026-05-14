# 📐 03 — Confidence Interval
## Estimating with Uncertainty

> [!info] Goal
> Learn how confidence intervals express uncertainty around sample estimates.

---

## What is a Confidence Interval?

A confidence interval is a range of plausible values for a population parameter.

Example:

```text
Average delivery time: 28.5 minutes
95% CI: [27.2, 29.8]
```

This means the estimate is 28.5, but uncertainty suggests the true average may reasonably be between 27.2 and 29.8.

---

## Why Confidence Intervals Matter

Point estimates alone can be misleading.

```text
Sample mean = 50
```

But how certain are we?

```text
95% CI = [49, 51]     # precise
95% CI = [30, 70]     # uncertain
```

---

## Confidence Level

Common confidence levels:

| Confidence | Meaning |
|------------|---------|
| 90% | Narrower, less certainty |
| 95% | Common default |
| 99% | Wider, more certainty |

Higher confidence creates a wider interval.

---

## Confidence Interval for Mean

```python
import numpy as np
from scipy import stats

data = np.array([28, 31, 29, 27, 30, 26, 32, 28, 29, 27])

mean = data.mean()
sem = stats.sem(data)
ci = stats.t.interval(
    confidence=0.95,
    df=len(data) - 1,
    loc=mean,
    scale=sem
)

print(mean)
print(ci)
```

---

## Interpretation

Good:

```text
We are 95% confident that the true population mean lies between X and Y.
```

Avoid:

```text
There is a 95% probability the true mean is in this interval.
```

The strict statistical meaning is subtle, but the first sentence is acceptable for practical communication.

---

## What Makes Intervals Wider?

- smaller sample size
- higher variability
- higher confidence level

What makes intervals narrower?

- larger sample size
- lower variability
- lower confidence level

---

## Confidence Intervals vs P-Values

| Tool | Answers |
|------|---------|
| P-value | Is there evidence against H0? |
| Confidence interval | What range of values is plausible? |

Confidence intervals are often more informative because they show effect size and uncertainty.

---

## Common Mistakes

- Reporting only p-values without intervals.
- Ignoring wide confidence intervals.
- Saying confidence interval proves the exact true value.
- Forgetting sample size affects interval width.

---

## Practice

Use this data:

```python
scores = np.array([72, 78, 74, 80, 76, 73, 77, 79])
```

Tasks:

- calculate mean
- calculate 95% confidence interval
- interpret the interval

---

## Interview Questions

**Q1:** What does a confidence interval show?

> A plausible range for a population value based on sample data.

**Q2:** What happens when sample size increases?

> The interval usually becomes narrower.

**Q3:** Why are confidence intervals useful?

> They show uncertainty and effect size, not just significance.

---

## ✅ Key Takeaways

- Confidence intervals express uncertainty around estimates.
- Wider intervals mean more uncertainty.
- Larger samples usually produce narrower intervals.
- Confidence intervals complement p-values.

---

## 🔗 What's Next?

➡️ [[04-correlation]] — Measure relationships between variables
