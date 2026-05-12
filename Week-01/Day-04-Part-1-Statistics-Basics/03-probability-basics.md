# 🎲 03 — Probability Basics
## Understanding Uncertainty

> [!info] Goal
> Learn the probability ideas used in statistics, machine learning, and data interpretation.

---

## What is Probability?

Probability measures how likely an event is.

```text
Probability = favorable outcomes / total possible outcomes
```

Probability ranges from `0` to `1`.

| Probability | Meaning |
|-------------|---------|
| `0` | Impossible |
| `0.5` | Even chance |
| `1` | Certain |

---

## Simple Example

For a fair coin:

```text
P(heads) = 1 / 2 = 0.5
```

For a six-sided die:

```text
P(rolling a 4) = 1 / 6
```

---

## Events

An **event** is something that can happen.

Examples:

- customer churns
- email is spam
- model prediction is correct
- sales exceed target

---

## Complement

The complement is the probability that an event does not happen.

```text
P(not A) = 1 - P(A)
```

If churn probability is `0.2`, non-churn probability is:

```text
1 - 0.2 = 0.8
```

---

## AND Rule

For independent events:

```text
P(A and B) = P(A) * P(B)
```

Example:

```text
P(two heads) = 0.5 * 0.5 = 0.25
```

---

## OR Rule

For mutually exclusive events:

```text
P(A or B) = P(A) + P(B)
```

Example: rolling a 1 or 2 on a die:

```text
1/6 + 1/6 = 2/6
```

For general events:

```text
P(A or B) = P(A) + P(B) - P(A and B)
```

---

## Conditional Probability

Conditional probability asks: what is the probability of A given B?

```text
P(A | B)
```

Example:

```text
P(customer buys | customer clicked ad)
```

This is central to real Data Science.

---

## Simulation with Python

```python
import numpy as np

np.random.seed(42)
coin_flips = np.random.choice(["H", "T"], size=1000)

prob_heads = (coin_flips == "H").mean()
print(prob_heads)
```

With enough trials, the simulated probability approaches the true probability.

---

## Data Science Uses

Probability appears in:

- classification models
- A/B testing
- confidence intervals
- risk scoring
- Bayesian reasoning
- sampling

---

## Common Mistakes

- Confusing probability with certainty.
- Assuming events are independent without checking.
- Forgetting probabilities must be between 0 and 1.
- Treating model probability as a guarantee.

---

## Practice

Tasks:

- calculate probability of rolling an even number on a die
- calculate probability of two heads in two coin flips
- simulate 10,000 coin flips in NumPy
- estimate probability of heads from simulation

---

## Interview Questions

**Q1:** What is conditional probability?

> Probability of one event happening given that another event has happened.

**Q2:** What does independent mean?

> One event does not affect the probability of the other.

**Q3:** Can probability be greater than 1?

> No. Probability ranges from 0 to 1.

---

## ✅ Key Takeaways

- Probability measures uncertainty.
- Complements use `1 - P(A)`.
- Independent AND events multiply.
- Conditional probability is written as `P(A | B)`.
- Many Data Science methods are probability-based.

---

## 🔗 What's Next?

➡️ [[04-distributions]] — Learn common data shapes
