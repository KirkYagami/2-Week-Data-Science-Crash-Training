# Hypothesis Testing

---

### Q1: What is a null hypothesis and an alternative hypothesis?

??? "Show answer"
    **Null hypothesis (H₀)**: the default position — there is no effect, no difference, or no relationship. It is the claim you are trying to disprove. The null is assumed true until you have sufficient evidence against it.

    **Alternative hypothesis (H₁ or Hₐ)**: what you expect or hope to show is true. It is the claim that something real is happening — a difference, an effect, or a relationship.

    The logic of hypothesis testing is an indirect proof: you assume H₀ is true, compute how likely the observed data (or something more extreme) would be under that assumption, and reject H₀ if that probability is small enough.

    Examples:
    - H₀: the new drug has no effect on blood pressure (μ_treatment = μ_control)
    - H₁: the new drug lowers blood pressure (μ_treatment < μ_control)

    - H₀: click-through rate is the same for both banner designs (p_A = p_B)
    - H₁: click-through rates differ (p_A ≠ p_B)

    Interview tip: the null hypothesis always contains an equality (=, ≤, ≥). The alternative is always the complement. You never "prove" the null — you either reject it or fail to reject it.

---

### Q2: What is a p-value and what does it actually mean?

??? "Show answer"
    A **p-value** is the probability of obtaining a test statistic as extreme as, or more extreme than, the one observed — *assuming the null hypothesis is true*.

    Formally: p = P(data as extreme as observed | H₀ is true)

    Interpretation: a small p-value means the observed result would be unusual if the null hypothesis were true. It is a measure of evidence against H₀, not a measure of the probability that H₀ is true.

    Example: you measure a difference in conversion rates between two website variants. You compute a test statistic and find p = 0.03. This means: if there were truly no difference (H₀ true), there would be only a 3% chance of seeing a difference this large or larger by random chance alone.

    Decision rule: if p < α (significance level, commonly 0.05), reject H₀.

    What p-value tells you:
    - Evidence against the null (small p = strong evidence against H₀)
    - Nothing directly about the size or importance of the effect
    - Nothing about the probability that H₀ is true

---

### Q3: What is a common misconception about p-values?

??? "Show answer"
    There are several widespread misconceptions — interviewers often probe for these specifically:

    **Misconception 1**: "p = 0.03 means there is a 3% probability that the null hypothesis is true."
    **Reality**: the p-value is not the probability that H₀ is true. It is P(data | H₀), not P(H₀ | data). To get P(H₀ | data), you need Bayes' theorem and a prior over H₀.

    **Misconception 2**: "1 − p is the probability that the alternative hypothesis is true."
    **Reality**: same error, reversed. Neither the p-value nor its complement is a probability statement about hypotheses.

    **Misconception 3**: "p > 0.05 means the null hypothesis is true / there is no effect."
    **Reality**: failing to reject H₀ does not mean H₀ is true. It means you don't have enough evidence to reject it. Absence of evidence is not evidence of absence. Your test might be underpowered.

    **Misconception 4**: "p < 0.05 means the effect is large or practically meaningful."
    **Reality**: with a large enough sample, even a trivially small effect will yield p < 0.05. Statistical significance ≠ practical significance.

    **Misconception 5**: "a p-value of 0.049 is meaningfully different from 0.051."
    **Reality**: the 0.05 threshold is arbitrary. The p-value is continuous evidence; a sharp cutoff creates a binary decision from a continuous quantity.

---

### Q4: What is statistical significance?

??? "Show answer"
    A result is called **statistically significant** when the p-value is below the pre-specified significance level α (alpha), meaning you reject the null hypothesis.

    The significance level α represents the acceptable probability of making a Type I error (rejecting H₀ when it is actually true). The most common convention is α = 0.05, though α = 0.01 and α = 0.10 are also used.

    Statistical significance means: the observed effect is unlikely to have arisen by chance alone, under the null hypothesis, at the chosen α level.

    Critical nuances:
    - α must be chosen *before* running the test — choosing it after seeing the data is p-hacking.
    - Statistical significance depends on sample size. The same effect will be significant with a large sample and not significant with a small one.
    - Statistical significance says nothing about whether the effect is meaningful in the real world — that requires examining **effect size** and **practical significance**.
    - The α = 0.05 threshold is a convention, not a law of nature. In particle physics, the threshold is 5σ (p ≈ 3×10⁻⁷); in genomics, corrections for millions of tests are applied.

