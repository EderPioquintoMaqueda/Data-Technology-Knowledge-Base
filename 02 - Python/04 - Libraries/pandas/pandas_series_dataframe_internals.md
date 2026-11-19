---
id: pandas-series-dataframe-internals
title: Pandas Module 1 - Series, DataFrames, and Internal Architecture
tags: [python, pandas, data-science, dataframe, memory-architecture]
aliases: [Pandas Architecture, DataFrame Internals, Series vs DataFrame]
---

# Pandas Module 1: Series, DataFrames & Internal Architecture

While NumPy handles homogenous multi-dimensional numerical matrices, real-world data is typically heterogeneous (mixing text, numbers, dates, and booleans) and indexed. **Pandas** is built on top of NumPy to provide high-performance, tabular data structures (`Series` and `DataFrame`) equipped with labeled axes.

---

## 1. Beginner Level: Core Data Structures

### The `Series` (1D Labeled Array)
A Pandas `Series` is a one-dimensional labeled array capable of holding any data type. It consists of an array of data and an array of labels called the **Index**.

```python
import pandas as pd

# Creating a Series with explicit index labels
temperatures = pd.Series([22.5, 24.1, 19.8, 25.3], index=['Mon', 'Tue', 'Wed', 'Thu'])
print("Series:\n", temperatures)
print("Accessing by label ('Tue'):", temperatures['Tue'])
```

---

## 2. Intermediate Level: DataFrames, Index and dtypes

A **DataFrame** is a dictionary of Series that share one **index** (row labels) and have **columns** (column labels).

```python
df = pd.DataFrame({
    "city": ["Lima", "Quito", "Bogotá"],
    "population_m": [10.9, 2.8, 7.9],
    "capital": [True, True, True],
}, index=["PE", "EC", "CO"])

df.shape          # (3, 3)
df.columns        # Index(['city', 'population_m', 'capital'])
df.index          # Index(['PE', 'EC', 'CO'])
df.dtypes         # object / float64 / bool
df.info()         # dtypes + non-null counts + memory
df.describe()     # statistics of numeric columns
df.head(2); df.tail(2); df.sample(2, random_state=0)

df.reset_index(names="code")    # index → column
df.set_index("city")            # column → index
```

### Index alignment — pandas' superpower (and trap)
```python
s1 = pd.Series([1, 2, 3], index=["a", "b", "c"])
s2 = pd.Series([10, 20, 30], index=["b", "c", "d"])
s1 + s2          # a NaN, b 12, c 23, d NaN  ← aligned by LABEL, not position
s1.add(s2, fill_value=0)
```

### Modern dtypes (pandas 2.x)
| Type | Use |
|---|---|
| `Int64`, `Float64`, `boolean` (capitalised) | nullable numbers/bools (`pd.NA`) |
| `string[pyarrow]` | fast, memory-efficient text |
| `category` | few repeated values (country, status) |
| `datetime64[ns, tz]` | timestamps with time zone |

```python
df = df.convert_dtypes(dtype_backend="pyarrow")
```

---

## 3. Advanced Level: Internals and Copy-on-Write

- Internally a DataFrame stores columns of the same dtype in **blocks** (2-D NumPy arrays) managed by the `BlockManager`; with the Arrow backend, columns are Arrow arrays.
- **Copy-on-Write** (default in pandas 3.0, opt-in in 2.x via `pd.options.mode.copy_on_write = True`): any derived object behaves as a copy, and data is only physically copied when modified. This removes the infamous `SettingWithCopyWarning`.
- **MultiIndex**: hierarchical labels — `df.set_index(["country", "year"])`, `df.loc[("MX", 2026)]`, `df.xs(2026, level="year")`, `stack()/unstack()`.

```python
pd.options.mode.copy_on_write = True
sub = df[df["population_m"] > 5]
sub["big"] = True        # safe: never modifies df
```

## 🧪 Practice
1. Create a DataFrame of 5 products, inspect it with `info()` and `describe()`.
2. Demonstrate index alignment between two Series with different labels.
3. Convert a DataFrame to Arrow-backed dtypes and compare `memory_usage(deep=True)`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| label | /ˈleɪ.bəl/ | **LÉI**-bol | etiqueta |
| row | /roʊ/ | rou | fila |
| column | /ˈkɑː.ləm/ | **KA**-lem | columna |
| alignment | /əˈlaɪn.mənt/ | a-**LÁIN**-ment | alineación |
| nullable | /ˈnʌl.ə.bəl/ | **NA**-la-bol | que admite nulos |
| hierarchical | /ˌhaɪ.ɚˈɑːr.kɪ.kəl/ | jai-e-**RAR**-ki-kol | jerárquico |
| warning | /ˈwɔːr.nɪŋ/ | **UOR**-ning | advertencia |

⬅️ [[numpy_linear_algebra_einsum]] · ➡️ [[pandas_io_data_loading]] · 🏠 [[Python Libraries Map]]
