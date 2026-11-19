---
id: sklearn-clustering-unsupervised
title: Scikit-Learn Module 6 - Unsupervised Learning, K-Means, and DBSCAN
tags: [python, scikit-learn, machine-learning, clustering, kmeans, dbscan]
aliases: [Scikit-Learn Clustering, Unsupervised Learning, K-Means DBSCAN]
---

# Scikit-Learn Module 6: Unsupervised Learning, K-Means & DBSCAN

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While supervised learning relies on labeled target variables ($y$), unsupervised learning discovers hidden structures, groupings, and anomaly patterns directly from unlabeled feature spaces ($X$). Clustering partitions data points into natural groups such that points within the same group (cluster) are more similar to each other than to those in other groups.

---

## 1. Beginner Level: K-Means Clustering

**K-Means** is the most widely used centroid-based clustering algorithm. It partitions $n$ observations into $K$ clusters, where each observation belongs to the cluster with the nearest mean (centroid).

```python
import numpy as np
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

# Generate synthetic isotropic blob clusters
X, _ = make_blobs(n_samples=300, centers=4, cluster_std=0.60, random_state=42)

# 1. Instantiate K-Means for 4 clusters
kmeans = KMeans(n_clusters=4, init='k-means++', n_init=10, random_state=42)

# 2. Fit and predict cluster labels in one step
cluster_labels = kmeans.fit_predict(X)

print("Cluster Labels Sample:", cluster_labels[:10])
print("Final Cluster Centroids Coordinates:\n", kmeans.cluster_centers_)
print("Inertia (Sum of squared distances to closest centroid):", kmeans.inertia_)
```
