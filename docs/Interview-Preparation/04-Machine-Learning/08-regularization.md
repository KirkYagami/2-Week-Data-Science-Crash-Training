# Regularisation

Regularisation is one of the most consistently tested topics in data science interviews — it sits at the intersection of theory (bias-variance tradeoff, Bayesian priors) and practice (how to tune it, when to use which form). Know the geometry, not just the formula.

---

### Q1: What is regularisation and why is it needed?

??? "Show answer"
    Regularisation is any technique that constrains or penalises model complexity to reduce overfitting and improve generalisation to unseen data.

    **Why it's needed:** machine learning models minimise training loss. Without regularisation, a sufficiently complex model will memorise the training data — including noise — producing low training error but high test error (overfitting). Regularisation adds a term to the objective that penalises complexity, forcing the model to learn the underlying signal rather than the noise.

    **The formal setup for regularised ERM (Empirical Risk Minimisation):**
    ```
    Minimise: (1/n) Σ L(yᵢ, f(xᵢ)) + λ · Ω(f)
    ```

    Where `L` is the task loss, `Ω(f)` is the complexity penalty, and `λ > 0` controls the tradeoff.

    **Forms of regularisation:**
    - **Explicit penalties on parameters**: L1, L2 norms on weights (Lasso, Ridge).
    - **Architectural constraints**: dropout, max-norm, weight decay in neural networks.
    - **Training process**: early stopping, data augmentation.
    - **Implicit regularisation**: stochastic gradient descent with small batch sizes has been shown to have an implicit regularisation effect.

    Regularisation is essentially the machine learning embodiment of Occam's Razor: prefer the simpler explanation that fits the data.

---

### Q2: What is L1 regularisation (Lasso)? What does it do to weights?

??? "Show answer"
    L1 regularisation adds the sum of absolute values of the model weights to the loss function:

    ```
    Loss = Data_loss + λ · Σ|wⱼ|
    ```

    **Effect on weights:** L1 regularisation pushes weights toward zero, and crucially, **sets some weights exactly to zero**. The result is a **sparse model** — most weights are zero, and only a small subset of features are retained.

    **Why does L1 produce sparsity?** The L1 penalty has a constant gradient with respect to each weight: `∂(λ|wⱼ|)/∂wⱼ = λ · sign(wⱼ)`. This constant "pull" toward zero doesn't slow down as the weight approaches zero — it pushes all the way. Once a weight crosses zero, the penalty reverses sign and the gradient pushes it back. The stable point at zero is a singularity (the derivative is undefined there), and weights often settle exactly at zero during optimisation.

    **Geometric intuition:** the L1 constraint region is a diamond (hypercube in higher dimensions) with sharp corners at the axes. The optimal solution is likely to be at a corner, which corresponds to one or more weights being exactly zero.

    ```python
    from sklearn.linear_model import Lasso, LassoCV

    # LassoCV automatically selects the best alpha via cross-validation
    lasso_cv = LassoCV(cv=5, alphas=None, max_iter=10000)
    lasso_cv.fit(X_scaled, y)

    print(f"Best alpha: {lasso_cv.alpha_:.4f}")
    print(f"Non-zero coefficients: {(lasso_cv.coef_ != 0).sum()}")
    ```

---

### Q3: What is L2 regularisation (Ridge)? What does it do to weights?

??? "Show answer"
    L2 regularisation adds the sum of squared weights to the loss function:

    ```
    Loss = Data_loss + λ · Σwⱼ²
    ```

    **Effect on weights:** L2 regularisation shrinks all weights toward zero but **never sets them exactly to zero**. It produces small, diffuse weights — many features are retained but with reduced magnitude.

    **Why doesn't L2 produce zeros?** The gradient of the L2 penalty is `2λwⱼ`. As a weight approaches zero, the gradient also approaches zero — the pull toward zero weakens. It's a "soft" constraint that doesn't have enough force to push weights all the way to zero.

    **Geometric intuition:** the L2 constraint region is a circle (hypersphere in higher dimensions) with no corners or edges. The optimal point where the loss ellipse touches the circle is almost never on an axis (where a weight is zero) — it's somewhere on the smooth boundary.

    **Closed-form solution for Ridge:**
    ```
    β_ridge = (XᵀX + λI)⁻¹Xᵀy
    ```
    The `λI` term makes `XᵀX + λI` always invertible, even when `XᵀX` is singular. This is why Ridge works well with multicollinear features.

    ```python
    from sklearn.linear_model import Ridge, RidgeCV

    ridge_cv = RidgeCV(alphas=[0.01, 0.1, 1.0, 10.0, 100.0], cv=5)
    ridge_cv.fit(X_scaled, y)
    print(f"Best alpha: {ridge_cv.alpha_}")
    print(f"Coefficient magnitudes: {np.abs(ridge_cv.coef_).mean():.4f}")
    ```

