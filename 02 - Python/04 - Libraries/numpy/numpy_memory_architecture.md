---
id: numpy-memory-architecture
title: NumPy Module 1 - Internal Memory Architecture and Data Types
tags: [python, numpy, data-science, memory-layout, c-python]
aliases: [NumPy Memory Layout, ndarray Architecture]
---

# NumPy Module 1: Internal Memory Architecture & Data Types

Understanding how NumPy stores data under the hood is what separates a beginner Python user from a professional data engineer. While standard Python lists store scattered object pointers in heap memory, NumPy `ndarray` objects rely on contiguous blocks of homogenous memory managed at the C level.

---

## 1. Beginner Syntax: The Fundamental Difference

In standard Python, a list is an array of pointers to Python objects scattered across memory. This incurs heavy pointer-chasing overhead and type-checking penalties.

```python
import sys

# Standard Python list of integers
py_list = [1, 2, 3, 4, 5]
print("Python list size (bytes):", sys.getsizeof(py_list))
```

```python
import numpy as np
np_arr = np.array([1, 2, 3, 4, 5], dtype=np.int64)
print("NumPy data bytes:", np_arr.nbytes)      # 40 bytes = 5 × 8 bytes
print("itemsize:", np_arr.itemsize, "| dtype:", np_arr.dtype)
```

---

## 2. Intermediate Level: dtypes, Shape and Strides

Every `ndarray` is described by **metadata** that points into one raw memory buffer:

| Attribute | Meaning | Example `(3, 4)` float64 |
|---|---|---|
| `dtype` | type of every element | `float64` |
| `shape` | size of each dimension | `(3, 4)` |
| `ndim` | number of dimensions | `2` |
| `itemsize` | bytes per element | `8` |
| `strides` | bytes to jump to the next element in each axis | `(32, 8)` |
| `flags` | C-contiguous? Fortran? writeable? | `C_CONTIGUOUS: True` |

```python
a = np.arange(12, dtype=np.float64).reshape(3, 4)
a.strides            # (32, 8): next row = 4 elements × 8 bytes
a.T.strides          # (8, 32): transpose = same memory, swapped strides (no copy!)
a.flags["C_CONTIGUOUS"], a.T.flags["C_CONTIGUOUS"]   # True, False
```

### Choosing dtypes to save memory
| dtype | Bytes | Range / precision |
|---|---|---|
| `int8` / `uint8` | 1 | -128..127 / 0..255 (images) |
| `int32` | 4 | ±2.1 billion |
| `int64` | 8 | default integer |
| `float32` | 4 | ~7 decimal digits (deep learning) |
| `float64` | 8 | ~15 digits (default, statistics) |
| `bool` | 1 | True/False |

```python
big = np.random.rand(10_000_000)          # 80 MB
small = big.astype(np.float32)            # 40 MB
np.iinfo(np.int16), np.finfo(np.float32)  # limits of each type
```
⚠️ **Overflow is silent** for integer arrays: `np.array([127], dtype=np.int8) + 1` → `-128`.

---

## 3. Advanced Level: Contiguity, Cache and Zero-Copy

- CPUs read memory in **cache lines** (~64 bytes). Iterating along the **last axis** of a C-ordered array is fastest.
- `np.ascontiguousarray(x)` fixes non-contiguous arrays before heavy loops or passing them to C/BLAS.
- `np.lib.stride_tricks.sliding_window_view(x, 7)` creates rolling windows **without copying**.
- **Memory-mapped files** let you work with arrays larger than RAM:

```python
mm = np.memmap("big.dat", dtype="float32", mode="w+", shape=(100_000_000,))
mm[:1000] = 1.0; mm.flush()
```
- **Arrow / buffer protocol:** pandas, Polars, PyTorch (`torch.from_numpy`) can share the same memory buffer → zero-copy hand-offs.

## 🧪 Practice
1. Compare `sys.getsizeof(list(range(10**6)))` with `np.arange(10**6).nbytes`.
2. Show that `a.T` shares memory with `a` using `np.shares_memory`.
3. Time summing rows vs columns of a 5000×5000 array and explain the difference.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| contiguous | /kənˈtɪɡ.ju.əs/ | kon-**TI**-guiu-os | contiguo |
| pointer | /ˈpɔɪn.t̬ɚ/ | **POIN**-ter | puntero |
| stride | /straɪd/ | straid | paso / salto |
| overflow | /ˌoʊ.vɚˈfloʊ/ | ou-ver-**FLOU** | desbordamiento |
| buffer | /ˈbʌf.ɚ/ | **BA**-fer | búfer |
| homogeneous | /ˌhoʊ.moʊˈdʒiː.ni.əs/ | jou-mo-**YI**-ni-os | homogéneo |
| zero-copy | /ˈzɪr.oʊ ˈkɑː.pi/ | **ZI**-rou **KA**-pi | sin copia |

⬅️ Start of NumPy path · ➡️ [[numpy_array_creation]] · 🏠 [[Python Libraries Map]]
