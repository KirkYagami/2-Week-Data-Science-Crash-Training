# Model Evaluation

Evaluation is where you determine whether your model is actually useful — not just on training data, but in the real world. Interviewers use these questions to test statistical rigour, practical judgement, and awareness of common failure modes.

---

### Q1: What is the difference between training error and generalisation error?

??? "Show answer"
    **Training error** is the loss measured on the same data used to train the model. It reflects how well the model has memorised the training set.

    **Generalisation error** (also called test error or out-of-sample error) is the expected loss on new, unseen data drawn from the same distribution. It reflects how well the model has *learned* the underlying pattern, not just the specific training examples.

    The gap between the two is the key signal for diagnosing model problems:

    - **Both high**: high bias — the model is too simple, underfitting.
    - **Training low, generalisation high**: high variance — the model is overfitting, memorising training data including noise.
    - **Both low**: good generalisation — the model has learned the true pattern.

    Training error is a biased (optimistic) estimate of generalisation error because the model was optimised on that data. This is why we never evaluate a model on its training set — the reported performance would be misleading.

    The fundamental goal of every regularisation, early stopping, and cross-validation technique is to close the gap between training error and generalisation error, ideally by keeping both low.

---

### Q2: What is cross-validation? Explain K-Fold, Stratified K-Fold, and Time-Series CV.

??? "Show answer"
    Cross-validation (CV) is a technique for estimating generalisation error by partitioning the training data into complementary subsets, training on some, and evaluating on others. It gives a more reliable estimate than a single train/val split.

    **K-Fold CV:**
    Split the data into K equal folds. In each of K iterations, train on K-1 folds and evaluate on the remaining fold. The CV score is the mean performance across K folds.

    ```python
    from sklearn.model_selection import KFold, cross_val_score

    kf = KFold(n_splits=5, shuffle=True, random_state=42)
    scores = cross_val_score(model, X, y, cv=kf, scoring='roc_auc')
    print(f"CV AUC: {scores.mean():.4f} ± {scores.std():.4f}")
    ```

    **Stratified K-Fold:**
    Like K-Fold but each fold preserves the class distribution of the full dataset. Critical for imbalanced classification — without stratification, some folds may have zero or very few minority-class examples.

    ```python
    from sklearn.model_selection import StratifiedKFold

    skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    scores = cross_val_score(model, X, y, cv=skf, scoring='f1')
    ```

    **Time-Series CV (Walk-Forward Validation):**
    For time-ordered data, random splits are invalid — you'd be training on the future to predict the past. Time-series CV uses expanding or rolling windows where training always comes before validation chronologically.

    ```python
    from sklearn.model_selection import TimeSeriesSplit

    tscv = TimeSeriesSplit(n_splits=5)
    for train_idx, val_idx in tscv.split(X):
        X_train, X_val = X[train_idx], X[val_idx]
        # train_idx always contains earlier dates than val_idx
    ```

    Use 5 or 10 folds as a default. Stratified K-Fold is the default choice for classification. Time-Series CV is mandatory whenever the order of observations matters.

---

### Q3: What is the difference between precision and recall? When do you prioritise each?

??? "Show answer"
    Both measure performance on the positive class only, from different perspectives.

    ```
    Precision = TP / (TP + FP)  — of predicted positives, how many are correct?
    Recall    = TP / (TP + FN)  — of actual positives, how many did we find?
    ```

    **Precision** answers: "When the model says positive, how often is it right?"
    **Recall** answers: "Of all actual positives, how many did the model catch?"

    There is a fundamental tradeoff: increasing recall (catching more true positives) typically increases false positives, reducing precision.

    **Prioritise precision when:**
    - False positives are costly. Email spam filters: flagging a legitimate email as spam (FP) is very disruptive. You'd rather miss some spam (low recall) than falsely block legitimate emails.
    - Recommendation systems: recommending irrelevant items (FP) damages user trust.

    **Prioritise recall when:**
    - False negatives are costly. Cancer screening: missing a real cancer (FN) is far worse than a false positive that triggers a follow-up biopsy.
    - Fraud detection: missing a real fraud (FN) has severe financial consequences. False alarms (FP) are tolerable.
    - Security threat detection: missing a real threat has catastrophic consequences.

    In interviews, always ask: "What is the cost of a false positive relative to a false negative?" — the answer drives the metric choice and threshold selection.

