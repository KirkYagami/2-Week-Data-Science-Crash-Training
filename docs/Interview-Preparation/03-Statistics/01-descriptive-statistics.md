# Descriptive Statistics

---

### Q1: When should you use mean, median, or mode?

??? "Show answer"
    Use **mean** when the data is roughly symmetric and free of extreme outliers — it uses every value and is algebraically tractable (you can average averages, for instance).

    Use **median** when the distribution is skewed or has outliers. Income, house prices, and response times are classic examples where the median is far more representative than the mean.

    Use **mode** for categorical data, or when you need the most-common value — e.g., the most-purchased product size, the most-frequent error code.

    A practical rule of thumb: if the mean and median diverge significantly, the distribution is skewed and the median is usually the better summary.

    ```python
    import numpy as np
    from scipy import stats

    data = [10, 12, 13, 14, 15, 100]  # outlier at 100
    print(np.mean(data))    # 27.3 — pulled up by outlier
    print(np.median(data))  # 13.5 — robust
    print(stats.mode(data).mode)  # no mode here; mode matters for discrete data
    ```

---

### Q2: What is variance and standard deviation?

??? "Show answer"
    **Variance** is the average squared deviation from the mean. Squaring serves two purposes: it makes all deviations positive, and it penalises large deviations more heavily.

    **Standard deviation** is the square root of variance, which brings the unit back to the original scale of the data — making it interpretable.

    Population variance: σ² = Σ(xᵢ − μ)² / N
    Sample variance: s² = Σ(xᵢ − x̄)² / (n − 1)

    The **n − 1** denominator (Bessel's correction) is used for sample variance because using n would systematically underestimate the true population variance. Dividing by n − 1 produces an unbiased estimator.

    In practice: standard deviation tells you, on average, how far individual observations fall from the mean. A low SD means values cluster tightly; a high SD means they're spread out.

    ```python
    import numpy as np

    data = [2, 4, 4, 4, 5, 5, 7, 9]
    print(np.var(data, ddof=0))   # population variance: 4.0
    print(np.var(data, ddof=1))   # sample variance: 4.57
    print(np.std(data, ddof=1))   # sample std dev: 2.14
    ```

---

### Q3: What is the difference between population and sample statistics?

??? "Show answer"
    A **population** is the entire group you care about. A **sample** is a subset drawn from that population.

    You almost never have access to the full population, so you compute **sample statistics** (x̄, s²) and use them to estimate **population parameters** (μ, σ²).

    Key differences in formulas:
    - Population mean: μ = Σxᵢ / N; Sample mean: x̄ = Σxᵢ / n (same formula, different meaning)
    - Population variance: divides by N; Sample variance: divides by n − 1

    The sample mean is an **unbiased estimator** of the population mean: E[x̄] = μ. The sample variance with n − 1 is an unbiased estimator of σ².

    Interview framing: when a recruiter asks about estimating a parameter, clarify whether you're treating your data as the full population or a sample from a larger one. This affects which formula and which hypothesis test you choose.

---

### Q4: What is a percentile / quantile?

??? "Show answer"
    The **p-th percentile** is the value below which p% of the data falls. The **50th percentile** is the median. The 25th and 75th percentiles are the first and third quartiles (Q1 and Q3).

    **Quantiles** are the same concept expressed as fractions (0–1) rather than percentages (0–100). The 0.9 quantile = 90th percentile.

    Percentiles are non-parametric — they don't assume a distribution. They're useful for:
    - Describing spread without being distorted by outliers
    - Setting SLAs (e.g., "p99 latency < 200 ms")
    - Reporting rank-based metrics

    ```python
    import numpy as np

    data = np.random.normal(loc=100, scale=15, size=1000)
    print(np.percentile(data, 25))   # Q1
    print(np.percentile(data, 50))   # median
    print(np.percentile(data, 75))   # Q3
    print(np.percentile(data, 99))   # p99 — useful for latency SLAs
    ```

---

### Q5: What is the interquartile range (IQR)?

??? "Show answer"
    The **IQR** is Q3 − Q1 — the range of the middle 50% of the data. It's a robust measure of spread that is not affected by outliers or the tails of the distribution.

    The IQR is used in:
    - **Box plots** to define the "box" and whisker lengths
    - **Outlier detection**: values below Q1 − 1.5×IQR or above Q3 + 1.5×IQR are flagged as outliers (Tukey's fence)

    IQR vs standard deviation: SD is sensitive to extreme values; IQR is not. For skewed or contaminated data, IQR is a better description of typical spread.

    ```python
    from scipy.stats import iqr
    import numpy as np

    data = [1, 2, 3, 4, 5, 6, 7, 100]  # outlier at 100
    q1, q3 = np.percentile(data, [25, 75])
    iqr_val = q3 - q1
    lower_fence = q1 - 1.5 * iqr_val
    upper_fence = q3 + 1.5 * iqr_val
    outliers = [x for x in data if x < lower_fence or x > upper_fence]
    print(outliers)  # [100]
    ```

---

### Q6: What is skewness? What is positive vs negative skew?

??? "Show answer"
    **Skewness** measures the asymmetry of a distribution around its mean.

    - **Positive skew (right-skewed)**: the tail extends to the right. Most values are low, but a few are very high. Mean > Median > Mode. Examples: income, house prices, insurance claim sizes.
    - **Negative skew (left-skewed)**: the tail extends to the left. Most values are high, but a few are very low. Mean < Median < Mode. Examples: age at retirement in many populations, exam scores with a ceiling effect.
    - **Zero skew**: symmetric, like a normal distribution.

    The rule of thumb for skewness values:
    - |skew| < 0.5: approximately symmetric
    - 0.5 ≤ |skew| < 1: moderately skewed
    - |skew| ≥ 1: highly skewed

    Why it matters for modelling: many algorithms assume normally distributed residuals. Skewed features may need a log or Box-Cox transform before being fed to a linear model.

    ```python
    from scipy.stats import skew
    import numpy as np

    data = np.random.lognormal(mean=0, sigma=1, size=1000)  # right-skewed
    print(skew(data))  # positive value expected
    ```

---

### Q7: What is kurtosis?

??? "Show answer"
    **Kurtosis** measures the "tailedness" of a distribution — how much probability mass is in the tails compared to a normal distribution.

    - **Leptokurtic** (kurtosis > 3, excess kurtosis > 0): heavy tails, sharp peak. More extreme values than a normal distribution. Financial returns often exhibit this.
    - **Platykurtic** (kurtosis < 3, excess kurtosis < 0): light tails, flat peak. Fewer extremes.
    - **Mesokurtic** (kurtosis = 3, excess = 0): normal distribution.

    Most software reports **excess kurtosis** (Fisher's definition): kurtosis − 3, so a normal distribution has excess kurtosis of 0.

    Kurtosis matters when:
    - Assessing tail risk (finance, insurance)
    - Checking normality assumptions before applying tests that require it
    - Understanding model residuals

    ```python
    from scipy.stats import kurtosis

    import numpy as np
    normal_data = np.random.normal(size=10000)
    heavy_tail = np.random.standard_t(df=3, size=10000)  # t with 3 df — heavy tails

    print(kurtosis(normal_data))   # near 0
    print(kurtosis(heavy_tail))    # positive, indicating heavy tails
    ```

---

### Q8: What is a box plot and what does it tell you?

??? "Show answer"
    A **box plot** (box-and-whisker plot) summarises the distribution of a dataset with five numbers: minimum (non-outlier), Q1, median, Q3, maximum (non-outlier).

    Components:
    - **Box**: spans Q1 to Q3 (the IQR)
    - **Line inside box**: median
    - **Whiskers**: extend to the furthest non-outlier values, typically Q1 − 1.5×IQR and Q3 + 1.5×IQR
    - **Points beyond whiskers**: potential outliers, plotted individually

    What you can read from a box plot:
    - **Central tendency**: where the median sits
    - **Spread**: width of the box and whisker length
    - **Skewness**: if the median is off-centre within the box, or one whisker is longer
    - **Outliers**: dots beyond the whiskers
    - **Comparisons**: placing multiple box plots side-by-side compares groups instantly

    Box plots are especially useful for comparing distributions across categories (e.g., salary by job title).

    ```python
    import matplotlib.pyplot as plt
    import numpy as np

    data = [np.random.normal(loc=m, scale=2, size=100) for m in [10, 12, 15]]
    plt.boxplot(data, labels=['Group A', 'Group B', 'Group C'])
    plt.show()
    ```

---

### Q9: What is the difference between a histogram and a density plot?

??? "Show answer"
    A **histogram** bins continuous data into intervals and counts the number of observations per bin. It is a frequency chart and the shape depends on the chosen bin width.

    A **density plot** (kernel density estimate, KDE) estimates the underlying continuous probability density function. It is smooth, integrates to 1, and is not affected by arbitrary bin-width choices.

    Key trade-offs:
    - Histograms are more interpretable for absolute counts; density plots are better for comparing shapes across groups with different sample sizes.
    - Histograms can obscure or reveal features depending on bin width; KDE has a bandwidth parameter that similarly needs tuning.
    - Density plots can extend into impossible regions (e.g., negative values for age) if not bounded properly.

    In practice, use a histogram when you want counts or when presenting to a non-technical audience. Use KDE when comparing distributions or overlaying multiple groups.

    ```python
    import matplotlib.pyplot as plt
    import numpy as np
    from scipy.stats import gaussian_kde

    data = np.random.normal(size=500)
    plt.hist(data, bins=30, density=True, alpha=0.5, label='Histogram')
    kde = gaussian_kde(data)
    x = np.linspace(data.min(), data.max(), 200)
    plt.plot(x, kde(x), label='KDE')
    plt.legend()
    plt.show()
    ```

---

### Q10: What is covariance vs correlation?

??? "Show answer"
    **Covariance** measures the direction of the linear relationship between two variables: positive covariance means they tend to move together; negative means they move in opposite directions. Its magnitude depends on the scale of the variables, so it is not directly comparable across different pairs.

    **Correlation** is standardised covariance — it divides by the product of the standard deviations, yielding a dimensionless value in [−1, 1]. This makes it comparable and interpretable:
    - +1: perfect positive linear relationship
    - −1: perfect negative linear relationship
    - 0: no linear relationship (but there could be a nonlinear one)

    Cov(X, Y) = E[(X − μX)(Y − μY)]
    Corr(X, Y) = Cov(X, Y) / (σX × σY)

    When to use each: covariance appears in portfolio theory and PCA; correlation is used for exploratory analysis and feature selection.

    ```python
    import numpy as np

    x = np.array([1, 2, 3, 4, 5])
    y = np.array([2, 4, 5, 4, 5])

    cov_matrix = np.cov(x, y)
    print("Covariance:", cov_matrix[0, 1])
    print("Correlation:", np.corrcoef(x, y)[0, 1])
    ```

---

### Q11: What is the difference between Pearson and Spearman correlation?

??? "Show answer"
    **Pearson correlation** measures the strength of the *linear* relationship between two continuous variables. It assumes both variables are continuous and roughly normally distributed, and it is sensitive to outliers.

    **Spearman correlation** is a rank-based measure. It converts both variables to ranks and then computes Pearson correlation on those ranks. It captures *monotonic* relationships (not necessarily linear) and is robust to outliers and non-normal distributions.

    When to use which:
    - Use Pearson when data is continuous, roughly normal, and you expect a linear relationship.
    - Use Spearman when data is ordinal, heavily skewed, has outliers, or the relationship may be monotonic but nonlinear.
    - For most EDA pipelines, Spearman is a safer default.

    A common interview gotcha: two variables can have Pearson ≈ 0 but Spearman ≠ 0 (nonlinear monotonic relationship), or both ≈ 0 with a clear nonlinear pattern (e.g., a U-shape).

    ```python
    from scipy.stats import pearsonr, spearmanr
    import numpy as np

    x = np.arange(1, 11)
    y = x ** 2  # nonlinear but perfectly monotonic

    print(pearsonr(x, y))   # r ≈ 0.97 (high but not 1)
    print(spearmanr(x, y))  # r = 1.0 (perfectly monotonic)
    ```

---

### Q12: When is the mean misleading?

??? "Show answer"
    The mean is misleading whenever the distribution is **skewed**, **multimodal**, or contains **outliers** — because the mean is pulled toward the tail or toward extreme values and no longer represents a "typical" observation.

    Common scenarios:
    - **Income data**: a few billionaires inflate the mean; the median income is far lower and more representative of the typical person.
    - **Response times / latency**: a small number of very slow requests can inflate the mean significantly. Median or p95/p99 are better.
    - **Multimodal distributions**: if you have two clusters (e.g., juniors and seniors in a salary dataset), the mean may fall in a gap between the two groups and represent nobody.
    - **Small samples with one extreme value**: one data entry error or genuine outlier can dominate the mean.

    The mean is also undefined or infinite for some heavy-tailed distributions (e.g., Cauchy distribution has no finite mean).

    Rule of thumb for interviews: always visualise your data before reporting summary statistics. Report median alongside the mean when you suspect skew.

---

### Q13: What is the coefficient of variation?

??? "Show answer"
    The **coefficient of variation (CV)** is the ratio of the standard deviation to the mean, expressed as a percentage:

    CV = (σ / μ) × 100%

    It measures *relative* variability — how large the spread is compared to the average. This makes it useful for comparing variability across variables measured on different scales.

    Example: if the average height of adults is 170 cm with SD = 10 cm, CV = 5.9%. If the average salary is $70,000 with SD = $20,000, CV = 28.6%. Despite the salary having a larger absolute SD, the CV lets you compare them meaningfully.

    Limitations:
    - Undefined when the mean is zero
    - Unreliable when the mean is close to zero (CV becomes huge and unstable)
    - Only meaningful for ratio-scale data with a meaningful zero

    CV is often used in quality control, finance (comparing asset volatility), and biological sciences.

    ```python
    import numpy as np
    from scipy.stats import variation

    data = [10, 12, 14, 13, 11, 15]
    cv = variation(data) * 100  # scipy returns CV as a fraction
    print(f"CV: {cv:.2f}%")
    ```

---

### Q14: What is the difference between nominal, ordinal, interval, and ratio data?

??? "Show answer"
    This is Stevens' typology of measurement scales. Each level adds properties:

    **Nominal**: categories with no intrinsic order. Operations: equality only. Examples: gender, colour, country, product category. You can count; you cannot add or rank.

    **Ordinal**: categories with a meaningful order, but the gaps between ranks are not necessarily equal. Examples: Likert scales (1–5 satisfaction), education level (high school < bachelor < master), star ratings. You can rank; you cannot say "5 stars is 5× better than 1 star."

    **Interval**: ordered with equal, meaningful spacing, but no true zero. Operations: addition, subtraction. Examples: temperature in Celsius/Fahrenheit, calendar year, IQ scores. You can say "30°C is 10° warmer than 20°C," but you cannot say "30°C is twice as hot as 15°C."

    **Ratio**: interval with a true zero that means absence of the quantity. Operations: all arithmetic including multiplication and ratios. Examples: height, weight, time duration, counts, temperature in Kelvin, income. "60 kg is twice 30 kg" is a valid statement.

    Why it matters for modelling:
    - Nominal → one-hot encode or use embeddings
    - Ordinal → label encode (preserving order) or target encode
    - Interval/Ratio → use directly; normalise/standardise as needed

---

### Q15: What is the central limit theorem?

??? "Show answer"
    The **Central Limit Theorem (CLT)** states: given a population with any distribution (with finite mean μ and finite variance σ²), the sampling distribution of the sample mean x̄ approaches a normal distribution as sample size n increases — regardless of the shape of the original population distribution.

    Specifically: x̄ ~ N(μ, σ²/n) for large enough n.

    Key implications:
    - You can apply z-tests and t-tests to sample means even when the underlying population is not normal, provided n is large enough (often cited as n ≥ 30 as a rough rule).
    - The standard deviation of the sampling distribution — called the **standard error** — is σ/√n. This tells you how much the sample mean varies from sample to sample.
    - As n increases, the standard error decreases, so larger samples give more precise estimates of μ.

    This theorem is foundational to nearly all frequentist inference — confidence intervals, hypothesis tests, A/B testing, and regression all rest on it.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt

    population = np.random.exponential(scale=2, size=100_000)  # not normal
    sample_means = [np.mean(np.random.choice(population, size=50)) for _ in range(5000)]

    plt.hist(sample_means, bins=50, density=True)
    plt.title("Sampling Distribution of the Mean (CLT in action)")
    plt.show()
    # Despite exponential population, sample means look normal
    ```
