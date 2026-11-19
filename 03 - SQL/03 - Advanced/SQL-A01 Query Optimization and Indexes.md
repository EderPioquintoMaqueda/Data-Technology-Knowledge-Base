---
title: "SQL-A01 Query Optimization and Indexes"
level: advanced
tags: [sql, advanced, performance, indexes, explain]
prev: "[[SQL-I04 Set Operations, Views and Data Modeling in Practice]]"
next: "[[SQL-A02 Transactions, Isolation and Locking in Practice]]"
---

# SQL-A01 — Query Optimization and Indexes

## 🎯 Learning objectives
- Read execution plans with `EXPLAIN (ANALYZE, BUFFERS)`.
- Design B-tree, composite, partial, expression and covering indexes.
- Write **sargable** predicates.
- Diagnose slow queries systematically.

Theory: [[02 - How a Query is Executed]] · [[03 - Storage Pages and Tables]] · [[04 - Indexes]]

---

## 1. The optimisation workflow
1. **Find** slow queries: `pg_stat_statements`, slow-query log, APM tools.
2. **Explain** them: `EXPLAIN (ANALYZE, BUFFERS) <query>;`
3. **Hypothesise**: missing index? bad estimate? too much data? fan-out?
4. **Change one thing** (index, rewrite, statistics) and **measure again**.

## 2. Reading a plan
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE customer_id = 1;
```
```
Seq Scan on orders  (cost=0.00..1.25 rows=4 width=40) (actual time=0.010..0.014 rows=5 loops=1)
  Filter: (customer_id = 1)
  Rows Removed by Filter: 15
```
| Field | Meaning |
|---|---|
| `cost=start..total` | planner's estimate (arbitrary units) |
| `rows=` (estimate) vs `actual rows=` | **big differences = bad statistics** → `ANALYZE table;` |
| `loops` | how many times the node ran (multiply!) |
| `Buffers: shared hit/read` | pages from cache vs disk |

### Common nodes
| Node | Means | Good when |
|---|---|---|
| **Seq Scan** | reads the whole table | small table or most rows needed |
| **Index Scan** | index → table rows | few rows (high selectivity) |
| **Index Only Scan** | answered from the index alone | covering index + visibility map |
| **Bitmap Heap Scan** | collects matches, then reads pages in order | medium selectivity, OR conditions |
| **Nested Loop** | for each outer row, look up inner | small outer side + indexed inner |
| **Hash Join** | build hash table on smaller side | large, unsorted equi-joins |
| **Merge Join** | both sides sorted | large, pre-sorted inputs |
| **Sort / HashAggregate** | ORDER BY / GROUP BY | watch for "external merge Disk" → raise `work_mem` |

## 3. Index types and when to use them
```sql
-- B-tree (default): =, <, >, BETWEEN, ORDER BY, LIKE 'abc%'
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- Composite: order matters! Leftmost prefix rule
CREATE INDEX idx_orders_cust_date ON orders (customer_id, order_date);
--   ✅ WHERE customer_id = 1
--   ✅ WHERE customer_id = 1 AND order_date >= '2026-01-01'
--   ❌ WHERE order_date >= '2026-01-01'   (cannot use the leading column)

-- Partial: index only the rows you query
CREATE INDEX idx_orders_pending ON orders (order_date) WHERE status = 'pending';

-- Expression: match a function in WHERE
CREATE INDEX idx_customers_email_lower ON customers (LOWER(email));

-- Covering (INCLUDE): enable index-only scans
CREATE INDEX idx_items_product ON order_items (product_id) INCLUDE (quantity, unit_price);

-- Unique
CREATE UNIQUE INDEX ux_customers_email ON customers (email);

