# 🏷️ 03 — Categorical Features

Machine learning models need categories converted to numbers.

---

## One-Hot Encoding

```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(handle_unknown="ignore")
```

Good for nominal categories:

- city
- product type
- browser

---

## Ordinal Encoding

Use when categories have real order.

```python
mapping = {"low": 1, "medium": 2, "high": 3}
df["risk_level_num"] = df["risk_level"].map(mapping)
```

Do not ordinal-encode unordered categories like city.

---

## Rare Categories

High-cardinality categories can overfit.

```python
top = df["city"].value_counts().head(10).index
df["city_clean"] = df["city"].where(df["city"].isin(top), "Other")
```

---

## Target Encoding Warning

Target encoding can be powerful but dangerous because it can leak target information.

Use cross-validation-based target encoding if needed.

---

## Next

➡️ [[04-datetime-and-text-features]]
