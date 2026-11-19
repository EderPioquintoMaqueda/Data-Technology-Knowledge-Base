
---
title: "Database Management Systems"
aliases:
  - "DBMS"
  - "Database Management System"
  - "Database Engine Fundamentals"
type: learning-note
domain: technology
category: database-systems
subject: database-management-systems
level: intermediate
difficulty: foundational-to-intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - dbms
  - database-engine
  - relational-databases
  - sql
  - postgresql
  - data-storage
  - transaction-management
  - query-processing

concepts:
  - database-management-system
  - database-engine
  - database-server
  - database-client
  - query-processing
  - storage-management
  - transaction-management
  - concurrency-control
  - access-control
  - buffer-management
  - logging
  - database-recovery

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - What is a Database System]]"
  - "[[05 - Database Architecture Fundamentals]]"
  - "[[06 - Schema Design Principles]]"

related:
  - "[[02 - How a Query is Executed]]"
  - "[[03 - Storage Pages and Tables]]"
  - "[[04 - Indexes]]"
  - "[[05 - Transactions and ACID]]"
  - "[[06 - Concurrency and Locks]]"
  - "[[PostgreSQL Architecture Overview]]"
  - "[[PostgreSQL Processes]]"

tools:
  - PostgreSQL
  - pgAdmin
  - SQL
  - psql
  - MySQL
  - Microsoft SQL Server
  - Oracle Database
  - IBM Db2

learning_objectives:
  - Define a Database Management System and explain its purpose.
  - Distinguish a DBMS, database engine, database server, and database application.
  - Understand the main responsibilities of a DBMS.
  - Explain how a database receives and processes requests.
  - Understand storage management, query processing, and transaction management.
  - Identify the role of memory, logs, indexes, and access control.
  - Compare major relational database management systems.
  - Relate DBMS responsibilities to a practical PostgreSQL application.

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 01 — Database Management Systems

## 1. Introduction

A Database Management System (DBMS) is software designed to create, organize, access, modify, protect, and maintain databases.

It provides the mechanisms that allow applications and users to interact with stored information without having to manage every detail of physical storage themselves.

A DBMS is responsible for much more than executing SQL statements.

Its responsibilities include:

- Managing database objects.
- Processing queries.
- Reading and writing data.
- Maintaining indexes.
- Enforcing constraints.
- Managing transactions.
- Controlling concurrent operations.
- Managing memory and storage.
- Authenticating users.
- Enforcing permissions.
- Recovering from failures.
- Maintaining database consistency.

Consider a vehicle management application.

When a user registers a maintenance operation, the application does not directly modify the physical files on the server.

Instead, it sends a request to the DBMS.

The DBMS interprets the request, checks the relevant permissions and constraints, performs the necessary operations, and returns a result.

The application relies on the DBMS to manage the complexity of database operations.

**The DBMS is the software layer that manages the database and provides controlled access to its information.**

## 2. What Is a DBMS?

A DBMS is a software system that acts as an intermediary between applications, users, and stored data.

Its purpose is to provide a structured and reliable way to manage information.

Without a DBMS, developers would need to implement many complex mechanisms themselves:

- Data organization.
- Record retrieval.
- Concurrency control.
- Storage management.
- Data validation.
- Recovery after failures.
- Access permissions.
- Transaction handling.

The DBMS provides these capabilities through a coordinated set of internal components.

### 2.1 Basic architecture

A simplified interaction looks like this:

```text
+-----------------------------+
|          USER               |
|                             |
|  Browser / Desktop / Mobile |
+--------------+--------------+
               |
               v
+-----------------------------+
|       APPLICATION           |
|                             |
|  Business Logic             |
|  Validation                 |
|  User Interface             |
+--------------+--------------+
               |
               v
+-----------------------------+
|      DATABASE DRIVER        |
|                             |
|  Connection Management      |
|  SQL Request Transmission   |
+--------------+--------------+
               |
               v
+-----------------------------+
|            DBMS             |
|                             |
|  Query Processing           |
|  Transaction Management     |
|  Storage Management         |
|  Security                   |
|  Recovery                   |
+--------------+--------------+
               |
               v
+-----------------------------+
|       DATABASE STORAGE      |
|                             |
|  Tables                     |
|  Indexes                    |
|  Transaction Logs           |
|  Metadata                   |
+-----------------------------+
```

