# 🎯 02 — K-Means

K-Means groups data into `k` clusters by minimizing distance to cluster centers.

---

## Example

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("model", KMeans(n_clusters=3, random_state=42, n_init="auto"))
])

clusters = pipe.fit_predict(X)
df["cluster"] = clusters
```

---

## Choosing K

Use inertia/elbow method:

```python
inertias = []

for k in range(1, 10):
    model = KMeans(n_clusters=k, random_state=42, n_init="auto")
    model.fit(StandardScaler().fit_transform(X))
    inertias.append(model.inertia_)
```

Plot `k` vs inertia and look for the elbow.

---

## Cluster Profiling

```python
df.groupby("cluster")[["spend", "visits", "tenure"]].mean()
```

This is where clustering becomes useful.

---

## Next

➡️ [[03-hierarchical-clustering]]
