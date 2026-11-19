---
title: "SQL-A03 Advanced Analytics Patterns"
level: advanced
tags: [sql, advanced, recursive-cte, cohorts, funnels, json, gaps-and-islands]
prev: "[[SQL-A02 Transactions, Isolation and Locking in Practice]]"
next: "[[SQL-A04 Functions, Procedures and Triggers]]"
---

# SQL-A03 — Advanced Analytics Patterns

## 🎯 Learning objectives
Solve the problems that appear in real analytics work and senior interviews:
recursive hierarchies, date spines, gaps & islands, sessionisation, cohort retention, funnels, percentiles, pivots and JSON.

---

## 1. Recursive CTEs — hierarchies
```sql
WITH RECURSIVE org AS (
    SELECT id, name, manager_id, 1 AS level, name::TEXT AS path     -- anchor: the CEO
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, org.level + 1, org.path || ' > ' || e.name
    FROM employees e
    JOIN org ON e.manager_id = org.id                                 -- recursive step
)
SELECT level, name, path FROM org ORDER BY path;
```
Always guarantee termination (a `WHERE level < 20` guard protects against cycles).

## 2. Date spine — fill missing days
```sql
WITH days AS (
    SELECT d::DATE AS day
    FROM generate_series(DATE '2026-01-01', DATE '2026-03-31', INTERVAL '1 day') AS g(d)
),
daily AS (
    SELECT order_date, COUNT(*) AS orders FROM orders GROUP BY order_date
)
SELECT days.day, COALESCE(daily.orders, 0) AS orders
FROM days LEFT JOIN daily ON daily.order_date = days.day
ORDER BY days.day;
```

## 3. Gaps and islands — consecutive streaks
Classic trick: `date - ROW_NUMBER()` is constant inside a run of consecutive dates.
```sql
-- Streaks of consecutive login days per user (table logins(user_id, login_date))
WITH d AS (
    SELECT DISTINCT user_id, login_date FROM logins
),
g AS (
    SELECT user_id, login_date,
           login_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date))::INT AS grp
    FROM d
)
SELECT user_id, MIN(login_date) AS streak_start, MAX(login_date) AS streak_end, COUNT(*) AS days
FROM g
GROUP BY user_id, grp
HAVING COUNT(*) >= 3
ORDER BY days DESC;
```

## 4. Sessionisation — split events by inactivity
```sql
-- events(user_id, ts): new session after 30 minutes of inactivity
WITH flagged AS (
    SELECT user_id, ts,
           CASE WHEN ts - LAG(ts) OVER (PARTITION BY user_id ORDER BY ts) > INTERVAL '30 minutes'
                  OR LAG(ts) OVER (PARTITION BY user_id ORDER BY ts) IS NULL
                THEN 1 ELSE 0 END AS new_session
    FROM events
)
SELECT user_id, ts,
       SUM(new_session) OVER (PARTITION BY user_id ORDER BY ts) AS session_id
FROM flagged;
```

## 5. Cohort retention
```sql
WITH first_order AS (
    SELECT customer_id, DATE_TRUNC('month', MIN(order_date)) AS cohort_month
    FROM orders WHERE status IN ('paid','shipped')
    GROUP BY customer_id
),
activity AS (
    SELECT DISTINCT o.customer_id, DATE_TRUNC('month', o.order_date) AS active_month
    FROM orders o WHERE o.status IN ('paid','shipped')
),
cohorts AS (
    SELECT f.cohort_month,
           (EXTRACT(YEAR FROM a.active_month) - EXTRACT(YEAR FROM f.cohort_month)) * 12
         + (EXTRACT(MONTH FROM a.active_month) - EXTRACT(MONTH FROM f.cohort_month)) AS month_number,
           a.customer_id
    FROM first_order f JOIN activity a USING (customer_id)
)
SELECT cohort_month, month_number, COUNT(DISTINCT customer_id) AS active_customers,
       ROUND(100.0 * COUNT(DISTINCT customer_id)
             / FIRST_VALUE(COUNT(DISTINCT customer_id)) OVER (PARTITION BY cohort_month ORDER BY month_number), 1) AS retention_pct
FROM cohorts
GROUP BY cohort_month, month_number
ORDER BY cohort_month, month_number;
```

