# Bayesian Thinking

---

### Q1: What is the difference between Bayesian and frequentist statistics?

??? "Show answer"
    The fundamental divide is in what "probability" means and what can carry a probability.

    **Frequentist statistics**:
    - Probability = long-run frequency of outcomes in repeated experiments
    - Parameters are fixed, unknown constants — they do not have distributions
    - Data is random (varies from sample to sample)
    - You cannot make probability statements about parameters: "there is a 95% probability the true mean is in this interval" is not a valid frequentist statement
    - Inference is about the procedure: a 95% confidence interval procedure will contain the true value 95% of the time in repeated sampling

    **Bayesian statistics**:
    - Probability = degree of belief, which can be updated with evidence
    - Parameters are uncertain and can have probability distributions
    - Data is fixed (you observed what you observed)
    - You can make direct probability statements: "there is a 95% probability the true conversion rate is between 5% and 7%, given what we observed"
    - Inference uses Bayes' theorem to update a prior belief into a posterior belief

    Practical implications:
    - Frequentist methods require no prior; Bayesian methods require specifying one (which can be controversial — but also forces you to be explicit about assumptions)
    - Frequentist hypothesis tests have fixed rules (reject if p < α); Bayesian decisions are based on posterior probabilities and loss functions
    - Bayesian methods can incorporate prior information naturally; frequentist methods discard prior information
    - Bayesian computation is heavier (often requires MCMC); frequentist tests are typically fast closed-form calculations

---

### Q2: What is a prior distribution?

??? "Show answer"
    A **prior distribution** (or simply "the prior") encodes your belief about a parameter *before* observing any data. It is the starting point of Bayesian inference.

    The prior can be:
    - **Informative**: based on previous studies, domain expertise, or historical data. Example: you've run many A/B tests and know conversion rates rarely exceed 15%, so you'd use a Beta(2, 20) prior for a new test.
    - **Weakly informative**: a regularising prior that keeps estimates sensible without strongly constraining them. Example: Normal(0, 1) prior on regression coefficients (equivalent to ridge regularisation).
    - **Non-informative / vague**: intended to have minimal influence on the posterior. Example: Uniform(0, 1) for a probability. Note: true non-informativeness is elusive — uniform on p is not uniform on log(p).
    - **Conjugate**: chosen so the posterior has the same distributional family as the prior (discussed later).

    The prior is where Bayesian analysis is most often criticised: different priors lead to different posteriors, which critics say makes results subjective. The defence is that assumptions are always present — Bayesian methods make them explicit rather than hiding them in study design choices.

    As sample size grows, the likelihood dominates and the prior matters less: the data eventually overwhelms even a moderately wrong prior.

---

### Q3: What is a posterior distribution?

??? "Show answer"
    The **posterior distribution** is the distribution of a parameter after updating the prior with observed data using Bayes' theorem. It encodes everything you know about the parameter given both your prior beliefs and the evidence.

    Posterior ∝ Likelihood × Prior

    Or formally: P(θ | data) = P(data | θ) × P(θ) / P(data)

    The posterior is the central object of Bayesian inference — all decisions, summaries, and predictions derive from it:
    - **Point estimate**: posterior mean, median, or mode (MAP estimate)
    - **Credible interval**: the central 95% of the posterior distribution
    - **Prediction**: integrate over the posterior to get the posterior predictive distribution
    - **Decision**: minimise expected loss under the posterior

    As you collect more data, the posterior narrows (becomes more concentrated), reflecting increased certainty. With infinite data, the posterior collapses to a point mass at the true parameter value (under regularity conditions) — Bayesian and frequentist estimates converge.

    The posterior also serves as the next experiment's prior — Bayesian updating is sequential and coherent.

---

### Q4: What is a likelihood?