---

### Q5: What is a Type I error (false positive)?

??? "Show answer"
    A **Type I error** occurs when you reject the null hypothesis H₀ even though it is actually true — concluding there is an effect when there is none.

    It is also called a **false positive**: the test "found" something that isn't really there.

    The probability of a Type I error is exactly **α**, the significance level. By choosing α = 0.05, you accept a 5% chance of falsely rejecting a true null on any single test.

    Real-world examples:
    - Concluding a new drug works when it doesn't (based on a lucky sample)
    - Concluding variant B converts better than A in an A/B test when the difference is just noise
    - A spam filter flagging a legitimate email as spam

    Reducing Type I error: lower α (e.g., from 0.05 to 0.01). Trade-off: this increases Type II error (false negatives), making it harder to detect real effects.

    Multiple testing inflates Type I error: if you run 20 tests at α = 0.05 with no real effects, you expect 1 false positive on average. Corrections like Bonferroni address this.

---

### Q6: What is a Type II error (false negative)?

??? "Show answer"
    A **Type II error** occurs when you fail to reject the null hypothesis H₀ even though it is actually false — missing a real effect.

    It is also called a **false negative**: the test failed to detect something that is genuinely there.

    The probability of a Type II error is denoted **β**. The **power** of a test is 1 − β — the probability of correctly detecting a real effect.

    Real-world examples:
    - Concluding a drug has no effect when it actually does
    - Failing to detect that variant B converts better, so you stick with the inferior variant A
    - A cancer screening test missing an actual tumour

    Factors that increase Type II error (decrease power):
    - Small sample size
    - Small true effect size
    - High variability in the data
    - Low significance level α (being too strict)

    Trade-off: lowering α (to reduce Type I error) raises β and increases Type II error risk. The right balance depends on the costs of each error type in context.

    |               | H₀ True      | H₀ False     |
    |---------------|--------------|--------------|
    | Reject H₀     | Type I (FP)  | Correct (TP) |
    | Fail to reject| Correct (TN) | Type II (FN) |

---

### Q7: What is statistical power?

??? "Show answer"
    **Statistical power** is the probability that a test correctly rejects a false null hypothesis — the probability of detecting a real effect when one exists.

    Power = 1 − β = P(reject H₀ | H₀ is false)

    A power of 0.8 (80%) is the conventional minimum: it means an 80% chance of detecting the effect if it's real (and a 20% chance of missing it).

    Power is determined by four interrelated quantities — given any three, you can solve for the fourth:
    1. **Effect size**: larger effects are easier to detect → higher power
    2. **Sample size**: more data → higher power
    3. **Significance level α**: raising α increases power but also Type I error risk
    4. **Variability**: lower variance → higher power

    Power analysis (pre-experiment): set α = 0.05 and target power = 0.80, specify the minimum effect size you care about (MDE), and solve for required sample size. This is essential for any well-designed experiment.

    ```python
    from statsmodels.stats.power import TTestIndPower

    analysis = TTestIndPower()
    # Required sample size per group for d=0.3, alpha=0.05, power=0.8
    n = analysis.solve_power(effect_size=0.3, alpha=0.05, power=0.8, alternative='two-sided')
    print(f"Required n per group: {n:.0f}")
    ```

---

### Q8: What is the difference between one-tailed and two-tailed tests?

??? "Show answer"
    The choice of "tails" determines which direction of deviation from H₀ counts as evidence against it.

    **Two-tailed test**: H₁ is that the effect could be in either direction. You split α across both tails. Example: H₀: μ_A = μ_B vs H₁: μ_A ≠ μ_B. Reject if the test statistic falls in either extreme tail.

    **One-tailed test**: H₁ specifies a direction. All of α goes into one tail. Example: H₀: μ_B ≤ μ_A vs H₁: μ_B > μ_A (testing specifically that B is better). More powerful if the effect truly is in that direction.

    When to use each:
    - **Two-tailed**: when you are genuinely agnostic about direction — appropriate for most exploratory tests and regulatory submissions.
    - **One-tailed**: when theory or prior evidence strongly constrains the direction, and you have committed to the direction *before* seeing the data. Using one-tailed after seeing which direction the data leans is p-hacking.

    In practice: most A/B tests use two-tailed tests because you usually want to know if either variant is better. One-tailed tests are sometimes used in non-inferiority trials (showing a new treatment is no worse than the standard).

    The critical value for a two-tailed test at α = 0.05 is ±1.96 (standard normal); for one-tailed it is 1.645.

