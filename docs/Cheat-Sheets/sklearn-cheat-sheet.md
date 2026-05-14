# Scikit-learn API Cheat Sheet

Every sklearn object follows the same contract. Once you understand the pattern, you can pick up any estimator without reading its docs from scratch. This sheet is about the API — how sklearn is designed and why — not about algorithm theory.

---

## The sklearn API Contract

sklearn enforces one rule: every estimator learns from data in `fit()`, then produces outputs in `predict()`, `transform()`, or `score()`. Parameters go in `__init__`. Learned attributes come out with a trailing underscore. That's the whole contract.

### fit()

`fit()` learns parameters from training data and stores them on the object. It always returns `self` so you can chain calls.

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

X_train = np.array([[1.0, 200.0], [2.0, 300.0], [3.0, 400.0]])

scaler = StandardScaler()
scaler.fit(X_train)

print(scaler.mean_)    # Output: [2.0, 300.0]
print(scaler.scale_)   # Output: [0.816..., 81.6...]
```

The trailing underscore on `mean_` and `scale_` signals these are learned attributes — only available after `fit()`. Accessing them before fitting raises `NotFittedError`.

### predict()

`predict()` applies a fitted model to new data. For classifiers it returns class labels; for regressors it returns continuous values.

```python
from sklearn.linear_model import LogisticRegression

X_train = [[0, 0], [1, 1], [2, 2], [3, 1]]
y_train = [0, 0, 1, 1]
X_new = [[1.5, 1.5], [2.5, 0.5]]

clf = LogisticRegression()
clf.fit(X_train, y_train)
clf.predict(X_new)
# Output: array([0, 1])
```

Never call `predict()` on training data to evaluate performance — use cross-validation instead.

### transform()

`transform()` applies a learned transformation (scaling, encoding, dimensionality reduction). Used by preprocessors and feature extractors, not classifiers or regressors.

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

X_train = np.array([[1.0, 200.0], [2.0, 300.0], [3.0, 400.0]])
X_test  = np.array([[4.0, 500.0]])

scaler = StandardScaler()
scaler.fit(X_train)

X_train_scaled = scaler.transform(X_train)
X_test_scaled  = scaler.transform(X_test)   # uses training mean/scale, not test stats

print(X_test_scaled)
# Output: [[ 2.449...,  2.449...]]
```

The critical rule: always `fit` on training data, then `transform` both train and test. Never `fit` on test data — that leaks information.

### fit_transform()

`fit_transform()` is a convenience shortcut for `fit(X).transform(X)`. Use it only on training data.

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

X_train = np.array([[1.0, 200.0], [2.0, 300.0], [3.0, 400.0]])

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
# Equivalent to: scaler.fit(X_train); scaler.transform(X_train)
```

Do not call `fit_transform()` on test data. That fits new parameters from the test set, which is data leakage.

### score()

`score()` returns a default metric for the estimator type: R² for regressors, accuracy for classifiers, silhouette-related for clustering. Convenient for quick checks, but use explicit metrics for real evaluation.

```python
from sklearn.linear_model import LinearRegression
import numpy as np

X = np.array([[1], [2], [3], [4], [5]])
y = np.array([2.1, 4.0, 5.9, 8.2, 9.8])

reg = LinearRegression().fit(X, y)
print(reg.score(X, y))
# Output: 0.998...  (R² on training data — optimistic, use CV for real evaluation)
```

---

## Estimator Parameters

### get_params() and set_params()

Every estimator stores its `__init__` arguments as attributes. `get_params()` returns them as a dict; `set_params()` changes them in place. This is how GridSearchCV modifies estimators during search.

```python
from sklearn.svm import SVC

clf = SVC(C=1.0, kernel='rbf', gamma='scale')
print(clf.get_params())
# Output: {'C': 1.0, 'break_ties': False, 'cache_size': 200, 'class_weight': None,
#          'coef0': 0.0, 'decision_function_shape': 'ovr', 'degree': 3,
#          'gamma': 'scale', 'kernel': 'rbf', 'max_iter': -1, 'probability': False,
#          'random_state': None, 'shrinking': True, 'tol': 0.001, 'verbose': False}

