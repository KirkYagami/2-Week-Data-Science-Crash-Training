# 🌿 03 — Hierarchical Clustering

Hierarchical clustering builds a tree of clusters.

---

## Agglomerative Clustering

Starts with each point as its own cluster, then merges similar clusters.

```python
from sklearn.cluster import AgglomerativeClustering
from sklearn.preprocessing import StandardScaler

X_scaled = StandardScaler().fit_transform(X)

model = AgglomerativeClustering(n_clusters=3)
labels = model.fit_predict(X_scaled)
```

---

## Dendrogram

SciPy can visualize merge structure:

```python
from scipy.cluster.hierarchy import linkage, dendrogram
import matplotlib.pyplot as plt

linked = linkage(X_scaled, method="ward")
dendrogram(linked)
plt.show()
```

---

## When Useful

- small to medium datasets
- when cluster hierarchy matters
- when you want to inspect possible cluster counts

---

## Next

➡️ [[04-dbscan]]
