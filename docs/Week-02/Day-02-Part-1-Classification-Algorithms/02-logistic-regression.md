# 📉 02 — Logistic Regression

Logistic regression is a classification model, despite the name "regression."

It predicts probabilities between 0 and 1.

---

## Example

```python
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report

data = load_breast_cancer(as_frame=True)
X = data.data
y = data.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(max_iter=1000))
])

pipe.fit(X_train, y_train)
y_pred = pipe.predict(X_test)

print(accuracy_score(y_test, y_pred))
print(classification_report(y_test, y_pred))
```

---

## Probabilities

```python
y_proba = pipe.predict_proba(X_test)
```

For binary classification, the second column is often probability of class 1.

---

## Decision Threshold

Default threshold is usually `0.5`.

```python
positive_prob = y_proba[:, 1]
y_pred_custom = (positive_prob >= 0.7).astype(int)
```

Raising threshold usually increases precision and lowers recall.

---

## Interview Questions

**Q1:** Is logistic regression used for regression?

> No. It is used for classification.

**Q2:** Why scale features?

> Logistic regression optimization and regularization behave better with scaled features.

---

## Next

➡️ [[03-knn-and-naive-bayes]]
