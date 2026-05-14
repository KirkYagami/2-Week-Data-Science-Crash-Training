# 🎯 02 — P-Value
## Evidence Against the Null

> [!info] Goal
> Understand what a p-value means, what it does not mean, and how to explain it in interviews.

---

## What is a P-Value?

A **p-value** tells us how surprising the observed data would be if the null hypothesis were true.

Small p-value means:

```text
The observed result is unlikely under H0.
```

Large p-value means:

```text
The observed result is not surprising under H0.
```

---

## Decision Rule

```text
if p-value <= alpha:
    reject H0
else:
    fail to reject H0
```

Common alpha:

```text
alpha = 0.05
```

---

## Example

```python
from scipy import stats

old = [100, 102, 98, 101, 99]
new = [108, 110, 107, 111, 109]

t_stat, p_value = stats.ttest_ind(old, new)

print(p_value)
```

If `p_value <= 0.05`, the difference is statistically significant.

---

## What P-Value Is Not

A p-value is **not**:

- probability that the null hypothesis is true
- probability that the alternative hypothesis is true
- size of the effect
- proof of a claim
- measure of business importance

---

## Statistical vs Practical Significance

With a huge sample, a tiny difference can become statistically significant.

Example:

```text
Average revenue increased from 1000.00 to 1000.50
```

This may be statistically significant but not useful for business.

Always ask:

- Is it statistically significant?
- Is the effect large enough to matter?
- What is the cost of acting on it?

---

## P-Value Interpretation Template

Use this sentence:

```text
If the null hypothesis were true, the probability of observing a result this extreme or more extreme is p.
```

Example:

```text
If there were truly no difference between designs, seeing a difference this large would be unlikely.
```

---

## Common Mistakes

- Saying "there is a 95% chance the result is true."
- Treating `p = 0.049` as magic and `p = 0.051` as useless.
- Ignoring effect size.
- Testing many hypotheses without correction.

---

## Practice

Given:

```text
p-value = 0.03
alpha = 0.05
```

Questions:

- reject or fail to reject `H0`?
- is this proof?
- what else should you check?

Answers:

- reject `H0`
- no, it is evidence, not proof
- effect size, sample size, assumptions, and business impact

---

## Interview Questions

**Q1:** What does a small p-value mean?

> The observed data would be unlikely if the null hypothesis were true.

**Q2:** Does p-value measure effect size?

> No. It measures evidence against the null, not the size or importance of the effect.

**Q3:** What does `p < 0.05` mean?

> The result is statistically significant at the 5% significance level.

---

## ✅ Key Takeaways

- P-value measures surprise under the null hypothesis.
- Small p-values provide evidence against `H0`.
- P-values do not prove truth.
- Always combine p-value with effect size and context.

---

## 🔗 What's Next?

➡️ [[03-confidence-interval]] — Estimate uncertainty around values
