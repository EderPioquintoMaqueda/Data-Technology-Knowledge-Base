---
id: pandas-merging-joining
title: Pandas Module 7 - Merging, Joining, and Concatenating DataFrames
tags: [python, pandas, data-science, merge, join, concat]
aliases: [Pandas Merging, DataFrame Joins, Concatenation]
---

# Pandas Module 7: Merging, Joining & Concatenating DataFrames

Real-world analytics often require combining data from multiple sources—such as merging customer demographic tables with transactional sales records or stacking monthly report files. Pandas provides three core combination functions: `concat`, `merge`, and `join`.

---

## 1. Beginner Level: Vertical and Horizontal Concatenation (`pd.concat`)

When you have datasets with the same columns (stacking rows) or the same row indices (stacking columns), use `pd.concat()`.

```python
import pandas as pd

# Two datasets with identical column structures (e.g., Q1 and Q2 reports)
q1_df = pd.DataFrame({'Month': ['Jan', 'Feb'], 'Sales': [15000, 18000]})
q2_df = pd.DataFrame({'Month': ['Apr', 'May'], 'Sales': [22000, 25000]})

# 1. Vertical Concatenation (Stacking rows)
# ignore_index=True resets the index sequence from 0 to N-1
stacked_df = pd.concat([q1_df, q2_df], ignore_index=True)
print("Stacked Rows:\n", stacked_df)
```

---

## 2. Intermediate Level: merge (SQL-style joins)

```python
customers = pd.DataFrame({"customer_id": [1, 2, 3], "name": ["Ana", "Luis", "Eva"]})
orders = pd.DataFrame({"order_id": [10, 11, 12], "customer_id": [1, 1, 4], "total": [50, 20, 99]})

pd.merge(orders, customers, on="customer_id", how="inner")   # only matches
pd.merge(orders, customers, on="customer_id", how="left")    # all orders
pd.merge(orders, customers, on="customer_id", how="outer", indicator=True)  # _merge column shows source
pd.merge(a, b, left_on="cust_id", right_on="id", suffixes=("_order", "_cust"))
```
| how | SQL | Keeps |
|---|---|---|
| `inner` | INNER JOIN | matching keys |
| `left` | LEFT JOIN | all left rows |
| `right` | RIGHT JOIN | all right rows |
| `outer` | FULL OUTER JOIN | everything |
| `cross` | CROSS JOIN | every combination |

### Anti-join (rows in A not in B)
```python
m = orders.merge(customers, on="customer_id", how="left", indicator=True)
orphans = m[m["_merge"] == "left_only"]          # orders with unknown customer
```

---

## 3. Advanced Level: Validation, Many-to-Many and As-of Joins

```python
pd.merge(orders, customers, on="customer_id", validate="many_to_one")   # raises if customers has duplicates
```
⚠️ A **many-to-many** merge multiplies rows (row explosion). Always compare `len()` before and after a merge.

```python
# merge_asof: nearest previous match on a sorted key (prices at trade time)
trades = trades.sort_values("ts"); quotes = quotes.sort_values("ts")
pd.merge_asof(trades, quotes, on="ts", by="ticker", direction="backward", tolerance=pd.Timedelta("5s"))

# join on index
left.join(right, how="left", rsuffix="_r")
# combine_first: fill gaps of one frame with another
master.combine_first(backup)
```
Theory: [[SQL-B03 JOINs]] · [[14 - Spark Joins and Aggregations]].

## 🧪 Practice
1. Find customers who never placed an order.
2. Show how a duplicate key creates extra rows and prevent it with `validate`.
3. Attach the latest exchange rate to each transaction with `merge_asof`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| merge | /mɝːdʒ/ | merch | combinar / fusionar |
| join | /dʒɔɪn/ | yoin | unión (join) |
| match | /mætʃ/ | match | coincidencia |
| orphan | /ˈɔːr.fən/ | **OR**-fan | huérfano |
| suffix | /ˈsʌf.ɪks/ | **SA**-fiks | sufijo |
| many-to-many | /ˈmen.i tə ˈmen.i/ | **ME**-ni tu **ME**-ni | muchos a muchos |
| stack / concatenate | /kənˈkæt.ə.neɪt/ | kon-**KA**-te-neit | apilar / concatenar |

⬅️ [[pandas_groupby_aggregation]] · ➡️ [[pandas_timeseries]]