---

### Q9: What is a t-test? When do you use one-sample vs two-sample vs paired?

??? "Show answer"
    A **t-test** compares means using the t-distribution, accounting for uncertainty in the estimated variance. There are three variants:

    **One-sample t-test**: tests if the mean of a single sample differs from a known reference value μ₀.
    - H₀: μ = μ₀
    - Example: testing if the average delivery time is 30 minutes (μ₀ = 30)
    - t = (x̄ − μ₀) / (s / √n)

    **Independent two-sample t-test**: tests if the means of two independent groups differ.
    - H₀: μ₁ = μ₂
    - Example: comparing average scores between two different teaching methods
    - Use Welch's t-test (unequal variances) unless you have strong reason to assume equal variances

    **Paired t-test**: tests if the mean *difference* between paired observations is zero.
    - Use when the same subjects are measured twice (before/after) or when data are naturally paired
    - Reduces variance by accounting for individual-level correlation
    - Example: comparing blood pressure before and after treatment in the same patients
    - t = (d̄) / (s_d / √n), where d is the pairwise difference

    Paired tests are more powerful than two-sample tests when the within-subject correlation is positive.

    ```python
    from scipy import stats
    import numpy as np

    before = [120, 132, 118, 125, 130]
    after  = [115, 125, 116, 119, 122]

    # One-sample: is mean difference zero?
    stat, p = stats.ttest_rel(before, after)
    print(f"Paired t-test: t={stat:.3f}, p={p:.4f}")
    ```

---

### Q10: What are the assumptions of a t-test?

??? "Show answer"
    The t-test relies on several assumptions. Violating them can make the test unreliable.

    **Core assumptions**:

    1. **Independence**: observations are independent of each other. Violating this (e.g., repeated measures, clustered data) inflates Type I error.

    2. **Normality of the population** (or sampling distribution of the mean): for small samples (n < 30), the underlying data should be approximately normal. For large samples, the CLT makes the t-test robust.

    3. **Equal variances** (for the standard two-sample t-test): Welch's t-test relaxes this assumption and is recommended as the default.

    4. **Continuous data**: the t-test is designed for continuous measurements, not counts or proportions (though it can be reasonable for proportions with large n).

    How to check assumptions:
    - Independence: by study design (randomisation, sampling procedure)
    - Normality: Q-Q plot, Shapiro-Wilk test (for small samples)
    - Equal variances: Levene's test or F-test for variance

    What to do when assumptions are violated:
    - Not normal + small sample → Mann-Whitney U test (non-parametric alternative)
    - Dependent observations → paired t-test or mixed models
    - Unequal variances → Welch's t-test

    ```python
    from scipy import stats
    import numpy as np

    group_a = np.random.normal(10, 2, 50)
    group_b = np.random.normal(11, 3, 50)

    # Welch's t-test (equal_var=False is the safe default)
    stat, p = stats.ttest_ind(group_a, group_b, equal_var=False)
    print(f"Welch t={stat:.3f}, p={p:.4f}")

    # Check normality
    print(stats.shapiro(group_a))
    # Check variance
    print(stats.levene(group_a, group_b))
    ```

---

### Q11: What is the Mann-Whitney U test and when would you use it over a t-test?

??? "Show answer"
    The **Mann-Whitney U test** (also called the Wilcoxon rank-sum test) is a non-parametric test that compares two independent groups without assuming normality. It tests whether one distribution tends to produce larger values than the other.

    Formally, it tests H₀: P(X > Y) = 0.5 — that a randomly selected value from group X is equally likely to be larger or smaller than a randomly selected value from group Y.

    When to use Mann-Whitney over t-test:
    - Data is not normally distributed and sample size is too small for CLT to apply
    - Data is **ordinal** (e.g., Likert scale ratings — you cannot meaningfully take means)
    - There are severe outliers that would distort the mean-based comparison
    - The outcome variable is a count or skewed continuous variable (revenue, response time)

    Trade-offs:
    - Less powerful than t-test when data actually is normal (parametric tests extract more information)
    - Tests a different hypothesis (distributional dominance vs mean difference) — the distinction matters when interpreting results
    - Does not directly test for difference in means; it tests for "stochastic dominance"

    ```python
    from scipy.stats import mannwhitneyu
    import numpy as np

    # Revenue data — highly skewed
    group_a = np.random.exponential(scale=10, size=100)
    group_b = np.random.exponential(scale=12, size=100)

    stat, p = mannwhitneyu(group_a, group_b, alternative='two-sided')
    print(f"Mann-Whitney U={stat:.1f}, p={p:.4f}")
    ```

