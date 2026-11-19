---
title: "DuckDB - SQL Inside Python"
tags: [python, duckdb, sql, olap, parquet]
---

# 🦆 DuckDB — SQL Inside Python

DuckDB is an **in-process OLAP database** ("SQLite for analytics"): no server, columnar, vectorised, multi-threaded. It queries **CSV, Parquet, JSON, pandas, Polars and even PostgreSQL** directly with SQL.

Use it when: you know SQL, your data is in files, and pandas is too slow or too memory-hungry.

---

## 🟢 Beginner
```python
import duckdb

duckdb.sql("SELECT 42 AS answer").show()

# Query files directly — no import step
duckdb.sql("""
    SELECT country, COUNT(*) AS orders, SUM(total) AS revenue
    FROM 'data/orders.csv'
    GROUP BY country
    ORDER BY revenue DESC
""").df()          # → pandas   (.pl() → Polars, .fetchall() → tuples)

# Query a pandas / Polars DataFrame by its variable name
import pandas as pd
sales = pd.DataFrame({"region": ["N", "S", "N"], "amount": [10, 5, 7]})
duckdb.sql("SELECT region, SUM(amount) FROM sales GROUP BY region").df()
```

## 🟡 Intermediate
### Persistent database file
```python
con = duckdb.connect("analytics.duckdb")
con.execute("CREATE TABLE orders AS SELECT * FROM read_parquet('data/orders/*.parquet')")
con.execute("SELECT * FROM orders WHERE total > ?", [100]).df()    # parameters!
con.close()
```
### Reading many files with options
```sql
SELECT * FROM read_csv('raw/*.csv', header = true, delim = ';', dateformat = '%d/%m/%Y', union_by_name = true);
SELECT * FROM read_parquet('s3://bucket/sales/year=*/month=*/*.parquet', hive_partitioning = true);
SELECT * FROM read_json_auto('events.json');
```
### Exporting
```sql
COPY (SELECT * FROM orders WHERE status = 'paid') TO 'paid.parquet' (FORMAT parquet, COMPRESSION zstd);
COPY orders TO 'out' (FORMAT parquet, PARTITION_BY (country));
```

## 🔴 Advanced
### Friendly SQL extensions
```sql
SELECT * EXCLUDE (password) FROM users;
SELECT * REPLACE (round(total, 2) AS total) FROM orders;
SELECT country, SUM(total) FROM orders GROUP BY ALL;     -- group by all non-aggregated cols
SELECT * FROM orders QUALIFY row_number() OVER (PARTITION BY customer_id ORDER BY created_at DESC) = 1;
PIVOT sales ON month USING SUM(amount) GROUP BY region;
SUMMARIZE orders;                                         -- instant profiling
```
### Attach PostgreSQL / SQLite
```sql
INSTALL postgres; LOAD postgres;
ATTACH 'dbname=shop user=me host=localhost' AS pg (TYPE postgres, READ_ONLY);
SELECT * FROM pg.public.customers LIMIT 10;
```
### Performance tips
- Prefer Parquet over CSV; DuckDB uses column statistics to skip row groups.
- `EXPLAIN ANALYZE <query>` to see the plan.
- `SET memory_limit = '4GB'; SET threads = 8;` — DuckDB spills to disk when data exceeds memory.

### DuckDB vs others
| | DuckDB | SQLite | PostgreSQL | pandas |
|---|---|---|---|---|
| Workload | analytics (OLAP) | small apps (OLTP) | apps + analytics | in-memory analysis |
| Server | no | no | yes | no |
| Storage | columnar | row | row | in RAM |
| Concurrency writes | single process | single writer | many | — |

Theory: [[04 - OLTP vs OLAP]] · SQL practice: [[00 - SQL Roadmap]] (all exercises run in DuckDB!).

## 🧪 Practice
1. Download a public CSV (e.g. NYC taxi Parquet) and compute trips per hour with DuckDB.
2. Convert a folder of CSVs into a partitioned Parquet dataset.
3. Use `QUALIFY` to keep the latest order per customer.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| in-process | /ɪn ˈprɑː.ses/ | in **PRA**-ses | dentro del proceso (embebido) |
| columnar | /kəˈlʌm.nɚ/ | ko-**LAM**-ner | columnar |
| spill (to disk) | /spɪl/ | spil | volcar (a disco) |
| attach | /əˈtætʃ/ | a-**TACH** | adjuntar / conectar |
| row group | /roʊ ɡruːp/ | rou grup | grupo de filas |
| hungry (memory-hungry) | /ˈhʌŋ.ɡri/ | **JAN**-gri | que consume mucha memoria |

🏠 [[Python Libraries Map]]

#python #duckdb #sql
