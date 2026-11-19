---
tags:
  - mathematics/linear-algebra
  - intermediate
  - matrices
  - transformations
  - obsidian-vault
aliases:
  - Matrices and Linear Transformations
---

# Linear Algebra: Matrices and Linear Transformations

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[LA_01_Vectors_Spaces_and_Norms]], [[LA_03_Eigenvalues_Eigenvectors_and_Diagonalization]]

---

## 1. Matrix Multiplication as a Linear Transformation

A matrix $A \in \mathbb{R}^{m \times n}$ acts as a mapping function that transforms vectors from an input space $\mathbb{R}^n$ to an output space $\mathbb{R}^m$.

### Mathematical Notation
Given a vector $x \in \mathbb{R}^n$, the linear transformation $T(x)$ is defined as:
$$T(x) = Ax = \sum_{j=1}^{n} x_j a_j$$
Where $a_j$ represents the $j$-th column vector of matrix $A$.

### Python Implementation
```python
import numpy as np

# A 2x2 transformation matrix (scaling and shearing)
A = np.array([[2.0, 1.0], [0.0, 3.0]], dtype=np.float32)

# Input vector x in R^2
x = np.array([1.0, 1.0], dtype=np.float32)

# Matrix-vector multiplication (Linear Transformation)
transformed_x = np.dot(A, x)
print("Transformed Vector T(x):", transformed_x)  # Output: [3., 3.]
```
