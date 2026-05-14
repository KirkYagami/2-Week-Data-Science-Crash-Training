# Machine Learning Cheat Sheet

A dense reference for practitioners. Each entry covers when to reach for a tool, what to watch out for, and a runnable code snippet.

---

## Core Concepts

### Bias-Variance Tradeoff

High bias = model too simple, underfits training data, high training error.
High variance = model memorizes training data, performs poorly on unseen data, large gap between train and test error.
The goal is to find the complexity level where both are low enough that generalization is good.

```python
from sklearn.linear_model import LinearRegression
from sklearn.tree import DecisionTreeRegressor
from sklearn.model_selection import cross_val_score
import numpy as np

# High bias: linear model on nonlinear data
linear_model = LinearRegression()
linear_scores = cross_val_score(linear_model, X_train, y_train, cv=5, scoring='r2')
print(f"Linear CV R²: {linear_scores.mean():.3f} ± {linear_scores.std():.3f}")

# High variance: unconstrained tree overfits
deep_tree = DecisionTreeRegressor(max_depth=None, random_state=42)
tree_scores = cross_val_score(deep_tree, X_train, y_train, cv=5, scoring='r2')
print(f"Deep Tree CV R²: {tree_scores.mean():.3f} ± {tree_scores.std():.3f}")

# Balanced: constrained tree
pruned_tree = DecisionTreeRegressor(max_depth=5, min_samples_leaf=10, random_state=42)
pruned_scores = cross_val_score(pruned_tree, X_train, y_train, cv=5, scoring='r2')
print(f"Pruned Tree CV R²: {pruned_scores.mean():.3f} ± {pruned_scores.std():.3f}")
```

### Overfitting / Underfitting Signals

Watch the gap between training score and validation score.

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

train_acc = accuracy_score(y_train, model.predict(X_train))
val_acc   = accuracy_score(y_val,   model.predict(X_val))

print(f"Train accuracy: {train_acc:.3f}")
print(f"Val   accuracy: {val_acc:.3f}")
print(f"Gap:            {train_acc - val_acc:.3f}")
# Output:
# Train accuracy: 0.998   ← suspiciously perfect
# Val   accuracy: 0.834   ← large gap → overfitting
# Gap:            0.164
```

> [!warning]
> A near-perfect training score with a large train/val gap is the clearest signal of overfitting. Reduce model complexity, add regularization, or get more data.

### The scikit-learn fit/predict API Pattern

Every estimator in scikit-learn follows the same interface. Learn this once and it applies everywhere.

```python
from sklearn.linear_model import LogisticRegression

# 1. Instantiate with hyperparameters
model = LogisticRegression(C=1.0, max_iter=1000, random_state=42)

# 2. Fit on training data
model.fit(X_train, y_train)

# 3. Predict
y_pred       = model.predict(X_test)           # class labels
y_pred_proba = model.predict_proba(X_test)     # probability per class

# 4. Score
score = model.score(X_test, y_test)            # default metric for the estimator type
print(f"Accuracy: {score:.3f}")
```

---

## Data Splitting

### train_test_split with stratify

Use `stratify` whenever the target is categorical and you want each split to reflect the class distribution of the full dataset. Skipping it on imbalanced data leads to splits with missing or underrepresented classes.

```python
from sklearn.model_selection import train_test_split
import pandas as pd

X = df.drop(columns=['churn'])
y = df['churn']

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    stratify=y,        # preserves class ratio in both splits
    random_state=42
)

print(y_train.value_counts(normalize=True))
# Output:
# 0    0.855
# 1    0.145
print(y_test.value_counts(normalize=True))
# Output:
# 0    0.855
# 1    0.145
```

### cross_val_score

Use when your dataset is too small to hold out a fixed validation set. Gives a less noisy estimate of generalization performance than a single train/val split.

```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
cv_scores = cross_val_score(model, X, y, cv=5, scoring='f1_weighted', n_jobs=-1)

print(f"F1 per fold: {cv_scores.round(3)}")
print(f"Mean: {cv_scores.mean():.3f}  Std: {cv_scores.std():.3f}")
# Output:
# F1 per fold: [0.821 0.834 0.818 0.841 0.829]
# Mean: 0.829  Std: 0.008
```

### KFold and StratifiedKFold

`KFold` for regression. `StratifiedKFold` for classification — ensures each fold has proportional class representation.

```python
from sklearn.model_selection import KFold, StratifiedKFold, cross_validate
from sklearn.linear_model import Ridge

