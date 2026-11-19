---
title: "PY-A05 Python and Databases"
level: advanced
tags: [python, advanced, databases, sqlalchemy, psycopg, sqlite, duckdb, etl]
prev: "[[PY-A04 Performance, Profiling and Memory]]"
next: "[[Python Libraries Map]]"
---

# PY-A05 — Python and Databases

## 🎯 Learning objectives
- Connect to SQLite, PostgreSQL and DuckDB from Python.
- Use **parameterised queries** (never string formatting) to prevent SQL injection.
- Manage connections, transactions and connection pools.
- Use **SQLAlchemy 2.0** (Core and ORM) and move data between pandas/Polars and SQL efficiently.
- Build a small, idempotent ETL load.

---

## 1. The DB-API pattern (PEP 249)
Every Python driver follows the same steps:
```
connect → cursor → execute(sql, params) → fetch / commit → close
```

### SQLite (built in — perfect for learning)
```python
import sqlite3

with sqlite3.connect("shop.db") as conn:          # commits on success, rolls back on error
    conn.execute("""
        CREATE TABLE IF NOT EXISTS customers (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            country TEXT
        )""")
    conn.execute("INSERT INTO customers (name, country) VALUES (?, ?)", ("Ana", "MX"))
    conn.executemany("INSERT INTO customers (name, country) VALUES (?, ?)",
                     [("Luis", "CO"), ("Eva", "PE")])
    conn.row_factory = sqlite3.Row                  # rows behave like dicts
    for row in conn.execute("SELECT * FROM customers WHERE country = ?", ("MX",)):
        print(row["id"], row["name"])
```

### PostgreSQL with psycopg 3
```python
import os, psycopg
from psycopg.rows import dict_row

with psycopg.connect(os.environ["PG_DSN"], row_factory=dict_row) as conn:
    with conn.cursor() as cur:
        cur.execute(
            "SELECT id, total FROM orders WHERE created_at >= %s AND status = %s",
            ("2026-11-01", "paid"),
        )
        rows = cur.fetchall()          # or fetchone(), fetchmany(1000), or iterate
    # leaving the 'with conn' block commits (or rolls back on exception)
```

## 2. 🚨 SQL injection — the #1 rule
```python
user_input = "x' OR '1'='1"
# ❌ NEVER
cur.execute(f"SELECT * FROM users WHERE name = '{user_input}'")   # returns ALL users
# ✅ ALWAYS use placeholders — the driver escapes values safely
cur.execute("SELECT * FROM users WHERE name = %s", (user_input,))
```
Placeholders: `?` (sqlite3), `%s` (psycopg), `:name` (SQLAlchemy `text()`).
Table/column names cannot be parameters → use `psycopg.sql.Identifier` or a whitelist.

## 3. Transactions
```python
with psycopg.connect(dsn) as conn:
    with conn.transaction():                # atomic block
        conn.execute("UPDATE accounts SET balance = balance - %s WHERE id = %s", (100, 1))
        conn.execute("UPDATE accounts SET balance = balance + %s WHERE id = %s", (100, 2))
    # if any statement fails → both are rolled back (ACID)
```
Theory: [[05 - Transactions and ACID]] · [[SQL-A02 Transactions, Isolation and Locking in Practice]].

## 4. SQLAlchemy 2.0
### Engine and connection pool
```python
from sqlalchemy import create_engine, text
engine = create_engine(
    "postgresql+psycopg://user:pass@localhost:5432/shop",
    pool_size=5, max_overflow=10, pool_pre_ping=True,
)
with engine.connect() as conn:
    result = conn.execute(text("SELECT name FROM customers WHERE country = :c"), {"c": "MX"})
    names = result.scalars().all()

with engine.begin() as conn:                 # begin() = auto-commit transaction
    conn.execute(text("UPDATE products SET price = price * 1.05 WHERE category = :cat"),
                 {"cat": "books"})
```

### ORM (Object Relational Mapper)
```python
from sqlalchemy import String, ForeignKey, select
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session

class Base(DeclarativeBase): pass

class Customer(Base):
    __tablename__ = "customers"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100))
    orders: Mapped[list["Order"]] = relationship(back_populates="customer")

class Order(Base):
    __tablename__ = "orders"
    id: Mapped[int] = mapped_column(primary_key=True)
    total: Mapped[float]
    customer_id: Mapped[int] = mapped_column(ForeignKey("customers.id"))
    customer: Mapped[Customer] = relationship(back_populates="orders")

Base.metadata.create_all(engine)

with Session(engine) as session:
    ana = Customer(name="Ana", orders=[Order(total=120.0), Order(total=80.5)])
    session.add(ana)
    session.commit()

    stmt = select(Customer).where(Customer.name == "Ana")
    customer = session.scalars(stmt).one()
    print([o.total for o in customer.orders])
```
| Use Core / raw SQL when… | Use ORM when… |
|---|---|
| analytics, bulk loads, complex SQL | application CRUD, business objects |
| you think in tables & sets | you think in objects & relationships |

