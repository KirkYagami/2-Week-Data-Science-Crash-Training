# 🎯 06 — Interview Prep: Inferential Statistics

> [!info] Goal
> Practice explaining inferential statistics clearly and confidently.

---

## Core Questions

**Q1:** What is inferential statistics?

> Inferential statistics uses sample data to make conclusions about a larger population.

---

**Q2:** What is a hypothesis test?

> A method for deciding whether sample evidence is strong enough to reject a default assumption.

---

**Q3:** What are `H0` and `H1`?

> `H0` is the null hypothesis, usually no effect or no difference. `H1` is the alternative hypothesis, usually an effect or difference.

---

**Q4:** What is a p-value?

> The probability of observing a result this extreme or more extreme if the null hypothesis were true.

---

**Q5:** What does `p < 0.05` mean?

> The result is statistically significant at the 5% level, so we reject the null hypothesis.

---

**Q6:** Does p-value prove the result is true?

> No. It provides evidence against the null hypothesis, not proof.

---

**Q7:** What is a confidence interval?

> A range of plausible values for a population parameter.

---

**Q8:** What is Type I error?

> A false positive: rejecting the null hypothesis when it is actually true.

---

**Q9:** What is Type II error?

> A false negative: failing to reject the null hypothesis when it is false.

---

**Q10:** What is correlation?

> A measure of the strength and direction of a relationship between variables.

---

**Q11:** Does correlation imply causation?

> No. There may be confounding variables or coincidence.

---

**Q12:** When would you use a t-test?

> To compare means, such as one sample against a known value or two group means.

---

**Q13:** When would you use chi-square test?

> To test whether two categorical variables are associated.

---

**Q14:** When would you use ANOVA?

> To compare means across three or more groups.

---

## Scenario Questions

**Q15:** An A/B test gives `p = 0.03`. What do you conclude?

> If alpha is 0.05, reject the null hypothesis. Then check effect size and business impact before recommending action.

---

**Q16:** A result is statistically significant but effect size is tiny. What should you say?

> The effect may be real but not practically important. Business value should guide the decision.

---

**Q17:** A confidence interval is very wide. What does that mean?

> The estimate is uncertain, often due to small sample size or high variability.

---

**Q18:** You test 100 metrics and 5 are significant. What is the concern?

> Multiple testing increases false positives. Use corrections or predefine key metrics.

---

## Quick Test Selection

| Situation | Test |
|-----------|------|
| One sample mean vs target | One-sample t-test |
| Two independent group means | Independent t-test |
| Same group before/after | Paired t-test |
| Three or more group means | ANOVA |
| Two categorical variables | Chi-square |
| Two numeric variables | Pearson/Spearman correlation |

---

## Practice Prompt

Explain this in plain English:

```text
An experiment comparing old and new checkout pages produced p = 0.02.
The new page increased conversion from 5.0% to 5.2%.
```

Strong answer:

> The result is statistically significant at alpha 0.05, but the absolute lift is only 0.2 percentage points. Before shipping, I would check sample size, revenue impact, implementation cost, and whether the result holds across important user segments.

---

## ✅ Final Checklist

- [ ] Explain p-value correctly
- [ ] Explain confidence intervals
- [ ] Choose common statistical tests
- [ ] Explain correlation vs causation
- [ ] Discuss statistical vs practical significance
- [ ] Mention assumptions and sample size

---

## 🔗 Next

➡️ [[../Day-05-Part-1-SQL-for-Data-Science/01-select-and-where|SQL for Data Science]]