The application communicates with the DBMS through a database driver or connection interface.

The DBMS manages the underlying database structures.

### 2.2 The DBMS as an intermediary

Suppose a user requests the maintenance history of vehicle 13990.

The application may generate:

```sql
SELECT *
FROM fleet.maintenance_records
WHERE vehicle_id = 13990;
```

The DBMS receives the statement and determines how to execute it.

It may need to:

1. Parse the SQL statement.
2. Validate its structure.
3. Check permissions.
4. Identify the referenced objects.
5. Select an execution strategy.
6. Access the required data.
7. Return the result.

The application does not need to know the exact physical disk location of every maintenance record.

That responsibility belongs to the DBMS.

## 3. Important Terminology

Several terms are frequently used together, but they do not mean exactly the same thing.

| Term | Meaning |
|---|---|
| DBMS | The complete database management software. |
| Database engine | The core mechanisms that process and manage database operations. |
| Database server | The running service or system that accepts database connections and processes requests. |
| Database | The organized collection of stored information and database objects. |
| Database client | Software that connects to and interacts with a database server. |
| Database driver | A software component that allows an application to communicate with a DBMS. |
| SQL | A language used to define, query, and manipulate relational database information. |

### 3.1 DBMS versus database

These concepts should never be confused.

Example:

```text
DBMS:
PostgreSQL

Database:
vehicle_management

Schema:
fleet

Table:
vehicles
```

PostgreSQL is the DBMS.

`vehicle_management` is a database managed by PostgreSQL.

`fleet` is a schema.

`vehicles` is a table.

### 3.2 DBMS versus database engine

The database engine is commonly understood as the core technology responsible for database operations.

Depending on the product architecture, the DBMS may include additional components such as:

- Administrative utilities.
- Monitoring interfaces.
- Backup tools.
- Client libraries.
- Replication management.
- Extensions.

In practice, the terms DBMS and database engine are sometimes used interchangeably, but the distinction is useful when studying architecture.

## 4. Main Responsibilities of a DBMS

A DBMS coordinates several responsibilities to provide reliable database operations.

### 4.1 Data definition

The DBMS allows users to define database structures.

This includes:

- Creating tables.
- Defining columns.
- Selecting data types.
- Creating relationships.
- Defining constraints.
- Creating indexes.
- Creating views.

Example:

```sql
CREATE TABLE fleet.vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    inventory_number VARCHAR(40) NOT NULL UNIQUE,
    brand VARCHAR(80) NOT NULL
);
```

This is a Data Definition Language (DDL) operation.

The DBMS interprets the statement and updates its internal catalog and database structures.

### 4.2 Data manipulation

The DBMS processes operations that retrieve or modify records.

Examples:

```sql
SELECT *
FROM fleet.vehicles;
```

```sql
INSERT INTO fleet.vehicles (
    inventory_number,
    brand
)
VALUES (
    '13990',
    'Shantui'
);
```

```sql
UPDATE fleet.vehicles
SET brand = 'Shantui'
WHERE inventory_number = '13990';
```

```sql
DELETE FROM fleet.vehicles
WHERE inventory_number = '13990';
```

These operations are commonly associated with Data Manipulation Language (DML).

### 4.3 Query processing

The DBMS analyzes SQL statements and determines how to execute them.

This process may involve:

- Parsing.
- Semantic analysis.
- Query rewriting.
- Optimization.
- Execution planning.
- Data retrieval.

For example, the DBMS may decide whether to use:

- A sequential scan.
- An index scan.
- A bitmap scan.
- A join algorithm.

The goal is to execute the requested operation correctly and efficiently.

Query processing will be studied in greater detail in [[02 - How a Query is Executed]].

### 4.4 Storage management

The DBMS manages how information is organized and stored.

It is responsible for operations involving:

- Tables.
- Rows.
- Pages.
- Index structures.
- Data files.
- Metadata.
- Temporary storage.

Applications interact with logical database objects rather than directly managing the physical organization of records.

### 4.5 Transaction management

A transaction is a logical unit of work that should be executed according to defined consistency and reliability rules.

Example:

Suppose an application transfers a vehicle assignment from one department to another.

The operation may require:

1. Closing the previous assignment.
2. Creating the new assignment.
3. Updating the current vehicle information.

