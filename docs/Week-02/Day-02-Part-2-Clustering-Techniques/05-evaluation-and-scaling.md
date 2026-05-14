# 📏 05 — Evaluation and Scaling

Clustering evaluation is harder because there is no target label.

---

## Scaling is Essential

Distance-based algorithms are affected by feature scale.

```python
from sklearn.preprocessing import StandardScaler

X_scaled = StandardScaler().fit_transform(X)
```

---

## Silhouette Score

```python
from sklearn.metrics import silhouette_score

score = silhouette_score(X_scaled, labels)
print(score)
```

Higher is generally better, but interpretation still matters.

---

## Cluster Profiling

Always profile clusters:

```python
df.groupby("cluster").agg(
    avg_spend=("spend", "mean"),
    avg_visits=("visits", "mean"),
    count=("cluster", "size")
)
```

---

## Business Validation

Ask:

- Are clusters stable?
- Are they explainable?
- Can the business act on them?
- Are they just artifacts of scale or outliers?

---

## Next

➡️ [[06-exercises]]
