# Regression

Regression is the backbone of predictive modelling. These questions come up in virtually every data science interview — expect them in phone screens, take-homes, and final rounds.

---

### Q1: What is linear regression and what are its assumptions?

??? "Show answer"
    Linear regression models the relationship between a continuous target variable and one or more input features by fitting a straight line (or hyperplane) that minimises the sum of squared residuals.

    The model form is: `y = β₀ + β₁x₁ + β₂x₂ + ... + βₙxₙ + ε`

    **The five classical assumptions (LINE + H):**

    1. **Linearity** — the relationship between predictors and the target is linear.
    2. **Independence** — observations are independent of each other (no autocorrelation in residuals).
    3. **Normality** — residuals are approximately normally distributed (matters most for inference, not prediction).
    4. **Equal variance (homoscedasticity)** — the variance of residuals is constant across all levels of the predictors.
    5. **No multicollinearity** — predictors are not highly correlated with each other.

    In practice, linear regression is often surprisingly robust to mild violations of these assumptions, especially when used for prediction rather than inference. The assumptions matter most when you care about the validity of confidence intervals and p-values.

---

### Q2: What happens when linear regression assumptions are violated?

??? "Show answer"
    Each violation has a different consequence:

    - **Non-linearity**: predictions will be systematically biased. The model will underfit certain regions of the input space. Fix: add polynomial terms, interaction features, or switch to a non-linear model.

    - **Autocorrelation in residuals**: standard errors are underestimated, making hypothesis tests unreliable. Common in time series. Fix: include lagged features, use time-series-specific models (ARIMA), or use Newey-West standard errors.

    - **Non-normal residuals**: minor for prediction, problematic for inference. With large samples, the Central Limit Theorem softens this. Fix: transform the target (log, Box-Cox) or switch to quantile regression.

    - **Heteroscedasticity**: OLS estimates remain unbiased but are no longer efficient — standard errors are wrong. Fix: use weighted least squares, transform the target, or use robust standard errors.

    - **Multicollinearity**: coefficient estimates become unstable and highly sensitive to small changes in data. Individual coefficients lose interpretability, though predictions remain fine. Fix: remove one of the correlated features, use PCA, or apply Ridge regression.

    The most dangerous violations for production models are non-linearity (causes systematic errors) and heteroscedasticity (breaks uncertainty estimates).

---

### Q3: What is the difference between simple and multiple linear regression?

??? "Show answer"
    **Simple linear regression** uses a single predictor: `y = β₀ + β₁x + ε`

    **Multiple linear regression** uses two or more predictors: `y = β₀ + β₁x₁ + ... + βₙxₙ + ε`

    The interpretation of coefficients changes in multiple regression. Each coefficient represents the expected change in `y` for a one-unit increase in that predictor, **holding all other predictors constant**. This partial effect interpretation is the key distinction.

    Multiple regression also introduces the risk of multicollinearity and requires more data to estimate coefficients reliably. As a rule of thumb, aim for at least 10–20 observations per predictor.

---

### Q4: How do you interpret regression coefficients?