---

### Q4: What is the F1 score and what is its limitation?

??? "Show answer"
    The F1 score is the harmonic mean of precision and recall:

    ```
    F1 = 2 × (Precision × Recall) / (Precision + Recall)
    ```

    The harmonic mean is used instead of the arithmetic mean because it penalises extreme imbalances between precision and recall. A model with precision = 1.0 and recall = 0.0 gets F1 = 0, not 0.5.

    F1 is the most common summary metric when you want to balance precision and recall and don't have a strong preference for one over the other.

    ```python
    from sklearn.metrics import f1_score, classification_report

    f1 = f1_score(y_true, y_pred)                          # binary
    f1_macro = f1_score(y_true, y_pred, average='macro')   # multiclass: unweighted mean
    f1_weighted = f1_score(y_true, y_pred, average='weighted')  # weighted by class support
    print(classification_report(y_true, y_pred))
    ```

    **Limitations:**
    - F1 treats precision and recall as equally important. If your use case weights one more heavily, use F-beta (see Q5).
    - F1 doesn't account for true negatives. The Matthews Correlation Coefficient (MCC) is a more informative metric for imbalanced binary classification as it considers all four confusion matrix cells.
    - F1 depends on the decision threshold (default 0.5). Comparing F1 at different thresholds requires the full precision-recall curve.
    - For multiclass, `macro` vs `weighted` averaging gives very different results on imbalanced datasets — always specify which you're reporting.

---

### Q5: What is the F-beta score?

??? "Show answer"
    F-beta is a generalisation of F1 that allows you to weight recall more or less than precision via the parameter β:

    ```
    F_β = (1 + β²) × (Precision × Recall) / (β² × Precision + Recall)
    ```

    - **β = 1**: F1 — equal weight to precision and recall.
    - **β = 2**: F2 — recall is weighted twice as heavily as precision. Use when missing a positive is more costly than a false alarm.
    - **β = 0.5**: F0.5 — precision is weighted twice as heavily as recall. Use when false positives are more costly than false negatives.

    ```python
    from sklearn.metrics import fbeta_score

    # β=2: recall is more important (e.g., disease detection)
    f2 = fbeta_score(y_true, y_pred, beta=2)

    # β=0.5: precision is more important (e.g., email spam filter)
    f_half = fbeta_score(y_true, y_pred, beta=0.5)
    ```

    The right value of β comes from the business context — specifically from the relative cost ratio of false negatives to false positives. If a missed fraud costs 10× more than a false alarm, β = √10 ≈ 3.16.

---

### Q6: What is the ROC curve and what does AUC represent?

??? "Show answer"
    The **ROC curve** (Receiver Operating Characteristic) plots the True Positive Rate (TPR / recall) against the False Positive Rate (FPR) as the classification threshold is swept from 0 to 1.

    ```
    TPR = TP / (TP + FN)  — same as recall / sensitivity
    FPR = FP / (FP + TN)  — false alarm rate = 1 - specificity
    ```

    **AUC** (Area Under the ROC Curve) summarises the curve as a single number between 0 and 1.

    - **AUC = 1.0**: perfect classifier — achieves TPR = 1, FPR = 0 for some threshold.
    - **AUC = 0.5**: no better than random guessing (diagonal line).
    - **AUC = 0.0**: perfect *inverse* classifier (flip the predictions).

    **Probabilistic interpretation of AUC:** the probability that a randomly chosen positive example is ranked higher (given a higher model score) than a randomly chosen negative example. AUC = 0.85 means the model correctly ranks a random positive above a random negative 85% of the time.

    ```python
    from sklearn.metrics import roc_auc_score, roc_curve
    import matplotlib.pyplot as plt

    auc = roc_auc_score(y_true, y_proba)

    fpr, tpr, thresholds = roc_curve(y_true, y_proba)
    plt.plot(fpr, tpr, label=f'AUC = {auc:.3f}')
    plt.plot([0,1], [0,1], 'k--')
    plt.xlabel('FPR'); plt.ylabel('TPR')
    ```

    AUC is threshold-independent — it evaluates the model's ranking ability across all thresholds, not just the default 0.5.

