---
id: pandas-data-cleaning-missing
title: Pandas Module 4 - Data Cleaning, Missing Values, and Type Casting
tags: [python, pandas, data-science, data-cleaning, missing-values]
aliases: [Pandas Data Cleaning, Handling Missing Values, Imputation]
---

# Pandas Module 4: Data Cleaning, Missing Values & Type Casting

Real-world datasets are rarely pristine; they contain missing entries (`NaN`), inconsistent whitespace, duplicate rows, and incorrect data types. Cleaning and preparing data accounts for the majority of time in any data science workflow.

---

## 1. Beginner Level: Detecting and Dropping Missing Values

Pandas represents missing or null data using `NaN` (Not a Number) for floating-point columns and `None` for object/string columns.

```python
import pandas as pd
import numpy as np

data = {
    'Project': ['Highway A', 'Bridge B', 'Tunnel C', 'Highway D'],
    'Budget_M': [15.2, np.nan, 8.4, 22.1],
    'Status': ['Active', 'Delayed', np.nan, 'Active']
}
df = pd.DataFrame(data)

# 1. Detecting missing values across the DataFrame
print("Null count per column:\n", df.isnull().sum())

# 2. Dropping rows containing ANY null values
cleaned_drop = df.dropna()
print("Dropped Nulls:\n", cleaned_drop)
```

---

## 2. Intermediate Level: Filling, Duplicates, Types and Text

```python
df.isna().mean().sort_values(ascending=False)       # % missing per column
df.dropna(subset=["Budget_M"])                       # drop only if this column is null
df.dropna(thresh=2)                                  # keep rows with ≥ 2 non-nulls
df["Budget_M"] = df["Budget_M"].fillna(df["Budget_M"].median())
df["Status"] = df["Status"].fillna("Unknown")
df["sensor"] = df["sensor"].ffill()                  # forward fill (time series)
df["temp"] = df["temp"].interpolate()                # linear interpolation

# Duplicates
df.duplicated(subset=["email"]).sum()
df = df.drop_duplicates(subset=["email"], keep="last")

# Fix types
df["price"] = pd.to_numeric(df["price"].str.replace("$", "").str.replace(",", ""), errors="coerce")
df["date"] = pd.to_datetime(df["date"], format="%d/%m/%Y", errors="coerce")

# Text normalisation
df["name"] = (df["name"].str.strip().str.replace(r"\s+", " ", regex=True).str.title())
df["country"] = df["country"].str.upper().replace({"MEXICO": "MX", "MÉXICO": "MX"})

# Rename and tidy columns
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")
```

---

## 3. Advanced Level: Missingness Strategy and Outliers

| Mechanism | Meaning | Strategy |
|---|---|---|
| **MCAR** — Missing Completely At Random | no pattern | drop or simple imputation |
| **MAR** — Missing At Random | depends on other columns | group-wise / model-based imputation |
| **MNAR** — Missing Not At Random | depends on the value itself | add a flag, model explicitly |

```python
# Group-wise imputation
df["salary"] = df["salary"].fillna(df.groupby("department")["salary"].transform("median"))
# Keep the information that a value was missing
df["salary_was_missing"] = df["salary"].isna()

# Outliers with IQR
q1, q3 = df["amount"].quantile([.25, .75])
iqr = q3 - q1
mask = df["amount"].between(q1 - 1.5 * iqr, q3 + 1.5 * iqr)
df_no_outliers = df[mask]
df["amount_capped"] = df["amount"].clip(upper=df["amount"].quantile(.99))   # winsorise
```
⚠️ In ML, compute imputation values on the **training set only** (use `SimpleImputer` in a pipeline) → [[sklearn_preprocessing]].

### A reusable cleaning function (method chaining)
```python
def clean_orders(raw: pd.DataFrame) -> pd.DataFrame:
    return (raw
        .rename(columns=str.lower)
        .drop_duplicates("order_id")
        .assign(
            amount=lambda d: pd.to_numeric(d["amount"], errors="coerce"),
            created_at=lambda d: pd.to_datetime(d["created_at"], utc=True),
        )
        .dropna(subset=["amount"])
        .query("amount >= 0")
    )
```

## 🧪 Practice
1. Report the percentage of missing values per column, sorted.
2. Impute missing salaries with the department median and flag them.
3. Write a `clean_customers()` function using method chaining.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| missing | /ˈmɪs.ɪŋ/ | **MI**-sing | faltante |
| fill | /fɪl/ | fil | rellenar |
| impute | /ɪmˈpjuːt/ | im-**PIUT** | imputar |
| drop | /drɑːp/ | drap | descartar |
| duplicate | /ˈduː.plɪ.kət/ | **DU**-pli-kit | duplicado |
| coerce | /koʊˈɝːs/ | kou-**ERS** | forzar |
| tidy | /ˈtaɪ.di/ | **TÁI**-di | ordenado |
| threshold | /ˈθreʃ.hoʊld/ | **ZRESH**-jould | umbral |

⬅️ [[pandas_selection_filtering]] · ➡️ [[pandas_transformations_apply]]
