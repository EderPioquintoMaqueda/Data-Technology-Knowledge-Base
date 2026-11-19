---
id: tensorflow-transformers-attention
title: TensorFlow Module 7 - Attention Mechanisms, Transformers, and Self-Attention
tags: [python, tensorflow, deep-learning, transformers, attention, nlp]
aliases: [TensorFlow Attention, Transformer Architecture, Self-Attention Mechanism]
---

# TensorFlow Module 7: Attention Mechanisms, Transformers & Self-Attention

> [!warning] 🚧 Incomplete note
> Only the beginner section exists (the original text was cut off). Intermediate and advanced sections are pending — tracked in [[Completion and Translation Queue]].


While Recurrent Neural Networks process sequences step-by-step, creating bottlenecks for parallel hardware and struggling with extremely long-term dependencies, the **Transformer** architecture relies entirely on **Self-Attention** to weigh relationships between any tokens in a sequence simultaneously, regardless of their distance.

---

## 1. Beginner Level: Understanding Scaled Dot-Product Attention

The core mathematical engine of a transformer is Scaled Dot-Product Attention. It computes similarity scores between Queries ($Q$) and Keys ($K$), scales them by the square root of the head dimension to prevent vanishing gradients, applies a Softmax function, and multiplies the result by Values ($V$).

```python
import tensorflow as tf

def scaled_dot_product_attention(q, k, v, mask=None):
    """
    Computes scaled dot-product attention.
    Q, K, V have shape: (batch_size, seq_len, depth)
    """
    matmul_qk = tf.matmul(q, k, transpose_b=True) # (..., seq_len_q, seq_len_k)
    
    # Scale scores by square root of key dimension (dk)
    dk = tf.cast(tf.shape(k)[-1], tf.float32)
    scaled_attention_logits = matmul_qk / tf.math.sqrt(dk)
    
    # Apply optional mask (e.g., causal mask for autoregressive generation)
    if mask is not None:
        scaled_attention_logits += (mask * -1e9)
        
    # Softmax normalization across key dimension
    attention_weights = tf.nn.softmax(scaled_attention_logits, axis=-1)
    
    # Output weighted sum of values
    output = tf.matmul(attention_weights, v)
    return output, attention_weights

# Test calculation
q = tf.random.uniform((2, 4, 64))
k = tf.random.uniform((2, 4, 64))
v = tf.random.uniform((2, 4, 64))
out, weights = scaled_dot_product_attention(q, k, v)
print("Attention Output Shape:", out.shape)
print("Attention Weights Shape:", weights.shape)
```