# Regression — KFold
kf = KFold(n_splits=5, shuffle=True, random_state=42)
results = cross_validate(Ridge(alpha=1.0), X, y, cv=kf,
                         scoring=['r2', 'neg_mean_absolute_error'])
print(f"R²: {results['test_r2'].mean():.3f}")

# Classification — StratifiedKFold
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
from sklearn.linear_model import LogisticRegression
clf_results = cross_validate(LogisticRegression(max_iter=1000), X_cls, y_cls,
                              cv=skf, scoring='roc_auc')
print(f"ROC-AUC: {clf_results['test_score'].mean():.3f}")
```

---

## Preprocessing

### StandardScaler

Use when the algorithm is sensitive to feature scale and the data is roughly Gaussian (e.g., Logistic Regression, SVM, KNN, linear models). Centers to mean=0, scales to std=1.

> [!warning]
> Always fit the scaler on training data only. Fitting on the full dataset leaks test statistics into training.

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit + transform on train
X_test_scaled  = scaler.transform(X_test)         # transform only on test

print(f"Mean (train): {X_train_scaled.mean(axis=0).round(4)}")   # ≈ 0
print(f"Std  (train): {X_train_scaled.std(axis=0).round(4)}")    # ≈ 1
```

### MinMaxScaler

Scales features to a fixed range, default [0, 1]. Use when you need bounded output (neural networks, image pixel values). Sensitive to outliers — one extreme point compresses everything else.

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler(feature_range=(0, 1))
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)

print(f"Min: {X_train_scaled.min(axis=0)}")   # Output: [0. 0. 0. ...]
print(f"Max: {X_train_scaled.max(axis=0)}")   # Output: [1. 1. 1. ...]
```

### RobustScaler

Uses median and IQR instead of mean and std. The right choice when your data has significant outliers that would distort StandardScaler.

```python
from sklearn.preprocessing import RobustScaler

scaler = RobustScaler(quantile_range=(25.0, 75.0))
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)
```

### LabelEncoder vs OrdinalEncoder

`LabelEncoder` encodes the target column (1D). `OrdinalEncoder` encodes feature columns (2D) with optional category ordering.

```python
from sklearn.preprocessing import LabelEncoder, OrdinalEncoder
import pandas as pd

# LabelEncoder — for target y only
le = LabelEncoder()
y_encoded = le.fit_transform(y_raw)
print(le.classes_)           # Output: ['High' 'Low' 'Medium']
print(le.transform(['Low'])) # Output: [1]

# OrdinalEncoder — for ordered features
size_order = [['Small', 'Medium', 'Large', 'XLarge']]
oe = OrdinalEncoder(categories=size_order)
X[['shirt_size']] = oe.fit_transform(X[['shirt_size']])
```

### OneHotEncoder

Converts nominal (unordered) categorical variables into binary columns. Use when the algorithm cannot interpret integer-encoded categories (linear models, SVMs). Tree models can handle label-encoded categoricals directly.

```python
from sklearn.preprocessing import OneHotEncoder
import pandas as pd

ohe = OneHotEncoder(sparse_output=False, handle_unknown='ignore', drop='first')
encoded = ohe.fit_transform(X[['city', 'payment_method']])

feature_names = ohe.get_feature_names_out(['city', 'payment_method'])
encoded_df = pd.DataFrame(encoded, columns=feature_names)
print(encoded_df.head())
# Output:
#    city_Mumbai  city_Pune  payment_method_UPI  ...
# 0          1.0        0.0                 1.0
```

---

## Regression Algorithms

### LinearRegression

Baseline for any regression task. Use when the relationship between features and target is approximately linear. No hyperparameters to tune — if it underperforms, that's signal to try a more complex model.

```python
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score

