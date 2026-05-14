# Clustering

Clustering sits at the intersection of statistics, geometry, and domain knowledge. Interviews test whether you understand the mechanics deeply enough to know when each algorithm fails — not just how to call `fit()`.

---

### Q1: What is unsupervised learning and how is clustering different from classification?

??? "Show answer"
    **Unsupervised learning** finds structure in data without labelled examples. There is no target variable — the algorithm must discover patterns purely from the input features.

    **Clustering** groups observations so that items within a group are more similar to each other than to items in other groups. The groups (clusters) are not predefined — the algorithm discovers them.

    **Key differences from classification:**

    | | Classification | Clustering |
    |---|---|---|
    | Labels | Required (supervised) | Not required (unsupervised) |
    | Goal | Predict known categories | Discover unknown groupings |
    | Evaluation | Accuracy, F1, AUC vs ground truth | Silhouette score, inertia, domain validity |
    | Output | Predefined class | Discovered group identity |

    Clustering is used for: customer segmentation, anomaly detection, document grouping, image compression, exploratory data analysis, and as a preprocessing step (e.g., finding sub-populations before building separate models).

    The hardest part of clustering in practice is not running the algorithm — it's evaluating whether the discovered clusters are meaningful and actionable.

---

### Q2: How does K-Means work step by step?