clf.set_params(C=10.0, kernel='linear')
print(clf.C, clf.kernel)
# Output: 10.0 linear
```

### clone()

`clone()` creates a new unfitted estimator with the same parameters. Use it when you need a fresh copy — for example, in a manual cross-validation loop — without carrying over any fitted state.

```python
from sklearn.base import clone
from sklearn.ensemble import RandomForestClassifier

original = RandomForestClassifier(n_estimators=100, random_state=42)
original.fit([[1, 2], [3, 4]], [0, 1])

fresh = clone(original)
# fresh has the same params but is NOT fitted — no trees_, no classes_
print(hasattr(fresh, 'classes_'))
# Output: False
```

---

## Preprocessing

### StandardScaler

Removes mean, scales to unit variance. Assumes features are roughly Gaussian. Sensitive to outliers because it uses mean and standard deviation.

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

X = np.array([[1.0, 10.0], [2.0, 20.0], [3.0, 30.0], [100.0, 40.0]])

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
print(X_scaled.mean(axis=0))   # Output: [0. 0.]
print(X_scaled.std(axis=0))    # Output: [1. 1.]
```

### MinMaxScaler

Scales features to a fixed range, default [0, 1]. Preserves zero values. Sensitive to outliers — one extreme value compresses everything else.

```python
from sklearn.preprocessing import MinMaxScaler
import numpy as np

X = np.array([[1.0, 200.0], [2.0, 300.0], [3.0, 400.0]])

scaler = MinMaxScaler(feature_range=(0, 1))
X_scaled = scaler.fit_transform(X)
print(X_scaled)
# Output: [[0.  0. ]
#          [0.5 0.5]
#          [1.  1. ]]

# Inverse transform: recover original values
print(scaler.inverse_transform([[0.5, 0.5]]))
# Output: [[2. 300.]]
```

### RobustScaler

Uses median and interquartile range instead of mean and std. Robust to outliers — outliers have less influence on the scaling parameters.

```python
from sklearn.preprocessing import RobustScaler
import numpy as np

# Dataset with outliers in feature 0
X = np.array([[1.0, 1.0], [2.0, 2.0], [3.0, 3.0], [100.0, 4.0]])

robust = RobustScaler()
standard = __import__('sklearn.preprocessing', fromlist=['StandardScaler']).StandardScaler()

print(robust.fit_transform(X)[:, 0])
# Feature 0 centered on median, IQR-scaled — outlier still visible but less dominant

print(standard.fit_transform(X)[:, 0])
# Feature 0: outlier pulls mean up; normal points cluster near -0.5
```

### Normalizer

Normalizes each *sample* (row) to unit norm, not each feature (column). Use when the direction of a feature vector matters more than its magnitude — e.g., text TF-IDF, cosine similarity models.

```python
from sklearn.preprocessing import Normalizer
import numpy as np

X = np.array([[3.0, 4.0], [1.0, 0.0], [0.0, 5.0]])

norm = Normalizer(norm='l2')  # also 'l1', 'max'
X_normed = norm.fit_transform(X)
print(X_normed)
# Output: [[0.6  0.8 ]
#          [1.   0.  ]
#          [0.   1.  ]]

# Each row now has L2 norm = 1
print(np.linalg.norm(X_normed, axis=1))
# Output: [1. 1. 1.]
```

### PolynomialFeatures

Generates interaction and polynomial features. `degree=2` on `[a, b]` produces `[1, a, b, a², ab, b²]`. Used to give linear models the ability to fit nonlinear relationships.

```python
from sklearn.preprocessing import PolynomialFeatures
import numpy as np

X = np.array([[2.0, 3.0]])

poly = PolynomialFeatures(degree=2, include_bias=False, interaction_only=False)
print(poly.fit_transform(X))
# Output: [[2. 3. 4. 6. 9.]]  → [a, b, a², ab, b²]

poly_interact = PolynomialFeatures(degree=2, include_bias=False, interaction_only=True)
print(poly_interact.fit_transform(X))
# Output: [[2. 3. 6.]]  → [a, b, ab] — no squared terms
```