---

### Q4: Why does L1 produce sparse models but L2 does not?

??? "Show answer"
    The answer comes from both geometry and calculus.

    **Geometric explanation:**
    Optimising a regularised loss is equivalent to minimising the data loss subject to a constraint on the parameter norm:
    - L1: `Σ|wⱼ| ≤ t` — this is a diamond/rhombus shape with sharp corners on the coordinate axes.
    - L2: `Σwⱼ² ≤ t` — this is a circle/sphere with a smooth surface.

    As you shrink the loss ellipse toward the feasible region, it will first touch:
    - The **L1 diamond** at a corner — corners are on the coordinate axes, where one or more weights are exactly 0.
    - The **L2 sphere** somewhere on its smooth surface — there are no corners, so the intersection is almost never on an axis.

    **Gradient/subgradient explanation:**
    - L2 gradient: `2λw` — shrinks proportionally to the weight magnitude. As `w → 0`, the gradient → 0. The shrinkage slows and stops just above zero.
    - L1 subgradient: `λ · sign(w)` — constant magnitude regardless of `w`. The pull toward zero is always the same strength, allowing it to "drag" the weight all the way to zero.

    **In proximal gradient descent (soft-thresholding for L1):**
    ```python
    # L1 update: soft threshold
    w_new = np.sign(w) * np.maximum(np.abs(w) - lambda_lr, 0)
    # This explicitly zeros out weights with |w| < lambda * learning_rate
    ```
    The soft-thresholding operator directly zeros out small weights — there's no equivalent for L2.

---

### Q5: What is ElasticNet?

??? "Show answer"
    ElasticNet combines L1 and L2 penalties in a weighted sum:

    ```
    Loss = Data_loss + λ[α · Σ|wⱼ| + (1-α) · Σwⱼ²]
    ```

    Where `α` (l1_ratio in sklearn) ∈ [0, 1] interpolates between pure Ridge (α=0) and pure Lasso (α=1). `λ` controls the overall regularisation strength.

    ```python
    from sklearn.linear_model import ElasticNet, ElasticNetCV

    # Tune both alpha (overall strength) and l1_ratio (L1 vs L2 mix)
    en_cv = ElasticNetCV(
        l1_ratio=[0.1, 0.5, 0.7, 0.9, 0.95, 1.0],
        alphas=None,  # auto-selected
        cv=5,
        max_iter=10000
    )
    en_cv.fit(X_scaled, y)
    print(f"Best alpha: {en_cv.alpha_:.4f}, Best l1_ratio: {en_cv.l1_ratio_}")
    ```

    **Why ElasticNet beats pure Lasso:**
    - **Correlated features**: when features are highly correlated, Lasso arbitrarily picks one and zeros out the rest. The L2 component of ElasticNet introduces a "grouping effect" — correlated features tend to be included or excluded together, with their weights distributed across the group.
    - **p >> n setting**: Lasso can select at most n features (when there are more features than observations). ElasticNet's L2 component allows it to select more features.
    - **Stability**: the L2 component makes the solution more stable across small perturbations of the training data.

    ElasticNet is a strong default for high-dimensional regression when you're uncertain whether sparsity or stability is more important.

---

### Q6: How do you choose the regularisation strength λ?

??? "Show answer"
    λ (alpha in sklearn) is the most important hyperparameter in regularised models. It controls the fundamental bias-variance tradeoff:
    - λ too small → insufficient regularisation → overfitting.
    - λ too large → over-regularisation → underfitting (all weights shrink to near zero).

    **Cross-validated search:**
    The most reliable approach is to search over a logarithmic range of λ values and choose the one with the best cross-validated loss.

    ```python
    from sklearn.linear_model import RidgeCV, LassoCV, ElasticNetCV
    import numpy as np

    # Logarithmic grid: 0.001 to 1000
    alphas = np.logspace(-3, 3, 100)

    ridge_cv = RidgeCV(alphas=alphas, cv=5).fit(X_scaled, y)
    lasso_cv = LassoCV(cv=5, max_iter=10000).fit(X_scaled, y)

    print(f"Ridge best alpha: {ridge_cv.alpha_}")
    print(f"Lasso best alpha: {lasso_cv.alpha_}")
    ```

    **Bayesian interpretation:** λ encodes a prior belief about the distribution of weights.
    - L2 with λ corresponds to a Gaussian prior on weights with variance σ² = 1/(2λ). Large λ → tight prior → weights close to zero.
    - L1 with λ corresponds to a Laplace prior on weights.
    Finding λ via cross-validation is equivalent to maximum likelihood estimation of the prior's variance given the data.

    **Practical guidance:**
    - Always search on a log scale (orders of magnitude, not linear steps).
    - Use the 1-SE rule: choose the largest λ whose cross-validated error is within 1 standard error of the minimum. This gives a simpler, more regularised model with similar performance.
    - The optimal λ will be different after feature scaling — always scale before choosing λ.

