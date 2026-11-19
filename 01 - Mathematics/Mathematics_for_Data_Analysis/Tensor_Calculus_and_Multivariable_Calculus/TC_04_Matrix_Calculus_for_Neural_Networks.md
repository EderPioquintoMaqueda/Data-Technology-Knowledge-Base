---
tags:
  - mathematics/tensor-calculus
  - pro
  - neural-networks
  - backpropagation
  - obsidian-vault
aliases:
  - Matrix Calculus for Neural Networks
---

# Tensor Calculus: Matrix Calculus for Neural Networks and Backpropagation

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Related notes: [[TC_03_Tensor_Algebra_and_Index_Notation]], [[OPT_02_Gradient_Descent_Variants_and_Convergence]]

---

## 1. The Matrix Chain Rule in Deep Learning

Training neural networks requires computing the gradient of a scalar loss function $L$ with respect to every weight matrix $W^{[l]}$ and bias vector $b^{[l]}$ across all layers $l$. This is executed via **backpropagation**, which is a direct application of the chain rule of multivariable calculus.

### Mathematical Notation
Consider a composite function layer:
$$Z = XW + b$$
$$A = f(Z)$$
$$L = g(A)$$

Given the gradient of the loss with respect to the layer activation outputs $\frac{\partial L}{\partial A}$, the chain rule computes the gradients with respect to weights $W$, bias $b$, and inputs $X$:
1. **Gradient w.r.t Pre-activation ($Z$):**
   $$\frac{\partial L}{\partial Z} = \frac{\partial L}{\partial A} \odot f'(Z)$$
   *(where $\odot$ denotes the Hadamard element-wise product).*

2. **Gradient w.r.t Weight Matrix ($W$):**
   $$\frac{\partial L}{\partial W} = X^T \frac{\partial L}{\partial Z}$$

3. **Gradient w.r.t Bias Vector ($b$):**
   $$\frac{\partial L}{\partial b} = \sum_{\text{batch}} \frac{\partial L}{\partial Z}$$

4. **Gradient w.r.t Input ($X$ - for propagating backward to previous layers):**
   $$\frac{\partial L}{\partial X} = \frac{\partial L}{\partial Z} W^T$$

---

## 2. Python Implementation: Forward and Backward Pass of a Linear Layer

```python
import numpy as np

# Set random seed for reproducibility
np.random.seed(42)

# Dimensions
batch_size, in_features, out_features = 4, 3, 2

# Simulated inputs (X) and parameters (W, b)
X = np.random.randn(batch_size, in_features).astype(np.float32)
W = np.random.randn(in_features, out_features).astype(np.float32)
b = np.zeros((1, out_features), dtype=np.float32)

# --- FORWARD PASS ---
Z = np.dot(X, W) + b  # Pre-activation
A = np.maximum(0, Z)  # ReLU activation function

print("Forward Pass Output (A):\n", A)

# --- BACKWARD PASS ---
# Suppose we receive the gradient of loss with respect to activation A from the next layer
dL_dA = np.ones_relu = np.random.randn(batch_size, out_features).astype(np.float32)

# 1. Gradient through ReLU activation: derivative of max(0, Z) is 1 if Z > 0 else 0
dL_dZ = dL_dA * (Z > 0)

# 2. Gradients with respect to weights, biases, and input X
dL_dW = np.dot(X.T, dL_dZ)
dL_db = np.sum(dL_dZ, axis=0, keepdims=True)
dL_dX = np.dot(dL_dZ, W.T)

print("\n--- Backward Gradients ---")
print("Gradient w.r.t Weights (dL/dW):\n", dL_dW)
print("Gradient w.r.t Biases (dL/db):\n", dL_db)
print("Gradient w.r.t Input (dL/dX):\n", dL_dX)
```
