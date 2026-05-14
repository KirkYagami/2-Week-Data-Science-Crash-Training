# 🔒 05 — Pipelines and Leakage

Pipelines keep preprocessing and modeling together.

---

## ColumnTransformer

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

numeric_features = ["age", "income"]
categorical_features = ["city", "segment"]

preprocess = ColumnTransformer([
    ("num", StandardScaler(), numeric_features),
    ("cat", OneHotEncoder(handle_unknown="ignore"), categorical_features)
])

model = Pipeline([
    ("preprocess", preprocess),
    ("model", LogisticRegression(max_iter=1000))
])

model.fit(X_train, y_train)
```

---

## Why This Prevents Leakage

When used correctly:

- scalers fit only on training data
- encoders learn categories from training data
- test data is transformed using training rules

---

## Common Leakage Examples

- filling missing values before split using full data
- scaling before split
- creating target-based features on full data
- using future information

---

## Next

➡️ [[06-exercises]]
