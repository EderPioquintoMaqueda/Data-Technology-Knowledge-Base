---
id: tensorflow-sequential-api
title: TensorFlow Module 2 - The Sequential API, Dense Layers, and Model Compilation
tags: [python, tensorflow, deep-learning, keras, neural-networks]
aliases: [TensorFlow Sequential API, Keras Dense Layers, Model Compilation]
---

# TensorFlow Module 2: The Sequential API, Dense Layers & Model Compilation

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


Building deep neural networks in TensorFlow is streamlined using **Keras**, TensorFlow's high-level neural networks API. The `Sequential` model allows you to build deep learning architectures layer-by-layer in a straightforward, linear stack.

---

## 1. Beginner Level: Building a Sequential Model (`tf.keras.Sequential`)

A sequential model is appropriate for a plain stack of layers where each layer has exactly one input tensor and one output tensor.

```python
import tensorflow as tf
from tensorflow.keras import Sequential
from tensorflow.keras.layers import Dense

# Instantiate a Sequential model
model = Sequential([
    # Hidden Layer 1: 64 neurons, ReLU activation, expecting 10 input features
    Dense(64, activation='relu', input_shape=(10,)),
    
    # Hidden Layer 2: 32 neurons, ReLU activation
    Dense(32, activation='relu'),
    
    # Output Layer: 1 neuron (for binary classification or regression), linear or sigmoid activation
    Dense(1, activation='sigmoid')
])

print(model.summary())
```
