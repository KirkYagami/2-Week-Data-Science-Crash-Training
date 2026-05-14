# Ensemble Methods

Ensemble methods dominate tabular data competitions and production ML systems. Interviews probe the mechanics of how and why ensembling works — not just the names of the algorithms.

---

### Q1: What is an ensemble method and why does it work?

??? "Show answer"
    An ensemble combines predictions from multiple models to produce a better prediction than any individual model alone.

    **Why it works — the bias-variance decomposition:**
    ```
    Expected Error = Bias² + Variance + Irreducible Noise
    ```

    - **Bagging** reduces variance by averaging many high-variance, low-bias models (e.g., deep decision trees). Individual trees overfit; their average doesn't.
    - **Boosting** reduces bias by sequentially fitting models to the residual errors of previous models. Each model corrects what the last got wrong.
    - **Stacking** reduces both by learning which base models are right in which regions of the input space.

    **Conditions for ensembling to help:**
    1. Individual models must be better than random (accuracy > 50% for binary classification).
    2. Individual models must make **different errors** — they need to be diverse. An ensemble of identical models is no better than a single model.

    The diversity requirement is why we use random sampling (bagging) or different model types (stacking) — we want errors to partially cancel out when aggregated.

---

### Q2: What is bagging and what problem does it solve?

??? "Show answer"
    **Bagging** (Bootstrap AGGregating) trains multiple copies of the same learning algorithm on different bootstrap samples of the training data, then aggregates their predictions.

    **The bootstrap sample:** draw n observations from the training set with replacement. Each bootstrap sample contains ~63.2% unique training observations (the rest are duplicates), with ~36.8% of original observations left out (out-of-bag observations).

    **Aggregation:** majority vote for classification, averaging for regression.

    **The problem it solves:** high-variance models. A single decision tree trained to full depth memorises the training data. Small changes in training data produce very different trees. Bagging averages over this instability — the bias stays roughly the same, the variance drops by ≈1/B (where B = number of trees), as long as trees are uncorrelated.

    ```python
    from sklearn.ensemble import BaggingClassifier
    from sklearn.tree import DecisionTreeClassifier

    bagging = BaggingClassifier(
        estimator=DecisionTreeClassifier(max_depth=None),  # fully grown trees
        n_estimators=100,
        bootstrap=True,
        n_jobs=-1,
        random_state=42
    )
    ```

    Bagging works best with unstable learners (deep decision trees, neural networks). It provides minimal benefit for stable, high-bias models (e.g., linear regression).

---

### Q3: How does Random Forest work?

??? "Show answer"
    Random Forest extends bagging by adding **feature randomness** at each split. Each tree in the forest is:

    1. Trained on a bootstrap sample of the data.
    2. At each node split, only a **random subset of features** (`max_features`) is considered as candidates for splitting — not all features.

    Predictions: majority vote (classification) or mean (regression) across all B trees.

    ```python
    from sklearn.ensemble import RandomForestClassifier

    rf = RandomForestClassifier(
        n_estimators=300,       # more trees = lower variance; diminishing returns
        max_features='sqrt',    # sqrt(p) features at each split (classification default)
        max_depth=None,         # grow full trees
        min_samples_leaf=1,
        bootstrap=True,
        oob_score=True,         # compute out-of-bag estimate
        n_jobs=-1,
        random_state=42
    )
    rf.fit(X_train, y_train)
    print(f"OOB score: {rf.oob_score_:.4f}")
    ```

    **Key hyperparameters to tune (in order of importance):**
    1. `max_features`: controls feature randomness and tree decorrelation. `sqrt` for classification, `1/3` of features for regression.
    2. `min_samples_leaf`: acts as regularisation — larger values reduce overfitting.
    3. `n_estimators`: more is always better but with diminishing returns; stop when OOB error stabilises.
    4. `max_depth`: rarely needs tuning if `min_samples_leaf` is set.

---

### Q4: What is the difference between Random Forest and a bagged decision tree?

