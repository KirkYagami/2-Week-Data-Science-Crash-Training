# Probability

---

### Q1: What is conditional probability?

??? "Show answer"
    **Conditional probability** is the probability of event A occurring *given* that event B has already occurred. It is written P(A | B) and defined as:

    P(A | B) = P(A ∩ B) / P(B),  provided P(B) > 0

    Conditioning narrows the sample space from the full universe to only the outcomes where B is true, and asks what fraction of those also satisfy A.

    Intuition: in a standard deck of 52 cards, P(Ace) = 4/52. But P(Ace | card is a spade) = 1/13 — we're now restricted to the 13 spades, only one of which is an ace.

    Why it matters in data science:
    - Naive Bayes classifiers are built entirely on conditional probabilities.
    - Causal inference requires distinguishing P(outcome | treatment) from P(outcome | treatment, confounders).
    - A/B test metrics are conditional: "conversion rate given user saw variant B."

    ```python
    import pandas as pd

    # Example: P(churn | premium subscriber)
    df = pd.DataFrame({'premium': [1,1,0,0,1,0,1,0], 'churn': [0,1,1,0,0,1,0,1]})
    p_churn_given_premium = df[df['premium'] == 1]['churn'].mean()
    print(p_churn_given_premium)
    ```

---

### Q2: What is Bayes' theorem and give a real example?

??? "Show answer"
    **Bayes' theorem** relates the conditional probability P(A | B) to its reverse P(B | A):

    P(A | B) = [P(B | A) × P(A)] / P(B)

    In words: the probability of A given B equals the likelihood of B given A, scaled by the prior probability of A, divided by the total probability of B.

    **Real example — medical test**:
    - Disease prevalence: P(Disease) = 0.01 (1% of population)
    - Test sensitivity: P(Positive | Disease) = 0.95
    - False positive rate: P(Positive | No Disease) = 0.05
    - P(No Disease) = 0.99

    What is P(Disease | Positive test)?

    P(Positive) = P(Pos | Disease)×P(Disease) + P(Pos | No Disease)×P(No Disease)
               = 0.95×0.01 + 0.05×0.99 = 0.0095 + 0.0495 = 0.059

    P(Disease | Positive) = (0.95 × 0.01) / 0.059 ≈ 0.161

    Despite a 95% sensitive test, a positive result only means ~16% chance of actually having the disease — because the disease is rare. This counterintuitive result is why base rates matter so much.

    ```python
    p_disease = 0.01
    p_pos_given_disease = 0.95
    p_pos_given_no_disease = 0.05

    p_pos = p_pos_given_disease * p_disease + p_pos_given_no_disease * (1 - p_disease)
    p_disease_given_pos = (p_pos_given_disease * p_disease) / p_pos
    print(f"P(Disease | Positive): {p_disease_given_pos:.3f}")
    ```

---

### Q3: What is the difference between independent and mutually exclusive events?

??? "Show answer"
    These are often confused but are conceptually distinct — and they are almost mutually exclusive themselves.

    **Independent events**: A and B are independent if knowing that A occurred gives no information about whether B occurred. Formally: P(A ∩ B) = P(A) × P(B), equivalently P(A | B) = P(A).

    Example: flipping a fair coin twice. The result of the first flip has no bearing on the second.

    **Mutually exclusive events**: A and B cannot both occur at the same time. Formally: P(A ∩ B) = 0.

    Example: rolling a die — you cannot get both a 3 and a 5 on the same roll.

    Key distinction: if A and B are mutually exclusive *and* have non-zero probability, they are **not** independent. Why? If A occurs, B definitely doesn't — so knowing A happened tells you a lot about B. P(B | A) = 0 ≠ P(B).

    Independent events *can* occur simultaneously; mutually exclusive events cannot.

    Interview tip: the formula P(A or B) = P(A) + P(B) only holds for mutually exclusive events. The general formula is P(A ∪ B) = P(A) + P(B) − P(A ∩ B).

---

### Q4: What is the law of total probability?

