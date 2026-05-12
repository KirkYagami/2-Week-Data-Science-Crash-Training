# 📐 02 — Linear Regression

Linear regression models a straight-line relationship between features and target.

```text
y = b0 + b1*x1 + b2*x2 + ... + error
```

---

## Example

```python
from sklearn.datasets import load_diabetes
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error, r2_score

data = load_diabetes(as_frame=True)
X = data.data
y = data.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

model = LinearRegression()
model.fit(X_train, y_train)

y_pred = model.predict(X_test)
print(mean_absolute_error(y_test, y_pred))
print(r2_score(y_test, y_pred))
```

---

## Interpreting Coefficients

```python
coef = pd.Series(model.coef_, index=X.columns)
print(coef.sort_values())
```

A coefficient shows how prediction changes when that feature increases by one unit, assuming other features stay constant.

---

## Assumptions

- relationship is roughly linear
- errors are independent
- no extreme multicollinearity
- residuals behave reasonably

Linear regression is still useful even when assumptions are imperfect, but interpretation becomes weaker.

---

## Common Mistakes

- treating correlation as causation
- ignoring outliers
- interpreting coefficients after heavy scaling without care
- expecting linear regression to capture complex nonlinear patterns

---

## Next

➡️ [[03-ridge-lasso-elasticnet]]
