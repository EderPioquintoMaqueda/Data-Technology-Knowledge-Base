---
id: tensorflow-functional-api
title: TensorFlow Module 4 - The Functional API, Multi-Input/Output Architectures, and Shared Layers
tags: [python, tensorflow, deep-learning, keras, functional-api]
aliases: [TensorFlow Functional API, Multi-Input Models, Shared Layers]
---

# TensorFlow Module 4: The Functional API, Multi-Input/Output Architectures & Shared Layers

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While Keras's `Sequential` API is ideal for linear stacks of layers, real-world deep learning architectures often require complex topologies—such as multi-input models (e.g., merging tabular metadata with image features), multi-output models, residual connections, and shared layers. The **Functional API** provides the flexibility needed to build these advanced graphs.

---

## 1. Beginner Level: Building a Functional Model

Unlike the `Sequential` API where layers are passed as a python list, the Functional API explicitly connects layers by calling a layer instance on a tensor input, returning a tensor output.

```python
import tensorflow as tf
from tensorflow.keras import Model
from tensorflow.keras.layers import Input, Dense

# 1. Define the input tensor explicitly with expected shape
inputs = Input(shape=(10,), name='feature_input')

# 2. Pass tensors through layers (Layer instance is called as a function)
x = Dense(64, activation='relu')(inputs)
x = Dense(32, activation='relu')(x)

# 3. Output layer
outputs = Dense(1, activation='sigmoid', name='prediction_output')(inputs)

# 4. Instantiate the Model by specifying inputs and outputs
model = Model(inputs=inputs, outputs=outputs, name='functional_mlp')

print(model.summary())
```
