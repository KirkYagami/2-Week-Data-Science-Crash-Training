# Interview Prep — Inferential Statistics

Inferential statistics is one of the highest-signal topic areas in data science interviews. It reveals whether you have thought carefully about uncertainty, causation, and the gap between statistical significance and real-world importance — or whether you have just memorized formulas. These questions appear in analyst, data scientist, and ML engineer interviews at every level. The answers below reflect how an experienced practitioner thinks, not just what a textbook says.

## How to Use This File

Read each question. Close the answer. Write or say your answer out loud. Then open the collapsible block and compare. The goal is not to memorize — it is to develop the reasoning so you can answer variations of each question in real time.

---

## Foundational Concepts

**Q1: What is the difference between descriptive and inferential statistics?**

??? "Show answer"
    Descriptive statistics summarize and describe the data you have — mean, median, standard deviation, histograms. They make no claims beyond the sample.

    Inferential statistics use sample data to make claims about a larger population, and to quantify how confident you are in those claims. A t-test, confidence interval, or regression coefficient are all inferential — they are generalizing from the sample to something broader.

    In practice: when you report "our average order value last month was $85," that is descriptive. When you say "we estimate the true population mean order value is between $81 and $89 with 95% confidence," that is inferential.

---

**Q2: What is the null hypothesis, and why do we start with it?**

??? "Show answer"
    The null hypothesis (H₀) is the default position: no effect, no difference, no relationship. We start with it because hypothesis testing is a framework for disproving, not proving. We cannot prove H₁ directly — we can only ask: "Is the data inconsistent enough with H₀ to make H₀ implausible?"

    Think of it like a court trial. H₀ is "not guilty." The burden of proof is on the data. We require strong evidence before overturning the default assumption.

    Starting with H₀ also forces us to be explicit about what "nothing happening" looks like, which prevents motivated reasoning — testing until we find the result we wanted.

---

**Q3: What is a p-value? Give me the definition precisely.**

??? "Show answer"
    A p-value is the probability of observing data at least as extreme as what was observed, assuming the null hypothesis is true.

    Three important clarifications:

    1. It is a probability of the data, not of the hypothesis. P(data | H₀ is true), not P(H₀ is true | data).
    2. "At least as extreme" means results this unusual or more unusual under H₀.
    3. It does not tell you the probability that H₁ is true, the size of the effect, or whether the result is practically important.

    A common analogy: imagine you flip a fair coin 10 times and get 9 heads. The p-value is the probability of getting 9 or more heads if the coin were truly fair. It is about 1%. That does not mean there is a 1% chance the coin is fair — it means this data is unlikely under the assumption of fairness.

---

**Q4: What does p < 0.05 actually mean?**

??? "Show answer"
    It means: if the null hypothesis were true, we would observe data this extreme or more extreme less than 5% of the time.

    It does not mean:
    - There is a 95% chance the result is real
    - There is a 5% chance the null is true
    - The effect is large or practically important
    - The result is guaranteed to replicate

    p < 0.05 is a threshold for a decision, not a certificate of truth. You committed to α = 0.05 before running the test, meaning you accepted a 5% chance of a false positive. A result crossing that threshold is worth acting on at that error rate — nothing more.

    The value of 0.05 is a convention from Ronald Fisher in the 1920s. Different fields use different thresholds: genomics typically uses p < 5 × 10⁻⁸ because they test millions of variants simultaneously. Physics uses p < 3 × 10⁻⁷ ("5 sigma") for particle discoveries.

---

**Q5: What are Type I and Type II errors? Which is worse?**

??? "Show answer"
    A Type I error (false positive) is rejecting H₀ when it is actually true. You conclude an effect exists when it does not.

    A Type II error (false negative) is failing to reject H₀ when it is actually false. You miss a real effect.

    Which is worse depends entirely on the context:

    - **Medical drug approval:** a Type I error means approving an ineffective drug. Patients may take it instead of better treatments. A Type II error means rejecting an effective drug — patients lose access to something that works. In this case, both matter, and regulators balance them carefully.

    - **Spam filter:** a Type I error means blocking legitimate email. A Type II error means letting spam through. Most users find a missed spam more acceptable than a blocked important email — so Type I error is worse here.

    - **A/B testing a UI change:** a Type I error means shipping a change that has no real effect, wasting engineering effort. A Type II error means abandoning a change that actually works, leaving value on the table.

    The tradeoff is controlled by α (Type I rate) and power (1 - Type II rate). Making α smaller reduces false positives but requires larger samples to maintain power.

---

**Q6: What is statistical power?**

