# Probability Distributions

---

### Q1: What is a normal distribution and what are its properties?

??? "Show answer"
    The **normal distribution** (Gaussian distribution) is a continuous, symmetric, bell-shaped probability distribution defined by two parameters: mean μ (location) and variance σ² (spread). Its PDF is:

    f(x) = (1 / (σ√(2π))) × exp(−(x−μ)² / (2σ²))

    Key properties:
    - **Symmetric** about the mean: mean = median = mode
    - **Unimodal**: single peak at the mean
    - **Asymptotic**: tails approach but never touch zero
    - Completely characterised by μ and σ²; all other moments (skewness = 0, excess kurtosis = 0) follow
    - **Stability**: a linear combination of independent normals is also normal
    - **Reproducibility**: if X ~ N(μ₁, σ₁²) and Y ~ N(μ₂, σ₂²) independently, then X + Y ~ N(μ₁+μ₂, σ₁²+σ₂²)

    Why it appears everywhere:
    - The Central Limit Theorem guarantees sample means converge to normal regardless of the original distribution.
    - Many natural phenomena are sums of many small independent factors (heights, measurement errors).
    - It is mathematically tractable: conjugate prior, closed-form moments, easy to compute probabilities.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.stats import norm

    x = np.linspace(-4, 4, 300)
    plt.plot(x, norm.pdf(x, loc=0, scale=1), label='N(0,1)')
    plt.plot(x, norm.pdf(x, loc=0, scale=2), label='N(0,4)')
    plt.legend()
    plt.title('Normal distributions with different variances')
    plt.show()
    ```

---

### Q2: What is the empirical rule (68-95-99.7)?

??? "Show answer"
    The **empirical rule** (also called the 68-95-99.7 rule or three-sigma rule) describes how probability is distributed for a normal distribution:

    - **68%** of values fall within **μ ± 1σ**
    - **95%** of values fall within **μ ± 2σ** (more precisely 1.96σ)
    - **99.7%** of values fall within **μ ± 3σ**

    This means only 0.3% of values fall beyond 3 standard deviations from the mean — which forms the basis for defining "outliers" and setting control limits in statistical process control.

    Practical uses:
    - **Anomaly detection**: a z-score > 3 or < −3 suggests an unusual observation.
    - **Quality control**: 6-sigma manufacturing means only 3.4 defects per million opportunities.
    - **Finance**: "value at risk" calculations rely on knowing the percentage of outcomes beyond a threshold.

    Important caveat: the empirical rule applies to *normal* distributions. For heavy-tailed distributions, far more data falls beyond 3σ than 0.3%. Always check your distributional assumption before applying this rule.

    ```python
    from scipy.stats import norm

    print(f"Within 1σ: {norm.cdf(1) - norm.cdf(-1):.4f}")   # 0.6827
    print(f"Within 2σ: {norm.cdf(2) - norm.cdf(-2):.4f}")   # 0.9545
    print(f"Within 3σ: {norm.cdf(3) - norm.cdf(-3):.4f}")   # 0.9973
    ```

---

### Q3: What is a standard normal distribution and what is a Z-score?

??? "Show answer"
    The **standard normal distribution** is a normal distribution with μ = 0 and σ = 1, written Z ~ N(0, 1). It is the reference distribution for all normal probability calculations.

    Any normal random variable X ~ N(μ, σ²) can be standardised:
    Z = (X − μ) / σ

    This Z-score tells you how many standard deviations X is above or below the mean.

    Why standardise?
    - All normal distributions can be mapped to a single table (the Z-table or standard normal table).
    - Z-scores make values from different scales directly comparable (e.g., a student's performance on two different exams).
    - They are used in hypothesis testing (z-tests), feature scaling, and anomaly detection.

    Z-scores in context:
    - |Z| > 2: observation is in the outer 5% — worth investigating
    - |Z| > 3: observation is in the outer 0.3% — likely an outlier or data error
    - Z is dimensionless, which makes it useful for comparing across different units

    ```python
    import numpy as np
    from scipy import stats

    data = np.array([150, 160, 170, 180, 250])  # 250 is the outlier
    z_scores = stats.zscore(data)
    print(z_scores)
    # 250 will have a z-score > 2, flagging it as unusual
    ```

---

### Q4: What is the t-distribution and how does it differ from the normal distribution?

??? "Show answer"
    The **t-distribution** (Student's t-distribution) is a symmetric, bell-shaped distribution similar to the normal but with **heavier tails**. It is parameterised by **degrees of freedom (df)**, which controls how heavy the tails are.

    As df → ∞, the t-distribution converges to the standard normal. For small df (e.g., df = 1, the Cauchy distribution), the tails are extremely heavy.

    Key differences from normal:
    - **Heavier tails**: more probability mass in the extremes — reflects greater uncertainty with small samples.
    - **Shape depends on df**: more data → more df → closer to normal.
    - The t-distribution has variance df/(df−2) for df > 2, which is always greater than 1 (the variance of the standard normal).

    Where it comes from: when you estimate the population mean from a sample, you replace the unknown σ with the sample estimate s. This substitution introduces extra uncertainty that inflates the tails — the t-distribution quantifies this uncertainty.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.stats import t, norm

    x = np.linspace(-5, 5, 300)
    plt.plot(x, norm.pdf(x), label='Normal')
    plt.plot(x, t.pdf(x, df=3), label='t (df=3)')
    plt.plot(x, t.pdf(x, df=10), label='t (df=10)')
    plt.legend()
    plt.title('t-distribution vs Normal — note heavier tails for low df')
    plt.show()
    ```

