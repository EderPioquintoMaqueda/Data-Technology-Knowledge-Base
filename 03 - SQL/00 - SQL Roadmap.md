---
title: SQL Roadmap
type: moc
tags: [sql, roadmap, moc]
---

# 🐘 SQL Roadmap — From Beginner to Advanced

> **SQL** (*Structured Query Language*, pronounced **"ES-kiu-EL"** or **"SI-kuel"**) is the language of relational databases. It is **declarative**: you describe *what* you want, and the database decides *how* to get it.

## ⚙️ Setup (choose one)
| Option | Best for | How |
|---|---|---|
| **DuckDB** | fastest start, no server | `pip install duckdb` → `duckdb practice.duckdb` |
| **PostgreSQL** | the professional standard | Docker: `docker run --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -d postgres:17` |
| **SQLite** | already in Python | `python -m sqlite3 practice.db` (3.12+) or DB Browser for SQLite |
| GUI client | writing queries comfortably | **DBeaver** (free) or **pgAdmin** |

Then create the training database: **[[SQL Practice Database]]** (all examples in this track use it).

## 🟢 Level 1 — Beginner
| # | Note | Key clauses |
|---|---|---|
| 1 | [[SQL-B01 SELECT, WHERE and ORDER BY]] | `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`, `DISTINCT` |
| 2 | [[SQL-B02 Aggregations - GROUP BY and HAVING]] | `COUNT`, `SUM`, `AVG`, `GROUP BY`, `HAVING` |
| 3 | [[SQL-B03 JOINs]] | `INNER`, `LEFT`, `RIGHT`, `FULL`, `CROSS`, self join |
| 4 | [[SQL-B04 Creating Tables - DDL, DML and Constraints]] | `CREATE`, `INSERT`, `UPDATE`, `DELETE`, keys |

## 🟡 Level 2 — Intermediate
| # | Note | Key clauses |
|---|---|---|
| 5 | [[SQL-I01 Subqueries and CTEs]] | `IN`, `EXISTS`, scalar subqueries, `WITH` |
| 6 | [[SQL-I02 CASE, NULLs, Strings and Dates]] | `CASE`, `COALESCE`, `NULLIF`, date/string functions |
| 7 | [[SQL-I03 Window Functions]] | `OVER`, `PARTITION BY`, `ROW_NUMBER`, `LAG`, running totals |
| 8 | [[SQL-I04 Set Operations, Views and Data Modeling in Practice]] | `UNION`, `INTERSECT`, `EXCEPT`, views |

## 🔴 Level 3 — Advanced
| # | Note | Key topics |
|---|---|---|
| 9 | [[SQL-A01 Query Optimization and Indexes]] | `EXPLAIN ANALYZE`, index design, sargability |
| 10 | [[SQL-A02 Transactions, Isolation and Locking in Practice]] | `BEGIN/COMMIT`, isolation levels, deadlocks |
| 11 | [[SQL-A03 Advanced Analytics Patterns]] | recursive CTEs, gaps & islands, cohorts, funnels, JSON |
| 12 | [[SQL-A04 Functions, Procedures and Triggers]] | PL/pgSQL, triggers, materialized views, security |

## 🏋️ Practice
- [[SQL Exercises - 60 Problems with Solutions]] — beginner → advanced, interview-style.

## 🧠 Logical order of execution (memorise this!)
```
FROM / JOIN  →  WHERE  →  GROUP BY  →  HAVING  →  SELECT  →  DISTINCT  →  ORDER BY  →  LIMIT
```
That is why you **cannot** use a `SELECT` alias inside `WHERE`, but you **can** use it in `ORDER BY`.

## 🔗 Theory behind SQL (Databases track)
- [[01 - What is a Database System]] · [[03 - Keys and Constraints]] · [[05 - Normalization]]
- [[02 - How a Query is Executed]] · [[04 - Indexes]] · [[05 - Transactions and ACID]] · [[06 - Concurrency and Locks]]
- Spark SQL at scale: [[12 - Spark SQL]]
- Python ↔ SQL: [[PY-A05 Python and Databases]] · [[DuckDB - SQL Inside Python]]

## 🗣️ Reading SQL out loud (English practice)
| SQL | Say it like this |
|---|---|
| `SELECT name FROM customers WHERE country = 'MX';` | "Select name from customers where country equals M-X." |
| `COUNT(*)` | "count star" |
| `<>` / `!=` | "not equal to" |
| `>=` | "greater than or equal to" |
| `GROUP BY` | "group by" |
| `ORDER BY total DESC` | "order by total descending" |

#sql #roadmap #moc
