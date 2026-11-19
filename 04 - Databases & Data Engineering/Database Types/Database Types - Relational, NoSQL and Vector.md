---
title: "Database Types - Relational, NoSQL and Vector"
tags: [databases, nosql, mongodb, redis, cassandra, neo4j, vector-database]
---

# 🧩 Database Types — Relational, NoSQL and Vector

> There is no "best database" — there is the right database **for an access pattern**. Start with PostgreSQL; add a specialised store only when a real requirement appears.

## 1. Overview
| Family | Data model | Examples | Strengths | Typical use |
|---|---|---|---|---|
| **Relational (RDBMS)** | tables, rows, SQL, joins | PostgreSQL, MySQL, SQL Server, Oracle, Db2 | ACID, integrity, flexible queries | transactions, ERP, most apps |
| **Analytical / columnar** | columns, SQL | DuckDB, ClickHouse, Snowflake, BigQuery, Redshift | fast aggregations on billions of rows | BI, reporting |
| **Document** | JSON documents | MongoDB, Couchbase, Firestore | flexible schema, nested data | catalogues, content, user profiles |
| **Key-value** | key → value | Redis, DynamoDB, Valkey | microsecond reads, caching | sessions, caches, rate limiting, queues |
| **Wide-column** | partitions of rows with many columns | Cassandra, ScyllaDB, HBase | massive writes, linear scaling | IoT, time-stamped events |
| **Time-series** | timestamped points | TimescaleDB, InfluxDB | retention, downsampling | sensors, metrics, monitoring |
| **Graph** | nodes + relationships | Neo4j, Amazon Neptune | traversing relationships | fraud rings, recommendations, networks |
| **Search** | inverted index | Elasticsearch, OpenSearch | full-text, relevance ranking | search boxes, log analytics |
| **Vector** | embeddings (arrays of floats) | pgvector, Pinecone, Weaviate, Milvus, Qdrant | similarity search | RAG, semantic search, LLM apps |

## 2. 🟢 Beginner — the same data in three models
**Relational**
```sql
SELECT c.first_name, o.id FROM customers c JOIN orders o ON o.customer_id = c.id WHERE c.id = 1;
```
**Document (MongoDB)** — data that is read together is stored together:
```javascript
db.customers.insertOne({
  _id: 1, name: "Ana López", country: "MX",
  orders: [{ id: 1001, date: ISODate("2025-02-01"), items: [{ sku: "TP-X1", qty: 1 }] }]
});
db.customers.find({ country: "MX" }, { name: 1, "orders.id": 1 });
db.orders.aggregate([{ $match: { status: "paid" } }, { $group: { _id: "$country", total: { $sum: "$amount" } } }]);
```
**Key-value (Redis)**
```bash
SET session:abc123 '{"user_id": 1}' EX 3600     # expires in 1 hour
GET session:abc123
INCR page_views:2026-11-18
```

## 3. 🟡 Intermediate — how to choose
Ask these questions:
1. **Access pattern:** do I know my queries in advance (key lookups) or need ad-hoc analysis (SQL)?
2. **Consistency:** must every read see the latest write (bank) or is "eventually" OK (likes counter)?
3. **Scale:** GBs (one server is fine) or TBs/PBs with huge write rates?
4. **Relationships:** many joins → relational / graph; self-contained aggregates → document.
5. **Team skills & operations:** who will run backups, upgrades, monitoring?

### CAP theorem (distributed systems)
During a network **P**artition, a system must choose between **C**onsistency and **A**vailability.
- CP: MongoDB (default), HBase, etcd · AP: Cassandra, DynamoDB (tunable).
- **PACELC** adds: Else (no partition), trade **L**atency vs **C**onsistency.

### ACID vs BASE
| ACID (relational) | BASE (many NoSQL) |
|---|---|
| Atomic, Consistent, Isolated, Durable | Basically Available, Soft state, Eventually consistent |

## 4. 🔴 Advanced — polyglot persistence & modern trends
- **Polyglot persistence:** PostgreSQL as system of record + Redis cache + Elasticsearch for search + a warehouse for analytics, synchronised with **CDC** (Debezium/Kafka).
- **PostgreSQL as a multi-model DB:** JSONB (documents), pgvector (vectors), TimescaleDB (time-series), PostGIS (geo) → often enough, fewer systems to operate.
- **NewSQL / distributed SQL:** CockroachDB, YugabyteDB, Google Spanner — SQL + horizontal scaling.
- **Lakehouse:** open table formats (Delta Lake, Apache Iceberg) on object storage → [[07 - Lakehouse Architecture]].

### Vector search in PostgreSQL (for RAG)
```sql
CREATE EXTENSION vector;
CREATE TABLE docs (id BIGSERIAL PRIMARY KEY, content TEXT, embedding VECTOR(1536));
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
SELECT id, content FROM docs ORDER BY embedding <=> :query_embedding LIMIT 5;   -- cosine distance
```
Related: [[Prompt Engineering y RAG]] · [[NLP y LLMs]].

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| relational | /rɪˈleɪ.ʃən.əl/ | ri-**LÉI**-sho-nal | relacional |
| document | /ˈdɑː.kjə.mənt/ | **DA**-kiu-ment | documento |
| key-value | /kiː ˈvæl.juː/ | ki **VAL**-yu | clave-valor |
| graph | /ɡræf/ | graf | grafo |
| embedding | /ɪmˈbed.ɪŋ/ | im-**BE**-ding | vector de representación |
| eventual consistency | /ɪˈven.tʃu.əl kənˈsɪs.tən.si/ | i-**VEN**-chu-al kon-**SIS**-ten-si | consistencia eventual |
| availability | /əˌveɪ.ləˈbɪl.ə.t̬i/ | a-vei-la-**BI**-li-ti | disponibilidad |
| access pattern | /ˈæk.ses ˈpæt̬.ɚn/ | **AK**-ses **PA**-tern | patrón de acceso |
| caching | /ˈkæʃ.ɪŋ/ | **KA**-shing | almacenamiento en caché |

🏠 [[00 - Databases Hub]]
