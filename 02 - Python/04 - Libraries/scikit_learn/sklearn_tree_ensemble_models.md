---
id: sklearn-tree-ensemble-models
title: Scikit-Learn Module 5 - Decision Trees, Random Forests, and Gradient Boosting
tags: [python, scikit-learn, machine-learning, decision-trees, random-forest, gradient-boosting]
aliases: [Scikit-Learn Tree Models, Random Forest, Gradient Boosting]
---

# Scikit-Learn Module 5: Decision Trees, Random Forests & Gradient Boosting

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While linear models capture linear relationships, real-world datasets often feature complex, non-linear interactions. Tree-based models partition feature spaces into orthogonal rectangles, serving as the foundation for powerful ensemble algorithms like Random Forests and Gradient Boosting.

---

## 1. Beginner Level: Decision Trees (`DecisionTreeClassifier` & `DecisionTreeRegressor`)

A decision tree recursively splits data based on feature thresholds that maximize information gain (using metrics like Gini impurity or entropy for classification, and Mean Squared Error for regression).

```python
from sklearn.datasets import load_iris
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score

# Load sample dataset
iris = load_iris()
X, y = iris.data, iris.target

# Instantiate and fit Decision Tree classifier
tree_clf = DecisionTreeClassifier(max_depth=3, random_state=42)
tree_clf.fit(X, y)

# Predictions and evaluation
y_pred = tree_clf.predict(X)
print("Decision Tree Accuracy:", accuracy_score(y, y_pred))
```
