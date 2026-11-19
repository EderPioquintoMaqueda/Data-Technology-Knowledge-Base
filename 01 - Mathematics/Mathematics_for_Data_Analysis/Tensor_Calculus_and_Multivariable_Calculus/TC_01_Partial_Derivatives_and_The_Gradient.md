---
tags:
  - mathematics/tensor-calculus
  - beginner
  - gradients
  - optimization
  - obsidian-vault
aliases:
  - Partial Derivatives and The Gradient
---

# Tensor Calculus: Partial Derivatives and The Gradient

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[LA_04_SVD_and_Principal_Component_Analysis]], [[TC_02_Jacobian_and_Hessian_Matrices]], [[OPT_02_Gradient_Descent_Variants_and_Convergence]]

---

## 1. Partial Derivatives

In data science and machine learning, cost functions $f(w_1, w_2, \dots, w_n)$ depend on multiple parameters (weights and biases). A partial derivative measures the rate of change of a multivariable function with respect to a single variable while holding all other variables constant.

### Mathematical Notation
For a function $f: \mathbb{R}^n \to \mathbb{R}$, the partial derivative with respect to the $i$-th variable $x_i$ is defined as:
$$\frac{\partial f}{\partial x_i} = \lim_{h \to 0} \frac{f(x_1, \dots, x_i + h, \dots, x_n) - f(x_1, \dots, x_i, \dots, x_n)}{h}$$

### Python Implementation (Numerical Approximation)
```python
import numpy as np


# Define a multivariable cost function: f(x, y) = x^2 + 3xy + y^2
def cost_function(x, y):
  return x**2 + 3 * x * y + y**2


# Numerical partial derivative with respect to x
def partial_derivative_x(x, y, h=1e-5):
  return (cost_function(x + h, y) - cost_function(x - h, y)) / (2 * h)


x_val, y_val = 2.0, 3.0
print(f"Partial derivative w.r.t x at (2, 3): {partial_derivative_x(x_val, y_val):.4f}")
```