model = LinearRegression(fit_intercept=True)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print(f"Coefficients: {model.coef_}")
print(f"Intercept:    {model.intercept_:.3f}")
print(f"R²:           {r2_score(y_test, y_pred):.3f}")
```

### Ridge (L2 Regularization)

Linear regression with L2 penalty on coefficient magnitude. Use when you have many correlated features — Ridge shrinks coefficients toward zero but rarely to exactly zero. `alpha` controls regularization strength; higher alpha = stronger shrinkage.

```python
from sklearn.linear_model import Ridge
from sklearn.model_selection import cross_val_score

# Key hyperparameter: alpha
for alpha in [0.01, 0.1, 1.0, 10.0, 100.0]:
    model = Ridge(alpha=alpha)
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='r2')
    print(f"alpha={alpha:6.2f}  CV R²={scores.mean():.3f}")

best_model = Ridge(alpha=1.0)
best_model.fit(X_train, y_train)
```

### Lasso (L1 Regularization)

Linear regression with L1 penalty. Drives some coefficients to exactly zero — performs built-in feature selection. Prefer over Ridge when you believe only a subset of features are relevant.

```python
from sklearn.linear_model import Lasso
import pandas as pd

model = Lasso(alpha=0.1, max_iter=10000)
model.fit(X_train, y_train)

coef_series = pd.Series(model.coef_, index=feature_names)
selected = coef_series[coef_series != 0]
print(f"Features selected by Lasso: {len(selected)} of {len(feature_names)}")
print(selected.sort_values(key=abs, ascending=False))
```

### ElasticNet

Combines L1 and L2 penalties. Use when you want Lasso's feature selection but with more stability when features are correlated (pure Lasso arbitrarily picks one of a correlated group). `l1_ratio=1` is Lasso, `l1_ratio=0` is Ridge.

```python
from sklearn.linear_model import ElasticNet

model = ElasticNet(
    alpha=0.1,        # overall regularization strength
    l1_ratio=0.5,     # 50% L1, 50% L2
    max_iter=10000,
    random_state=42
)
model.fit(X_train, y_train)
print(f"R²: {model.score(X_test, y_test):.3f}")
```

### DecisionTreeRegressor

Non-parametric, captures nonlinear relationships and interactions. Prone to overfitting without depth limits. Use as a baseline for tree-based approaches before moving to ensembles.

```python
from sklearn.tree import DecisionTreeRegressor

model = DecisionTreeRegressor(
    max_depth=5,           # primary regularizer — start here
    min_samples_leaf=10,   # require at least 10 samples per leaf
    min_samples_split=20,  # require at least 20 samples to split a node
    random_state=42
)
model.fit(X_train, y_train)
print(f"Train R²: {model.score(X_train, y_train):.3f}")
print(f"Test  R²: {model.score(X_test,  y_test):.3f}")
```

### RandomForestRegressor

Ensemble of decision trees trained on bootstrap samples with random feature subsets. Reduces variance substantially vs. a single tree. A reliable default for tabular regression.

```python
from sklearn.ensemble import RandomForestRegressor

model = RandomForestRegressor(
    n_estimators=200,       # more trees = lower variance, diminishing returns after ~200
    max_depth=None,         # let trees grow fully; bootstrap + feature randomness controls variance
    max_features='sqrt',    # features considered at each split — 'sqrt' is standard
    min_samples_leaf=5,
    n_jobs=-1,
    random_state=42
)
model.fit(X_train, y_train)
print(f"OOB R²: {model.oob_score_:.3f}")   # requires oob_score=True
print(f"Test R²: {model.score(X_test, y_test):.3f}")
```

---

## Classification Algorithms

### LogisticRegression

Use for binary and multiclass classification when you need probability outputs and model interpretability. Despite the name, it is a classifier. Requires scaled features.

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(
    C=1.0,               # inverse of regularization strength; smaller C = more regularization
    penalty='l2',        # 'l1', 'l2', 'elasticnet', or None
    solver='lbfgs',      # 'saga' for l1/elasticnet; 'lbfgs' for l2
    max_iter=1000,
    class_weight='balanced',  # useful for imbalanced targets
    random_state=42
)
model.fit(X_train_scaled, y_train)
print(f"Accuracy: {model.score(X_test_scaled, y_test):.3f}")
print(f"Coefficients shape: {model.coef_.shape}")
```

### KNeighborsClassifier