---

### Q7: What is dropout regularisation in neural networks?

??? "Show answer"
    Dropout randomly deactivates a fraction `p` (the drop rate) of neurons during each forward pass of training. The deactivated neurons output zero for that pass, and their weights are not updated. At inference, all neurons are active but their outputs are scaled by `(1 - p)` to maintain the expected activation magnitude.

    ```python
    import torch.nn as nn

    model = nn.Sequential(
        nn.Linear(256, 128),
        nn.ReLU(),
        nn.Dropout(p=0.5),     # 50% of neurons dropped during training
        nn.Linear(128, 64),
        nn.ReLU(),
        nn.Dropout(p=0.3),
        nn.Linear(64, 1)
    )

    model.train()   # dropout active
    model.eval()    # dropout disabled
    ```

    **Why dropout works:**
    1. **Prevents co-adaptation**: neurons can't rely on specific other neurons always being present. Each neuron must learn a feature that is independently useful.
    2. **Implicit ensemble**: training with dropout is approximately training an exponential number of thinned networks (`2^n` subnetworks for n neurons) and averaging their predictions at inference — similar in spirit to bagging.
    3. **Noise injection**: adding random noise to the network activations acts as a regulariser, analogous to adding noise to training data.

    Typical drop rates: 0.5 for fully connected layers, 0.1–0.2 for convolutional layers (which have more spatial weight sharing). Apply only during training — use `model.train()` / `model.eval()` to toggle correctly.

---

### Q8: What is early stopping and why is it a form of regularisation?

??? "Show answer"
    Early stopping halts model training when the validation loss stops improving, preventing further fitting to training data noise.

    ```python
    import xgboost as xgb

    model = xgb.XGBClassifier(n_estimators=10000, learning_rate=0.01, early_stopping_rounds=50)
    model.fit(X_train, y_train, eval_set=[(X_val, y_val)], verbose=100)
    print(f"Stopped at iteration: {model.best_iteration}")
    ```

    **Why it's regularisation:**
    - Training loss continues to decrease with more iterations; validation loss eventually increases.
    - Stopping before convergence limits the complexity of the learned model — it hasn't had time to memorise the training set.
    - For gradient boosting, fewer trees = less complex ensemble. For neural networks, earlier stopping = weights haven't moved far from initialisation (which is typically near zero) — this is analogous to L2 regularisation on the parameter trajectory.

    **The formal connection (for SGD-trained neural networks):** under certain conditions, early stopping in gradient descent is equivalent to L2 (Ridge) regularisation. The number of training steps plays the role of 1/λ: more steps → less regularisation.

    Early stopping is one of the most effective and computationally free regularisation techniques — it costs nothing extra and often provides substantial gains over fixed epoch counts.

---

### Q9: What is data augmentation as a regularisation technique?

??? "Show answer"
    Data augmentation creates new training examples by applying label-preserving transformations to existing ones, effectively increasing the size and diversity of the training set without collecting new data.

    **Why it regularises:** the model must learn to produce the same prediction for all augmented versions of an input, which forces it to learn invariances rather than memorising specific pixel patterns, token positions, or exact feature values. This improves generalisation.

    **Image augmentation:**
    ```python
    from torchvision import transforms

    train_transforms = transforms.Compose([
        transforms.RandomHorizontalFlip(p=0.5),
        transforms.RandomRotation(degrees=15),
        transforms.ColorJitter(brightness=0.2, contrast=0.2),
        transforms.RandomResizedCrop(224, scale=(0.8, 1.0)),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
    ])
    ```

    **Text augmentation:**
    - Synonym replacement, back-translation, random deletion/insertion of words.
    - Mixup: interpolate between two training examples and their labels.

    **Tabular data augmentation:**
    - Gaussian noise injection to continuous features.
    - SMOTE for oversampling minority class.
    - Feature dropout during training (randomly zero out feature values).

    **Mixup (general technique):**
    ```python
    # Mixup: create training examples as convex combinations
    lam = np.random.beta(alpha, alpha)
    x_mix = lam * x_i + (1 - lam) * x_j
    y_mix = lam * y_i + (1 - lam) * y_j
    ```

    The strength of augmentation is itself a regularisation hyperparameter — too much augmentation introduces bias.