These changes should not be partially applied.

A transaction allows the DBMS to coordinate the operations as a single logical unit.

Example:

```sql
BEGIN;

UPDATE fleet.vehicle_assignments
SET released_at = CURRENT_TIMESTAMP
WHERE vehicle_id = 1001
  AND released_at IS NULL;

INSERT INTO fleet.vehicle_assignments (
    vehicle_id,
    department_id,
    assigned_at
)
VALUES (
    1001,
    20,
    CURRENT_TIMESTAMP
);

COMMIT;
```

If an error occurs before the transaction is committed, the application or database can roll back the transaction.

Transaction management is studied in [[05 - Transactions and ACID]].

### 4.6 Concurrency control

A DBMS may receive requests from many users at the same time.

For example:

- One user updates a vehicle record.
- Another user retrieves its information.
- A reporting process calculates maintenance costs.
- An administrator modifies a department.

The DBMS must coordinate these operations to prevent inconsistent results.

Concurrency control may involve:

- Locks.
- Multi-Version Concurrency Control (MVCC).
- Isolation levels.
- Transaction scheduling.

These concepts are expanded in [[06 - Concurrency and Locks]].

### 4.7 Security and access control

A DBMS controls who can access database resources and which operations they can perform.

For example:

```sql
GRANT SELECT
ON fleet.vehicles
TO reporting_reader;
```

This grants read access to the specified table.

Other users may receive permissions to:

- Insert records.
- Update records.
- Delete records.
- Create database objects.
- Manage roles.
- Perform administrative operations.

A properly configured DBMS helps protect information from unauthorized access.

### 4.8 Backup and recovery

A DBMS provides mechanisms to recover information after failures.

Possible failures include:

- Application crashes.
- Operating system failures.
- Power interruptions.
- Hardware failures.
- Accidental data modification.
- Database corruption.

Recovery mechanisms may use:

- Transaction logs.
- Backups.
- Checkpoints.
- Write-Ahead Logging (WAL).
- Replication.

These capabilities are essential for production environments.

## 5. Internal Components of a DBMS

A DBMS is not a single isolated mechanism.

It consists of components that cooperate to process requests and manage information.

A simplified conceptual architecture includes:

```text
+------------------------------------+
|           DBMS                     |
|                                    |
|  +------------------------------+  |
|  | Connection / Session Manager |  |
|  +------------------------------+  |
|                 |                  |
|                 v                  |
|  +------------------------------+  |
|  | Query Processor              |  |
|  |                              |  |
|  | Parser                       |  |
|  | Analyzer                     |  |
|  | Rewriter                     |  |
|  | Optimizer                    |  |
|  | Executor                     |  |
|  +------------------------------+  |
|                 |                  |
|                 v                  |
|  +------------------------------+  |
|  | Transaction Manager          |  |
|  |                              |  |
|  | Concurrency Control          |  |
|  | Recovery Coordination        |  |
|  +------------------------------+  |
|                 |                  |
|                 v                  |
|  +------------------------------+  |
|  | Storage Manager              |  |
|  |                              |  |
|  | Buffer Management            |  |
|  | File Management              |  |
|  | Index Management             |  |
|  +------------------------------+  |
|                 |                  |
|                 v                  |
|  +------------------------------+  |
|  | Persistent Storage           |  |
|  +------------------------------+  |
+------------------------------------+
```

This is a conceptual model, not a universal implementation diagram.

Different DBMS products organize their internal components differently.

### 5.1 Connection and session management

The DBMS receives connections from clients.

A connection represents a communication channel between a client and the database server.

A session maintains the context associated with a connection.

This context may include:

- Authenticated user.
- Current database.
- Session parameters.
- Transaction state.
- Temporary objects.
- Current search path.

Connection management is important because creating and maintaining database connections consumes resources.

Applications often use connection pools to reuse connections rather than creating a new one for every request.

### 5.2 Query processor

The query processor interprets and prepares SQL statements.

Its responsibilities include:

- Parsing SQL.
- Checking object references.
- Validating data types.
- Rewriting queries.
- Generating execution plans.
- Coordinating execution.

It is one of the most important components of a relational DBMS.

### 5.3 Transaction manager

The transaction manager coordinates logical operations.

It helps enforce transaction boundaries and works with concurrency and recovery mechanisms.

