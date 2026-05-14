# 🧰 05 — Statistical Tests
## Choosing the Right Test

> [!info] Goal
> Learn common statistical tests and when to use each one.

---

## Test Selection Guide

| Question | Data Type | Common Test |
|----------|-----------|-------------|
| Is one mean different from a known value? | Numeric | One-sample t-test |
| Are two group means different? | Numeric + 2 groups | Independent t-test |
| Before vs after for same subjects? | Paired numeric | Paired t-test |
| Are 3+ group means different? | Numeric + 3+ groups | ANOVA |
| Are two categorical variables related? | Categorical | Chi-square test |
| Are two numeric variables related? | Numeric | Correlation test |

---

## One-Sample t-test

```python
from scipy import stats

t_stat, p_value = stats.ttest_1samp(sample, popmean=50)
```

Use when comparing sample mean to a known target.

---

## Independent Two-Sample t-test

```python
t_stat, p_value = stats.ttest_ind(group_a, group_b)
```

Use when comparing means of two independent groups.

Example:

- average spend of male vs female customers
- old design vs new design users

---

## Paired t-test

```python
t_stat, p_value = stats.ttest_rel(before, after)
```

Use when the same subjects are measured twice.

Example:

- before training vs after training scores
- before campaign vs after campaign sales for same stores

---

## ANOVA

```python
f_stat, p_value = stats.f_oneway(group_1, group_2, group_3)
```

Use when comparing means across 3 or more groups.

Example:

- average spend across Bronze, Silver, and Gold customers

---

## Chi-Square Test

```python
import pandas as pd
from scipy import stats

table = pd.crosstab(df["gender"], df["purchased"])
chi2, p_value, dof, expected = stats.chi2_contingency(table)
```

Use for relationships between categorical variables.

Example:

- gender vs purchase
- region vs churn

---

## Correlation Test

```python
r, p_value = stats.pearsonr(df["ad_spend"], df["sales"])
```

Use to test linear relationship between two numeric variables.

---

## Assumptions to Check

Different tests have different assumptions, but common checks include:

- independent observations
- appropriate data type
- no extreme outliers
- roughly normal numeric data for t-tests
- enough sample size

When assumptions fail, consider non-parametric tests.

---

## Non-Parametric Alternatives

| Parametric Test | Alternative |
|-----------------|-------------|
| Independent t-test | Mann-Whitney U test |
| Paired t-test | Wilcoxon signed-rank test |
| ANOVA | Kruskal-Wallis test |
| Pearson correlation | Spearman correlation |

---

## Common Mistakes

- Using many tests without correction.
- Choosing a test after looking at results.
- Using t-test for 3+ groups instead of ANOVA.
- Treating statistical significance as business importance.
- Ignoring assumptions.

---

## Practice

Create two groups:

```python
old_design = [100, 102, 98, 101, 99]
new_design = [108, 110, 107, 111, 109]
```

Tasks:

- choose the test
- run the test
- interpret p-value
- explain if result is practically meaningful

---

## Interview Questions

**Q1:** Which test compares two independent means?

> Independent two-sample t-test.

**Q2:** Which test compares categorical variables?

> Chi-square test.

**Q3:** When do you use ANOVA?

> When comparing means across three or more groups.

---

## ✅ Key Takeaways

- Test choice depends on the question and data types.
- t-tests compare means.
- Chi-square tests categorical relationships.
- ANOVA compares 3+ group means.
- Always interpret p-value with context and effect size.

---

## 🔗 What's Next?

➡️ [[06-interview-prep]] — Review inferential statistics interview questions