---

### Q7: When should you use PR-AUC instead of ROC-AUC?

??? "Show answer"
    Use **PR-AUC** (Precision-Recall AUC) when the positive class is rare (class imbalance).

    ROC-AUC can be misleadingly optimistic on imbalanced datasets. Because ROC includes the True Negative Rate (TN/(TN+FP)) implicitly through FPR, a model that correctly identifies most negatives will have a low FPR even if it misses many positives. With 99% negatives, even a poor model can achieve AUC > 0.95.

    PR-AUC, on the other hand, focuses entirely on the positive class:
    - Precision: what fraction of predicted positives are real?
    - Recall: what fraction of real positives did we catch?
    - True negatives don't appear in either metric.

    ```python
    from sklearn.metrics import average_precision_score, precision_recall_curve

    pr_auc = average_precision_score(y_true, y_proba)

    precision, recall, thresholds = precision_recall_curve(y_true, y_proba)
    import matplotlib.pyplot as plt
    plt.plot(recall, precision, label=f'PR-AUC = {pr_auc:.3f}')
    plt.xlabel('Recall'); plt.ylabel('Precision')
    ```

    **Rule of thumb:**
    - Balanced classes → use ROC-AUC.
    - Severe imbalance (< 10% positive) → use PR-AUC.
    - Both are complementary — report both for fraud detection, medical diagnosis, anomaly detection.

---

### Q8: What is log loss and when is it used?

??? "Show answer"
    Log loss (cross-entropy loss) measures the quality of predicted probabilities, not just predicted classes. It penalises confident wrong predictions heavily.

    For binary classification:
    ```
    Log Loss = -(1/n) Σ [yᵢ log(pᵢ) + (1 - yᵢ) log(1 - pᵢ)]
    ```

    Where `pᵢ` is the predicted probability of class 1 for observation i.

    - A perfect model (predicting 1.0 for all positives, 0.0 for all negatives) has log loss = 0.
    - Predicting 0.5 for everything gives log loss = log(2) ≈ 0.693.
    - Lower is always better.
    - Predicting 0.99 for a true negative gives massive loss: `-log(1 - 0.99) = -log(0.01) ≈ 4.6`.

    ```python
    from sklearn.metrics import log_loss

    ll = log_loss(y_true, y_proba)
    ```

    **When to use it:**
    - When predicted probabilities matter, not just the predicted class (e.g., risk scoring, model calibration evaluation).
    - As a training objective for logistic regression, neural networks, and boosted classifiers.
    - In competitions (Kaggle often uses log loss for probabilistic outputs).
    - For comparing calibration: a model with better calibrated probabilities will have lower log loss even at the same AUC.

---

### Q9: What metrics do you use for regression?

??? "Show answer"
    The right regression metric depends on the error distribution, the cost structure, and whether you need interpretability in the same units as the target.

    **MSE (Mean Squared Error):** `(1/n) Σ(yᵢ - ŷᵢ)²`
    - Heavily penalises large errors (squared).
    - Differentiable everywhere — good as a training loss.
    - Not in the same units as y.

    **RMSE (Root MSE):** `√MSE`
    - Same units as y — interpretable as "typical error magnitude".
    - Still penalises large errors heavily.
    - Most commonly reported regression metric.

    **MAE (Mean Absolute Error):** `(1/n) Σ|yᵢ - ŷᵢ|`
    - More robust to outliers than RMSE.
    - Same units as y.
    - Harder to optimise (non-differentiable at zero).

    **MAPE (Mean Absolute Percentage Error):** `(100/n) Σ|( yᵢ - ŷᵢ)/yᵢ|`
    - Scale-independent — useful for comparing across datasets.
    - Undefined when yᵢ = 0; biased (penalises under-prediction more than over-prediction).

    **R²:** proportion of variance explained. Useful for relative comparison; misleading as an absolute measure.

    ```python
    from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
    import numpy as np

    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae  = mean_absolute_error(y_true, y_pred)
    r2   = r2_score(y_true, y_pred)
    mape = np.mean(np.abs((y_true - y_pred) / y_true)) * 100
    ```

