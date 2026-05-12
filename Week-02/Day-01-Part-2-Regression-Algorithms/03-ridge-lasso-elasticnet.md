# 🧲 03 — Ridge, Lasso, and ElasticNet

Regularization discourages overly complex models by penalizing large coefficients.

---

## Why Regularize?

Linear regression can overfit when:

- many features exist
- features are correlated
- noise is high
- sample size is small

---

## Ridge Regression

Ridge uses L2 penalty.

```python
from sklearn.linear_model import Ridge

model = Ridge(alpha=1.0)
model.fit(X_train, y_train)
```

Ridge shrinks coefficients but usually does not make them exactly zero.

---

## Lasso Regression

Lasso uses L1 penalty.

```python
from sklearn.linear_model import Lasso

model = Lasso(alpha=0.1)
model.fit(X_train, y_train)
```

Lasso can set some coefficients to zero, acting like feature selection.

---

## ElasticNet

ElasticNet combines Ridge and Lasso.

```python
from sklearn.linear_model import ElasticNet

model = ElasticNet(alpha=0.1, l1_ratio=0.5)
model.fit(X_train, y_train)
```

---

## Scaling Matters

Regularized linear models should usually use feature scaling.

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("model", Ridge(alpha=1.0))
])
```

---

## Interview Questions

**Q1:** Difference between Ridge and Lasso?

> Ridge shrinks coefficients; Lasso can shrink some to zero.

**Q2:** What is alpha?

> The regularization strength.

---

## Next

➡️ [[04-tree-based-regression]]
