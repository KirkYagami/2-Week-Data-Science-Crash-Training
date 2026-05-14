# 🎛️ 05 — Model Selection and Tuning

Model selection compares candidate models using fair evaluation.

---

## Grid Search

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    "model__n_estimators": [100, 200],
    "model__max_depth": [None, 5, 10]
}

search = GridSearchCV(
    model,
    param_grid,
    cv=5,
    scoring="accuracy"
)

search.fit(X_train, y_train)
print(search.best_params_)
```

---

## Random Search

```python
from sklearn.model_selection import RandomizedSearchCV
```

Random search is often faster for large parameter spaces.

---

## Rules

- tune on training/cross-validation
- evaluate final model once on test set
- keep metric aligned with business goal
- avoid chasing tiny leaderboard gains

---

## Next

➡️ [[06-exercises]]
