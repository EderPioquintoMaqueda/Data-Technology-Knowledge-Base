---
tags:
  - mathematics/tensor-calculus
  - advanced
  - tensors
  - einstein-summation
  - obsidian-vault
aliases:
  - Tensor Algebra and Index Notation
---

# Tensor Calculus: Tensor Algebra and Index Notation

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[TC_02_Jacobian_and_Hessian_Matrices]], [[LA_01_Vectors_Spaces_and_Norms]]

---

## 1. What is a Tensor? (Rank Definition)

In data science and physics, a tensor is a multi-dimensional array of numerical values that transforms according to specific rules under coordinate changes. We classify tensors by their **rank** (or order):
* **Rank 0:** A scalar (single number, e.g., temperature).
* **Rank 1:** A vector (1D array of numbers, e.g., feature embedding).
* **Rank 2:** A matrix (2D grid of numbers, e.g., covariance matrix or weight layer).
* **Rank 3+:** Higher-order tensors (e.g., batch of RGB images of shape `[Batch, Height, Width, Channels]`).

---

## 2. Einstein Summation Convention (Index Notation)

To write complex linear algebra and tensor operations compactly without writing repetitive summation signs ($\sum$), physicist Albert Einstein introduced the **Summation Convention**: *whenever a tensor index appears twice in a single term (once as a superscript/subscript pair), summation over that index is implied.*

### Mathematical Notation
Instead of writing the matrix multiplication product $C_{ij} = \sum_{k} A_{ik} B_{kj}$, we write:
$$C_{ij} = A_{ik} B_{kj}$$
Where index $k$ is a **dummy index** (contracted over), and indices $i$ and $j$ are **free indices**.

---

## 3. Common Tensor Operations in Index Notation

1. **Inner Product (Dot Product):** $u_i v_i = \sum_{i} u_i v_i$
2. **Matrix-Vector Multiplication:** $y_i = A_{ij} x_j$
3. **Matrix-Matrix Multiplication:** $C_{ik} = A_{ij} B_{jk}$
4. **Outer Product:** $T_{ij} = u_i v_j$

### Python Implementation via `np.einsum`
NumPy provides `np.einsum`, which implements Einstein summation directly in Python. It is extremely powerful for optimizing complex multi-dimensional tensor contractions without manually reshaping or transposing arrays.

```python
import numpy as np

# Rank-2 Tensors (Matrices) A and B
A = np.array([[1.0, 2.0], [3.0, 4.0]], dtype=np.float32)
B = np.array([[5.0, 6.0], [7.0, 8.0]], dtype=np.float32)

# 1. Matrix Multiplication using Einstein Summation: C_ik = A_ij * B_jk
# 'ik,kj->ij' means: match index i,k from A and k,j from B, contracting over k.
matmul_result = np.einsum("ik,kj->ij", A, B)
print("Matrix Multiplication via einsum:\n", matmul_result)

# 2. Trace of a Matrix (Sum of diagonal elements): A_ii = sum_i A_ii
matrix_trace = np.einsum("ii->", A)
print("Matrix Trace:", matrix_trace)  # 1.0 + 4.0 = 5.0

# 3. Batch Matrix Multiplication (e.g., in Transformer Attention mechanisms)
# Shape: [Batch_Size, Seq_Len, Feature_Dim]
batch_a = np.random.rand(16, 10, 32).astype(np.float32)
batch_b = np.random.rand(16, 32, 64).astype(np.float32)

# 'blc,bcf->blf' -> batch, length, channels multiplied by batch, channels, features
batch_matmul = np.einsum("blc,bcf->blf", batch_a, batch_b)
print("Batch Matrix Multiplication Shape:", batch_matmul.shape)
```
