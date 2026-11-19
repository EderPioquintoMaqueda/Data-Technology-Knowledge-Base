---
id: numpy-broadcasting-engine
title: NumPy Module 5 - The Broadcasting Engine and Dimension Expansion
tags: [python, numpy, data-science, broadcasting, vectorization]
aliases: [NumPy Broadcasting, Array Alignment, Dimension Expansion]
---

# NumPy Module 5: The Broadcasting Engine & Dimension Expansion

Broadcasting is one of NumPy's most powerful and mathematically elegant features. It allows arithmetic operations to be performed on arrays of different shapes without manually duplicating or replicating data in memory.

---

## 1. Beginner Level: Scalar vs. Array Operations

Before looking at multi-dimensional broadcasting, consider how a scalar interacts with an array. NumPy automatically stretches (broadcasts) the scalar to match the shape of the array.

```python
import numpy as np

arr = np.array([1, 2, 3, 4])
scalar = 10

# The scalar 10 is virtually broadcasted across all elements
result = arr + scalar
print("Scalar Addition:", result) # Output: [11 12 13 14]
```

---

## 2. Intermediate Level: The Broadcasting Rules

NumPy compares shapes **from the right** (trailing dimensions). Two dimensions are compatible when:
1. they are **equal**, or
2. one of them is **1** (it gets stretched).

| Shape A | Shape B | Result |
|---|---|---|
| `(3, 4)` | `(4,)` | `(3, 4)` ✅ row vector applied to every row |
| `(3, 4)` | `(3, 1)` | `(3, 4)` ✅ column vector applied to every column |
| `(3, 1)` | `(1, 4)` | `(3, 4)` ✅ outer operation |
| `(3, 4)` | `(3,)` | ❌ error: 4 vs 3 |

```python
X = np.array([[1., 2., 3.],
              [4., 5., 6.]])            # (2, 3)
col_means = X.mean(axis=0)              # (3,)
X_centered = X - col_means              # (2,3) - (3,) ✅

row_sums = X.sum(axis=1, keepdims=True) # (2, 1)  keepdims keeps it broadcastable
X_share = X / row_sums                  # each row sums to 1

# Standardisation (z-score) of every feature in one line
Z = (X - X.mean(axis=0)) / X.std(axis=0)
```

---

## 3. Advanced Level: Outer Operations and Pairwise Distances

```python
a = np.array([1, 2, 3])
b = np.array([10, 20])
a[:, None] * b[None, :]       # (3,1)*(1,2) → (3,2) multiplication table

# Pairwise Euclidean distance matrix between N points and M centroids (k-means core)
P = rng.random((1000, 2))
C = rng.random((5, 2))
D = np.sqrt(((P[:, None, :] - C[None, :, :]) ** 2).sum(axis=2))   # (1000, 5)
nearest = D.argmin(axis=1)
```
⚠️ Broadcasting can silently create **huge temporary arrays**: `(100_000, 1, 512) - (1, 100_000, 512)` needs ~40 TB! For big pairwise problems, chunk the data or use `scipy.spatial.distance.cdist`.

`np.broadcast_to(a, (4, 3))` shows the broadcast result as a read-only view; `np.broadcast_shapes((3,1),(1,4))` → `(3, 4)`.

## 🧪 Practice
1. Normalise every column of a matrix to the range [0, 1] using broadcasting.
2. Build a 10×10 multiplication table without loops.
3. Compute the distance from 1,000 customers to 3 warehouses and assign each to the nearest one.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| broadcasting | /ˈbrɑːd.kæs.tɪŋ/ | **BROD**-kas-ting | difusión (expansión automática) |
| trailing | /ˈtreɪ.lɪŋ/ | **TRÉI**-ling | final / del final |
| stretch | /stretʃ/ | strech | estirar |
| compatible | /kəmˈpæt̬.ə.bəl/ | kom-**PA**-ti-bol | compatible |
| pairwise | /ˈper.waɪz/ | **PER**-uaiz | por pares |
| centroid | /ˈsen.trɔɪd/ | **SEN**-troid | centroide |

⬅️ [[numpy_boolean_masking]] · ➡️ [[numpy_linear_algebra_einsum]]
