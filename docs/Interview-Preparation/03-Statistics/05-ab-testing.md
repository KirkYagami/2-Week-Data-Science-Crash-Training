# A/B Testing

---

### Q1: What is an A/B test and what is it used for?

??? "Show answer"
    An **A/B test** (randomised controlled experiment) is a method for comparing two or more variants of something by randomly assigning users (or other units) to each variant and measuring outcomes. The goal is to establish whether a change causes a measurable improvement in a key metric — not merely correlates with it.

    The fundamental idea: randomisation ensures that the groups are comparable at baseline, so any observed difference in outcomes can be attributed to the intervention rather than pre-existing differences between users.

    A/B testing is used to:
    - Evaluate product changes: new UI, copy, recommendation algorithm, pricing
    - Compare models: does the new ranking model increase clicks or revenue?
    - Measure the causal impact of features before a full rollout
    - Reduce risk: expose only a fraction of users to an unproven change

    Why not just observe before and after? Time confounds everything — seasonality, market events, product changes all move metrics. Concurrent randomisation controls for these.

---

### Q2: What are the steps to design a valid A/B test?

??? "Show answer"
    A rigorous A/B test design follows these steps:

    1. **Define the question and hypothesis**: what change are you testing and what mechanism should cause the effect? H₀: the change has no effect on the primary metric.

    2. **Choose the primary metric**: one metric that directly reflects the goal. Secondary/guardrail metrics monitor for harm.

    3. **Define the randomisation unit**: user, session, device, request? This must match the analysis unit (discussed later).

    4. **Calculate required sample size**: use power analysis. Inputs: significance level (α = 0.05), power (1 − β = 0.80), baseline metric value, and minimum detectable effect (MDE).

    5. **Choose the experiment duration**: long enough to reach sample size, and to cover natural variation (full week cycles to avoid day-of-week effects). Account for novelty effect by sometimes running longer.

    6. **Implement and validate**: ensure randomisation is working (AA test), that there is no leakage between groups, and logging is correct.

    7. **Run the test**: don't peek at results until you've reached your target sample size.

    8. **Analyse results**: compute the test statistic and p-value. Check guardrail metrics. Compute effect size.

    9. **Make the decision**: ship, iterate, or reject. Document everything.

---

### Q3: What is the control group and the treatment group?

??? "Show answer"
    The **control group** (group A) receives the existing, unchanged experience. It serves as the baseline against which the treatment is compared. The control should represent your "do nothing" scenario.

    The **treatment group** (group B) receives the new variant — the change being tested (new feature, different copy, different algorithm, etc.).

    Users are **randomly assigned** to one of the two groups. With proper randomisation, the only systematic difference between the groups is the treatment itself.

    Key properties of a good control/treatment split:
    - **Random assignment**: not based on any user characteristic that could correlate with the outcome
    - **Exclusivity**: a user is in exactly one group for the duration of the experiment (no overlap)
    - **Balance**: roughly equal split (50/50 is most efficient for equal-variance outcomes; can be unequal when you want to limit exposure of an unproven change)

    Collecting data on both groups simultaneously — not sequentially — is essential. A sequential before/after design conflates the treatment effect with time trends.

---

### Q4: What is the null hypothesis in an A/B test?

??? "Show answer"
    The null hypothesis in an A/B test is typically:

    **H₀**: the treatment has no effect on the primary metric — the metric is the same in both groups.

    Written formally: H₀: μ_treatment = μ_control (for a mean metric), or H₀: p_treatment = p_control (for a proportion metric).

    The alternative hypothesis (H₁) is usually two-sided: μ_treatment ≠ μ_control. This tests whether the treatment caused any change, positive or negative. One-sided (directional) tests are used when you only care about improvement, but they require you to pre-commit to the direction.

    The p-value you compute is the probability of observing a difference as large as you measured (or larger) purely by chance, *if the null were true*.

    If p < α (typically 0.05), you reject the null and conclude the treatment had a statistically significant effect. You then examine the direction and magnitude to decide whether to ship.

---

### Q5: What is statistical power in the context of A/B testing?