Non-parametric: classifies by majority vote among k nearest neighbors. No training step — prediction is expensive on large datasets. Features must be scaled. Works well with small datasets and low-dimensional feature spaces.

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import cross_val_score

# Find best k
for k in [3, 5, 7, 11, 15]:
    knn = KNeighborsClassifier(n_neighbors=k, metric='euclidean', weights='uniform')
    scores = cross_val_score(knn, X_train_scaled, y_train, cv=5, scoring='accuracy')
    print(f"k={k:2d}  CV Accuracy={scores.mean():.3f}")

best_knn = KNeighborsClassifier(n_neighbors=7, weights='distance')
best_knn.fit(X_train_scaled, y_train)
```

### DecisionTreeClassifier

Interpretable, handles mixed feature types, no scaling needed. Prone to overfitting. Use when explainability is required or as a component in an ensemble.

```python
from sklearn.tree import DecisionTreeClassifier, export_text

model = DecisionTreeClassifier(
    max_depth=4,
    min_samples_leaf=15,
    criterion='gini',         # 'gini' or 'entropy'
    class_weight='balanced',
    random_state=42
)
model.fit(X_train, y_train)

# Human-readable tree
print(export_text(model, feature_names=list(feature_names), max_depth=3))
```

### RandomForestClassifier

Strong general-purpose baseline. Handles high-dimensional data, implicit feature selection, and is robust to outliers. No scaling required.

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    n_estimators=200,
    max_depth=None,
    max_features='sqrt',
    class_weight='balanced',    # addresses class imbalance
    oob_score=True,
    n_jobs=-1,
    random_state=42
)
model.fit(X_train, y_train)

print(f"OOB accuracy: {model.oob_score_:.3f}")
print(f"Test accuracy: {model.score(X_test, y_test):.3f}")
```

### GradientBoostingClassifier

Builds trees sequentially, each correcting the errors of the previous. Often the top performer on structured/tabular data. Slower to train than Random Forest, more hyperparameters to tune.

```python
from sklearn.ensemble import GradientBoostingClassifier

model = GradientBoostingClassifier(
    n_estimators=200,        # number of boosting stages
    learning_rate=0.05,      # shrinkage — lower rate needs more trees
    max_depth=4,             # shallow trees work best for boosting
    subsample=0.8,           # stochastic boosting reduces variance
    min_samples_leaf=10,
    random_state=42
)
model.fit(X_train, y_train)
print(f"Test accuracy: {model.score(X_test, y_test):.3f}")
```

> [!tip]
> For large datasets, use `HistGradientBoostingClassifier` from scikit-learn or XGBoost/LightGBM — they are 10–100x faster than `GradientBoostingClassifier`.

### SVC (Support Vector Classifier)

Maximizes the margin between classes. Effective in high-dimensional spaces and when classes are separable. Requires feature scaling. Slow on large datasets (>50k rows).

```python
from sklearn.svm import SVC

model = SVC(
    C=1.0,               # regularization — larger C = smaller margin, fewer misclassifications
    kernel='rbf',        # 'linear', 'poly', 'rbf', 'sigmoid'
    gamma='scale',       # 'scale' = 1/(n_features * X.var()), 'auto' = 1/n_features
    probability=True,    # enables predict_proba — adds overhead
    class_weight='balanced',
    random_state=42
)
model.fit(X_train_scaled, y_train)
y_pred_proba = model.predict_proba(X_test_scaled)[:, 1]
```

---

## Clustering

### KMeans with Elbow Method

Partitions data into k spherical clusters by minimizing inertia (sum of squared distances to cluster centroids). Assumes clusters are roughly equal-sized and convex. Sensitive to outliers and initialization.

Use elbow method or silhouette score to choose k — there is no ground truth label to validate against.

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
import numpy as np
import matplotlib.pyplot as plt

scaler = StandardScaler()
X_scaled = scaler.fit_transform(customer_features)

# Elbow method
inertias = []
k_range = range(2, 11)

for k in k_range:
    km = KMeans(n_clusters=k, init='k-means++', n_init=10, random_state=42)
    km.fit(X_scaled)
    inertias.append(km.inertia_)

plt.plot(k_range, inertias, marker='o')
plt.xlabel('Number of clusters k')
plt.ylabel('Inertia')
plt.title('Elbow Method')
plt.show()

