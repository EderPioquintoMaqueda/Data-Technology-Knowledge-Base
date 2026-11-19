---
tags:
  - mathematics/linear-algebra
  - beginner
  - vectors
  - obsidian-vault
aliases:
  - Vectors Spaces and Norms
---

# Linear Algebra: Vectors, Spaces, and Norms

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[LA_02_Matrices_and_Linear_Transformations]], [[numpy_array_creation]]

---

## 1. Geometric vs. Algebraic View of Vectors
In data analysis, a vector represents an individual data point, a feature embedding, or a state vector in an $n$-dimensional vector space ($\mathbb{R}^n$). 

* **Algebraic definition:** An ordered tuple of real numbers $v = [v_1, v_2, \dots, v_n]^T$.
* **Geometric definition:** A directed line segment possessing both magnitude (length) and direction originating from the origin $(0, 0, \dots, 0)$.

```python
import numpy as np

# A feature vector representing an engineering asset or data sample
v = np.array([3.0, 4.0, 5.0], dtype=np.float32)
```