Its purpose is to ensure that database operations follow the required consistency and isolation rules.

### 5.4 Storage manager

The storage manager controls how data is organized and accessed.

Its responsibilities may include:

- Managing data files.
- Accessing table pages.
- Managing indexes.
- Coordinating reads and writes.
- Interacting with memory buffers.

### 5.5 Buffer manager

Reading from storage devices is generally more expensive than accessing data already available in memory.

A buffer manager maintains database pages in memory to reduce unnecessary disk access.

In PostgreSQL, the shared buffer area is an important part of this mechanism.

A query may find the required page already in memory, avoiding a physical read from the data files.

This is commonly called a cache hit.

### 5.6 Log and recovery mechanisms

DBMS products use logging mechanisms to support durability and recovery.

PostgreSQL uses Write-Ahead Logging (WAL).

The basic principle is that the relevant log records must be safely written before modified data pages are considered safely persisted.

This allows PostgreSQL to recover database consistency after certain failures.

WAL is explored further in the PostgreSQL architecture block.

## 6. How a DBMS Processes a Request

Consider a user who wants to retrieve a vehicle's information.

The application sends an SQL query to the database server.

```sql
SELECT
    vehicle_id,
    inventory_number,
    brand,
    model
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

A simplified execution flow is:

### Step 1 — Connection

The application establishes a connection to the database server.

### Step 2 — Authentication

The DBMS verifies the identity of the connecting user.

### Step 3 — Parsing

The SQL statement is checked for valid syntax.

### Step 4 — Semantic analysis

The DBMS verifies that referenced objects and columns exist and that the statement is meaningful.

### Step 5 — Permission validation

The DBMS checks whether the user has permission to perform the requested operation.

### Step 6 — Planning

The query optimizer evaluates possible execution strategies.

For example, it may choose an index scan if an appropriate index exists.

### Step 7 — Execution

The executor requests the required data through the storage mechanisms.

### Step 8 — Result delivery

The DBMS returns the resulting rows to the client.

The application receives the result and presents it to the user.

This process is simplified. Actual execution can involve additional internal operations.

## 7. Different Types of DBMS

Database management systems can be classified according to their data models and architectural characteristics.

### 7.1 Relational DBMS

A Relational Database Management System (RDBMS) organizes information using tables with rows and columns.

Relationships are established using keys.

Examples:

- [[PostgreSQL]]
- [[MySQL]]
- [[Microsoft SQL Server]]
- [[Oracle Database]]
- [[IBM Db2]]

Common characteristics:

- Relational data model.
- SQL support.
- Primary and foreign keys.
- Constraints.
- Transactions.
- Structured schemas.

RDBMS platforms are widely used for transactional and analytical applications.

### 7.2 Document-oriented DBMS

Document databases store information as documents, often using JSON-like structures.

Example:

[[Database Types - Relational, NoSQL and Vector]]

A document may contain nested objects and arrays.

This model can be useful when information has flexible or evolving structures.

However, document databases still require deliberate modeling and consistency decisions.

### 7.3 Key-value databases

Key-value databases store information as pairs:

```text
Key -> Value
```

Example:

```text
vehicle:13990 -> {"brand":"Shantui","model":"SG16"}
```

A key-value system is often optimized for simple lookups using unique keys.

An example technology is [[Database Types - Relational, NoSQL and Vector]].

### 7.4 Wide-column databases

Wide-column systems organize information around rows, column families, and distributed storage patterns.

An example is [[Database Types - Relational, NoSQL and Vector]].

They are commonly used in distributed applications that require high availability and horizontal scalability.

### 7.5 Graph databases

Graph databases represent information through:

- Nodes.
- Relationships.
- Properties.

They are useful when connections between entities are central to the application.

An example is [[Database Types - Relational, NoSQL and Vector]].

### 7.6 Classification summary

| DBMS type | Main data model | Example |
|---|---|---|
| Relational | Tables and relationships | PostgreSQL |
| Document | Documents | MongoDB |
| Key-value | Key-value pairs | Redis |
| Wide-column | Distributed column families | Cassandra |
| Graph | Nodes and relationships | Neo4j |

The choice of DBMS depends on data structure, query patterns, consistency requirements, scalability, operational constraints, and application needs.

## 8. Major Relational DBMS Platforms

Several relational database systems are widely used in professional environments.

### 8.1 PostgreSQL

PostgreSQL is an open-source object-relational database management system.

Important characteristics:

- Advanced SQL support.
- ACID-compliant transactions.
- MVCC.
- Extensibility.
- JSON and JSONB support.
- Advanced indexing.
- Replication capabilities.
- Strong standards support.

It is useful for transactional applications, analytical workloads, and systems with complex relational requirements.

### 8.2 MySQL

MySQL is a widely used relational DBMS.

Important characteristics:

- Broad web application adoption.
- Client-server architecture.
- SQL support.
- InnoDB transactional storage engine.
- Replication capabilities.
- Ecosystem integration.

It is commonly used in web applications and business systems.

### 8.3 Microsoft SQL Server

Microsoft SQL Server is a relational DBMS developed by Microsoft.

Important characteristics:

- Integration with Microsoft technologies.
- SQL Server Management Studio.
- T-SQL.
- Business intelligence ecosystem.
- Enterprise administration tools.
- High availability features.

It is frequently used in enterprise environments.

### 8.4 Oracle Database

Oracle Database is an enterprise relational database platform.

Important characteristics:

- Advanced transaction management.
- Enterprise availability features.
- Partitioning capabilities.
- Advanced security mechanisms.
- Large-scale workload support.
- Enterprise administration tools.

It is used in many large organizations and critical business environments.

### 8.5 IBM Db2

IBM Db2 is a relational database management system developed by IBM.

Important characteristics:

- Enterprise database capabilities.
- Transaction management.
- SQL support.
- High availability options.
- Performance optimization.
- Integration with IBM enterprise environments.

Db2 is particularly relevant when studying enterprise database architecture within the IBM ecosystem.

### 8.6 Platform comparison

| Platform | Main context | Important technologies |
|---|---|---|
| PostgreSQL | Open-source relational systems | SQL, MVCC, WAL, JSONB |
| MySQL | Web and business applications | SQL, InnoDB, replication |
| SQL Server | Microsoft enterprise environments | T-SQL, SSMS, integration |
| Oracle Database | Enterprise-critical systems | PL/SQL, RAC, partitioning |
| IBM Db2 | IBM enterprise environments | SQL, enterprise transaction processing |

The platforms share relational principles but differ in architecture, administration, implementation details, and available features.

Learning PostgreSQL provides a strong foundation for understanding many concepts that also exist in other relational systems.

However, product-specific behavior should always be studied separately.

## 9. Client Tools Versus the DBMS

A common misunderstanding is to consider database administration tools equivalent to the DBMS itself.

They are different components.

### 9.1 pgAdmin

[[PostgreSQL Practical Guide]] is a graphical administration and development tool for PostgreSQL.

It allows users to:

- Connect to PostgreSQL servers.
- Browse database objects.
- Execute SQL statements.
- Manage roles.
- Inspect tables.
- Review query results.
- Configure certain administrative operations.

pgAdmin does not replace PostgreSQL.

It communicates with the PostgreSQL server.

### 9.2 psql

`psql` is PostgreSQL's interactive command-line client.

Example:

```bash
psql -h localhost -p 5432 -U postgres -d vehicle_management
```

Parameters:

| Parameter | Meaning |
|---|---|
| `-h` | Host |
| `-p` | Port |
| `-U` | Database user |
| `-d` | Database name |

Once connected, you can execute SQL statements:

```sql
SELECT version();
```

You can also use PostgreSQL-specific commands:

```text
\l
```

Lists databases.

```text
\dt
```

Lists tables visible in the current search path.

```text
\dn
```

Lists schemas.

```text
\du
```

Lists roles.

These commands are `psql` meta-commands, not ordinary SQL statements.

### 9.3 Application drivers

Applications communicate with a DBMS using drivers or database connectivity libraries.

Examples:

| Language | PostgreSQL connectivity |
|---|---|
| Python | psycopg |
| Java | JDBC PostgreSQL driver |
| JavaScript / Node.js | node-postgres (`pg`) |
| C# | Npgsql |
| PHP | PDO PostgreSQL driver |

The driver translates application-level requests into the communication protocol understood by the database server.

## 10. What Happens When PostgreSQL Runs?

PostgreSQL provides a useful example for understanding a real DBMS.

When PostgreSQL is installed and started, it runs a database server process.

Applications connect to this server and submit SQL statements.

A simplified PostgreSQL architecture includes:

```text
+------------------------------------+
|          APPLICATION               |
|                                    |
| Python / Web App / pgAdmin / psql  |
+----------------+-------------------+
                 |
                 | Connection
                 v
