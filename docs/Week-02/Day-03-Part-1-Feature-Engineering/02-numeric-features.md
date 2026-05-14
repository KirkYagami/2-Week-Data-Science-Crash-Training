# 🔢 02 — Numeric Features

Numeric features often need scaling, transformation, or binning.

---

## Scaling

Use scaling for distance-based and linear models.

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

StandardScaler()
MinMaxScaler()
```

Models that often benefit:

- Logistic Regression
- Linear Regression with regularization
- KNN
- SVM
- Neural Networks

Tree models usually do not require scaling.

---

## Log Transform

Useful for right-skewed values like income or spend.

```python
import numpy as np

df["log_spend"] = np.log1p(df["spend"])
```

`log1p` handles zero safely.

---

## Binning

```python
df["age_group"] = pd.cut(
    df["age"],
    bins=[0, 25, 40, 60, 100],
    labels=["young", "adult", "middle", "senior"]
)
```

Binning can improve interpretability but may lose detail.

---

## Interaction Features

```python
df["revenue_per_order"] = df["revenue"] / df["order_count"]
df["price_per_unit"] = df["revenue"] / df["quantity"]
```

Create interactions when domain logic supports them.

---

## Next

➡️ [[03-categorical-features]]
