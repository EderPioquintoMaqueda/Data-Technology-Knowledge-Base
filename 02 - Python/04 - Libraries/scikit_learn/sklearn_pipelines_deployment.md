---
id: sklearn-pipelines-deployment
title: Scikit-Learn Module 8 - Pipelines, ColumnTransformers, and Model Persistence
tags: [python, scikit-learn, machine-learning, pipelines, deployment, joblib]
aliases: [Scikit-Learn Pipelines, ColumnTransformer, Model Persistence Joblib]
---

# Scikit-Learn Module 8: Pipelines, ColumnTransformers & Model Persistence

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


In production environments, machine learning code must be modular, reproducible, and immune to data leakage. Applying scaling, imputation, encoding, and modeling steps separately across training and test sets introduces bugs. Scikit-Learn provides **Pipelines** and **ColumnTransformers** to encapsulate entire preprocessing and modeling workflows into a single unified estimator object.

---

## 1. Beginner Level: Model Persistence with `joblib`

Once a model is trained and evaluated, you must serialize it to disk so it can be loaded into production web servers or inference APIs without retraining. `joblib` is optimized specifically for serializing Python objects containing large NumPy arrays.

```python
import joblib
from sklearn.linear_model import LogisticRegression
import numpy as np

# Sample training data
X = np.array([[1, 2], [3, 4], [5, 6]])
y = np.array([0, 0, 1])

model = LogisticRegression()
model.fit(X, y)

# 1. Save trained model to disk
joblib.dump(model, 'production_model.joblib')
print("Model successfully saved to disk.")

# 2. Load model back into memory for inference
loaded_model = joblib.load('production_model.joblib')
predictions = loaded_model.predict([[2, 3]])
print("Inference Predictions from Loaded Model:", predictions)
```