??? "Show answer"
    The **likelihood** L(θ | data) = P(data | θ) is the probability of the observed data as a function of the parameter θ. The data is fixed; θ varies.

    Critically, the likelihood is not a probability distribution over θ — it does not integrate to 1 over parameter space. It is a score that allows you to compare how plausible different parameter values are, given the observed data.

    In Bayes' theorem: the likelihood is the updating factor — it tells you how much to revise your prior in light of the data. A high likelihood for a given θ means that θ explains the observed data well.

    In maximum likelihood estimation (MLE), you find θ̂ = argmax L(θ | data) — the parameter value that makes the observed data most probable. MLE is purely frequentist; it doesn't use a prior.

    In Bayesian inference, the likelihood is multiplied by the prior to produce (an unnormalised) posterior:

    Posterior ∝ Likelihood × Prior

    The normalising constant P(data) = ∫ P(data | θ) P(θ) dθ is often intractable analytically — which is why MCMC exists.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.stats import binom

    # Observed: 7 heads in 10 flips
    p_grid = np.linspace(0, 1, 200)
    likelihood = binom.pmf(7, 10, p_grid)

    plt.plot(p_grid, likelihood)
    plt.xlabel('p (probability of heads)')
    plt.ylabel('Likelihood')
    plt.title('Likelihood function — peaks at MLE = 0.7')
    plt.show()
    ```

---

### Q5: State Bayes' theorem and explain each term.

??? "Show answer"
    **Bayes' theorem**:

    P(θ | data) = [P(data | θ) × P(θ)] / P(data)

    Each term:
    - **P(θ | data)** — **Posterior**: the updated belief about θ after seeing the data. This is what we want.
    - **P(data | θ)** — **Likelihood**: how probable the observed data is, given a specific value of θ. Measures how well θ explains the data.
    - **P(θ)** — **Prior**: the belief about θ before seeing data. Encodes domain knowledge or assumptions.
    - **P(data)** — **Marginal likelihood** (or evidence): the probability of the data averaged over all possible θ values. Acts as a normalising constant to ensure the posterior integrates to 1. P(data) = ∫ P(data | θ) P(θ) dθ.

    The intuition:
    - Start with a prior belief (P(θ))
    - Observe data that was generated by some θ
    - Ask how likely this data would be under each possible θ (likelihood)
    - Weight the prior by the likelihood; normalise
    - The result is an updated, data-informed belief (posterior)

    Because P(data) does not depend on θ, you often write: **Posterior ∝ Likelihood × Prior** — which is sufficient for MCMC sampling and most practical Bayesian computation.

---

### Q6: What is a conjugate prior?

??? "Show answer"
    A **conjugate prior** is a prior distribution that, when combined with a particular likelihood, produces a posterior of the same distributional family as the prior. This makes the update analytically tractable — you don't need numerical integration or MCMC.

    Common conjugate pairs:

    | Likelihood | Conjugate Prior | Posterior |
    |------------|----------------|-----------|
    | Binomial (successes/failures) | Beta(α, β) | Beta(α + successes, β + failures) |
    | Poisson (count data) | Gamma(α, β) | Gamma(α + Σxᵢ, β + n) |
    | Normal (known variance) | Normal(μ₀, σ₀²) | Normal (updated mean and variance) |
    | Multinomial | Dirichlet | Dirichlet |

    The Beta-Binomial conjugate is the most important for data science interviews. If you start with Beta(α, β) prior over a probability p and observe k successes in n trials, the posterior is Beta(α + k, β + n − k). Updating is just adding counts.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.stats import beta

    # Prior: Beta(2, 2) — weakly believes p ≈ 0.5
    alpha_prior, beta_prior = 2, 2

    # Observe: 7 successes in 10 trials
    successes, failures = 7, 3
    alpha_post = alpha_prior + successes
    beta_post = beta_prior + failures

    x = np.linspace(0, 1, 300)
    plt.plot(x, beta.pdf(x, alpha_prior, beta_prior), label='Prior')
    plt.plot(x, beta.pdf(x, alpha_post, beta_post), label='Posterior')
    plt.legend()
    plt.title(f'Beta-Binomial update: Prior Beta(2,2) → Posterior Beta({alpha_post},{beta_post})')
    plt.show()

    print(f"Posterior mean: {alpha_post / (alpha_post + beta_post):.3f}")
    ```

---

### Q7: What is Markov Chain Monte Carlo (MCMC)?