---

## Encoding Categoricals

### LabelEncoder

Encodes a single categorical column to integers 0..n-1. Designed for target labels `y`, not features. If you use it on a feature column, models may interpret the integers as ordinal, which is usually wrong.

```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
y = ['cat', 'dog', 'bird', 'dog', 'cat']
y_encoded = le.fit_transform(y)
print(y_encoded)         # Output: [1 2 0 2 1]
print(le.classes_)       # Output: ['bird' 'cat' 'dog']

# Decode back
print(le.inverse_transform([1, 2, 0]))
# Output: ['cat' 'dog' 'bird']
```

### OrdinalEncoder

Encodes multiple categorical feature columns to ordinal integers. Lets you specify the category order, so models that see integer distances (like tree-based models) get the ordering right.

```python
from sklearn.preprocessing import OrdinalEncoder
import numpy as np

X = np.array([['low', 'small'],
              ['medium', 'large'],
              ['high', 'medium']])

enc = OrdinalEncoder(categories=[['low', 'medium', 'high'],
                                  ['small', 'medium', 'large']])
print(enc.fit_transform(X))
# Output: [[0. 0.]
#          [1. 2.]
#          [2. 1.]]
```

### OneHotEncoder

Creates binary columns for each category. The right choice for nominal categoricals fed into linear models, SVMs, or neural nets. `sparse_output=False` returns a dense array; `handle_unknown='ignore'` silently zeros out unseen categories at inference time.

```python
from sklearn.preprocessing import OneHotEncoder
import numpy as np

X_train = np.array([['red'], ['green'], ['blue'], ['red']])
X_test  = np.array([['green'], ['purple']])  # 'purple' is unseen

ohe = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
ohe.fit(X_train)

print(ohe.transform(X_train))
# Output: [[0. 0. 1.]
#          [0. 1. 0.]
#          [1. 0. 0.]
#          [0. 0. 1.]]

print(ohe.transform(X_test))
# Output: [[0. 1. 0.]   ← 'green' encoded normally
#          [0. 0. 0.]]  ← 'purple' → all zeros (not an error)

print(ohe.categories_)
# Output: [array(['blue', 'green', 'red'], dtype='<U5')]
```

### TargetEncoder

Encodes categories using the mean of the target variable per category. Useful for high-cardinality categoricals. Includes smoothing and cross-fitting internally to prevent leakage — sklearn's implementation uses cross-fitting by default.

```python
from sklearn.preprocessing import TargetEncoder
import numpy as np

X = np.array([['A'], ['B'], ['A'], ['C'], ['B'], ['A']])
y = np.array([10.0, 20.0, 12.0, 30.0, 22.0, 11.0])

enc = TargetEncoder(smooth='auto')
X_encoded = enc.fit_transform(X, y)
print(X_encoded)
# Output: each category replaced by a smoothed version of its target mean
# 'A' → near 11.0, 'B' → near 21.0, 'C' → near 30.0
```

---

## Imputation

### SimpleImputer

Fills missing values with a summary statistic. `strategy='mean'` and `strategy='median'` work on numeric data; `strategy='most_frequent'` works on numeric and categorical; `strategy='constant'` fills with a value you specify.

```python
from sklearn.impute import SimpleImputer
import numpy as np

X = np.array([[1.0, np.nan],
              [2.0, 3.0],
              [np.nan, 4.0],
              [4.0, np.nan]])

imp = SimpleImputer(strategy='mean')
print(imp.fit_transform(X))
# Output: [[1.   3.5]
#          [2.   3. ]
#          [2.33 4. ]
#          [4.   3.5]]

imp_const = SimpleImputer(strategy='constant', fill_value=-1)
print(imp_const.fit_transform(X))
# Output: [[ 1. -1.]
#          [ 2.  3.]
#          [-1.  4.]
#          [ 4. -1.]]
```

