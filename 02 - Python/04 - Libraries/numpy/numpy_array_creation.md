---
id: numpy-array-creation
title: NumPy Module 2 - Specialized Array Creation and Initializers
tags: [python, numpy, data-science, array-creation, initialization]
aliases: [NumPy Initializers, Array Creation Methods]
---

# NumPy Module 2: Specialized Array Creation & Initializers

While `np.array()` converts standard Python sequences into NumPy arrays, initializing large datasets manually via nested lists is inefficient and slow. NumPy provides optimized factory functions to instantiate arrays directly in memory with specific shapes, dtypes, and memory patterns.

---

## 1. Beginner Level: Basic Initializers

When setting up placeholders, masks, or identity matrices for mathematical operations, use NumPy's built-in shape-based initializers.

```python
import numpy as np

# 1. Zeros matrix (useful for initialization and zero-padding)
zeros_arr = np.zeros((3, 4), dtype=np.float64)
print("Zeros Matrix:\n", zeros_arr)

# 2. Ones matrix (useful for scaling and weight initialization)
ones_arr = np.ones((2, 3), dtype=np.int32)
print("Ones Matrix:\n", ones_arr)

# 3. Constant filled matrix with full()
constant_arr = np.full((2, 2), fill_value=7.14)
print("Constant Matrix:\n", constant_arr)
```

---

## 2. Intermediate Level: Ranges, Grids, Random and Reshape

```python
np.arange(0, 10, 2)                 # [0 2 4 6 8]       step-based
np.linspace(0, 1, 5)                # [0. .25 .5 .75 1.] count-based (includes end)
np.logspace(0, 3, 4)                # [1 10 100 1000]
np.eye(3)                           # identity matrix
np.zeros_like(other), np.ones_like(other), np.empty((2, 3))   # empty = uninitialised (fast)

xx, yy = np.meshgrid(np.linspace(-1, 1, 3), np.linspace(-1, 1, 3))   # coordinate grids

rng = np.random.default_rng(seed=42)          # modern, reproducible generator
rng.random((2, 3))                            # uniform [0, 1)
rng.normal(loc=100, scale=15, size=1000)      # normal distribution
rng.integers(1, 7, size=10)                   # dice rolls
rng.choice(["A", "B", "C"], size=5, p=[.5, .3, .2])
rng.permutation(10)

a = np.arange(12)
a.reshape(3, 4)          # new view with a different shape
a.reshape(-1, 6)         # -1 = "infer this dimension"
a.reshape(3, 4).ravel()  # flatten (view if possible)
a[:, np.newaxis]         # (12,) → (12, 1)
np.expand_dims(a, 0)     # (12,) → (1, 12)
```

### Combining arrays
```python
np.concatenate([a, b], axis=0)
np.vstack([a, b]); np.hstack([a, b]); np.stack([a, b], axis=0)   # stack adds a NEW axis
np.split(np.arange(9), 3)
```

---

## 3. Advanced Level: Structured Arrays, I/O and Performance

```python
# Structured (record) arrays — typed columns without pandas
dt = np.dtype([("id", "i4"), ("price", "f8"), ("sku", "U10")])
rec = np.array([(1, 9.99, "A-01"), (2, 4.50, "B-07")], dtype=dt)
rec["price"].mean()

# Saving / loading
np.save("x.npy", a); np.load("x.npy")                     # binary, exact dtype
np.savez_compressed("bundle.npz", X=X, y=y)
np.loadtxt("data.csv", delimiter=",", skiprows=1)          # small text files only

# Build arrays without Python loops
np.fromiter((x * x for x in range(1000)), dtype=np.int64, count=1000)
np.fromfunction(lambda i, j: i * 10 + j, (3, 4), dtype=int)
```
💡 Pre-allocate (`np.empty(n)`) and fill, instead of `np.append` in a loop (which copies the whole array every time → O(n²)).

## 🧪 Practice
1. Generate 1,000 simulated daily sales with mean 500 and sd 80 (seeded).
2. Create a 5×5 matrix with 1 on the border and 0 inside.
3. Reshape `np.arange(24)` into shape `(2, 3, 4)` and explain each axis.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| initialise | /ɪˈnɪʃ.əl.aɪz/ | i-**NI**-sho-laiz | inicializar |
| evenly spaced | /ˈiː.vən.li speɪst/ | **I**-ven-li speist | espaciado uniforme |
| seed | /siːd/ | sid | semilla |
| reshape | /riːˈʃeɪp/ | ri-**SHÉIP** | cambiar forma |
| infer | /ɪnˈfɝː/ | in-**FER** | inferir |
| stack | /stæk/ | stak | apilar |
| pre-allocate | /priː ˈæl.ə.keɪt/ | pri-**A**-lo-keit | reservar de antemano |

⬅️ [[numpy_memory_architecture]] · ➡️ [[numpy_indexing_slicing_views]]
