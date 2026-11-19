---
title: "SQL-B04 Creating Tables - DDL, DML and Constraints"
level: beginner
tags: [sql, beginner, ddl, dml, constraints]
prev: "[[SQL-B03 JOINs]]"
next: "[[SQL-I01 Subqueries and CTEs]]"
---

# SQL-B04 — Creating Tables: DDL, DML and Constraints

## 🎯 Learning objectives
- Create, alter and drop tables (DDL).
- Insert, update and delete rows (DML) safely.
- Choose data types and protect data quality with constraints.

| Family | Statements | Purpose |
|---|---|---|
| **DDL** — Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | structure |
| **DML** — Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE`, `MERGE` | data |
| **DQL** — Data Query Language | `SELECT` | reading |
| **DCL** — Data Control Language | `GRANT`, `REVOKE` | permissions |
| **TCL** — Transaction Control | `BEGIN`, `COMMIT`, `ROLLBACK` | transactions |

---

## 1. Common data types (PostgreSQL)
| Type | Use | Example |
|---|---|---|
| `INTEGER` / `BIGINT` | whole numbers / big ids | `42` |
| `NUMERIC(10,2)` / `DECIMAL` | **money** (exact) | `1299.00` |
| `REAL` / `DOUBLE PRECISION` | measurements (approximate) | `3.14159` |
| `VARCHAR(n)` / `TEXT` | text | `'Ana'` |
| `CHAR(n)` | fixed-length codes | `'MX'` |
| `BOOLEAN` | true/false | `TRUE` |
| `DATE` / `TIMESTAMP` / `TIMESTAMPTZ` | dates / with time / with time zone | `'2026-11-18'` |
| `UUID` | global identifiers | `gen_random_uuid()` |
| `JSONB` | semi-structured data | `'{"tier":"gold"}'` |

## 2. CREATE TABLE with constraints
```sql
CREATE TABLE suppliers (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,   -- auto-increment
    name        VARCHAR(100) NOT NULL,
    tax_id      VARCHAR(20)  NOT NULL UNIQUE,
    country     CHAR(2)      NOT NULL DEFAULT 'MX',
    rating      SMALLINT     CHECK (rating BETWEEN 1 AND 5),
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE TABLE supplier_products (
    supplier_id INTEGER NOT NULL REFERENCES suppliers(id) ON DELETE CASCADE,
    product_id  INTEGER NOT NULL REFERENCES products(id),
    cost        NUMERIC(10,2) NOT NULL CHECK (cost > 0),
    PRIMARY KEY (supplier_id, product_id)          -- composite key
);
```
| Constraint | Guarantees |
|---|---|
| `PRIMARY KEY` | unique + not null identifier |
| `FOREIGN KEY ... REFERENCES` | the referenced row exists (referential integrity) |
| `UNIQUE` | no duplicates |
| `NOT NULL` | value required |
| `CHECK` | custom rule |
| `DEFAULT` | value when none is given |

`ON DELETE CASCADE` deletes child rows automatically; `ON DELETE SET NULL` / `RESTRICT` are alternatives. Theory: [[03 - Keys and Constraints]].

## 3. INSERT
```sql
INSERT INTO suppliers (name, tax_id, rating) VALUES ('TechParts SA', 'TPA010101AAA', 4);

INSERT INTO suppliers (name, tax_id, country, rating) VALUES
  ('Andes Books', 'CO900123', 'CO', 5),
  ('Lima Monitors', 'PE2045', 'PE', 3)
RETURNING id, name;                       -- PostgreSQL/DuckDB: get generated ids back

-- Insert from a query
INSERT INTO archived_orders SELECT * FROM orders WHERE order_date < '2025-01-01';
```

## 4. UPDATE and DELETE — always with WHERE!
```sql
UPDATE products SET price = price * 1.05 WHERE category_id = 4;
UPDATE products SET stock = stock - 1, price = 24.99 WHERE id = 4;

DELETE FROM orders WHERE status = 'cancelled' AND order_date < '2025-06-01';
TRUNCATE TABLE staging_orders;            -- delete ALL rows, fast, no WHERE
```
🛡️ **Safety habit:** run the `SELECT` with the same `WHERE` first, check the rows, then run `UPDATE/DELETE` inside a transaction:
```sql
BEGIN;
UPDATE products SET price = price * 1.10 WHERE category_id = 1;
SELECT id, name, price FROM products WHERE category_id = 1;   -- verify
COMMIT;      -- or ROLLBACK;
```

## 5. UPSERT — insert or update
```sql
INSERT INTO products (id, name, category_id, price, stock)
VALUES (4, 'Wireless Mouse', 2, 23.90, 320)
ON CONFLICT (id) DO UPDATE
SET price = EXCLUDED.price, stock = EXCLUDED.stock;
```
Standard SQL / SQL Server / PostgreSQL 15+: `MERGE INTO ... USING ... WHEN MATCHED THEN UPDATE WHEN NOT MATCHED THEN INSERT`.

## 6. ALTER and DROP
```sql
ALTER TABLE customers ADD COLUMN phone VARCHAR(20);
ALTER TABLE customers ALTER COLUMN phone TYPE VARCHAR(30);
ALTER TABLE customers RENAME COLUMN phone TO mobile;
ALTER TABLE customers ADD CONSTRAINT chk_country CHECK (country ~ '^[A-Z]{2}$');
ALTER TABLE customers DROP COLUMN mobile;
DROP TABLE IF EXISTS supplier_products;
```

## 7. Naming conventions
- `snake_case`, plural table names (`orders`) or singular — **be consistent**.
- Primary key `id`; foreign key `<table>_id` (`customer_id`).
- Dates end with `_date`/`_at`; booleans start with `is_`/`has_`.

---

## 🧪 Practice
1. Create a `reviews` table (id, product_id FK, customer_id FK, rating 1–5, comment, created_at default now).
2. Insert 5 reviews, one of them violating the CHECK — read the error message.
3. Increase the price of all books by 8% inside a transaction and verify before committing.
4. Upsert two products (one existing, one new).

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| create | /kriˈeɪt/ | kri-**ÉIT** | crear |
| insert | /ɪnˈsɝːt/ | in-**SERT** | insertar |
| update | /ʌpˈdeɪt/ | ap-**DÉIT** | actualizar |
| delete | /dɪˈliːt/ | di-**LIT** | eliminar |
| drop | /drɑːp/ | drap | eliminar (objeto) |
| alter | /ˈɑːl.t̬ɚ/ | **OL**-ter | modificar |
| constraint | /kənˈstreɪnt/ | kon-**STRÉINT** | restricción |
| integrity | /ɪnˈteɡ.rə.t̬i/ | in-**TE**-gri-ti | integridad |
| cascade | /kæˈskeɪd/ | kas-**KÉID** | cascada |
| auto-increment | /ˌɑː.t̬oʊ ˈɪŋ.krə.mənt/ | o-tou **IN**-kre-ment | autoincremento |
| enforce | /ɪnˈfɔːrs/ | in-**FORS** | hacer cumplir |

### 💬 Useful sentences
- *"The foreign key **prevents** orders **from referencing** customers that **don't exist**."*
- *"**Never run** an UPDATE **without** a WHERE clause."*

---
⬅️ [[SQL-B03 JOINs]] · ➡️ [[SQL-I01 Subqueries and CTEs]] · 🏠 [[00 - SQL Roadmap]]

#sql #beginner