# Fit with chosen k
best_km = KMeans(n_clusters=4, init='k-means++', n_init=10, random_state=42)
cluster_labels = best_km.fit_predict(X_scaled)

customer_features['segment'] = cluster_labels
print(customer_features.groupby('segment').mean())
```

### DBSCAN

Density-Based Spatial Clustering of Applications with Noise. Discovers clusters of arbitrary shape and automatically labels outliers as noise (label = -1). Does not require specifying k.

Use when: clusters are non-spherical, you have significant noise/outliers, you don't know the number of clusters.

```python
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(location_data)

db = DBSCAN(
    eps=0.5,              # maximum distance between two points to be neighbors
    min_samples=5,        # minimum points to form a core point
    metric='euclidean'
)
labels = db.fit_predict(X_scaled)

n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise    = (labels == -1).sum()

print(f"Clusters found: {n_clusters}")
print(f"Noise points:   {n_noise} ({100 * n_noise / len(labels):.1f}%)")
```

> [!tip]
> Use `NearestNeighbors` to find a good `eps` value: sort the distances to the kth neighbor and look for the elbow in that plot.

### AgglomerativeClustering

Hierarchical bottom-up clustering. Starts with each point as its own cluster and merges the most similar pair at each step. No assumption about cluster shape. Computationally expensive for large datasets (O(n² log n)).

Use when: you need a hierarchy of clusters, or cluster count is uncertain and you want to inspect the dendrogram.

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage
import matplotlib.pyplot as plt

# Dendrogram for choosing n_clusters
linkage_matrix = linkage(X_scaled, method='ward')
plt.figure(figsize=(10, 5))
dendrogram(linkage_matrix, truncate_mode='level', p=5)
plt.title('Hierarchical Clustering Dendrogram')
plt.show()

# Fit with chosen number of clusters
model = AgglomerativeClustering(
    n_clusters=4,
    linkage='ward',       # 'ward', 'complete', 'average', 'single'
    metric='euclidean'
)
labels = model.fit_predict(X_scaled)
```

---

## Regression Metrics

### MAE, MSE, RMSE, R²

| Metric | Formula | Interpretation |
|---|---|---|
| MAE | mean(|y - ŷ|) | Average absolute error, same units as target |
| MSE | mean((y - ŷ)²) | Penalizes large errors more heavily |
| RMSE | sqrt(MSE) | Same units as target, emphasizes large errors |
| R² | 1 - SS_res/SS_tot | Proportion of variance explained; 1.0 is perfect |

```python
from sklearn.metrics import (
    mean_absolute_error,
    mean_squared_error,
    r2_score
)
import numpy as np

y_test = [100, 200, 150, 300, 250]
y_pred = [110, 195, 160, 280, 260]

mae  = mean_absolute_error(y_test, y_pred)
mse  = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2   = r2_score(y_test, y_pred)

print(f"MAE:  {mae:.2f}")    # Output: MAE:  12.00
print(f"MSE:  {mse:.2f}")    # Output: MSE:  170.00
print(f"RMSE: {rmse:.2f}")   # Output: RMSE: 13.04
print(f"R²:   {r2:.3f}")     # Output: R²:   0.964
```

### Adjusted R²

Penalizes adding features that do not improve the model. Use instead of R² when comparing models with different numbers of features.

```python
import numpy as np

def adjusted_r2(r2, n_samples, n_features):
    """
    r2         : R² score from sklearn
    n_samples  : number of rows in the test set
    n_features : number of features used by the model
    """
    return 1 - (1 - r2) * (n_samples - 1) / (n_samples - n_features - 1)

r2 = r2_score(y_test, y_pred)
adj_r2 = adjusted_r2(r2, n_samples=len(y_test), n_features=X_test.shape[1])
print(f"R²:          {r2:.4f}")
print(f"Adjusted R²: {adj_r2:.4f}")
```

---

## Classification Metrics

### Accuracy, Precision, Recall, F1

Use accuracy when classes are balanced. For imbalanced data, prefer precision, recall, and F1. Choose precision when false positives are expensive (spam filter). Choose recall when false negatives are expensive (cancer screening).

