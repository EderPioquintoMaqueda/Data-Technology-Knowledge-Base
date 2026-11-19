---
title: "SQL-I02 CASE, NULLs, Strings and Dates"
level: intermediate
tags: [sql, intermediate, case, null, dates, strings]
prev: "[[SQL-I01 Subqueries and CTEs]]"
next: "[[SQL-I03 Window Functions]]"
---

# SQL-I02 — CASE, NULLs, Strings and Dates

## 🎯 Learning objectives
- Write conditional logic with `CASE`.
- Handle NULLs with `COALESCE`, `NULLIF` and three-valued logic.
- Clean and transform text.
- Work with dates: extract, truncate, add intervals, compute differences.

Examples use PostgreSQL/DuckDB syntax; differences with MySQL/SQL Server are noted.

---

## 1. CASE
```sql
SELECT name, price,
       CASE
           WHEN price >= 1000 THEN 'premium'
           WHEN price >= 100  THEN 'mid'
           ELSE 'budget'
       END AS price_tier
FROM products;
```
### Pivot with CASE (conditional aggregation)
```sql
SELECT EXTRACT(YEAR FROM order_date) AS year,
       SUM(CASE WHEN status = 'shipped'   THEN 1 ELSE 0 END) AS shipped,
       SUM(CASE WHEN status = 'paid'      THEN 1 ELSE 0 END) AS paid,
       SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) AS cancelled
FROM orders
GROUP BY 1
ORDER BY 1;
```
### Custom sort order
```sql
SELECT id, status FROM orders
ORDER BY CASE status WHEN 'pending' THEN 1 WHEN 'paid' THEN 2 WHEN 'shipped' THEN 3 ELSE 4 END;
```

## 2. NULL — "unknown"
| Expression | Result |
|---|---|
| `NULL = NULL` | NULL (not TRUE!) |
| `NULL + 5` | NULL |
| `TRUE AND NULL` | NULL |
| `FALSE AND NULL` | FALSE |
| `COUNT(col)` | ignores NULLs |

```sql
SELECT first_name, COALESCE(email, 'no-email') AS email      -- first non-null value
FROM customers;

SELECT order_id, quantity * unit_price / NULLIF(quantity, 0)  -- NULLIF avoids division by zero
FROM order_items;

SELECT * FROM orders WHERE employee_id IS DISTINCT FROM 3;    -- NULL-safe comparison (includes NULL rows)
```
MySQL: `IFNULL(x, y)` · SQL Server: `ISNULL(x, y)` · all: `COALESCE`.

## 3. Strings
```sql
SELECT UPPER(first_name), LOWER(email), LENGTH(last_name),
       first_name || ' ' || last_name           AS full_name,      -- MySQL/SQL Server: CONCAT(...)
       CONCAT_WS(', ', city, country)           AS location,
       TRIM('  hi  '), LTRIM('  hi'), RTRIM('hi  '),
       SUBSTRING(email FROM 1 FOR 3)            AS first3,         -- also LEFT(email, 3)
       SPLIT_PART(email, '@', 2)                AS domain,         -- PostgreSQL/DuckDB
       POSITION('@' IN email)                   AS at_pos,
       REPLACE(city, ' ', '_')                  AS city_slug,
       LPAD(CAST(id AS VARCHAR), 5, '0')        AS padded_id
FROM customers;

-- Pattern matching
SELECT * FROM customers WHERE email ~ '^[a-z.]+@mail\.com$';          -- regex (PostgreSQL)
SELECT REGEXP_REPLACE('Tel: 55-1234-5678', '[^0-9]', '', 'g');         -- '5512345678'
```

## 4. Dates and times
```sql
SELECT CURRENT_DATE, CURRENT_TIMESTAMP, NOW();

SELECT order_date,
       EXTRACT(YEAR  FROM order_date)  AS year,
       EXTRACT(MONTH FROM order_date)  AS month,
       EXTRACT(DOW   FROM order_date)  AS weekday,          -- 0 = Sunday (PostgreSQL)
       DATE_TRUNC('month', order_date) AS month_start,      -- great for GROUP BY
       order_date + INTERVAL '30 days' AS due_date,
       CURRENT_DATE - order_date       AS days_ago,         -- integer in PostgreSQL
       TO_CHAR(order_date, 'YYYY-MM')  AS year_month,       -- PostgreSQL; DuckDB: strftime(order_date, '%Y-%m')
       AGE(CURRENT_DATE, order_date)   AS age               -- PostgreSQL interval
FROM orders;

-- Monthly orders
SELECT DATE_TRUNC('month', order_date) AS month, COUNT(*) AS orders
FROM orders
GROUP BY 1
ORDER BY 1;
```
| Task | PostgreSQL / DuckDB | MySQL | SQL Server |
|---|---|---|---|
| Truncate to month | `DATE_TRUNC('month', d)` | `DATE_FORMAT(d,'%Y-%m-01')` | `DATETRUNC(month, d)` (2022+) |
| Add days | `d + INTERVAL '7 days'` | `DATE_ADD(d, INTERVAL 7 DAY)` | `DATEADD(day, 7, d)` |
| Difference in days | `d2 - d1` | `DATEDIFF(d2, d1)` | `DATEDIFF(day, d1, d2)` |

⚠️ For timestamp ranges, use half-open intervals: `WHERE ts >= '2026-11-01' AND ts < '2026-12-01'` (not `BETWEEN`, which misses times on the last day).

## 5. Type conversion
```sql
SELECT CAST('42' AS INTEGER), '2026-11-18'::DATE, CAST(price AS VARCHAR), ROUND(price::NUMERIC, 0)
FROM products LIMIT 1;
```

---

## 🧪 Practice
1. Classify customers as 'new' (signed up in 2026) or 'existing'.
2. Show each email's domain and count customers per domain.
3. Orders per weekday name.
4. Days between each customer's signup and their first order.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| null | /nʌl/ | nal | nulo |
| unknown | /ʌnˈnoʊn/ | an-**NOUN** | desconocido |
| fallback | /ˈfɑːl.bæk/ | **FOL**-bak | valor de respaldo |
| truncate | /ˈtrʌŋ.keɪt/ | **TRAN**-keit | truncar |
| interval | /ˈɪn.t̬ɚ.vəl/ | **IN**-ter-val | intervalo |
| leading zeros | /ˈliː.dɪŋ ˈzɪr.oʊz/ | **LI**-ding **ZI**-rouz | ceros a la izquierda |
| half-open | /ˌhæf ˈoʊ.pən/ | jaf **OU**-pen | semiabierto |
| weekday | /ˈwiːk.deɪ/ | **UIK**-dei | día de la semana |

### 💬 Useful sentences
- *"`COALESCE` **returns the first** non-null argument."*
- *"We **truncated** the timestamps **to the month** to build the trend."*

---
⬅️ [[SQL-I01 Subqueries and CTEs]] · ➡️ [[SQL-I03 Window Functions]] · 🏠 [[00 - SQL Roadmap]]

#sql #intermediate