??? "Show answer"
    **MCMC** is a family of algorithms for sampling from a probability distribution — typically a posterior distribution that is intractable to compute analytically.

    Why it's needed: for most real Bayesian models, the posterior P(θ | data) ∝ P(data | θ) × P(θ) can be evaluated up to a normalising constant, but the normalising constant P(data) = ∫ P(data | θ) P(θ) dθ requires a high-dimensional integral that has no closed form.

    How MCMC works: construct a Markov chain whose stationary distribution is the target posterior. Run the chain long enough, and the distribution of samples approximates the posterior.

    Common algorithms:
    - **Metropolis-Hastings**: proposes a new θ from a proposal distribution; accepts or rejects based on the likelihood ratio. Simple but slow for high dimensions.
    - **Gibbs sampling**: samples each parameter from its conditional distribution given all others. Efficient when conditionals are available.
    - **Hamiltonian Monte Carlo (HMC) / NUTS**: uses gradient information (like momentum in physics) to make proposals that explore the posterior efficiently. Used by Stan and PyMC — the modern default.

    Diagnostics:
    - **Trace plots**: should look like "fuzzy caterpillars" (mixing well)
    - **R-hat**: should be ≈ 1.0 (chains have converged)
    - **Effective sample size**: should be high relative to the number of iterations

    ```python
    import pymc as pm
    import numpy as np

    # Bayesian coin flip
    data = np.array([1, 0, 1, 1, 0, 1, 1, 1, 0, 1])  # 7 heads in 10 flips
    with pm.Model() as model:
        p = pm.Beta('p', alpha=2, beta=2)  # prior
        obs = pm.Binomial('obs', n=10, p=p, observed=data.sum())
        trace = pm.sample(2000, tune=1000, return_inferencedata=True)

    pm.plot_posterior(trace, var_names=['p'])
    ```

---

### Q8: What is a credible interval and how does it differ from a confidence interval?

??? "Show answer"
    A **Bayesian credible interval** (or posterior interval) is an interval that contains the parameter with a specified probability, given the observed data.

    A 95% credible interval means: **P(θ ∈ [a, b] | data) = 0.95** — there is a 95% probability that the true parameter lies in this interval, given what we observed. This is the natural, intuitive interpretation most people want.

    A **frequentist 95% confidence interval** means: if you repeated this procedure on many different samples, 95% of the resulting intervals would contain the true parameter. It does NOT mean "there is a 95% probability the true parameter is in this particular interval." The true parameter is fixed; this interval either contains it or it doesn't.

    The difference in plain English:
    - Confidence interval: "95% of these intervals contain the truth" (property of the procedure)
    - Credible interval: "there's a 95% chance the truth is in here" (probability statement about the parameter)

    Numerically, credible intervals and confidence intervals are often similar — especially with large samples and weak priors. But they answer different questions and justify different statements.

    Types of credible intervals:
    - **Central credible interval**: equal tails (2.5% and 97.5%)
    - **Highest Posterior Density (HPD) interval**: the shortest interval that contains 95% of the posterior mass

    ```python
    import numpy as np
    from scipy.stats import beta

    # Posterior: Beta(9, 5) after observing 7 heads in 10 flips with Beta(2,2) prior
    alpha_post, beta_post = 9, 5
    lower, upper = beta.ppf([0.025, 0.975], alpha_post, beta_post)
    print(f"95% credible interval for p: [{lower:.3f}, {upper:.3f}]")
    print(f"Posterior mean: {alpha_post/(alpha_post+beta_post):.3f}")
    ```

---

### Q9: What is the problem with confidence intervals that Bayesian credible intervals solve?