+------------------------------------+
|       POSTGRESQL SERVER            |
|                                    |
|  Connection Handling               |
|  Query Processing                  |
|  Transaction Management            |
|  Storage Management                |
+----------------+-------------------+
                 |
                 v
+------------------------------------+
|         MEMORY                     |
|                                    |
|  Shared Buffers                    |
|  Work Memory                       |
|  Other Memory Areas                |
+----------------+-------------------+
                 |
                 v
+------------------------------------+
|       PERSISTENT STORAGE           |
|                                    |
|  Data Files                        |
|  Index Files                       |
|  WAL Files                         |
|  Metadata                          |
+------------------------------------+
```

This is a conceptual overview.

PostgreSQL has a more specific process architecture, which will be studied in:

- [[01 - PostgreSQL Architecture Overview]]
- [[02 - PostgreSQL Processes]]
- [[03 - PostgreSQL Storage]]

### 10.1 PostgreSQL server process

PostgreSQL uses a process-based architecture.

The main server process coordinates the database server.

Client connections are handled by dedicated backend processes in the traditional PostgreSQL architecture.

These backend processes execute queries and interact with shared resources.

### 10.2 Shared memory

PostgreSQL uses shared memory areas that can be accessed by multiple server processes.

One important example is `shared_buffers`.

It stores database pages in memory to reduce physical storage reads.

### 10.3 Persistent storage

PostgreSQL stores database information on persistent storage.

This includes:

- Table data.
- Indexes.
- Transaction logs.
- Metadata.

The exact physical layout is more complex than the logical table structure seen in SQL.

## 11. Why the DBMS Matters for Application Architecture

An application may have a sophisticated interface and business logic, but its reliability also depends on the database management system.

Consider a vehicle management application.

The application handles:

- User interaction.
- Forms.
- Validation.
- Reports.
- Business workflows.

The DBMS handles:

- Persistent data storage.
- Referential integrity.
- Transactions.
- Concurrent access.
- Query execution.
- Permissions.
- Recovery.

These responsibilities complement each other.

### 11.1 Example: Registering a vehicle

A user submits a vehicle registration form.

The application sends an INSERT statement:

```sql
INSERT INTO fleet.vehicles (
    inventory_number,
    brand,
    model
)
VALUES (
    '13990',
    'Shantui',
    'SG16'
);
```

The DBMS then:

1. Receives the SQL statement.
2. Checks its syntax.
3. Verifies the referenced table and columns.
4. Checks permissions.
5. Validates constraints.
6. Generates the required internal operations.
7. Writes the new record.
8. Completes the transaction.
9. Returns a result to the application.

If the inventory number already exists, the UNIQUE constraint prevents the insertion.

The application can then report the problem to the user.

The important point is that data integrity is not dependent exclusively on application code.

The DBMS also enforces it.

### 11.2 Example: Multiple users

Imagine that two administrators attempt to modify the same vehicle record.

The DBMS coordinates concurrent operations according to its transaction and isolation mechanisms.

This helps prevent uncontrolled interference between users.

The exact behavior depends on the isolation level and the operations being executed.

### 11.3 Example: Database failure

Suppose the server loses power during a transaction.

The DBMS uses its recovery mechanisms to determine which changes were committed and how to restore a consistent database state.

This is one reason transaction logging and recovery are essential in production environments.

## 12. Practical Laboratory

The following exercises help connect the concepts to an actual PostgreSQL installation.

### Exercise 1 — Identify your DBMS

Connect to PostgreSQL using pgAdmin or `psql`.

Execute:

```sql
SELECT version();
```

Identify:

- DBMS name.
- Version.
- Operating system or platform information.
- Whether the output indicates a PostgreSQL distribution.

### Exercise 2 — Inspect the current database

Execute:

```sql
SELECT current_database();
```

Then:

```sql
SELECT current_user;
```

Then:

```sql
SELECT current_schema();
```

Explain what each result represents.

### Exercise 3 — Inspect database objects

Execute:

```sql
SELECT
    table_schema,
    table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
