# 🌲 04 — Trees, Forests, and Boosting

## Decision Tree Classifier

```python
from sklearn.tree import DecisionTreeClassifier

tree = DecisionTreeClassifier(max_depth=4, random_state=42)
tree.fit(X_train, y_train)
```

Decision trees learn rules like:

```text
if age < 30 and usage < 5 -> high churn risk
```

---

## Random Forest

```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(n_estimators=200, random_state=42)
rf.fit(X_train, y_train)
```

Random forests average many trees and usually perform better than a single tree.

---

## Gradient Boosting

```python
from sklearn.ensemble import GradientBoostingClassifier

gb = GradientBoostingClassifier(random_state=42)
gb.fit(X_train, y_train)
```

Boosting builds trees sequentially to correct previous mistakes.

---

## Feature Importance

```python
importance = pd.Series(rf.feature_importances_, index=X.columns)
print(importance.sort_values(ascending=False).head(10))
```

Feature importance is useful for explanation, but it does not prove causality.

---

## Practical Guidance

| Need | Try |
|---|---|
| interpretable rules | Decision Tree |
| strong baseline | Random Forest |
| high accuracy on tabular data | Gradient Boosting |

---

## Next

➡️ [[05-classification-metrics]]