---

### Q5: When would you use a t-distribution vs normal distribution?

??? "Show answer"
    Use the **t-distribution** when:
    - Sample size is small (rough rule: n < 30)
    - The population standard deviation σ is **unknown** and must be estimated from the sample
    - You're computing confidence intervals or p-values for a mean

    Use the **normal distribution** when:
    - Sample size is large (n ≥ 30 as a common rule of thumb) — the CLT kicks in and the t-distribution approximates normal anyway
    - The population standard deviation σ is **known** (rare in practice)
    - You're working with proportions with large n (normal approximation to binomial)

    Practical reality: for n ≥ 30, the difference between t and normal critical values is negligible (e.g., at df=30, the t critical value at α=0.05 two-tailed is 2.042 vs 1.96 for normal). Most practitioners use t-tests throughout since it's always valid; using normal when you should use t leads to overly narrow intervals.

    ```python
    from scipy.stats import t, norm

    alpha = 0.05
    # Two-tailed critical values
    for df in [5, 10, 30, 100, 1000]:
        print(f"df={df}: t critical = {t.ppf(1 - alpha/2, df):.4f}")
    print(f"Normal critical = {norm.ppf(1 - alpha/2):.4f}")
    # Notice how t converges to normal as df increases
    ```

---

### Q6: What is the binomial distribution?

??? "Show answer"
    The **binomial distribution** models the number of successes in n independent Bernoulli trials, where each trial succeeds with probability p.

    X ~ Binomial(n, p)
    PMF: P(X = k) = C(n, k) × pᵏ × (1−p)ⁿ⁻ᵏ

    Parameters and moments:
    - Mean: E[X] = np
    - Variance: Var(X) = np(1−p)
    - Range: X ∈ {0, 1, 2, …, n}

    Conditions for using binomial:
    1. Fixed number of trials n
    2. Each trial has exactly two outcomes (success/failure)
    3. Constant probability p across trials
    4. Trials are independent

    Real-world examples:
    - Number of users who click an ad out of n impressions (each impression is a Bernoulli trial)
    - Number of defective items in a batch of n
    - Number of email recipients who open the email

    When n is large and p is not close to 0 or 1, the binomial is well approximated by N(np, np(1−p)). When p is very small and n is large, Poisson(λ=np) is a good approximation.

    ```python
    from scipy.stats import binom
    import matplotlib.pyplot as plt

    n, p = 20, 0.3
    x = range(0, n + 1)
    plt.bar(x, binom.pmf(x, n, p))
    plt.xlabel('Number of successes')
    plt.title(f'Binomial(n={n}, p={p})')
    plt.show()

    print(f"P(X=6): {binom.pmf(6, n, p):.4f}")
    print(f"P(X<=6): {binom.cdf(6, n, p):.4f}")
    ```

---

### Q7: What is the Poisson distribution and when is it used?

