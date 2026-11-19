---
title: "SQL-I04 Set Operations, Views and Data Modeling in Practice"
level: intermediate
tags: [sql, intermediate, union, views, star-schema]
prev: "[[SQL-I03 Window Functions]]"
next: "[[SQL-A01 Query Optimization and Indexes]]"
---

# SQL-I04 — Set Operations, Views and Data Modeling in Practice

## 🎯 Learning objectives
- Combine result sets with `UNION`, `INTERSECT`, `EXCEPT`.
- Encapsulate logic in views and materialized views.
- Design a simple star schema for analytics.

---

## 1. Set operations
Both queries must return the **same number of columns** with compatible types.
```sql
-- All countries where we have customers OR suppliers
SELECT country FROM customers
UNION                              -- removes duplicates
SELECT country FROM suppliers;

SELECT country FROM customers
UNION ALL                          -- keeps duplicates, faster
SELECT country FROM suppliers;

-- Customers who ordered in 2025 AND in 2026
SELECT customer_id FROM orders WHERE order_date <  '2026-01-01'
INTERSECT
SELECT customer_id FROM orders WHERE order_date >= '2026-01-01';

-- Customers who ordered in 2025 but NOT in 2026 (churned)
SELECT customer_id FROM orders WHERE order_date <  '2026-01-01'
EXCEPT                              -- Oracle: MINUS
SELECT customer_id FROM orders WHERE order_date >= '2026-01-01';
```
💡 Use `UNION ALL` unless you really need de-duplication — it avoids a sort.

## 2. Views
A **view** is a saved query that behaves like a table.
```sql
CREATE OR REPLACE VIEW v_order_totals AS
SELECT o.id AS order_id, o.customer_id, o.order_date, o.status,
       SUM(oi.quantity * oi.unit_price * (1 - oi.discount)) AS total
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
GROUP BY o.id, o.customer_id, o.order_date, o.status;

SELECT customer_id, SUM(total) FROM v_order_totals WHERE status = 'shipped' GROUP BY customer_id;
```
Benefits: one definition of business logic ("what is revenue?"), simpler queries for analysts, security (expose only some columns).

### Materialized views (PostgreSQL)
Store the result physically — fast reads, must be refreshed.
```sql
CREATE MATERIALIZED VIEW mv_monthly_revenue AS
SELECT DATE_TRUNC('month', order_date) AS month, SUM(total) AS revenue
FROM v_order_totals WHERE status IN ('paid','shipped') GROUP BY 1;

REFRESH MATERIALIZED VIEW mv_monthly_revenue;
```

## 3. From OLTP tables to a star schema
Our DataShop tables are **normalised** (good for writing). Analytics prefers a **star schema** (good for reading):
```
                 dim_date
                    │
 dim_customer ── fact_sales ── dim_product
                    │
               dim_employee
```
| Table type | Contains | Example columns |
|---|---|---|
| **Fact** | measurable events, many rows | `date_key`, `customer_key`, `product_key`, `quantity`, `revenue` |
| **Dimension** | descriptive context | `customer_name`, `country`, `category`, `month_name` |

```sql
CREATE TABLE fact_sales AS
SELECT o.order_date                     AS date_key,
       o.customer_id                    AS customer_key,
       oi.product_id                    AS product_key,
       o.employee_id                    AS employee_key,
       oi.quantity,
       oi.quantity * oi.unit_price * (1 - oi.discount) AS revenue
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
WHERE o.status IN ('paid','shipped');

CREATE TABLE dim_date AS
SELECT d::DATE                       AS date_key,
       EXTRACT(YEAR FROM d)          AS year,
       EXTRACT(QUARTER FROM d)       AS quarter,
       EXTRACT(MONTH FROM d)         AS month,
       EXTRACT(ISODOW FROM d) IN (6,7) AS is_weekend
FROM generate_series(DATE '2025-01-01', DATE '2026-12-31', INTERVAL '1 day') AS g(d);
```
Theory: [[04 - Conceptual Logical and Physical Models]] · [[06 - Schema Design Principles]] · [[05 - Data Warehouses]] · BI: [[Power BI]].

### Slowly Changing Dimensions (SCD)
- **Type 1:** overwrite the old value (no history).
- **Type 2:** add a new row with `valid_from`, `valid_to`, `is_current` (full history) — standard in warehouses (dbt snapshots).

---

## 🧪 Practice
1. Customers who bought both a laptop and a book (INTERSECT).
2. Create a view `v_customer_value` and query the top 5.
3. Build `dim_product` (product + category) and query revenue by category from the star schema.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| union | /ˈjuː.njən/ | **YU**-nion | unión |
| intersection | /ˌɪn.t̬ɚˈsek.ʃən/ | in-ter-**SEK**-shon | intersección |
| except | /ɪkˈsept/ | ik-**SEPT** | excepto |
| view | /vjuː/ | viu | vista |
| materialized | /məˈtɪr.i.ə.laɪzd/ | ma-**TI**-ria-laizd | materializada |
| refresh | /rɪˈfreʃ/ | ri-**FRESH** | actualizar / refrescar |
| fact table | /fækt ˈteɪ.bəl/ | fakt **TÉI**-bol | tabla de hechos |
| dimension | /daɪˈmen.ʃən/ | dai-**MEN**-shon | dimensión |
| churn | /tʃɝːn/ | chern | abandono (de clientes) |
| star schema | /stɑːr ˈskiː.mə/ | star **SKI**-ma | esquema estrella |

### 💬 Useful sentences
- *"The view **encapsulates** the revenue logic so everyone **uses the same definition**."*
- *"Fact tables **store** events; dimension tables **describe** them."*

---
⬅️ [[SQL-I03 Window Functions]] · ➡️ [[SQL-A01 Query Optimization and Indexes]] · 🏠 [[00 - SQL Roadmap]]

#sql #intermediate
