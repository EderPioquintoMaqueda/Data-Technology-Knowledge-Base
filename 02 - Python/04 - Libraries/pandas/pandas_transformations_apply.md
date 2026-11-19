---
id: pandas-transformations-apply
title: Pandas Module 5 - Data Transformations, Vectorized Operations, and Custom Functions
tags: [python, pandas, data-science, transformations, apply, vectorization]
aliases: [Pandas Transformations, Apply vs Vectorized, Custom Functions]
---

# Pandas Module 5: Data Transformations, Vectorized Operations & Custom Functions

Transforming column data—whether calculating new metrics, converting units, or applying complex custom logic—is fundamental to feature engineering. Understanding when to use vectorized operations versus `.apply()` is critical for pipeline performance.

---

## 1. Beginner Level: Vectorized Arithmetic & String Methods

Whenever possible, rely on Pandas' built-in vectorized operations. They operate directly on contiguous NumPy arrays under the hood, executing in C without Python loop overhead.

```python
import pandas as pd

data = {
    'Item': [' Asphalt ', 'CONCRETE', ' steel beam '],
    'Unit_Cost': [1200.0, 3400.0, 950.0],
    'Quantity': [50, 20, 100]
}
df = pd.DataFrame(data)

# 1. Vectorized arithmetic calculation
df['Total_Cost'] = df['Unit_Cost'] * df['Quantity']

# 2. Vectorized string manipulations via the .str accessor
df['Item'] = df['Item'].str.strip().str.title()
print("Vectorized Transformations:\n", df)
```

---

## 2. Intermediate Level: assign, map, cut, apply

```python
df = df.assign(
    total=lambda d: d["Unit_Cost"] * d["Quantity"],
    cost_band=lambda d: pd.cut(d["Unit_Cost"], bins=[0, 1000, 2000, float("inf")],
                               labels=["low", "mid", "high"]),
)
df["quartile"] = pd.qcut(df["total"], q=4, labels=["Q1", "Q2", "Q3", "Q4"])
df["category"] = df["Item"].map({"Asphalt": "Road", "Concrete": "Structure"})   # dict lookup
df["flag"] = np.where(df["total"] > 50_000, "review", "ok")

# apply: only when no vectorised alternative exists
df["slug"] = df["Item"].apply(lambda s: s.lower().replace(" ", "-"))
```
| Method | Works on | Speed |
|---|---|---|
| vectorised ops / `.str` / `.dt` | whole column | ⚡⚡⚡ |
| `map(dict)` | Series values | ⚡⚡ |
| `apply(func)` on a Series | each value (Python loop) | ⚡ |
| `apply(func, axis=1)` | each row (Python loop) | 🐢 |

---

## 3. Advanced Level: Reshaping — pivot, melt, explode

```python
long = pd.DataFrame({"region": ["N", "N", "S"], "month": ["Jan", "Feb", "Jan"], "sales": [10, 12, 7]})
wide = long.pivot_table(index="region", columns="month", values="sales", aggfunc="sum", fill_value=0)
back = wide.reset_index().melt(id_vars="region", var_name="month", value_name="sales")

tags = pd.DataFrame({"id": [1, 2], "tags": [["sql", "python"], ["excel"]]})
tags.explode("tags")              # one row per list element

pd.crosstab(df["region"], df["status"], normalize="index")   # contingency table
df.pipe(clean_orders).pipe(add_features)                      # functional pipelines
```
**Tidy data** rules: each variable a column, each observation a row, each value a cell.

## 🧪 Practice
1. Bin ages into groups (`cut`) and count customers per group.
2. Convert a wide monthly report into long format with `melt`.
3. Replace an `apply(axis=1)` with vectorised code and time both.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| bin | /bɪn/ | bin | intervalo / contenedor |
| quartile | /ˈkwɔːr.taɪl/ | **KUÓR**-tail | cuartil |
| wide / long format | /waɪd/ /lɑːŋ/ | uaid / long | formato ancho / largo |
| melt | /melt/ | melt | derretir (despivotar) |
| explode | /ɪkˈsploʊd/ | ik-**SPLOUD** | expandir (una lista en filas) |
| lookup | /ˈlʊk.ʌp/ | **LUK**-ap | búsqueda |

⬅️ [[pandas_data_cleaning_missing]] · ➡️ [[pandas_groupby_aggregation]]