??? "Show answer"
    The **Poisson distribution** models the number of events occurring in a fixed interval (time, area, volume) when events happen at a constant average rate λ and independently of each other.

    X ~ Poisson(λ)
    PMF: P(X = k) = e^{−λ} × λᵏ / k!

    Key property: mean = variance = λ. If your observed variance differs substantially from the mean, the Poisson assumption may be violated (overdispersion is common — use negative binomial instead).

    When to use Poisson:
    - Number of customer support tickets per hour
    - Number of server errors per minute
    - Number of rare events (accidents, equipment failures) per period
    - Number of mutations in a DNA strand per base pair

    Requirements:
    1. Events are independent
    2. The average rate λ is constant over the interval
    3. Two events cannot occur at exactly the same instant (no batching)

    As λ → ∞, Poisson approaches N(λ, λ). As a rule, normal approximation is good when λ > 10.

    ```python
    from scipy.stats import poisson
    import numpy as np

    lam = 4  # average 4 events per hour
    print(f"P(X=0): {poisson.pmf(0, lam):.4f}")  # probability of zero events
    print(f"P(X<=3): {poisson.cdf(3, lam):.4f}")

    # Check mean ≈ variance
    data = poisson.rvs(lam, size=10000)
    print(f"Sample mean: {data.mean():.2f}, Sample variance: {data.var():.2f}")
    ```

---

### Q8: What is the exponential distribution?

??? "Show answer"
    The **exponential distribution** models the time between events in a Poisson process — i.e., how long you wait for the next event when events occur at a constant rate.

    X ~ Exponential(λ)
    PDF: f(x) = λe^{−λx}, x ≥ 0
    Mean: 1/λ; Variance: 1/λ²

    The most important property of the exponential distribution is **memorylessness**: P(X > s + t | X > s) = P(X > t). The distribution "forgets" how long it has already waited. This means the remaining waiting time is always distributed the same way, regardless of how long you've already waited.

    Common applications:
    - Time between customer arrivals
    - Time until a machine fails
    - Time between network packets
    - Time until a radioactive atom decays

    When memorylessness is violated (e.g., equipment wear-out increases failure rate over time), use the Weibull distribution instead.

    ```python
    from scipy.stats import expon
    import numpy as np

    lam = 0.5  # rate: 0.5 events per unit time -> mean time = 2
    print(f"Mean time between events: {1/lam}")
    print(f"P(X > 3): {1 - expon.cdf(3, scale=1/lam):.4f}")

    # Verify memorylessness
    # P(X > 5 | X > 2) should equal P(X > 3)
    p_x_gt_5 = 1 - expon.cdf(5, scale=1/lam)
    p_x_gt_2 = 1 - expon.cdf(2, scale=1/lam)
    print(f"P(X>5|X>2) = {p_x_gt_5/p_x_gt_2:.4f}")
    print(f"P(X>3) = {1 - expon.cdf(3, scale=1/lam):.4f}")
    ```

---

### Q9: What is the chi-squared distribution?

??? "Show answer"
    The **chi-squared distribution** with k degrees of freedom is the distribution of the sum of squares of k independent standard normal random variables:

    If Z₁, Z₂, …, Zₖ ~ N(0,1) independently, then X = Z₁² + Z₂² + … + Zₖ² ~ χ²(k)

    Properties:
    - Support: x ≥ 0 (sum of squares is non-negative)
    - Mean: k; Variance: 2k
    - Right-skewed, especially for small k; approaches normal as k grows large
    - Additive: if X ~ χ²(m) and Y ~ χ²(n) independently, then X + Y ~ χ²(m+n)

    Where it appears:
    - **Chi-squared test of independence**: tests if two categorical variables are related
    - **Chi-squared goodness-of-fit test**: tests if observed frequencies match expected
    - **Confidence intervals for variance**: if the population is normal, (n−1)s²/σ² ~ χ²(n−1)
    - Hypothesis tests on variance and covariance matrices

    ```python
    from scipy.stats import chi2
    import numpy as np

    # Chi-squared test statistic for categorical data
    observed = np.array([10, 20, 30, 40])
    expected = np.array([25, 25, 25, 25])  # uniform distribution hypothesis
    chi_stat = np.sum((observed - expected)**2 / expected)
    df = len(observed) - 1
    p_value = 1 - chi2.cdf(chi_stat, df)
    print(f"Chi² = {chi_stat:.2f}, df = {df}, p = {p_value:.4f}")
    ```

---

### Q10: What is the F-distribution?

