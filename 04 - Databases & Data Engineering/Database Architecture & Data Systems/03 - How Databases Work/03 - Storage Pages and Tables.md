
---
title: "Storage Pages and Tables"
aliases:
  - "Database Storage Internals"
  - "Heap Pages and Tuples"
  - "PostgreSQL Table Storage"
type: learning-note
domain: technology
category: database-systems
subject: physical-storage
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - database-storage
  - storage-pages
  - heap-storage
  - tuples
  - mvcc
  - postgresql
  - vacuum
  - physical-data-model

concepts:
  - logical-storage
  - physical-storage
  - database-pages
  - database-blocks
  - heap-tables
  - heap-tuples
  - page-layout
  - relation-files
  - storage-forks
  - mvcc
  - dead-tuples
  - toast
  - free-space-map
  - visibility-map
  - vacuum
  - tablespaces

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Database Management Systems]]"
  - "[[02 - How a Query is Executed]]"
  - "[[06 - Schema Design Principles]]"

related:
  - "[[04 - Indexes]]"
  - "[[05 - Transactions and ACID]]"
  - "[[06 - Concurrency and Locks]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[03 - PostgreSQL Storage]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"

tools:
  - PostgreSQL
  - SQL
  - pgAdmin
  - psql
  - pageinspect

learning_objectives:
  - "Distinguish logical tables from their physical storage representation."
  - "Explain database pages, blocks, heap tuples, and relation files."
  - "Understand the basic internal layout of a PostgreSQL page."
  - "Explain how MVCC creates and manages row versions."
  - "Understand the purpose of VACUUM and autovacuum."
  - "Identify the roles of TOAST, FSM, and VM."
  - "Inspect table storage and relation sizes using PostgreSQL functions."
  - "Connect physical storage decisions with database performance."

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 03 — Storage Pages and Tables

## 1. Introduction

A database table looks simple from the SQL perspective:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT PRIMARY KEY,
    plate_number VARCHAR(15),
    brand VARCHAR(50),
    model VARCHAR(50),
    manufacture_year INTEGER
);
```

We interact with this structure using familiar operations:

```sql
INSERT INTO vehicles VALUES (...);

SELECT * FROM vehicles;

UPDATE vehicles
SET model = 'New Model'
WHERE vehicle_id = 1;

DELETE FROM vehicles
WHERE vehicle_id = 1;
```

However, the database management system must translate these logical operations into physical storage operations.

It must determine:

- Where the table data is stored.
- How records are organized.
- How individual rows are located.
- How space is allocated.
- How updates are represented.
- How deleted records are managed.
- How data is retrieved from memory or disk.

These responsibilities belong to the database storage subsystem.

**The central distinction is:**

- Logical representation: tables, columns, rows, and relationships.
- Physical representation: files, pages, tuple structures, indexes, and storage locations.

Understanding this distinction is essential for database architecture and performance engineering.

---

## 2. Logical Storage vs. Physical Storage

### 2.1 Logical representation

A relational database exposes data through logical structures.

The main structures are:

| Logical structure | Meaning |
|---|---|
| Database | An organized collection of database objects |
| Schema | A namespace that organizes database objects |
| Table | A logical structure containing records |
| Column | An attribute describing a type of information |
| Row | A record containing values |
| Constraint | A rule that protects data integrity |
| Index | A structure that accelerates data retrieval |

For example:

```sql
SELECT
    vehicle_id,
    plate_number,
    brand
FROM vehicles;
```

This query operates on the logical table `vehicles`.

The user does not need to know where its physical bytes are stored.

### 2.2 Physical representation

Internally, PostgreSQL represents table data through physical storage structures.

These include:

- Relation files.
- Storage pages.
- Heap tuples.
- Page metadata.
- Free-space information.
- Visibility information.
- Index structures.
- Auxiliary storage for large values.

The database engine manages these structures.

A simplified representation is:

```text
Database
│
└── Schema
    │
    └── Table: vehicles
        │
        └── Heap relation
            │
            ├── Page 0
            │   ├── Tuple 1
            │   ├── Tuple 2
            │   └── Tuple 3
            │
            ├── Page 1
            │   ├── Tuple 4
            │   └── Tuple 5
            │
            └── Page 2
                ├── Tuple 6
                └── Tuple 7
```

This diagram is conceptual. Actual page contents, free space, and tuple placement depend on the operations performed and the physical state of the relation.

### 2.3 Why the distinction matters

Imagine a table containing 200 vehicles.

At the logical level, it contains 200 records.

At the physical level, PostgreSQL must determine:

- How many pages are needed.
- How much space each record occupies.
- Where each record is located.
- Whether the required pages are already cached in memory.
- Whether an index can help locate a particular record.

A database architect must understand both perspectives.

---

## 3. Database Pages and Blocks

### 3.1 What is a database page?

A database page is a fixed-size unit used by a database system to organize and manage stored data.

PostgreSQL calls this unit a **block** or **page**.

In a standard PostgreSQL installation, the default block size is:

```text
8 KB = 8192 bytes
```

This value is determined when PostgreSQL is compiled and is not normally changed through an ordinary configuration setting.

A relation is divided into pages, and each page can contain multiple rows.

For example:

```text
Table: vehicles

Page 0
┌──────────────────────────────┐
│ Tuple 1                      │
│ Tuple 2                      │
│ Tuple 3                      │
│ Tuple 4                      │
│ Free space                   │
└──────────────────────────────┘

