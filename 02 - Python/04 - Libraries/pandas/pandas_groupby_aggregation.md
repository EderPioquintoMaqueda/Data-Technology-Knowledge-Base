---
id: pandas-groupby-aggregation
title: Pandas Module 6 - Groupby, Aggregations, and Window Functions
tags: [python, pandas, data-science, groupby, aggregation, window-functions]
aliases: [Pandas Groupby, Split-Apply-Combine, Pivot Tables]
---

# Pandas Module 6: Groupby, Aggregations & Window Functions

Summarizing, grouping, and analyzing tabular datasets by categories is the core of exploratory data analysis. Pandas implements the **Split-Apply-Combine** pattern via the `.groupby()` engine.

---

## 1. Beginner Level: Basic Groupby & Single Aggregations

The `groupby` operation splits the DataFrame into separate groups based on unique values in one or more columns, applies an aggregation function independently to each group, and combines the results.

```python
import pandas as pd

data = {
    'Region': ['North', 'South', 'North', 'East', 'South', 'East'],
    'Project': ['Road A', 'Bridge B', 'Tunnel C', 'Highway D', 'Dam E', 'Overpass F'],
    'Budget_M': [12.5, 8.1, 15.0, 6.2, 22.4, 4.5]
}
df = pd.DataFrame(data)

# Group by Region and calculate the mean budget per region
mean_budget = df.groupby('Region')['Budget_M'].mean()
print("Mean Budget per Region:\n", mean_budget)
```

---

## 2. Intermediate Level: Multiple and Named Aggregations

```python
summary = df.groupby("Region").agg(
    projects=("Project", "count"),
    total_budget=("Budget_M", "sum"),
    avg_budget=("Budget_M", "mean"),
    max_budget=("Budget_M", "max"),
).sort_values("total_budget", ascending=False).reset_index()

df.groupby(["Region", "Status"], as_index=False)["Budget_M"].sum()
df.groupby("Region")["Budget_M"].agg(["min", "median", "max", "std"])
df["Region"].value_counts(normalize=True)          # frequency %
df.pivot_table(index="Region", columns="Year", values="Budget_M", aggfunc="sum", margins=True)
```
SQL equivalent → [[SQL-B02 Aggregations - GROUP BY and HAVING]].

---

## 3. Advanced Level: transform, filter and Window Functions

```python
# transform → result has the SAME length as df (like SQL window functions)
df["region_total"] = df.groupby("Region")["Budget_M"].transform("sum")
df["share_of_region"] = df["Budget_M"] / df["region_total"]
df["rank_in_region"] = df.groupby("Region")["Budget_M"].rank(ascending=False, method="dense")

# filter groups (like HAVING)
big_regions = df.groupby("Region").filter(lambda g: g["Budget_M"].sum() > 20)

# running and rolling calculations per group
df = df.sort_values(["customer_id", "date"])
df["cum_spend"] = df.groupby("customer_id")["amount"].cumsum()
df["prev_amount"] = df.groupby("customer_id")["amount"].shift(1)
df["rolling_3"] = df.groupby("customer_id")["amount"].transform(lambda s: s.rolling(3, min_periods=1).mean())

# Top-N per group
df.sort_values("Budget_M", ascending=False).groupby("Region").head(2)
```
SQL equivalent → [[SQL-I03 Window Functions]].

## 🧪 Practice
1. Revenue, orders and average ticket per country, sorted.
2. Each customer's share of their country's revenue (`transform`).
3. Top 3 products per category.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| group by | /ɡruːp baɪ/ | grup bai | agrupar por |
| aggregate | /ˈæɡ.rɪ.ɡeɪt/ | **A**-gri-gueit | agregar (resumir) |
| split-apply-combine | /splɪt əˈplaɪ kəmˈbaɪn/ | split a-**PLÁI** kom-**BÁIN** | dividir-aplicar-combinar |
| running total | /ˈrʌn.ɪŋ ˈtoʊ.t̬əl/ | **RA**-ning **TOU**-tal | total acumulado |
| share | /ʃer/ | sher | proporción / participación |
| rank | /ræŋk/ | rank | posición / clasificar |

⬅️ [[pandas_transformations_apply]] · ➡️ [[pandas_merging_joining]]