### KNNImputer

Fills each missing value using the mean of the k nearest neighbors (measured on non-missing features). Better than mean imputation when features are correlated, but slower.

```python
from sklearn.impute import KNNImputer
import numpy as np

X = np.array([[1.0, 2.0, np.nan],
              [3.0, 4.0, 3.0],
              [np.nan, 6.0, 5.0],
              [8.0, 8.0, 7.0]])

imp = KNNImputer(n_neighbors=2)
print(imp.fit_transform(X))
# Missing values filled using weighted average of 2 nearest complete neighbors
```

### IterativeImputer

Models each feature with missing values as a function of the other features, then iterates. The most powerful imputer; effectively multivariate imputation by chained equations (MICE). Still experimental in sklearn — import from `sklearn.experimental` first.

```python
from sklearn.experimental import enable_iterative_imputer  # required import
from sklearn.impute import IterativeImputer
from sklearn.linear_model import BayesianRidge
import numpy as np

X = np.array([[1.0, 2.0, np.nan],
              [3.0, np.nan, 3.0],
              [np.nan, 6.0, 5.0],
              [8.0, 8.0, 7.0]])

imp = IterativeImputer(estimator=BayesianRidge(), max_iter=10, random_state=42)
print(imp.fit_transform(X))
# Each missing value estimated from a regression on all other features
```

---

## Feature Selection

### SelectKBest

Selects the k highest-scoring features according to a univariate statistical test. `f_classif` (ANOVA F-value) for classification; `f_regression` for regression; `mutual_info_classif` for nonlinear dependencies.

```python
from sklearn.feature_selection import SelectKBest, f_classif
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)  # 4 features

selector = SelectKBest(score_func=f_classif, k=2)
X_reduced = selector.fit_transform(X, y)
print(X_reduced.shape)
# Output: (150, 2)

print(selector.scores_)
# Output: [119.26...,  49.16..., 1180.16...,  960.00...]
# Feature 2 and 3 (petal length/width) dominate

print(selector.get_support())
# Output: [False False  True  True]
```

### SelectFromModel

Uses a model's `coef_` or `feature_importances_` attribute to select features whose importance exceeds a threshold. Works with any estimator that exposes these attributes.

```python
from sklearn.feature_selection import SelectFromModel
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)  # 30 features

rf = RandomForestClassifier(n_estimators=100, random_state=42)
selector = SelectFromModel(rf, threshold='mean')
X_reduced = selector.fit_transform(X, y)
print(X.shape, X_reduced.shape)
# Output: (569, 30) (569, 13)  — roughly half the features kept
```

### RFE

Recursive Feature Elimination: fits the model, removes the weakest feature, refits, repeats. More expensive than SelectFromModel but considers feature interactions through the elimination sequence.

```python
from sklearn.feature_selection import RFE
from sklearn.svm import SVC
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)

svc = SVC(kernel='linear')
rfe = RFE(estimator=svc, n_features_to_select=10, step=1)
rfe.fit(X, y)

print(rfe.support_)   # Boolean mask: True for selected features
print(rfe.ranking_)   # 1 = selected; higher = eliminated earlier
```

### VarianceThreshold

Removes features with variance below a threshold. A pre-filter to drop near-constant features before running any model-based selection. Fast because it requires no `y`.

```python
from sklearn.feature_selection import VarianceThreshold
import numpy as np

X = np.array([[0, 2, 0.1],
              [0, 3, 0.2],
              [0, 4, 0.3],
              [0, 5, 0.4]])

# Feature 0 is constant (variance = 0)
sel = VarianceThreshold(threshold=0.0)
print(sel.fit_transform(X))
# Output: [[2.  0.1]
#          [3.  0.2]
#          [4.  0.3]
#          [5.  0.4]]

print(sel.variances_)
# Output: [0.    1.25  0.0125]
```

---

## Dimensionality Reduction

### PCA

