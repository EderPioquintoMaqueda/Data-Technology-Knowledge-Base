---
title: "Polars - Beginner to Advanced"
tags: [python, polars, dataframes, lazy, performance]
---

# ⚡ Polars — Beginner to Advanced

Polars is a DataFrame library written in **Rust** on top of **Apache Arrow**. It is multi-threaded, has a **lazy query optimiser**, and an expression API that looks a lot like SQL.

| | pandas | Polars |
|---|---|---|
| Engine | NumPy/Arrow, mostly single-threaded | Rust, all CPU cores |
| Index | yes (row labels) | **no index** |
| Execution | eager | eager **or lazy** |
| Missing values | `NaN`/`None`/`NA` | single `null` |
| Syntax | methods + `[]` | **expressions** `pl.col(...)` |

---

## 🟢 1. Beginner
```python
import polars as pl

df = pl.read_csv("sales.csv", try_parse_dates=True)
df.head(); df.shape; df.schema; df.describe()

df.select("region", "amount")                          # choose columns
df.filter(pl.col("amount") > 100)                       # filter rows
df.sort("amount", descending=True)
df.with_columns((pl.col("amount") * 1.16).alias("amount_tax"))   # new column
df.rename({"amount": "total"})
df.drop("notes")
df.unique(subset=["order_id"])
df.null_count()
```

### Expressions — the heart of Polars
An expression describes a computation on columns; Polars runs them **in parallel**.
```python
df.select(
    pl.col("amount").sum().alias("total"),
    pl.col("amount").mean().round(2).alias("avg"),
    pl.col("customer_id").n_unique().alias("customers"),
)
```

### Group by
```python
(df.group_by("region")
   .agg(
       pl.len().alias("orders"),
       pl.col("amount").sum().alias("revenue"),
       pl.col("amount").max().alias("biggest"),
   )
   .sort("revenue", descending=True))
```

## 🟡 2. Intermediate
### Conditional columns
```python
df.with_columns(
    pl.when(pl.col("amount") >= 1000).then(pl.lit("high"))
      .when(pl.col("amount") >= 100).then(pl.lit("medium"))
      .otherwise(pl.lit("low"))
      .alias("tier")
)
```
### Strings and dates
```python
df.with_columns(
    pl.col("email").str.to_lowercase().str.strip_chars(),
    pl.col("name").str.split(" ").list.first().alias("first_name"),
    pl.col("order_date").dt.year().alias("year"),
    pl.col("order_date").dt.truncate("1mo").alias("month"),
)
```
### Joins
```python
orders.join(customers, on="customer_id", how="left")     # inner, left, full, semi, anti, cross
orders.join(customers, on="customer_id", how="anti")     # orders without a valid customer
```
### Window functions (`over`) — like SQL `OVER (PARTITION BY ...)`
```python
df.with_columns(
    pl.col("amount").sum().over("region").alias("region_total"),
    (pl.col("amount") / pl.col("amount").sum().over("region")).alias("share"),
    pl.col("amount").rank("dense", descending=True).over("region").alias("rank_in_region"),
    pl.col("amount").shift(1).over("customer_id").alias("prev_amount"),
)
```
### Reshape
```python
df.pivot(on="month", index="region", values="amount", aggregate_function="sum")
df.unpivot(index="region", on=["jan", "feb", "mar"], variable_name="month", value_name="amount")
```

## 🔴 3. Advanced — Lazy API
```python
lf = (
    pl.scan_parquet("data/orders/*.parquet")             # nothing is read yet
      .filter(pl.col("status") == "paid")
      .with_columns(pl.col("created_at").dt.truncate("1mo").alias("month"))
      .group_by("month", "country")
      .agg(pl.col("total").sum().alias("revenue"))
      .sort("month")
)
print(lf.explain())          # see the optimised plan (predicate & projection pushdown)
result = lf.collect()        # execute
# Larger than RAM:
lf.collect(engine="streaming")
lf.sink_parquet("out/revenue.parquet")      # stream straight to disk
```
The optimiser applies:
- **Predicate pushdown** — filters are applied while reading files.
- **Projection pushdown** — only needed columns are read.
- **Common subexpression elimination**, join reordering, parallel scans.

### Time-series helpers
```python
(df.sort("ts")
   .group_by_dynamic("ts", every="1h", group_by="sensor_id")
   .agg(pl.col("value").mean()))
df.with_columns(pl.col("value").rolling_mean(window_size=7).over("sensor_id"))
df.join_asof(prices, on="ts", by="ticker", strategy="backward")
```

### Interop
```python
pl.from_pandas(pdf); df.to_pandas(); df.to_arrow()
pl.read_database_uri("SELECT * FROM orders", "postgresql://user:pass@host/db")
import duckdb; duckdb.sql("SELECT * FROM df WHERE amount > 100").pl()   # DuckDB reads Polars directly
```

### Anti-patterns
- ❌ `map_elements(lambda ...)` row by row (loses parallelism) → use native expressions.
- ❌ Calling `.collect()` in the middle of a lazy pipeline.
- ❌ Reading CSV repeatedly → convert once to Parquet.

Deep dive: [[Polars_High_Performance]] · Compare with [[Pandas_High_Performance]].

---

## 🧪 Practice
1. Load a CSV of orders, compute revenue by month and country with the lazy API, and print `explain()`.
2. Add a column with each customer's previous order date and days between orders.
3. Rewrite one of your pandas notebooks in Polars and compare runtime.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| expression | /ɪkˈspreʃ.ən/ | ik-**SPRE**-shon | expresión |
| lazy evaluation | /ˈleɪ.zi ɪˌvæl.juˈeɪ.ʃən/ | **LÉI**-zi i-va-liu-**ÉI**-shon | evaluación diferida |
| eager | /ˈiː.ɡɚ/ | **I**-guer | inmediata (ejecución) |
| pushdown | /ˈpʊʃ.daʊn/ | **PUSH**-daun | empuje (de filtros) hacia abajo |
| query plan | /ˈkwɪr.i plæn/ | **KUI**-ri plan | plan de consulta |
| streaming | /ˈstriː.mɪŋ/ | **STRI**-ming | procesamiento en flujo |
| sink | /sɪŋk/ | sink | destino (escribir a) |
| reshape | /riːˈʃeɪp/ | ri-**SHÉIP** | reestructurar |
| rolling | /ˈroʊ.lɪŋ/ | **ROU**-ling | móvil (ventana) |

🏠 [[Python Libraries Map]]

#python #polars