## 6. Funnel conversion
```sql
-- events(user_id, event_name, ts) with steps: view → add_to_cart → checkout → purchase
WITH steps AS (
    SELECT user_id,
           MIN(ts) FILTER (WHERE event_name = 'view')        AS t_view,
           MIN(ts) FILTER (WHERE event_name = 'add_to_cart') AS t_cart,
           MIN(ts) FILTER (WHERE event_name = 'checkout')    AS t_checkout,
           MIN(ts) FILTER (WHERE event_name = 'purchase')    AS t_purchase
    FROM events GROUP BY user_id
)
SELECT COUNT(t_view)                                              AS viewed,
       COUNT(t_cart)     FILTER (WHERE t_cart > t_view)           AS carted,
       COUNT(t_checkout) FILTER (WHERE t_checkout > t_cart)       AS checked_out,
       COUNT(t_purchase) FILTER (WHERE t_purchase > t_checkout)   AS purchased
FROM steps;
```

## 7. Percentiles and statistics
```sql
SELECT category_id,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price,
       PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY price) AS p90_price,
       STDDEV_SAMP(price) AS sd_price,
       CORR(price, stock)  AS corr_price_stock
FROM products
GROUP BY category_id;
```

## 8. Pareto (80/20) analysis
```sql
WITH rev AS (
    SELECT p.name, SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS revenue
    FROM order_items oi JOIN products p ON p.id = oi.product_id
    GROUP BY p.name
)
SELECT name, ROUND(revenue, 2) AS revenue,
       ROUND(100 * SUM(revenue) OVER (ORDER BY revenue DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
             / SUM(revenue) OVER (), 1) AS cumulative_pct
FROM rev
ORDER BY revenue DESC;
```

## 9. JSON in PostgreSQL (JSONB)
```sql
CREATE TABLE api_events (id SERIAL PRIMARY KEY, payload JSONB);
INSERT INTO api_events (payload) VALUES
 ('{"user": {"id": 1, "tier": "gold"}, "items": [{"sku": "A1", "qty": 2}, {"sku": "B7", "qty": 1}]}');

SELECT payload -> 'user' ->> 'tier'          AS tier,          -- -> json, ->> text
       (payload #>> '{user,id}')::INT        AS user_id,
       jsonb_array_length(payload -> 'items') AS n_items
FROM api_events
WHERE payload @> '{"user": {"tier": "gold"}}';                  -- containment (GIN-indexable)

SELECT e.id, item ->> 'sku' AS sku, (item ->> 'qty')::INT AS qty
FROM api_events e, jsonb_array_elements(e.payload -> 'items') AS item;   -- explode array → rows

CREATE INDEX idx_events_payload ON api_events USING GIN (payload jsonb_path_ops);
```

## 10. Pivot / unpivot
```sql
-- PostgreSQL: conditional aggregation (portable); DuckDB has native PIVOT/UNPIVOT
SELECT c.country,
       SUM(CASE WHEN EXTRACT(YEAR FROM o.order_date) = 2025 THEN 1 ELSE 0 END) AS orders_2025,
       SUM(CASE WHEN EXTRACT(YEAR FROM o.order_date) = 2026 THEN 1 ELSE 0 END) AS orders_2026
FROM orders o JOIN customers c ON c.id = o.customer_id
GROUP BY c.country;
```

---

## 🧪 Practice
1. Print the employee hierarchy with indentation by level.
2. Build a daily revenue series for 2025 with zeros for days without sales and a 7-day moving average.
3. Find each customer's longest gap (days) between orders.
4. Build a monthly cohort table for DataShop.
5. Which products make up the top 80% of revenue?

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| recursive | /rɪˈkɝː.sɪv/ | ri-**KER**-siv | recursivo |
| hierarchy | /ˈhaɪ.ɚ.ɑːr.ki/ | **JÁI**-e-rar-ki | jerarquía |
| anchor | /ˈæŋ.kɚ/ | **AN**-ker | ancla (caso base) |
| spine | /spaɪn/ | spain | columna (serie base de fechas) |
| streak | /striːk/ | strik | racha |
| gap | /ɡæp/ | gap | hueco / brecha |
| cohort | /ˈkoʊ.hɔːrt/ | **KOU**-jort | cohorte |
| retention | /rɪˈten.ʃən/ | ri-**TEN**-shon | retención |
| funnel | /ˈfʌn.əl/ | **FA**-nel | embudo |
| conversion | /kənˈvɝː.ʒən/ | kon-**VER**-yon | conversión |
| percentile | /pɚˈsen.taɪl/ | per-**SEN**-tail | percentil |
| median | /ˈmiː.di.ən/ | **MI**-di-an | mediana |

### 💬 Useful sentences
- *"Retention **drops sharply** after the first month and **levels off** around 20%."*
- *"The biggest **drop-off** in the funnel **happens between** cart and checkout."*

---
⬅️ [[SQL-A02 Transactions, Isolation and Locking in Practice]] · ➡️ [[SQL-A04 Functions, Procedures and Triggers]] · 🏠 [[00 - SQL Roadmap]]

#sql #advanced