??? "Show answer"
    The only difference is **feature randomness at each split**.

    - **Bagged decision trees**: each tree sees a bootstrap sample of data. At each node, the best split is chosen from **all** features.
    - **Random Forest**: each tree sees a bootstrap sample of data. At each node, the best split is chosen from a **random subset** of `max_features` features.

    This seemingly small change has a large impact on model quality.

    **Why feature randomness matters:** if there is one very strong predictor, every tree in a plain bagged ensemble will use that feature at the root node and produce similar splits throughout. The trees are correlated — averaging correlated trees reduces variance less than averaging uncorrelated trees.

    By restricting features at each split, Random Forest forces trees to find different ways to partition the data. Trees become less correlated, and averaging them reduces variance more effectively.

    Mathematically, the variance of the ensemble mean is:
    ```
    Var(mean) = ρσ² + (1-ρ)σ²/B
    ```
    Where ρ is the pairwise correlation between trees. Reducing ρ (via feature randomness) reduces ensemble variance directly.

---

### Q5: What is feature randomness in Random Forest?

??? "Show answer"
    At each node split, instead of searching through all p features for the best split, Random Forest randomly selects `max_features` features and only considers those as candidates.

    **Default values:**
    - Classification: `max_features = sqrt(p)` — e.g., for 100 features, try 10 at each split.
    - Regression: `max_features = p/3` — e.g., for 100 features, try 33 at each split.

    ```python
    from sklearn.ensemble import RandomForestClassifier

    # sqrt(p) — sklearn default for classification
    rf_sqrt = RandomForestClassifier(max_features='sqrt')

    # Try all features — this is just bagging (no feature randomness)
    rf_all = RandomForestClassifier(max_features=None)

    # Custom: try 20 features
    rf_20 = RandomForestClassifier(max_features=20)
    ```

    **Tuning `max_features`:**
    - Smaller values → more randomness, more decorrelated trees, lower variance but higher bias per tree.
    - Larger values → less randomness, trees are more similar, higher variance but more optimal splits.
    - It's a hyperparameter — cross-validate over `[sqrt(p), log2(p), p/3, p]`.

    For very high-dimensional data (p >> 1000), even `sqrt(p)` might be too many. Reducing `max_features` further can improve both speed and accuracy.

---

### Q6: What is out-of-bag (OOB) error?

??? "Show answer"
    When bootstrap sampling, each tree only sees ~63.2% of the training data. The remaining ~36.8% of observations (those not selected in the bootstrap sample) are called **out-of-bag (OOB) samples** for that tree.

    For each training observation i, the OOB predictions come from all trees that did **not** train on observation i. Aggregating these OOB predictions across all observations gives the OOB error estimate.

    **Why it's useful:** OOB error is a free validation estimate. It approximates the generalisation error without a separate validation split — the OOB observations functionally act like a hold-out set for each tree.

    ```python
    from sklearn.ensemble import RandomForestClassifier

    rf = RandomForestClassifier(n_estimators=200, oob_score=True, random_state=42)
    rf.fit(X_train, y_train)

    print(f"OOB accuracy: {rf.oob_score_:.4f}")
    # Access OOB predictions:
    # rf.oob_decision_function_  → shape (n_train, n_classes)
    ```

    **OOB vs cross-validation:**
    - OOB is free (no extra computation).
    - OOB tends to slightly underestimate true test performance (equivalent to a 63/37 train/test split).
    - K-fold CV is still preferable for final model evaluation, but OOB is excellent for quick hyperparameter tuning.

---

### Q7: What is boosting and how is it different from bagging?

??? "Show answer"
    **Bagging:** trains models in **parallel**, independently, on bootstrap samples. Final prediction aggregates them (vote/average). Goal: reduce variance.

    **Boosting:** trains models **sequentially**. Each model focuses on the mistakes of its predecessors. Final prediction is a weighted combination. Goal: reduce bias.

    | | Bagging | Boosting |
    |---|---|---|
    | Training | Parallel | Sequential |
    | Weighting | Equal weight on all samples | More weight on misclassified/high-error samples |
    | Base learner | High-variance, low-bias (deep trees) | Low-variance, high-bias (stumps/shallow trees) |
    | Reduces | Variance | Bias |
    | Risk | Low risk of overfitting | Can overfit with too many rounds |

    **Why boosting reduces bias:** each successive model fits the residual errors left by the ensemble so far. A sequence of weak learners (each better than random) can collectively model complex relationships that no single learner could capture.

    In practice, boosting (XGBoost, LightGBM) typically outperforms bagging (Random Forest) on structured tabular data, but is more sensitive to hyperparameters and more prone to overfitting.

