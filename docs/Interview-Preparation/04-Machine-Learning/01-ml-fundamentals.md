# ML Fundamentals

Core concepts tested in almost every data science interview. These questions appear in phone screens, take-homes, and final rounds alike.

---

### Q1: What is the bias-variance tradeoff?

??? "Show answer"
    **Bias** is the error from wrong assumptions in the learning algorithm — a high-bias model is too simple and underfits the data.

    **Variance** is the error from sensitivity to small fluctuations in training data — a high-variance model is too complex and overfits.

    The tradeoff: decreasing bias tends to increase variance, and vice versa. The goal is to find the sweet spot that minimises total error on unseen data.

    ```
    Total Error = Bias² + Variance + Irreducible Noise
    ```

    Practical signals:
    - High bias → training error and validation error are both high
    - High variance → training error is low but validation error is much higher

---

### Q2: What is overfitting, and how do you detect it?

??? "Show answer"
    Overfitting occurs when a model learns the training data too well — including its noise — and fails to generalise to new data.

    **How to detect:**
    - Training accuracy >> validation accuracy (large gap)
    - Model performs well on seen data, poorly on held-out data
    - Learning curves show training loss decreasing while validation loss increases or plateaus

    **How to address:**
    - Reduce model complexity
    - Add regularisation (L1/L2)
    - Get more training data
    - Use dropout (for neural networks)
    - Early stopping
    - Cross-validation to get a realistic estimate of generalisation

---

### Q3: What is data leakage and why is it dangerous?

??? "Show answer"
    Data leakage occurs when information from outside the training dataset is used to build the model — information that would not be available at prediction time.

    **Types:**
    - **Target leakage**: a feature is derived from or correlated with the target after the fact (e.g. using "days in hospital" to predict whether someone was admitted)
    - **Train-test contamination**: preprocessing (e.g. scaling, encoding) is fit on the full dataset before splitting, so test data influences the training pipeline

    **Why dangerous:** the model appears to perform extremely well in evaluation but fails in production. A suspiciously high validation score is often the first sign.

    **Prevention:**
    - Always split data before any preprocessing
    - Use pipelines to encapsulate all transformations
    - Think carefully about the temporal ordering of features relative to the target

---

### Q4: What is the curse of dimensionality?

??? "Show answer"
    As the number of features grows, the volume of the feature space grows exponentially. This means:

    - Data points become increasingly sparse — distance between any two points converges
    - Distance-based algorithms (KNN, K-Means) degrade because "nearest neighbour" loses meaning
    - Models need exponentially more data to maintain the same statistical reliability
    - Visualisation and intuition break down

    **Practical implication:** feature selection and dimensionality reduction (PCA, t-SNE, UMAP) become essential as feature counts grow beyond ~20–50.

---

### Q5: What is cross-validation, and why is it better than a single train-test split?

??? "Show answer"
    Cross-validation evaluates a model on multiple held-out subsets of data, giving a more reliable estimate of generalisation performance.

    **K-Fold CV:**
    1. Split data into K equal folds
    2. Train on K-1 folds, validate on the remaining fold
    3. Repeat K times, rotating the validation fold
    4. Average the K validation scores

    **Why better than one split:**
    - A single split's score depends on which samples end up in the test set (high variance)
    - CV uses all data for both training and validation over the K rounds
    - Gives a mean and standard deviation of performance — more honest assessment

    **When to use stratified K-Fold:** when classes are imbalanced, to ensure each fold has the same class proportions.

---

### Q6: What is the difference between parametric and non-parametric models?

??? "Show answer"
    **Parametric models** assume a fixed functional form and learn a fixed set of parameters from data. The model structure does not grow with the training data.
    - Examples: Linear Regression, Logistic Regression, Naive Bayes
    - Pros: fast, interpretable, need less data
    - Cons: strong assumptions — if the assumption is wrong, performance suffers

    **Non-parametric models** make minimal assumptions about the data distribution. The model complexity can grow with the amount of training data.
    - Examples: KNN, Decision Trees, Kernel SVM, Gaussian Processes
    - Pros: flexible, can capture complex patterns
    - Cons: slower, need more data, risk of overfitting

---

### Q7: What is the difference between discriminative and generative models?

