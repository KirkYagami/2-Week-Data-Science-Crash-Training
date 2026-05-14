# 🎯 04 — Classification Evaluation

Classification metrics depend on error cost.

---

## Report

```python
from sklearn.metrics import classification_report, confusion_matrix

print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
```

---

## ROC-AUC and PR-AUC

```python
from sklearn.metrics import roc_auc_score, average_precision_score

roc_auc_score(y_test, y_proba)
average_precision_score(y_test, y_proba)
```

PR-AUC is often useful for imbalanced positive classes.

---

## Threshold Tuning

```python
y_pred = (y_proba >= 0.3).astype(int)
```

Threshold choice should match business cost.

---

## Next

➡️ [[05-model-selection-and-tuning]]
