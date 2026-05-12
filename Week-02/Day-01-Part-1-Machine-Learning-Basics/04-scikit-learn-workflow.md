# 🧰 04 — Scikit-learn Workflow

Scikit-learn gives a consistent API for ML models:

```python
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

---

## Minimal Classification Example

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score

iris = load_iris(as_frame=True)
X = iris.data
y = iris.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

model = DecisionTreeClassifier(random_state=42)
model.fit(X_train, y_train)

y_pred = model.predict(X_test)
print(accuracy_score(y_test, y_pred))
```

---

## The Estimator Pattern

Most scikit-learn objects use:

| Method | Use |
|---|---|
| `fit()` | learn from data |
| `predict()` | predict labels/values |
| `transform()` | transform features |
| `fit_transform()` | fit and transform training data |
| `score()` | quick evaluation |

---

## Pipelines

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression())
])

pipe.fit(X_train, y_train)
pipe.score(X_test, y_test)
```

Pipelines reduce leakage and keep workflow clean.

---

## Common Workflow Checklist

- [ ] define features and target
- [ ] split train/test
- [ ] build preprocessing
- [ ] train model
- [ ] evaluate on test set
- [ ] inspect errors
- [ ] document metric and limitations

---

## Interview Questions

**Q1:** What does `fit()` do?

> It trains or learns parameters from data.

**Q2:** Why use pipelines?

> They combine preprocessing and modeling safely and reproducibly.

---

## Next

➡️ [[05-exercises]]
