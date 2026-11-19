---
title: PostgreSQL Practical Guide
tags: [databases, postgresql, psql, administration, backup]
---

# 🐘 PostgreSQL Practical Guide — Beginner to Advanced

PostgreSQL ("**POST**-gres-kiu-el", or just "Postgres") is the most popular open-source relational database: ACID, extensible, standards-compliant, with JSON, full-text search, geospatial (PostGIS) and vector search (pgvector).

---

## 🟢 Beginner — install, connect, explore
```bash
docker run -d --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -v pgdata:/var/lib/postgresql/data postgres:17
docker exec -it pg psql -U postgres
```
### psql cheat sheet
| Command | Does |
|---|---|
| `\l` | list databases |
| `\c datashop` | connect to a database |
| `\dt` / `\dt+` | list tables (with sizes) |
| `\d orders` | describe a table (columns, indexes, FKs) |
| `\dn` / `\du` | schemas / roles |
| `\x` | expanded display (one column per line) |
| `\timing` | show query time |
| `\i file.sql` | run a SQL file |
| `\copy t FROM 'f.csv' CSV HEADER` | import a CSV from your machine |
| `\q` | quit |

```sql
CREATE DATABASE datashop;
\c datashop
\i datashop.sql          -- the script from [[SQL Practice Database]]
```

### Object hierarchy
`cluster (server) → databases → schemas (public, analytics, …) → tables, views, functions`
```sql
CREATE SCHEMA analytics;
SET search_path TO analytics, public;
```

## 🟡 Intermediate — everyday administration
### Roles and access
```sql
CREATE ROLE app_rw LOGIN PASSWORD '...';
GRANT CONNECT ON DATABASE datashop TO app_rw;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_rw;
```
### Backups and restore
```bash
pg_dump -U postgres -Fc datashop > datashop_2026-11-18.dump      # logical backup (custom format)
pg_restore -U postgres -d datashop_copy datashop_2026-11-18.dump
pg_dump -U postgres --schema-only datashop > schema.sql
pg_dumpall --globals-only > roles.sql
```
Physical backups + point-in-time recovery (PITR) for production: `pg_basebackup` + WAL archiving, or tools like **pgBackRest**. Theory: [[07 - Backup and Disaster Recovery]].

### Useful extensions
```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;   -- slowest queries
CREATE EXTENSION IF NOT EXISTS pg_trgm;              -- fuzzy text search / LIKE '%x%' indexes
CREATE EXTENSION IF NOT EXISTS pgcrypto;             -- gen_random_uuid(), hashing
CREATE EXTENSION IF NOT EXISTS postgis;              -- geospatial
CREATE EXTENSION IF NOT EXISTS vector;               -- pgvector: embeddings for AI/RAG
```

### Monitoring queries
```sql
-- What is running now?
SELECT pid, now() - query_start AS duration, state, query
FROM pg_stat_activity WHERE state <> 'idle' ORDER BY duration DESC;

-- Top queries by total time
SELECT calls, round(total_exec_time) AS ms, round(mean_exec_time, 1) AS avg_ms, query
FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 10;

-- Table and index sizes
SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) AS size
FROM pg_catalog.pg_statio_user_tables ORDER BY pg_total_relation_size(relid) DESC;

-- Cancel / terminate
SELECT pg_cancel_backend(12345);  SELECT pg_terminate_backend(12345);
```

## 🔴 Advanced — internals and tuning
| Concept | Key idea |
|---|---|
| **MVCC** | updates create new row versions; old ones become *dead tuples* |
| **VACUUM / autovacuum** | reclaims dead tuples, prevents transaction-ID wraparound |
| **WAL** (Write-Ahead Log) | every change is logged first → durability, replication, PITR |
| **Shared buffers** | PostgreSQL's page cache (start at ~25% of RAM) |
| **work_mem** | memory per sort/hash operation (too low → disk spills) |
| **Checkpoints** | flush dirty pages to disk periodically |
| **Connection limits** | each connection = a process → use a pooler (**PgBouncer**) |

```sql
SHOW shared_buffers; SHOW work_mem; SHOW max_connections;
ALTER SYSTEM SET work_mem = '64MB';  SELECT pg_reload_conf();
```
Replication: streaming replicas (read scaling, failover) and logical replication (CDC to other systems) → [[03 - Replication]] · [[06 - High Availability]].
Managed services: AWS RDS/Aurora, Azure Database for PostgreSQL, Google Cloud SQL/AlloyDB, IBM Cloud Databases for PostgreSQL, Supabase, Neon.

### Spanish deep dives (archived)
[[PostgreSQL]] · [[Postgres_Internals]] (in `99 - Archive`)

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| server | /ˈsɝː.vɚ/ | **SER**-ver | servidor |
| role | /roʊl/ | roul | rol |
| dump | /dʌmp/ | damp | volcado (respaldo lógico) |
| restore | /rɪˈstɔːr/ | ri-**STOR** | restaurar |
| tuning | /ˈtuː.nɪŋ/ | **TU**-ning | ajuste fino |
| dead tuple | /ded ˈtuː.pəl/ | ded **TU**-pol | tupla muerta |
| failover | /ˈfeɪlˌoʊ.vɚ/ | **FÉIL**-ou-ver | conmutación por error |
| pooler | /ˈpuː.lɚ/ | **PU**-ler | agrupador de conexiones |
| wraparound | /ˈræp.ə.raʊnd/ | **RAP**-a-raund | desbordamiento circular |

🏠 [[00 - Databases Hub]]
