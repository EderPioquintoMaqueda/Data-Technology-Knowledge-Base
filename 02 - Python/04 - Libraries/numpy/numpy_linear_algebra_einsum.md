---
id: numpy-linear-algebra-einsum
title: NumPy Module 6 - Linear Algebra, Matrix Operations, and Einstein Summation
tags: [python, numpy, linear-algebra, matrix-multiplication, einsum]
aliases: [NumPy Linear Algebra, Einstein Summation, Matrix Operations]
---

# NumPy Module 6: Linear Algebra, Matrix Operations & Einstein Summation

Linear algebra is the mathematical backbone of data science, machine learning, and neural networks. NumPy provides a robust linear algebra submodule (`np.linalg`) alongside high-performance tensor contraction tools.

---

## 1. Beginner Level: Basic Matrix Multiplication & Transposition

In NumPy, the asterisk operator (`*`) performs **element-wise multiplication** (Hadamard product), not matrix multiplication. To perform true matrix multiplication, use the dot product or the `@` operator.

```python
import numpy as np

A = np.array([[1, 2], 
              [3, 4]])

B = np.array([[5, 6], 
              [7, 8]])

# Element-wise multiplication (NOT matrix multiplication)
hadamard = A * B
print("Hadamard Product:\n", hadamard)

# True Matrix Multiplication (Dot Product)
# Method 1: @ operator (Python 3.5+)
matmul_1 = A @ B

# Method 2: np.dot()
matmul_2 = np.dot(A, B)

# Transposing a matrix
transposed = A.T
print("Transposed Matrix:\n", transposed)
```

---

## 2. Intermediate Level: `np.linalg`

```python
A = np.array([[3., 1.], [1., 2.]])
b = np.array([9., 8.])

np.linalg.det(A)            # determinant → 5.0
np.linalg.inv(A)            # inverse (avoid when possible)
x = np.linalg.solve(A, b)   # solves Ax = b → [2., 3.]  (faster & more stable than inv(A) @ b)
np.linalg.norm(b)           # Euclidean length
np.linalg.matrix_rank(A)
vals, vecs = np.linalg.eig(A)        # eigen-decomposition → [[AL - Eigenvalues y Eigenvectors]]
U, S, Vt = np.linalg.svd(A)           # singular value decomposition → [[LA_04_SVD_and_Principal_Component_Analysis]]

# Least squares: fit y = w0 + w1*x
x_data = np.array([1, 2, 3, 4]); y = np.array([2.1, 3.9, 6.2, 7.8])
X = np.column_stack([np.ones_like(x_data), x_data])
w, *_ = np.linalg.lstsq(X, y, rcond=None)     # [intercept, slope]
```

---

## 3. Advanced Level: `einsum` and Batched Operations

**Einstein summation** describes tensor operations with index letters: indices that disappear from the output are summed.

| Operation | einsum | Equivalent |
|---|---|---|
| Sum of all elements | `np.einsum("ij->", A)` | `A.sum()` |
| Transpose | `np.einsum("ij->ji", A)` | `A.T` |
| Matrix product | `np.einsum("ik,kj->ij", A, B)` | `A @ B` |
| Dot product | `np.einsum("i,i->", u, v)` | `u @ v` |
| Outer product | `np.einsum("i,j->ij", u, v)` | `np.outer(u, v)` |
| Diagonal | `np.einsum("ii->i", A)` | `np.diag(A)` |
| Batched matmul | `np.einsum("bij,bjk->bik", X, Y)` | `X @ Y` |
| Attention scores | `np.einsum("bqd,bkd->bqk", Q, K)` | `Q @ K.transpose(0,2,1)` |

```python
Q = rng.random((8, 10, 64)); K = rng.random((8, 10, 64))
scores = np.einsum("bqd,bkd->bqk", Q, K) / np.sqrt(64)    # (8, 10, 10)
np.einsum("bqd,bkd->bqk", Q, K, optimize=True)             # let NumPy find the best contraction order
```
The `@` operator also broadcasts over leading batch dimensions. Under the hood NumPy calls optimised **BLAS/LAPACK** (OpenBLAS, MKL) libraries, which are multi-threaded.

Theory links: [[LA_02_Matrices_and_Linear_Transformations]] · [[TC_03_Tensor_Algebra_and_Index_Notation]] · [[tensorflow_transformers_attention]].

## 🧪 Practice
1. Solve a 3×3 linear system with `solve` and verify with `A @ x`.
2. Fit a linear regression with `lstsq` and compare to scikit-learn.
3. Rewrite three operations of your choice with `einsum`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| matrix (pl. matrices) | /ˈmeɪ.trɪks/ (/ˈmeɪ.trɪ.siːz/) | **MÉI**-triks (**MÉI**-tri-siz) | matriz (matrices) |
| determinant | /dɪˈtɝː.mɪ.nənt/ | di-**TER**-mi-nant | determinante |
| inverse | /ɪnˈvɝːs/ | in-**VERS** | inversa |
| eigenvalue | /ˈaɪ.ɡənˌvæl.juː/ | **ÁI**-guen-val-yu | valor propio |
| least squares | /liːst skwerz/ | list skuerz | mínimos cuadrados |
| contraction | /kənˈtræk.ʃən/ | kon-**TRAK**-shon | contracción |
| dot product | /dɑːt ˈprɑː.dʌkt/ | dat **PRA**-dakt | producto punto |

⬅️ [[numpy_broadcasting_engine]] · ➡️ [[pandas_series_dataframe_internals]] · 🏠 [[Python Libraries Map]]