Page 1
┌──────────────────────────────┐
│ Tuple 5                      │
│ Tuple 6                      │
│ Tuple 7                      │
│ Tuple 8                      │
│ Free space                   │
└──────────────────────────────┘
```

A page does not necessarily contain the same number of rows as another page.

The number of rows depends on:

- The size of each tuple.
- The amount of free space.
- Tuple headers and metadata.
- Alignment and padding.
- Previous inserts, updates, and deletes.

### 3.2 Why use pages?

Reading and writing individual bytes or tiny fragments would be inefficient.

Pages provide a manageable unit for storage operations.

They help the DBMS:

1. Read groups of related data.
2. Manage free space.
3. Organize tuples.
4. Reduce the complexity of disk I/O.
5. Maintain page-level metadata.
6. Coordinate memory buffers and physical storage.

A query that needs one row may cause PostgreSQL to access a page containing several rows.

This is one reason why the physical organization of a table influences performance.

### 3.3 Page size and memory

The database does not necessarily read a page directly from a physical disk every time it is needed.

PostgreSQL uses shared buffers to cache database pages in memory.

A simplified flow is:

```text
SQL Query
    |
    v
Query Executor
    |
    v
Request a database page
    |
    v
Check shared buffers
    |
    +---- Page found ----> Use cached page
    |
    +---- Page missing --> Read page from storage
                              |
                              v
                         Load into memory
                              |
                              v
                         Execute operation
```

A page found in shared buffers avoids a physical storage read at that moment.

The operating system can also cache file contents, so actual storage behavior involves more than one memory layer.

**Important:** A query that returns one row does not necessarily read only the bytes occupied by that row.

---

## 4. Internal Structure of a PostgreSQL Page

A PostgreSQL heap page contains several internal regions.

A simplified layout is:

```text
┌──────────────────────────────────────┐
│ Page Header                          │
├──────────────────────────────────────┤
│ Item Identifier Array                │
│                                      │
│ Line Pointer 1                       │
│ Line Pointer 2                       │
│ Line Pointer 3                       │
│ ...                                  │
├──────────────────────────────────────┤
│                                      │
│ Free Space                           │
│                                      │
├──────────────────────────────────────┤
│ Tuple Data                           │
│                                      │
│ Tuple 3                              │
│ Tuple 2                              │
│ Tuple 1                              │
├──────────────────────────────────────┤
│ Special Space                        │
└──────────────────────────────────────┘
```

This is a conceptual representation of the standard page organization.

### 4.1 Page header

The page header contains metadata about the page.

It includes information such as:

- Page-related flags.
- Transaction-related information used by storage mechanisms.
- The location where the item identifier array ends.
- The beginning of the free-space region.
- The location of special space.
- Page-level identifiers and consistency information.

The header allows PostgreSQL to interpret and manage the page.

### 4.2 Item identifiers

The item identifier array contains entries commonly called **line pointers**.

A line pointer identifies the location of a tuple within the page.

It helps PostgreSQL locate a tuple without requiring the tuple to remain at exactly the same byte offset forever.

Conceptually:

```text
Item Identifier Array
│
├── Line Pointer 1 ───────> Tuple 1
├── Line Pointer 2 ───────> Tuple 2
└── Line Pointer 3 ───────> Tuple 3
```

This indirection is important because tuple data can move within a page while the corresponding line pointer provides a reference to its current location.

### 4.3 Free space

Free space is the unused area available for future storage operations.

PostgreSQL can use this space when inserting new tuples or creating new row versions.

The amount of available free space changes as records are inserted, updated, and removed.

### 4.4 Tuple data

Tuple data contains the actual stored row versions.

For example, a tuple might represent:

```text
vehicle_id: 1001
plate_number: ABC-123
brand: Nissan
model: Frontier
manufacture_year: 2024
```

The tuple includes more than just the user-defined column values.

It also contains internal metadata.

### 4.5 Special space

Special space is reserved for access-method-specific information.

For an ordinary PostgreSQL heap table, this region is generally not used for additional heap tuple data.

Other storage access methods may use it differently.

---

## 5. What Is a Heap Table?

### 5.1 Heap storage

PostgreSQL normally stores ordinary table rows using a structure called a **heap**.

A heap is a collection of tuples stored in pages.

The term does not mean that the records are stored in a binary heap data structure or necessarily arranged in a particular logical order.

In PostgreSQL, heap storage does not guarantee that rows are physically ordered by:

- Primary key.
- Insertion time.
- A particular column.
- The order in which a query requests them.

For example:

```sql
SELECT *
FROM vehicles;
```

Without an explicit `ORDER BY`, the result order is not guaranteed.

### 5.2 Heap tuples

A tuple is the internal representation of a row version.

A simplified tuple structure contains:

```text
Heap Tuple
│
├── Tuple Header
│   ├── Transaction metadata
│   ├── Tuple identification information
│   ├── Null bitmap information, when needed
│   └── Other internal metadata
│
├── User Data
│   ├── vehicle_id
│   ├── plate_number
│   ├── brand
│   ├── model
│   └── manufacture_year
│
└── Alignment and Padding
```

The exact physical representation is more complex and depends on PostgreSQL's internal storage rules.

### 5.3 Tuple header

The tuple header contains information required to manage the row version.

Among other things, PostgreSQL uses transaction identifiers and tuple metadata to determine whether a particular row version is visible to a transaction.

This is closely related to MVCC, which will be discussed later.

### 5.4 Null bitmap

A tuple may contain a bitmap indicating which columns have `NULL` values.

This information is used when interpreting stored column values.

The bitmap is not necessarily present in every tuple.

### 5.5 Alignment and padding

Some values must begin at specific memory or storage alignment boundaries.

PostgreSQL may introduce padding between values to satisfy these requirements.

Consequently, the physical size of a row is not always equal to the sum of the visible value lengths.

This matters when estimating table storage.

---

## 6. Physical Row Identification: CTID

PostgreSQL exposes a system column called `ctid`.

It identifies the physical location of a particular row version within its table.

Its format is:

```text
(block_number, item_number)
```

For example:

```text
(0, 1)
(0, 2)
(1, 1)
```

This means:

- The first value identifies the page or block.
- The second value identifies the tuple's line pointer within that page.

### 6.1 Inspecting CTID

```sql
SELECT
    vehicle_id,
    plate_number,
    ctid