---

### Q8: How does AdaBoost work?

??? "Show answer"
    AdaBoost (Adaptive Boosting) sequentially trains weak classifiers (typically decision stumps — single-node trees), adapting sample weights so that each new classifier focuses on the observations the previous classifiers got wrong.

    **Algorithm:**
    1. Initialise sample weights uniformly: `wᵢ = 1/n`.
    2. For each round t = 1, ..., T:
       - Train a weak learner `hₜ` on the weighted training set.
       - Compute the weighted error: `εₜ = Σᵢ wᵢ · 𝟙[hₜ(xᵢ) ≠ yᵢ]`
       - Compute the learner weight: `αₜ = 0.5 · ln((1 - εₜ)/εₜ)` — better classifiers get higher weight.
       - Update sample weights: misclassified samples get higher weight, correctly classified samples get lower weight.
       - Normalise weights.
    3. Final prediction: `H(x) = sign(Σₜ αₜ · hₜ(x))`

    ```python
    from sklearn.ensemble import AdaBoostClassifier
    from sklearn.tree import DecisionTreeClassifier

    ada = AdaBoostClassifier(
        estimator=DecisionTreeClassifier(max_depth=1),  # stumps
        n_estimators=200,
        learning_rate=0.1,
        random_state=42
    )
    ```

    AdaBoost is sensitive to outliers — noisy observations that are consistently misclassified will receive very large weights and can destabilise training. Gradient Boosting (which operates on residuals rather than sample reweighting) is generally more robust.

---

### Q9: How does Gradient Boosting work?

??? "Show answer"
    Gradient Boosting builds an ensemble by sequentially fitting new models to the **negative gradient of the loss function** with respect to the current ensemble's predictions. This is a generalisation of AdaBoost to arbitrary differentiable loss functions.

    **Algorithm:**
    1. Initialise with a constant prediction: `F₀(x) = argmin_γ Σ L(yᵢ, γ)` (e.g., the mean for MSE loss).
    2. For each round t = 1, ..., T:
       - Compute pseudo-residuals (negative gradients): `rᵢₜ = -∂L(yᵢ, Fₜ₋₁(xᵢ))/∂Fₜ₋₁(xᵢ)`
       - Fit a weak learner (shallow tree) to the pseudo-residuals.
       - Find the optimal step size η (line search).
       - Update the ensemble: `Fₜ(x) = Fₜ₋₁(x) + η · hₜ(x)`

    For MSE loss, the pseudo-residuals are the actual residuals `yᵢ - ŷᵢ`, making the connection to "fitting residuals" literal. For other losses (log loss, MAE), the pseudo-residuals are different.

    ```python
    from sklearn.ensemble import GradientBoostingClassifier

    gb = GradientBoostingClassifier(
        n_estimators=300,
        learning_rate=0.05,     # shrinkage — smaller is better with more trees
        max_depth=4,            # shallow trees are typical
        subsample=0.8,          # stochastic gradient boosting
        min_samples_leaf=20,
        random_state=42
    )
    ```

    **Stochastic Gradient Boosting:** setting `subsample < 1.0` uses a random subsample of training data at each round, introducing randomness that reduces variance and speeds up training.

---

### Q10: What is XGBoost and what makes it better than vanilla gradient boosting?

