# 📏 05 — Regression Metrics

Regression metrics measure prediction error.

---

## MAE

Mean Absolute Error:

```python
from sklearn.metrics import mean_absolute_error

mae = mean_absolute_error(y_test, y_pred)
```

Easy to interpret: average absolute mistake.

---

## MSE and RMSE

```python
from sklearn.metrics import mean_squared_error

mse = mean_squared_error(y_test, y_pred)
rmse = mean_squared_error(y_test, y_pred, squared=False)
```

RMSE penalizes large errors more than MAE.

---

## R² Score

```python
from sklearn.metrics import r2_score

r2 = r2_score(y_test, y_pred)
```

R² measures how much variance is explained by the model. Higher is better, but it can be misleading alone.

---

## Residuals

```python
residuals = y_test - y_pred
```

Plot residuals to inspect patterns:

```python
import matplotlib.pyplot as plt

plt.scatter(y_pred, residuals)
plt.axhline(0, color="red")
plt.show()
```

Random-looking residuals are better than patterned residuals.

---

## Metric Choice

| Metric | Use When |
|---|---|
| MAE | you want easy interpretation |
| RMSE | large errors are especially bad |
| R² | you want variance explained |

---

## Next

➡️ [[06-exercises]]