---

### Q10: What is the difference between RMSE and MAE?

??? "Show answer"
    Both measure average error magnitude, but they treat large errors differently.

    **RMSE** squares the errors before averaging, so large errors are penalised quadratically. A single prediction that's 100 units off contributes as much to RMSE as 100 predictions that are each 10 units off.

    **MAE** treats all errors proportionally — a 100-unit error counts as 100 times a 1-unit error, not 10,000 times.

    **Consequence:** when your dataset has outliers or your loss function genuinely should penalise large errors more (e.g., delivering a package 10 days late is far worse than 10 packages delivered 1 day late), use RMSE. When errors should be treated proportionally and you don't want outliers to dominate your evaluation metric, use MAE.

    **Which to use:**
    - RMSE is more sensitive to outliers — if your test set has unusual extreme values, RMSE will be dominated by them.
    - MAE is more interpretable as "the average error is X units."
    - If RMSE >> MAE, your model has large errors on a small subset of points — investigate those cases.
    - Report both in practice: RMSE reveals if you have problematic outlier predictions; MAE gives the typical error.

    **Optimisation:** models trained to minimise MSE learn to predict conditional means; models trained on MAE learn to predict conditional medians.

---

### Q11: What is MAPE and what is its weakness?

??? "Show answer"
    MAPE (Mean Absolute Percentage Error) expresses the average prediction error as a percentage of the actual value:

    ```
    MAPE = (100/n) Σ |( yᵢ - ŷᵢ) / yᵢ|
    ```

    **Why it's useful:** scale-independent. MAPE = 12% means errors are roughly 12% of the true value, regardless of whether the target is measured in dollars, kilograms, or page views. This makes it easy to communicate to non-technical stakeholders and to compare models across different products or datasets.

    **Weaknesses:**

    1. **Undefined when yᵢ = 0**: division by zero. Demand forecasting with zero-sales periods is a common failure case. Workaround: use sMAPE (symmetric MAPE) or skip zero-actual observations.

    2. **Asymmetric penalty**: MAPE penalises under-predictions more than over-predictions. If yᵢ = 100 and ŷᵢ = 50 → error = 50%. If yᵢ = 100 and ŷᵢ = 150 → error = 50%. But: if yᵢ = 50 and ŷᵢ = 100 → error = 100%, while if yᵢ = 150 and ŷᵢ = 100 → error = 33%. The model is incentivised to over-predict.

    3. **Sensitive to small actual values**: a small absolute error on a near-zero actual produces a huge MAPE. A forecast of 1 vs actual of 0.5 gives 100% error.

    **sMAPE (symmetric MAPE):**
    ```
    sMAPE = (100/n) Σ |yᵢ - ŷᵢ| / ((|yᵢ| + |ŷᵢ|) / 2)
    ```
    Still not perfect but more balanced between over- and under-prediction.

---

### Q12: What is model calibration?

