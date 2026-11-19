---
id: numpy-boolean-masking
title: NumPy Module 4 - Boolean Masking and Conditional Filtering
tags: [python, numpy, data-science, filtering, boolean-masking]
aliases: [NumPy Filtering, Boolean Indexing, Conditional Logic]
---

# NumPy Module 4: Boolean Masking & Conditional Filtering

Filtering data dynamically based on conditions is essential for data cleaning and exploratory data analysis. In pure Python, this requires explicit `for` loops and `if` statements, which are extremely slow. NumPy uses **Boolean Masking** (vectorized element-wise comparison) to filter large arrays instantly in C.

---

## 1. Beginner Level: Element-Wise Comparisons

When you apply a comparison operator to a NumPy array, it evaluates every single element simultaneously and returns a boolean array of the exact same shape.

```python
import numpy as np

arr = np.array([10, 25, 30, 45, 50])

# Condition: Which elements are greater than 30?
mask = arr > 30
print("Boolean Mask:", mask) 
# Output: [False False False  True  True]
```

```python
arr[mask]                 # [45 50]  → keep only True positions
arr[arr > 30]             # same in one line
```

---

## 2. Intermediate Level: Combining Conditions and Replacing Values

Use `&` (and), `|` (or), `~` (not) — **with parentheses** (Python's `and`/`or` do not work element-wise).
```python
temps = np.array([12, 35, 28, 41, 19, 33])
hot_or_cold = temps[(temps > 32) | (temps < 15)]
mild = temps[~((temps > 32) | (temps < 15))]

np.count_nonzero(temps > 30)       # how many hot days → 3
(temps > 30).mean()                 # proportion → 0.5
np.any(temps > 40), np.all(temps > 0)

clipped = np.where(temps > 35, 35, temps)          # if/else per element
np.clip(temps, 15, 35)                              # cap both sides
labels = np.select([temps < 18, temps < 30], ["cold", "mild"], default="hot")
np.nonzero(temps > 30)                              # indices where True
np.isin(codes, ["MX", "CO"])                        # membership mask
```

### Missing values
```python
x = np.array([1.0, np.nan, 3.0])
np.isnan(x)            # [False  True False]
x[~np.isnan(x)].mean() # 2.0
np.nanmean(x), np.nansum(x), np.nanmax(x)
```

---

## 3. Advanced Level: Masks on 2D Data and Masked Arrays

```python
X = rng.normal(size=(1000, 5))
z = (X - X.mean(axis=0)) / X.std(axis=0)
outlier_rows = np.any(np.abs(z) > 3, axis=1)   # one bool per row
X_clean = X[~outlier_rows]

# In-place conditional update
X[X < 0] = 0

# Masked arrays keep the shape but ignore invalid entries
import numpy.ma as ma
m = ma.masked_where(X > 2.5, X)
m.mean(axis=0)
```
Pandas uses exactly the same idea: `df[df["amount"] > 100]` → [[pandas_selection_filtering]].

## 🧪 Practice
1. Replace all negative values in an array with the column mean.
2. Remove rows containing any NaN from a 2D array.
3. Label exam scores as A (≥90), B (≥80), C (≥70), F with `np.select`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| mask | /mæsk/ | mask | máscara |
| element-wise | /ˈel.ə.mənt waɪz/ | **E**-le-ment uaiz | elemento a elemento |
| outlier | /ˈaʊtˌlaɪ.ɚ/ | **ÁUT**-lai-er | valor atípico |
| clip | /klɪp/ | klip | recortar |
| missing value | /ˈmɪs.ɪŋ ˈvæl.juː/ | **MI**-sing **VAL**-yu | valor faltante |
| proportion | /prəˈpɔːr.ʃən/ | pro-**POR**-shon | proporción |

⬅️ [[numpy_indexing_slicing_views]] · ➡️ [[numpy_broadcasting_engine]]