FROM vehicles;
```

Possible output:

| vehicle_id | plate_number | ctid |
|---:|---|---|
| 1001 | ABC-123 | (0,1) |
| 1002 | DEF-456 | (0,2) |
| 1003 | GHI-789 | (1,1) |

The values above are illustrative.

### 6.2 Why CTID is not a primary key

A common misconception is that `ctid` can be used as a permanent row identifier.

It cannot.

The physical location may change when:

- A row is updated.
- A table is rewritten.
- `VACUUM FULL` reorganizes the table.
- Other storage operations relocate tuples.

A primary key identifies a logical record.

A `ctid` identifies a physical row version at a particular moment.

| Primary key | CTID |
|---|---|
| Logical identifier | Physical location |
| Designed for application use | Designed for internal storage identification |
| Stable unless explicitly changed | Can change |
| Enforces uniqueness when defined as a primary key | Does not provide a permanent business identity |

**Architecture principle:** Never use `ctid` as an application-level identifier.

---

## 7. MVCC: Why a Row Can Have Multiple Versions

One of PostgreSQL's most important storage concepts is **Multi-Version Concurrency Control (MVCC)**.

MVCC allows transactions to access consistent versions of data while other transactions are modifying it.

Instead of simply overwriting a row every time it changes, PostgreSQL generally creates a new row version when an update occurs.

### 7.1 A simple example

Suppose a vehicle has the following record:

```text
vehicle_id: 1001
model: Frontier
```

An application executes:

```sql
UPDATE vehicles
SET model = 'Navara'
WHERE vehicle_id = 1001;
```

Conceptually, PostgreSQL may represent the change like this:

```text
Before UPDATE

Old Tuple
┌──────────────────────┐
│ vehicle_id: 1001     │
│ model: Frontier      │
└──────────────────────┘


After UPDATE

Old Tuple
┌──────────────────────┐
│ vehicle_id: 1001     │
│ model: Frontier      │
│ Old version metadata │
└──────────────────────┘

New Tuple
┌──────────────────────┐
│ vehicle_id: 1001     │
│ model: Navara        │
│ New version metadata │
└──────────────────────┘
```

This is a simplified representation.

The old tuple is not immediately erased simply because a new version exists.

### 7.2 Transaction identifiers

PostgreSQL uses transaction-related metadata in tuple headers.

Two important fields are:

- `xmin`: the transaction identifier associated with the creation of a tuple version.
- `xmax`: the transaction identifier associated with the transaction that deleted or superseded a tuple version, when applicable.

These fields help PostgreSQL determine visibility.

They are not ordinary application columns.

### 7.3 Visibility

A transaction does not simply read every tuple that physically exists.

It evaluates which tuple versions are visible according to its snapshot and transaction rules.

For example:

```text
Transaction A:
    Reads a vehicle record.

Transaction B:
    Updates the same vehicle record.

Transaction A:
    Continues reading using its visibility rules.
```

MVCC helps prevent readers from seeing inconsistent intermediate changes.

The exact visibility rules depend on the transaction isolation level and transaction state.

### 7.4 Dead tuples

When an old tuple version is no longer visible to any active transaction, it can eventually become eligible for cleanup.

Such obsolete versions are commonly called **dead tuples**.

They occupy physical space until PostgreSQL can reclaim that space.

This is one reason that database storage can grow even when the number of currently visible rows remains relatively stable.

### 7.5 DELETE and physical removal

Consider:

```sql
DELETE FROM vehicles
WHERE vehicle_id = 1001;
```

The logical effect is that the row is no longer visible to transactions for which the deletion is visible.

However, the physical bytes are not necessarily erased immediately.

PostgreSQL must preserve the old version while transactions may still need it.

Later, cleanup processes can reclaim the space.

This distinction is essential:

- Logical deletion: the record is no longer visible as an active row.
- Physical cleanup: obsolete tuple storage is made available for reuse.

---

## 8. VACUUM and Dead Tuple Management

### 8.1 What is VACUUM?

`VACUUM` is a PostgreSQL maintenance operation that reclaims space occupied by obsolete tuple versions.

It also performs important maintenance related to transaction visibility and database operation.

Example:

```sql
VACUUM vehicles;
```

Or:

```sql
VACUUM (ANALYZE) vehicles;
```

The second command also updates planner statistics.

### 8.2 Why is VACUUM necessary?

Imagine a table that is updated frequently.

```text
Initial state:
100 active row versions

After many updates:
100 current row versions
+ 350 obsolete row versions
```

The application may still see approximately 100 current records.

However, the physical relation may contain significantly more tuple storage.

VACUUM helps reclaim space associated with versions that are no longer needed.

### 8.3 VACUUM does not necessarily shrink the file

An important distinction:

**Ordinary VACUUM generally makes reusable space available inside the relation. It does not normally return all that space to the operating system.**

For example:

```text
Before VACUUM

Page 0:
[Active][Dead][Active][Dead][Free]

