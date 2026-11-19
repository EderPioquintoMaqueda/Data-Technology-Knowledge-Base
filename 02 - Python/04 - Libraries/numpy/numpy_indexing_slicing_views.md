---
id: numpy-indexing-slicing-views
title: NumPy Module 3 - Indexing, Slicing, Views vs. Copies
tags: [python, numpy, data-science, memory-management, slicing]
aliases: [NumPy Slicing, Views vs Copies, Advanced Indexing]
---

# NumPy Module 3: Indexing, Slicing, Views vs. Copies

Efficient data manipulation requires understanding how NumPy accesses subset data. Unlike standard Python lists—which automatically duplicate data when sliced—NumPy slice operations create **Views** to avoid unnecessary memory allocation and copying overhead.

---

## 1. Beginner Level: Basic Indexing & Slicing

Basic slicing in NumPy works similarly to Python lists for 1D arrays, but extends naturally to multi-dimensional matrices along axes.

```python
import numpy as np

# 1D Array indexing
arr_1d = np.array([10, 20, 30, 40, 50])
print(arr_1d[0])   # First element: 10
print(arr_1d[-1])  # Last element: 50
print(arr_1d[1:4]) # Slice from index 1 to 3: [20 30 40]

# 2D Matrix indexing: matrix[row_index, column_index]
matrix = np.array([[1, 2, 3], 
                   [4, 5, 6], 
                   [7, 8, 9]])

print(matrix[0, 2])  # Row 0, Column 2 -> Output: 3
print(matrix[1, :])  # Entire row 1 -> Output: [4 5 6]
print(matrix[:, 0])  # Entire column 0 -> Output: [1 4 7]
```

---

## 2. Intermediate Level: Views vs Copies

```python
a = np.arange(10)
v = a[2:5]          # VIEW → shares memory
v[0] = 99
print(a)            # a[2] is now 99!
c = a[2:5].copy()   # independent COPY
np.shares_memory(a, v), np.shares_memory(a, c)   # True, False
v.base is a         # True → v is a view of a
```
| Operation | Returns |
|---|---|
| basic slicing `a[1:5]`, `a[:, 0]` | **view** |
| `reshape`, `.T`, `ravel` (when possible) | view |
| fancy indexing `a[[0, 2]]` | **copy** |
| boolean masking `a[a > 0]` | copy |
| `flatten()`, `.copy()` | copy |

### Steps and multidimensional slicing
```python
m = np.arange(25).reshape(5, 5)
m[::2, ::2]       # every other row and column
m[1:4, 1:4]       # inner 3×3 block
m[::-1]           # rows reversed
m[..., -1]        # Ellipsis: last column for any number of dims
```

---

## 3. Advanced Level: Fancy Indexing and `np.ix_`

```python
m[[0, 2, 4]]               # rows 0, 2, 4
m[[0, 1], [3, 4]]          # elements (0,3) and (1,4) — pairs!
m[np.ix_([0, 1], [3, 4])]  # the 2×2 sub-matrix rows {0,1} × cols {3,4}

idx = np.argsort(scores)[::-1][:3]    # indices of the top-3 scores
top3 = names[idx]

np.take(m, [0, 2], axis=1)           # columns 0 and 2
np.put(a, [0, 1], [-1, -2])          # write by flat index
```
⚠️ Assigning through fancy indexing with repeated indices does not accumulate: `a[[0, 0]] += 1` adds 1 once → use `np.add.at(a, [0, 0], 1)`.

## 🧪 Practice
1. Prove that modifying a slice changes the original array; then avoid it.
2. From a 10×10 matrix extract the anti-diagonal using fancy indexing.
3. Get the 5 largest values of a 1M-element array efficiently (`np.argpartition`).

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| view | /vjuː/ | viu | vista |
| copy | /ˈkɑː.pi/ | **KA**-pi | copia |
| axis (pl. axes) | /ˈæk.sɪs/ (/ˈæk.siːz/) | **AK**-sis (**AK**-siz) | eje (ejes) |
| fancy indexing | /ˈfæn.si ˈɪn.dek.sɪŋ/ | **FAN**-si **IN**-dek-sing | indexación avanzada |
| share | /ʃer/ | sher | compartir |
| ellipsis | /iˈlɪp.sɪs/ | i-**LIP**-sis | puntos suspensivos |

⬅️ [[numpy_array_creation]] · ➡️ [[numpy_boolean_masking]]
