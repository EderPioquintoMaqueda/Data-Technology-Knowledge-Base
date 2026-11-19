---
tags:
  - mathematics/linear-algebra
  - pro
  - svd
  - pca
  - dimensionality-reduction
  - obsidian-vault
aliases:
  - SVD and Principal Component Analysis
---

# Linear Algebra: Singular Value Decomposition (SVD) and PCA

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[LA_03_Eigenvalues_Eigenvectors_and_Diagonalization]], [[sklearn_pipelines_deployment]]

---

## 1. Singular Value Decomposition (SVD)

While eigendecomposition only applies to square matrices, **Singular Value Decomposition** can factorize any real matrix $A \in \mathbb{R}^{m \times n}$ of any shape. SVD is one of the most important tools in data science, powering dimensionality reduction, recommendation systems, and noise filtering.

### Mathematical Notation
Any matrix $A$ can be decomposed into three constituent matrices:
$$A = U \Sigma V^T$$

Where:
* $U \in \mathbb{R}^{m \times m}$ is an orthogonal matrix whose columns are the **left-singular vectors** (eigenvectors of $AA^T$).
* $\Sigma \in \mathbb{R}^{m \times n}$ is a diagonal matrix containing **singular values** $\sigma_1 \ge \sigma_2 \ge \dots \ge \sigma_min(m,n) \ge 0$.
* $V^T \in \mathbb{R}^{n \times n}$ is an orthogonal matrix whose rows are the **right-singular vectors** (eigenvectors of $A^TA$).

### Python Implementation
```python
import numpy as np

# Simulating a dataset matrix (samples x features)
np.random.seed(42)
X = np.random.rand(100, 5)

# Centering the data (subtracting feature means)
X_centered = X - np.mean(X, axis=0)

# Computing SVD
U, S, Vt = np.linalg.svd(X_centered, full_matrices=False)

print("Shape of U (Left Singular Vectors):", U.shape)
print("Singular Values (Sigma):", np.round(S, 4))
print("Shape of Vt (Right Singular Vectors Transposed):", Vt.shape)
```
