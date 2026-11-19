---
title: "SQL-A04 Functions, Procedures and Triggers"
level: advanced
tags: [sql, advanced, plpgsql, triggers, security, roles]
prev: "[[SQL-A03 Advanced Analytics Patterns]]"
next: "[[SQL Exercises - 60 Problems with Solutions]]"
---

# SQL-A04 — Functions, Procedures, Triggers and Security (PostgreSQL)

## 🎯 Learning objectives
- Write SQL and PL/pgSQL functions and procedures.
- Automate rules with triggers (audit log, `updated_at`).
- Manage roles, privileges and row-level security.
- Know when **not** to put logic in the database.

---

## 1. SQL functions
```sql
CREATE OR REPLACE FUNCTION line_total(qty INT, price NUMERIC, disc NUMERIC)
RETURNS NUMERIC
LANGUAGE sql IMMUTABLE
AS $$
    SELECT ROUND(qty * price * (1 - disc), 2);
$$;

SELECT order_id, line_total(quantity, unit_price, discount) FROM order_items;
```
Volatility: `IMMUTABLE` (same input → same output, indexable) · `STABLE` (same within a query) · `VOLATILE` (default, e.g. `random()`).

### Table-returning function
```sql
CREATE OR REPLACE FUNCTION customer_orders(p_customer_id INT)
RETURNS TABLE (order_id INT, order_date DATE, total NUMERIC)
LANGUAGE sql STABLE
AS $$
    SELECT o.id, o.order_date, SUM(line_total(oi.quantity, oi.unit_price, oi.discount))
    FROM orders o JOIN order_items oi ON oi.order_id = o.id
    WHERE o.customer_id = p_customer_id
    GROUP BY o.id, o.order_date
    ORDER BY o.order_date;
$$;

SELECT * FROM customer_orders(1);
```

## 2. PL/pgSQL — procedural logic
```sql
CREATE OR REPLACE FUNCTION customer_tier(p_customer_id INT)
RETURNS TEXT
LANGUAGE plpgsql STABLE
AS $$
DECLARE
    v_total NUMERIC;
BEGIN
    SELECT COALESCE(SUM(total), 0) INTO v_total FROM customer_orders(p_customer_id);
    IF v_total >= 3000 THEN
        RETURN 'gold';
    ELSIF v_total >= 1000 THEN
        RETURN 'silver';
    ELSE
        RETURN 'bronze';
    END IF;
END;
$$;
```

## 3. Procedures (can COMMIT inside) — PostgreSQL 11+
```sql
CREATE OR REPLACE PROCEDURE archive_old_orders(p_before DATE)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO orders_archive SELECT * FROM orders WHERE order_date < p_before;
    DELETE FROM orders WHERE order_date < p_before;
    RAISE NOTICE 'Archived orders before %', p_before;
END;
$$;

CALL archive_old_orders('2025-01-01');
```

## 4. Triggers
### Keep `updated_at` current
```sql
ALTER TABLE products ADD COLUMN updated_at TIMESTAMPTZ DEFAULT now();

CREATE OR REPLACE FUNCTION set_updated_at() RETURNS TRIGGER
LANGUAGE plpgsql AS $$
BEGIN
    NEW.updated_at := now();
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_products_updated
BEFORE UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```
### Audit log of price changes
```sql
CREATE TABLE price_audit (
    product_id INT, old_price NUMERIC, new_price NUMERIC,
    changed_by TEXT DEFAULT current_user, changed_at TIMESTAMPTZ DEFAULT now()
);

CREATE OR REPLACE FUNCTION log_price_change() RETURNS TRIGGER
LANGUAGE plpgsql AS $$
BEGIN
    IF NEW.price IS DISTINCT FROM OLD.price THEN
        INSERT INTO price_audit (product_id, old_price, new_price) VALUES (OLD.id, OLD.price, NEW.price);
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_price_audit
AFTER UPDATE OF price ON products
FOR EACH ROW EXECUTE FUNCTION log_price_change();
```
⚠️ Triggers are "invisible" logic: document them, keep them small, and avoid chains of triggers.

## 5. Roles, privileges and least privilege
```sql
CREATE ROLE analyst NOLOGIN;
GRANT USAGE ON SCHEMA public TO analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO analyst;

CREATE ROLE ana LOGIN PASSWORD 'use-a-secret-manager';
GRANT analyst TO ana;
REVOKE DELETE ON orders FROM ana;
```

### Row-Level Security (RLS)
```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY rep_sees_own_orders ON orders
    FOR SELECT USING (employee_id = current_setting('app.employee_id')::INT);
```

## 6. Database logic vs application logic
| Put it in the database | Keep it in the application |
|---|---|
| Data integrity (constraints, FKs) | Business workflows that change often |
| Audit trails, `updated_at` | Calls to external APIs, emails |
| Set-based heavy data processing | Complex branching / testing-heavy logic |
| Security (roles, RLS) | Anything you need to version and unit-test easily |

Modern analytics stacks put transformation logic in **dbt models** (versioned SQL) → [[dbt - Data Build Tool]].

---

## 🧪 Practice
1. Write a function `order_total(p_order_id)` and use it in a query.
2. Add an audit trigger on `orders.status` changes.
3. Create a read-only role and verify it cannot `DELETE`.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| stored procedure | /stɔːrd prəˈsiː.dʒɚ/ | stord pro-**SI**-yer | procedimiento almacenado |
| trigger | /ˈtrɪɡ.ɚ/ | **TRI**-guer | disparador |
| fire (a trigger) | /faɪr/ | fáir | dispararse |
| privilege | /ˈprɪv.əl.ɪdʒ/ | **PRI**-vi-lich | privilegio |
| grant / revoke | /ɡrænt/ /rɪˈvoʊk/ | grant / ri-**VOUK** | otorgar / revocar |
| least privilege | /liːst ˈprɪv.əl.ɪdʒ/ | list **PRI**-vi-lich | mínimo privilegio |
| audit | /ˈɑː.dɪt/ | **O**-dit | auditoría |
| policy | /ˈpɑː.lə.si/ | **PA**-li-si | política |
| immutable | /ɪˈmjuː.t̬ə.bəl/ | i-**MIU**-ta-bol | inmutable |

### 💬 Useful sentences
- *"The trigger **fires** after every price update and **writes** a row to the audit table."*
- *"Analysts **have** read-only **access** — we **follow the principle of least privilege**."*

---
⬅️ [[SQL-A03 Advanced Analytics Patterns]] · ➡️ [[SQL Exercises - 60 Problems with Solutions]] · 🏠 [[00 - SQL Roadmap]]

#sql #advanced
