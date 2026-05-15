# Probability Basics

Every classification model outputs a probability. Every A/B test conclusion rests on probability. Bayes' theorem powers spam filters, medical diagnostics, and recommendation engines. If you want to understand why your model says "70% chance of churn," you need a firm grip on what that 70% actually means.

## Learning Objectives

- Define probability, sample space, and events precisely
- Apply the complement, AND, and OR rules correctly
- Compute conditional probability and explain it in plain language
- Distinguish independence from correlation
- Apply Bayes' theorem to a real problem (medical tests, spam detection)
- Simulate probability in Python and verify theoretical results numerically

---

## What Is Probability?

Probability quantifies uncertainty. It assigns a number between 0 and 1 to how likely an event is to occur.

> [!info] Definition
> `P(event) = (number of favorable outcomes) / (total number of possible outcomes)`
> This only applies when all outcomes are equally likely. For unequal outcomes, probability comes from data or a model.

| Value | Meaning |
|-------|---------|
| 0 | Impossible — will never occur |
| 0.5 | Equal chance either way |
| 1 | Certain — will always occur |

The **sample space** (S) is the set of all possible outcomes. An **event** (A) is any subset of the sample space.

```python
import numpy as np

# Rolling a six-sided die
sample_space = [1, 2, 3, 4, 5, 6]
event_even   = [2, 4, 6]  # rolling an even number

p_even = len(event_even) / len(sample_space)
print(f"P(even) = {p_even:.4f}")  # Output: 0.3333
```

---

## The Complement Rule

The complement of event A is everything in the sample space that is NOT A.

> [!info] Complement Formula
> `P(not A) = 1 - P(A)`

```python
p_churn     = 0.15   # 15% of customers churn
p_no_churn  = 1 - p_churn

print(f"P(churn):    {p_churn:.2f}")     # Output: 0.15
print(f"P(no churn): {p_no_churn:.2f}")  # Output: 0.85
```

> [!tip] Complement is one of your most useful tools
> Sometimes the probability of the complement is much easier to compute than the event itself. "What is the probability of getting at least one head in 5 flips?" is tedious directly. P(at least one head) = 1 - P(no heads) = 1 - (0.5)^5 = 0.969. Always check if the complement is simpler.

---

## The AND Rule — Intersection

P(A and B) is the probability that both A and B occur simultaneously.

For **independent** events (where one outcome does not affect the other):

> [!info] AND Rule (Independent Events)
> `P(A ∩ B) = P(A) × P(B)`

```python
p_heads     = 0.5   # one coin flip
p_two_heads = p_heads * p_heads   # two independent flips

print(f"P(two heads) = {p_two_heads:.4f}")  # Output: 0.2500

# Verify by simulation
np.random.seed(42)
flips = np.random.choice(["H", "T"], size=(100_000, 2))  # 100,000 pairs of flips
both_heads = np.sum((flips[:, 0] == "H") & (flips[:, 1] == "H")) / len(flips)

print(f"Simulated P(two heads) = {both_heads:.4f}")  # Output: ~0.2500
```

> [!warning] Independence is an assumption, not a given
> Two events are independent only if knowing one occurred gives you no information about the other. Coin flips are independent. Whether a customer clicks an ad and whether they buy are NOT independent — click rates are correlated with purchase intent. Assuming independence when it does not hold produces wrong probability estimates.

---

## The OR Rule — Union

P(A or B) is the probability that at least one of A or B occurs.

For **mutually exclusive** events (they cannot both happen at the same time):

> [!info] OR Rule (Mutually Exclusive)
> `P(A ∪ B) = P(A) + P(B)`

For **general** events (both can occur together):

> [!info] OR Rule (General — Addition Rule)
> `P(A ∪ B) = P(A) + P(B) - P(A ∩ B)`
> You subtract P(A ∩ B) because the overlap gets counted twice if you just add P(A) and P(B).