??? "Show answer"
    A model is **calibrated** if the predicted probabilities match the actual empirical frequencies. When a calibrated model predicts P = 0.7 for a set of instances, approximately 70% of those instances are truly positive.

    **Why it matters:** any downstream use of the probability value depends on calibration — risk thresholds, expected value calculations, decision cost analysis. A model can have high AUC (good ranking) but poor calibration (bad probabilities), making the raw probabilities useless for decisions.

    **Diagnosis:**
    - **Reliability diagram (calibration curve)**: bin predictions into deciles, plot mean predicted probability vs. observed fraction of positives.
    - **Expected Calibration Error (ECE)**: weighted average gap between mean predicted probability and observed frequency across bins.
    - **Brier Score**: MSE between predicted probabilities and binary outcomes — rewards both accuracy and calibration.

    ```python
    from sklearn.calibration import CalibrationDisplay
    from sklearn.metrics import brier_score_loss

    CalibrationDisplay.from_predictions(y_true, y_proba, n_bins=10)
    brier = brier_score_loss(y_true, y_proba)  # lower is better; 0 is perfect
    ```

    **Calibration methods:**
    - **Platt scaling**: fit logistic regression on top of model scores using a held-out set.
    - **Isotonic regression**: non-parametric, more flexible, but needs more calibration data.

    ```python
    from sklearn.calibration import CalibratedClassifierCV

    calibrated = CalibratedClassifierCV(base_model, method='isotonic', cv=5)
    calibrated.fit(X_train, y_train)
    ```

---

### Q13: How do you detect data drift and model degradation in production?

??? "Show answer"
    In production, data distributions change over time (data drift) and model performance degrades (model drift / concept drift). Monitoring for this is a core MLOps responsibility.

    **Types of drift:**
    - **Feature drift (covariate shift)**: the input distribution P(X) changes — e.g., user demographics shift, a new device type appears.
    - **Label drift (prior shift)**: the target distribution P(y) changes — e.g., fraud rate increases.
    - **Concept drift**: the relationship P(y|X) changes — the same features now map to different outcomes.

    **Detection methods:**

    - **Statistical tests on feature distributions**: compare recent data to a reference window using KS test (continuous), chi-squared test (categorical), or Population Stability Index (PSI).

    ```python
    from scipy.stats import ks_2samp

    stat, p_value = ks_2samp(reference['feature'], current['feature'])
    if p_value < 0.05:
        print("Significant drift detected")

    # PSI: > 0.1 is moderate drift, > 0.25 is severe
    def psi(expected, actual, bins=10):
        breakpoints = np.percentile(expected, np.linspace(0, 100, bins+1))
        exp_pct = np.histogram(expected, breakpoints)[0] / len(expected)
        act_pct = np.histogram(actual, breakpoints)[0] / len(actual)
        return np.sum((act_pct - exp_pct) * np.log((act_pct + 1e-8) / (exp_pct + 1e-8)))
    ```

    - **Prediction distribution monitoring**: track the distribution of model outputs over time. Sudden shifts in score distributions signal drift.
    - **Performance monitoring**: track business metrics (conversion rate, fraud rate) and model metrics (precision, recall) when labels are available with latency.
    - **Shadow deployment / champion-challenger**: run a new model in shadow mode alongside the champion to compare predictions before promoting.

---

### Q14: What is a learning curve and what does it tell you?

??? "Show answer"
    A learning curve plots model performance (training and validation) as a function of training set size. It diagnoses whether you need more data or a better model.

    ```python
    from sklearn.model_selection import learning_curve
    import numpy as np
    import matplotlib.pyplot as plt

    train_sizes, train_scores, val_scores = learning_curve(
        model, X, y,
        train_sizes=np.linspace(0.1, 1.0, 10),
        cv=5,
        scoring='roc_auc',
        n_jobs=-1
    )

    train_mean = train_scores.mean(axis=1)
    val_mean = val_scores.mean(axis=1)

    plt.plot(train_sizes, train_mean, label='Training score')
    plt.plot(train_sizes, val_mean, label='Validation score')
    plt.xlabel('Training set size'); plt.ylabel('AUC')
    plt.legend()
    ```

    **Interpreting the curve:**

    - **High bias (underfitting)**: both training and validation performance are low and converge to a low value. Adding more data won't help much. Fix: more complex model, better features.
    - **High variance (overfitting)**: training performance is high, validation is much lower. A gap persists even as training size grows. Fix: regularise, reduce model complexity, or add more data (the gap usually narrows with more data).
    - **Good fit**: training and validation converge to a high value. The model is learning the true pattern.

    Learning curves are one of the most diagnostic tools available — they tell you whether your effort should go toward collecting more data or improving the model.

