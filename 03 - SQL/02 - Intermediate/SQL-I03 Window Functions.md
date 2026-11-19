---
title: "SQL-I03 Window Functions"
level: intermediate
tags: [sql, intermediate, window-functions, analytics]
prev: "[[SQL-I02 CASE, NULLs, Strings and Dates]]"
next: "[[SQL-I04 Set Operations, Views and Data Modeling in Practice]]"
---

# SQL-I03 — Window Functions

## 🎯 Learning objectives
- Understand `OVER (PARTITION BY ... ORDER BY ...)`.
- Rank rows (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`).
- Compare rows (`LAG`, `LEAD`, `FIRST_VALUE`).
- Compute running totals and moving averages with frames.

> A window function calculates over a **set of rows related to the current row** — without collapsing them like `GROUP BY` does. Each row keeps its identity and gains a new column.

---

## 1. Anatomy
```sql
function_name(...) OVER (
    PARTITION BY col      -- split into groups (optional)
    ORDER BY col          -- order inside each group (optional)
    ROWS BETWEEN ...      -- frame: which rows around the current one (optional)
)
```

## 2. Aggregates as windows
```sql
WITH order_totals AS (
    SELECT o.id, o.customer_id, o.order_date,
           SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS total
    FROM orders o JOIN order_items oi ON oi.order_id = o.id
    WHERE o.status IN ('paid','shipped')
    GROUP BY o.id, o.customer_id, o.order_date
)
SELECT id, customer_id, order_date, ROUND(total, 2) AS total,
       ROUND(SUM(total) OVER (), 2)                                  AS grand_total,
       ROUND(100 * total / SUM(total) OVER (), 2)                    AS pct_of_all,
       ROUND(SUM(total) OVER (PARTITION BY customer_id), 2)          AS customer_total,
       COUNT(*) OVER (PARTITION BY customer_id)                      AS customer_orders
FROM order_totals
ORDER BY customer_id, order_date;
```

## 3. Ranking
```sql
SELECT name, category_id, price,
       ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS row_num,
       RANK()       OVER (PARTITION BY category_id ORDER BY price DESC) AS rnk,
       DENSE_RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS dense_rnk,
       NTILE(4)     OVER (ORDER BY price)                               AS price_quartile
FROM products;
```
| Prices | 100 | 90 | 90 | 80 |
|---|---|---|---|---|
| `ROW_NUMBER` | 1 | 2 | 3 | 4 |
| `RANK` | 1 | 2 | 2 | **4** |
| `DENSE_RANK` | 1 | 2 | 2 | **3** |

### Top-N per group (very common interview question)
```sql
WITH ranked AS (
    SELECT name, category_id, price,
           ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rn
    FROM products
)
SELECT * FROM ranked WHERE rn <= 2;
-- DuckDB / Snowflake / BigQuery shortcut: ... QUALIFY rn <= 2
```

### Deduplicate — keep the latest row per key
```sql
WITH latest AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM orders
)
SELECT id, customer_id, order_date, status FROM latest WHERE rn = 1;
```

## 4. LAG / LEAD — compare with previous/next rows
```sql
SELECT customer_id, id AS order_id, order_date,
       LAG(order_date)  OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_order_date,
       order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS days_since_prev,
       LEAD(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_order_date,
       FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS first_order
FROM orders
ORDER BY customer_id, order_date;
```

### Month-over-month growth
```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', o.order_date) AS month,
           SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS revenue
    FROM orders o JOIN order_items oi ON oi.order_id = o.id
    WHERE o.status IN ('paid','shipped')
    GROUP BY 1
)
SELECT month, ROUND(revenue, 2) AS revenue,
       ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
             / NULLIF(LAG(revenue) OVER (ORDER BY month), 0), 1) AS mom_growth_pct
FROM monthly
ORDER BY month;
```

## 5. Frames — running totals and moving averages
```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', order_date) AS month, COUNT(*) AS orders
    FROM orders GROUP BY 1
)
SELECT month, orders,
       SUM(orders) OVER (ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
       ROUND(AVG(orders) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS moving_avg_3m
FROM monthly
ORDER BY month;
```
| Frame | Meaning |
|---|---|
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | from the first row to the current one (running) |
| `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` | last 3 rows |
| `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` | centred window of 3 |
| `RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW` | time-based (PostgreSQL, DuckDB) |

⚠️ With `ORDER BY` and no frame, the default is `RANGE ... CURRENT ROW`, which groups **ties** together. Specify `ROWS` explicitly for predictable results.

### Named windows
```sql
SELECT name, price,
       AVG(price) OVER w, MIN(price) OVER w, MAX(price) OVER w
FROM products
WINDOW w AS (PARTITION BY category_id);
```

## 6. Where can window functions go?
Only in `SELECT` and `ORDER BY` (they run *after* `WHERE`/`GROUP BY`/`HAVING`). To filter on them, wrap in a CTE/subquery (or use `QUALIFY`).

pandas equivalent: `groupby().transform()`, `rank()`, `shift()`, `rolling()` → [[pandas_groupby_aggregation]] · Polars: `.over()` → [[Polars - Beginner to Advanced]].

---

## 🧪 Practice
1. Rank customers by lifetime value within their country.
2. For each order, show days since that customer's previous order.
3. Cumulative revenue by month.
4. Keep only the most recent order of each customer.
5. Each product's price as % of its category's maximum.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| window | /ˈwɪn.doʊ/ | **UIN**-dou | ventana |
| partition | /pɑːrˈtɪʃ.ən/ | par-**TI**-shon | partición |
| frame | /freɪm/ | freim | marco |
| preceding | /priˈsiː.dɪŋ/ | pri-**SI**-ding | anterior / precedente |
| following | /ˈfɑː.loʊ.ɪŋ/ | **FA**-lou-ing | siguiente |
| unbounded | /ʌnˈbaʊn.dɪd/ | an-**BÁUN**-did | sin límite |
| running total | /ˈrʌn.ɪŋ ˈtoʊ.t̬əl/ | **RA**-ning **TOU**-tal | total acumulado |
| tie | /taɪ/ | tai | empate |
| growth | /ɡroʊθ/ | grouz | crecimiento |

### 💬 Useful sentences
- *"Unlike GROUP BY, a window function **doesn't collapse** the rows."*
- *"We **partitioned by** customer and **ordered by** date **to get** the previous purchase."*

---
⬅️ [[SQL-I02 CASE, NULLs, Strings and Dates]] · ➡️ [[SQL-I04 Set Operations, Views and Data Modeling in Practice]] · 🏠 [[00 - SQL Roadmap]]

#sql #intermediate
