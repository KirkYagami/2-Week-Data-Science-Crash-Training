# 🌳 04 — Tree-Based Regression

Tree models split data into regions and predict values from similar examples.

---

## Decision Tree Regressor

```python
from sklearn.tree import DecisionTreeRegressor

tree = DecisionTreeRegressor(max_depth=4, random_state=42)
tree.fit(X_train, y_train)
```

Pros:

- captures nonlinear patterns
- no scaling required
- easy to visualize conceptually

Cons:

- can overfit easily
- unstable with small data changes

---

## Random Forest Regressor

```python
from sklearn.ensemble import RandomForestRegressor

rf = RandomForestRegressor(
    n_estimators=200,
    max_depth=None,
    random_state=42
)
rf.fit(X_train, y_train)
```

Random forests average many trees, reducing overfitting.

---

## Gradient Boosting

```python
from sklearn.ensemble import GradientBoostingRegressor

gb = GradientBoostingRegressor(random_state=42)
gb.fit(X_train, y_train)
```

Boosting builds trees sequentially, each one correcting previous errors.

---

## Feature Importance

```python
import pandas as pd

importance = pd.Series(rf.feature_importances_, index=X.columns)
print(importance.sort_values(ascending=False))
```

Feature importance is useful, but not always causal.

---

## Next

➡️ [[05-regression-metrics]]