---

### Q15: What is a validation curve?

??? "Show answer"
    A validation curve plots training and validation performance as a function of a single hyperparameter value. It shows the optimal range for that hyperparameter and its effect on bias-variance.

    ```python
    from sklearn.model_selection import validation_curve
    import numpy as np
    import matplotlib.pyplot as plt

    param_range = [1, 2, 5, 10, 20, 50, 100, 200]

    train_scores, val_scores = validation_curve(
        RandomForestClassifier(n_jobs=-1),
        X, y,
        param_name='n_estimators',
        param_range=param_range,
        cv=5,
        scoring='roc_auc'
    )

    plt.semilogx(param_range, train_scores.mean(axis=1), label='Train')
    plt.semilogx(param_range, val_scores.mean(axis=1), label='Validation')
    ```

    **Interpreting the curve:**
    - **Both low across the range**: the hyperparameter doesn't matter much, or other hyperparameters are more important.
    - **Training high, validation peaks then drops**: overfitting begins after the peak — the optimal value is at the peak of the validation curve.
    - **Training and validation both increase and plateau**: the hyperparameter is being increased in a direction that helps but has diminishing returns.

    Validation curves are more targeted than full grid search — use them to understand the sensitivity to one parameter at a time before launching a broader search.

---

### Q16: What is hyperparameter tuning? Grid Search vs Random Search vs Bayesian Optimisation.

??? "Show answer"
    Hyperparameters are model configuration settings set before training (e.g., learning rate, max depth, regularisation strength). Hyperparameter tuning finds the configuration that maximises validation performance.

    **Grid Search (GridSearchCV):**
    Exhaustively tries every combination of specified hyperparameter values. Guarantees finding the best combination *within the grid* but scales exponentially with the number of hyperparameters.

    ```python
    from sklearn.model_selection import GridSearchCV

    param_grid = {'max_depth': [3, 5, 7], 'learning_rate': [0.01, 0.1, 0.3]}
    gs = GridSearchCV(model, param_grid, cv=5, scoring='roc_auc', n_jobs=-1)
    gs.fit(X_train, y_train)
    print(gs.best_params_, gs.best_score_)
    ```

    **Random Search (RandomizedSearchCV):**
    Samples hyperparameter combinations randomly from specified distributions. With the same compute budget, random search explores a wider range of values than grid search, especially when some hyperparameters matter much more than others.

    ```python
    from sklearn.model_selection import RandomizedSearchCV
    from scipy.stats import loguniform, randint

    param_dist = {'max_depth': randint(2, 15), 'learning_rate': loguniform(0.001, 0.5)}
    rs = RandomizedSearchCV(model, param_dist, n_iter=50, cv=5, scoring='roc_auc', n_jobs=-1)
    ```

    **Bayesian Optimisation:**
    Builds a probabilistic model (surrogate) of the objective function and uses it to decide the next hyperparameter configuration to try — exploiting known good regions while exploring uncertain ones. More efficient than random search when function evaluations are expensive.

    ```python
    # Using optuna
    import optuna

    def objective(trial):
        params = {
            'max_depth': trial.suggest_int('max_depth', 2, 10),
            'learning_rate': trial.suggest_float('learning_rate', 1e-3, 0.5, log=True)
        }
        model = XGBClassifier(**params)
        return cross_val_score(model, X_train, y_train, cv=5, scoring='roc_auc').mean()

    study = optuna.create_study(direction='maximize')
    study.optimize(objective, n_trials=100)
    ```

    **Choosing:** Random search for fast exploration of wide ranges. Bayesian optimisation when training is expensive and you need to be sample-efficient.

---

### Q17: How do you compare two models — is the difference statistically significant?

