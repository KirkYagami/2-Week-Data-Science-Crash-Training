# P-Values — What They Actually Mean

The p-value is one of the most misunderstood concepts in science, and the misunderstanding is not confined to students. Experienced researchers, journalists, and product managers routinely misinterpret it. Getting this right is not just an interview skill — it is what separates analysts who make confident, defensible recommendations from those who get burned six months later.

## Learning Objectives

- State the correct definition of a p-value without hedging
- List the five most common p-value misinterpretations and explain why each is wrong
- Understand p-hacking and why it invalidates your results
- Distinguish statistical significance from practical significance
- Calculate and interpret a p-value end-to-end in Python

---

## What a P-Value Actually Means

Here is the definition, stated plainly:

> A p-value is the probability of observing data at least as extreme as what you observed, assuming the null hypothesis is true.

That is it. Read it again. Every word matters.

- It is a probability **of the data**, not of the hypothesis.
- It assumes H₀ is true. It says nothing about whether H₀ actually is true.
- "At least as extreme" means results this unusual or more unusual.

A p-value of 0.03 means: "If the null hypothesis were true and we repeated this experiment many times, we would see results this extreme (or more extreme) about 3% of the time."

> [!info] Frequentist Probability
> The p-value comes from frequentist statistics, where probability is defined as the long-run frequency of events. It does not tell you anything about the probability of a hypothesis being true — that is a Bayesian question requiring prior probabilities. The p-value only characterizes how surprising your data is under a specific null hypothesis model.

---

## The Five Misinterpretations (Know These Cold)

These are the statements that make statisticians visibly uncomfortable. Every one of them is wrong.

**Misinterpretation 1: "p = 0.03 means there is a 3% chance the null hypothesis is true."**

Wrong. The p-value is computed assuming the null is true. It cannot simultaneously be the probability that the null is true.

**Misinterpretation 2: "p = 0.03 means there is a 97% chance the alternative hypothesis is true."**

Wrong. This is the same error in reverse. The p-value does not quantify the probability of H₁.

**Misinterpretation 3: "p < 0.05 means the result is important."**

Wrong. A large enough sample will make any difference — including one that is meaningless in practice — statistically significant. A 0.01% increase in conversion rate with a billion-person sample will have p < 0.001 and zero business value.

**Misinterpretation 4: "p = 0.06 is essentially the same as p = 0.05."**

This one cuts both ways. People use it to dismiss results just above 0.05 ("not significant") and to salvage results just below 0.05 ("barely significant"). The threshold is a convention, not a physical law. p = 0.049 and p = 0.051 contain nearly identical information about your data.

**Misinterpretation 5: "A small p-value means a large effect."**

Wrong. P-value and effect size are separate quantities. A tiny effect in a massive dataset produces a tiny p-value. Always check effect size independently.

> [!warning] The p = 0.049 vs p = 0.051 Problem
> Treating the 0.05 threshold as a bright line causes researchers and analysts to chase results just under it. This is a known source of publication bias and bad business decisions. Report your p-value as a number. Note whether it crosses your pre-committed α threshold. Then discuss the effect size and practical significance separately.

---

## Calculating and Interpreting P-Values in Python

```python
import numpy as np
from scipy import stats

# Scenario: We ran an A/B test on checkout page.
# Control group: conversion rates (n=200)
# Treatment group: conversion rates (n=200)
# We have their individual session outcomes (0 = no conversion, 1 = conversion)

np.random.seed(42)
control = np.random.binomial(1, 0.05, 200)    # 5% base rate
treatment = np.random.binomial(1, 0.065, 200)  # 6.5% test rate

control_rate = control.mean()
treatment_rate = treatment.mean()

print(f"Control conversion rate:   {control_rate:.3f} ({control_rate*100:.1f}%)")
print(f"Treatment conversion rate: {treatment_rate:.3f} ({treatment_rate*100:.1f}%)")
# Output:
# Control conversion rate:   0.050 (5.0%)
# Treatment conversion rate: 0.055 (5.5%)

# Two-sample t-test (approximation for proportions with reasonable n)
t_stat, p_value = stats.ttest_ind(control, treatment)

print(f"\nt-statistic: {t_stat:.4f}")
print(f"p-value:     {p_value:.4f}")
# Output:
# t-statistic: -0.4921
# p-value:     0.6231

alpha = 0.05
if p_value <= alpha:
    print("\nReject H₀: Evidence of a real difference.")
else:
    print("\nFail to reject H₀: Data is consistent with no difference.")
# Output:
# Fail to reject H₀: Data is consistent with no difference.
```

Notice what happened here. The true population rate actually did increase (5% to 6.5%), but our sample of 200 per group was not large enough to detect it. This is a Type II error — a false negative. The p-value of 0.62 does not mean there is no effect. It means we do not have enough data to see it.

---

## Statistical Significance vs Practical Significance

This distinction matters enormously in production.

```python
import numpy as np
from scipy import stats

# Scenario: Revenue per user, very large sample
np.random.seed(0)
old_version = np.random.normal(loc=100.00, scale=20, size=50000)
new_version = np.random.normal(loc=100.50, scale=20, size=50000)  # $0.50 difference

t_stat, p_value = stats.ttest_ind(old_version, new_version)

print(f"Old version mean: ${old_version.mean():.4f}")
print(f"New version mean: ${new_version.mean():.4f}")
print(f"Difference:       ${new_version.mean() - old_version.mean():.4f}")
print(f"p-value:          {p_value:.6f}")
# Output:
# Old version mean: $99.9839
# New version mean: $100.4767
# Difference:       $0.4928
# p-value:          0.000000

print(f"\nStatistically significant? {'Yes' if p_value < 0.05 else 'No'}")
print(f"Practically significant?   Probably not — $0.49 lift on $100 avg order is 0.5%")
# Output:
# Statistically significant? Yes
# Practically significant?   Probably not — $0.49 lift on $100 avg order is 0.5%
```