---

### Q10: What is batch normalisation and does it regularise?

??? "Show answer"
    Batch normalisation (BatchNorm) normalises the activations within each mini-batch to have zero mean and unit variance, then applies learnable scale and shift parameters:

    ```
    x̂ᵢ = (xᵢ - μ_batch) / √(σ²_batch + ε)
    yᵢ = γ · x̂ᵢ + β
    ```

    Where γ and β are learned parameters (restored after normalisation).

    ```python
    import torch.nn as nn

    model = nn.Sequential(
        nn.Linear(256, 128),
        nn.BatchNorm1d(128),   # normalise across the batch
        nn.ReLU(),
        nn.Linear(128, 64),
        nn.BatchNorm1d(64),
        nn.ReLU()
    )
    ```

    **Primary purpose:** BatchNorm stabilises and accelerates training by:
    - Reducing internal covariate shift (the change in layer input distributions as weights update).
    - Allowing higher learning rates.
    - Reducing sensitivity to weight initialisation.

    **Does it regularise?** Yes, secondarily. The noise introduced by using mini-batch statistics (instead of the full population mean/variance) acts as a regulariser — similar to adding noise to activations. This is why BatchNorm often reduces the need for dropout. However, it's not primarily a regularisation technique.

    **At inference:** BatchNorm uses running statistics (exponential moving averages of training batch statistics) rather than test-batch statistics. Always call `model.eval()` before inference — this switches BatchNorm to use running stats.

---

### Q11: What is weight decay?

??? "Show answer"
    Weight decay is L2 regularisation implemented directly in the parameter update step of gradient descent, rather than as an explicit penalty term added to the loss function.

    **Standard SGD update:**
    ```
    w ← w - η · ∇L
    ```

    **SGD with weight decay:**
    ```
    w ← w - η · ∇L - η · λ · w
    = w · (1 - η · λ) - η · ∇L
    ```

    The weight is multiplied by `(1 - η·λ)` at each step — it "decays" toward zero regardless of the gradient. This is equivalent to L2 regularisation for standard SGD.

    ```python
    import torch.optim as optim

    # weight_decay IS L2 regularisation for SGD
    optimizer = optim.SGD(model.parameters(), lr=0.01, weight_decay=1e-4)

    # For Adam: weight_decay in standard Adam = L2 on gradient (NOT true weight decay)
    # Use AdamW for true weight decay with Adam
    optimizer = optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-2)
    ```

    **The Adam / AdamW distinction matters:** standard Adam with L2 penalty does not produce true weight decay because the L2 gradient is scaled by Adam's adaptive learning rates. AdamW decouples the weight decay from the gradient update, applying weight decay directly to the weights. AdamW is generally preferred for transformers and modern deep learning.

---

### Q12: What is the relationship between regularisation and the bias-variance tradeoff?

??? "Show answer"
    Regularisation directly controls where on the bias-variance tradeoff curve your model sits.

    **No regularisation (λ = 0):** the model fits the training data as closely as possible. For a flexible model, this means low bias (it can model the true function well) but high variance (it also fits the noise in each specific training set). Small changes in training data → very different models.

    **Strong regularisation (large λ):** the model is constrained to be simple — weights are forced toward zero or the training is stopped early. This introduces bias (the model can no longer represent complex functions) but reduces variance (the model is more stable across different training samples).

    **Optimal λ:** minimises the sum of squared bias and variance, which is the generalisation error.

    ```
    Generalisation Error ≈ Bias² + Variance + Irreducible Noise

    Low λ  → Low Bias, High Variance
    High λ → High Bias, Low Variance
    Best λ → Minimises Bias² + Variance
    ```

    **Diagnostic approach:**
    - If validation error > training error by a large gap → high variance → increase regularisation (larger λ).
    - If both training and validation error are high → high bias → decrease regularisation (smaller λ) or use a more expressive model.

    This is the central insight: regularisation is the main tool you have for explicitly shifting the bias-variance tradeoff in a controlled way.

---

### Q13: When would you prefer Lasso over Ridge in practice?

