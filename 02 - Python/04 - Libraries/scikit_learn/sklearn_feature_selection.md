---
id: sklearn-feature-selection
title: Scikit-Learn Module 3 - Feature Selection, Dimensionality Reduction, and PCA
tags: [python, scikit-learn, machine-learning, feature-selection, pca, dimensionality-reduction]
aliases: [Scikit-Learn Feature Selection, Principal Component Analysis, Dimensionality Reduction]
---

# Scikit-Learn Module 3: Feature Selection, Dimensionality Reduction & PCA

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


As datasets grow wider with hundreds or thousands of features, models suffer from the **Curse of Dimensionality**—overfitting, increased training time, and redundancy caused by multicollinearity. Feature selection and dimensionality reduction isolate the most informative signal.

---

## 1. Beginner Level: Removing Low-Variance Features

The simplest form of unsupervised feature selection is dropping features that have little to no variance (e.g., columns where 99% of rows contain the exact same value).

```python
import numpy as np
from sklearn.feature_selection import VarianceThreshold

# Feature matrix where column 1 is entirely constant (variance = 0)
X = np.array([[0, 2, 0], 
              [0, 3, 0], 
              [0, 4, 1], 
              [0, 5, 0]])

# Instantiate VarianceThreshold (default threshold removes features with zero variance)
selector = VarianceThreshold(threshold=0.0)
X_reduced = selector.fit_transform(X)

print("Original shape:", X.shape)
print("Reduced shape (constant column dropped):", X_reduced.shape)
```