ORDER BY
    table_schema,
    table_name;
```

Identify:

- Schemas.
- Tables.
- System schemas.
- Application schemas.

### Exercise 4 — Execute a controlled query

Create a small table in a test database:

```sql
CREATE TABLE public.dbms_lab (
    record_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    description TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Insert a record:

```sql
INSERT INTO public.dbms_lab (description)
VALUES ('First DBMS laboratory record');
```

Retrieve it:

```sql
SELECT *
FROM public.dbms_lab;
```

Observe how the DBMS:

- Generates an identifier.
- Applies the default timestamp.
- Stores the record.
- Returns the requested information.

### Exercise 5 — Test a constraint

Try inserting a NULL description:

```sql
INSERT INTO public.dbms_lab (description)
VALUES (NULL);
```

Observe the error.

Explain why PostgreSQL rejects the operation.

Then inspect the table definition and identify the constraint responsible.

## 13. Knowledge Check

Answer these questions without consulting the previous sections.

1. What is a Database Management System?
2. What is the difference between a DBMS and a database?
3. Why is a DBMS necessary for multi-user applications?
4. What are the main responsibilities of a DBMS?
5. What is the role of a query processor?
6. What does a storage manager do?
7. Why does a DBMS need transaction management?
8. What is concurrency control?
9. What is the purpose of a buffer manager?
10. Why are logs important for database recovery?
11. What is the difference between PostgreSQL and pgAdmin?
12. What is the role of a database driver?
13. What is the difference between relational and document-oriented databases?
14. Why is IBM Db2 relevant to enterprise database architecture?
15. How does a DBMS help enforce data integrity?

## 14. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Database Management System (DBMS) | Sistema gestor de bases de datos |
| Relational Database Management System (RDBMS) | Sistema gestor de bases de datos relacional |
| Database engine | Motor de base de datos |
| Database server | Servidor de base de datos |
| Database client | Cliente de base de datos |
| Database driver | Controlador de conexión a la base de datos |
| Query processor | Procesador de consultas |
| Query optimizer | Optimizador de consultas |
| Query executor | Ejecutor de consultas |
| Storage manager | Gestor de almacenamiento |
| Buffer manager | Gestor de búferes |
| Transaction manager | Gestor de transacciones |
| Concurrency control | Control de concurrencia |
| Access control | Control de acceso |
| Authentication | Autenticación |
| Authorization | Autorización |
| Persistent storage | Almacenamiento persistente |
| Transaction log | Registro de transacciones |
| Recovery | Recuperación |
| Database catalog | Catálogo de base de datos |
| Connection pooling | Agrupación y reutilización de conexiones |
| Shared memory | Memoria compartida |
| Cache hit | Acierto de caché |
| Query execution | Ejecución de consultas |

## 15. Key Takeaways

- A DBMS is the software responsible for managing databases.
- It acts as an intermediary between applications and stored information.
- It manages database structures, queries, storage, transactions, concurrency, security, and recovery.
- The query processor interprets and executes SQL statements.
- The storage manager coordinates access to persistent data.
- The buffer manager helps reduce unnecessary storage access.
- Transaction management protects the logical consistency of operations.
- Concurrency control coordinates simultaneous database requests.
- Recovery mechanisms help restore database consistency after failures.
- PostgreSQL, MySQL, SQL Server, Oracle, and IBM Db2 are examples of relational DBMS platforms.
- pgAdmin and psql are clients, not database engines.
- A DBMS is a critical component of application architecture.

**Understanding the DBMS means understanding the software responsible for turning database structures and SQL instructions into reliable operations.**

## 16. Related Notes

### Previous concepts

- [[01 - What is a Database System]]
- [[05 - Database Architecture Fundamentals]]
- [[06 - Schema Design Principles]]

### Next concepts

- [[02 - How a Query is Executed]]
- [[03 - Storage Pages and Tables]]
- [[04 - Indexes]]
- [[05 - Transactions and ACID]]
- [[06 - Concurrency and Locks]]

### PostgreSQL architecture

- [[01 - PostgreSQL Architecture Overview]]
- [[02 - PostgreSQL Processes]]
- [[03 - PostgreSQL Storage]]
- [[04 - PostgreSQL Query Planning]]