Principal Component Analysis projects data onto the directions of maximum variance. `n_components` can be an integer (number of components) or a float between 0 and 1 (minimum fraction of variance to retain). After fitting, `explained_variance_ratio_` tells you how much variance each component captures.

```python
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits
import numpy as np

X, y = load_digits(return_X_y=True)  # 1797 samples, 64 features

pca = PCA(n_components=0.95, random_state=42)  # retain 95% of variance
X_reduced = pca.fit_transform(X)
print(X.shape, X_reduced.shape)
# Output: (1797, 64) (1797, 29)

print(pca.explained_variance_ratio_[:5])
# Output: [0.148 0.136 0.118 0.084 0.058]  (first 5 components)

print(pca.explained_variance_ratio_.sum())
# Output: ~0.95

# Reconstruct (lossy)
X_reconstructed = pca.inverse_transform(X_reduced)
```

### TruncatedSVD

Like PCA but works on sparse matrices — no centering required. The method of choice for text data (TF-IDF matrices). `n_components` must be an integer.

```python
from sklearn.decomposition import TruncatedSVD
from sklearn.feature_extraction.text import TfidfVectorizer

corpus = [
    "machine learning is powerful",
    "deep learning uses neural networks",
    "supervised learning uses labeled data",
    "neural networks power deep learning",
]

vectorizer = TfidfVectorizer()
X_sparse = vectorizer.fit_transform(corpus)   # sparse matrix
print(X_sparse.shape)
# Output: (4, 12)

svd = TruncatedSVD(n_components=2, random_state=42)
X_reduced = svd.fit_transform(X_sparse)
print(X_reduced.shape)
# Output: (4, 2)
```

### TSNE

t-SNE reduces high-dimensional data to 2 or 3 dimensions for visualization. It preserves local structure (nearby points stay nearby) at the cost of global structure. Use it only for visualization, not as a preprocessing step for a downstream model — the transform is not stable and `transform()` is not supported (you must `fit_transform()` the full dataset at once).

```python
from sklearn.manifold import TSNE
from sklearn.datasets import load_digits
import numpy as np

X, y = load_digits(return_X_y=True)

tsne = TSNE(n_components=2, perplexity=30, random_state=42, n_iter=1000)
X_2d = tsne.fit_transform(X)
print(X_2d.shape)
# Output: (1797, 2)

# Typical pattern: plot X_2d colored by y to see cluster separation
# import matplotlib.pyplot as plt
# plt.scatter(X_2d[:, 0], X_2d[:, 1], c=y, cmap='tab10', s=5)
```

> [!warning]
> TSNE results change with `random_state` and `perplexity`. Never use t-SNE embeddings as input features for a model.

---

## Pipelines

Pipelines chain preprocessing steps and a final estimator into a single object. The key benefit: calling `pipeline.fit(X_train, y_train)` correctly fits each step only on training data and prevents data leakage automatically.

### Pipeline and make_pipeline

`Pipeline` takes a list of `(name, estimator)` tuples. `make_pipeline` infers names from class names (lowercase). Access individual steps via `pipeline.named_steps['name']` or `pipeline['name']`.

```python
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Explicit names
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('pca',    PCA(n_components=10)),
    ('clf',    LogisticRegression(max_iter=1000)),
])

pipe.fit(X_train, y_train)
print(pipe.score(X_test, y_test))
# Output: ~0.96

# Access a step's fitted attributes
print(pipe['pca'].explained_variance_ratio_.sum())

# make_pipeline: shorter, auto-names steps
pipe2 = make_pipeline(StandardScaler(), PCA(n_components=10), LogisticRegression(max_iter=1000))
```

### ColumnTransformer and make_column_transformer

`ColumnTransformer` applies different transformers to different subsets of columns. Essential for real-world datasets that mix numeric and categorical features.