??? "Show answer"
    **Discriminative models** learn the decision boundary directly — they model P(Y|X).
    - Examples: Logistic Regression, SVM, Neural Networks
    - Goal: classify inputs without modelling how inputs are generated

    **Generative models** learn the joint distribution P(X, Y) and use Bayes' theorem to derive P(Y|X).
    - Examples: Naive Bayes, Gaussian Mixture Models, GANs, VAEs
    - Goal: model how data is generated — can also generate new samples

    **In practice:** discriminative models generally outperform generative models on classification tasks when labelled data is abundant. Generative models are more useful when data is scarce or when you need to generate synthetic data.

---

### Q8: How do you handle imbalanced datasets?

??? "Show answer"
    **At the data level:**
    - **Oversampling the minority class**: SMOTE generates synthetic minority samples
    - **Undersampling the majority class**: randomly remove majority samples (risk: losing information)
    - **Combination**: oversample minority + undersample majority

    **At the algorithm level:**
    - Use `class_weight='balanced'` in scikit-learn models
    - Adjust the decision threshold (instead of 0.5, use a threshold that maximises precision-recall tradeoff)
    - Use algorithms robust to imbalance: tree-based ensembles

    **At the evaluation level:**
    - Never use accuracy as the primary metric on imbalanced data
    - Use precision, recall, F1, PR-AUC, ROC-AUC
    - Check confusion matrix

    ```python
    from sklearn.linear_model import LogisticRegression
    model = LogisticRegression(class_weight='balanced')
    ```

---

### Q9: What is the No Free Lunch theorem?

??? "Show answer"
    No single learning algorithm works best for all problems. Across all possible datasets, every algorithm performs the same on average.

    **Practical implication:** there is no universally "best" model — you must choose and tune based on your specific data, task, and constraints. This is why model selection and experimentation matter.

---

### Q10: What is regularisation and when would you use it?

??? "Show answer"
    Regularisation adds a penalty to the loss function to discourage overly complex models, reducing variance at the cost of a small increase in bias.

    - **L1 (Lasso)**: penalty = λ × Σ|weights| → drives some weights to exactly zero → automatic feature selection
    - **L2 (Ridge)**: penalty = λ × Σweights² → shrinks all weights toward zero, none become exactly zero
    - **ElasticNet**: combination of L1 + L2

    **When to use:**
    - Many features relative to samples
    - Suspected multicollinearity (Ridge)
    - Need feature selection (Lasso)
    - High validation error relative to training error (overfitting signal)

---

### Q11: What is the difference between bagging and boosting?

??? "Show answer"
    Both are ensemble methods that combine multiple weak learners, but they do so differently.

    **Bagging (Bootstrap Aggregating):**
    - Trains models **in parallel** on random subsets of data (with replacement)
    - Reduces variance
    - Models are independent — errors are uncorrelated
    - Example: Random Forest

    **Boosting:**
    - Trains models **sequentially** — each model corrects the errors of the previous
    - Reduces bias
    - Models are dependent — later models focus on hard examples
    - Examples: AdaBoost, Gradient Boosting, XGBoost, LightGBM

    **Rule of thumb:** if your model underfits → try boosting. If it overfits → try bagging.

---

### Q12: What is a confusion matrix and what can you derive from it?

??? "Show answer"
    A confusion matrix shows actual vs predicted classes across all samples.

    ```
                    Predicted Positive   Predicted Negative
    Actual Positive       TP                   FN
    Actual Negative       FP                   TN
    ```

    Derived metrics:
    - **Accuracy** = (TP + TN) / total — misleading on imbalanced data
    - **Precision** = TP / (TP + FP) — of all predicted positives, how many are correct
    - **Recall (Sensitivity)** = TP / (TP + FN) — of all actual positives, how many did we catch
    - **F1 Score** = 2 × (Precision × Recall) / (Precision + Recall) — harmonic mean
    - **Specificity** = TN / (TN + FP) — how well negatives are identified

    **When to prioritise recall:** false negatives are costly (e.g. cancer screening, fraud detection)
    **When to prioritise precision:** false positives are costly (e.g. spam filters, legal investigations)

---

### Q13: What is the ROC curve and what does AUC tell you?

??? "Show answer"
    The **ROC (Receiver Operating Characteristic) curve** plots True Positive Rate (Recall) against False Positive Rate at every possible classification threshold.

    **AUC (Area Under the Curve):**
    - AUC = 1.0 → perfect classifier
    - AUC = 0.5 → random classifier (diagonal line)
    - AUC < 0.5 → worse than random (flip predictions)

    **What AUC actually means:** the probability that the model ranks a random positive example higher than a random negative example.

    **Limitation:** ROC-AUC can be misleading on heavily imbalanced datasets. Use **Precision-Recall AUC** (PR-AUC) instead — it is more sensitive to performance on the minority class.