??? "Show answer"
    The core problem: **frequentist confidence intervals cannot make probability statements about the parameter** — the statement "there is a 95% probability that μ is in [4.2, 7.8]" is not valid in frequentist statistics. The true μ is a fixed (non-random) quantity; the interval is what's random.

    This causes practical problems:

    1. **Misinterpretation is universal**: nearly everyone — including researchers, doctors, and journalists — interprets confidence intervals as credible intervals ("95% chance the true value is here"). They're using the correct intuition with the wrong mathematics.

    2. **No sequential updating**: frequentist CIs cannot be updated as data arrives without recomputing from scratch and risking multiple testing issues. Bayesian posteriors update naturally.

    3. **Decision-making**: you cannot make probability-weighted decisions from a confidence interval. "There is a 70% probability the new drug is better" is a Bayesian statement — it's exactly what doctors and policymakers need. The frequentist equivalent is "we reject H₀ at α = 0.05," which answers a different question.

    4. **One-sided bounds and asymmetric posteriors**: when distributions are skewed or bounded, frequentist CIs can be unintuitive or even extend into impossible regions. Bayesian HPD intervals naturally respect constraints.

    The Bayesian credible interval directly answers: "What values of θ are plausible, given what I observed?" — which is the question practitioners actually have.

---

### Q10: What is a Bayesian A/B test and how does it differ from frequentist A/B testing?

??? "Show answer"
    A **Bayesian A/B test** models conversion rates (or other metrics) as random variables with prior distributions, updates them with observed data, and makes decisions based on posterior probabilities.

    **Typical setup** (binary metric, e.g., conversion):
    - Prior: p_A ~ Beta(α, β), p_B ~ Beta(α, β) (usually weakly informative)
    - Observe: k_A successes in n_A trials, k_B successes in n_B trials
    - Posterior: p_A | data ~ Beta(α + k_A, β + n_A − k_A), similarly for p_B
    - Decision metric: P(p_B > p_A | data) — computed by simulation or analytically

    **Key differences from frequentist A/B testing**:

    | Aspect | Frequentist | Bayesian |
    |--------|-------------|----------|
    | Output | p-value, CI | Posterior distribution, P(B > A) |
    | Early stopping | Inflates Type I error | Valid at any time (no fixed horizon) |
    | Interpretation | "Reject H₀" | "B is better with 92% probability" |
    | Prior knowledge | Ignored | Incorporated |
    | Computation | Fast, closed-form | Requires sampling or conjugate tricks |
    | Required sample size | Fixed in advance | Can be monitored continuously |

    Bayesian A/B testing is popular because it allows continuous monitoring, produces interpretable probabilities, and naturally handles the early stopping problem that plagues frequentist tests.

    ```python
    import numpy as np
    from scipy.stats import beta

    # Observed data
    n_A, k_A = 1000, 102  # 10.2% conversion
    n_B, k_B = 1000, 115  # 11.5% conversion

    # Posterior samples (conjugate Beta)
    samples_A = beta.rvs(1 + k_A, 1 + n_A - k_A, size=100_000)
    samples_B = beta.rvs(1 + k_B, 1 + n_B - k_B, size=100_000)

    prob_B_better = np.mean(samples_B > samples_A)
    expected_lift = np.mean((samples_B - samples_A) / samples_A)
    print(f"P(B > A): {prob_B_better:.3f}")
    print(f"Expected relative lift: {expected_lift:.3%}")
    ```

---

### Q11: When would you choose Bayesian over frequentist methods?

??? "Show answer"
    Neither framework is universally better — the right choice depends on the problem, the data, and the decision you need to make.

    **Choose Bayesian when**:
    - You have **informative prior knowledge** (previous experiments, domain expertise) that would be wasteful to ignore.
    - You need **continuous monitoring** and the ability to stop whenever you have enough evidence (sequential decision-making).
    - **Small samples**: priors can regularise and prevent overfitting in ways that frequentist methods don't easily accommodate.
    - You need **interpretable probability statements** about parameters (clinical decision-making, risk assessment).
    - The problem is **hierarchical**: data has grouped structure (patients within hospitals, users within markets) — hierarchical Bayesian models handle this naturally.
    - You want **full uncertainty quantification**: a posterior gives you the full distribution of plausible parameter values, not just a point estimate and a symmetric interval.

    **Stick with frequentist when**:
    - You want **speed**: closed-form tests require no sampling.
    - The team and stakeholders expect p-values (regulatory environments, scientific publishing).
    - **No meaningful prior**: for novel problems where historical data is absent and any prior would be speculative.
    - **Adversarial settings**: where anyone could challenge your prior choice.
    - **Large n**: when samples are large, prior choice matters little and frequentist and Bayesian results converge.

    In practice, many data scientists use a hybrid: frequentist tests for routine A/B tests and Bayesian methods for multi-armed bandits, model calibration, and hierarchical modelling.