??? "Show answer"
    Power is the probability that a test correctly detects a real effect when one exists. It is the complement of the Type II error rate: Power = 1 - β.

    Power depends on four factors:
    1. **Sample size (n):** larger samples → more power
    2. **Effect size:** larger real effects are easier to detect
    3. **Significance level (α):** higher α → more power, but more false positives
    4. **Variability (σ):** less noisy data → more power

    The conventional minimum is 80% power — meaning you accept at most a 20% chance of missing a real effect. This is not a law; it is a convention. Some fields require 90% or 95%.

    In practice: before running an important experiment, do a power analysis to determine the sample size needed to achieve 80% power for a given effect size. An underpowered study that finds "no significant result" is nearly uninterpretable — you cannot tell whether there is no effect or whether you just did not collect enough data.

---

**Q7: What is a confidence interval and how is it different from a p-value?**

??? "Show answer"
    A 95% confidence interval is a range constructed such that, if you repeated the sampling procedure many times, 95% of the intervals would contain the true population parameter.

    The critical nuance: the CI is a statement about the procedure, not about any specific interval. Once calculated, the true parameter either is or is not inside it — you do not know which.

    The practical interpretation: "We are 95% confident the true mean lies between A and B" is widely accepted and useful, even though it is not technically precise.

    How CIs differ from p-values:
    - A p-value gives a binary decision signal (significant or not)
    - A CI gives a range — it shows where the effect plausibly lives and how uncertain you are
    - A CI carries effect size information; a p-value does not
    - If a 95% CI excludes zero (for a difference), the test is significant at α = 0.05

    If I could only report one, I would report the CI. It contains everything the p-value does plus the effect magnitude and uncertainty bounds.

---

**Q8: What does it mean when a confidence interval is very wide?**

??? "Show answer"
    A wide confidence interval means high uncertainty in the estimate. This usually happens because:

    - The sample size is small (the most common cause)
    - The data has high variability
    - You requested a higher confidence level (99% vs 90%)

    A wide CI is not a failure — it is honest reporting. It is telling you the data is not precise enough to narrow down the true value much.

    What to do: collect more data. Width shrinks as 1/√n, so to cut the width in half you need four times as many observations.

    In practice: a CI of [−0.1%, 5.2%] for a conversion rate lift means you cannot rule out that the effect is zero OR that it is a 5% increase. You do not have enough data to tell. Do not ship — run a larger experiment.

---

**Q9: What is p-hacking? How do you guard against it?**

??? "Show answer"
    P-hacking (also called data dredging or fishing) is the practice of running multiple tests, trying different subsets, or adjusting the analysis until p < 0.05 appears. It exploits the fact that at α = 0.05, 1 in 20 tests on pure noise will produce a "significant" result.

    Common forms:
    - Testing many outcome variables and reporting only the significant one
    - Segmenting results by age, gender, region until one segment is significant
    - Stopping the experiment early as soon as significance is reached
    - Switching from two-tailed to one-tailed test after seeing the direction

    How to guard against it:
    1. Pre-register your hypothesis and primary metric before data collection
    2. Apply multiple testing corrections (Bonferroni or Benjamini-Hochberg) when testing many hypotheses
    3. Distinguish exploratory analysis (hypothesis generation) from confirmatory analysis (hypothesis testing)
    4. Replicate findings on held-out data before acting on them

    In A/B testing platforms: this is why you specify your primary metric before the experiment launches, and why most platforms now use sequential testing methods (like CUPED or always-valid p-values) that are robust to early stopping.

---

**Q10: What assumptions does a t-test make? What do you do when they are violated?**