??? "Show answer"
    **Power** in A/B testing is the probability that your test detects a real effect when one exists — the probability of correctly concluding "this change worked" when it actually did.

    Power = 1 − β = P(reject H₀ | true effect ≥ MDE)

    The conventional target is 80% power: you accept a 20% chance of missing a real effect.

    Why power matters in practice:
    - An underpowered test that returns p > 0.05 cannot tell you "the change doesn't work." It tells you "we didn't have enough data to detect the effect, if it exists."
    - Shipping an underpowered negative result as "no effect" is a false economy — you may be killing a genuinely good idea.
    - Running many underpowered tests amplifies both false negatives and, if you cherry-pick significant results, false positives.

    Power is determined by: effect size (MDE), sample size, significance level (α), and metric variance. To increase power without changing α, you must either increase sample size or reduce variance (e.g., through CUPED).

    ```python
    from statsmodels.stats.power import NormalIndPower

    analysis = NormalIndPower()
    # What power do we have with n=500 per group, detecting a 5% relative lift
    # on a baseline conversion of 0.10?
    baseline = 0.10
    mde_abs = 0.005  # 0.5 percentage points absolute lift
    effect_size = mde_abs / (baseline * (1 - baseline)) ** 0.5  # Cohen's h approximation

    power = analysis.solve_power(effect_size=effect_size, nobs1=500, alpha=0.05)
    print(f"Power: {power:.3f}")
    ```

---

### Q6: What is sample size calculation? What factors affect it?

??? "Show answer"
    **Sample size calculation** determines the minimum number of observations needed in each group to reliably detect an effect of a given size with specified error rates.

    The formula for comparing two proportions (simplified):
    n ≈ 2 × (z_{α/2} + z_β)² × p(1−p) / (MDE)²

    Where:
    - z_{α/2} = 1.96 for α = 0.05 (two-tailed)
    - z_β = 0.84 for 80% power
    - p = baseline conversion rate
    - MDE = minimum detectable effect (absolute difference you want to be able to detect)

    Factors that affect required sample size:
    - **MDE**: smaller MDE requires more data. Detecting a 0.1% lift needs far more users than detecting a 5% lift.
    - **Baseline metric**: a 10% conversion rate with MDE = 1pp requires a different n than a 50% rate with MDE = 1pp.
    - **Variance / metric spread**: more variable metrics require larger samples.
    - **α**: lower α (stricter) → larger n.
    - **Power (1 − β)**: higher power target → larger n.
    - **Number of variants**: more arms → larger total sample needed.

    ```python
    from statsmodels.stats.proportion import proportion_effectsize
    from statsmodels.stats.power import NormalIndPower

    baseline_rate = 0.10
    target_rate = 0.105  # expecting 5% relative lift = 0.5pp absolute
    effect = proportion_effectsize(baseline_rate, target_rate)
    analysis = NormalIndPower()
    n = analysis.solve_power(effect_size=effect, alpha=0.05, power=0.80)
    print(f"Required n per group: {n:.0f}")
    ```

---

### Q7: What is the minimum detectable effect (MDE)?

??? "Show answer"
    The **minimum detectable effect (MDE)** is the smallest effect size your experiment is powered to detect, given your sample size, significance level, and target power. It is the boundary between "detectable" and "too small to find."

    MDE is a design choice, not a discovery. You set it before the experiment based on what effect size would be meaningful enough to act on. If the true effect is smaller than the MDE, your experiment will likely miss it (insufficient power).

    MDE can be expressed as:
    - **Absolute**: "we can detect a 0.5 percentage point difference in conversion rate"
    - **Relative**: "we can detect a 5% relative lift from a 10% baseline"

    How to set the MDE:
    - Ask: what is the smallest improvement that would justify shipping this change?
    - Consider: engineering cost, technical debt, product risk
    - Business context: a 0.1% lift in revenue per user might be worth millions at scale

    Setting MDE too small forces an impractically large sample size. Setting it too large means you might miss real but modest improvements. The MDE should reflect minimum business relevance.

---

### Q8: What happens if you stop an A/B test early?

??? "Show answer"
    Stopping a test early (before reaching your pre-calculated sample size) because the results "look significant" almost always leads to **inflated Type I error rates** — you declare a winner more often than your stated α would suggest.

    Why: if you keep checking results as data accumulates and stop as soon as p < 0.05, you're essentially running multiple looks at the data. Each look is an additional opportunity to cross the threshold by chance. With enough looks, you will almost surely cross 0.05 eventually — even if there is no real effect.

    Simulations show that with continuous monitoring, the effective Type I error rate can be 5–10× the nominal α.

    What you should do instead:
    - **Pre-commit** to a sample size and run until you hit it.
    - Use **sequential testing** methods (e.g., alpha spending functions, always-valid p-values, mSPRT) that are designed for early stopping without inflating Type I error.
    - Use Bayesian A/B testing, which does not require a fixed sample size in the same way.

    Legitimate reasons to stop early:
    - A guardrail metric shows clear harm (stop for safety, not for winning)
    - Pre-specified interim analyses with alpha spending functions

