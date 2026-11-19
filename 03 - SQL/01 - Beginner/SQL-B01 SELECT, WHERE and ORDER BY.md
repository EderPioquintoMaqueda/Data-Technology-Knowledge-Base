---
title: "SQL-B01 SELECT, WHERE and ORDER BY"
level: beginner
tags: [sql, beginner, select, where]
next: "[[SQL-B02 Aggregations - GROUP BY and HAVING]]"
---

# SQL-B01 — SELECT, WHERE and ORDER BY

## 🎯 Learning objectives
- Read data with `SELECT ... FROM`.
- Filter rows with `WHERE` and its operators.
- Sort with `ORDER BY` and limit results with `LIMIT`.
- Create calculated columns and aliases.

Dataset: [[SQL Practice Database]]

---

## 1. Your first query
```sql
SELECT first_name, last_name, country
FROM customers;
```
- `SELECT` → **which columns**. `FROM` → **which table**.
- `SELECT *` returns every column (fine for exploring, avoid in production code).
- SQL keywords are case-insensitive; by convention we write them in UPPERCASE.
- End statements with `;`.

## 2. Calculated columns and aliases
```sql
SELECT name,
       price,
       price * 1.16           AS price_with_tax,
       ROUND(price * 0.9, 2)  AS sale_price
FROM products;
```
`AS` gives a column a readable name (**alias**). Use double quotes for aliases with spaces: `AS "Sale Price"`.

## 3. Filtering with WHERE
```sql
SELECT name, price
FROM products
WHERE price > 100;
```
| Operator | Meaning | Example |
|---|---|---|
| `=` | equal | `country = 'MX'` |
| `<>` or `!=` | not equal | `status <> 'cancelled'` |
| `>`, `<`, `>=`, `<=` | comparison | `price >= 50` |
| `BETWEEN a AND b` | inclusive range | `price BETWEEN 30 AND 100` |
| `IN (...)` | in a list | `country IN ('MX','CO','PE')` |
| `LIKE` | text pattern (`%` any, `_` one char) | `email LIKE '%@mail.com'` |
| `ILIKE` | case-insensitive LIKE (PostgreSQL, DuckDB) | `name ILIKE '%book%'` |
| `IS NULL` / `IS NOT NULL` | missing value | `email IS NULL` |

Text values go in **single quotes** `'MX'`. Dates too: `'2026-01-01'`.

### Combining conditions
```sql
SELECT id, customer_id, order_date, status
FROM orders
WHERE status IN ('paid', 'shipped')
  AND order_date >= '2026-01-01'
  AND (employee_id = 3 OR employee_id IS NULL);
```
⚠️ `AND` is evaluated before `OR` → use **parentheses**.
⚠️ `= NULL` never works; NULL means "unknown" → always `IS NULL`.

## 4. Sorting and limiting
```sql
SELECT name, price
FROM products
ORDER BY price DESC, name ASC     -- DESC = descending, ASC = ascending (default)
LIMIT 5;                          -- SQL Server: SELECT TOP 5 ... ; Oracle/standard: FETCH FIRST 5 ROWS ONLY
```
Pagination: `LIMIT 10 OFFSET 20` → rows 21–30.

## 5. Removing duplicates
```sql
SELECT DISTINCT country FROM customers ORDER BY country;
SELECT DISTINCT country, city FROM customers;   -- unique combinations
```

## 6. Comments
```sql
-- single-line comment
/* multi-line
   comment */
```

---

## 🧪 Practice (answers in [[SQL Exercises - 60 Problems with Solutions]])
1. List all products cheaper than $50, ordered by price.
2. Customers from Mexico or Colombia who signed up in 2025.
3. Customers without an email.
4. The 3 most expensive products.
5. Products whose name contains "Monitor" (any case).
6. Every distinct order status.

<details><summary>Solutions 1–3</summary>

```sql
SELECT name, price FROM products WHERE price < 50 ORDER BY price;
SELECT * FROM customers WHERE country IN ('MX','CO') AND signup_date BETWEEN '2025-01-01' AND '2025-12-31';
SELECT first_name, last_name FROM customers WHERE email IS NULL;
```
</details>

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| query | /ˈkwɪr.i/ | **KUI**-ri | consulta |
| table | /ˈteɪ.bəl/ | **TÉI**-bol | tabla |
| row / record | /roʊ/ /ˈrek.ɚd/ | rou / **RE**-kerd | fila / registro |
| column / field | /ˈkɑː.ləm/ | **KA**-lem | columna / campo |
| clause | /klɑːz/ | kloz | cláusula |
| alias | /ˈeɪ.li.əs/ | **ÉI**-li-as | alias |
| ascending | /əˈsen.dɪŋ/ | a-**SEN**-ding | ascendente |
| descending | /dɪˈsen.dɪŋ/ | di-**SEN**-ding | descendente |
| wildcard | /ˈwaɪld.kɑːrd/ | **UÁILD**-kard | comodín |
| quote | /kwoʊt/ | kuout | comilla |
| retrieve | /rɪˈtriːv/ | ri-**TRIV** | recuperar / obtener |

### 💬 Useful sentences
- *"This query **returns** all customers **whose** country **is** Mexico."*
- *"**Filter out** cancelled orders **before** aggregating."* — Excluye los pedidos cancelados antes de agregar.

---
➡️ [[SQL-B02 Aggregations - GROUP BY and HAVING]] · 🏠 [[00 - SQL Roadmap]]

#sql #beginner