```python
from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score,
    classification_report,
    confusion_matrix,
    roc_auc_score
)

y_test = [0, 0, 1, 1, 0, 1, 0, 1, 1, 0]
y_pred = [0, 1, 1, 0, 0, 1, 0, 1, 0, 0]

print(f"Accuracy:  {accuracy_score(y_test, y_pred):.3f}")
print(f"Precision: {precision_score(y_test, y_pred):.3f}")   # TP / (TP + FP)
print(f"Recall:    {recall_score(y_test, y_pred):.3f}")      # TP / (TP + FN)
print(f"F1:        {f1_score(y_test, y_pred):.3f}")          # harmonic mean of P and R
# Output:
# Accuracy:  0.700
# Precision: 0.750
# Recall:    0.600
# F1:        0.667
```

### confusion_matrix and classification_report

```python
from sklearn.metrics import confusion_matrix, classification_report
import pandas as pd

cm = confusion_matrix(y_test, y_pred)
cm_df = pd.DataFrame(
    cm,
    index=['Actual Negative', 'Actual Positive'],
    columns=['Predicted Negative', 'Predicted Positive']
)
print(cm_df)
# Output:
#                   Predicted Negative  Predicted Positive
# Actual Negative                   4                   1
# Actual Positive                   2                   3

print(classification_report(y_test, y_pred, target_names=['No Churn', 'Churn']))
```

### ROC-AUC

Measures the model's ability to discriminate between classes across all thresholds. AUC = 0.5 means no discrimination (random). AUC = 1.0 means perfect. Use `roc_auc_score` with probability predictions, not binary predictions.

```python
from sklearn.metrics import roc_auc_score, roc_curve
import matplotlib.pyplot as plt

y_pred_proba = model.predict_proba(X_test)[:, 1]  # probability of positive class
auc = roc_auc_score(y_test, y_pred_proba)
print(f"ROC-AUC: {auc:.3f}")

fpr, tpr, thresholds = roc_curve(y_test, y_pred_proba)

plt.plot(fpr, tpr, label=f'ROC curve (AUC = {auc:.3f})')
plt.plot([0, 1], [0, 1], linestyle='--', color='gray', label='Random classifier')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.legend()
plt.show()
```

---

## Hyperparameter Tuning

### GridSearchCV

Exhaustively searches all combinations of specified hyperparameters. Use when the search space is small and you need reproducible, thorough results.

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

param_grid = {
    'n_estimators':    [100, 200, 300],
    'max_depth':       [None, 5, 10],
    'min_samples_leaf':[1, 5, 10],
    'max_features':    ['sqrt', 'log2']
}

grid_search = GridSearchCV(
    estimator=RandomForestClassifier(random_state=42),
    param_grid=param_grid,
    cv=5,
    scoring='f1_weighted',
    n_jobs=-1,
    verbose=1,
    refit=True           # refit best model on full training set
)
grid_search.fit(X_train, y_train)

print(f"Best params: {grid_search.best_params_}")
print(f"Best CV F1:  {grid_search.best_score_:.3f}")

best_model = grid_search.best_estimator_
print(f"Test F1: {f1_score(y_test, best_model.predict(X_test), average='weighted'):.3f}")
```

### RandomizedSearchCV

Samples a fixed number of hyperparameter combinations from distributions. Use when the search space is large — far more efficient than GridSearch and often finds equally good solutions.

```python
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import GradientBoostingClassifier
from scipy.stats import randint, uniform

param_distributions = {
    'n_estimators':    randint(50, 500),
    'learning_rate':   uniform(0.01, 0.3),
    'max_depth':       randint(2, 8),
    'subsample':       uniform(0.6, 0.4),      # samples from [0.6, 1.0]
    'min_samples_leaf':randint(5, 50)
}

random_search = RandomizedSearchCV(
    estimator=GradientBoostingClassifier(random_state=42),
    param_distributions=param_distributions,
    n_iter=50,            # number of combinations to sample
    cv=5,
    scoring='roc_auc',
    n_jobs=-1,
    random_state=42,
    verbose=1
)
random_search.fit(X_train, y_train)