??? "Show answer"
    K-Means partitions n observations into K clusters, minimising within-cluster variance (inertia).

    **Algorithm (Lloyd's algorithm):**

    1. **Initialise**: choose K cluster centroids (randomly or via K-Means++).
    2. **Assign**: assign each observation to the nearest centroid based on Euclidean distance.
    3. **Update**: recompute each centroid as the mean of all points assigned to it.
    4. **Repeat**: repeat steps 2 and 3 until centroids stop moving (convergence) or a maximum number of iterations is reached.

    ```python
    from sklearn.cluster import KMeans

    km = KMeans(n_clusters=4, init='k-means++', n_init=10, random_state=42)
    km.fit(X)

    labels = km.labels_          # cluster assignment for each point
    centroids = km.cluster_centers_
    inertia = km.inertia_        # sum of squared distances to centroids
    ```

    **Convergence guarantee:** K-Means always converges (inertia decreases or stays constant each iteration), but it may converge to a local minimum. Running with multiple random initialisations (`n_init=10`) and keeping the best result mitigates this.

    **Objective function (inertia):**
    ```
    Inertia = Σᵢ Σₓ∈Cᵢ ||x - μᵢ||²
    ```

    K-Means is guaranteed to find a local minimum of inertia, not the global minimum. The result depends on initialisation.

---

### Q3: What are the limitations of K-Means?

??? "Show answer"
    K-Means has several well-known failure modes that every practitioner should recognise:

    - **Must specify K upfront**: the number of clusters is a hyperparameter, not something the algorithm discovers. Bad K choices produce meaningless results.

    - **Assumes spherical, equally-sized clusters**: K-Means minimises Euclidean distance to centroids, so it implicitly assumes clusters are convex and roughly the same size. It fails on elongated, crescent-shaped, or nested clusters.

    - **Sensitive to initialisation**: different random starts can produce very different clusterings. Use `n_init > 1` and K-Means++ initialisation.

    - **Sensitive to outliers**: centroids are means — a single extreme point pulls the centroid away from the true cluster centre. K-Medoids (uses actual data points as centres) is more robust.

    - **Euclidean distance assumption**: sensitive to feature scale and doesn't handle high-dimensional data well (curse of dimensionality). Always standardise features before running K-Means.

    - **Only works on numeric data**: can't handle categorical features directly (use K-Modes or K-Prototypes for mixed data).

    - **Hard assignment**: each point belongs to exactly one cluster with full certainty. Gaussian Mixture Models offer soft (probabilistic) assignments.

    ```python
    # Always scale before K-Means
    from sklearn.preprocessing import StandardScaler
    from sklearn.pipeline import Pipeline

    pipe = Pipeline([
        ('scaler', StandardScaler()),
        ('km', KMeans(n_clusters=4, random_state=42))
    ])
    ```

---

### Q4: How do you choose K in K-Means?

??? "Show answer"
    There is no single correct answer — choosing K combines quantitative heuristics with domain knowledge.

    **Elbow method:**
    Plot inertia (within-cluster sum of squares) vs. K. Look for the "elbow" — the point where adding more clusters yields diminishing reductions in inertia. The elbow is not always obvious and can be subjective.

    ```python
    import matplotlib.pyplot as plt

    inertias = []
    K_range = range(2, 12)
    for k in K_range:
        km = KMeans(n_clusters=k, n_init=10, random_state=42)
        km.fit(X_scaled)
        inertias.append(km.inertia_)

    plt.plot(K_range, inertias, 'bo-')
    plt.xlabel('K'); plt.ylabel('Inertia')
    plt.title('Elbow Method')
    ```

    **Silhouette score:**
    For each K, compute the average silhouette score. Higher is better. This is more principled than the elbow method because it incorporates both cohesion (within-cluster distance) and separation (between-cluster distance).

    ```python
    from sklearn.metrics import silhouette_score

    scores = []
    for k in range(2, 12):
        km = KMeans(n_clusters=k, n_init=10, random_state=42)
        labels = km.fit_predict(X_scaled)
        scores.append(silhouette_score(X_scaled, labels))
    ```

    **Other methods:**
    - **Gap statistic**: compares inertia to that of a reference random distribution; more statistically principled but computationally expensive.
    - **Domain knowledge**: often the most valuable input. If you're segmenting customers for a 3-tier pricing model, K=3 is the business constraint.

---

### Q5: What is the silhouette score?

??? "Show answer"
    The silhouette score measures how well each point fits in its assigned cluster relative to other clusters.

    For a point i:
    - **a(i)**: mean distance to all other points in the same cluster (cohesion — lower is better).
    - **b(i)**: mean distance to all points in the nearest neighbouring cluster (separation — higher is better).

    ```
    s(i) = (b(i) - a(i)) / max(a(i), b(i))
    ```

    The silhouette score ranges from -1 to +1:
    - **+1**: the point is well inside its cluster and far from others. Perfect assignment.
    - **0**: the point is on the boundary between two clusters.
    - **-1**: the point is closer to another cluster than its own — likely misassigned.

    The **mean silhouette score** over all points summarises overall clustering quality. Use it to:
    - Compare different values of K.
    - Compare different clustering algorithms.
    - Identify potentially misassigned points (individual silhouette values).

    ```python
    from sklearn.metrics import silhouette_score, silhouette_samples
    import numpy as np

    avg_score = silhouette_score(X_scaled, labels)  # overall quality
    sample_scores = silhouette_samples(X_scaled, labels)  # per-point scores
    ```

    Limitation: computationally O(n²) due to pairwise distance calculations — slow for very large datasets. Use approximate methods or subsample for large n.

---

### Q6: What is hierarchical clustering and what are the linkage methods?

??? "Show answer"
    Hierarchical clustering builds a nested hierarchy of clusters without requiring a pre-specified K. There are two approaches:

    - **Agglomerative (bottom-up)**: start with each point as its own cluster, then iteratively merge the two most similar clusters. The default in most libraries.
    - **Divisive (top-down)**: start with one cluster containing all points, then recursively split.

    The result is a **dendrogram** — a tree diagram showing the merge sequence and distances. You choose K by cutting the dendrogram at a desired height.

    **Linkage methods determine how inter-cluster distance is measured:**

    - **Single linkage (minimum)**: distance between clusters = distance between their closest points. Tends to produce long, chain-like clusters ("chaining effect"). Sensitive to outliers.
    - **Complete linkage (maximum)**: distance = distance between their farthest points. Produces compact, roughly equal-sized clusters. Sensitive to outliers.
    - **Average linkage (UPGMA)**: distance = average pairwise distance between all points in the two clusters. A good compromise.
    - **Ward's linkage**: merges clusters that minimise the increase in total within-cluster variance. Produces compact clusters and is the most commonly used in practice.

    ```python
    from sklearn.cluster import AgglomerativeClustering
    import scipy.cluster.hierarchy as sch

    # Ward linkage, 4 clusters
    hc = AgglomerativeClustering(n_clusters=4, linkage='ward')
    labels = hc.fit_predict(X_scaled)

    # Visualise dendrogram
    linkage_matrix = sch.linkage(X_scaled, method='ward')
    sch.dendrogram(linkage_matrix)
    ```

---

### Q7: When would you prefer hierarchical clustering over K-Means?

??? "Show answer"
    **Prefer hierarchical clustering when:**

    - **You don't know K in advance**: the dendrogram lets you explore the cluster structure visually and choose K after seeing the data. K-Means forces a commitment upfront.
    - **You need nested/hierarchical structure**: taxonomies, phylogenetic trees, and document hierarchies are naturally hierarchical. The dendrogram directly represents this structure.
    - **Your clusters are non-spherical**: with the right linkage (e.g., single linkage for elongated clusters, Ward for compact), hierarchical clustering handles non-convex shapes that K-Means cannot.
    - **Your dataset is small-to-medium**: hierarchical clustering is O(n² log n) or O(n³) depending on implementation — impractical for large n.
    - **Interpretability matters**: a dendrogram tells a story about how data points relate, which can be valuable for presenting results to stakeholders.

    **Stick with K-Means when:**
    - Dataset is large (millions of points).
    - You already know K from domain knowledge.
    - Speed and scalability are priorities.
    - You want to update the clustering as new data arrives (hierarchical clustering must rerun from scratch).

---

### Q8: What is DBSCAN and how does it handle noise?

??? "Show answer"
    DBSCAN (Density-Based Spatial Clustering of Applications with Noise) discovers clusters as dense regions of points separated by sparse regions. It doesn't require specifying K and naturally identifies outliers.

    **Core concepts:**
    - **Core point**: a point with at least `min_samples` neighbours within radius `epsilon`.
    - **Border point**: within epsilon of a core point but doesn't have enough neighbours to be a core point itself.
    - **Noise point (outlier)**: not a core point, not within epsilon of any core point. Assigned label = -1.

    **Algorithm:**
    1. For each unvisited point, check if it's a core point.
    2. If yes, start a new cluster and expand it by adding all density-reachable points.
    3. If no, mark as noise temporarily (may be absorbed as a border point later).

    ```python
    from sklearn.cluster import DBSCAN

    db = DBSCAN(eps=0.5, min_samples=5)
    labels = db.fit_predict(X_scaled)

    n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
    n_noise = (labels == -1).sum()
    print(f"Clusters: {n_clusters}, Noise points: {n_noise}")
    ```

    **Why it handles noise:** points that are isolated (not within epsilon of enough other points) are explicitly labelled as noise — they're excluded from all clusters rather than being forced into the nearest one (as K-Means would do).

---

### Q9: What are epsilon and min_samples in DBSCAN?

??? "Show answer"
    These two hyperparameters fully define what DBSCAN considers "dense" and therefore what constitutes a cluster.

    **epsilon (ε):** the radius of the neighbourhood around each point. Two points are "neighbours" if the distance between them is ≤ ε.
    - Too small: most points become noise; clusters fragment.
    - Too large: clusters merge; eventually everything is one cluster.

    **min_samples:** the minimum number of points (including the point itself) required within epsilon for a point to be a core point.
    - Too small (e.g., 1): every point is a core point; no noise is identified.
    - Too large: fewer core points; clusters must be very dense to form.

    **How to choose them:**

    - **k-distance plot for epsilon**: compute the distance from each point to its k-th nearest neighbour (k = min_samples - 1). Sort these distances and plot them. Look for the "elbow" — that value is a good epsilon.

    ```python
    from sklearn.neighbors import NearestNeighbors
    import numpy as np

    k = 4  # = min_samples - 1
    nbrs = NearestNeighbors(n_neighbors=k+1).fit(X_scaled)
    distances, _ = nbrs.kneighbors(X_scaled)
    k_distances = np.sort(distances[:, -1])[::-1]

    import matplotlib.pyplot as plt
    plt.plot(k_distances)
    plt.ylabel(f'{k}-th nearest neighbour distance')
    plt.xlabel('Points (sorted)')
    ```

    - **min_samples rule of thumb**: use `min_samples ≥ dimensionality + 1`. For 2D data, min_samples = 3–4 is common. For higher dimensions, increase accordingly.

---

### Q10: What is the difference between K-Means and Gaussian Mixture Models (GMM)?

??? "Show answer"
    Both partition data into K groups, but they operate very differently.

    **K-Means:**
    - Hard assignment: each point belongs to exactly one cluster.
    - Assumes spherical clusters of equal variance.
    - Minimises Euclidean distance to centroids.
    - No probabilistic interpretation for individual assignments.

    **Gaussian Mixture Models:**
    - Soft assignment: each point has a probability of belonging to each cluster.
    - Assumes data is generated from a mixture of K Gaussian distributions, each with its own mean and covariance.
    - Fit via Expectation-Maximisation (EM).
    - Can model elliptical clusters of different sizes and orientations.

    ```python
    from sklearn.mixture import GaussianMixture

    gmm = GaussianMixture(n_components=4, covariance_type='full', random_state=42)
    gmm.fit(X_scaled)

    labels = gmm.predict(X_scaled)            # hard assignment (argmax)
    probs = gmm.predict_proba(X_scaled)       # soft assignments
    log_likelihood = gmm.score(X_scaled)      # model fit
    ```

    **When to use GMM over K-Means:**
    - Clusters are not spherical or have different densities/sizes.
    - You need uncertainty quantification (soft assignments).
    - You want model selection via BIC/AIC (GMM provides a proper likelihood).

    ```python
    # Model selection for GMM
    bic_scores = [GaussianMixture(n_components=k).fit(X_scaled).bic(X_scaled)
                  for k in range(2, 10)]
    ```

    K-Means is a special case of GMM where each Gaussian has equal, spherical covariance and the EM reduces to hard assignments.

---

### Q11: How do you evaluate clustering when there are no labels?

??? "Show answer"
    Without ground truth labels, evaluation is harder and more subjective. You have two categories of metrics:

    **Internal metrics (no labels required):**

    - **Silhouette score**: measures cohesion vs. separation for each point. Range: [-1, 1]; higher is better. Works for any clustering algorithm.
    - **Davies-Bouldin index**: ratio of within-cluster scatter to between-cluster separation. Lower is better.
    - **Calinski-Harabasz index (Variance Ratio Criterion)**: ratio of between-cluster to within-cluster dispersion. Higher is better. Fast to compute.
    - **Inertia (WCSS)**: only meaningful for K-Means, and lower is always better with more K — use only for elbow method comparison.

    ```python
    from sklearn.metrics import silhouette_score, davies_bouldin_score, calinski_harabasz_score

    sil = silhouette_score(X_scaled, labels)
    db  = davies_bouldin_score(X_scaled, labels)
    ch  = calinski_harabasz_score(X_scaled, labels)
    ```

    **External metrics (when ground truth is available for benchmarking):**
    - Adjusted Rand Index (ARI), Normalised Mutual Information (NMI), Fowlkes-Mallows score.

    **Qualitative evaluation:**
    - Visualise clusters in 2D via PCA or t-SNE — do the clusters look meaningful?
    - Check whether clusters align with domain knowledge or business logic.
    - Measure cluster stability across different random seeds or bootstrap samples.

    Internal metrics give you a relative score — use them to compare models, not as absolute measures of quality.

---

### Q12: What is the difference between hard and soft clustering?

??? "Show answer"
    **Hard clustering**: each data point is assigned to exactly one cluster with certainty. The assignment is binary — a point either belongs to cluster k or it doesn't.

    - K-Means: hard by design. Each point is assigned to the nearest centroid.
    - Hierarchical clustering: hard — cutting the dendrogram gives discrete group memberships.
    - DBSCAN: hard (plus explicit noise label).

    **Soft (fuzzy) clustering**: each point has a degree of membership in each cluster, represented as probabilities or membership coefficients. All memberships for a point sum to 1.

    - Gaussian Mixture Models: `P(cluster k | point x)` for each k.
    - Fuzzy C-Means: membership coefficients instead of crisp assignments.

    **When soft clustering is preferable:**
    - A point genuinely sits on the boundary between two groups (e.g., a customer with mixed shopping patterns belonging to both "budget shopper" and "occasional luxury buyer" segments).
    - Downstream decisions require uncertainty quantification.
    - You want to model overlapping clusters.

    ```python
    from sklearn.mixture import GaussianMixture

    gmm = GaussianMixture(n_components=3).fit(X_scaled)
    # Soft: probabilities
    probs = gmm.predict_proba(X_scaled)  # shape: (n_samples, 3)
    # Hard: take argmax
    hard_labels = gmm.predict(X_scaled)
    ```

---

### Q13: What is dimensionality reduction's role before clustering?

??? "Show answer"
    High-dimensional data causes two problems for clustering:

    1. **Curse of dimensionality**: distances between points become approximately equal in high dimensions, destroying the distance-based notion of "closeness" that all clustering algorithms rely on.

    2. **Noise and irrelevant features**: most features in high-dimensional data are noise. Including them dilutes the signal from the few features that actually define cluster structure.

    **Common approaches:**

    - **PCA before K-Means**: reduces noise, removes redundancy, and often makes clusters more visually interpretable. Keep enough components to explain 80–95% of variance.

    ```python
    from sklearn.decomposition import PCA
    from sklearn.pipeline import Pipeline
    from sklearn.cluster import KMeans
    from sklearn.preprocessing import StandardScaler

    pipe = Pipeline([
        ('scaler', StandardScaler()),
        ('pca', PCA(n_components=0.95)),   # retain 95% of variance
        ('km', KMeans(n_clusters=4, n_init=10))
    ])
    pipe.fit(X)
    ```

    - **t-SNE / UMAP**: primarily for 2D/3D visualisation of cluster structure. Do not use t-SNE's low-dimensional embedding as input to K-Means — t-SNE distorts global distances and the resulting clusters are unreliable.

    - **Autoencoders**: for very high-dimensional or complex data (images, text), a learned latent representation can be a better basis for clustering than PCA.

    The pipeline pattern (scale → reduce → cluster) is the standard approach and protects against data leakage in cross-validated cluster evaluation.

---

### Q14: What is cluster stability?

??? "Show answer"
    Cluster stability measures how consistently a clustering algorithm produces the same groupings when applied to slightly different versions of the dataset (e.g., different bootstrap samples, different random seeds).

    **Why it matters:** internal metrics like silhouette score tell you about the current clustering. Stability tells you whether the discovered structure is robust or an artefact of the specific sample.

    **How to assess it:**

    1. Generate B bootstrap samples of the data.
    2. Run your clustering algorithm on each sample.
    3. Compare cluster assignments across runs using a similarity metric (Adjusted Rand Index, Jaccard index).
    4. If ARI is consistently high (≈ 1), the clusters are stable. If ARI varies widely, the cluster structure is unreliable.

    ```python
    from sklearn.utils import resample
    from sklearn.metrics import adjusted_rand_score
    from sklearn.cluster import KMeans
    import numpy as np

    ari_scores = []
    base_labels = KMeans(n_clusters=4, n_init=10).fit_predict(X_scaled)

    for _ in range(20):
        X_boot = resample(X_scaled)
        boot_labels = KMeans(n_clusters=4, n_init=10).fit_predict(X_boot)
        # Can only compare on overlapping indices — simplification:
        ari_scores.append(adjusted_rand_score(
            base_labels[:len(boot_labels)], boot_labels
        ))

    print(f"Mean ARI: {np.mean(ari_scores):.3f}, Std: {np.std(ari_scores):.3f}")
    ```

    A stable clustering is a necessary (but not sufficient) condition for a meaningful clustering. Stable noise clusters can be stably wrong.

---

### Q15: How does K-Means++ differ from standard K-Means?

??? "Show answer"
    K-Means++ improves the initialisation step of K-Means to produce better clusterings and faster convergence.

    **Standard K-Means initialisation**: choose K centroids uniformly at random from the data. This can place multiple centroids in the same dense region, leaving other regions unrepresented, leading to poor local minima.

    **K-Means++ initialisation:**
    1. Choose the first centroid uniformly at random.
    2. For each subsequent centroid: compute the distance from every point to the nearest already-chosen centroid. Sample the next centroid with probability proportional to this squared distance.
    3. Points far from existing centroids are more likely to be chosen as the next centroid.
    4. Repeat until K centroids are chosen, then run standard K-Means.

    ```python
    from sklearn.cluster import KMeans

    # K-Means++ is the default in sklearn
    km_plus = KMeans(n_clusters=4, init='k-means++', n_init=10)

    # Standard random initialisation for comparison
    km_rand = KMeans(n_clusters=4, init='random', n_init=10)
    ```

    **Benefits of K-Means++:**
    - Tends to find better local optima (lower final inertia) than random initialisation.
    - Converges in fewer iterations.
    - The theoretical guarantee is that K-Means++ produces a solution within O(log K) of the optimal — no such guarantee exists for random initialisation.

    K-Means++ is the default in sklearn (`init='k-means++'`) and should always be preferred over random initialisation. It comes with no computational downside relative to the iterations saved.

---