??? "Show answer"
    XGBoost (Extreme Gradient Boosting) is an optimised implementation of gradient boosting with several engineering and algorithmic improvements:

    **Algorithmic improvements:**
    - **Regularised objective**: explicitly adds L1 (α) and L2 (λ) regularisation on leaf weights to the boosting objective, preventing overfitting.
    - **Second-order gradients**: uses both the gradient (first derivative) and the Hessian (second derivative) of the loss for more accurate tree fitting — analogous to Newton's method vs. gradient descent.
    - **Pruning**: grows trees to max depth and then prunes back branches that don't improve the objective, rather than stopping early (depth-first vs. breadth-first).

    **Engineering improvements:**
    - **Column block structure**: pre-sorts feature values once and reuses the sorted order across iterations — avoids repeated sorting overhead.
    - **Cache-aware access patterns**: designed for CPU cache efficiency.
    - **Out-of-core computation**: handles datasets larger than RAM.
    - **Parallel and distributed training**.
    - **Sparse-aware split finding**: handles missing values natively by learning the optimal default direction for missing values.

    ```python
    import xgboost as xgb

    model = xgb.XGBClassifier(
        n_estimators=500,
        learning_rate=0.05,
        max_depth=6,
        subsample=0.8,
        colsample_bytree=0.8,   # fraction of features per tree
        reg_alpha=0.1,          # L1
        reg_lambda=1.0,         # L2
        eval_metric='logloss',
        early_stopping_rounds=50,
        random_state=42
    )
    model.fit(X_train, y_train, eval_set=[(X_val, y_val)], verbose=False)
    ```

---

### Q11: What are the key hyperparameters of XGBoost?

??? "Show answer"
    XGBoost has many parameters but a focused subset drives most of the performance.

    **Tree structure:**
    - `max_depth` (default 6): depth of each tree. Deeper trees → more complex interactions, more overfitting. Typical range: 3–8.
    - `min_child_weight` (default 1): minimum sum of instance weights in a leaf. Larger values → more conservative model. Key regularisation parameter.
    - `gamma` (default 0): minimum loss reduction required to make a split. Larger → fewer splits, more conservative.

    **Randomisation (variance reduction):**
    - `subsample` (default 1.0): fraction of training data sampled per tree. 0.5–0.9 is typical.
    - `colsample_bytree` (default 1.0): fraction of features sampled per tree. 0.5–0.9 is typical.
    - `colsample_bylevel` / `colsample_bynode`: further granularity for feature sampling.

    **Regularisation:**
    - `reg_alpha` (L1 on leaf weights, default 0): promotes sparsity in leaf weights.
    - `reg_lambda` (L2 on leaf weights, default 1): shrinks leaf weights.

    **Boosting:**
    - `learning_rate` / `eta` (default 0.3): shrinkage applied to each tree's contribution. Lower → need more trees, but generalises better. Tune jointly with `n_estimators`.
    - `n_estimators`: set high and use early stopping to find the optimal value.

    **Tuning strategy:** start with `max_depth=6, learning_rate=0.1`, tune tree structure, then add randomisation, then fine-tune learning rate. Always use early stopping.

---

### Q12: What is LightGBM and how does it differ from XGBoost?

??? "Show answer"
    LightGBM (Light Gradient Boosting Machine, by Microsoft) achieves faster training and often better accuracy than XGBoost through two core innovations:

    **1. Gradient-based One-Side Sampling (GOSS):**
    Retains all instances with large gradients (high error — these are most informative for the next tree) and randomly samples only a fraction of instances with small gradients. This reduces the data size without sacrificing much accuracy.

    **2. Exclusive Feature Bundling (EFB):**
    Sparse features that rarely take non-zero values simultaneously are bundled together into a single feature, reducing the effective feature dimensionality.

    **3. Leaf-wise tree growth vs. depth-wise:**
    - XGBoost grows trees level by level (depth-wise / breadth-first).
    - LightGBM grows the leaf with the maximum loss reduction (leaf-wise / best-first). This produces more accurate trees with fewer leaves, but can overfit on small datasets.

    ```python
    import lightgbm as lgb

    model = lgb.LGBMClassifier(
        n_estimators=1000,
        learning_rate=0.05,
        num_leaves=31,          # key LightGBM parameter (not max_depth)
        min_child_samples=20,   # minimum data in a leaf
        subsample=0.8,
        colsample_bytree=0.8,
        reg_alpha=0.1,
        reg_lambda=0.1,
        random_state=42
    )
    model.fit(X_train, y_train,
              eval_set=[(X_val, y_val)],
              callbacks=[lgb.early_stopping(50), lgb.log_evaluation(0)])
    ```

    **In practice:** LightGBM is typically 5–10x faster than XGBoost on large datasets. `num_leaves` replaces `max_depth` as the primary complexity control — a rule of thumb: `num_leaves < 2^max_depth`.

