---
id: tensorflow-convolutional-neural-networks
title: TensorFlow Module 5 - Convolutional Neural Networks (CNNs) and Spatial Feature Extraction
tags: [python, tensorflow, deep-learning, cnn, computer-vision, convolutional-layers]
aliases: [TensorFlow CNNs, Conv2D Layers, Image Classification Architectures]
---

# TensorFlow Module 5: Convolutional Neural Networks (CNNs) & Spatial Feature Extraction

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While standard dense (Fully Connected) layers treat input pixels as independent features, spatial data like images contain strong local pixel correlations. **Convolutional Neural Networks (CNNs)** apply sliding filter kernels across spatial dimensions to automatically learn translation-invariant feature hierarchies—from edges and textures in early layers to complex shapes in deeper layers.

---

## 1. Beginner Level: Core Convolutional & Pooling Layers (`Conv2D` & `MaxPooling2D`)

A CNN architecture alternates between convolutional feature extraction layers and spatial downsampling pooling layers before passing flattened representations into dense classification heads.

```python
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Conv2D, MaxPooling2D, Flatten, Dense

# Instantiate a Sequential CNN for 28x28 grayscale images (e.g., MNIST digit classification)
model = Sequential([
    # 1. Convolutional Layer: 32 filters of size (3x3), ReLU activation
    Conv2D(32, kernel_size=(3, 3), activation='relu', input_shape=(28, 28, 1)),
    
    # 2. Max Pooling Layer: Downsamples spatial dimensions by taking maximum over 2x2 windows
    MaxPooling2D(pool_size=(2, 2)),
    
    # 3. Flatten 2D spatial feature maps into a 1D vector for dense layers
    Flatten(),
    
    # 4. Fully Connected Dense Classification Head
    Dense(64, activation='relu'),
    Dense(10, activation='softmax') # 10 output classes
])

model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
print(model.summary())
```
