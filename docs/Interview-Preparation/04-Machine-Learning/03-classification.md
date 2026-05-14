# Classification

Classification questions appear at every stage of a data science interview — from the fundamentals of logistic regression to the subtleties of threshold selection and class imbalance. Know the mechanics, not just the API calls.

---

### Q1: What is logistic regression and how does it work?

??? "Show answer"
    Logistic regression is a linear classifier that models the probability that an observation belongs to a class. Despite the name, it's used for classification, not regression.

    It applies the sigmoid function to a linear combination of the input features:

    ```
    P(y=1 | x) = σ(β₀ + β₁x₁ + ... + βₙxₙ)
               = 1 / (1 + e^(-(Xβ)))
    ```

    The model outputs a probability between 0 and 1. A threshold (default 0.5) converts it to a class label.

    **Training:** coefficients are estimated by maximising the log-likelihood (equivalent to minimising binary cross-entropy loss). Unlike linear regression, there is no closed-form solution — gradient descent or iteratively reweighted least squares (IRLS) is used.

    **What makes it "linear":** the decision boundary is a hyperplane in the feature space. The log-odds (logit) is linear in the features:

    ```
    log(P/(1-P)) = Xβ
    ```

    **Key properties:**
    - Outputs calibrated probabilities (when properly trained).
    - Interpretable: a coefficient `βᵢ` represents the change in log-odds for a one-unit increase in `xᵢ`.
    - Assumes a linear decision boundary — won't work well on non-linearly separable data without feature engineering.

---

### Q2: What is the sigmoid function and why is it used in logistic regression?

??? "Show answer"
    The sigmoid (logistic) function maps any real number to the range (0, 1):

    ```
    σ(z) = 1 / (1 + e^(-z))
    ```

    **Why it's used:**
    - Probabilities must lie between 0 and 1. A raw linear combination `Xβ` can produce any real value — the sigmoid squashes it to the valid probability range.
    - It's differentiable everywhere, enabling gradient-based optimisation.
    - Its derivative has a convenient form: `σ'(z) = σ(z)(1 - σ(z))`, which simplifies backpropagation.

    ```python
    import numpy as np
    def sigmoid(z):
        return 1 / (1 + np.exp(-z))

    # Key values:
    # sigmoid(0)   = 0.5
    # sigmoid(2)   ≈ 0.88
    # sigmoid(-2)  ≈ 0.12
    # sigmoid(±∞)  → 1 or 0
    ```

    **The connection to log-odds:** if `P = σ(z)`, then `log(P/(1-P)) = z`. This is why logistic regression is said to model the log-odds as a linear function of the features.

    In neural networks, the sigmoid is used as an activation function in the output layer for binary classification, and the softmax generalises it for multiclass problems.

---

### Q3: How does a decision tree make splits?

??? "Show answer"
    A decision tree recursively partitions the feature space by selecting the feature and threshold that best separates the target classes (for classification) or reduces variance (for regression).

    **The splitting algorithm (CART):**
    1. For each feature, consider every possible threshold.
    2. For each (feature, threshold) pair, measure the impurity of the resulting left and right child nodes.
    3. Select the split that maximises the reduction in impurity (information gain).
    4. Recurse on each child node until a stopping criterion is met (max depth, min samples per leaf, minimum impurity decrease).

    **Impurity measures for classification:**
    - Gini impurity (default in sklearn)
    - Entropy / information gain

    **Stopping criteria prevent overfitting:**
    - `max_depth`: limits tree depth.
    - `min_samples_split`: minimum samples required to split a node.
    - `min_samples_leaf`: minimum samples in a leaf node.

    ```python
    from sklearn.tree import DecisionTreeClassifier

    clf = DecisionTreeClassifier(
        criterion='gini',
        max_depth=5,
        min_samples_leaf=10
    )
    clf.fit(X_train, y_train)
    ```

    Decision trees are greedy — they make the locally optimal split at each step without backtracking. This means they don't always find the globally optimal tree, but the greedy approach is computationally tractable.

---

### Q4: What is Gini impurity vs entropy, and which should you use?