---

### Q12: What is ANOVA and when do you use it?

??? "Show answer"
    **ANOVA (Analysis of Variance)** tests whether the means of three or more independent groups are all equal. It generalises the two-sample t-test to multiple groups.

    H₀: μ₁ = μ₂ = … = μₖ (all group means are equal)
    H₁: at least one group mean differs

    **How it works**: ANOVA computes the F-statistic — the ratio of between-group variance to within-group variance. A large F means the groups differ more than would be expected by chance.

    F = (between-group variance) / (within-group variance) = MSG / MSE

    Why not just run all pairwise t-tests? With k groups, there are C(k,2) pairwise comparisons. Each test has a 5% Type I error rate. Running many tests inflates the family-wise error rate — ANOVA controls it at α for the overall test.

    ANOVA assumptions: independence, normality within groups, homogeneity of variances (Levene's test). If variances are unequal, use Welch's ANOVA.

    After a significant ANOVA, you need **post-hoc tests** (e.g., Tukey HSD, Bonferroni) to determine which specific pairs differ.

    ```python
    from scipy import stats

    group_a = [10, 12, 11, 13, 12]
    group_b = [14, 15, 13, 16, 14]
    group_c = [11, 10, 12, 11, 13]

    f_stat, p_value = stats.f_oneway(group_a, group_b, group_c)
    print(f"F={f_stat:.3f}, p={p_value:.4f}")
    # If p < 0.05, conclude at least one group mean differs — run post-hoc
    ```

---

### Q13: What is the chi-squared test of independence?

??? "Show answer"
    The **chi-squared test of independence** tests whether two categorical variables are statistically independent. H₀: the two variables are independent. H₁: there is an association.

    It compares the observed frequencies in a contingency table to the expected frequencies under independence.

    Expected frequency for cell (i, j): E_ij = (row_i total × column_j total) / grand total

    Test statistic: χ² = Σ (O_ij − E_ij)² / E_ij

    This follows a χ²(df) distribution where df = (r−1)(c−1), r = number of rows, c = number of columns.

    Assumptions:
    - Observations are independent
    - Expected frequency in each cell ≥ 5 (for large expected frequencies, the chi-squared approximation is accurate; for smaller cells, use Fisher's exact test)

    Common uses:
    - Testing if gender is associated with product preference
    - Testing if a drug outcome differs between treatment groups
    - Testing if two features are associated in a dataset

    ```python
    from scipy.stats import chi2_contingency
    import numpy as np

    # Contingency table: rows = gender, cols = preference (A, B, C)
    observed = np.array([[30, 10, 20],
                         [25, 20, 15]])
    chi2, p, dof, expected = chi2_contingency(observed)
    print(f"χ²={chi2:.3f}, df={dof}, p={p:.4f}")
    print("Expected frequencies:\n", expected)
    ```

---

### Q14: What is the Kolmogorov-Smirnov test?

??? "Show answer"
    The **Kolmogorov-Smirnov (KS) test** compares a sample distribution to a reference distribution (one-sample KS) or compares two sample distributions (two-sample KS).

    It measures the maximum absolute difference between the empirical CDF (ECDF) and the theoretical CDF:
    D = max_x |F_n(x) − F(x)|

    Large D → reject the null hypothesis that the data follows the reference distribution.

    One-sample KS test:
    - H₀: the sample comes from a specified distribution (e.g., normal with given μ, σ)
    - Use case: checking if residuals are normally distributed

    Two-sample KS test:
    - H₀: the two samples come from the same distribution
    - Use case: checking if a treatment changed the full distribution (not just the mean), comparing model score distributions between training and serving data (data drift detection)

    Limitations:
    - Less powerful than Shapiro-Wilk for testing normality
    - Most sensitive in the middle of the distribution, less sensitive in the tails
    - Requires parameters to be known in one-sample version (if estimated from data, it's anti-conservative)

    ```python
    from scipy.stats import ks_2samp, kstest, norm
    import numpy as np

    data = np.random.normal(size=100)

    # One-sample: test against standard normal
    stat, p = kstest(data, 'norm')
    print(f"KS one-sample: D={stat:.4f}, p={p:.4f}")

    # Two-sample: compare two datasets
    data2 = np.random.normal(loc=0.5, size=100)
    stat2, p2 = ks_2samp(data, data2)
    print(f"KS two-sample: D={stat2:.4f}, p={p2:.4f}")
    ```

---

### Q15: What is the multiple comparisons problem?

??? "Show answer"
    The **multiple comparisons problem** (also called the multiple testing problem) occurs when you perform many hypothesis tests simultaneously. Each test has an α probability of producing a false positive. When you run many tests, the probability that *at least one* is a false positive grows substantially.

    If you run m independent tests each at α = 0.05, the probability of at least one false positive is:
    P(at least one FP) = 1 − (1 − 0.05)^m

    At m = 20: P ≈ 0.64. At m = 100: P ≈ 0.994. You are virtually guaranteed a false positive.

    Common scenarios where this arises:
    - Testing many features for correlation with an outcome (data mining)
    - A/B testing with many metrics (clicking through all 50 dashboard metrics)
    - Genomics (testing millions of SNPs for disease association)
    - Running many sub-group analyses

    Solutions:
    - **Bonferroni correction**: divide α by the number of tests
    - **Benjamini-Hochberg**: controls the false discovery rate (FDR)
    - **Pre-register** your primary metric before running the test
    - Report all tests conducted, not just significant ones

---

### Q16: What is the Bonferroni correction?

??? "Show answer"
    The **Bonferroni correction** is the simplest method to control the **family-wise error rate (FWER)** — the probability of making *at least one* false positive — when conducting multiple hypothesis tests.

    Adjusted threshold: α_adjusted = α / m, where m is the number of tests.

    Example: if you run 10 tests and want FWER ≤ 0.05, each individual test must have p < 0.005 to be declared significant.

    Alternatively: multiply each p-value by m (adjusted p-value = min(m × p_i, 1)) and compare to original α.

    Pros:
    - Simple to apply and understand
    - Guarantees FWER ≤ α regardless of test correlations
    - Conservative — it's the safest correction

    Cons:
    - Very conservative when tests are positively correlated (redundant tests) — reduces power substantially
    - With many tests (e.g., m = 10,000 in genomics), it is so strict it misses almost all real effects

    Use Bonferroni when the cost of any false positive is high and m is small. When m is large, prefer FDR control (Benjamini-Hochberg).

    ```python
    from statsmodels.stats.multitest import multipletests
    import numpy as np

    p_values = [0.001, 0.008, 0.04, 0.05, 0.20, 0.60]
    reject, p_corrected, _, _ = multipletests(p_values, alpha=0.05, method='bonferroni')
    for orig, corr, rej in zip(p_values, p_corrected, reject):
        print(f"p={orig:.3f} → corrected={corr:.3f}, reject={rej}")
    ```

---

### Q17: What is the Benjamini-Hochberg procedure?

??? "Show answer"
    The **Benjamini-Hochberg (BH) procedure** controls the **False Discovery Rate (FDR)** — the expected proportion of false positives among all rejected hypotheses. It is less conservative than Bonferroni and is the standard approach in high-dimensional settings.

    FDR = E[FP / max(R, 1)], where R is the total number of rejections.

    Algorithm:
    1. Sort the m p-values in ascending order: p₍₁₎ ≤ p₍₂₎ ≤ … ≤ p₍ₘ₎
    2. Find the largest k such that p₍ₖ₎ ≤ (k/m) × α
    3. Reject all hypotheses H₍₁₎, …, H₍ₖ₎

    The BH procedure guarantees FDR ≤ α when tests are independent (or have positive dependence).

    Example: m = 1000 tests, target FDR = 0.05. If BH identifies 50 significant results, the expected number of false positives is ≤ 0.05 × 50 = 2.5.

    When to use BH vs Bonferroni:
    - Bonferroni: when any false positive is unacceptable (e.g., regulatory decisions)
    - Benjamini-Hochberg: when you're doing exploratory research and some false positives are acceptable as long as most discoveries are real (genomics, feature selection)

    ```python
    from statsmodels.stats.multitest import multipletests
    import numpy as np

    p_values = np.array([0.001, 0.008, 0.04, 0.05, 0.20, 0.60])
    reject, p_adjusted, _, _ = multipletests(p_values, alpha=0.05, method='fdr_bh')
    for orig, adj, rej in zip(p_values, p_adjusted, reject):
        print(f"p={orig:.3f} → BH-adjusted={adj:.3f}, reject={rej}")
    ```

---

### Q18: What is the difference between statistical significance and practical significance?

??? "Show answer"
    **Statistical significance** tells you that an effect is unlikely to be due to chance (p < α). It is entirely determined by sample size, effect size, and variability.

    **Practical significance** (or **clinical significance** in medical contexts) asks whether the effect is large enough to matter in the real world. It requires domain knowledge and context.

    The gap between the two is crucial:
    - With a very large sample, you can detect an effect of essentially zero practical value. Example: an A/B test with 10 million users might show p < 0.0001 for a 0.001% lift in conversion rate. Statistically significant — but implementing the change may not be worth engineering cost.
    - With a small sample, a large, meaningful effect might not reach p < 0.05. Practically significant — but statistically underpowered.

    Always report **effect sizes** alongside p-values. Ask: "Is this effect large enough for us to act on, or change anything in the product/business?"

    Practical significance thresholds are business decisions: a 0.5% improvement in click-through rate might be huge at Google scale and irrelevant at a startup.

---

### Q19: What is effect size?

??? "Show answer"
    **Effect size** is a standardised, scale-free measure of the magnitude of an effect, independent of sample size. It tells you *how big* the effect is, not just whether it exists.

    Common measures:

    **Cohen's d** (for comparing two means):
    d = (μ₁ − μ₂) / s_pooled
    - Small: |d| = 0.2, Medium: |d| = 0.5, Large: |d| = 0.8

    **r (correlation)**: the Pearson correlation coefficient serves as an effect size.
    - Small: |r| = 0.1, Medium: |r| = 0.3, Large: |r| = 0.5

    **Odds ratio / relative risk**: for binary outcomes

    **η² (eta-squared)**: proportion of variance explained in ANOVA (analogous to R²)

    Why effect size matters:
    - It is not inflated by sample size (unlike p-values)
    - It is needed for power analysis and sample size calculation
    - It enables meta-analysis (combining results across studies)
    - It communicates whether an effect is worth caring about

    ```python
    import numpy as np

    def cohens_d(group1, group2):
        n1, n2 = len(group1), len(group2)
        pooled_std = np.sqrt(((n1-1)*np.var(group1, ddof=1) + (n2-1)*np.var(group2, ddof=1)) / (n1+n2-2))
        return (np.mean(group1) - np.mean(group2)) / pooled_std

    a = np.random.normal(10, 2, 100)
    b = np.random.normal(10.5, 2, 100)
    print(f"Cohen's d = {cohens_d(a, b):.3f}")
    ```

---

### Q20: What does "reject the null hypothesis" actually mean in plain English?

??? "Show answer"
    "Reject the null hypothesis" means: **the data are sufficiently inconsistent with the null hypothesis that we no longer believe H₀ is a plausible explanation for what we observed.**

    More precisely: the probability of seeing data as extreme as ours, *if H₀ were true*, is less than our pre-chosen threshold α. We're saying: "If nothing were going on, the chance of getting a result like this is less than 5% — so we're acting as if something is going on."

    What it does NOT mean:
    - H₀ is definitely false (we could be making a Type I error)
    - The effect is large or important
    - The result will replicate (with α = 0.05 and no effect, 5% of replications will also be "significant")

    The verdict is always provisional: we're making a decision under uncertainty, not a logical proof. The accumulation of evidence from multiple well-powered replications is what actually builds scientific confidence.

    For practitioners: "we reject H₀" translates to "we have enough evidence to act — to ship the feature, adopt the treatment, or update the model." The decision to act still weighs effect size, cost, business risk, and reversibility.
