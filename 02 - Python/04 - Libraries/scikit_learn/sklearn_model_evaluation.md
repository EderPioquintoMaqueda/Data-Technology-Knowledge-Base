---
id: sklearn-model-evaluation
title: Scikit-Learn Module 7 - Model Evaluation, Cross-Validation, and Performance Metrics
tags: [python, scikit-learn, machine-learning, model-evaluation, cross-validation, metrics]
aliases: [Scikit-Learn Evaluation, Cross-Validation, ROC AUC and Confusion Matrix]
---

# Scikit-Learn Module 7: Model Evaluation, Cross-Validation & Performance Metrics

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Building a machine learning model is only half the battle; rigorously evaluating its generalization performance on unseen data is critical to prevent overfitting and deployment failure. Scikit-Learn provides robust utilities for splitting datasets, cross-validation, and performance metrics.

---

## 1. Beginner Level: Train-Test Split & Basic Regression/Classification Metrics

Never evaluate a model on the same data used to train it. The `train_test_split` function partitions data into randomized training and testing subsets.

```python
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report, mean_squared_error

# Synthetic dataset
X = np.random.randn(200, 5)
y = (X[:, 0] + X[:, 1] > 0).astype(int)

# 1. Train-Test Split (80% training, 20% testing, stratified for balanced class distribution)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 2. Train classifier
model = LogisticRegression()
model.fit(X_train, y_train)

# 3. Predict and evaluate
y_pred = model.predict(X_test)
print("Accuracy Score:", accuracy_score(y_test, y_pred))
print("Classification Report:\n", classification_report(y_test, y_pred))
```