```python
# P(rolling a 2 or rolling an even number) — NOT mutually exclusive
# because rolling a 2 is also rolling an even number
p_rolling_2    = 1/6
p_rolling_even = 3/6
p_rolling_2_and_even = 1/6  # 2 is both — the intersection

p_2_or_even = p_rolling_2 + p_rolling_even - p_rolling_2_and_even
print(f"P(2 or even) = {p_2_or_even:.4f}")  # Output: 0.5000 (= 3/6, just the even numbers)

# Mutually exclusive example
p_rolling_1 = 1/6
p_rolling_6 = 1/6
p_1_or_6 = p_rolling_1 + p_rolling_6  # mutually exclusive, no overlap
print(f"P(1 or 6) = {p_1_or_6:.4f}")  # Output: 0.3333
```

---

## Conditional Probability

Conditional probability answers: given that we know B happened, what is the probability that A also happened?

> [!info] Conditional Probability Formula
> `P(A | B) = P(A ∩ B) / P(B)`
> Read as: "Probability of A given B."

```python
# In a dataset of 1000 website visitors:
# 200 clicked an ad
# 80 of those 200 also made a purchase
# 40 made a purchase without clicking the ad

total         = 1000
clicked       = 200
purchased_and_clicked = 80

p_clicked       = clicked / total               # 0.20
p_purchase_given_click = purchased_and_clicked / clicked  # 0.40

print(f"P(clicked ad):          {p_clicked:.2f}")             # Output: 0.20
print(f"P(purchase | clicked):  {p_purchase_given_click:.2f}")# Output: 0.40

# Compare: unconditional purchase rate
total_purchases = 80 + 40  # clicked + didn't click
p_purchase = total_purchases / total
print(f"P(purchase overall):    {p_purchase:.2f}")            # Output: 0.12
```

Knowing the customer clicked the ad changes the purchase probability from 12% to 40%. That gap is what makes ad targeting valuable — it selects for people more likely to buy.

> [!tip] Conditional probability is the foundation of ML
> Every classification model is estimating a conditional probability: P(label = 1 | features). Logistic regression does it directly. Neural networks do it implicitly. Understanding conditional probability means understanding what your model is actually computing.

---

## Independence vs Correlation

Two events are **independent** if knowing one occurred tells you nothing about the other.

> [!info] Independence Condition
> `P(A | B) = P(A)` — B gives no information about A.
> Equivalently: `P(A ∩ B) = P(A) × P(B)`

```python
# Test independence: does ad click predict purchase?
p_purchase          = 0.12  # from above
p_purchase_clicked  = 0.40  # from above

# If independent, P(purchase | click) should equal P(purchase)
print(f"Independent? {np.isclose(p_purchase, p_purchase_clicked)}")  # Output: False
# They are NOT independent — clicking is informative about purchasing
```