```python
from sklearn.compose import ColumnTransformer, make_column_transformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression
import pandas as pd
import numpy as np

# Simulated mixed dataset
data = pd.DataFrame({
    'age':    [25.0, np.nan, 45.0, 30.0, 55.0],
    'income': [50000.0, 80000.0, np.nan, 60000.0, 90000.0],
    'city':   ['NYC', 'LA', 'NYC', 'Chicago', 'LA'],
    'edu':    ['bachelor', 'master', 'phd', 'bachelor', 'master'],
})
y = np.array([0, 1, 1, 0, 1])

numeric_features = ['age', 'income']
categorical_features = ['city', 'edu']

numeric_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler',  StandardScaler()),
])

categorical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('ohe',     OneHotEncoder(handle_unknown='ignore', sparse_output=False)),
])

preprocessor = ColumnTransformer([
    ('num', numeric_transformer,  numeric_features),
    ('cat', categorical_transformer, categorical_features),
])

full_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier',   LogisticRegression(max_iter=1000)),
])

full_pipeline.fit(data, y)
print(full_pipeline.predict(data))
# Output: [0 1 1 0 1]
```

### GridSearchCV with Pipelines

Pipeline parameter names use double underscore notation: `stepname__param`. This integrates cleanly with `GridSearchCV`.

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC
from sklearn.model_selection import GridSearchCV
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)

pipe = Pipeline([('scaler', StandardScaler()), ('svc', SVC())])

param_grid = {
    'svc__C':      [0.1, 1, 10],
    'svc__kernel': ['rbf', 'linear'],
}

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy')
grid.fit(X, y)
print(grid.best_params_)
# Output: {'svc__C': 1, 'svc__kernel': 'rbf'}  (or similar)
print(grid.best_score_)
# Output: ~0.98
```

---

## Model Selection

### cross_val_score

The simplest cross-validation interface. Returns an array of scores for each fold. Use this for a quick estimate of generalization performance.

```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)

rf = RandomForestClassifier(n_estimators=100, random_state=42)
scores = cross_val_score(rf, X, y, cv=5, scoring='roc_auc')

print(scores)
# Output: [0.993 0.997 0.985 0.997 0.991]
print(f"Mean: {scores.mean():.3f}  Std: {scores.std():.3f}")
# Output: Mean: 0.993  Std: 0.004
```

### cross_validate

Like `cross_val_score` but returns a dict with fit time, score time, and multiple metrics at once. Also lets you return the fitted estimators per fold.

```python
from sklearn.model_selection import cross_validate
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)

rf = RandomForestClassifier(n_estimators=100, random_state=42)
results = cross_validate(rf, X, y, cv=5,
                         scoring=['accuracy', 'roc_auc'],
                         return_estimator=True)

print(results.keys())
# dict_keys: ['fit_time', 'score_time', 'estimator', 'test_accuracy', 'test_roc_auc']

print(results['test_accuracy'].mean())
print(results['test_roc_auc'].mean())
```

### KFold and StratifiedKFold

`KFold` splits data into k consecutive folds. `StratifiedKFold` preserves the class proportion in each fold — always use StratifiedKFold for classification to avoid folds dominated by one class.

```python
from sklearn.model_selection import KFold, StratifiedKFold
import numpy as np

X = np.arange(20).reshape(10, 2)
y = np.array([0, 0, 0, 0, 0, 1, 1, 1, 1, 1])  # balanced

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
for fold, (train_idx, val_idx) in enumerate(skf.split(X, y)):
    print(f"Fold {fold}: train={train_idx.tolist()}, val={val_idx.tolist()}")
    # Each fold has 4 train, 1 val per class

# For regression or when class balance is not a concern
kf = KFold(n_splits=5, shuffle=True, random_state=42)
```

### GridSearchCV

Exhaustive search over a parameter grid. Every combination is evaluated with cross-validation. Fitting GridSearchCV on training data, then evaluating on test data gives an unbiased estimate.

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

param_grid = {
    'n_estimators':   [50, 100],
    'learning_rate':  [0.05, 0.1],
    'max_depth':      [2, 3],
}

grid = GridSearchCV(
    GradientBoostingClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='roc_auc',
    n_jobs=-1,
    refit=True,  # default: refit best model on full train set
)
grid.fit(X_train, y_train)

print(grid.best_params_)
print(f"CV best: {grid.best_score_:.3f}")
print(f"Test:    {grid.score(X_test, y_test):.3f}")
```