---

### Q9: What is the peeking problem?

??? "Show answer"
    The **peeking problem** is the inflated Type I error that results from repeatedly checking experiment results as data accumulates and making a stopping decision based on what you see — even if you don't formally stop the test early.

    Even "just looking" is problematic if it influences your decision about when to stop or what to report. If you run until significance and then stop, you are implicitly doing sequential testing without controlling for it.

    The core issue: under the null hypothesis, the p-value follows a U(0,1) distribution at any single point in time. But if you track the p-value over time, it will wander — and with enough time and enough looks, it will almost certainly dip below 0.05, even if H₀ is true.

    Solutions:
    - **Fixed-horizon testing**: pre-determine n, run exactly that long, look once.
    - **Sequential testing / always-valid inference**: methods like mSPRT or the mixture sequential probability ratio test allow continuous monitoring with valid error guarantees.
    - **Bayesian monitoring**: posterior probabilities are "valid at any time" in a different sense — they update correctly as data arrives without the frequentist multiple testing issue.
    - **Alpha spending functions** (O'Brien-Fleming, Pocock): pre-specified rules for how to spend the α budget across multiple interim looks.

---

### Q10: What is novelty effect and how does it bias results?

??? "Show answer"
    The **novelty effect** (also called newness effect) is the tendency for users to engage more with a new experience simply because it is new — not because it is genuinely better. This inflates the apparent effect size during the experiment.

    Conversely, a **change aversion effect** can depress engagement with a new design because users are uncomfortable with unfamiliarity, even if the design is objectively superior.

    Both effects are temporary and decay as users habituate to the change.

    Signs of novelty effect:
    - The treatment group shows a spike in the metric in the first days, then gradually declines toward the control level
    - The metric improvement is concentrated in users who rarely use the feature (curious exploration)
    - Long-tenured users in the treatment don't show the same lift as new users

    How to handle it:
    - **Run the experiment longer**: 2–4 weeks instead of 1, so the novelty wears off and you measure steady-state behaviour.
    - **Analyse by user tenure**: compare metric trajectories for long-tenured vs new users.
    - **Monitor metric decay**: plot the daily difference between treatment and control over time.
    - **Holdback tests / long-term holdouts**: keep a small group on the old experience for months after launch to measure the sustained effect.

---

### Q11: What is the network effect and why does it invalidate standard A/B tests?

??? "Show answer"
    The **network effect** (or **interference** between units) occurs when one user's treatment assignment affects another user's outcomes. This violates the **Stable Unit Treatment Value Assumption (SUTVA)**, which is required for standard A/B tests to be valid.

    SUTVA requires: the potential outcome for any unit depends only on that unit's own treatment — not on the treatment assigned to other units.

    Where network interference occurs:
    - **Social networks**: showing user A a new feed algorithm affects what content they produce, which affects user B's feed even if B is in the control group.
    - **Marketplaces**: showing more supply to treatment buyers increases competition for control buyers; or giving discounts to some sellers affects supply available to all buyers.
    - **Ride-sharing / delivery**: routing algorithm changes for treatment drivers affect wait times for control riders.
    - **Multiplayer games**: matchmaking changes affect all players in a match.

    Standard A/B tests in these settings produce biased estimates of the global average treatment effect (GATE).

    Solutions:
    - **Cluster randomisation**: randomise at the level of the cluster (city, market, social community) rather than individual users.
    - **Switchback experiments**: alternate treatment and control over time windows in the same market.
    - **Ego network experiments**: assign by ego network clusters.
    - **Bipartite graph experiments**: used in marketplace platforms.

---

### Q12: What is a switchback experiment?

??? "Show answer"
    A **switchback experiment** (time-based randomisation) alternates between treatment and control over time periods within the same unit (typically a geographic market or system). Instead of user-level randomisation, you flip the entire system between A and B at regular intervals.

    Example: a ride-sharing company wants to test a new pricing algorithm. They flip the algorithm for a city between old and new pricing every 30 minutes. During "treatment" periods, the new algorithm is live; during "control" periods, the old algorithm runs.

    When to use switchbacks:
    - When user-level randomisation causes network interference (marketplace effects)
    - When you can't randomise individual users independently (e.g., a shared dispatch system)
    - When the system state resets quickly between periods

    Challenges:
    - **Carryover effects**: the effect of one period may "leak" into the next (e.g., a surge in supply during control takes time to clear). Requires washout periods between flips.
    - **Period length selection**: too short → carryover; too long → fewer periods, less statistical power.
    - **Time confounding**: if treatment periods coincide with systematically different times (rush hour, weekday), results are biased. Randomise which periods get treatment vs control.

    Analysis uses a regression with period and market fixed effects to control for time trends.

---

### Q13: What is the difference between user-level and session-level randomisation?

??? "Show answer"
    The **randomisation unit** is the entity assigned to treatment or control. The most common choices are users or sessions, and they have significantly different implications.

    **User-level randomisation**: each user is permanently assigned to one variant for the duration of the experiment.
    - Pros: consistent experience for the user (no switching); valid for analysing user-level outcomes (retention, LTV).
    - Cons: slower to accumulate observations; cannot be used when user identity is unknown (anonymous traffic).
    - Required for: any experiment where the experience must be consistent to be meaningful (personalisation, UI changes).

    **Session-level randomisation**: each session (visit) is independently randomised.
    - Pros: faster to accumulate data; works without user identification.
    - Cons: the same user may see different variants in different sessions — inconsistent experience, and the independence assumption for test statistics may be violated (multiple sessions from the same user are correlated).
    - Risk: if a user sees variant A Monday and variant B Wednesday and converts Wednesday, is that the treatment effect?

    **The key rule**: the randomisation unit and the analysis unit must match. If you randomise by user but analyse by session, your standard errors are incorrect (sessions from the same user are not independent). Always aggregate to the randomisation unit before computing statistics.

---

### Q14: What is a metric? How do you choose the primary metric for an A/B test?

??? "Show answer"
    A **metric** in the context of an A/B test is a quantitative measurement of user or system behaviour that serves as the basis for evaluating the experiment's success.

    Choosing the primary metric is one of the hardest and most consequential decisions in A/B testing.

    Good primary metrics are:
    - **Causally linked to the goal**: the mechanism should make sense. Don't use a proxy metric just because it moves.
    - **Sensitive enough** to detect the effects your experiment produces (low variance, responsive to the change).
    - **Not gameable**: the treatment should not be able to artificially inflate the metric without creating real value.
    - **User-centric**: ideally reflects user value, not just business value (these usually align long-term).
    - **Measurable in the experiment window**: don't use LTV (months) as a primary metric if your experiment runs 2 weeks.

    Common metric types:
    - **Rates / proportions**: click-through rate, conversion rate
    - **Averages**: average session duration, average revenue per user
    - **Ratios**: items sold / sessions, revenue / DAU
    - **Counts**: number of actions per user

    One primary metric. Supporting metrics are secondary. Having too many "primary" metrics is equivalent to having none — you'll always find one that's significant by chance.

---

### Q15: What is a guardrail metric?

??? "Show answer"
    A **guardrail metric** is a metric you monitor to ensure the experiment does not cause unacceptable harm, even if the primary metric improves. It is a safety constraint, not an optimisation target.

    You do not need guardrail metrics to improve — you need them to **not get worse** (they are held to a "do no harm" standard).

    Examples:
    - Primary metric: click-through rate. Guardrail: page load time (don't let latency degrade).
    - Primary metric: revenue per user. Guardrail: user satisfaction score / complaint rate.
    - Primary metric: engagement. Guardrail: user churn / unsubscribe rate.
    - Primary metric: ad revenue. Guardrail: organic search clicks (don't cannibalise organic traffic).

    How guardrails work in practice:
    - Define guardrail metrics and their acceptable bounds before the experiment.
    - If a guardrail metric significantly worsens, stop the experiment regardless of primary metric performance — the improvement isn't worth the side effect.
    - Guardrail metrics are often the business-critical metrics that you're not directly trying to optimise in this experiment.

    Guardrails are distinct from secondary metrics (which you're curious about) — you must act on a guardrail failure; a secondary metric movement is informational.

---

### Q16: What is the difference between ratio metrics and mean metrics in A/B testing?

??? "Show answer"
    **Mean metrics**: the average of a per-user (or per-observation) quantity, such as average revenue per user or average session duration. Computing the variance of a mean is straightforward using the standard formula: Var(x̄) = s²/n.

    **Ratio metrics**: the ratio of two aggregated quantities, such as click-through rate (total clicks / total impressions) or revenue per session (total revenue / total sessions). These are not the same as the mean of a per-user ratio.

    The challenge with ratio metrics: the numerator and denominator are both random variables. The variance of the ratio Y/X cannot be computed as simply as the variance of a mean. Naive standard errors are incorrect.

    Why this matters: if you use the wrong variance estimator, your confidence intervals and p-values are wrong. You may over- or under-reject the null hypothesis.

    Solutions:
    - **Delta method**: a first-order Taylor expansion to approximate the variance of a ratio (discussed next).
    - **Bootstrap**: resample users with replacement many times; compute the ratio each time; use the bootstrap distribution to estimate variance.
    - Use per-user ratio when possible: compute click-through rate per user, then average the per-user rates. This converts a ratio metric into a mean metric with correct variance.

---

### Q17: What is the delta method and why is it needed for ratio metrics?

??? "Show answer"
    The **delta method** is a technique for approximating the variance of a function of random variables using a first-order Taylor expansion.

    For a ratio metric R = Y/X, where Y is the sum of numerator values and X is the sum of denominator values (both averages across users):

    Var(R) ≈ (1/n²) × [Var(Y) / X̄² − 2 × (Ȳ/X̄) × Cov(Y, X) / X̄² + (Ȳ²/X̄⁴) × Var(X)]

    Or equivalently, defining yᵢ = yᵢ − (Ȳ/X̄) × xᵢ (the linearised metric per user), then:

    Var(R) ≈ Var(ȳ − R̄ × x̄) / n × X̄²

    This lets you compute the correct standard error for a ratio metric and apply a standard z-test.

    When the delta method is necessary:
    - Revenue per session: total revenue / total sessions
    - Click-through rate as a ratio: clicks / impressions
    - Any metric where the numerator and denominator vary together at the user level

    In practice, most experimentation platforms (Airbnb, Netflix, LinkedIn) implement the delta method internally. Knowing it exists and why it matters shows sophistication in A/B testing methodology.

    ```python
    import numpy as np

    def delta_method_var(y, x):
        """Variance of ratio metric E[Y]/E[X] using delta method"""
        n = len(y)
        r = np.mean(y) / np.mean(x)
        z = y - r * x  # linearised metric
        return np.var(z, ddof=1) / (n * np.mean(x)**2)

    # Example: revenue (y) per session (x) per user
    y = np.random.exponential(10, 1000)
    x = np.random.poisson(3, 1000)
    print(f"Ratio: {np.mean(y)/np.mean(x):.3f}")
    print(f"Delta method SE: {np.sqrt(delta_method_var(y, x)):.5f}")
    ```

---

### Q18: What is CUPED (Controlled-experiment Using Pre-Experiment Data)?

??? "Show answer"
    **CUPED** is a variance reduction technique for A/B tests that uses pre-experiment data on the same users to reduce noise in the treatment effect estimate — without changing the experiment design or randomisation.

    Motivation: the more noise in your metric, the larger the sample you need to detect an effect. If you can reduce variance, you can either detect smaller effects or reach significance with fewer users.

    The idea: instead of analysing the raw outcome metric Y, you analyse a **covariate-adjusted metric**:

    Ỹ = Y − θ × (X − E[X])

    Where X is the pre-experiment value of the same metric (or a correlated covariate) and θ = Cov(Y, X) / Var(X) (the OLS coefficient).

    The adjusted metric Ỹ has a smaller variance than Y when X and Y are positively correlated (which pre-experiment data usually is). Variance reduction of 20–50% is common, effectively multiplying your sample size.

    Properties:
    - Unbiasedness: the expected treatment effect estimate is unchanged — CUPED removes noise, not bias.
    - Requires pre-experiment data on the same users (impossible for new users).
    - The adjustment is done independently in treatment and control; randomisation is not affected.

    CUPED is equivalent to ANCOVA regression with the pre-experiment metric as a covariate.

    ```python
    import numpy as np
    from scipy import stats

    # Simulate: pre-experiment metric X, post-experiment metric Y
    n = 1000
    X = np.random.normal(100, 15, n)  # pre-experiment purchases
    treatment = np.concatenate([np.ones(n//2), np.zeros(n//2)])
    true_effect = 2.0
    Y = 0.7 * X + true_effect * treatment + np.random.normal(0, 10, n)

    # Raw t-test
    ctrl_Y = Y[treatment == 0]; trt_Y = Y[treatment == 1]
    raw_se = np.sqrt(np.var(trt_Y, ddof=1)/len(trt_Y) + np.var(ctrl_Y, ddof=1)/len(ctrl_Y))

    # CUPED adjustment
    ctrl_X = X[treatment == 0]; trt_X = X[treatment == 1]
    theta = np.cov(Y, X)[0, 1] / np.var(X, ddof=1)
    Y_adj = Y - theta * (X - np.mean(X))
    ctrl_Y_adj = Y_adj[treatment == 0]; trt_Y_adj = Y_adj[treatment == 1]
    cuped_se = np.sqrt(np.var(trt_Y_adj, ddof=1)/len(trt_Y_adj) + np.var(ctrl_Y_adj, ddof=1)/len(ctrl_Y_adj))

    print(f"Raw SE: {raw_se:.3f}")
    print(f"CUPED SE: {cuped_se:.3f}")
    print(f"Variance reduction: {1 - cuped_se**2/raw_se**2:.1%}")
    ```

---

### Q19: How do you handle multiple variants (A/B/C tests)?

??? "Show answer"
    A/B/C tests (or multi-arm tests) compare multiple variants simultaneously. The challenges are:

    **Multiple comparisons inflation**: with k variants, there are C(k, 2) pairwise comparisons. At α = 0.05, the probability of at least one false positive grows. You must correct for this.

    Common approaches:

    1. **One overall F-test first** (ANOVA for continuous metrics): test whether any variant differs from control. Only proceed to pairwise comparisons if the overall test is significant. This controls the family-wise error rate.

    2. **Bonferroni correction**: divide α by the number of pairwise comparisons. E.g., with 3 variants (A vs B, A vs C, B vs C), use α/3 = 0.0167 per test.

    3. **Tukey HSD** (Honest Significant Difference): designed for all pairwise comparisons while controlling FWER. More powerful than Bonferroni when all pairs are tested.

    4. **Dunnett's test**: designed specifically for comparing multiple treatment groups to a single control. More powerful than Bonferroni in this common A/B/C scenario.

    5. **FDR control (Benjamini-Hochberg)**: appropriate when you're willing to tolerate some false discoveries as long as most are real.

    Practical recommendation: define one primary comparison (e.g., best variant vs control) before seeing results. Treat other pairwise comparisons as exploratory.

    ```python
    from scipy.stats import f_oneway
    from statsmodels.stats.multicomp import pairwise_tukeyhsd
    import numpy as np

    control = np.random.normal(10, 2, 200)
    variant_b = np.random.normal(10.5, 2, 200)
    variant_c = np.random.normal(10.3, 2, 200)

    # Overall F-test
    f_stat, p = f_oneway(control, variant_b, variant_c)
    print(f"ANOVA: F={f_stat:.3f}, p={p:.4f}")

    # Post-hoc Tukey HSD
    all_data = np.concatenate([control, variant_b, variant_c])
    groups = ['control']*200 + ['B']*200 + ['C']*200
    result = pairwise_tukeyhsd(all_data, groups, alpha=0.05)
    print(result)
    ```

---

### Q20: What is a holdout group?

??? "Show answer"
    A **holdout group** (or holdback group) is a group of users who are permanently excluded from a feature after it launches — kept on the old experience to measure the long-term, sustained impact of the change.

    Why holdouts exist: standard A/B tests run for 1–4 weeks, during which novelty effects, learning curves, and ecosystem effects may not have stabilised. A holdout measures the true long-term effect by maintaining a control group post-launch.

    How it works:
    - At launch, 90–95% of users get the new feature. The remaining 5–10% are held back on the old experience.
    - The holdout continues for weeks or months after the feature ships.
    - Comparing the holdout group's metrics to the general population measures the long-term treatment effect.

    What holdouts capture that standard A/B tests miss:
    - **Novelty/change aversion decay**: effects that inflate or deflate short-term results
    - **Ecosystem and network effects**: how the feature changes user behaviour over time and through network interactions
    - **Compounding effects**: features that change user habits have growing effects over weeks

    Costs of holdouts:
    - You're deliberately giving some users an inferior experience (ethical and business trade-off)
    - Users in the holdout may be confused if they see inconsistent experiences (e.g., shared content from feature-enabled users)
    - Maintaining holdouts at scale is operationally complex

    Holdouts are common at large tech companies for significant product launches.
