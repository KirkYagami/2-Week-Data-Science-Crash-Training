# 🛠️ 03 — Feature Engineering

## Candidate Features

- tenure months
- monthly spend
- total spend
- support tickets
- contract type
- payment method
- usage frequency

## Engineered Features

```python
df["spend_per_month"] = df["total_spend"] / df["tenure_months"].replace(0, 1)
df["high_support_usage"] = df["support_tickets"] >= 3
```

## Encoding

Use `OneHotEncoder(handle_unknown="ignore")` for nominal categories.

## Scaling

Use `StandardScaler` for linear models, KNN, and neural networks.

## Leakage Check

Remove columns that would only exist after churn:

- cancellation date
- final bill status
- retention offer accepted after churn decision

---

## Next

➡️ [[04-modeling-pipeline]]