??? "Show answer"
    The **law of total probability** lets you compute the probability of an event A by partitioning the sample space into mutually exclusive, exhaustive events B₁, B₂, …, Bₙ:

    P(A) = Σᵢ P(A | Bᵢ) × P(Bᵢ)

    It is the denominator in Bayes' theorem and is what makes Bayes' theorem computable.

    **Example**: A company has three data centres: DC1 handles 50% of traffic, DC2 handles 30%, DC3 handles 20%. The probability of a request failing is 1%, 2%, and 5% respectively.

    P(failure) = P(fail|DC1)×P(DC1) + P(fail|DC2)×P(DC2) + P(fail|DC3)×P(DC3)
               = 0.01×0.5 + 0.02×0.3 + 0.05×0.2
               = 0.005 + 0.006 + 0.010 = 0.021 (2.1% overall failure rate)

    ```python
    p_dc = [0.5, 0.3, 0.2]
    p_fail_given_dc = [0.01, 0.02, 0.05]
    p_total_failure = sum(p * f for p, f in zip(p_dc, p_fail_given_dc))
    print(f"Overall failure rate: {p_total_failure:.3f}")
    ```

---

### Q5: What is expected value?

??? "Show answer"
    The **expected value** (or expectation) of a random variable is its probability-weighted average — the long-run average value if you repeated the experiment infinitely many times.

    For a discrete random variable: E[X] = Σ xᵢ × P(X = xᵢ)
    For a continuous random variable: E[X] = ∫ x × f(x) dx

    Properties:
    - **Linearity**: E[aX + b] = aE[X] + b; E[X + Y] = E[X] + E[Y] (no independence required)
    - E[XY] = E[X]×E[Y] only if X and Y are independent

    **Example**: a game where you roll a die. If it lands on 6, you win $10; otherwise you lose $1. E[payout] = (1/6)×10 + (5/6)×(−1) = 10/6 − 5/6 = 5/6 ≈ $0.83. A positive expected value means you should play.

    Expected value is the foundation of:
    - Decision theory and risk analysis
    - Reinforcement learning (value functions)
    - Evaluating classifiers (expected cost under a loss matrix)

---

### Q6: What is the difference between permutations and combinations?

??? "Show answer"
    Both count ways to select items from a set, but they differ on whether order matters.

    **Permutations**: order matters. P(n, k) = n! / (n − k)!

    Example: how many ways can you arrange 3 people in 3 positions from a group of 5?
    P(5, 3) = 5! / 2! = 60

    **Combinations**: order does not matter. C(n, k) = n! / [k! × (n − k)!]

    Example: how many ways can you choose a committee of 3 from 5 people?
    C(5, 3) = 10 (the committee {Alice, Bob, Carol} is the same regardless of who is "first")

    Relationship: C(n, k) = P(n, k) / k! — combinations are permutations with the k! orderings of the chosen items divided out.

    In data science contexts: permutations appear in ranking/recommendation metrics (like NDCG where position matters); combinations appear in feature selection and bootstrap sampling.

    ```python
    from math import comb, perm

    print(perm(5, 3))  # 60 — permutations
    print(comb(5, 3))  # 10 — combinations
    ```

---

### Q7: What is the birthday problem and what does it illustrate?

??? "Show answer"
    The **birthday problem** asks: how many people do you need in a room before the probability that at least two share a birthday exceeds 50%? The answer is **23** — far fewer than most people intuit.

    With 23 people, there are C(23, 2) = 253 possible pairs. Each pair has a 1/365 chance of sharing a birthday. The cumulative probability over many pairs becomes substantial quickly.

    Precise calculation via the complement:
    P(no shared birthday with n people) = (365/365) × (364/365) × (363/365) × … × ((365−n+1)/365)
    P(at least one shared birthday) = 1 − P(none)

    At n = 23: P ≈ 0.507

    What it illustrates:
    1. **Human intuition underestimates collision probabilities** — we think linearly when the number of pairings grows quadratically.
    2. **Hash collision rates** in computer science follow the same logic — relevant for data deduplication, hash tables, and cryptographic analysis.
    3. **Multiple comparisons**: with many tests/features, spurious correlations become likely by chance alone.

    ```python
    import numpy as np

    def birthday_prob(n):
        p_none = np.prod([(365 - i) / 365 for i in range(n)])
        return 1 - p_none

    for n in [10, 23, 30, 50]:
        print(f"n={n}: P(shared birthday) = {birthday_prob(n):.3f}")
    ```

---

### Q8: What is the Monty Hall problem?