After VACUUM

Page 0:
[Active][Free][Active][Free][Free]
```

The space can be reused by future inserts or updates.

The physical relation file may remain the same size.

### 8.4 VACUUM FULL

PostgreSQL also provides:

```sql
VACUUM FULL vehicles;
```

Unlike ordinary VACUUM, `VACUUM FULL` rewrites the table into a compact form.

It can return unused space to the operating system.

However, it has important operational consequences:

- Requires an exclusive lock on the table.
- Rewrites the relation.
- Requires additional disk space during the operation.
- Can be disruptive for applications that need continuous access.

It should not be treated as a routine substitute for ordinary VACUUM.

### 8.5 Autovacuum

PostgreSQL includes an automatic maintenance mechanism called **autovacuum**.

It helps perform routine cleanup and statistics maintenance.

Autovacuum monitors database activity and schedules maintenance according to configured thresholds and other conditions.

Its responsibilities include:

- Cleaning obsolete tuple versions.
- Preventing excessive accumulation of dead tuples.
- Updating statistics through automatic analysis.
- Helping prevent transaction ID wraparound problems.

An architect or database administrator must understand autovacuum because poorly configured or ineffective maintenance can contribute to storage growth and performance problems.

---

## 9. PostgreSQL Storage Forks

A PostgreSQL relation can have multiple physical storage components called **forks**.

A fork represents a particular type of storage information associated with a relation.

The principal forks are:

| Fork | Purpose |
|---|---|
| `main` | Main relation data |
| `fsm` | Free Space Map |
| `vm` | Visibility Map |
| `init` | Initialization data for unlogged relations |

### 9.1 Main fork

The main fork contains the primary relation data.

For a regular heap table, this is where the table's heap pages are stored.

### 9.2 Free Space Map (FSM)

The Free Space Map helps PostgreSQL track available space in relation pages.

It helps identify pages that may have enough room for new tuple storage.

Without this information, locating available space could require more extensive searching.

The FSM is an auxiliary structure.

It is not the actual table data.

### 9.3 Visibility Map (VM)

The Visibility Map tracks page-level visibility information.

In particular, it records whether all tuples on a page are visible to all transactions.

This can help PostgreSQL perform **index-only scans** more efficiently.

When a page is marked all-visible, PostgreSQL may avoid visiting the heap page to verify tuple visibility for an index-only scan.

This is one reason why vacuum maintenance can influence query performance.

### 9.4 Initialization fork

The initialization fork is associated with unlogged relations.

Unlogged tables do not provide the same crash-recovery guarantees as ordinary logged tables.

Their contents are not preserved in the same way after a crash or unclean shutdown.

The initialization fork allows PostgreSQL to restore an unlogged relation to its initial empty state when required.

### 9.5 Conceptual representation

```text
Relation: vehicles
│
├── main
│   └── Heap table data
│
├── fsm
│   └── Free-space information
│
├── vm
│   └── Visibility information
│
└── init
    └── Initialization data
        (for unlogged relations)
```

Not every relation has every fork.

---

## 10. TOAST: Storing Large Values

PostgreSQL supports variable-length values such as:

- `TEXT`
- `VARCHAR`
- `BYTEA`
- Other variable-length data types

Some values can become large enough that storing them directly inside ordinary heap tuples would be inefficient.

PostgreSQL uses a mechanism called **TOAST**.

TOAST stands for:

**The Oversized-Attribute Storage Technique.**

### 10.1 Why TOAST exists

Imagine a maintenance record containing:

```text
maintenance_id: 5001
vehicle_id: 1001
description: A very long technical report
```

If the description contains a large amount of text, PostgreSQL may need to manage that value differently from a small field.

TOAST can:

- Compress large values.
- Store large values outside the main heap tuple.
- Divide external values into smaller chunks.
- Retrieve the value when required.

Not every large value is necessarily stored externally. PostgreSQL applies storage strategies according to the data type, storage configuration, and value size.

### 10.2 Conceptual representation

```text
Main Heap Table
│
└── Maintenance Tuple
    │
    ├── maintenance_id
    ├── vehicle_id
    └── TOAST reference
             │
             v
        TOAST Relation
             │
             ├── Chunk 1
             ├── Chunk 2
             └── Chunk 3
```

This is a conceptual diagram. Actual TOAST storage includes its own internal structures and associated index.

### 10.3 Why TOAST matters

TOAST allows PostgreSQL to support large variable-length attributes without forcing every heap page to contain the complete value.

However, large values can still affect:

- Storage consumption.
- Query execution.
- Network transfer.
- Memory requirements.
- Backup size.

### 10.4 Practical architectural decision

Consider a fleet management application.

A maintenance record may contain:

- A short description.
- A detailed technical report.
- Photographs.
- PDF inspection documents.

A reasonable architecture might store:

| Information | Possible storage |
|---|---|
| Maintenance identifier | PostgreSQL |
| Vehicle identifier | PostgreSQL |
| Maintenance date | PostgreSQL |
| Short description | PostgreSQL |
| Detailed report text | PostgreSQL |
| Large photographs | Object or file storage |
| PDF documents | Object or file storage |
| Document metadata | PostgreSQL |

The decision depends on requirements, security, backup strategy, access patterns, and operational complexity.

TOAST is not a replacement for a complete document or object-storage architecture.

---

## 11. Relation Files and Tablespaces

### 11.1 What is a relation?

In PostgreSQL, the term **relation** is broader than an ordinary table.

Physical relations can include:

- Tables.
- Indexes.
- Materialized views.
- Sequences.
- Other storage-backed database objects.

Different relation types use different storage structures.

For example:

- A heap table stores tuple data.
- A B-tree index stores an index structure.
- A sequence stores sequence state.

### 11.2 Physical relation files

PostgreSQL stores relation data in files inside its database cluster.

A relation may occupy:

- One or more physical files.
- Multiple storage forks.
- Additional associated relations, such as TOAST storage.
- Separate index relations.

Large relations can be divided into multiple file segments.

Therefore, it is incorrect to assume that every logical table corresponds to exactly one operating-system file.

### 11.3 Tablespaces

A tablespace allows database objects to be associated with a particular storage location.

It provides a way to organize physical storage across different filesystem locations.

For example:

```text
Storage Architecture
│
├── Fast Storage
│   ├── Frequently accessed tables
│   └── Important indexes
│
└── Capacity Storage
    ├── Historical tables
    └── Less frequently accessed data
