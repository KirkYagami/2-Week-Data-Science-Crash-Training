# 🧪 06 — Exercises: Feature Engineering

## Dataset

```python
customers = pd.DataFrame({
    "age": [22, 35, 42, 29],
    "city": ["Delhi", "Mumbai", "Delhi", "Pune"],
    "segment": ["Basic", "Premium", "Premium", "Basic"],
    "signup_date": ["2024-01-01", "2023-06-10", "2022-03-05", "2025-01-20"],
    "monthly_spend": [500, 2500, 3200, 700],
    "churn": [1, 0, 0, 1]
})
```

Tasks:

- create `log_monthly_spend`
- create customer tenure days
- one-hot encode city and segment
- build a `ColumnTransformer`
- train a Logistic Regression pipeline
- explain leakage risks

---

## Next

➡️ [[../Day-03-Part-2-Model-Evaluation/00-agenda|Model Evaluation]]