---

### Q13: What is CatBoost?

??? "Show answer"
    CatBoost (Categorical Boosting, by Yandex) is a gradient boosting library designed to handle categorical features natively and to reduce prediction shift (a form of overfitting specific to gradient boosting).

    **Key innovations:**

    **1. Native categorical feature support:**
    CatBoost computes an ordered target statistic for each categorical level — a form of target encoding that's computed on a random permutation of the training data to avoid target leakage. You pass categorical column indices directly; no manual encoding needed.

    **2. Ordered boosting:**
    Standard gradient boosting has a subtle leakage issue: residuals are computed using a model trained on the same data used to compute the gradient. CatBoost uses an ordered (permutation-based) approach that computes residuals on data the tree hasn't seen, reducing overfitting.

    **3. Symmetric trees:**
    CatBoost grows oblivious (symmetric) trees — at each level, the same feature and threshold are used for all splits. This creates smaller, faster-to-evaluate models and acts as regularisation.

    ```python
    from catboost import CatBoostClassifier

    cat_features = [0, 2, 5]  # column indices of categorical features

    model = CatBoostClassifier(
        iterations=500,
        learning_rate=0.05,
        depth=6,
        cat_features=cat_features,
        eval_metric='Logloss',
        early_stopping_rounds=50,
        random_seed=42,
        verbose=0
    )
    model.fit(X_train, y_train, eval_set=(X_val, y_val))
    ```

    CatBoost often outperforms XGBoost and LightGBM on datasets with many categorical features without manual preprocessing.

---

### Q14: What is stacking?

??? "Show answer"
    Stacking (stacked generalisation) trains a meta-model on the predictions of multiple base models. The meta-model learns which base model to trust in which regions of the input space.

    **Architecture:**
    - **Level 0 (base learners)**: diverse models trained on the training data (e.g., Random Forest, XGBoost, logistic regression, KNN).
    - **Level 1 (meta-learner)**: a model trained on the out-of-fold predictions of the base learners, with the original target as the label.

    **Critical implementation detail:** base learner predictions for the training set must be generated via cross-validation (out-of-fold predictions) to avoid data leakage. If a base model sees the training label during training and then predicts on that same data for the meta-model, it leaks information.

    ```python
    from sklearn.ensemble import StackingClassifier
    from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
    from sklearn.linear_model import LogisticRegression

    estimators = [
        ('rf', RandomForestClassifier(n_estimators=100, random_state=42)),
        ('gb', GradientBoostingClassifier(n_estimators=100, random_state=42))
    ]

    stack = StackingClassifier(
        estimators=estimators,
        final_estimator=LogisticRegression(),
        cv=5,              # generates out-of-fold predictions for meta-learner
        passthrough=False  # set True to also pass original features to meta-learner
    )
    stack.fit(X_train, y_train)
    ```

    Stacking typically outperforms any individual base learner when base models are sufficiently diverse and the meta-learner is kept simple (logistic regression or linear regression are common choices).

---

### Q15: What is blending?

