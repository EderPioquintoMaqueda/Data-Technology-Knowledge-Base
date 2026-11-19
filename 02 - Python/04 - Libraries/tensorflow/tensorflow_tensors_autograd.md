---
id: tensorflow-tensors-autograd
title: TensorFlow Module 1 - Tensors, Variables, and Automatic Differentiation
tags: [python, tensorflow, deep-learning, tensors, autograd]
aliases: [TensorFlow Tensors, tf.Variable, GradientTape Autograd]
---

# TensorFlow Module 1: Tensors, Variables & Automatic Differentiation

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While Scikit-Learn excels at classical machine learning, deep learning requires scalable, multi-dimensional array operations capable of running on hardware accelerators (GPUs/TPUs) with automatic gradient computation. **TensorFlow** is an end-to-end open-source machine learning platform built around tensors and computational graphs.

---

## 1. Beginner Level: Tensors and Constants (`tf.constant`)

A **Tensor** is a generalization of vectors and matrices to n-dimensions. In TensorFlow, `tf.constant` creates immutable multi-dimensional arrays stored in memory.

```python
import tensorflow as tf

# 1. Creating basic tensors
scalar = tf.constant(7)
vector = tf.constant([1.0, 2.0, 3.0])
matrix = tf.constant([[1, 2], 
                      [3, 4]], dtype=tf.float32)

print("Scalar Tensor:", scalar)
print("Vector Shape:", vector.shape)
print("Matrix Datatype:", matrix.dtype)

# Basic tensor arithmetic (element-wise)
tensor_sum = matrix + 10
print("Added Matrix:\n", tensor_sum)
```
