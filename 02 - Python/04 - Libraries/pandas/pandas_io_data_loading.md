---
id: pandas-io-data-loading
title: Pandas Module 2 - Data Loading, IO Operations, and Memory Efficiency
tags: [python, pandas, data-science, io, csv, parquet, sql]
aliases: [Pandas IO, Data Loading, Reading CSV Parquet SQL]
---

# Pandas Module 2: Data Loading, IO Operations & Memory Efficiency

Real-world data rarely lives inside Python scripts; it resides in CSV files, Excel spreadsheets, relational databases, or compressed columnar stores like Parquet. Knowing how to load data efficiently while managing RAM is a core professional skill.

---

## 1. Beginner Level: Reading and Writing CSV Files

The workhorse function for loading tabular text data is `pd.read_csv()`.

```python
import pandas as pd

# Basic CSV loading
# df = pd.read_csv('dataset.csv')

# Writing a DataFrame to a CSV file (excluding row index numbers)
# df.to_csv('output.csv', index=False)
```

---

## 2. Intermediate Level: Real-World read_csv Options and Other Formats

```python
df = pd.read_csv(
    "ventas_2026.csv",
    sep=";",                      # European/LatAm exports often use ;
    decimal=",", thousands=".",   # 1.234,56 → 1234.56
    encoding="utf-8",             # try "latin-1" / "cp1252" for old Excel exports
    usecols=["fecha", "cliente", "monto"],
    dtype={"cliente": "string"},
    parse_dates=["fecha"], dayfirst=True,
    na_values=["", "NA", "-", "N/D"],
    skiprows=2, nrows=100_000,
)
df.to_csv("clean.csv", index=False, encoding="utf-8-sig")   # utf-8-sig opens nicely in Excel

pd.read_excel("budget.xlsx", sheet_name=None)               # dict of ALL sheets
pd.read_json("events.json", lines=True)                      # JSON Lines
pd.read_parquet("sales.parquet", columns=["date", "amount"])
pd.read_html("https://...")                                  # tables from web pages
pd.read_clipboard()                                          # copy from Excel → DataFrame
```

### Databases
```python
from sqlalchemy import create_engine
engine = create_engine("postgresql+psycopg://user:pass@localhost/shop")
orders = pd.read_sql("SELECT * FROM orders WHERE created_at >= %(d)s", engine, params={"d": "2026-01-01"})
orders.to_sql("orders_copy", engine, if_exists="append", index=False, chunksize=10_000)
```
More: [[PY-A05 Python and Databases]].

---

## 3. Advanced Level: Large Files

| Technique | Code |
|---|---|
| Read in chunks | `for ch in pd.read_csv(f, chunksize=200_000): ...` |
| Fast parser | `pd.read_csv(f, engine="pyarrow")` |
| Arrow dtypes | `dtype_backend="pyarrow"` |
| Columnar storage | write Parquet once, read many times |
| Partitioned datasets | `df.to_parquet("out/", partition_cols=["year", "month"])` |
| Cloud storage | `pd.read_parquet("s3://bucket/key.parquet")` (needs `s3fs`) |

```python
totals = []
for chunk in pd.read_csv("transactions_20GB.csv", chunksize=500_000, usecols=["store", "amount"]):
    totals.append(chunk.groupby("store")["amount"].sum())
result = pd.concat(totals).groupby(level=0).sum()
```
When files exceed RAM regularly → [[Polars - Beginner to Advanced]] or [[DuckDB - SQL Inside Python]].

## 🧪 Practice
1. Load a CSV exported from Spanish Excel (`;` separator, `,` decimal) correctly.
2. Convert it to Parquet and compare file size and load time.
3. Aggregate a big CSV in chunks.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| load | /loʊd/ | loud | cargar |
| separator / delimiter | /ˈsep.ə.reɪ.t̬ɚ/ /dɪˈlɪm.ɪ.t̬ɚ/ | **SE**-pa-rei-ter / di-**LI**-mi-ter | separador / delimitador |
| thousands | /ˈθaʊ.zəndz/ | **ZÁU**-zends | miles |
| sheet | /ʃiːt/ | shit (i larga: "shiit") | hoja |
| clipboard | /ˈklɪp.bɔːrd/ | **KLIP**-bord | portapapeles |
| partition | /pɑːrˈtɪʃ.ən/ | par-**TI**-shon | partición |

⬅️ [[pandas_series_dataframe_internals]] · ➡️ [[pandas_selection_filtering]]