??? "Show answer"
    Blending is a simpler alternative to stacking. Instead of out-of-fold cross-validation, blending uses a held-out validation set:

    1. Split training data into train and hold-out (blend) sets.
    2. Train base models on the train split.
    3. Generate predictions on the blend set and on the test set.
    4. Train a meta-model on the blend set predictions (using blend labels as targets).
    5. Apply the meta-model to the test set base predictions for final output.

    **Stacking vs. Blending:**

    | | Stacking | Blending |
    |---|---|---|
    | Training data for meta-model | Out-of-fold (full training set via CV) | Held-out validation split |
    | Leakage risk | Low (OOF prevents leakage) | Low (blend set is unseen by base models) |
    | Data efficiency | Uses all training data | Wastes the blend set for base model training |
    | Complexity | Higher | Lower |

    Blending is simpler to implement and faster, but wastes data (the blend set isn't used to train base models). Stacking is more data-efficient and is preferred when training data is limited. In Kaggle competitions, blending is popular due to its simplicity.

---

### Q16: When would you use a simple model over an ensemble?

??? "Show answer"
    Ensembles are not always the right choice. Use a simpler model when:

    - **Interpretability is required**: a single decision tree, logistic regression, or linear model can be explained to regulators, doctors, or business stakeholders. An ensemble is a black box.
    - **Latency constraints**: serving 500 trees at inference time adds milliseconds that may be unacceptable in real-time applications (fraud detection at checkout, ad bidding). A single model can be orders of magnitude faster.
    - **Training data is small**: ensembles require enough data to train many diverse models meaningfully. With very limited data, a simple regularised model often generalises better.
    - **Deployment simplicity**: a single model is easier to serialise, version, monitor, and debug in production.
    - **The problem is approximately linear**: if a linear model achieves 95% of the performance of an XGBoost ensemble, the engineering cost of deploying the ensemble may not be justified.
    - **Debugging and maintenance**: when a model produces a wrong prediction, a simple model makes it far easier to understand why.

    The right model is the simplest one that meets the business requirements — not the most complex one that wins a benchmark.

---

### Q17: How do you interpret feature importance in a gradient boosting model?

??? "Show answer"
    Gradient boosting models expose several types of feature importance, each with different meanings and limitations:

    **Gain (default in XGBoost):** the average improvement in the loss function brought by a feature across all splits where it is used. Features with higher gain contribute more to reducing the training loss. This is the most meaningful measure.

    **Frequency / Split count:** the number of times a feature is used in splits. Biased toward high-cardinality features and can be misleading.

    **Coverage:** the average number of training samples affected by splits using this feature.

    ```python
    import xgboost as xgb
    import pandas as pd

    importance_df = pd.DataFrame({
        'feature': X.columns,
        'gain': model.get_booster().get_score(importance_type='gain').values(),
        'freq': model.get_booster().get_score(importance_type='weight').values()
    }).sort_values('gain', ascending=False)
    ```

    **SHAP values (preferred):** model-agnostic, theoretically grounded (game theory / Shapley values). For each prediction, SHAP decomposes the output into additive contributions from each feature, showing both magnitude and direction of effect.

    ```python
    import shap

    explainer = shap.TreeExplainer(model)
    shap_values = explainer.shap_values(X_test)

    shap.summary_plot(shap_values, X_test)         # global feature importance
    shap.force_plot(explainer.expected_value,
                    shap_values[0], X_test.iloc[0]) # single prediction explanation
    ```

    Use built-in importance for quick iteration; use SHAP for production explanations or whenever you need to understand directionality.

---

### Q18: What is early stopping in boosting?

??? "Show answer"
    Early stopping halts the boosting process when validation performance stops improving, preventing overfitting without requiring a pre-specified number of estimators.

    **How it works:**
    1. Set aside a validation set.
    2. After each boosting round, evaluate the model on the validation set.
    3. Track the best validation score seen so far.
    4. If the validation score does not improve for `early_stopping_rounds` consecutive rounds, stop training and restore the best model.

    ```python
    import xgboost as xgb

    model = xgb.XGBClassifier(
        n_estimators=10000,        # set high — early stopping will cut it short
        learning_rate=0.01,
        early_stopping_rounds=50,  # stop if no improvement for 50 rounds
        eval_metric='logloss'
    )
    model.fit(
        X_train, y_train,
        eval_set=[(X_val, y_val)],
        verbose=100
    )
    print(f"Best round: {model.best_iteration}")
    print(f"Best score: {model.best_score:.4f}")
    ```

    ```python
    import lightgbm as lgb

    model = lgb.LGBMClassifier(n_estimators=10000)
    model.fit(X_train, y_train,
              eval_set=[(X_val, y_val)],
              callbacks=[lgb.early_stopping(50), lgb.log_evaluation(100)])
    ```

    **Why it's a form of regularisation:** fewer trees means a less complex model. Early stopping implicitly regularises the ensemble by preventing it from fitting noise in the training data. It's one of the most effective and practical regularisation tools for boosted models.

    **Practical note:** use a small learning rate (0.01–0.05) and let early stopping find the right number of trees. This is generally more reliable than guessing `n_estimators` upfront.

---