??? "Show answer"
    A coefficient `βᵢ` means: for a one-unit increase in `xᵢ`, holding all other features constant, the predicted `y` changes by `βᵢ` units.

    Interpretation depends on the scale and transformations applied:

    - **Raw features**: straightforward — e.g., β = 2.5 for "years of experience" means each additional year adds 2.5 units to the predicted salary.
    - **Log-transformed target**: `β` is approximately the percent change in `y` for a one-unit increase in `x`. (Multiply by 100 to get the percentage.)
    - **Log-transformed predictor**: `β` is the change in `y` for a 1% increase in `x` (divide by 100).
    - **Log-log**: `β` is an elasticity — a 1% change in `x` → `β`% change in `y`.
    - **Standardised features**: coefficients become comparable in magnitude, telling you which features have the largest effect relative to their variance.

    Coefficients in multiple regression are **partial effects**, not total effects. If two predictors are correlated, individual coefficients can be misleading or even flip sign (Simpson's paradox territory). Always check VIF before trusting coefficient magnitudes.

---

### Q5: What is R² and what are its limitations?

??? "Show answer"
    R² (coefficient of determination) measures the proportion of variance in the target that is explained by the model.

    ```
    R² = 1 - (SS_res / SS_tot)
       = 1 - (Σ(yᵢ - ŷᵢ)²) / (Σ(yᵢ - ȳ)²)
    ```

    R² = 1 means perfect fit; R² = 0 means the model does no better than predicting the mean.

    **Limitations:**

    - **Always increases with more predictors**, even if those predictors are noise. Adding irrelevant variables never hurts R² — it either stays flat or goes up. This makes it useless for comparing models with different numbers of features.
    - **Doesn't indicate whether the model is appropriate**. A high R² can coexist with a misspecified model (e.g., fitting a line to a curve).
    - **Sensitive to outliers**. A few high-leverage points can inflate R² dramatically.
    - **Not meaningful for non-linear models** in the same way — comparing R² across fundamentally different model types is unreliable.
    - **Doesn't tell you about prediction accuracy in absolute terms**. R² = 0.9 could mean errors of ±1 unit or ±10,000 units depending on the scale of `y`.

    In production, RMSE or MAE on a held-out set is usually more actionable than R².

---

### Q6: What is adjusted R² and when do you use it?

??? "Show answer"
    Adjusted R² penalises the addition of predictors that don't improve the model's explanatory power.

    ```
    Adjusted R² = 1 - [(1 - R²)(n - 1) / (n - p - 1)]
    ```

    Where `n` is the number of observations and `p` is the number of predictors.

    Unlike R², adjusted R² can **decrease** when you add a predictor that doesn't help. This makes it useful for comparing models with different numbers of features during feature selection.

    Use it when:
    - You're doing model selection among linear regression models with different predictor sets.
    - You want a quick sanity check that adding a feature actually improved the model.

    Don't rely on it exclusively — AIC, BIC, or cross-validated RMSE are generally more reliable for model comparison because they account for the train/test generalisation gap.

---

### Q7: What is the difference between Ridge and Lasso regression?

??? "Show answer"
    Both are regularised versions of linear regression that add a penalty term to the loss function to shrink coefficients and reduce overfitting. The difference is in the penalty:

    **Ridge (L2 regularisation):**
    ```
    Loss = Σ(yᵢ - ŷᵢ)² + λΣβⱼ²
    ```
    Ridge shrinks all coefficients toward zero but never sets them exactly to zero. Every feature stays in the model.

    **Lasso (L1 regularisation):**
    ```
    Loss = Σ(yᵢ - ŷᵢ)² + λΣ|βⱼ|
    ```
    Lasso can shrink coefficients all the way to exactly zero, performing automatic feature selection. The result is a sparse model.

    **Geometric intuition:** The L2 penalty is a circle; the L1 penalty is a diamond. The optimal point where the loss ellipse touches the constraint region is likely to land on a corner of the diamond (where some βⱼ = 0) but unlikely to land on the smooth surface of the circle.

    ```python
    from sklearn.linear_model import Ridge, Lasso, ElasticNet

    ridge = Ridge(alpha=1.0)          # alpha = λ
    lasso = Lasso(alpha=0.1)
    ```

---

### Q8: When would you choose Lasso over Ridge?

??? "Show answer"
    Choose Lasso when:

    - **You believe only a subset of features are truly relevant** and you want the model to identify them automatically. Lasso will zero out irrelevant coefficients.
    - **Interpretability matters** — a sparse model with 10 features is easier to explain than one with 200 features all having small non-zero weights.
    - **You have many more features than observations** (p >> n) and need implicit feature selection.
    - **You want the model itself to act as a feature selector** before feeding into another model.

    Choose Ridge when:
    - **You believe most features contribute something** — e.g., gene expression data where thousands of genes each have a small effect.
    - **Your predictors are highly correlated** — Lasso tends to arbitrarily pick one from a correlated group and zero out the rest. Ridge distributes the effect across all correlated features, which is usually more stable.
    - **Stability of coefficients matters** more than sparsity.

    In practice, run both and compare cross-validated error. ElasticNet is a good default when you're unsure.

---

### Q9: What is ElasticNet?

??? "Show answer"
    ElasticNet combines both L1 and L2 penalties:

    ```
    Loss = Σ(yᵢ - ŷᵢ)² + λ₁Σ|βⱼ| + λ₂Σβⱼ²
    ```

    Or equivalently, parameterised by a mixing ratio `α` (not to be confused with sklearn's `alpha` for overall strength):

    ```
    Loss = Σ(yᵢ - ŷᵢ)² + λ[α·Σ|βⱼ| + (1-α)·Σβⱼ²]
    ```

    ```python
    from sklearn.linear_model import ElasticNet
    # l1_ratio=1 → Lasso, l1_ratio=0 → Ridge
    en = ElasticNet(alpha=0.1, l1_ratio=0.5)
    ```

    **Why use it:**
    - Gets the sparsity of Lasso (some coefficients go to zero) while also handling correlated features better than pure Lasso (Ridge part stabilises the solution).
    - When there are groups of correlated predictors, Lasso tends to pick one arbitrarily. ElasticNet tends to keep or drop the group together.

    ElasticNet is a solid default for high-dimensional regression problems when you're uncertain whether sparsity or stability is more important.

---

### Q10: What are the normal equations in linear regression?

??? "Show answer"
    The normal equations give a closed-form solution for the OLS (ordinary least squares) regression coefficients:

    ```
    β = (XᵀX)⁻¹Xᵀy
    ```

    Where `X` is the design matrix (n × p), `y` is the target vector (n × 1), and `β` is the coefficient vector (p × 1).

    **Derivation intuition:** We minimise `||y - Xβ||²`. Taking the derivative with respect to β and setting it to zero gives `XᵀXβ = Xᵀy`, which we solve directly.

    **Why not always use it:**
    - Computing `(XᵀX)⁻¹` is O(p³) — expensive when p is large (thousands of features).
    - If `XᵀX` is singular or near-singular (multicollinearity), the inverse doesn't exist or is numerically unstable.
    - For large datasets, gradient descent is far more computationally tractable.

    Ridge regression modifies the normal equations to: `β = (XᵀX + λI)⁻¹Xᵀy`, which also guarantees invertibility even when `XᵀX` is singular.

---

### Q11: What is heteroscedasticity and why does it matter?

??? "Show answer"
    Heteroscedasticity means the variance of the residuals is not constant across the range of fitted values or predictors. The opposite — constant variance — is homoscedasticity.

    **Example:** Predicting house prices — residuals might be small for cheap houses and large for expensive ones. The model is more uncertain at higher price points.

    **Why it matters:**
    - OLS coefficient estimates remain **unbiased** but are no longer **efficient** (i.e., not the minimum variance estimator).
    - Standard errors are incorrect, making p-values and confidence intervals unreliable. You can't trust inference.
    - In forecasting, it means your prediction intervals are wrong — too narrow in some regions, too wide in others.

    **Detection:**
    - Plot residuals vs. fitted values — look for a fan or cone shape.
    - Breusch-Pagan test or White test for formal statistical testing.

    ```python
    import matplotlib.pyplot as plt
    residuals = y_true - y_pred
    plt.scatter(y_pred, residuals)
    plt.axhline(0, color='red')
    plt.xlabel('Fitted values')
    plt.ylabel('Residuals')
    ```

    **Fixes:**
    - Log-transform the target variable (common and effective).
    - Weighted least squares (WLS) — downweight high-variance observations.
    - Use robust standard errors (heteroscedasticity-consistent standard errors).

---

### Q12: How do you detect outliers in regression?

??? "Show answer"
    There are three related but distinct concepts:

    - **Outlier**: a point with an unusually large residual (far from the regression line in the y-direction).
    - **High-leverage point**: a point that is extreme in the feature space (unusual x values). It has the *potential* to influence the regression line heavily.
    - **Influential point**: a point that actually *changes* the regression coefficients significantly when removed. An influential point is usually both an outlier and high leverage.

    **Detection methods:**

    - **Standardised/studentised residuals**: flag observations where `|eᵢ/se| > 2` or `> 3`.
    - **Cook's distance**: measures how much all fitted values change when observation i is deleted. `D > 4/n` is a common threshold for concern.
    - **Leverage (hat matrix diagonal)**: `hᵢᵢ > 2p/n` indicates high leverage.

    ```python
    import numpy as np
    from sklearn.linear_model import LinearRegression

    model = LinearRegression().fit(X_train, y_train)
    residuals = y_train - model.predict(X_train)
    std_resid = residuals / residuals.std()
    outlier_mask = np.abs(std_resid) > 3
    ```

    For a principled approach, use `statsmodels` which computes influence measures directly:

    ```python
    import statsmodels.api as sm
    ols_model = sm.OLS(y, sm.add_constant(X)).fit()
    influence = ols_model.get_influence()
    cooks_d = influence.cooks_distance[0]
    ```

---

### Q13: What is polynomial regression and when would you use it?

??? "Show answer"
    Polynomial regression extends linear regression by adding polynomial terms of the predictors (x², x³, etc.) as new features. Despite the curved fit, it's still a linear model — linear in the coefficients.

    ```python
    from sklearn.preprocessing import PolynomialFeatures
    from sklearn.pipeline import Pipeline
    from sklearn.linear_model import LinearRegression

    model = Pipeline([
        ('poly', PolynomialFeatures(degree=3, include_bias=False)),
        ('lr', LinearRegression())
    ])
    model.fit(X_train, y_train)
    ```

    **When to use it:**
    - The relationship between `x` and `y` is clearly non-linear but smooth (e.g., a growth curve, a U-shape).
    - You have enough data to support higher-degree terms without overfitting.
    - You need interpretability — polynomial terms remain explainable.

    **Risks:**
    - High-degree polynomials overfit aggressively, especially near the boundaries of the training data.
    - Extrapolation is dangerous — polynomial functions swing wildly outside the training range.
    - Feature space grows fast with many predictors: degree-2 polynomial with p features creates O(p²) terms.

    In practice, degree 2 or 3 covers most real-world non-linearities. If you need more flexibility, consider splines or tree-based models.

---

### Q14: What is the difference between regression and correlation?

??? "Show answer"
    **Correlation** measures the strength and direction of a linear relationship between two variables. It's symmetric and unitless (bounded between -1 and 1).

    ```
    r = Cov(X, Y) / (σ_X · σ_Y)
    ```

    Correlation answers: "Are X and Y related, and how strongly?"

    **Regression** models the relationship in order to predict one variable from another. It's asymmetric — predicting Y from X is a different model than predicting X from Y.

    Regression answers: "How does Y change when X changes, and what is Y's value for a given X?"

    **Key differences:**
    - Correlation is symmetric: `r(X,Y) = r(Y,X)`. Regression is not: the slope of Y~X ≠ 1/(slope of X~Y).
    - Correlation is unitless; regression coefficients have units (change in Y per unit of X).
    - Correlation says nothing about causation or direction of effect; regression models a directional relationship (but doesn't prove causation either).
    - `r² = R²` in simple linear regression — the squared correlation equals the coefficient of determination.

    A common interview trap: two variables can have high correlation but a regression slope near zero (if one variable has much larger variance than the other).

---

### Q15: What metrics do you use to evaluate regression models?

??? "Show answer"
    **Mean Squared Error (MSE):**
    ```
    MSE = (1/n) Σ(yᵢ - ŷᵢ)²
    ```
    Penalises large errors heavily due to squaring. Differentiable everywhere, so useful as a loss function during training. Not in the same units as y.

    **Root Mean Squared Error (RMSE):**
    ```
    RMSE = √MSE
    ```
    Same units as y, making it interpretable. Still penalises large errors more than small ones. The most commonly reported metric in regression.

    **Mean Absolute Error (MAE):**
    ```
    MAE = (1/n) Σ|yᵢ - ŷᵢ|
    ```
    Robust to outliers — a single massive error doesn't dominate. Easier to interpret ("average error is X units"). Non-differentiable at zero, which complicates gradient-based optimisation.

    **Mean Absolute Percentage Error (MAPE):**
    ```
    MAPE = (100/n) Σ|( yᵢ - ŷᵢ) / yᵢ|
    ```
    Scale-independent — useful for comparing across datasets with different y scales. Fails when `yᵢ = 0` and is asymmetric (penalises under-prediction more than over-prediction).

    **R²:** proportion of variance explained — useful for relative model comparison, not absolute error assessment.

    ```python
    from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
    import numpy as np

    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    r2 = r2_score(y_true, y_pred)
    ```

    **Choosing between them:** use RMSE when large errors are especially costly. Use MAE when outliers shouldn't dominate evaluation. Use MAPE when relative error matters more than absolute error.

---

### Q16: What is quantile regression?

??? "Show answer"
    Quantile regression estimates conditional quantiles of the target distribution rather than the conditional mean. The most common case is median regression (50th percentile), but you can fit any quantile τ ∈ (0, 1).

    **Why it matters:**
    - OLS regression estimates `E[Y|X]` — the mean. If the distribution of Y is skewed or has heavy tails, the mean is misleading.
    - Quantile regression estimates `Q_τ[Y|X]` — the τ-th quantile. For τ = 0.5 you get the median; for τ = 0.9 you get the 90th percentile conditional on X.

    **Loss function (pinball/quantile loss):**
    ```
    L_τ(r) = r · τ       if r ≥ 0
           = r · (τ - 1)  if r < 0
    ```
    This asymmetrically penalises over- and under-prediction, biasing the fit toward the desired quantile.

    ```python
    from sklearn.linear_model import QuantileRegressor

    qr_median = QuantileRegressor(quantile=0.5).fit(X_train, y_train)
    qr_90 = QuantileRegressor(quantile=0.9).fit(X_train, y_train)

    # Prediction interval: 10th to 90th percentile
    lower = QuantileRegressor(quantile=0.1).fit(X_train, y_train).predict(X_test)
    upper = qr_90.predict(X_test)
    ```

    **Use cases:**
    - Predicting delivery times (you care about the 95th percentile, not the mean).
    - Financial risk (Value at Risk is literally a quantile).
    - Any setting where the distribution of outcomes is skewed or where different quantiles have different business implications.
    - Building prediction intervals without distributional assumptions.

---
