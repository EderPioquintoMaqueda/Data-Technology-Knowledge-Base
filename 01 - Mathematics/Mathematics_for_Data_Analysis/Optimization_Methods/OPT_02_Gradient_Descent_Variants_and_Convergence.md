---
tags:
  - mathematics/optimization
  - pro
  - gradient-descent
  - machine-learning
  - obsidian-vault
aliases:
  - Gradient Descent Variants and Convergence
---

# Optimization Methods: Gradient Descent Variants and Convergence

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[TC_01_Partial_Derivatives_and_The_Gradient]], [[TC_04_Matrix_Calculus_for_Neural_Networks]], [[OPT_01_Convex_Optimization_and_Lagrange_Multipliers]]

---

## 1. Gradient Descent and Learning Rate Dynamics

Gradient descent is an iterative first-order optimization algorithm used to find the local minimum of a differentiable cost function $J(w)$.

### Mathematical Notation
The parameters $w$ are updated iteratively by stepping in the opposite direction of the gradient $\nabla J(w)$ scaled by a learning rate $\eta$:
$$w_{t+1} = w_t - \eta \nabla J(w_t)$$

* **Too large $\eta$:** The algorithm overshoots the minimum and diverges.
* **Too small $\eta$:** Convergence becomes extremely slow, getting stuck in local plateaus.

### Python Implementation (Vanilla Gradient Descent)
```python
import numpy as np

# Quadratic cost function: J(w) = w^2 - 4w + 4
# Derivative: dJ/dw = 2w - 4
def cost_derivative(w):
  return 2 * w - 4


# Gradient Descent hyperparameters
w = 0.0  # Initial weight guess
learning_rate = 0.1
iterations = 20

# Iterative update loop
for i in range(iterations):
  grad = cost_derivative(w)
  w = w - (learning_rate * grad)
  if i % 4 == 0:
    print(f"Iteration {i}: w = {w:.4f}, Grad = {grad:.4f}")
```