??? "Show answer"
    **Setup**: there are 3 doors. Behind one is a car; behind the other two are goats. You pick a door. The host (who knows what's behind each door) opens a different door, always revealing a goat. You're offered the chance to switch your pick. Should you?

    **Answer**: yes, always switch. Switching wins 2/3 of the time; staying wins 1/3.

    **Why**: when you first pick, you have a 1/3 chance of being right. That probability is fixed to your original door. The remaining 2/3 probability is distributed across the other two doors. When the host reveals one goat, the full 2/3 migrates to the remaining unopened door.

    Alternatively: P(car behind original door) = 1/3 always. P(car behind the other remaining door) = 2/3.

    What makes it a classic interview trap:
    - Almost everyone initially says it doesn't matter (50/50), which is wrong.
    - It illustrates that **the host's action is informative** — it is not a random reveal. The host always reveals a goat, which changes the information landscape.
    - Key concept: **conditional probability** and the importance of how information is revealed.

    ```python
    import random

    def monty_hall_sim(switch, trials=100_000):
        wins = 0
        for _ in range(trials):
            car = random.randint(0, 2)
            pick = random.randint(0, 2)
            # Host reveals a goat (not picked, not car)
            goat_doors = [d for d in range(3) if d != pick and d != car]
            revealed = random.choice(goat_doors)
            if switch:
                pick = [d for d in range(3) if d != pick and d != revealed][0]
            wins += (pick == car)
        return wins / trials

    print(f"Stay:   {monty_hall_sim(False):.3f}")   # ~0.333
    print(f"Switch: {monty_hall_sim(True):.3f}")    # ~0.667
    ```

---

### Q9: What is the difference between frequentist and Bayesian probability?

??? "Show answer"
    **Frequentist probability** defines probability as the long-run frequency of an event in infinitely repeated identical experiments. Parameters are fixed (but unknown) constants; data is random. You cannot make probabilistic statements about parameters — only about procedures (e.g., confidence intervals).

    **Bayesian probability** defines probability as a degree of belief, which can be updated as new evidence arrives. Parameters are treated as random variables with distributions. You can make direct probability statements like "there is a 90% probability the true conversion rate is between 5% and 7%."

    Practical differences:
    - Frequentist methods require no prior; Bayesian methods require specifying a prior distribution.
    - Frequentist confidence intervals are statements about the procedure, not the parameter. Bayesian credible intervals are direct probability statements about the parameter.
    - Bayesian methods naturally incorporate prior knowledge and update sequentially with new data.
    - Bayesian inference is computationally heavier (often requires MCMC).

    In practice: many data scientists use frequentist tests for A/B testing speed and simplicity, and Bayesian methods when they need to incorporate domain knowledge, handle small samples, or make direct probabilistic decisions.

---

### Q10: What is a random variable?

??? "Show answer"
    A **random variable** is a function that maps outcomes of a random experiment to numerical values. It provides a mathematical handle on randomness.

    Example: flip a coin twice. The sample space is {HH, HT, TH, TT}. Define X = number of heads. Then X(HH) = 2, X(HT) = 1, X(TH) = 1, X(TT) = 0. X is a random variable.

    Two types:
    - **Discrete**: takes countable values. Examples: number of defects, number of clicks, outcome of a die roll.
    - **Continuous**: takes any value in an interval. Examples: temperature, height, time until failure.

    Key properties of a random variable:
    - It has a **probability distribution** that assigns probabilities to its possible values.
    - It has an **expected value** (mean) and a **variance**.
    - Two random variables can be **independent** or **correlated**.

    In machine learning: model outputs, loss values, and data features are all treated as realisations of random variables. Understanding distributions of these variables is essential for inference.

---

### Q11: What is the difference between discrete and continuous probability distributions?

??? "Show answer"
    **Discrete distributions** assign probability to a countable set of outcomes. The probability function is called the **probability mass function (PMF)**: P(X = x) for each specific value x. All PMF values sum to 1. Examples: Binomial, Poisson, Bernoulli, Geometric.

    **Continuous distributions** assign probability to intervals — the probability of any single exact value is zero. The function describing them is the **probability density function (PDF)**: f(x) where probabilities are computed as areas: P(a ≤ X ≤ b) = ∫ₐᵇ f(x) dx. The total area under the PDF integrates to 1. Examples: Normal, Exponential, Uniform (continuous), Beta.

    Key consequences:
    - For continuous distributions, P(X = 5) = 0; you must ask P(4.9 ≤ X ≤ 5.1).
    - Computing probabilities from discrete distributions: sum the PMF. From continuous: integrate the PDF.
    - Both can be described by their CDF: F(x) = P(X ≤ x).

    Interview tip: be able to name two or three distributions of each type and explain when you'd use them.

---

### Q12: What is the law of large numbers?

??? "Show answer"
    The **Law of Large Numbers (LLN)** states that as sample size n increases, the sample mean x̄ converges to the population mean μ.

    There are two forms:
    - **Weak LLN**: the sample mean converges in probability (x̄ → μ for large n)
    - **Strong LLN**: the sample mean converges almost surely

    What it means in practice: you need a sufficient sample size for empirical averages to be reliable estimates of true expectations. With small samples, you can observe wild deviations from the expected value by chance.

    Common confusions:
    - **Gambler's fallacy**: the LLN does not say that after 10 coin flip heads, tails must become more likely. Each flip is independent. The LLN guarantees convergence over the long run, not "compensation" in the short run.
    - The LLN requires finite mean. For distributions with infinite mean (like some Pareto distributions), averages do not stabilise.

    Applications: Monte Carlo simulation (more samples → more accurate estimates), bootstrapping, online learning update rules.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt

    running_means = np.cumsum(np.random.normal(loc=5, size=1000)) / np.arange(1, 1001)
    plt.plot(running_means)
    plt.axhline(5, color='red', linestyle='--', label='True mean')
    plt.xlabel('Sample size')
    plt.ylabel('Running mean')
    plt.legend()
    plt.show()
    ```

---

### Q13: What is a joint probability distribution?

??? "Show answer"
    A **joint probability distribution** describes the probability of two (or more) random variables taking specific values simultaneously.

    For discrete variables X and Y: P(X = x, Y = y) for all combinations (x, y).
    For continuous variables: joint PDF f(x, y) such that P(a ≤ X ≤ b, c ≤ Y ≤ d) = ∫∫ f(x,y) dx dy.

    From the joint distribution you can derive:
    - **Marginal distributions**: P(X = x) = Σᵧ P(X = x, Y = y) — sum (or integrate) over the other variable.
    - **Conditional distributions**: P(X = x | Y = y) = P(X = x, Y = y) / P(Y = y)
    - **Independence check**: X and Y are independent iff P(X = x, Y = y) = P(X = x) × P(Y = y) for all x, y.

    In machine learning: the joint distribution P(features, label) is the fundamental object of supervised learning. A generative model learns this joint; a discriminative model learns P(label | features) directly.

---

### Q14: What is a marginal probability?

??? "Show answer"
    A **marginal probability** is the probability of one variable without reference to the other variables — obtained by integrating or summing out the other variables from a joint distribution.

    For discrete variables: P(X = x) = Σᵧ P(X = x, Y = y) — this is called "marginalising over Y."
    For continuous variables: f_X(x) = ∫ f(x, y) dy

    The name "marginal" comes from the historical practice of writing the totals in the margins of a contingency table.

    **Example** with a contingency table:

    ```
                   Churn=0   Churn=1   Total (marginal)
    Premium=0        300       100          400
    Premium=1        450        50          500
    Total (marginal) 750       150          900
    ```

    P(Churn=1) = 150/900 ≈ 0.167 — the marginal probability of churn, ignoring premium status.
    P(Churn=1 | Premium=0) = 100/400 = 0.25 — the conditional probability.

    Marginal probabilities summarise the distribution of one variable, collapsing all others. They're foundational in any analysis involving multiple variables.

---

### Q15: What is the difference between probability and likelihood?

??? "Show answer"
    This is a subtle but important distinction, especially for understanding maximum likelihood estimation (MLE).

    **Probability** is a function of the *data* given fixed parameters. P(data | θ). You know the model parameters and ask how probable a particular outcome is.

    **Likelihood** is a function of the *parameters* given fixed observed data. L(θ | data). You have observed data and ask how probable (or plausible) different parameter values are.

    Crucially, the likelihood is *not* a probability distribution over parameters — it does not integrate to 1 over θ. It's a score you use to compare different parameter values.

    **Example**: you flip a coin 10 times and observe 7 heads.
    - Probability: given p = 0.7, P(7 heads in 10 flips) = C(10,7) × 0.7⁷ × 0.3³ ≈ 0.267
    - Likelihood: L(p | 7 heads) treats the 7-heads result as fixed and evaluates how plausible each value of p is. L(p=0.7 | data) > L(p=0.5 | data) — so p=0.7 is a more likely parameter value.

    MLE finds the parameter θ that maximises L(θ | data). Bayesian inference treats θ as having a prior and updates to a posterior using the likelihood.

    ```python
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.stats import binom

    p_values = np.linspace(0, 1, 200)
    # Likelihood of observing 7 heads in 10 flips as a function of p
    likelihoods = binom.pmf(k=7, n=10, p=p_values)
    plt.plot(p_values, likelihoods)
    plt.xlabel('p (parameter)')
    plt.ylabel('Likelihood L(p | data)')
    plt.title('Likelihood function — peaks at p=0.7 (MLE)')
    plt.show()
    ```
