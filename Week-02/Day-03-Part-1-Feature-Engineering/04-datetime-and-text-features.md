# 🗓️ 04 — Date, Time, and Text Features

## Date Features

```python
df["order_date"] = pd.to_datetime(df["order_date"])

df["order_month"] = df["order_date"].dt.month
df["order_dayofweek"] = df["order_date"].dt.dayofweek
df["is_weekend"] = df["order_dayofweek"].isin([5, 6])
```

---

## Time Since Event

```python
reference_date = pd.Timestamp("2026-01-01")
df["customer_age_days"] = (reference_date - df["signup_date"]).dt.days
```

Use reference dates carefully to avoid future leakage.

---

## Text Features

Simple text features:

```python
df["review_length"] = df["review"].str.len()
df["word_count"] = df["review"].str.split().str.len()
```

Vectorized text features:

```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer(max_features=5000, stop_words="english")
```

---

## Next

➡️ [[05-pipelines-and-leakage]]
