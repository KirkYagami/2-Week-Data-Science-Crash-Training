# 📈 03 — Regression Evaluation

Use regression metrics based on error cost.

---

## Core Metrics

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

mae = mean_absolute_error(y_test, y_pred)
rmse = mean_squared_error(y_test, y_pred, squared=False)
r2 = r2_score(y_test, y_pred)
```

---

## Error Analysis

```python
errors = y_test - y_pred
```

Check:

- largest errors
- error by segment
- residual plot
- underprediction vs overprediction

---

## Reporting Template

```text
Model: Random Forest Regressor
Metric: MAE = 12.4
Interpretation: average prediction misses by 12.4 units
Weakness: larger errors for high-value customers
```

---

## Next

➡️ [[04-classification-evaluation]]