```

This is an architectural example, not a recommendation to separate objects without measuring workload behavior.

Tablespaces do not change the logical schema.

They influence where physical database objects are stored.

### 11.4 Inspecting a relation's file path

PostgreSQL provides:

```sql
SELECT pg_relation_filepath('public.vehicles');
```

This returns the relation's path relative to the database cluster directory.

The exact output depends on the database and tablespace configuration.

You can also inspect relation size:

```sql
SELECT pg_relation_size('public.vehicles');
```

Physical paths are implementation details.

Applications should not depend on them as permanent identifiers.

---

## 12. Row-Oriented vs. Column-Oriented Storage

Another important storage concept is the distinction between row-oriented and column-oriented organization.

### 12.1 Row-oriented storage

A row-oriented system stores the values of a record together as a logical unit.

Conceptually:

```text
Row 1:
[1001, ABC-123, Nissan, Frontier, 2024]

Row 2:
[1002, DEF-456, Ford, Ranger, 2023]

Row 3:
[1003, GHI-789, Toyota, Hilux, 2022]
```

This organization is useful for workloads that frequently access or modify complete records.

Examples include:

- Inserting a new vehicle.
- Updating a maintenance record.
- Retrieving a complete customer profile.
- Processing operational transactions.

PostgreSQL's standard heap table storage is row-oriented.

### 12.2 Column-oriented storage

A column-oriented system organizes data by column.

Conceptually:

```text
vehicle_id:
[1001, 1002, 1003]

plate_number:
[ABC-123, DEF-456, GHI-789]

brand:
[Nissan, Ford, Toyota]

model:
[Frontier, Ranger, Hilux]
```

This can be useful when analytical queries scan a small number of columns across very large datasets.

For example:

```sql
SELECT
    brand,
    COUNT(*)
FROM vehicles
GROUP BY brand;
```

A columnar system may avoid reading unrelated columns when executing this type of analytical workload.

### 12.3 Comparison

| Characteristic | Row-oriented | Column-oriented |
|---|---|---|
| Main organization | Records | Columns |
| Typical strength | Transactional operations | Analytical scans |
| Complete row retrieval | Often efficient | May require multiple column reads |
| Aggregations over selected columns | May read additional data | Often efficient |
| Common workload | OLTP | OLAP |

This is a general architectural comparison. Actual performance depends on the engine, compression, indexes, query patterns, and hardware.

### 12.4 Connection with OLTP and OLAP

The storage organization should reflect workload requirements.

Review:

- [[04 - OLTP vs OLAP]]
- [[05 - Normalization]]

A transactional application and an analytical platform may use different storage architectures even when they work with related business information.

---

## 13. Measuring Physical Storage in PostgreSQL

Understanding storage becomes much more useful when we can inspect it.

PostgreSQL provides several functions for measuring relation sizes.

### 13.1 pg_relation_size()

Returns the size of a relation fork, in bytes.

```sql
SELECT
    pg_relation_size('public.vehicles');
```

By default, it measures the main fork.

You can specify a fork:

```sql
SELECT
    pg_relation_size('public.vehicles', 'main');
```

Other possible fork names include:

```text
main
fsm
vm
init
```

The initialization fork exists only where applicable.

### 13.2 pg_table_size()

Returns the size of a table and associated storage, excluding its indexes.

It includes the table's auxiliary storage, such as TOAST storage where applicable.

```sql
SELECT
    pg_table_size('public.vehicles');
```

### 13.3 pg_indexes_size()

Returns the total size of indexes associated with a table.

```sql
SELECT
    pg_indexes_size('public.vehicles');
```

### 13.4 pg_total_relation_size()

Returns the total disk space used by a table and its associated indexes and auxiliary storage.

```sql
SELECT
    pg_total_relation_size('public.vehicles');
```

### 13.5 pg_size_pretty()

Converts a byte value into a readable representation.

```sql
SELECT
    pg_size_pretty(
        pg_total_relation_size('public.vehicles')
    );
```

### 13.6 Compare storage components

```sql
SELECT
    pg_size_pretty(
        pg_relation_size('public.vehicles')
    ) AS main_relation_size,

    pg_size_pretty(
        pg_table_size('public.vehicles')
    ) AS table_size,

    pg_size_pretty(
        pg_indexes_size('public.vehicles')
    ) AS indexes_size,

    pg_size_pretty(
        pg_total_relation_size('public.vehicles')
    ) AS total_size;