print(f"Best params: {random_search.best_params_}")
print(f"Best CV AUC: {random_search.best_score_:.3f}")
```

---

## Pipelines

### Pipeline — canonical pattern

A `Pipeline` chains preprocessing and modeling steps so that `fit` and `transform` are called consistently, preventing data leakage from the test set during cross-validation.

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

pipeline = Pipeline(steps=[
    ('scaler', StandardScaler()),
    ('classifier', LogisticRegression(C=1.0, max_iter=1000, random_state=42))
])

# Entire pipeline participates in cross-validation correctly
cv_scores = cross_val_score(pipeline, X, y, cv=5, scoring='roc_auc')
print(f"CV ROC-AUC: {cv_scores.mean():.3f} ± {cv_scores.std():.3f}")

# Train final model
pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)
```

### ColumnTransformer — mixed feature types

Apply different preprocessing to numeric and categorical columns in one step.

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier
import pandas as pd

numeric_features = ['age', 'annual_income', 'account_balance']
categorical_features = ['city', 'product_category', 'payment_method']

preprocessor = ColumnTransformer(transformers=[
    ('num', StandardScaler(), numeric_features),
    ('cat', OneHotEncoder(handle_unknown='ignore', sparse_output=False), categorical_features)
])

pipeline = Pipeline(steps=[
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(n_estimators=200, random_state=42))
])

pipeline.fit(X_train, y_train)
print(f"Test accuracy: {pipeline.score(X_test, y_test):.3f}")

# Hyperparameter tuning still works end-to-end
from sklearn.model_selection import GridSearchCV
param_grid = {
    'classifier__n_estimators': [100, 200],
    'classifier__max_depth':    [None, 5]
}
gs = GridSearchCV(pipeline, param_grid, cv=5, scoring='accuracy', n_jobs=-1)
gs.fit(X_train, y_train)
```

### make_pipeline — shorthand

Identical to `Pipeline` but infers step names from class names automatically.

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import RobustScaler
from sklearn.svm import SVC

pipeline = make_pipeline(
    RobustScaler(),
    SVC(C=1.0, kernel='rbf', probability=True, random_state=42)
)
pipeline.fit(X_train, y_train)
print(pipeline.steps)
# Output: [('robustscaler', RobustScaler()), ('svc', SVC(probability=True, ...))]
```

---

## Feature Importance

### Impurity-Based Importance (.feature_importances_)

Available on all tree-based models. Fast to compute. Can be misleading when features have different cardinalities or when features are correlated — high-cardinality features (like IDs) are favored.

```python
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=200, random_state=42)
model.fit(X_train, y_train)

importance_df = pd.DataFrame({
    'feature':   feature_names,
    'importance': model.feature_importances_
}).sort_values('importance', ascending=False)

print(importance_df.head(10))
importance_df.head(15).plot.barh(x='feature', y='importance', figsize=(8, 6))
plt.title('Feature Importances (Impurity-Based)')
plt.gca().invert_yaxis()
plt.show()
```

### permutation_importance

Model-agnostic. Measures the drop in score when a feature's values are randomly shuffled. More reliable than impurity-based importance, especially for correlated features.

```python
from sklearn.inspection import permutation_importance
import pandas as pd

result = permutation_importance(
    model, X_test, y_test,
    n_repeats=10,         # number of shuffles per feature
    scoring='roc_auc',
    random_state=42,
    n_jobs=-1
)

perm_df = pd.DataFrame({
    'feature':  feature_names,
    'importance_mean': result.importances_mean,
    'importance_std':  result.importances_std
}).sort_values('importance_mean', ascending=False)

print(perm_df.head(10))
```

### SHAP (brief)

SHAP (SHapley Additive exPlanations) provides consistent, locally-accurate feature attributions for any model. The gold standard for model explainability. Requires `pip install shap`.

```python
import shap

explainer = shap.TreeExplainer(model)              # fast path for tree models
shap_values = explainer.shap_values(X_test)

# Global summary plot
shap.summary_plot(shap_values[1], X_test, feature_names=feature_names)

# Single prediction explanation
shap.waterfall_plot(
    shap.Explanation(values=shap_values[1][0],
                     base_values=explainer.expected_value[1],
                     data=X_test.iloc[0],
                     feature_names=feature_names)
)
```

---

## Imbalanced Data

### class_weight='balanced'

The simplest fix. Automatically adjusts sample weights inversely proportional to class frequency. Supported by LogisticRegression, SVC, DecisionTree, RandomForest, and GradientBoosting.