---

### Q14: Explain the train / validation / test split. Why three sets, not two?

??? "Show answer"
    - **Training set**: the model learns from this
    - **Validation set**: used to tune hyperparameters and compare models during development
    - **Test set**: used exactly once to report final performance — never touched during development

    **Why not two sets:** if you repeatedly evaluate and tune on the test set, you are implicitly leaking information about it into your decisions. Over many rounds of tuning, you optimise for the test set and your reported metric becomes overly optimistic. The validation set absorbs this leakage; the test set remains a true held-out measure.

---

### Q15: What is feature importance and how do tree-based models calculate it?

??? "Show answer"
    Feature importance quantifies how much each feature contributes to predictions.

    **In tree-based models (impurity-based importance):**
    Each node in a tree splits on a feature. Importance is measured by the total reduction in impurity (Gini or entropy) that a feature achieves, weighted by the number of samples reaching that node, averaged across all trees.

    **Limitations of impurity-based importance:**
    - Biased toward high-cardinality features (many unique values)
    - Can rank correlated features arbitrarily

    **Better alternative:** **permutation importance** — randomly shuffle a feature's values and measure the drop in model performance. Available in scikit-learn.

    ```python
    from sklearn.inspection import permutation_importance
    result = permutation_importance(model, X_val, y_val, n_repeats=10)
    ```

---

### Q16: What is the difference between a hyperparameter and a model parameter?

??? "Show answer"
    **Model parameters** are learned from data during training.
    - Examples: weights in linear regression, split thresholds in decision trees

    **Hyperparameters** are set before training and control the learning process.
    - Examples: learning rate, max depth, number of estimators, regularisation strength λ

    Hyperparameter tuning methods: Grid Search, Random Search, Bayesian Optimisation (most efficient).

---

### Q17: Why can't you use accuracy as the only metric for a classification problem?

??? "Show answer"
    On an imbalanced dataset, a model that always predicts the majority class achieves high accuracy while being completely useless.

    **Example:** 95% of transactions are legitimate, 5% are fraud.
    A model that always predicts "legitimate" gets 95% accuracy but catches 0% of fraud.

    **Better metrics for imbalanced problems:**
    - Precision, Recall, F1 — focus on the positive (minority) class
    - PR-AUC — performance across all thresholds, sensitive to minority class
    - Matthews Correlation Coefficient (MCC) — balanced even for imbalanced classes

---

### Q18: What is gradient descent and what are its variants?

??? "Show answer"
    Gradient descent is an optimisation algorithm that iteratively moves model parameters in the direction of the negative gradient of the loss function to minimise loss.

    **Variants:**
    - **Batch GD**: computes gradient over the entire dataset per step — accurate but slow and memory-heavy
    - **Stochastic GD (SGD)**: one sample per step — fast and noisy, can escape local minima
    - **Mini-batch GD**: small batch per step — practical compromise, used in most deep learning

    **Common issues:**
    - Learning rate too high → diverges
    - Learning rate too low → converges too slowly
    - Use learning rate scheduling or adaptive optimisers (Adam, RMSProp) to address this

---

### Q19: What is the difference between L1 and L2 loss functions?

??? "Show answer"
    These refer to loss functions, not regularisation (though the names overlap).

    **L1 loss (Mean Absolute Error):**
    - Loss = Σ |y_true - y_pred|
    - Robust to outliers — large errors are not penalised quadratically
    - The gradient is constant, which can cause oscillation near the minimum

    **L2 loss (Mean Squared Error):**
    - Loss = Σ (y_true - y_pred)²
    - Penalises large errors heavily — sensitive to outliers
    - Smooth gradient → easier to optimise
    - Most commonly used default for regression

    **When to use L1:** when your data has significant outliers and you want the model to be robust to them.

---

### Q20: What happens when you have highly correlated features (multicollinearity)?

??? "Show answer"
    In linear models, multicollinearity causes:
    - Unstable, high-variance coefficient estimates — small changes in data cause large swings
    - Coefficients become unreliable for interpretation
    - Standard errors inflate, making significance tests invalid

    **Detection:** Variance Inflation Factor (VIF). VIF > 5–10 signals problematic multicollinearity.

    **Remedies:**
    - Remove one of the correlated features
    - Use Ridge regression (L2 regularisation) — it handles multicollinearity well
    - Use PCA to create orthogonal features
    - For tree-based models: multicollinearity is less problematic (trees split one feature at a time)