```

### 13.7 Important distinctions

| Function | What it measures |
|---|---|
| `pg_relation_size()` | A particular relation fork; main by default |
| `pg_table_size()` | Table storage, including associated auxiliary storage, excluding indexes |
| `pg_indexes_size()` | Index storage associated with the table |
| `pg_total_relation_size()` | Table, indexes, and associated auxiliary storage |
| `pg_size_pretty()` | Formats a size for readability |

These functions are useful for storage monitoring, capacity planning, and performance investigations.

---

## 14. Inspecting Table Statistics

PostgreSQL exposes statistics about tables through system catalog views.

One useful view is:

```sql
pg_stat_user_tables
```

It contains information about user-defined tables.

For example:

```sql
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE schemaname = 'public';
```

Important fields include:

| Field | Meaning |
|---|---|
| `schemaname` | Schema containing the table |
| `relname` | Table name |
| `n_live_tup` | Estimated number of live tuples |
| `n_dead_tup` | Estimated number of dead tuples |
| `last_vacuum` | Last manual vacuum |
| `last_autovacuum` | Last automatic vacuum |
| `last_analyze` | Last manual analyze |
| `last_autoanalyze` | Last automatic analyze |

These values are statistics and estimates, not necessarily exact real-time counts.

They help identify tables that may need investigation.

---

## 15. Practical Lab — Inspecting PostgreSQL Storage

The purpose of this lab is to observe the relationship between logical records and physical storage.

Use a development database or a disposable environment.

### 15.1 Create a laboratory schema

```sql
CREATE SCHEMA IF NOT EXISTS storage_lab;
```

### 15.2 Create a table

```sql
CREATE TABLE storage_lab.maintenance_records (
    maintenance_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    vehicle_id BIGINT NOT NULL,
    maintenance_date DATE NOT NULL,
    maintenance_type VARCHAR(50) NOT NULL,
    description TEXT,
    estimated_cost NUMERIC(12, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

This table contains:

- An identity column.
- Integer values.
- Date values.
- Variable-length text.
- Numeric values.
- A timestamp.

### 15.3 Insert sample records

```sql
INSERT INTO storage_lab.maintenance_records (
    vehicle_id,
    maintenance_date,
    maintenance_type,
    description,
    estimated_cost
)
VALUES
(
    1001,
    '2026-01-10',
    'Preventive',
    'Oil and filter replacement.',
    2500.00
),
(
    1002,
    '2026-01-15',
    'Corrective',
    'Inspection and replacement of damaged components.',
    4800.00
),
(
    1003,
    '2026-02-01',
    'Preventive',
    'Scheduled inspection and lubrication.',
    1800.00
);
```

### 15.4 Inspect logical records

```sql
SELECT *
FROM storage_lab.maintenance_records;
```

At this stage, you are inspecting the logical representation.

### 15.5 Inspect CTID

```sql
SELECT
    maintenance_id,
    vehicle_id,
    ctid
FROM storage_lab.maintenance_records;
```

Observe how PostgreSQL exposes the physical location of each current row version.

Do not assume that the same CTID will remain after updates or table rewrites.

### 15.6 Inspect individual row sizes

```sql
SELECT
    maintenance_id,
    pg_column_size(maintenance_records) AS row_size_bytes
FROM storage_lab.maintenance_records;
```

`pg_column_size()` reports the storage size of a value or record representation.

It is useful for comparison, but it is not a complete accounting of every physical overhead associated with the table.

### 15.7 Inspect relation sizes

```sql
SELECT
    pg_size_pretty(
        pg_relation_size(
            'storage_lab.maintenance_records'
        )
    ) AS main_relation_size,

    pg_size_pretty(
        pg_table_size(
            'storage_lab.maintenance_records'
        )
    ) AS table_size,

    pg_size_pretty(
        pg_indexes_size(
            'storage_lab.maintenance_records'
        )
    ) AS indexes_size,

    pg_size_pretty(
        pg_total_relation_size(
            'storage_lab.maintenance_records'
        )
    ) AS total_size;
```

Compare the different measurements.

A small table may occupy at least one database page even if its actual user data is much smaller.

### 15.8 Inspect the physical relation path

```sql
SELECT
    pg_relation_filepath(
        'storage_lab.maintenance_records'
    ) AS relation_path;
```

This demonstrates that a logical table has an underlying physical storage representation.

### 15.9 Generate new row versions

```sql
UPDATE storage_lab.maintenance_records
SET description = 'Updated maintenance description.'
WHERE maintenance_id = 1;
```

Now inspect:

```sql
SELECT
    maintenance_id,
    description,
    ctid
FROM storage_lab.maintenance_records;
```

The row version may have a different CTID after the update.

The previous tuple version may remain physically present until it becomes eligible for cleanup.

### 15.10 Delete a record

```sql
DELETE FROM storage_lab.maintenance_records
WHERE maintenance_id = 2;
```

Then inspect the table:

```sql
SELECT *
FROM storage_lab.maintenance_records;
```

The deleted record is no longer visible as an active row.

However, its physical storage may remain until cleanup.

### 15.11 Inspect dead tuple statistics

```sql
SELECT
    relname,
    n_live_tup,
    n_dead_tup,
    last_autovacuum
FROM pg_stat_user_tables
WHERE schemaname = 'storage_lab';
```

Statistics may not immediately reflect the latest changes.

For a small test table, the estimated values may be imprecise.

### 15.12 Run VACUUM

```sql
VACUUM (ANALYZE) storage_lab.maintenance_records;
```

Important:

`VACUUM` cannot run inside an explicit transaction block.

If you use pgAdmin or another SQL client, ensure that the command is executed outside an explicit transaction.

Inspect the statistics again:

```sql
SELECT
    relname,
    n_live_tup,
    n_dead_tup,
    last_vacuum
FROM pg_stat_user_tables
WHERE schemaname = 'storage_lab';
```

Observe the changes, keeping in mind that statistics are estimates and maintenance timing matters.

### 15.13 Clean up the laboratory

When you finish:

```sql
DROP SCHEMA storage_lab CASCADE;
```

This removes the laboratory schema and its objects.

Do not execute the cleanup command if you want to preserve the lab for later review.

---

## 16. Advanced Inspection: pageinspect

PostgreSQL provides an optional extension called `pageinspect`.

It allows advanced inspection of the internal contents of database pages.

It can help investigate:

- Page headers.
- Line pointers.
- Heap tuple metadata.
- Physical page organization.

The extension may not be installed or enabled in every environment.

It also requires appropriate permissions.

### 16.1 Enable the extension

In a disposable development environment:

```sql
CREATE EXTENSION IF NOT EXISTS pageinspect;
```

### 16.2 Read a page

A simplified example:

```sql
SELECT *
FROM page_header(
    get_raw_page(
        'storage_lab.maintenance_records',
        0
    )
);
```

This reads the first page of the specified relation and displays its page-header information.

Another example:

```sql
SELECT *
FROM heap_page_items(
    get_raw_page(
        'storage_lab.maintenance_records',
        0
    )
);
```

This can expose information about tuples and line pointers on the page.

### 16.3 Important precautions

- Use a disposable development database.
- Do not assume the extension is available in production.
- Do not modify raw page contents.
- Do not treat internal page information as a stable application interface.
- Understand that low-level inspection requires knowledge of PostgreSQL internals.

For now, understanding the purpose of `pageinspect` is sufficient.

---

## 17. Common Storage Misconceptions

| Misconception | Correct understanding |
|---|---|
| One logical table always equals one physical file. | A relation may have multiple forks, segments, and associated storage. |
| Rows are stored in primary-key order. | Heap storage does not guarantee logical row ordering. |
| CTID is a permanent row identifier. | CTID represents a physical tuple location and can change. |
| DELETE immediately erases bytes from disk. | Old tuple versions may remain until cleanup. |
| VACUUM always reduces the file size. | Ordinary VACUUM generally makes space reusable inside the relation. |
| Every row occupies the same number of bytes. | Tuple size depends on values, metadata, nulls, alignment, and storage representation. |
| PostgreSQL reads the disk for every query. | Pages can be served from shared buffers or operating-system cache. |
| All large text values are stored externally. | TOAST applies storage strategies depending on the value and configuration. |
| Indexes are physically part of the heap table. | Indexes are separate relations with their own storage structures. |
| A page contains only one row. | A page can contain multiple tuples. |

---

## 18. Applying These Concepts to SIAPV

Consider an application for managing a fleet of vehicles and machinery.

Its database may contain:

```text
vehicles
maintenance_records
meter_readings
insurance_records
accidents
operators
workshops
```

At the logical level, each table contains records with defined relationships.

At the physical level, PostgreSQL organizes these records into relation storage structures.

### 18.1 Vehicle inventory

A vehicle table may contain:

- Inventory number.
- Plate number.
- Brand.
- Model.
- Serial number.
- Year.
- Operational status.

These fields represent structured data that is frequently queried.

### 18.2 Maintenance history

Maintenance records may be updated or inserted frequently.

MVCC becomes relevant because the application must support concurrent reads and modifications.

For example:

- One employee registers a maintenance event.
- Another employee consults the vehicle history.
- A report generates operational information.

The database must preserve consistent visibility while these operations occur.

### 18.3 Meter readings

Horometer and mileage records are historical observations.

A new reading should normally be represented as a new record rather than overwriting the previous observation.

This preserves the historical sequence.

A possible model:

```text
meter_readings
│
├── reading_id
├── vehicle_id
├── reading_date
├── reading_type
├── reading_value
└── registered_by
```

### 18.4 Large documents and attachments

A vehicle or maintenance record may be associated with:

- Inspection reports.
- Photographs.
- Insurance documents.
- Technical PDFs.
- Repair evidence.

The architecture should distinguish between the metadata describing a document and the document itself.

For example:

```text
maintenance_records
│
├── maintenance_id
├── vehicle_id
├── maintenance_date
└── description

maintenance_documents
│
├── document_id
├── maintenance_id
├── document_name
├── storage_location
└── uploaded_at
```

The binary file may be stored in object or file storage, while PostgreSQL stores its metadata and relationship.

This is a design option, not a mandatory rule.

### 18.5 Architectural questions

When evaluating SIAPV storage, ask:

1. Which tables receive the most inserts?
2. Which tables receive frequent updates?
3. Which tables accumulate historical records?
4. Are large documents stored in the database or externally?
5. How much space do indexes consume?
6. Are dead tuples accumulating?
7. Is autovacuum operating effectively?
8. Are queries retrieving complete records or only selected columns?
9. Which tables are growing fastest?
10. Does the physical storage strategy support expected future growth?

These questions connect database implementation with architecture-level reasoning.

---

## 19. Key Architectural Principles

1. **Logical and physical structures are different.** Applications interact with tables, while the DBMS manages pages, tuples, files, and auxiliary structures.

2. **Pages are fundamental storage units.** PostgreSQL normally uses 8 KB blocks, although the size is determined at build time.

3. **Heap tables do not guarantee row ordering.** Query results require an explicit `ORDER BY` when order matters.

4. **A tuple contains internal metadata.** Physical row storage includes more than visible column values.

5. **CTID is not a business identifier.** It identifies a physical tuple location, not a permanent logical record.

6. **MVCC maintains multiple row versions.** This supports transactional visibility and concurrent access.

7. **Dead tuples require maintenance.** VACUUM helps reclaim space for reuse.

8. **Autovacuum is an important operational mechanism.** Its effectiveness affects storage health and database maintenance.

9. **Indexes are separate storage structures.** Their size must be considered when evaluating total database consumption.

10. **TOAST manages oversized attributes.** It allows PostgreSQL to handle large variable-length values efficiently.

11. **Storage architecture should reflect workload requirements.** Transactional and analytical systems may benefit from different physical organizations.

12. **Physical implementation details should not become application dependencies.** File paths, page locations, and CTIDs are not permanent application interfaces.

---

## 20. Knowledge Check

Answer these questions without looking at the explanations above.

1. What is the difference between a logical table and its physical representation?
2. What is a database page?
3. What is the default PostgreSQL page size in a standard build?
4. What information does a page header contain?
5. What is a line pointer?
6. What is a heap tuple?
7. Why does PostgreSQL use tuple headers?
8. What does CTID represent?
9. Why should CTID not be used as a primary key?
10. What is MVCC?
11. Why can multiple versions of a row exist?
12. What is a dead tuple?
13. What is the purpose of VACUUM?
14. Why does ordinary VACUUM not necessarily reduce the physical file size?
15. What is the difference between VACUUM and VACUUM FULL?
16. What is autovacuum?
17. What does the Free Space Map track?
18. How does the Visibility Map help index-only scans?
19. What is TOAST?
20. Why are indexes measured separately from table storage?
21. What is a tablespace?
22. What is the difference between row-oriented and column-oriented storage?

---

## 21. Practical Exercises

### Exercise 1 — Physical inspection

Create a test table containing:

- A primary key.
- Several numeric columns.
- Variable-length text.
- A date column.

Then inspect:

- Its CTID values.
- Its relation path.
- Its relation size.
- Its total size.

Explain what each result tells you.

### Exercise 2 — Observe MVCC

1. Insert a record.
2. Inspect its CTID.
3. Update one of its columns.
4. Inspect its CTID again.
5. Explain why the physical location may have changed.
6. Explain what can happen to the previous tuple version.

### Exercise 3 — Storage measurement

Create a table with several indexes.

Compare:

```sql
pg_relation_size()
pg_table_size()
pg_indexes_size()
pg_total_relation_size()
```

Explain why the values are different.

### Exercise 4 — Maintenance analysis

Perform multiple updates and deletes on a test table.

Inspect:

```sql
pg_stat_user_tables
```

Then execute:

```sql
VACUUM (ANALYZE) schema_name.table_name;
```

Compare the statistics before and after maintenance.

Explain why the physical file size may remain unchanged.

### Exercise 5 — SIAPV architecture review

Choose one SIAPV entity, such as:

- Vehicles.
- Maintenance records.
- Meter readings.
- Insurance records.

Describe:

1. Its logical structure.
2. Its expected physical storage behavior.
3. Its likely insert and update frequency.
4. Its potential indexes.
5. Whether it may contain large values.
6. How its storage requirements may grow over time.
7. Which maintenance or monitoring mechanisms are relevant.

---

## 22. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Physical storage | Almacenamiento físico |
| Logical representation | Representación lógica |
| Database page | Página de base de datos |
| Database block | Bloque de base de datos |
| Heap | Estructura de almacenamiento de filas |
| Heap tuple | Tupla almacenada en una tabla |
| Tuple header | Encabezado de tupla |
| Line pointer | Puntero de línea |
| Page header | Encabezado de página |
| Free space | Espacio libre |
| Relation file | Archivo de relación |
| Storage fork | Rama o componente de almacenamiento |
| Free Space Map | Mapa de espacio libre |
| Visibility Map | Mapa de visibilidad |
| Dead tuple | Tupla obsoleta |
| Row version | Versión de una fila |
| MVCC | Control de concurrencia multiversión |
| Vacuum | Proceso de limpieza y recuperación de espacio |
| Autovacuum | Mantenimiento automático |
| Tablespace | Espacio de tablas |
| TOAST | Mecanismo de almacenamiento de atributos grandes |
| Row-oriented storage | Almacenamiento orientado a filas |
| Column-oriented storage | Almacenamiento orientado a columnas |
| Relation size | Tamaño de una relación |
| Tuple visibility | Visibilidad de una tupla |
| Page layout | Distribución interna de una página |
| Buffer cache | Caché de páginas en memoria |

---

## 23. Related Notes

### Previous concepts

- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[05 - Database Architecture Fundamentals]]
- [[03 - Keys and Constraints]]
- [[05 - Normalization]]

### Next concepts

- [[04 - Indexes]]
- [[05 - Transactions and ACID]]
- [[06 - Concurrency and Locks]]
- [[01 - PostgreSQL Architecture Overview]]
- [[03 - PostgreSQL Storage]]
- [[05 - PostgreSQL Performance Fundamentals]]

---

## 24. Final Understanding

A relational database is not simply a collection of tables saved in files.

It is a storage system that manages pages, tuple versions, metadata, indexes, memory buffers, and maintenance operations.

PostgreSQL uses these structures to provide transactional consistency, concurrent access, and efficient data retrieval.

Understanding physical storage makes it easier to explain:

- Why database size grows.
- Why updates create additional storage work.
- Why deleted records may remain physically present.
- Why indexes consume disk space.
- Why vacuum operations matter.
- Why memory caching influences performance.
- Why a database architect must understand both logical design and physical implementation.

The next step is to understand how PostgreSQL locates records efficiently instead of scanning entire tables.

**That leads directly to database indexes.**