??? "Show answer"
    The **F-distribution** is the ratio of two independent chi-squared random variables, each divided by their degrees of freedom:

    F = (χ²(d₁)/d₁) / (χ²(d₂)/d₂)

    It is parameterised by two degrees of freedom: d₁ (numerator) and d₂ (denominator). The distribution is right-skewed and has support on x ≥ 0.

    Where the F-distribution appears:
    - **ANOVA (Analysis of Variance)**: the F-statistic compares between-group variance to within-group variance. A large F suggests the groups have different means.
    - **F-test for equality of variances**: tests if two populations have the same variance.
    - **Regression**: the overall F-test checks if any predictor in a linear model has a non-zero coefficient.
    - **Nested model comparison**: comparing a restricted model to an unrestricted one.

    Relationship to other distributions:
    - If X ~ F(d₁, d₂), then 1/X ~ F(d₂, d₁)
    - t² ~ F(1, n−1), so the t-test is a special case of the F-test

    ```python
    from scipy.stats import f

    # F-distribution with 3 and 36 degrees of freedom (e.g., 4 groups, 10 per group ANOVA)
    f_stat = 4.5
    p_value = 1 - f.cdf(f_stat, dfn=3, dfd=36)
    print(f"p-value: {p_value:.4f}")
    ```

---

### Q11: What is the difference between a PDF and a CDF?