⚠️ **N+1 problem:** looping over customers and lazily loading each `customer.orders` fires one query per customer. Fix with `selectinload(Customer.orders)` or a JOIN.

### Migrations
Use **Alembic** to version schema changes: `alembic revision --autogenerate -m "add email"` → `alembic upgrade head`.

## 5. pandas / Polars ↔ SQL
```python
import pandas as pd
df = pd.read_sql("SELECT * FROM orders WHERE created_at >= '2026-01-01'", engine)
df.to_sql("orders_staging", engine, if_exists="replace", index=False,
          chunksize=10_000, method="multi")

import polars as pl
pl_df = pl.read_database_uri("SELECT * FROM orders", uri="postgresql://user:pass@localhost/shop")
pl_df.write_database("orders_copy", connection="postgresql://...", if_table_exists="replace")
```
🚀 **Bulk load fast with PostgreSQL COPY** (10–100× faster than INSERTs):
```python
with psycopg.connect(dsn) as conn, conn.cursor() as cur:
    with cur.copy("COPY orders_staging (id, total, created_at) FROM STDIN") as copy:
        for row in df.itertuples(index=False):
            copy.write_row(row)
```

## 6. Idempotent upsert (safe to re-run)
```sql
INSERT INTO orders (id, total, status)
SELECT id, total, status FROM orders_staging
ON CONFLICT (id) DO UPDATE
SET total = EXCLUDED.total, status = EXCLUDED.status;
```
Pattern: **load to staging → validate → upsert into target → truncate staging**, all in one transaction.

## 7. DuckDB — analytics without a server
```python
import duckdb
duckdb.sql("SELECT country, SUM(total) FROM 'data/orders/*.parquet' GROUP BY 1").df()
```
Full note: [[DuckDB - SQL Inside Python]].

## 8. Checklist for production DB code
- [ ] Credentials from env vars / secret manager.
- [ ] Parameterised queries only.
- [ ] Connection pooling + `pool_pre_ping`.
- [ ] Explicit transactions; idempotent loads.
- [ ] Bulk operations (`COPY`, `executemany`, chunks) instead of row-by-row.
- [ ] Indexes on filter/join columns → [[04 - Indexes]].
- [ ] Schema migrations with Alembic.

---

## 🧪 Practice
1. Create the SQLite practice DB from [[SQL Practice Database]] using Python.
2. Write `load_orders(df, engine)` that upserts via a staging table.
3. Demonstrate SQL injection with an f-string on a toy table, then fix it.
4. Compare `to_sql` vs `COPY` for 1M rows.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| driver | /ˈdraɪ.vɚ/ | **DRÁI**-ver | controlador |
| connection | /kəˈnek.ʃən/ | ko-**NEK**-shon | conexión |
| cursor | /ˈkɝː.sɚ/ | **KER**-ser | cursor |
| query | /ˈkwɪr.i/ | **KUI**-ri | consulta |
| placeholder | /ˈpleɪsˌhoʊl.dɚ/ | **PLÉIS**-joul-der | marcador de posición |
| injection | /ɪnˈdʒek.ʃən/ | in-**YEK**-shon | inyección |
| commit | /kəˈmɪt/ | ko-**MIT** | confirmar (transacción) |
| rollback | /ˈroʊl.bæk/ | **ROUL**-bak | revertir |
| pool | /puːl/ | pul | grupo / reserva (de conexiones) |
| mapper | /ˈmæp.ɚ/ | **MA**-per | mapeador |
| migration | /maɪˈɡreɪ.ʃən/ | mai-**GRÉI**-shon | migración |
| upsert | /ˈʌp.sɝːt/ | **AP**-sert | insertar o actualizar |
| staging table | /ˈsteɪ.dʒɪŋ ˈteɪ.bəl/ | **STÉI**-ying **TÉI**-bol | tabla intermedia |
| idempotent | /ˌaɪ.dəmˈpoʊ.tənt/ | ai-dem-**POU**-tent | idempotente |
| bulk load | /bʌlk loʊd/ | balk loud | carga masiva |

### 💬 Useful sentences
- *"We **use parameterised queries** to **prevent** SQL injection."*
- *"The load **is idempotent**: if it **fails halfway**, we can **re-run** it safely."*

---
⬅️ [[PY-A04 Performance, Profiling and Memory]] · ➡️ [[Python Libraries Map]] · 🏠 [[00 - Python Roadmap]]

#python #advanced #databases
