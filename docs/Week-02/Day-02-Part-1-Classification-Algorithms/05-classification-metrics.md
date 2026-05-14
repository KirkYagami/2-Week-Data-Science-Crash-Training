# 📏 05 — Classification Metrics

Accuracy is not enough, especially with imbalanced data.

---

## Confusion Matrix

```python
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred)
print(cm)
```

| Term | Meaning |
|---|---|
| TP | predicted positive, actually positive |
| TN | predicted negative, actually negative |
| FP | predicted positive, actually negative |
| FN | predicted negative, actually positive |

---

## Accuracy

```python
from sklearn.metrics import accuracy_score

accuracy_score(y_test, y_pred)
```

Good when classes are balanced and errors have similar cost.

---

## Precision and Recall

```python
from sklearn.metrics import precision_score, recall_score, f1_score

precision_score(y_test, y_pred)
recall_score(y_test, y_pred)
f1_score(y_test, y_pred)
```

| Metric | Focus |
|---|---|
| Precision | avoid false positives |
| Recall | avoid false negatives |
| F1 | balance precision and recall |

---

## ROC-AUC

```python
from sklearn.metrics import roc_auc_score

roc_auc_score(y_test, y_proba[:, 1])
```

ROC-AUC measures ranking quality across thresholds.

---

## Metric Choice

| Scenario | Metric Priority |
|---|---|
| fraud detection | recall + precision |
| spam filter | precision |
| medical screening | recall |
| balanced simple task | accuracy |

---

## Next

➡️ [[06-exercises]]