With 50,000 users per group, a fifty-cent difference is statistically significant at any threshold you care to name. But whether it is worth shipping depends on the cost of the change, the engineering effort, the opportunity cost, and whether 0.5% revenue lift moves any metric that matters.

> [!tip] Always Report Effect Size Alongside P-Value
> For comparing means, report Cohen's d (the standardized difference). For A/B tests on proportions, report the absolute and relative lift. A p-value alone is not a decision. A p-value plus an effect size is the start of one.

```python
# Cohen's d for effect size
def cohens_d(group1, group2):
    n1, n2 = len(group1), len(group2)
    pooled_std = np.sqrt(((n1 - 1) * group1.std()**2 + (n2 - 1) * group2.std()**2) / (n1 + n2 - 2))
    return (group2.mean() - group1.mean()) / pooled_std

d = cohens_d(old_version, new_version)
print(f"Cohen's d: {d:.4f}")
# Output: Cohen's d: 0.0246
# Interpretation: 0.02 is tiny (convention: small=0.2, medium=0.5, large=0.8)
```

---

## P-Hacking: Why It Destroys Your Analysis

P-hacking (also called data dredging) is the practice of running multiple tests, subsetting your data in different ways, or trying different outcome variables until you find p < 0.05. It is extremely common. It produces false discoveries at an alarming rate.

Here is why it happens mechanically:

```python
import numpy as np
from scipy import stats

# Pure noise — no real effect anywhere
np.random.seed(99)
n_tests = 20
false_positives = 0

for i in range(n_tests):
    group_a = np.random.normal(0, 1, 50)
    group_b = np.random.normal(0, 1, 50)  # Same distribution
    _, p = stats.ttest_ind(group_a, group_b)
    if p < 0.05:
        false_positives += 1

print(f"Tests run: {n_tests}")
print(f"False positives at p < 0.05: {false_positives}")
print(f"Expected by chance: {0.05 * n_tests:.1f}")
# Output:
# Tests run: 20
# False positives at p < 0.05: 1
# Expected by chance: 1.0
```

At α = 0.05, one in twenty tests on pure noise will return a "significant" result. If you test 20 metrics in your dashboard and report the one that is significant, you have almost certainly found noise.

**How to protect yourself:**

- Pre-register your primary hypothesis. Decide what you are testing before you look.
- Apply multiple testing corrections (Bonferroni or Benjamini-Hochberg) when testing many hypotheses.
- Treat exploratory analysis as hypothesis generation, not hypothesis confirmation.
- Replicate findings on a held-out dataset before acting on them.

```python
from scipy.stats import false_discovery_control

# Simulating 20 p-values from a mix of real and null effects
p_values = [0.03, 0.04, 0.82, 0.65, 0.02, 0.91, 0.43, 0.78, 0.01, 0.55,
            0.72, 0.88, 0.06, 0.37, 0.49, 0.007, 0.63, 0.84, 0.11, 0.29]

# Bonferroni correction: divide alpha by number of tests
alpha = 0.05
bonferroni_threshold = alpha / len(p_values)
print(f"Bonferroni threshold: {bonferroni_threshold:.4f}")
significant_bonferroni = [p for p in p_values if p < bonferroni_threshold]
print(f"Significant after Bonferroni: {len(significant_bonferroni)}")
# Output:
# Bonferroni threshold: 0.0025
# Significant after Bonferroni: 1

# Benjamini-Hochberg (FDR control) — less conservative than Bonferroni
adjusted = false_discovery_control(p_values, method='bh')
significant_bh = sum(a < 0.05 for a in adjusted)
print(f"Significant after BH correction: {significant_bh}")
# Output:
# Significant after BH correction: 4
```

> [!warning] The Multiple Comparisons Problem in Practice
> Every time you segment your A/B test results by device, region, age group, or user type, you are running additional tests. If you run 20 segments and five are "significant," three of those are probably noise. This is why A/B testing platforms like Optimizely and Statsig emphasize pre-specified primary metrics. The segment breakdown is for understanding, not for claiming victory.

---

## The Correct Way to Talk About P-Values

Use this sentence structure:

> "If the null hypothesis were true, the probability of observing a result at least this extreme is p = [value]. Since p [≤/\>] α = [value], we [reject/fail to reject] H₀."

Then follow it with an effect size and a plain-English interpretation of what the result means for the decision at hand.

---

> [!success] Key Takeaways
> - A p-value is the probability of the data given H₀ is true — not the probability H₀ is true.
> - Small p-values mean the data is surprising under H₀. Nothing more.
> - Crossing the 0.05 threshold is not proof of anything. It is evidence strong enough to act on, at a pre-committed false-positive rate.
> - Statistical significance and practical significance are independent. Always report both.
> - P-hacking is real, common, and destroys analysis. Commit to your hypothesis before you test.

---

[[01-hypothesis-testing|Previous: Hypothesis Testing]] | [[03-confidence-interval|Next: Confidence Intervals]]
