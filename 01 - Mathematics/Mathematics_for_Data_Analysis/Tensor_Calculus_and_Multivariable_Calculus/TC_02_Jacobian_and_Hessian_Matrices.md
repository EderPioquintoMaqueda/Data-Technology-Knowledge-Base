---
tags:
  - mathematics/tensor-calculus
  - intermediate
  - jacobian
  - hessian
  - optimization
  - obsidian-vault
aliases:
  - Jacobian and Hessian Matrices
---

# Tensor Calculus: Jacobian and Hessian Matrices

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[TC_01_Partial_Derivatives_and_The_Gradient]], [[OPT_01_Convex_Optimization_and_Lagrange_Multipliers]], [[OPT_02_Gradient_Descent_Variants_and_Convergence]]

---

## 1. The Jacobian Matrix (First-Order Vector Derivatives)

When a function maps a vector input to a vector output ($f: \mathbb{R}^n \to \mathbb{R}^m$), the collection of all first-order partial derivatives is organized into a matrix called the **Jacobian**. It represents the best linear approximation of a vector-valued function near a given point (crucial in neural network backpropagation and coordinate transformations).

### Mathematical Notation
Let $f(x) = [f_1(x), f_2(x), \dots, f_m(x)]^T$ be a vector function of $n$ variables $x = [x_1, x_2, \dots, x_n]^T$. The Jacobian matrix $J \in \mathbb{R}^{m \times n}$ is defined as:
$$J = \frac{\partial f}{\partial x} = \begin{bmatrix} \frac{\partial f_1}{\partial x_1} & \frac{\partial f_1}{\partial x_2} & \cdots & \frac{\partial f_1}{\partial x_n} \\  \frac{\partial f_2}{\partial x_1} & \frac{\partial f_2}{\partial x_2} & \cdots & \frac{\partial f_2}{\partial x_n} \\  \vdots & \vdots & \ddots & \vdots \\  \frac{\partial f_m}{\partial x_1} & \frac{\partial f_m}{\partial x_2} & \cdots & \frac{\partial f_m}{\partial x_n} \end{bmatrix}$$

### Python Implementation
```python
import numpy as np


# Example vector-valued function: f(x, y) = [x^2 * y, x + y^3]
def vector_function(coords):
  x, y = coords
  f1 = (x**2) * y
  f2 = x + (y**3)
  return np.array([f1, f2], dtype=np.float32)


# Numerical Jacobian calculation using finite differences
def compute_jacobian(func, coords, h=1e-5):
  n = len(coords)
  f_base = func(coords)
  m = len(f_base)
  jacobian = np.zeros((m, n), dtype=np.float32)

  for i in range(n):
    coords_plus = coords.copy()
    coords_minus = coords.copy()
    coords_plus[i] += h
    coords_minus[i] -= h
    jacobian[:, i] = (func(coords_plus) - func(coords_minus)) / (2 * h)

  return jacobian


point = np.array([2.0, 3.0], dtype=np.float32)
J = compute_jacobian(vector_function, point)
print("Jacobian Matrix J at (2, 3):\n", J)
```