```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report

# Without balancing
unweighted = LogisticRegression(max_iter=1000, random_state=42)
unweighted.fit(X_train_scaled, y_train)
print(classification_report(y_test, unweighted.predict(X_test_scaled)))

# With balancing
balanced = LogisticRegression(class_weight='balanced', max_iter=1000, random_state=42)
balanced.fit(X_train_scaled, y_train)
print(classification_report(y_test, balanced.predict(X_test_scaled)))
# Expect higher recall for minority class, lower precision
```

### SMOTE (Synthetic Minority Oversampling)

Generates synthetic minority class samples by interpolating between existing minority examples. Use when `class_weight` is insufficient and you have enough minority class samples to interpolate (>= 6). Requires `pip install imbalanced-learn`.

> [!warning]
> Apply SMOTE only to training data, never to validation or test data. Doing otherwise inflates performance metrics.

```python
from imblearn.over_sampling import SMOTE
from imblearn.pipeline import Pipeline as ImbPipeline
from sklearn.ensemble import RandomForestClassifier

# SMOTE must be inside a pipeline to avoid leaking into validation folds
pipeline = ImbPipeline(steps=[
    ('smote', SMOTE(sampling_strategy='auto', k_neighbors=5, random_state=42)),
    ('classifier', RandomForestClassifier(n_estimators=200, random_state=42))
])

from sklearn.model_selection import cross_val_score
scores = cross_val_score(pipeline, X_train, y_train, cv=5, scoring='f1')
print(f"CV F1 with SMOTE: {scores.mean():.3f} ± {scores.std():.3f}")
```

### Threshold Adjustment

The default decision threshold for `predict()` is 0.5. Lowering the threshold increases recall at the cost of precision. Use when the cost of false negatives is higher than false positives (e.g., fraud, churn, medical diagnosis).

```python
from sklearn.metrics import precision_recall_curve, f1_score
import numpy as np

model.fit(X_train_scaled, y_train)
y_pred_proba = model.predict_proba(X_test_scaled)[:, 1]

# Find threshold that maximizes F1
precisions, recalls, thresholds = precision_recall_curve(y_test, y_pred_proba)
f1_scores = 2 * precisions * recalls / (precisions + recalls + 1e-9)
best_threshold = thresholds[np.argmax(f1_scores)]

print(f"Default threshold (0.5) F1: {f1_score(y_test, y_pred_proba >= 0.5):.3f}")
print(f"Optimal threshold ({best_threshold:.2f}) F1: {f1_score(y_test, y_pred_proba >= best_threshold):.3f}")

y_pred_adjusted = (y_pred_proba >= best_threshold).astype(int)
```

---

## Saving Models

### joblib — preferred for scikit-learn

`joblib` serializes numpy arrays efficiently, making it faster than `pickle` for models that contain large arrays (most scikit-learn models). Use this by default.

```python
import joblib

# Save
joblib.dump(model, 'churn_model_v1.joblib')
print("Model saved.")

# Save entire pipeline (always preferred — scaler + model together)
joblib.dump(pipeline, 'churn_pipeline_v1.joblib')

# Load
loaded_model = joblib.load('churn_model_v1.joblib')
y_pred = loaded_model.predict(X_test)
print(f"Predictions from loaded model: {y_pred[:5]}")
# Output: Predictions from loaded model: [0 1 0 0 1]
```

### pickle — standard library fallback

Use when you cannot install joblib, or when interoperability with non-scikit-learn code is required. Slower for large numpy arrays. Format is Python-version-specific — document the Python version alongside the file.

```python
import pickle

# Save
with open('churn_model_v1.pkl', 'wb') as f:
    pickle.dump(model, f)

# Load
with open('churn_model_v1.pkl', 'rb') as f:
    loaded_model = pickle.load(f)

y_pred = loaded_model.predict(X_test)
```

> [!warning]
> Never load a pickle file from an untrusted source. Pickle can execute arbitrary code on deserialization. For model serving in production, prefer ONNX, PMML, or a framework-native format.

> [!tip]
> Always save the model and preprocessing pipeline together as a single object. If you save them separately, you risk applying the wrong scaler to incoming data at inference time.