### RandomizedSearchCV

Samples `n_iter` random combinations from distributions or lists. Far more efficient than grid search when the parameter space is large — with the same compute budget it often finds better parameters.

```python
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.datasets import load_breast_cancer
from scipy.stats import randint, uniform

X, y = load_breast_cancer(return_X_y=True)

param_dist = {
    'n_estimators':  randint(50, 300),
    'learning_rate': uniform(0.01, 0.3),
    'max_depth':     randint(2, 6),
    'subsample':     uniform(0.6, 0.4),
}

search = RandomizedSearchCV(
    GradientBoostingClassifier(random_state=42),
    param_distributions=param_dist,
    n_iter=30,
    cv=5,
    scoring='roc_auc',
    random_state=42,
    n_jobs=-1,
)
search.fit(X, y)
print(search.best_params_)
print(f"CV best roc_auc: {search.best_score_:.3f}")
```

---

## Calibration and Thresholds

### predict_proba() and predict_log_proba()

Classifiers that support probability output expose `predict_proba()`, which returns a 2-D array of shape `(n_samples, n_classes)`. Column order matches `clf.classes_`. `predict_log_proba()` returns log probabilities, numerically stable for very small probabilities.

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

clf = LogisticRegression(max_iter=500)
clf.fit(X_train, y_train)

proba = clf.predict_proba(X_test)
print(proba[:3])
# Output: probability of each of the 3 classes for the first 3 samples
# e.g., [[9.8e-01, 1.9e-02, 6.7e-05], ...]

print(clf.classes_)
# Output: [0 1 2]
```

### CalibratedClassifierCV

Many classifiers (SVM, GBM) produce uncalibrated scores that are not reliable probabilities. `CalibratedClassifierCV` wraps them and applies Platt scaling (`method='sigmoid'`) or isotonic regression (`method='isotonic'`) to turn scores into proper probabilities.

```python
from sklearn.calibration import CalibratedClassifierCV
from sklearn.svm import SVC
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

svc = SVC()  # base SVC does not support predict_proba
calibrated = CalibratedClassifierCV(svc, method='sigmoid', cv=5)
calibrated.fit(X_train, y_train)

proba = calibrated.predict_proba(X_test)
print(proba[:3])  # now proper probabilities in [0, 1]
```

### Adjusting the Decision Threshold

`predict()` uses a 0.5 threshold by default for binary classifiers. For imbalanced datasets or when false positives and false negatives have different costs, adjust the threshold on `predict_proba()` output directly.

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
import numpy as np

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

clf = LogisticRegression(max_iter=1000)
clf.fit(X_train, y_train)

proba_positive = clf.predict_proba(X_test)[:, 1]

threshold = 0.3  # lower threshold → more positives, higher recall, lower precision
y_pred_custom = (proba_positive >= threshold).astype(int)

from sklearn.metrics import classification_report
print(classification_report(y_test, y_pred_custom))
```

---

## Inspection

### permutation_importance

Measures a feature's importance by randomly shuffling it and observing how much model performance drops. Works with any fitted estimator and any scoring metric. More reliable than `coef_` or `feature_importances_` for detecting true importance.

```python
from sklearn.inspection import permutation_importance
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
import pandas as pd

X, y = load_breast_cancer(return_X_y=True)
feature_names = load_breast_cancer().feature_names
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

result = permutation_importance(rf, X_test, y_test,
                                n_repeats=10,
                                random_state=42,
                                scoring='roc_auc')

importance_df = pd.DataFrame({
    'feature':    feature_names,
    'importance': result.importances_mean,
    'std':        result.importances_std,
}).sort_values('importance', ascending=False)

print(importance_df.head(5))
```

### partial_dependence and PartialDependenceDisplay

Partial dependence plots show the marginal effect of one or two features on the predicted outcome, averaging out all other features. Useful for understanding whether a relationship is monotonic, linear, or has threshold effects.

