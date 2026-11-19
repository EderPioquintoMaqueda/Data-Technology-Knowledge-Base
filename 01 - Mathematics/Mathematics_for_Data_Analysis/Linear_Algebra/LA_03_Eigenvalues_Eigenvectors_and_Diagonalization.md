---
tags:
  - mathematics/linear-algebra
  - intermediate
  - eigenvalues
  - eigenvectors
  - obsidian-vault
aliases:
  - Eigenvalues and Eigenvectors
---

# Linear Algebra: Eigenvalues, Eigenvectors, and Diagonalization

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[LA_02_Matrices_and_Linear_Transformations]], [[LA_04_SVD_and_Principal_Component_Analysis]]

---

## 1. Eigenvalues and Eigenvectors Definition

When a square matrix acts on certain vectors, the resulting vector points in the exact same direction as the original vector, only scaled by a constant factor.

### Mathematical Notation
Let $A \in \mathbb{R}^{n \times n}$ be a square matrix. A non-zero vector $v \in \mathbb{R}^n$ is an **eigenvector** of $A$, and a scalar $\lambda \in \mathbb{R}$ is its corresponding **eigenvalue**, if:
$$Av = \lambda v$$

To find the eigenvalues, we rewrite the equation using the identity matrix $I$:
$$(A - \lambda I)v = 0$$
For a non-trivial solution to exist, the characteristic polynomial must equal zero:
$$\det(A - \lambda I) = 0$$

### Python Implementation
```python
import numpy as np

# A square matrix
A = np.array([[4.0, 2.0], [1.0, 3.0]], dtype=np.float32)

# Calculating eigenvalues and right eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)

print("Eigenvalues:", eigenvalues)
print("Eigenvectors (columns):\n", eigenvectors)
```