---

### Q12: What is hierarchical modelling?

??? "Show answer"
    **Hierarchical models** (also called multilevel models or mixed-effects models) handle data with a grouped structure, where observations within a group share some commonality — but groups themselves vary.

    The key insight: groups share a common prior distribution, so estimates for data-poor groups are "borrowed" from data-rich groups. This is called **partial pooling** or **shrinkage**.

    The three extremes:
    - **Complete pooling**: ignore group structure, estimate one global parameter. Biased — ignores real group differences.
    - **No pooling**: estimate each group independently. High variance — small groups have unreliable estimates.
    - **Partial pooling (hierarchical)**: estimate group-level parameters, but constrain them to come from a shared distribution. The optimal middle ground.

    Example: you want to estimate conversion rates for each of 50 marketing channels. Some channels have thousands of observations; others have 10. A hierarchical model shrinks the small-data channel estimates toward the overall mean, reducing variance without ignoring the group-level signal.

    Bayesian hierarchical models specify:
    - A likelihood: p(data | θ_group)
    - A group-level prior: θ_group ~ Normal(μ, σ²)
    - A hyperprior: p(μ, σ²) — distribution over the parameters of the group distribution

    ```python
    import pymc as pm
    import numpy as np

    # 10 channels, varying amounts of data
    n_obs = np.array([1000, 500, 200, 100, 50, 30, 20, 10, 5, 3])
    conversions = np.array([103, 48, 22, 12, 8, 6, 5, 3, 2, 1])

    with pm.Model() as hierarchical_model:
        # Hyperpriors
        mu = pm.Normal('mu', mu=0, sigma=1)
        sigma = pm.HalfNormal('sigma', sigma=1)

        # Group-level parameters
        theta = pm.Normal('theta', mu=mu, sigma=sigma, shape=len(n_obs))
        p = pm.Deterministic('p', pm.math.sigmoid(theta))

        # Likelihood
        obs = pm.Binomial('obs', n=n_obs, p=p, observed=conversions)
        trace = pm.sample(1000, tune=1000, return_inferencedata=True)
    ```

---

### Q13: What is the beta distribution and why is it used as a prior for probabilities?

??? "Show answer"
    The **beta distribution** Beta(α, β) is a continuous distribution on [0, 1], making it a natural prior for probabilities and proportions.

    PDF: f(x; α, β) = x^(α−1) × (1−x)^(β−1) / B(α, β), where B(α, β) is the beta function (normalising constant).

    Parameters and their intuition:
    - α and β can be thought of as pseudo-counts: α − 1 is like the number of prior successes, β − 1 like prior failures.
    - Mean: α / (α + β). The more data (large α + β), the more concentrated the distribution.
    - As α and β grow larger with fixed ratio, the distribution concentrates around its mean.

    Special cases:
    - Beta(1, 1) = Uniform(0, 1): complete ignorance about p.
    - Beta(α, α) with α > 1: symmetric, centred at 0.5, increasingly concentrated as α grows.
    - Beta(0.5, 0.5): Jeffreys prior — a non-informative choice.

    Why it's the natural prior for probabilities:
    1. **Support is [0, 1]**: matches the constraint on a probability — unlike a normal distribution, it never assigns mass outside valid probability range.
    2. **Conjugate to Binomial**: posterior after observing k successes in n trials is Beta(α + k, β + n − k) — a trivial, closed-form update.
    3. **Flexible shape**: can be uniform, unimodal, bimodal, U-shaped, or J-shaped by tuning α and β.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.stats import beta

    x = np.linspace(0.001, 0.999, 300)
    configs = [(1, 1, 'Uniform'), (2, 5, 'Biased low'), (5, 2, 'Biased high'), (10, 10, 'Near 0.5')]
    for a, b, label in configs:
        plt.plot(x, beta.pdf(x, a, b), label=f'Beta({a},{b}) — {label}')

    plt.legend()
    plt.xlabel('p')
    plt.title('Beta distribution shapes as priors for probability')
    plt.show()
    ```