```python
from sklearn.inspection import PartialDependenceDisplay
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt

X, y = fetch_california_housing(return_X_y=True, as_frame=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

gbm = GradientBoostingRegressor(n_estimators=100, random_state=42)
gbm.fit(X_train, y_train)

# Plot PDP for the first two features individually, plus a 2D interaction
features_to_plot = [0, 1, (0, 1)]
disp = PartialDependenceDisplay.from_estimator(
    gbm, X_train, features_to_plot,
    feature_names=X.columns.tolist(),
    kind='average',
)
# plt.show()
```

---

## Custom Transformers

### BaseEstimator and TransformerMixin

Subclass both to build a transformer that integrates with Pipelines and GridSearchCV. `BaseEstimator` gives you `get_params()`/`set_params()` for free (as long as your `__init__` stores all args as same-named attributes). `TransformerMixin` gives you `fit_transform()` for free.

```python
from sklearn.base import BaseEstimator, TransformerMixin
import numpy as np

class LogTransformer(BaseEstimator, TransformerMixin):
    """Apply log1p to specified columns; leave others unchanged."""

    def __init__(self, columns=None):
        self.columns = columns  # must be same name as parameter

    def fit(self, X, y=None):
        # Nothing to learn; return self to enable chaining
        return self

    def transform(self, X, y=None):
        X = X.copy()
        cols = self.columns if self.columns else list(range(X.shape[1]))
        X[:, cols] = np.log1p(X[:, cols])
        return X


import numpy as np

X = np.array([[1.0, 100.0], [2.0, 200.0], [3.0, 300.0]])
t = LogTransformer(columns=[1])
print(t.fit_transform(X))
# Output: [[1.      4.615...]
#          [2.      5.298...]
#          [3.      5.704...]]

# Works in a pipeline
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LinearRegression

pipe = Pipeline([('log', LogTransformer(columns=[1])), ('lr', LinearRegression())])
```

### FunctionTransformer

Wraps a plain Python function (or lambda) as a transformer. No class needed. Use for simple stateless transformations.

```python
from sklearn.preprocessing import FunctionTransformer
from sklearn.pipeline import make_pipeline
from sklearn.linear_model import LogisticRegression
import numpy as np

X = np.array([[1.0, 100.0], [2.0, 200.0], [3.0, 300.0], [4.0, 400.0]])
y = np.array([0, 0, 1, 1])

log_transformer = FunctionTransformer(np.log1p, validate=True)

pipe = make_pipeline(log_transformer, LogisticRegression())
pipe.fit(X, y)
print(pipe.predict(X))
# Output: [0 0 1 1]
```

### check_is_fitted

Call this at the start of `transform()` or `predict()` in custom estimators to raise a meaningful `NotFittedError` if the user forgot to call `fit()` first.

```python
from sklearn.base import BaseEstimator, TransformerMixin
from sklearn.utils.validation import check_is_fitted
import numpy as np

class MeanSubtractor(BaseEstimator, TransformerMixin):
    def fit(self, X, y=None):
        self.mean_ = np.mean(X, axis=0)  # trailing underscore = learned attribute
        return self

    def transform(self, X, y=None):
        check_is_fitted(self)  # raises NotFittedError if mean_ does not exist
        return X - self.mean_


t = MeanSubtractor()
try:
    t.transform(np.array([[1, 2]]))
except Exception as e:
    print(type(e).__name__)
    # Output: NotFittedError

t.fit(np.array([[1.0, 2.0], [3.0, 4.0]]))
print(t.transform(np.array([[1.0, 2.0]])))
# Output: [[-1. -1.]]
```

---

> [!tip]
> The pattern that unlocks everything: `fit` on train only, `transform` train and test using the fitted object, wrap all steps in a `Pipeline` so the rule is enforced automatically.

> [!success]
> See also: `ml-cheat-sheet.md` for algorithm selection, `pandas-cheat-sheet.md` for data preparation before sklearn, and the `Projects/` directory for end-to-end examples of these patterns on real datasets.
