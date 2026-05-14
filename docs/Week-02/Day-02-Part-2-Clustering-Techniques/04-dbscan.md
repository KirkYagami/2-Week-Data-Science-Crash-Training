# 🌌 04 — DBSCAN

DBSCAN finds dense regions and marks isolated points as noise.

---

## Example

```python
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler

X_scaled = StandardScaler().fit_transform(X)

model = DBSCAN(eps=0.5, min_samples=5)
labels = model.fit_predict(X_scaled)

df["cluster"] = labels
```

Cluster label `-1` means noise/outlier.

---

## Key Parameters

| Parameter | Meaning |
|---|---|
| `eps` | neighborhood radius |
| `min_samples` | points needed for dense area |

---

## Strengths

- detects non-spherical clusters
- identifies noise
- does not require choosing number of clusters

## Weaknesses

- sensitive to `eps`
- struggles with varying density
- needs scaling

---

## Next

➡️ [[05-evaluation-and-scaling]]
