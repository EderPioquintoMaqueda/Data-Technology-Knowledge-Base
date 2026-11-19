---
id: pandas-selection-filtering
title: Pandas Module 3 - Data Selection, Indexing, and Boolean Filtering
tags: [python, pandas, data-science, selection, indexing, filtering]
aliases: [Pandas Selection, loc vs iloc, Filtering DataFrames]
---

# Pandas Module 3: Data Selection, Indexing & Boolean Filtering

Extracting specific subsets of data based on row labels, integer positions, or logical conditions is a daily task in data analysis. Pandas provides dedicated accessors (`.loc` and `.iloc`) to make data retrieval explicit and unambiguous.

---

## 1. Beginner Level: Column Selection & Basic Filtering

Selecting columns in a DataFrame can be done using dot notation or bracket notation.

```python
import pandas as pd

data = {
    'Department': ['Infrastructure', 'Finance', 'Engineering', 'Operations'],
    'Employees': [45, 12, 30, 25],
    'Budget_M': [12.5, 3.1, 8.4, 5.0]
}
df = pd.DataFrame(data)

# 1. Selecting a single column (returns a Series)
depts = df['Department']

# 2. Selecting multiple columns (returns a DataFrame)
subset = df[['Department', 'Budget_M']]
print(subset)
```

---

## 2. Intermediate Level: `loc`, `iloc` and Boolean Filters

| Accessor | Selects by | Example |
|---|---|---|
| `df.loc[rows, cols]` | **labels** (end included) | `df.loc[0:2, "Department":"Budget_M"]` |
| `df.iloc[rows, cols]` | **positions** (end excluded) | `df.iloc[0:2, 0:2]` |
| `df.at[label, col]` / `df.iat[i, j]` | single value (fast) | `df.at[0, "Employees"]` |

```python
df.loc[df["Budget_M"] > 5]                                  # filter rows
df.loc[df["Budget_M"] > 5, ["Department", "Budget_M"]]      # filter + columns
df.loc[(df["Employees"] > 20) & (df["Budget_M"] < 10)]      # & | ~ with parentheses
df[df["Department"].isin(["Finance", "Operations"])]
df[df["Department"].str.contains("ing", case=False, na=False)]
df[df["Budget_M"].between(5, 10)]
df.query("Employees > 20 and Budget_M < @limit")            # SQL-like, @ = Python variable
df.nlargest(3, "Budget_M"); df.nsmallest(2, "Employees")
```

### Assigning safely
```python
df.loc[df["Budget_M"] > 10, "priority"] = "high"     # ✅ one step with .loc
# ❌ df[df["Budget_M"] > 10]["priority"] = "high"     chained indexing — may do nothing
```

---

## 3. Advanced Level: Column Selection Patterns and MultiIndex Slicing

```python
df.select_dtypes(include="number")
df.filter(like="_M")                 # columns containing "_M"
df.filter(regex=r"^sales_\d{4}$")
df.loc[:, df.isna().mean() < 0.3]    # keep columns with < 30% nulls

# MultiIndex
sales = df2.set_index(["region", "year"]).sort_index()
sales.loc["North"]
sales.loc[("North", 2026)]
idx = pd.IndexSlice
sales.loc[idx[:, 2025:2026], :]       # all regions, years 2025–2026
```
SQL equivalent: `WHERE`, `IN`, `LIKE`, `BETWEEN` → [[SQL-B01 SELECT, WHERE and ORDER BY]].

## 🧪 Practice
1. Select employees from two departments hired after 2024 with `query`.
2. Set a column `segment` with `.loc` for three different conditions.
3. Keep only numeric columns whose standard deviation is > 0.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| select | /səˈlekt/ | se-**LEKT** | seleccionar |
| filter | /ˈfɪl.tɚ/ | **FIL**-ter | filtrar |
| subset | /ˈsʌb.set/ | **SAB**-set | subconjunto |
| between | /bɪˈtwiːn/ | bi-**TUIN** | entre |
| chained | /tʃeɪnd/ | cheind | encadenado |
| inclusive | /ɪnˈkluː.sɪv/ | in-**KLU**-siv | inclusivo |

⬅️ [[pandas_io_data_loading]] · ➡️ [[pandas_data_cleaning_missing]]