??? "Show answer"
    A t-test makes these assumptions:

    1. **Independence:** observations are independent of each other (no repeated measures, no clustering)
    2. **Normality:** the data is approximately normally distributed (relaxed for large n by CLT)
    3. **For independent t-test:** the two groups have approximately equal variances (relaxed by using Welch's t-test)

    When assumptions are violated:
    - **Non-normality with small n:** use Mann-Whitney U (non-parametric alternative)
    - **Unequal variances:** use Welch's t-test (set equal_var=False in scipy) — this is actually the default in scipy and should be your default too
    - **Non-independent observations:** restructure the problem (use paired t-test, hierarchical models, or mixed effects models)

    The Central Limit Theorem saves t-tests in most practical situations with n > 30: the sampling distribution of the mean becomes approximately normal even if the underlying data is not, allowing t-tests to perform well on non-normal data at reasonable sample sizes.

---

**Q11: When would you use a paired t-test vs an independent t-test?**

??? "Show answer"
    Use a paired t-test when the same subjects appear in both conditions, or when observations are matched by design. Use an independent t-test when the two groups consist of completely different subjects.

    Examples of paired data:
    - Blood pressure before and after medication (same patients)
    - Employee performance before and after training (same employees)
    - Sales at a store in January vs February (same store)

    Examples of independent data:
    - Conversion rates for users randomly assigned to control vs treatment
    - Average spend of male vs female customers
    - Exam scores for two different class sections

    The paired test is more powerful when pairing is appropriate, because it eliminates between-subject variability. Using an independent test on paired data wastes statistical power. Using a paired test on independent data can produce invalid results.

---

**Q12: What is the difference between Pearson and Spearman correlation?**

??? "Show answer"
    Pearson correlation measures the strength and direction of the linear relationship between two continuous variables. It is sensitive to outliers and assumes both variables are continuous and the relationship is linear.

    Spearman correlation measures the strength and direction of the monotonic relationship. It works by ranking both variables and computing Pearson on the ranks. It is robust to outliers and appropriate for ordinal data.

    When to use which:
    - Use Pearson when the relationship is plausibly linear, both variables are continuous, and there are no extreme outliers (verify with a scatter plot)
    - Use Spearman when the data is ordinal, has outliers, or the relationship might be monotonic but not linear
    - When uncertain, compute both and compare — large disagreement signals non-linearity or outliers

    Example: income and spending might have a Pearson r of 0.7 but a Spearman r of 0.9, because the relationship is monotonic (more income always means more spending) but not linear (the growth is not proportional at all income levels).

---

**Q13: When would you use ANOVA instead of multiple t-tests?**

??? "Show answer"
    Use ANOVA whenever you are comparing means across three or more groups.

    The reason: if you run three t-tests to compare groups A vs B, A vs C, and B vs C (each at α = 0.05), your family-wise Type I error rate is no longer 5%. With three independent tests it is approximately 1 - (0.95)³ ≈ 14%. With 10 tests it is about 40%.

    ANOVA tests all groups simultaneously with a single F-statistic, maintaining your Type I error rate at α.

    The caveat: ANOVA tells you that at least one group differs, but not which pairs. For pairwise comparisons, run a post-hoc test (Tukey HSD is standard) that corrects for multiple comparisons within the ANOVA framework.

    When ANOVA is not appropriate: when variance is unequal across groups (use Welch's ANOVA), when data is severely non-normal with small samples (use Kruskal-Wallis).

---

**Q14: What is Simpson's paradox and why does it matter?**

??? "Show answer"
    Simpson's paradox is when a trend that appears in aggregated data disappears or reverses when the data is broken into subgroups.

    Classic example: University of California Berkeley admissions data from 1973 showed men had a higher overall admission rate than women. But when broken down by department, women had higher or equal admission rates in almost every department. The reversal happened because women applied disproportionately to competitive departments with low acceptance rates.

    Why it matters for data scientists:
    - Aggregated correlations can be completely misleading if there are confounding variables in the data structure
    - A positive correlation between X and Y in the full dataset can become negative within every subgroup
    - This is why you must always understand the data-generating process, not just compute correlations on the raw numbers
    - Segment your analyses. Check whether aggregate trends hold within meaningful subgroups. If they do not, the aggregate trend is a structural artifact, not a causal signal.

---

**Q15: What is the difference between statistical significance and practical significance?**

??? "Show answer"
    Statistical significance tells you whether an observed effect is unlikely to be due to random chance, given your sample size and α. It is a statement about the data.

    Practical significance tells you whether the effect is large enough to matter in the real world. It is a statement about business or scientific value.

    With a large enough sample, any effect — no matter how tiny — becomes statistically significant. A t-test on one million users will find a statistically significant difference between groups that differ by 0.001%. That difference is meaningless in practice.

    How to assess practical significance:
    - Report effect size: Cohen's d for means (small=0.2, medium=0.5, large=0.8), r² for explained variance, relative and absolute lift for proportions
    - Frame the effect in business terms: "$0.50 average order value increase on a $100 AOV = 0.5% lift — is the engineering cost worth it?"
    - Compare the effect to your minimum detectable effect (MDE): the smallest effect that would actually change a decision

    The right answer in an interview is to always check both. Statistical significance without practical significance is a false alarm. Practical significance without statistical significance is noise you cannot yet distinguish from signal.

---

## Scenario Questions

**Q16: Your A/B test shows p = 0.03 and the new design increased conversion by 0.2 percentage points (4.8% → 5.0%). Do you ship it?**

??? "Show answer"
    Not automatically. Here is the analysis:

    Statistical significance: p = 0.03 < 0.05 = α. The result clears the threshold.

    Practical significance: 0.2pp lift on a 4.8% base rate is about a 4.2% relative improvement. Whether this matters depends on:
    - Traffic volume: with 10 million monthly users, 0.2pp × 10M = 20,000 additional conversions — significant business value.
    - Revenue per conversion: if average order value is $5, that is $100K/month. If AOV is $50, that is $1M/month.
    - Implementation cost: is it a CSS change or a full backend redesign?
    - Confidence interval: what is the range? If the CI is [0.0pp, 0.4pp], the lower bound is zero — not reassuring.
    - Segment behavior: does the lift hold across all devices and user types, or is it driven by one segment?

    The answer is: calculate the revenue impact, check the CI, verify across key segments, then make a judgment call. Statistical significance is one input, not the decision.

---

**Q17: You run 20 A/B tests and 2 are significant. What is your concern?**

??? "Show answer"
    At α = 0.05, you expect 5% of tests to produce false positives by chance. Running 20 tests, you expect 1 false positive (20 × 0.05 = 1). Finding 2 significant results is close to what you would expect purely from chance — even if no real effects exist.

    The concern is multiple comparisons (family-wise error rate inflation). Your overall false positive rate for the 20-test family is much higher than 5%.

    What to do:
    - Apply a multiple testing correction. Bonferroni divides α by the number of tests (0.05/20 = 0.0025 per test). Benjamini-Hochberg controls false discovery rate and is less conservative.
    - Pre-specify primary metrics before testing. Secondary analyses are exploratory — treat significant results as hypotheses to test, not conclusions.
    - Replicate: run the two "significant" experiments again independently. If they replicate, they are more likely real.

---

**Q18: How do you explain a confidence interval to a non-technical stakeholder?**

??? "Show answer"
    "We ran a test and the result is that the new design increases conversion rate. Our best estimate is that it increases conversion by about 1.5 percentage points. But since we are working with a sample, there is uncertainty in that number. Based on our data, we are 95% confident the true improvement is somewhere between 0.8 and 2.2 percentage points.

    The bottom range is 0.8 — so even in a pessimistic scenario, the improvement is meaningful. The top range is 2.2 — so the upside could be even better than our central estimate. Given that even the lower end of our range clears the business threshold we set, I recommend we ship this change."

    Key principles for non-technical communication:
    - Lead with the estimate, not the uncertainty
    - Express the interval as a range of plausible outcomes
    - Tie the bounds to business decisions (does the lower bound still justify the action?)
    - Never say "95% probability" — say "95% confident" and quickly move to what it means for the decision

---

## Quick Reference

| Situation | Test | scipy function |
|---|---|---|
| Sample mean vs target value | One-sample t-test | `stats.ttest_1samp(data, popmean)` |
| Two independent group means | Welch's t-test | `stats.ttest_ind(a, b, equal_var=False)` |
| Same subjects, two conditions | Paired t-test | `stats.ttest_rel(before, after)` |
| Three or more group means | One-way ANOVA | `stats.f_oneway(g1, g2, g3)` |
| Two categorical variables | Chi-square independence | `stats.chi2_contingency(table)` |
| One variable vs distribution | Chi-square goodness of fit | `stats.chisquare(observed, expected)` |
| Linear relationship, numeric | Pearson correlation | `stats.pearsonr(x, y)` |
| Monotonic / robust correlation | Spearman correlation | `stats.spearmanr(x, y)` |
| Non-parametric two groups | Mann-Whitney U | `stats.mannwhitneyu(a, b)` |
| Non-parametric 3+ groups | Kruskal-Wallis | `stats.kruskal(g1, g2, g3)` |

---

## Final Checklist

Before leaving this topic, make sure you can do each of these without notes:

- [ ] Define p-value precisely (probability of the data, not the hypothesis)
- [ ] Explain the difference between Type I and Type II errors with examples
- [ ] State what a 95% confidence interval means — and what it does not mean
- [ ] Pick the correct test given: data type, number of groups, independence
- [ ] Explain why correlation does not imply causation with a concrete example
- [ ] Explain what p-hacking is and how to defend against it
- [ ] Articulate the difference between statistical and practical significance

---

> [!success] The Pattern That Wins in Interviews
> The analysts who impress interviewers do not just answer "what is a p-value?" They answer and then add: "and here is the most common mistake people make with it." They give the textbook definition and the practitioner perspective. That combination — knowing the concept and knowing how it fails in the real world — is what separates strong candidates from forgettable ones.

---

[[05-statistical-tests|Previous: Statistical Tests]] | [[../Day-05-Part-1-SQL-for-Data-Science/01-select-and-where|Next: SQL for Data Science]]
