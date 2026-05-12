# 📈 01 — Regression Overview

Regression predicts a continuous numeric value.

Examples:

- house price
- sales revenue
- delivery time
- customer lifetime value

---

## Basic Workflow

```python
X = df.drop(columns=["price"])
y = df["price"]
```

Then:

1. split data
2. preprocess features
3. train regression model
4. evaluate error
5. inspect residuals

---

## Common Regression Models

| Model | Strength |
|---|---|
| Linear Regression | simple baseline, interpretable |
| Ridge | handles multicollinearity |
| Lasso | can shrink features to zero |
| Decision Tree Regressor | captures nonlinear patterns |
| Random Forest Regressor | strong general-purpose model |
| Gradient Boosting | often high accuracy |

---

## Key Idea

Regression is not about being exactly correct. It is about minimizing prediction error and understanding when errors are acceptable.

---

## Next

➡️ [[02-linear-regression]]