??? "Show answer"
    Choose Lasso over Ridge in these situations:

    **1. Feature selection is the goal:**
    If you want the model to identify which features matter and which don't, Lasso's sparsity is ideal. After fitting, non-zero coefficients point to the relevant features. This is common in genomics, clinical research, and any domain with many candidate predictors.

    **2. You believe most features are irrelevant:**
    If you have 500 features but only expect ~20 to be truly predictive, Lasso will identify and retain those 20. Ridge retains all 500 with small weights, which is wasteful and less interpretable.

    **3. Interpretability and parsimony:**
    A model with 10 features is easier to explain, validate with domain experts, and maintain than one with 200 features. Sparse models also have lower inference latency.

    **4. p >> n (more features than observations):**
    Lasso naturally handles this regime by selecting a subset of at most n features.

    **Choose Ridge over Lasso when:**
    - Most features contribute small, non-zero effects (e.g., genetic data with polygenic traits).
    - Features are highly correlated — Lasso picks one arbitrarily; Ridge distributes weights stably across the group.
    - Coefficient stability matters more than sparsity.
    - You care more about prediction than identification of important features.

    **When you're unsure:** use ElasticNet with cross-validated l1_ratio. If the best l1_ratio > 0.7, you effectively have Lasso-like behaviour. If it's < 0.3, Ridge-like.

---

### Q14: How do you implement L1/L2 regularisation in scikit-learn?

??? "Show answer"
    Sklearn's API is consistent: the regularisation strength is controlled by `alpha` (for regression) or `C` (for classifiers, where `C = 1/λ` — larger C means less regularisation).

    **Linear regression:**

    ```python
    from sklearn.linear_model import Ridge, Lasso, ElasticNet, RidgeCV, LassoCV

    # Ridge (L2)
    ridge = Ridge(alpha=1.0)                # alpha = λ
    ridge_cv = RidgeCV(alphas=[0.1, 1, 10], cv=5)

    # Lasso (L1)
    lasso = Lasso(alpha=0.1, max_iter=10000)
    lasso_cv = LassoCV(cv=5, max_iter=10000)

    # ElasticNet (L1 + L2)
    en = ElasticNet(alpha=0.1, l1_ratio=0.5)
    ```

    **Logistic regression (classification):**

    ```python
    from sklearn.linear_model import LogisticRegression

    # L2 regularisation (default)
    lr_l2 = LogisticRegression(penalty='l2', C=1.0, solver='lbfgs')

    # L1 regularisation (requires compatible solver)
    lr_l1 = LogisticRegression(penalty='l1', C=1.0, solver='liblinear')

    # ElasticNet for logistic regression
    lr_en = LogisticRegression(penalty='elasticnet', l1_ratio=0.5, solver='saga', C=1.0)

    # No regularisation
    lr_none = LogisticRegression(penalty=None)
    ```

    **Important: always scale features before applying regularisation.** The penalty penalises large weights — if features are on different scales, coefficients will have different magnitudes for reasons unrelated to their predictive importance. Scale first so the regularisation is applied fairly.

    ```python
    from sklearn.pipeline import Pipeline
    from sklearn.preprocessing import StandardScaler

    # This is the correct pattern
    pipe = Pipeline([
        ('scaler', StandardScaler()),
        ('model', Ridge(alpha=1.0))
    ])
    ```

---

### Q15: What is max-norm regularisation?

??? "Show answer"
    Max-norm regularisation constrains the L2 norm of the incoming weight vector for each neuron to be at most a fixed value `c`:

    ```
    ||wⱼ||₂ ≤ c   for all neurons j
    ```

    After each gradient update, if a neuron's weight vector exceeds the constraint, it is projected back onto the constraint surface by rescaling:

    ```python
    # Manual implementation of max-norm constraint
    import torch

    def apply_max_norm(model, max_norm=3.0):
        for param in model.parameters():
            if param.dim() > 1:  # weight matrices, not biases
                norm = param.data.norm(2, dim=1, keepdim=True)
                scale = (max_norm / norm).clamp(max=1.0)
                param.data.mul_(scale)
    ```

    ```python
    # In Keras
    from tensorflow.keras.constraints import MaxNorm
    from tensorflow.keras.layers import Dense

    Dense(128, kernel_constraint=MaxNorm(max_value=3.0, axis=0))
    ```

    **Why use it over L2:**
    - Unlike L2, max-norm bounds the absolute size of the weights regardless of the learning rate or gradient magnitude.
    - It pairs well with large learning rates and dropout: if a neuron's weights grow uncontrollably (due to large gradients with a high learning rate), max-norm clips them. This provides a safety net that L2 doesn't guarantee.
    - Useful in RNNs to prevent exploding gradients (though gradient clipping is more common there).

    **Typical values:** `c = 3` or `c = 4` are common choices for fully connected layers in deep networks. The right value is dataset-dependent and should be tuned.

    Max-norm is less commonly used than L1/L2 or dropout, but it's a useful tool when training instability is the primary concern.

---
