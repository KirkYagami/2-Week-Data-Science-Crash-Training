# 👥 03 — KNN and Naive Bayes

## K-Nearest Neighbors

KNN predicts based on nearby training examples.

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

knn = Pipeline([
    ("scaler", StandardScaler()),
    ("model", KNeighborsClassifier(n_neighbors=5))
])

knn.fit(X_train, y_train)
```

KNN needs scaling because it uses distances.

---

## Choosing K

Small `k`:

- flexible
- can overfit

Large `k`:

- smoother
- can underfit

---

## Naive Bayes

Naive Bayes is a probabilistic classifier. It is especially common for text classification.

```python
from sklearn.naive_bayes import GaussianNB

nb = GaussianNB()
nb.fit(X_train, y_train)
```

For text:

```python
from sklearn.naive_bayes import MultinomialNB
```

---

## Model Comparison

| Model | Pros | Cons |
|---|---|---|
| KNN | simple, intuitive | slow on large data, needs scaling |
| Naive Bayes | fast, strong for text | independence assumption is unrealistic |

---

## Next

➡️ [[04-trees-forests-boosting]]
