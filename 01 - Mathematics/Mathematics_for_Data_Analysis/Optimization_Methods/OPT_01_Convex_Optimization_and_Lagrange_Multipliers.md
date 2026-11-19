---
tags:
  - mathematics/optimization
  - beginner
  - convex-optimization
  - lagrange-multipliers
  - obsidian-vault
aliases:
  - Convex Optimization and Lagrange Multipliers
---

# Optimization Methods: Convex Optimization and Lagrange Multipliers

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[TC_01_Partial_Derivatives_and_The_Gradient]], [[OPT_02_Gradient_Descent_Variants_and_Convergence]]

---

## 1. Convexity: The Holy Grail of Optimization

In machine learning and data science, optimization problems are much easier to solve when they are **convex**. A convex optimization problem guarantees that any local minimum is also a global minimum.

### Mathematical Notation
* **Convex Set:** A set $C$ is convex if, for any two points $x, y \in C$ and any scalar $\theta \in [0, 1]$, the line segment connecting them lies entirely within $C$:
  $$\theta x + (1 - \theta)y \in C$$

* **Convex Function:** A function $f: \mathbb{R}^n \to \mathbb{R}$ is convex if its domain is a convex set and for all $x, y$ in its domain and $\theta \in [0, 1]$:
  $$f(\theta x + (1 - \theta)y) \le \theta f(x) + (1 - \theta)f(y)$$

```python
import numpy as np

# A simple convex function: f(x) = x^2 + 5
# Its second derivative (Hessian) is 2 > 0 everywhere, proving strict convexity.
def convex_func(x):
  return x**2 + 5
```
