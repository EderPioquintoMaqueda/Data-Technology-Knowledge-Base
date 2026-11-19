---
id: sklearn-preprocessing
title: Scikit-Learn Module 2 - Feature Preprocessing, Scaling, and Encoding
tags: [python, scikit-learn, machine-learning, preprocessing, scaling, encoding]
aliases: [Scikit-Learn Preprocessing, StandardScaler, OneHotEncoder]
---

# Scikit-Learn Module 2: Feature Preprocessing, Scaling & Encoding

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Raw datasets contain features with wildly different scales (e.g., age ranging from 1 to 100 vs. income ranging from 20,000 to 500,000) and categorical text attributes that machine learning algorithms cannot process mathematically. Preprocessing prepares raw features into clean numerical arrays.

---

## 1. Beginner Level: Numerical Scaling (`StandardScaler` & `MinMaxScaler`)

Many machine learning algorithms (such as Gradient Descent based models, Support Vector Machines, and K-Nearest Neighbors) are sensitive to feature scales. Unscaled features with larger numeric ranges dominate cost functions.

```python
import numpy as np
from sklearn.preprocessing import StandardScaler, MinMaxScaler

# Raw feature matrix (e.g., Age and Annual Salary)
X = np.array([[22, 25000], 
              [35, 60000], 
              [48, 120000]])

# 1. Standardization (Z-score normalization: mean = 0, variance = 1)
# Formula: z = (x - mean) / std
scaler_std = StandardScaler()
X_standardized = scaler_std.fit_transform(X)
print("Standardized Data:\n", X_standardized)

# 2. Min-Max Scaling (Bounding features between a range, typically 0 and 1)
# Formula: x_scaled = (x - min) / (max - min)
scaler_minmax = MinMaxScaler(feature_range=(0, 1))
X_minmax = scaler_minmax.fit_transform(X)
print("Min-Max Scaled Data:\n", X_minmax)
```