-- Build without blocking writes in production
CREATE INDEX CONCURRENTLY idx_orders_status ON orders (status);
```
| Other index types (PostgreSQL) | Use |
|---|---|
| **GIN** | JSONB, arrays, full-text search (`@>`, `?`, `@@`) |
| **GiST / SP-GiST** | geometry, ranges, nearest neighbour |
| **BRIN** | huge, naturally ordered tables (time-series logs) — tiny index |
| **Hash** | equality only |

### Composite column order rule of thumb
1. Columns used with **equality** first, 2. then **range** columns, 3. then columns for **ORDER BY**; most selective first among equals.

### The cost of indexes
Every index slows down `INSERT/UPDATE/DELETE`, uses disk and memory, and needs maintenance. Find unused ones:
```sql
SELECT relname, indexrelname, idx_scan FROM pg_stat_user_indexes ORDER BY idx_scan;
```

## 4. Sargable predicates (Search ARGument ABLE)
| ❌ Non-sargable (index ignored) | ✅ Sargable |
|---|---|
| `WHERE EXTRACT(YEAR FROM order_date) = 2026` | `WHERE order_date >= '2026-01-01' AND order_date < '2027-01-01'` |
| `WHERE LOWER(email) = 'a@b.com'` | expression index or store emails lower-cased |
| `WHERE price * 1.16 > 100` | `WHERE price > 100 / 1.16` |
| `WHERE name LIKE '%book'` | trigram index (`pg_trgm`) or full-text search |
| `WHERE customer_id::TEXT = '5'` | compare with the right type: `= 5` |
| `WHERE a = 1 OR b = 2` (sometimes) | `UNION ALL` of two indexed queries |

## 5. Query rewrites that matter
- Select only needed columns (enables index-only scans, less I/O).
- Filter early; aggregate before joining "many" tables (avoid fan-out).
- `EXISTS` instead of `IN` with large/nullable subqueries; `NOT EXISTS` instead of `NOT IN`.
- **Keyset pagination** instead of large `OFFSET`:
```sql
-- ❌ slow on page 10,000
SELECT * FROM orders ORDER BY id LIMIT 50 OFFSET 500000;
-- ✅ constant time
SELECT * FROM orders WHERE id > :last_seen_id ORDER BY id LIMIT 50;
```
- Batch writes (`INSERT ... VALUES (...), (...)`, `COPY`) instead of row-by-row.
- Avoid functions/correlated subqueries per row on millions of rows.

## 6. Statistics and maintenance (PostgreSQL)
```sql
ANALYZE orders;                        -- refresh planner statistics
VACUUM (ANALYZE) orders;               -- reclaim dead tuples (MVCC) + stats
SELECT relname, n_live_tup, n_dead_tup, last_autovacuum FROM pg_stat_user_tables;
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000;   -- finer histograms
CREATE STATISTICS st_city_country (dependencies) ON city, country FROM customers;  -- correlated columns
```

## 7. Partitioning big tables
```sql
CREATE TABLE events (id BIGINT, ts TIMESTAMPTZ NOT NULL, payload JSONB) PARTITION BY RANGE (ts);
CREATE TABLE events_2026_11 PARTITION OF events FOR VALUES FROM ('2026-11-01') TO ('2026-12-01');
```
The planner skips irrelevant partitions (**partition pruning**). Theory: [[04 - Partitioning]].

---

## 🧪 Practice (generate data first)
```sql
CREATE TABLE big_orders AS
SELECT g AS id, (random()*100000)::INT AS customer_id,
       DATE '2023-01-01' + (random()*1400)::INT AS order_date,
       (ARRAY['paid','shipped','pending','cancelled'])[1 + (random()*3)::INT] AS status,
       round((random()*1000)::NUMERIC, 2) AS total
FROM generate_series(1, 2000000) g;
ANALYZE big_orders;
```
1. Explain `SELECT * FROM big_orders WHERE customer_id = 42;` before and after an index.
2. Make `WHERE EXTRACT(YEAR FROM order_date) = 2025` sargable and compare plans.
3. Create a composite index that makes `WHERE customer_id = 42 ORDER BY order_date DESC LIMIT 5` instant.
4. Compare `OFFSET 1000000` vs keyset pagination.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| execution plan | /ˌek.sɪˈkjuː.ʃən plæn/ | ek-si-**KIU**-shon plan | plan de ejecución |
| estimate | /ˈes.tɪ.mət/ (n.) | **ES**-ti-mit | estimación |
| scan | /skæn/ | skan | recorrido / escaneo |
| selectivity | /səˌlek.ˈtɪv.ə.t̬i/ | se-lek-**TI**-vi-ti | selectividad |
| leading column | /ˈliː.dɪŋ ˈkɑː.ləm/ | **LI**-ding **KA**-lem | columna inicial |
| covering index | /ˈkʌv.ɚ.ɪŋ/ | **KA**-ve-ring | índice cubriente |
| sargable | /ˈsɑːr.ɡə.bəl/ | **SAR**-ga-bol | aprovechable por índice |
| tuple | /ˈtuː.pəl/ | **TU**-pol | tupla (fila) |
| pruning | /ˈpruː.nɪŋ/ | **PRU**-ning | poda |
| throughput | /ˈθruː.pʊt/ | **ZRU**-put | rendimiento |
| latency | /ˈleɪ.tən.si/ | **LÉI**-ten-si | latencia |

### 💬 Useful sentences
- *"The planner **chose** a sequential scan **because** the estimate **was off** by a factor of 100."*
- *"**Adding** a composite index on (customer_id, order_date) **brought** the query **down from** 2 seconds **to** 3 milliseconds."*

---
⬅️ [[SQL-I04 Set Operations, Views and Data Modeling in Practice]] · ➡️ [[SQL-A02 Transactions, Isolation and Locking in Practice]] · 🏠 [[00 - SQL Roadmap]]

#sql #advanced
