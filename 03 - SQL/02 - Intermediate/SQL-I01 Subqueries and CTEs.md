---
title: "SQL-I01 Subqueries and CTEs"
level: intermediate
tags: [sql, intermediate, subqueries, cte]
prev: "[[SQL-B04 Creating Tables - DDL, DML and Constraints]]"
next: "[[SQL-I02 CASE, NULLs, Strings and Dates]]"
---

# SQL-I01 — Subqueries and CTEs

## 🎯 Learning objectives
- Use subqueries in `WHERE`, `FROM` and `SELECT`.
- Understand correlated subqueries and `EXISTS`.
- Write readable multi-step queries with CTEs (`WITH`).

---

## 1. Subquery in WHERE
```sql
-- Products more expensive than the average
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);      -- scalar subquery (one value)

-- Customers who bought at least one laptop
SELECT first_name, last_name
FROM customers
WHERE id IN (
    SELECT o.customer_id
    FROM orders o
    JOIN order_items oi ON oi.order_id = o.id
    JOIN products p ON p.id = oi.product_id
    WHERE p.category_id = 1
);
```

## 2. EXISTS / NOT EXISTS (correlated subquery)
A **correlated** subquery references the outer query and is evaluated per outer row (the optimiser usually turns it into a semi-join).
```sql
-- Customers with at least one paid order
SELECT c.id, c.first_name
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id AND o.status = 'paid');

-- Products never sold
SELECT p.name
FROM products p
WHERE NOT EXISTS (SELECT 1 FROM order_items oi WHERE oi.product_id = p.id);
```
⚠️ `NOT IN (subquery)` returns **nothing** if the subquery contains a NULL. Prefer `NOT EXISTS`.

## 3. Subquery in FROM (derived table)
```sql
SELECT AVG(order_total) AS avg_ticket
FROM (
    SELECT order_id, SUM(quantity * unit_price * (1 - discount)) AS order_total
    FROM order_items
    GROUP BY order_id
) AS t;
```

## 4. Subquery in SELECT
```sql
SELECT c.first_name,
       (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.id) AS order_count
FROM customers c;
```

## 5. CTEs — Common Table Expressions
A CTE names a subquery so you can read the query **top to bottom** like steps of a recipe.
```sql
WITH order_totals AS (                       -- step 1: total per order
    SELECT order_id,
           SUM(quantity * unit_price * (1 - discount)) AS order_total
    FROM order_items
    GROUP BY order_id
),
valid_orders AS (                            -- step 2: only paid/shipped
    SELECT o.id, o.customer_id, ot.order_total
    FROM orders o
    JOIN order_totals ot ON ot.order_id = o.id
    WHERE o.status IN ('paid', 'shipped')
),
customer_value AS (                          -- step 3: total per customer
    SELECT customer_id, COUNT(*) AS orders, SUM(order_total) AS lifetime_value
    FROM valid_orders
    GROUP BY customer_id
)
SELECT c.first_name, c.country, cv.orders, ROUND(cv.lifetime_value, 2) AS lifetime_value
FROM customer_value cv
JOIN customers c ON c.id = cv.customer_id
ORDER BY lifetime_value DESC;
```
### Why CTEs?
- Readability and debugging: run each step on its own.
- Reuse the same intermediate result several times.
- Avoid **fan-out**: aggregate each "many" table in its own CTE, then join.
- Enable recursion → [[SQL-A03 Advanced Analytics Patterns]].

> In PostgreSQL 12+ CTEs are inlined by the optimiser (fast). Use `WITH x AS MATERIALIZED (...)` to force computing once.

## 6. Comparing with ANY / ALL
```sql
SELECT name, price FROM products
WHERE price > ALL (SELECT price FROM products WHERE category_id = 4);   -- pricier than every book
```

---

## 🧪 Practice
1. Customers whose lifetime value is above the average customer value.
2. The most expensive product of each category (subquery or CTE).
3. Orders whose total is above that customer's own average order (correlated).
4. Rewrite one of your long queries as a chain of CTEs.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| subquery | /ˈsʌbˌkwɪr.i/ | **SAB**-kui-ri | subconsulta |
| nested | /ˈnes.tɪd/ | **NES**-tid | anidado |
| correlated | /ˈkɔːr.ə.leɪ.t̬ɪd/ | **KO**-re-lei-tid | correlacionado |
| derived | /dɪˈraɪvd/ | di-**RÁIVD** | derivado |
| step | /step/ | step | paso |
| readable | /ˈriː.də.bəl/ | **RI**-da-bol | legible |
| lifetime value | /ˈlaɪf.taɪm ˈvæl.juː/ | **LÁIF**-taim **VAL**-yu | valor de vida del cliente |
| intermediate | /ˌɪn.t̬ɚˈmiː.di.ət/ | in-ter-**MI**-di-et | intermedio |

### 💬 Useful sentences
- *"**Let's break** this query **into** CTEs so it's easier to debug."*
- *"`NOT EXISTS` **is safer than** `NOT IN` **when** the subquery may return NULLs."*

---
⬅️ [[SQL-B04 Creating Tables - DDL, DML and Constraints]] · ➡️ [[SQL-I02 CASE, NULLs, Strings and Dates]] · 🏠 [[00 - SQL Roadmap]]

#sql #intermediate
