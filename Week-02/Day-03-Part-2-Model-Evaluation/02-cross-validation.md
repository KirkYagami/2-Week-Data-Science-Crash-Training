# 🔁 02 — Cross-Validation

Cross-validation evaluates a model across multiple train/validation splits.

---

## K-Fold Cross-Validation

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, X, y, cv=5, scoring="accuracy")
print(scores)
print(scores.mean())
```

---

## Why Use It?

- more reliable than one split
- helps detect unstable models
- useful for model selection

---

## Stratified K-Fold

For classification, preserve class balance:

```python
from sklearn.model_selection import StratifiedKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
```

---

## Time Series Warning

Do not randomly cross-validate time series.

Use:

```python
from sklearn.model_selection import TimeSeriesSplit
```

---

## Next

➡️ [[03-regression-evaluation]]
