---
title: "SQL-A02 Transactions, Isolation and Locking in Practice"
level: advanced
tags: [sql, advanced, transactions, acid, isolation, locks]
prev: "[[SQL-A01 Query Optimization and Indexes]]"
next: "[[SQL-A03 Advanced Analytics Patterns]]"
---

# SQL-A02 — Transactions, Isolation and Locking in Practice

## 🎯 Learning objectives
- Use `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`.
- Recognise concurrency anomalies and pick an isolation level.
- Use row locks (`SELECT ... FOR UPDATE`, `SKIP LOCKED`) and avoid deadlocks.

Theory: [[05 - Transactions and ACID]] · [[06 - Concurrency and Locks]]

---

## 1. ACID in one table
| Letter | Property | Meaning |
|---|---|---|
| **A** | Atomicity | all statements succeed or none do |
| **C** | Consistency | constraints always hold |
| **I** | Isolation | concurrent transactions don't see each other's partial work |
| **D** | Durability | once committed, it survives a crash (WAL) |

## 2. Transaction control
```sql
BEGIN;
UPDATE products SET stock = stock - 1 WHERE id = 7 AND stock > 0;
INSERT INTO orders (id, customer_id, employee_id, order_date, status) VALUES (2001, 4, 3, CURRENT_DATE, 'paid');
INSERT INTO order_items VALUES (2001, 7, 1, 399.00, 0);
COMMIT;            -- or ROLLBACK; to undo everything
```
```sql
BEGIN;
INSERT INTO categories VALUES (6, 'Gaming');
SAVEPOINT before_products;
INSERT INTO products VALUES (99, 'Bad product', 6, -5, 0);   -- fails CHECK
ROLLBACK TO SAVEPOINT before_products;                        -- keep the category
COMMIT;
```
⚠️ In PostgreSQL, after an error inside a transaction every statement fails until you `ROLLBACK` (or roll back to a savepoint).

## 3. Concurrency anomalies
| Anomaly | What happens |
|---|---|
| **Dirty read** | read uncommitted changes of another transaction |
| **Non-repeatable read** | same row read twice returns different values |
| **Phantom read** | same query returns new rows the second time |
| **Lost update** | two transactions read-modify-write; one overwrites the other |
| **Write skew** | two transactions check a condition, then both write — invariant broken |

## 4. Isolation levels
| Level | Dirty | Non-repeatable | Phantom | Notes |
|---|---|---|---|---|
| Read Uncommitted | (PG treats as RC) | ✔ possible | ✔ | — |
| **Read Committed** (PG default) | ✖ | ✔ possible | ✔ possible | each statement sees a fresh snapshot |
| Repeatable Read | ✖ | ✖ | ✖ in PG | one snapshot per transaction; may raise serialization errors |
| **Serializable** | ✖ | ✖ | ✖ | as if run one by one; app must **retry** on error 40001 |

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- ...
COMMIT;
```
PostgreSQL uses **MVCC**: readers never block writers and writers never block readers.

## 5. Preventing lost updates
```sql
-- ❌ read in the app, compute, write back → race condition
-- ✅ 1. atomic update
UPDATE products SET stock = stock - 1 WHERE id = 7 AND stock >= 1;   -- check rows affected = 1

-- ✅ 2. pessimistic lock
BEGIN;
SELECT stock FROM products WHERE id = 7 FOR UPDATE;   -- other writers wait
UPDATE products SET stock = stock - 1 WHERE id = 7;
COMMIT;

-- ✅ 3. optimistic locking with a version column
UPDATE products SET price = 24.90, version = version + 1
WHERE id = 4 AND version = 3;        -- 0 rows updated → someone changed it, reload & retry
```

## 6. Job queues with SKIP LOCKED
```sql
BEGIN;
SELECT id FROM jobs
WHERE status = 'queued'
ORDER BY created_at
LIMIT 10
FOR UPDATE SKIP LOCKED;     -- many workers can pull different jobs without waiting
-- process, then:
UPDATE jobs SET status = 'done' WHERE id = ANY(:ids);
COMMIT;
```

## 7. Deadlocks
T1 locks row A then wants B; T2 locks B then wants A → PostgreSQL detects it and aborts one.
Avoid them by:
- Locking rows in a **consistent order** (e.g., by id).
- Keeping transactions **short** (no user interaction or API calls inside).
- Using `lock_timeout` / `statement_timeout`.
```sql
SET lock_timeout = '2s';
SELECT pid, wait_event_type, state, query FROM pg_stat_activity WHERE state <> 'idle';
SELECT * FROM pg_locks WHERE NOT granted;
```

## 8. Lab: see isolation in action
Open **two psql sessions**:
| Time | Session A | Session B |
|---|---|---|
| 1 | `BEGIN;` `SELECT price FROM products WHERE id=4;` → 25.50 | |
| 2 | | `UPDATE products SET price=30 WHERE id=4;` (autocommit) |
| 3 | `SELECT price FROM products WHERE id=4;` → **30** (Read Committed) | |
Repeat with `BEGIN ISOLATION LEVEL REPEATABLE READ;` → step 3 returns **25.50**.

Python side: [[PY-A05 Python and Databases]] (`conn.transaction()`), and retries with [[PY-A01 Decorators, Closures and Context Managers]].

---

## 🧪 Practice
1. Write a transfer between two "accounts" that never leaves money half-moved.
2. Reproduce a lost update with two sessions, then fix it with `FOR UPDATE`.
3. Build a jobs table and a worker query with `SKIP LOCKED`.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| transaction | /trænˈzæk.ʃən/ | tran-**ZAK**-shon | transacción |
| atomicity | /ˌæt̬.əˈmɪs.ə.t̬i/ | a-to-**MI**-si-ti | atomicidad |
| durability | /ˌdʊr.əˈbɪl.ə.t̬i/ | du-ra-**BI**-li-ti | durabilidad |
| isolation | /ˌaɪ.səlˈeɪ.ʃən/ | ai-so-**LÉI**-shon | aislamiento |
| lock | /lɑːk/ | lak | bloqueo |
| deadlock | /ˈded.lɑːk/ | **DED**-lak | interbloqueo |
| snapshot | /ˈsnæp.ʃɑːt/ | **SNAP**-shat | instantánea |
| savepoint | /ˈseɪv.pɔɪnt/ | **SÉIV**-point | punto de guardado |
| lost update | /lɑːst ˈʌp.deɪt/ | last **AP**-deit | actualización perdida |
| retry | /ˌriːˈtraɪ/ | ri-**TRÁI** | reintentar |
| invariant | /ɪnˈver.i.ənt/ | in-**VE**-ri-ant | invariante |

### 💬 Useful sentences
- *"**Wrap** both updates **in a single transaction** so they **succeed or fail together**."*
- *"Serializable transactions **may fail** with a serialization error, so the application **must retry** them."*

---
⬅️ [[SQL-A01 Query Optimization and Indexes]] · ➡️ [[SQL-A03 Advanced Analytics Patterns]] · 🏠 [[00 - SQL Roadmap]]

#sql #advanced
