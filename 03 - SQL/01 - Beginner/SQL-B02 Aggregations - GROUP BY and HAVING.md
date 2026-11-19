---
title: "SQL-B02 Aggregations - GROUP BY and HAVING"
level: beginner
tags: [sql, beginner, aggregation, group-by]
prev: "[[SQL-B01 SELECT, WHERE and ORDER BY]]"
next: "[[SQL-B03 JOINs]]"
---

# SQL-B02 — Aggregations: GROUP BY and HAVING

## 🎯 Learning objectives
- Summarise data with aggregate functions.
- Group rows with `GROUP BY`.
- Filter groups with `HAVING` (and understand how it differs from `WHERE`).

---

## 1. Aggregate functions
```sql
SELECT COUNT(*)              AS products,
       COUNT(DISTINCT category_id) AS categories,
       MIN(price)            AS cheapest,
       MAX(price)            AS most_expensive,
       ROUND(AVG(price), 2)  AS avg_price,
       SUM(stock)            AS units_in_stock
FROM products;
```
| Function | Returns | NULL handling |
|---|---|---|
| `COUNT(*)` | number of rows | counts all rows |
| `COUNT(col)` | non-null values | ignores NULL |
| `COUNT(DISTINCT col)` | unique non-null values | ignores NULL |
| `SUM`, `AVG`, `MIN`, `MAX` | numbers | ignore NULL |

```sql
SELECT COUNT(*) AS total_customers, COUNT(email) AS with_email FROM customers;   -- 12 vs 11
```

## 2. GROUP BY — one result row per group
```sql
SELECT country, COUNT(*) AS customers
FROM customers
GROUP BY country
ORDER BY customers DESC;
```
**Golden rule:** every column in `SELECT` must be either inside an aggregate function **or** listed in `GROUP BY`.

```sql
-- Revenue per order
SELECT order_id,
       SUM(quantity * unit_price * (1 - discount)) AS order_total
FROM order_items
GROUP BY order_id
ORDER BY order_total DESC;

-- Group by several columns
SELECT country, city, COUNT(*) AS customers
FROM customers
GROUP BY country, city;

-- Group by an expression (orders per year)
SELECT EXTRACT(YEAR FROM order_date) AS year, COUNT(*) AS orders
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date)
ORDER BY year;
```

## 3. HAVING — filter groups
```sql
SELECT customer_id, COUNT(*) AS orders
FROM orders
WHERE status <> 'cancelled'        -- filters ROWS before grouping
GROUP BY customer_id
HAVING COUNT(*) >= 3               -- filters GROUPS after grouping
ORDER BY orders DESC;
```
| | `WHERE` | `HAVING` |
|---|---|---|
| Filters | rows | groups |
| Runs | before `GROUP BY` | after `GROUP BY` |
| Can use aggregates? | ❌ | ✅ |

## 4. Conditional aggregation (preview of CASE)
```sql
SELECT COUNT(*) AS all_orders,
       COUNT(*) FILTER (WHERE status = 'shipped') AS shipped,          -- PostgreSQL/DuckDB
       SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled   -- works everywhere
FROM orders;
```

## 5. ROLLUP — subtotals and grand total
```sql
SELECT country, city, COUNT(*) AS customers
FROM customers
GROUP BY ROLLUP (country, city)
ORDER BY country, city;
```

---

## 🧪 Practice
1. Number of products and average price per category id.
2. Number of orders per status.
3. Customers (id) with more than 2 orders.
4. Total units sold per product, only products with ≥ 3 units.
5. Orders per month in 2025.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| aggregate | /ˈæɡ.rɪ.ɡət/ (n.) | **A**-gri-guet | agregado |
| count | /kaʊnt/ | kaunt | contar / conteo |
| sum | /sʌm/ | sam | suma |
| average | /ˈæv.ɚ.ɪdʒ/ | **A**-ve-rich | promedio |
| summarise | /ˈsʌm.ə.raɪz/ | **SA**-ma-raiz | resumir |
| subtotal | /ˈsʌbˌtoʊ.t̬əl/ | **SAB**-tou-tal | subtotal |
| grand total | /ɡrænd ˈtoʊ.t̬əl/ | grand **TOU**-tal | total general |
| revenue | /ˈrev.ə.nuː/ | **RE**-ve-nu | ingresos |

### 💬 Useful sentences
- *"`HAVING` filters groups, **whereas** `WHERE` filters individual rows."*
- *"We **grouped** sales **by** country and **kept only** countries **with more than** ten orders."*

---
⬅️ [[SQL-B01 SELECT, WHERE and ORDER BY]] · ➡️ [[SQL-B03 JOINs]] · 🏠 [[00 - SQL Roadmap]]

#sql #beginner
