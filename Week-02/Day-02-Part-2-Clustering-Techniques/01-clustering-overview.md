# 🧩 01 — Clustering Overview

Clustering is unsupervised learning that groups similar rows together.

Examples:

- customer segments
- product groups
- anomaly detection
- document/topic grouping

---

## No Target Column

```python
X = df[["spend", "visits", "tenure"]]
# no y
```

The model discovers structure without labels.

---

## Common Algorithms

| Algorithm | Idea |
|---|---|
| K-Means | assign points to nearest center |
| Hierarchical | build nested clusters |
| DBSCAN | find dense regions and noise |

---

## Important Warning

Clusters are not automatically meaningful. You must profile them and interpret whether they make business sense.

---

## Next

➡️ [[02-k-means]]