??? "Show answer"
    Both are measures of node impurity — how mixed the class labels are at a given node. A pure node (all one class) has impurity = 0.

    **Gini impurity:**
    ```
    Gini = 1 - Σ pᵢ²
    ```
    For binary classification: `Gini = 2 · p · (1 - p)`, which is maximised at p = 0.5 (value = 0.5).

    **Entropy:**
    ```
    Entropy = -Σ pᵢ · log₂(pᵢ)
    ```
    For binary classification, maximised at p = 0.5 (value = 1.0).

    **Differences:**
    - Computationally, Gini is slightly faster (no logarithm).
    - Entropy tends to create more balanced trees because it penalises impurity more aggressively near equal class proportions.
    - In practice, the difference in model quality is almost always negligible. Both produce similar trees on most datasets.

    **When does it matter?** Almost never in practice — the main benefit of knowing the distinction is being able to explain it in an interview. Default to Gini (sklearn's default) and tune other hyperparameters first.

    The information gain from a split is: `IG = H(parent) - weighted_avg(H(children))`, where H is either Gini or entropy.

---

### Q5: What is the difference between a decision tree and a random forest?

??? "Show answer"
    A **decision tree** is a single model — it partitions the feature space greedily and is prone to overfitting (high variance). A fully grown tree memorises the training data.

    A **random forest** is an ensemble of decision trees, each trained on:
    1. A **bootstrap sample** of the training data (sampling with replacement — bagging).
    2. A **random subset of features** at each split (feature randomness).

    Predictions are made by majority vote (classification) or averaging (regression) across all trees.

    **Why random forests outperform single trees:**
    - Individual trees have high variance. Averaging many trees reduces variance while keeping bias roughly constant (bias-variance tradeoff).
    - Feature randomness decorrelates the trees. If one strong predictor dominates, all trees in a plain bagged ensemble would be similar. By restricting features at each split, trees see different "views" of the data and their errors partially cancel out.

    **Tradeoffs:**
    - Random forests lose interpretability — you can't visualise a forest the way you can a single tree.
    - Training is parallelisable (trees are independent), so they scale well.
    - Slower prediction than a single tree (need to query all trees).

    ```python
    from sklearn.ensemble import RandomForestClassifier
    rf = RandomForestClassifier(n_estimators=200, max_features='sqrt', n_jobs=-1)
    ```

---

### Q6: How does KNN work and what are its limitations?

??? "Show answer"
    K-Nearest Neighbours (KNN) is a lazy, non-parametric algorithm. To classify a new point:
    1. Compute the distance (usually Euclidean) from the query point to every training point.
    2. Find the K nearest training points.
    3. Return the majority class among those K neighbours (for classification) or their average (for regression).

    There is no training phase — the model is the training data itself. All computation happens at prediction time.

    **Hyperparameter K:**
    - Small K (e.g., 1): very sensitive to noise — high variance, low bias.
    - Large K: smoother decision boundary — low variance, higher bias.
    - Choose K via cross-validation, typically trying odd values to avoid ties.

    **Limitations:**
    - **Computationally expensive at inference**: O(n) distance calculations per query — scales poorly with dataset size. Mitigated with KD-trees or ball trees.
    - **Curse of dimensionality**: in high-dimensional spaces, all points become equidistant and the notion of "nearest" breaks down. Feature selection or dimensionality reduction is essential.
    - **Sensitive to feature scale**: features with larger ranges dominate the distance calculation. Always normalise/standardise before using KNN.
    - **Memory intensive**: must store all training data.
    - **Performs poorly on imbalanced data**: the majority class dominates neighbourhoods.

    ```python
    from sklearn.preprocessing import StandardScaler
    from sklearn.neighbors import KNeighborsClassifier
    from sklearn.pipeline import Pipeline

    pipe = Pipeline([('scaler', StandardScaler()), ('knn', KNeighborsClassifier(n_neighbors=5))])
    ```

---

### Q7: What is the kernel trick in SVM?

??? "Show answer"
    The kernel trick allows SVMs to learn non-linear decision boundaries without explicitly computing the coordinates of data points in a high-dimensional feature space.

    **The core idea:** the SVM optimisation only requires inner products `xᵢ · xⱼ` between training points, not the coordinates themselves. A kernel function `K(xᵢ, xⱼ)` computes this inner product in the transformed space directly:

    ```
    K(xᵢ, xⱼ) = φ(xᵢ) · φ(xⱼ)
    ```

    Where `φ` is the (potentially infinite-dimensional) feature mapping. You get the benefit of working in the transformed space without the computational cost of explicitly constructing it.

    **Common kernels:**
    - **Linear**: `K(xᵢ, xⱼ) = xᵢᵀxⱼ` — no transformation; use when data is linearly separable.
    - **Polynomial**: `K(xᵢ, xⱼ) = (xᵢᵀxⱼ + c)^d` — captures interactions up to degree d.
    - **RBF (Gaussian)**: `K(xᵢ, xⱼ) = exp(-γ||xᵢ - xⱼ||²)` — infinite-dimensional feature space; most commonly used.
    - **Sigmoid**: `K(xᵢ, xⱼ) = tanh(αxᵢᵀxⱼ + c)`.

    ```python
    from sklearn.svm import SVC
    svm_rbf = SVC(kernel='rbf', C=1.0, gamma='scale')
    ```

    The RBF kernel's γ controls how far the influence of each training point reaches. High γ → narrow influence → high variance; low γ → broad influence → high bias.

---

### Q8: What is the difference between hard margin and soft margin SVM?

??? "Show answer"
    **Hard margin SVM** requires that all training points be correctly classified and lie outside the margin. It finds the hyperplane that maximises the margin with zero training errors.

    Problem: it only works when data is perfectly linearly separable. In practice, data is noisy and classes overlap — hard margin SVM either fails to converge or massively overfits to outliers.

    **Soft margin SVM** introduces slack variables `ξᵢ ≥ 0` that allow some points to violate the margin or even be misclassified:

    ```
    Minimise: (1/2)||w||² + C · Σξᵢ
    Subject to: yᵢ(wᵀxᵢ + b) ≥ 1 - ξᵢ, ξᵢ ≥ 0
    ```

    The hyperparameter **C** controls the tradeoff:
    - **Large C**: strong penalty for margin violations — smaller margin, fewer training errors, risk of overfitting.
    - **Small C**: allows more violations — larger margin, more regularised, risk of underfitting.

    ```python
    from sklearn.svm import SVC
    # C=1.0 is a reasonable default; tune via cross-validation
    svm = SVC(C=1.0, kernel='rbf')
    ```

    In practice, you always use soft margin SVM. Hard margin is a theoretical construct. The key interview point is that C is a regularisation parameter: small C = more regularisation.

---

### Q9: What is Naive Bayes and when does the "naive" assumption hold?

??? "Show answer"
    Naive Bayes is a generative classifier based on Bayes' theorem. It predicts the class with the highest posterior probability:

    ```
    P(y | x₁, ..., xₙ) ∝ P(y) · P(x₁|y) · P(x₂|y) · ... · P(xₙ|y)
    ```

    The "naive" assumption is **conditional independence**: given the class label, each feature is independent of every other feature. This is almost never literally true but is computationally powerful — it reduces the parameter estimation problem from O(2^n) to O(n).

    **When the naive assumption holds well enough:**
    - **Text classification** (spam detection, sentiment analysis): word frequencies are treated as independent given the class. In practice, even though words co-occur, Naive Bayes performs remarkably well because the rank ordering of probabilities is preserved even when the exact values are wrong.
    - **High-dimensional problems where features are only weakly correlated**.
    - **Very small datasets** where more complex models overfit.

    **Variants:**
    - **Gaussian NB**: assumes features follow a Gaussian distribution (continuous features).
    - **Multinomial NB**: for count data (word frequencies in text).
    - **Bernoulli NB**: for binary features (word presence/absence).

    ```python
    from sklearn.naive_bayes import MultinomialNB, GaussianNB
    gnb = GaussianNB()
    mnb = MultinomialNB(alpha=1.0)  # alpha = Laplace smoothing
    ```

---

### Q10: How do you handle the zero-frequency problem in Naive Bayes?

??? "Show answer"
    The zero-frequency problem occurs when a feature value that appears in the test set was never seen during training for a particular class. This gives `P(xᵢ | y) = 0`, which zeroes out the entire product regardless of how well all other features match the class.

    **Example:** if the word "cryptocurrency" never appeared in training emails labelled as "not spam", then any test email containing that word gets P(not spam) = 0 no matter what else it says.

    **Solution: Laplace (additive) smoothing**

    Add a pseudocount `α` (usually 1) to every feature count:

    ```
    P(xᵢ | y) = (count(xᵢ, y) + α) / (count(y) + α · |V|)
    ```

    Where `|V|` is the vocabulary size (number of unique feature values). This ensures no probability is ever exactly zero.

    ```python
    from sklearn.naive_bayes import MultinomialNB

    # alpha=1 is standard Laplace smoothing
    # alpha < 1 is Lidstone smoothing (lighter regularisation)
    mnb = MultinomialNB(alpha=1.0)
    ```

    Laplace smoothing is a form of regularisation — it prevents the model from being overconfident about feature absence. The effect diminishes as you have more training data.

---

### Q11: What is the difference between one-vs-rest and one-vs-one for multiclass classification?

??? "Show answer"
    These are strategies for extending binary classifiers to handle K > 2 classes.

    **One-vs-Rest (OvR) / One-vs-All (OvA):**
    - Train K binary classifiers, one per class.
    - Classifier k learns to distinguish class k from all other classes combined.
    - At inference: run all K classifiers, pick the class whose classifier outputs the highest score/confidence.
    - Trains K models, each on the full dataset.

    **One-vs-One (OvO):**
    - Train a binary classifier for every pair of classes: K(K-1)/2 classifiers total.
    - Classifier (i,j) distinguishes class i from class j only.
    - At inference: each classifier votes; the class with the most votes wins.
    - Each classifier trains on a smaller subset of data (only two classes), but you have many more classifiers.

    **When to use which:**
    - **OvR** is the default for most algorithms (logistic regression, linear SVM). Computationally cheaper for large K.
    - **OvO** is preferred for SVMs with non-linear kernels — each classifier trains on fewer points, making SVM's O(n²) complexity manageable.

    ```python
    from sklearn.multiclass import OneVsRestClassifier, OneVsOneClassifier
    from sklearn.svm import SVC

    ovr = OneVsRestClassifier(SVC())
    ovo = OneVsOneClassifier(SVC())
    ```

    Logistic regression and tree-based methods handle multiclass natively (softmax for logistic regression, majority vote for trees).

---

### Q12: What is the softmax function?

??? "Show answer"
    Softmax converts a vector of raw scores (logits) into a probability distribution over K classes:

    ```
    softmax(zᵢ) = e^zᵢ / Σⱼ e^zⱼ
    ```

    Each output is between 0 and 1, and all outputs sum to 1 — making it a valid probability distribution.

    ```python
    import numpy as np

    def softmax(z):
        e_z = np.exp(z - np.max(z))  # subtract max for numerical stability
        return e_z / e_z.sum()

    logits = np.array([2.0, 1.0, 0.1])
    probs = softmax(logits)
    # array([0.659, 0.242, 0.099])
    ```

    **Why subtract max(z)?** To prevent numerical overflow from `e^z` when z is large. It doesn't change the output because `e^(z-c) / Σe^(z-c)` = `e^z / Σe^z`.

    **Relationship to sigmoid:** for K=2, softmax reduces to the sigmoid function.

    **Used in:**
    - The output layer of neural networks for multiclass classification.
    - Multinomial logistic regression.
    - Attention mechanisms in transformers (over the sequence dimension).

    The cross-entropy loss paired with softmax is the standard training objective for multiclass classification problems.

---

### Q13: How do you choose the decision threshold in binary classification?

??? "Show answer"
    Most classifiers output a probability score. The threshold converts this to a binary label: predict class 1 if `P(y=1|x) ≥ threshold`. The default of 0.5 is rarely optimal.

    **How to choose:**

    1. **Based on business costs**: define the cost of a false positive vs. a false negative. For fraud detection, a missed fraud (FN) is much costlier than a false alarm (FP). Lower the threshold to catch more frauds, accepting more false alarms.

    2. **Precision-Recall curve**: plot precision and recall at every threshold. Choose the threshold that hits your target precision or recall based on business requirements.

    3. **F1 or F-beta score optimisation**: find the threshold that maximises F1 (or Fβ if you weight recall more/less than precision).

    4. **ROC curve**: plot TPR vs FPR. The optimal threshold is often near the "elbow" of the curve (maximising Youden's J = TPR - FPR).

    ```python
    from sklearn.metrics import precision_recall_curve, roc_curve
    import numpy as np

    probs = model.predict_proba(X_test)[:, 1]

    # Find threshold that maximises F1
    precision, recall, thresholds = precision_recall_curve(y_test, probs)
    f1_scores = 2 * precision * recall / (precision + recall + 1e-8)
    best_threshold = thresholds[np.argmax(f1_scores[:-1])]

    y_pred = (probs >= best_threshold).astype(int)
    ```

    **Key insight**: never use the default 0.5 threshold for imbalanced datasets. The model's calibrated probability might put most true positives at 0.1–0.3, and 0.5 would miss all of them.

---

### Q14: What is label imbalance and how does it affect classifiers differently?

??? "Show answer"
    Class imbalance occurs when one class vastly outnumbers the other (e.g., 99% negative, 1% positive in fraud detection). A naive classifier that predicts all-negative achieves 99% accuracy — which is useless.

    **How different classifiers are affected:**

    - **Logistic regression and SVMs**: optimise accuracy-related objectives, which are dominated by the majority class. They tend to underpredict the minority class. Fix: use `class_weight='balanced'` to re-weight the loss.
    - **Decision trees / Random forests**: similarly biased toward majority class splits. `class_weight='balanced'` or `min_samples_leaf` tuning helps.
    - **KNN**: majority class dominates neighbourhood voting.
    - **Naive Bayes**: prior probability estimate is skewed; can be fixed by adjusting the prior.

    **Mitigation strategies:**

    - **Algorithmic**: `class_weight='balanced'` in sklearn re-weights each class inversely proportional to its frequency.
    - **Resampling**: oversample the minority class (SMOTE creates synthetic minority examples), undersample the majority class, or both.
    - **Threshold tuning**: lower the classification threshold so fewer minority-class instances are missed.
    - **Evaluation**: use PR-AUC, F1, or Matthews Correlation Coefficient instead of accuracy.

    ```python
    from imblearn.over_sampling import SMOTE
    from sklearn.linear_model import LogisticRegression

    X_res, y_res = SMOTE().fit_resample(X_train, y_train)
    clf = LogisticRegression(class_weight='balanced')
    ```

---

### Q15: What is a calibrated classifier?

??? "Show answer"
    A classifier is **calibrated** if its output probabilities match the actual frequency of positive outcomes. If a calibrated model predicts P = 0.7 for a set of instances, approximately 70% of those instances should actually be positive.

    **Why calibration matters:** many downstream decisions depend on accurate probabilities — risk scoring, expected value calculations, decision thresholds. A well-ranking model (high AUC) can still be poorly calibrated.

    **Diagnosing calibration:**
    - Plot a **reliability diagram** (calibration curve): bin predictions by probability, plot mean predicted probability vs. actual fraction of positives. A perfectly calibrated model sits on the diagonal.

    ```python
    from sklearn.calibration import CalibrationDisplay, CalibratedClassifierCV

    # Visualise calibration
    CalibrationDisplay.from_predictions(y_test, probs, n_bins=10)
    ```

    **Common calibration issues:**
    - **Random forests**: tend to compress probabilities toward 0.5 (under-confident).
    - **SVMs**: raw margin scores are not probabilities at all.
    - **Naive Bayes**: overconfident — probabilities cluster near 0 and 1.

    **Calibration methods:**
    - **Platt scaling**: fit a logistic regression on the model's output scores using a held-out calibration set.
    - **Isotonic regression**: non-parametric, more flexible, but needs more data.

    ```python
    from sklearn.calibration import CalibratedClassifierCV
    from sklearn.svm import SVC

    svm = SVC()
    calibrated_svm = CalibratedClassifierCV(svm, method='platt', cv=5)
    calibrated_svm.fit(X_train, y_train)
    probs = calibrated_svm.predict_proba(X_test)
    ```

    Always calibrate when your use case depends on the probability value, not just the predicted class.

---
