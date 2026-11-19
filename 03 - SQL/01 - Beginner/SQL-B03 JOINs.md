---
title: "SQL-B03 JOINs"
level: beginner
tags: [sql, beginner, joins]
prev: "[[SQL-B02 Aggregations - GROUP BY and HAVING]]"
next: "[[SQL-B04 Creating Tables - DDL, DML and Constraints]]"
---

# SQL-B03 — JOINs

## 🎯 Learning objectives
- Combine tables using primary/foreign keys.
- Choose between INNER, LEFT, RIGHT, FULL and CROSS joins.
- Write self joins and multi-table joins.
- Avoid the classic mistakes: row multiplication and filtering away LEFT JOIN rows.

---

## 1. Why joins?
Normalised databases split data into tables ([[05 - Normalization]]). `orders.customer_id` is a **foreign key** pointing to `customers.id` (the **primary key**). A join rebuilds the full picture.

## 2. INNER JOIN — only matching rows
```sql
SELECT o.id AS order_id, o.order_date, c.first_name, c.country
FROM orders AS o
INNER JOIN customers AS c ON c.id = o.customer_id;
```
Table aliases (`o`, `c`) keep queries short. `JOIN` alone means `INNER JOIN`.

## 3. LEFT JOIN — keep everything from the left table
```sql
-- All customers, with their orders if they have any
SELECT c.id, c.first_name, o.id AS order_id
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
ORDER BY c.id;
```
Customers without orders appear with `order_id = NULL`.

### Anti-join: rows WITHOUT a match
```sql
SELECT c.id, c.first_name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE o.id IS NULL;            -- customers who never ordered
```

## 4. Visual summary
```
 INNER          LEFT           RIGHT          FULL OUTER
 (A ∩ B)        (A + match)    (B + match)    (A ∪ B)
  ○●○            ●●○            ○●●            ●●●
```
| Join | Returns |
|---|---|
| `INNER JOIN` | rows with a match in both tables |
| `LEFT JOIN` | all left rows + matches (NULL if none) |
| `RIGHT JOIN` | all right rows + matches (rarely used; swap tables and use LEFT) |
| `FULL OUTER JOIN` | all rows from both sides |
| `CROSS JOIN` | every combination (Cartesian product) |

## 5. Joining several tables
```sql
SELECT o.id            AS order_id,
       c.first_name || ' ' || c.last_name AS customer,
       p.name          AS product,
       cat.name        AS category,
       oi.quantity,
       oi.quantity * oi.unit_price * (1 - oi.discount) AS line_total
FROM orders o
JOIN customers   c   ON c.id = o.customer_id
JOIN order_items oi  ON oi.order_id = o.id
JOIN products    p   ON p.id = oi.product_id
JOIN categories  cat ON cat.id = p.category_id
WHERE o.status IN ('paid', 'shipped')
ORDER BY o.id;
```

### Join + aggregate: revenue per category
```sql
SELECT cat.name AS category,
       ROUND(SUM(oi.quantity * oi.unit_price * (1 - oi.discount)), 2) AS revenue
FROM order_items oi
JOIN products   p   ON p.id = oi.product_id
JOIN categories cat ON cat.id = p.category_id
JOIN orders     o   ON o.id = oi.order_id
WHERE o.status IN ('paid', 'shipped')
GROUP BY cat.name
ORDER BY revenue DESC;
```

## 6. Self join — a table joined to itself
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON m.id = e.manager_id;
```

## 7. CROSS JOIN — combinations
```sql
SELECT c.name AS category, s.status
FROM categories c
CROSS JOIN (SELECT DISTINCT status FROM orders) s;   -- a "skeleton" for reports
```

## 8. ⚠️ Classic mistakes
1. **Filtering the right table in WHERE turns a LEFT JOIN into an INNER JOIN.**
```sql
-- ❌ loses customers with no 2026 orders
SELECT c.first_name, o.id FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE o.order_date >= '2026-01-01';
-- ✅ put the condition in ON
SELECT c.first_name, o.id FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id AND o.order_date >= '2026-01-01';
```
2. **Row multiplication (fan-out):** joining two "many" tables to the same parent multiplies rows and inflates `SUM`. Aggregate each side first (CTE) and then join → [[SQL-I01 Subqueries and CTEs]].
3. **Ambiguous columns:** `id` exists in many tables → always prefix: `o.id`.
4. **Joining on the wrong key** (e.g., `p.id = o.id`) silently produces nonsense. Check row counts!

`USING (customer_id)` is a shortcut when both columns share the same name. Avoid `NATURAL JOIN` (fragile).

---

## 🧪 Practice
1. List orders with the customer's full name and the employee's name (include orders without employee).
2. Products that have never been sold.
3. Revenue per country (paid + shipped orders).
4. Each employee and the number of orders they handled (including 0).
5. Pairs of customers who live in the same country (self join, no duplicates).

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| join | /dʒɔɪn/ | yoin | unir / unión |
| primary key | /ˈpraɪ.mer.i kiː/ | **PRÁI**-me-ri ki | clave primaria |
| foreign key | /ˈfɔːr.ən kiː/ | **FO**-ren ki | clave foránea |
| match | /mætʃ/ | match | coincidir |
| outer | /ˈaʊ.t̬ɚ/ | **ÁU**-ter | externo |
| inner | /ˈɪn.ɚ/ | **I**-ner | interno |
| Cartesian product | /kɑːrˈtiː.ʒən ˈprɑː.dʌkt/ | kar-**TI**-yan **PRA**-dakt | producto cartesiano |
| ambiguous | /æmˈbɪɡ.ju.əs/ | am-**BI**-guiu-os | ambiguo |
| fan-out | /ˈfæn.aʊt/ | **FAN**-aut | multiplicación de filas |
| skeleton | /ˈskel.ə.tən/ | **SKE**-le-ton | esqueleto (estructura base) |

### 💬 Useful sentences
- *"We **joined** orders **to** customers **on** customer ID."*
- *"A LEFT JOIN **keeps** all rows from the left table, **even if** there is no match."*

---
⬅️ [[SQL-B02 Aggregations - GROUP BY and HAVING]] · ➡️ [[SQL-B04 Creating Tables - DDL, DML and Constraints]] · 🏠 [[00 - SQL Roadmap]]

#sql #beginner
