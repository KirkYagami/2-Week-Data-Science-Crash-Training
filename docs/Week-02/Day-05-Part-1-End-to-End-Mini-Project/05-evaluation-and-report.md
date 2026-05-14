# 📋 05 — Evaluation and Report

## Evaluation

```python
from sklearn.metrics import classification_report, confusion_matrix, roc_auc_score

y_pred = model.predict(X_test)
y_proba = model.predict_proba(X_test)[:, 1]

print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
print(roc_auc_score(y_test, y_proba))
```

## Error Analysis

Inspect:

- false negatives
- false positives
- performance by segment
- threshold tradeoffs

## Report Structure

```text
Problem:
Data:
EDA findings:
Cleaning steps:
Features:
Models tried:
Best model:
Metric:
Business recommendation:
Limitations:
Next steps:
```

---

## Next

➡️ [[06-submission-checklist]]