??? "Show answer"
    Simply comparing mean CV scores is insufficient — you need to account for the variance across folds before claiming one model is better.

    **McNemar's test (for classifiers):**
    Tests whether two classifiers make the same mistakes on the same examples. Works on paired predictions from a held-out test set.

    ```python
    from statsmodels.stats.contingency_tables import mcnemar
    import numpy as np

    # b: A correct, B wrong; c: A wrong, B correct
    preds_a = model_a.predict(X_test)
    preds_b = model_b.predict(X_test)

    b = np.sum((preds_a == y_test) & (preds_b != y_test))
    c = np.sum((preds_a != y_test) & (preds_b == y_test))

    result = mcnemar([[b, c], [c, b]])
    print(f"p-value: {result.pvalue:.4f}")
    ```

    **Paired t-test on CV fold scores:**
    If you have K-fold CV scores for both models, a paired t-test tests whether the mean difference is significantly non-zero.

    ```python
    from scipy.stats import ttest_rel

    scores_a = cross_val_score(model_a, X, y, cv=10, scoring='roc_auc')
    scores_b = cross_val_score(model_b, X, y, cv=10, scoring='roc_auc')

    t_stat, p_value = ttest_rel(scores_a, scores_b)
    print(f"Mean diff: {scores_a.mean() - scores_b.mean():.4f}, p={p_value:.4f}")
    ```

    **Practical guidance:** for production decisions, statistical significance is necessary but not sufficient. A p-value < 0.05 on a 0.001 AUC difference is statistically significant but practically irrelevant. Ask: "Does the improvement justify the engineering cost of deploying a more complex model?"

---

### Q18: What is the holdout method vs cross-validation tradeoff?

??? "Show answer"
    **Holdout method:** split data once into train/validation/test. Train on train, tune on validation, report final performance on test.

    - Pros: fast, no repeated training, test set is truly unseen.
    - Cons: high variance in the estimate (a single unlucky split can make a model look much better or worse than it is). Wastes data — validation set examples are never used for training.

    **Cross-validation:** repeatedly split data, train on K-1 folds, evaluate on 1 fold. Average the K evaluation scores.

    - Pros: lower variance in the performance estimate (uses all data for both training and evaluation). More reliable, especially with small datasets.
    - Cons: K times more expensive to run. Doesn't give a single model to deploy — you need to retrain on the full dataset after finding the best hyperparameters.

    **The tradeoff:**

    | | Holdout | Cross-Validation |
    |---|---|---|
    | Estimate variance | High | Low |
    | Computational cost | Low | K× higher |
    | Data efficiency | Lower | Higher |
    | Best for | Large datasets | Small/medium datasets |

    **Best practice:** use cross-validation for model selection and hyperparameter tuning, then retrain the final model on the full training set with the best hyperparameters. Report performance on a held-out test set that was never used during model selection.

---

### Q19: What is the difference between online and offline evaluation?

??? "Show answer"
    **Offline evaluation** assesses model performance on historical held-out data before deployment. It uses logged data, pre-collected labels, and static metrics like AUC, RMSE, F1.

    - Fast, cheap, reproducible.
    - Can be run before any real users are affected.
    - Limitation: historical data may not represent future behaviour. The evaluation environment doesn't match production — users interact with the world differently than they would if your model were deployed.

    **Online evaluation** assesses model performance in production by exposing real users to the model's predictions and measuring the real-world impact.

    - **A/B testing (randomised controlled trial)**: split users randomly between a control group (old model) and a treatment group (new model). Measure business metrics (click-through rate, conversion, revenue) directly. Most reliable form of online evaluation because it isolates causal impact.
    - **Interleaving**: for ranking/recommendation systems, interleave results from both models and track which items users actually click. More statistically efficient than A/B testing for ranking problems.
    - **Bandit testing**: dynamically allocates traffic to better-performing variants, balancing exploration and exploitation.

    **The gap between offline and online metrics** is the central challenge in applied ML. A model can have better offline AUC but worse online business metrics if the offline labels don't capture the true objective, or if the model changes user behaviour in ways the historical data doesn't anticipate (the Goodhart's law problem).

    Always validate offline improvements with online tests before committing to a model change.

---
