---
title: SQL Practice Database
tags: [sql, practice, dataset]
---

# 🏪 SQL Practice Database — "DataShop"

A small online shop used by **every example and exercise** in the SQL track. The script works in **PostgreSQL** and **DuckDB** (and SQLite with minor changes noted below).

## Entity-relationship diagram
```
categories 1───* products 1───* order_items *───1 orders *───1 customers
                                                        │
employees (manager_id → employees.id)   orders.employee_id → employees.id
```

| Table | Description |
|---|---|
| `categories` | product categories |
| `products` | catalogue with price and stock |
| `customers` | buyers in Latin America and the US |
| `employees` | sales staff with a manager hierarchy |
| `orders` | one row per order (header) |
| `order_items` | one row per product in an order (detail) |

## Script
```sql
DROP TABLE IF EXISTS order_items; DROP TABLE IF EXISTS orders; DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS categories;  DROP TABLE IF EXISTS customers; DROP TABLE IF EXISTS employees;

CREATE TABLE categories (
    id    INTEGER PRIMARY KEY,
    name  VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE products (
    id           INTEGER PRIMARY KEY,
    name         VARCHAR(100) NOT NULL,
    category_id  INTEGER REFERENCES categories(id),
    price        DECIMAL(10,2) NOT NULL CHECK (price > 0),
    stock        INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0)
);

CREATE TABLE customers (
    id          INTEGER PRIMARY KEY,
    first_name  VARCHAR(50) NOT NULL,
    last_name   VARCHAR(50) NOT NULL,
    email       VARCHAR(100) UNIQUE,
    country     CHAR(2) NOT NULL,
    city        VARCHAR(50),
    signup_date DATE NOT NULL
);

CREATE TABLE employees (
    id          INTEGER PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    title       VARCHAR(50),
    manager_id  INTEGER REFERENCES employees(id),
    hire_date   DATE,
    salary      DECIMAL(10,2)
);

CREATE TABLE orders (
    id           INTEGER PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(id),
    employee_id  INTEGER REFERENCES employees(id),
    order_date   DATE NOT NULL,
    status       VARCHAR(20) NOT NULL CHECK (status IN ('pending','paid','shipped','cancelled','refunded'))
);

CREATE TABLE order_items (
    order_id    INTEGER REFERENCES orders(id),
    product_id  INTEGER REFERENCES products(id),
    quantity    INTEGER NOT NULL CHECK (quantity > 0),
    unit_price  DECIMAL(10,2) NOT NULL,
    discount    DECIMAL(4,2) NOT NULL DEFAULT 0,      -- 0.10 = 10 %
    PRIMARY KEY (order_id, product_id)
);

INSERT INTO categories VALUES (1,'Laptops'),(2,'Accessories'),(3,'Monitors'),(4,'Books'),(5,'Software');

INSERT INTO products VALUES
 (1,'ThinkPad X1 Carbon',1,1899.00,12),(2,'MacBook Air 13',1,1299.00,20),(3,'Dell XPS 15',1,2199.00,5),
 (4,'Wireless Mouse',2,25.50,300),(5,'Mechanical Keyboard',2,89.90,150),(6,'USB-C Hub',2,45.00,0),
 (7,'27in 4K Monitor',3,399.00,40),(8,'34in Ultrawide',3,649.00,8),
 (9,'SQL for Data Analysis',4,39.99,60),(10,'Python Crash Course',4,35.50,75),(11,'Designing Data-Intensive Applications',4,49.90,30),
 (12,'IDE Pro License',5,199.00,999),(13,'Cloud Backup 1yr',5,59.00,999),(14,'Laptop Stand',2,32.00,85),(15,'HD Webcam',2,59.90,25);

INSERT INTO customers VALUES
 (1,'Ana','López','ana.lopez@mail.com','MX','Mexico City','2025-01-15'),
 (2,'Luis','García','luis.garcia@mail.com','CO','Bogotá','2025-02-03'),
 (3,'Eva','Martínez','eva.m@mail.com','PE','Lima','2025-02-20'),
 (4,'John','Smith','john.smith@mail.com','US','Austin','2025-03-11'),
 (5,'María','Fernández',NULL,'MX','Guadalajara','2025-04-01'),
 (6,'Carlos','Ruiz','carlos.ruiz@mail.com','AR','Buenos Aires','2025-05-22'),
 (7,'Sofía','Torres','sofia.t@mail.com','CL','Santiago','2025-06-30'),
 (8,'Emily','Johnson','emily.j@mail.com','US','Seattle','2025-08-14'),
 (9,'Diego','Herrera','diego.h@mail.com','MX','Monterrey','2025-09-09'),
 (10,'Valentina','Rojas','vale.rojas@mail.com','CO','Medellín','2026-01-05'),
 (11,'Pedro','Sánchez','pedro.s@mail.com','PE','Arequipa','2026-03-18'),
 (12,'Laura','Gómez','laura.g@mail.com','MX','Puebla','2026-06-02');

INSERT INTO employees VALUES (1,'Isabel Navarro','CEO',NULL,'2020-01-10',9500);   -- managers first
INSERT INTO employees VALUES (2,'Roberto Díaz','Sales Director',1,'2021-03-01',6500),
 (6,'Tomás Peña','Data Analyst',1,'2025-01-20',4100);
INSERT INTO employees VALUES
 (3,'Camila Vargas','Sales Rep',2,'2022-05-16',3200),
 (4,'Andrés Castro','Sales Rep',2,'2023-02-01',3000),
 (5,'Lucía Morales','Sales Rep',2,'2024-07-15',2900);

INSERT INTO orders VALUES
 (1001,1,3,'2025-02-01','paid'),(1002,2,4,'2025-02-15','shipped'),(1003,1,3,'2025-03-10','shipped'),
 (1004,3,5,'2025-03-28','cancelled'),(1005,4,4,'2025-04-12','shipped'),(1006,5,3,'2025-05-05','paid'),
 (1007,1,3,'2025-06-18','refunded'),(1008,6,5,'2025-07-02','shipped'),(1009,7,4,'2025-08-09','shipped'),
 (1010,8,5,'2025-09-21','paid'),(1011,2,4,'2025-10-30','shipped'),(1012,9,3,'2025-11-11','shipped'),
 (1013,4,4,'2025-12-01','shipped'),(1014,1,3,'2026-01-07','paid'),(1015,10,5,'2026-02-14','shipped'),
 (1016,3,5,'2026-03-03','shipped'),(1017,8,4,'2026-04-22','pending'),(1018,2,4,'2026-05-19','paid'),
 (1019,11,NULL,'2026-07-01','paid'),(1020,1,3,'2026-08-15','shipped');

INSERT INTO order_items VALUES
 (1001,1,1,1899.00,0),(1001,4,2,25.50,0),(1002,9,1,39.99,0),(1002,10,1,35.50,0.10),
 (1003,7,2,399.00,0.05),(1003,5,1,89.90,0),(1004,3,1,2199.00,0),(1005,2,1,1299.00,0),
 (1005,14,1,32.00,0),(1006,12,3,199.00,0.15),(1007,8,1,649.00,0),(1008,11,2,49.90,0),
 (1008,4,1,25.50,0),(1009,2,1,1299.00,0.05),(1009,13,1,59.00,0),(1010,5,2,89.90,0),
 (1010,7,1,399.00,0),(1011,1,1,1899.00,0.10),(1012,10,3,35.50,0),(1012,9,1,39.99,0),
 (1013,3,1,2199.00,0),(1013,8,2,649.00,0.05),(1014,13,2,59.00,0),(1015,6,4,45.00,0),
 (1016,11,1,49.90,0),(1016,9,1,39.99,0),(1017,7,1,399.00,0),(1018,4,5,25.50,0),
 (1019,2,1,1299.00,0),(1020,12,1,199.00,0),(1020,5,1,89.90,0.10);
```

### SQLite notes
- Run `PRAGMA foreign_keys = ON;` first.
- `DECIMAL` is stored as REAL; dates are TEXT (`'2025-02-01'`) and work with `strftime`.

### Load it from Python
```python
import duckdb
con = duckdb.connect("datashop.duckdb")
con.execute(open("datashop.sql", encoding="utf-8").read())
con.sql("SELECT COUNT(*) FROM orders").show()
```

## Useful line-total expression (used everywhere)
```sql
quantity * unit_price * (1 - discount)   AS line_total
```

🏠 [[00 - SQL Roadmap]]

#sql #practice