> [!warning] Independence is not the same as zero correlation
> Statistical independence means no relationship of any kind. Zero correlation (Pearson's r = 0) only means no LINEAR relationship. Two variables can have a strong non-linear relationship and still have zero correlation. Do not conflate the two.

---

## Bayes' Theorem

Bayes' theorem lets you update a probability estimate when new evidence arrives. It is one of the most important ideas in data science.

> [!info] Bayes' Theorem
> `P(A | B) = P(B | A) × P(A) / P(B)`
>
> - `P(A)` — **prior probability**: your initial belief before seeing evidence
> - `P(B | A)` — **likelihood**: probability of seeing the evidence if A is true
> - `P(B)` — **marginal probability**: total probability of seeing the evidence
> - `P(A | B)` — **posterior probability**: updated belief after seeing evidence

### Example 1: Medical Test

A disease affects 1% of the population. A test for it is 95% accurate (detects it 95% of the time when present) and has a 5% false positive rate (flags it 5% of the time when absent).

You test positive. What is the probability you actually have the disease?

Most people guess high — 90%+. The correct answer is around 16%.

```python
# Define the probabilities
p_disease          = 0.01   # prevalence: 1% of population has it
p_no_disease       = 1 - p_disease   # 0.99

p_positive_given_disease    = 0.95   # sensitivity
p_positive_given_no_disease = 0.05   # false positive rate

# Total probability of testing positive (law of total probability)
p_positive = (p_positive_given_disease * p_disease +
              p_positive_given_no_disease * p_no_disease)

# Bayes' theorem
p_disease_given_positive = (p_positive_given_disease * p_disease) / p_positive

print(f"P(disease | positive test) = {p_disease_given_positive:.4f}")
# Output: 0.1610 — about 16%
```

Why so low? Because the disease is rare (1%), so among all positive tests, most come from the 99% of healthy people who trigger a false positive. The prior probability (1%) dominates.

> [!warning] Base rate neglect is a real cognitive bias
> Doctors, lawyers, and managers routinely overestimate the posterior probability after a positive test because they ignore the prior. This is called the base rate fallacy. Bayes' theorem is the cure.

### Example 2: Spam Filter

A spam filter observes that the word "FREE" appears in 90% of spam emails and 2% of legitimate emails. If 30% of all incoming emails are spam, and an email contains "FREE," what is the probability it is spam?

```python
p_spam          = 0.30
p_not_spam      = 0.70

p_free_given_spam     = 0.90
p_free_given_not_spam = 0.02

# Total probability of seeing "FREE"
p_free = (p_free_given_spam * p_spam) + (p_free_given_not_spam * p_not_spam)

# Posterior probability
p_spam_given_free = (p_free_given_spam * p_spam) / p_free

print(f"P(spam | 'FREE') = {p_spam_given_free:.4f}")
# Output: 0.9508 — about 95% likely spam
```

The word "FREE" updates the prior from 30% to 95%. Naive Bayes classifiers chain this calculation across all words in an email. This is literally how early spam filters worked.

---

## Simulating Probability in Python

Simulation lets you verify theoretical results and model situations too complex for formulas.

```python
np.random.seed(42)
n_trials = 100_000

# P(at least one 6 in three die rolls)
rolls = np.random.randint(1, 7, size=(n_trials, 3))
at_least_one_six = np.any(rolls == 6, axis=1).mean()

# Theoretical: 1 - P(no 6 in 3 rolls) = 1 - (5/6)^3
theoretical = 1 - (5/6)**3

print(f"Simulated:   {at_least_one_six:.4f}")  # Output: ~0.4213
print(f"Theoretical: {theoretical:.4f}")        # Output: 0.4213
```

```python
# Simulate conditional probability directly
# P(both even | first is even) — rolling two dice
dice_a = np.random.randint(1, 7, n_trials)
dice_b = np.random.randint(1, 7, n_trials)

first_is_even = dice_a % 2 == 0
both_even     = (dice_a % 2 == 0) & (dice_b % 2 == 0)

# Conditional: filter to cases where first is even, then check both
p_both_even_given_first = both_even[first_is_even].mean()

print(f"P(both even | first even) = {p_both_even_given_first:.4f}")  # Output: ~0.5000
# Makes sense: given first is even, second is still 50/50
```

> [!tip] Simulation is your sanity check
> Whenever you derive a probability analytically, simulate it in 100,000 trials and check whether the numbers match. If they do, your formula is right. If they do not, find the error before building something on top of a wrong assumption.

---

## Probability in Data Science

You use probability constantly in data science, often without naming it:

| Application | Probability Concept |
|-------------|---------------------|
| Logistic regression output | P(class = 1 \| features) |
| A/B test p-value | P(observed data \| null hypothesis is true) |
| Naive Bayes classifier | Bayes' theorem applied to features |
| Random forests (bootstrap) | Sampling with replacement |
| Dropout in neural networks | Each neuron dropped with probability p |
| Confidence interval | P(interval contains true value) = 95% |

---

> [!success] Key Takeaways
> - Probability ranges from 0 to 1. It measures likelihood, not certainty.
> - Complement: `P(not A) = 1 - P(A)`. Always check if the complement is simpler.
> - AND (independent): `P(A ∩ B) = P(A) × P(B)`. Independence is an assumption — verify it.
> - OR (general): `P(A ∪ B) = P(A) + P(B) - P(A ∩ B)`. Subtract the overlap.
> - Conditional probability `P(A | B)` is the foundation of machine learning.
> - Bayes' theorem lets you update beliefs with new evidence. The prior matters enormously.
> - Simulate to verify your probability calculations.

---

[[02-variance-and-std|Back: Variance and Standard Deviation]] | [[04-distributions|Next: Distributions]]
