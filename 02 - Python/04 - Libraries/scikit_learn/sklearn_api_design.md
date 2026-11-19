---
id: sklearn-api-design
title: Scikit-Learn Module 1 - Unified API Design, Estimators, and Transformers
tags: [python, scikit-learn, machine-learning, api-design, estimators]
aliases: [Scikit-Learn Architecture, Estimator and Transformer API]
---

# Scikit-Learn Module 1: Unified API Design, Estimators & Transformers

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While NumPy, Pandas, and Matplotlib handle computation and visualization, **Scikit-Learn** is the premier library for machine learning in Python. Its success relies on a strict, elegant, and consistent object-oriented API design shared across every algorithm.

---

## 1. Beginner Level: The Estimator Interface (`fit` and `predict`)

Every machine learning model in Scikit-Learn is implemented as an **Estimator**. An estimator learns parameters from data via the `.fit()` method.

```python
import numpy as np
from sklearn.linear_model import LinearRegression

# 1. Prepare training data (Features X must be 2D, Target y must be 1D or 2D)
X_train = np.array([[1], [2], [3], [4], [5]])
y_train = np.array([2, 4, 6, 8, 10]) # y = 2 * x

# 2. Instantiate the estimator
model = LinearRegression()

# 3. Fit the model to the training data (Learns internal coefficients/weights)
model.fit(X_train, y_train)

# 4. Predict on new unseen data using .predict()
X_new = np.array([[6], [7]])
predictions = model.predict(X_new)
print("Predictions:", predictions) # Output: [12. 14.]

# Inspecting learned model parameters (attributes ending with trailing underscore)
print("Learned Coefficient (Slope):", model.coef_)
print("Learned Intercept:", model.intercept_)
```