??? "Show answer"
    **PDF (Probability Density Function)**: for a continuous random variable, f(x) describes the density of probability at each point. The probability of X falling in an interval is the area under the PDF: P(a ≤ X ≤ b) = ∫ₐᵇ f(x)dx. The PDF can be greater than 1 at a point (it's a density, not a probability).

    **CDF (Cumulative Distribution Function)**: F(x) = P(X ≤ x) — the probability that the random variable takes a value ≤ x. The CDF:
    - Is always non-decreasing
    - Ranges from 0 (as x → −∞) to 1 (as x → +∞)
    - Is continuous for continuous random variables
    - Is the integral of the PDF: F(x) = ∫_{-∞}^{x} f(t)dt
    - The PDF is the derivative of the CDF: f(x) = F'(x)

    For discrete variables: the PMF gives P(X = x); the CDF is F(x) = Σ_{t ≤ x} P(X = t).

    Practical uses:
    - CDF for computing percentiles: the p-th percentile is x such that F(x) = p (the quantile function is the inverse CDF)
    - CDF for hypothesis testing p-values: P(test statistic ≥ observed value) = 1 − F(observed)
    - PDF for visualising the shape of distributions

    ```python
    from scipy.stats import norm
    import numpy as np
    import matplotlib.pyplot as plt

    x = np.linspace(-4, 4, 300)
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))
    ax1.plot(x, norm.pdf(x)); ax1.set_title('PDF')
    ax2.plot(x, norm.cdf(x)); ax2.set_title('CDF')
    plt.show()

    # P(X between -1 and 1)
    print(norm.cdf(1) - norm.cdf(-1))  # ~0.683
    ```

---

### Q12: How do you test whether data is normally distributed?

??? "Show answer"
    There are several approaches, ranging from visual to formal:

    **Visual methods**:
    - **Histogram**: does it look bell-shaped and symmetric?
    - **Q-Q plot (quantile-quantile plot)**: plots observed quantiles against theoretical normal quantiles. Points should fall along the diagonal line if the data is normal. Deviations indicate departures from normality (S-curves indicate skew; heavy tails show as points curving away at the ends).

    **Formal statistical tests**:
    - **Shapiro-Wilk test**: most powerful for small samples (n < 50); tests H₀: data comes from a normal distribution. Reject H₀ if p < 0.05.
    - **Kolmogorov-Smirnov test**: compares empirical CDF to theoretical CDF. Less powerful than Shapiro-Wilk for normality.
    - **D'Agostino-Pearson test**: based on skewness and kurtosis.
    - **Anderson-Darling test**: gives more weight to the tails.

    **Practical advice**: for large samples (n > 1000), formal tests will reject normality for trivially small departures that have no practical impact. Always combine visual inspection with formal tests, and ask whether the deviation from normality actually matters for your downstream analysis. Many procedures are robust to mild non-normality, especially with large samples (CLT).

    ```python
    from scipy import stats
    import numpy as np

    data = np.random.normal(size=100)
    # Shapiro-Wilk
    stat, p = stats.shapiro(data)
    print(f"Shapiro-Wilk: statistic={stat:.4f}, p={p:.4f}")

    # Q-Q plot
    import matplotlib.pyplot as plt
    stats.probplot(data, dist='norm', plot=plt)
    plt.show()
    ```

---

### Q13: What is a log-normal distribution?

??? "Show answer"
    A random variable X follows a **log-normal distribution** if log(X) is normally distributed. Equivalently, X = e^Y where Y ~ N(μ, σ²).

    Properties:
    - Support: x > 0 (always positive — useful for modelling quantities that can't be negative)
    - Right-skewed: mean > median > mode
    - Mean: E[X] = e^{μ + σ²/2}; Median: e^μ
    - The product of log-normal variables is log-normal (analogous to the sum of normals being normal)

    When it appears:
    - **Financial returns**: stock prices are often modelled as log-normal (Black-Scholes model)
    - **Income and wealth distributions**
    - **File sizes, city populations, biological measurements** (anything that grows multiplicatively)
    - **Response time / latency data** in systems

    If you're working with log-normal data, taking the log transforms it to normal, after which standard regression and inference apply. Always log-transform right-skewed positive data before fitting linear models.

    ```python
    import numpy as np
    from scipy.stats import lognorm
    import matplotlib.pyplot as plt

    # Generate log-normal data
    data = np.random.lognormal(mean=0, sigma=1, size=1000)
    print(f"Skewness: {((data - data.mean())**3).mean() / data.std()**3:.2f}")

    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))
    ax1.hist(data, bins=50); ax1.set_title('Log-normal data')
    ax2.hist(np.log(data), bins=50); ax2.set_title('After log transform → Normal')
    plt.show()
    ```

---

### Q14: What is a uniform distribution?

??? "Show answer"
    The **continuous uniform distribution** assigns equal probability density to all values in an interval [a, b].

    PDF: f(x) = 1/(b−a) for a ≤ x ≤ b; 0 otherwise
    Mean: (a + b) / 2; Variance: (b−a)² / 12

    The **discrete uniform distribution** assigns equal probability 1/n to each of n outcomes (e.g., a fair die).

    Why the uniform distribution matters:
    - **Simulation**: random number generators produce U(0,1) variates; inverse transform sampling converts these to any desired distribution.
    - **Prior distributions**: a uniform prior on [0,1] represents complete ignorance about a probability parameter (though this is debated — it is not always "non-informative" under reparameterisation).
    - **P-values under the null hypothesis** are uniformly distributed — if the null is true, p-values are U(0,1). This is the basis for checking test calibration.
    - Random shuffling and sampling without replacement use uniform draws.

    ```python
    import numpy as np
    from scipy.stats import uniform

    # Inverse transform sampling: generate exponential from uniform
    u = np.random.uniform(size=10000)
    lam = 2
    exponential_samples = -np.log(1 - u) / lam  # inverse CDF of exponential

    # Verify
    print(f"Theoretical mean: {1/lam}")
    print(f"Sample mean: {exponential_samples.mean():.3f}")
    ```

---

### Q15: What is the relationship between the Poisson and exponential distributions?

??? "Show answer"
    The **Poisson** and **exponential** distributions are two sides of the same process — the **Poisson process** — viewed from different perspectives:

    - **Poisson distribution**: count the number of events in a fixed time interval. If events arrive at rate λ per unit time, the number of events in interval T follows Poisson(λT).
    - **Exponential distribution**: measure the time between consecutive events. If events arrive at rate λ, the waiting time between events follows Exponential(λ) with mean 1/λ.

    They are connected through the Poisson process:
    - If N(t) ~ Poisson(λt) (event count), then the inter-arrival times T₁, T₂, … are i.i.d. Exponential(λ).
    - The first arrival time after time 0 follows Exponential(λ).
    - The sum of k exponential(λ) variables follows Gamma(k, λ) — the time until the k-th arrival.

    Practical example: if a call centre receives calls at rate 5 per minute (Poisson), the time between consecutive calls follows Exponential(5), with mean waiting time of 1/5 = 0.2 minutes = 12 seconds.

    ```python
    import numpy as np
    from scipy.stats import poisson, expon

    lam = 5  # rate: 5 events per unit time

    # Simulate the Poisson process
    inter_arrival_times = np.random.exponential(scale=1/lam, size=1000)
    arrival_times = np.cumsum(inter_arrival_times)

    # Count events in [0, 1]
    n_in_unit = np.sum(arrival_times <= 1)
    print(f"Events in [0,1]: {n_in_unit}, Poisson mean: {lam}")

    # Over many simulations, this count should be Poisson(lam)
    counts = [np.sum(np.cumsum(np.random.exponential(1/lam, 100)) <= 1) for _ in range(10000)]
    print(f"Simulated mean: {np.mean(counts):.2f}, Poisson mean: {lam}")
    ```
