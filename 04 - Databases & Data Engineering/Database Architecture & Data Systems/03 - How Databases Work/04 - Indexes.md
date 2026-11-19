
---
title: "Indexes"
aliases:
  - "Database Indexes"
  - "Database Indexing"
  - "PostgreSQL Index Architecture"
type: learning-note
domain: technology
category: database-systems
subject: indexing-and-query-performance
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - database-indexes
  - indexing
  - query-performance
  - postgresql
  - btree
  - hash-index
  - query-optimization
  - sql

concepts:
  - database-index
  - index-structure
  - index-scan
  - sequential-scan
  - btree
  - hash-index
  - composite-index
  - unique-index
  - partial-index
  - expression-index
  - covering-index
  - index-only-scan
  - selectivity
  - cardinality
  - index-maintenance
  - query-planner

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Database Management Systems]]"
  - "[[02 - How a Query is Executed]]"
  - "[[03 - Storage Pages and Tables]]"
  - "[[03 - Keys and Constraints]]"

related:
  - "[[05 - Transactions and ACID]]"
  - "[[06 - Concurrency and Locks]]"
  - "[[02 - Database Scalability]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"
  - "[[02 - PostgreSQL Processes]]"

tools:
  - PostgreSQL
  - SQL
  - pgAdmin
  - psql
  - EXPLAIN
  - EXPLAIN ANALYZE

learning_objectives:
  - "Explain why database indexes exist and how they improve data retrieval."
  - "Distinguish sequential scans from index scans."
  - "Understand the internal structure of a B-tree index."
  - "Identify common PostgreSQL index types and their use cases."
  - "Create and manage indexes using SQL."
  - "Understand composite, unique, partial, and expression indexes."
  - "Explain index selectivity and its effect on query planning."
  - "Interpret basic index-related execution plans."
  - "Evaluate the performance costs and trade-offs of indexing."

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 04 — Indexes

## 1. Introduction

A database table can contain thousands, millions, or even billions of rows.

When an application needs to retrieve a specific record, the database must determine where that record is stored.

Consider this query:

```sql
SELECT *
FROM vehicles
WHERE plate_number = 'ABC-123';
```

If the table contains only ten records, checking every row may be inexpensive.

However, imagine a table containing ten million records.

Searching every row to find one vehicle can become inefficient.

This is where **database indexes** become important.

An index is an additional data structure that helps the database locate records more efficiently.

The central idea is:

> An index provides an organized access path to data without requiring the database to inspect every row in the table.

Indexes are essential for query performance, but they also introduce storage and maintenance costs.

---

## 2. What Is a Database Index?

A database index is a separate data structure associated with a table or another indexed relation.

It stores indexed values and references that allow the database engine to locate corresponding table records.

A simplified representation:

```text
Table: vehicles

vehicle_id | plate_number | brand
-----------|--------------|--------
1001       | ABC-123      | Nissan
1002       | DEF-456      | Ford
1003       | GHI-789      | Toyota
1004       | JKL-012      | Chevrolet
```

Suppose PostgreSQL creates an index on `plate_number`.

Conceptually:

```text
Index: vehicles_plate_number_idx

ABC-123 ───────> Row 1001
DEF-456 ───────> Row 1002
GHI-789 ───────> Row 1003
JKL-012 ───────> Row 1004
```

This is a simplified illustration.

Real indexes have their own internal structures, pages, metadata, and access methods.

### 2.1 Table vs. index

| Table | Index |
|---|---|
| Stores the table's row versions | Stores indexed values and references |
| Contains the actual table data | Provides an access path to the data |
| Usually organized as heap storage in PostgreSQL | Uses a specific index access method |
| Can contain many columns | Contains selected indexed expressions or columns |
| Maintained by database operations | Must be maintained when indexed data changes |

An index does not normally replace the table.

It complements the table.

### 2.2 Why are indexes separate structures?

Indexes have different organizational requirements from ordinary table storage.

A table needs to store complete records.

An index needs to organize selected values to support particular search operations.

For example:

- A B-tree organizes values for equality and range searches.
- A Hash index organizes values for equality comparisons.
- A GIN index supports searches within composite or multi-component values.

The access method determines how the index is organized and which operations it can accelerate.

---

## 3. Sequential Scan vs. Index Scan

One of the most important concepts in query performance is the difference between a sequential scan and an index scan.

### 3.1 Sequential scan

A sequential scan reads the table's pages and examines its tuples to find records that satisfy a condition.

Example:

```sql
SELECT *
FROM vehicles
WHERE plate_number = 'ABC-123';
```

Without a useful index, PostgreSQL may perform a sequential scan.

Conceptually:

```text
Table: vehicles

Page 0:
    Row 1 -> ABC-001
    Row 2 -> DEF-002
    Row 3 -> ABC-123  <-- Match

Page 1:
    Row 4 -> GHI-004
    Row 5 -> JKL-005

Page 2:
    Row 6 -> MNO-006
    Row 7 -> PQR-007
```

The database checks the relevant rows until it has processed the table.

A sequential scan is not necessarily bad.

It can be efficient when:

- The table is small.
- A large percentage of rows must be returned.
- The data is already cached.
- Reading pages sequentially is cheaper than following index references.

### 3.2 Index scan

An index scan uses an index to locate candidate table records.

Conceptually:

```text
SQL Query
    |
    v
Search the index
    |
    v
Find indexed value
    |
    v
Obtain tuple location
    |
    v
Access the corresponding heap page
    |
    v
Retrieve row
```

Instead of examining every table row, PostgreSQL can use the index to identify a smaller set of candidates.

### 3.3 Comparison

| Characteristic | Sequential scan | Index scan |
|---|---|---|
| Main access path | Table pages | Index followed by table access |
| Reads | Potentially many table pages | Index pages and selected heap pages |
| Small tables | Often efficient | May have unnecessary overhead |
| Selective conditions | May do unnecessary work | Often advantageous |
| Large result sets | Can be efficient | May involve many random heap accesses |
| Requires an index | No | Yes |

### 3.4 The optimizer chooses the access path

Creating an index does not force PostgreSQL to use it.

The query planner evaluates available access paths and estimates their costs.

For example:

```sql
EXPLAIN
SELECT *
FROM vehicles
WHERE plate_number = 'ABC-123';
```

The execution plan may contain:

```text
Seq Scan on vehicles
```

Or:

```text
Index Scan using vehicles_plate_number_idx on vehicles
```

The choice depends on factors such as:

- Table size.
- Index availability.
- Number of matching rows.
- Data distribution.
- Statistics.
- Estimated I/O cost.
- Memory configuration.
- Cost parameters.

**Important:** An index is an available access path, not a guarantee that a query will execute faster.

---

## 4. How B-tree Indexes Work

The most common PostgreSQL index type is the **B-tree**.

B-tree indexes are designed to support ordered searches.

They are particularly useful for:

- Equality comparisons.
- Range comparisons.
- Sorting.
- Minimum and maximum values.
- Prefix comparisons under compatible ordering rules.

### 4.1 What is a B-tree?

A B-tree is a balanced search tree.

It organizes keys across multiple levels.

A simplified example:

```text
                     [50]
                    /    \
                   /      \
              [20, 35]   [70, 90]
              /  |  \     /  |  \
             /   |   \   /   |   \
           Data Data Data Data Data
```

This diagram illustrates the general idea of hierarchical search.

PostgreSQL uses a B-tree implementation based on pages with a root, internal pages, and leaf pages.

The exact structure is more complex than this illustration.

### 4.2 Root page

The root page is the starting point for traversing the tree.

It contains information that helps direct the search toward the appropriate child page.

### 4.3 Internal pages

Internal pages contain separator keys and references to child pages.

They guide the search through the tree.

### 4.4 Leaf pages

Leaf pages contain indexed keys and references to corresponding table tuples.

They are the final level of the search path.

Conceptually:

```text
B-tree Index
│
├── Root Page
│
├── Internal Pages
│
└── Leaf Pages
    │
    ├── Indexed Key
    ├── Tuple Reference
    ├── Indexed Key
    └── Tuple Reference
```

PostgreSQL's B-tree implementation also maintains sibling links between pages at the same level, supporting efficient ordered traversal.

### 4.5 Searching a B-tree

Imagine searching for the value `75`.

```text
                  [50]
                 /    \
                /      \
          Values <50   Values >=50
                          |
                          v
                      [70, 90]
                      /  |  \
                     /   |   \
                    v    v    v
                  [60] [75] [95]
```

The database navigates through the tree rather than scanning every indexed value.

### 4.6 Why is a B-tree efficient?

A balanced tree keeps its height relatively small as the number of indexed entries grows.

Search operations therefore require traversing a limited number of levels rather than examining every entry.

B-tree operations generally have logarithmic search complexity under the usual theoretical model.

For a database, actual performance also depends on:

- Page size.
- Tree height.
- Cache behavior.
- I/O patterns.
- Key size.
- Index maintenance.
- Data distribution.

### 4.7 Range queries

B-tree indexes are particularly useful for range queries.

Example:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_date
      BETWEEN DATE '2026-01-01'
          AND DATE '2026-03-31';
```

A B-tree can navigate to the beginning of the requested range and scan the relevant leaf entries.

This is one reason B-tree indexes are useful for dates, numerical values, and ordered identifiers.

---

## 5. PostgreSQL Index Types

PostgreSQL supports multiple index access methods.

Each one is designed for different data structures and query operations.

### 5.1 B-tree

<badge color="success">General-purpose index</badge>

B-tree is the default index type.

Example:

```sql
CREATE INDEX idx_vehicles_plate
ON vehicles (plate_number);
```

Typical use cases:

- Equality searches.
- Range searches.
- Ordered retrieval.
- Sorting.
- Minimum and maximum values.

Common operators include:

```sql
=
<
<=
>
>=
BETWEEN
```

The actual supported operators depend on the relevant operator class and data type.

### 5.2 Hash

Hash indexes are designed for equality comparisons.

Example:

```sql
CREATE INDEX idx_vehicles_brand_hash
ON vehicles USING HASH (brand);
```

Typical operation:

```sql
SELECT *
FROM vehicles
WHERE brand = 'Nissan';
```

Hash indexes do not provide the ordered traversal capabilities of B-tree indexes.

They are not suitable for range searches or sorting.

B-tree is generally the preferred default unless there is a measured reason to use another access method.

### 5.3 GIN — Generalized Inverted Index

GIN is useful for values containing multiple searchable components.

Common applications include:

- Arrays.
- JSONB documents.
- Full-text search.

Example:

```sql
CREATE INDEX idx_documents_metadata
ON documents
USING GIN (metadata);
```

For a JSONB column:

```sql
SELECT *
FROM documents
WHERE metadata @> '{"category": "maintenance"}';
```

GIN indexes can help locate documents containing particular elements or key-value structures.

They can be especially useful when a single column contains multiple independently searchable values.

### 5.4 GiST — Generalized Search Tree

GiST provides a flexible framework for implementing different search structures.

It is used for operations involving:

- Geometric data.
- Spatial searches.
- Range types.
- Specialized operator classes.

Example using a geometric data type:

```sql
CREATE INDEX idx_locations_point
ON locations
USING GIST (coordinates);
```

The usefulness of a GiST index depends on the data type and available operators.

### 5.5 SP-GiST — Space-Partitioned GiST

SP-GiST supports search structures based on space partitioning.

It can be useful for data with particular hierarchical or partitionable structures.

Potential applications include:

- Spatial data.
- Tries.
- Certain geometric and multidimensional structures.

Example:

```sql
CREATE INDEX idx_locations_point_spgist
ON locations
USING SPGIST (coordinates);
```

Whether this index is appropriate depends on the data type and workload.

### 5.6 BRIN — Block Range Index

BRIN stands for **Block Range Index**.

It stores summarized information about ranges of table pages.

BRIN indexes are useful for very large tables where indexed values correlate with the physical order of rows.

Typical examples include:

- Event timestamps.
- Historical measurements.
- Sequentially inserted records.
- Large append-oriented datasets.

Example:

```sql
CREATE INDEX idx_events_created_at_brin
ON events
USING BRIN (created_at);
```

A BRIN index is generally much smaller than a B-tree index for suitable workloads.

However, it is less precise.

It identifies candidate page ranges that may contain matching records, and PostgreSQL must inspect those pages to find the actual tuples.

### 5.7 Index type comparison

| Index type | Main purpose | Common use |
|---|---|---|
| B-tree | Ordered searches | IDs, dates, numbers, text |
| Hash | Equality searches | Equality comparisons |
| GIN | Multiple searchable components | Arrays, JSONB, full-text search |
| GiST | Flexible search structures | Spatial and range operations |
| SP-GiST | Space-partitioned search | Specialized spatial structures |
| BRIN | Summarized page ranges | Very large correlated tables |

### 5.8 Choosing an index type

The correct index depends on:

1. Data type.
2. Query operators.
3. Table size.
4. Data distribution.
5. Physical data organization.
6. Insert and update frequency.
7. Storage constraints.
8. Measured execution performance.

Do not choose an index type simply because it sounds more advanced.

Start with the workload and the query operators.

---

## 6. Single-Column Indexes

A single-column index uses one column as its indexed key.

Example:

```sql
CREATE INDEX idx_vehicles_plate_number
ON vehicles (plate_number);
```

It can help with:

```sql
SELECT *
FROM vehicles
WHERE plate_number = 'ABC-123';
```

It may also help with:

```sql
SELECT *
FROM vehicles
ORDER BY plate_number;
```

The planner determines whether the index is beneficial.

### 6.1 When to consider a single-column index

A column may be a good candidate when:

- It appears frequently in selective WHERE conditions.
- It is frequently used in JOIN conditions.
- It is used for sorting or grouping.
- Its values allow the index to narrow down the search efficiently.

Not every column needs an index.

---

## 7. Composite Indexes

A composite index contains more than one indexed column.

Example:

```sql
CREATE INDEX idx_maintenance_vehicle_date
ON maintenance_records (
    vehicle_id,
    maintenance_date
);
```

This index organizes entries using both columns.

### 7.1 Why column order matters

For a B-tree composite index, the order of the indexed columns is important.

Consider:

```sql
CREATE INDEX idx_maintenance_vehicle_date
ON maintenance_records (
    vehicle_id,
    maintenance_date
);
```

This index is particularly useful for queries such as:

```sql
SELECT *
FROM maintenance_records
WHERE vehicle_id = 1001;
```

It can also help with:

```sql
SELECT *
FROM maintenance_records
WHERE vehicle_id = 1001
  AND maintenance_date >= DATE '2026-01-01';
```

The first column provides the leading part of the index search.

### 7.2 Leading-column principle

A composite B-tree index is generally most useful when queries constrain its leading columns.

Consider:

```sql
CREATE INDEX idx_vehicle_date_type
ON maintenance_records (
    vehicle_id,
    maintenance_date,
    maintenance_type
);
```

Conceptually:

| Indexed column | Position |
|---|---:|
| vehicle_id | 1 |
| maintenance_date | 2 |
| maintenance_type | 3 |

Queries filtering by `vehicle_id` can use the leading index column.

Queries filtering by both `vehicle_id` and `maintenance_date` can use the first two columns.

Queries filtering only by `maintenance_type` generally cannot use the leading portion of this index as efficiently.

Modern PostgreSQL can sometimes use skip scan optimizations for certain multicolumn B-tree situations, but this should not replace thoughtful index ordering.

### 7.3 Example of different column orders

Index A:

```sql
CREATE INDEX idx_a
ON maintenance_records (
    vehicle_id,
    maintenance_date
);
```

Index B:

```sql
CREATE INDEX idx_b
ON maintenance_records (
    maintenance_date,
    vehicle_id
);
```

These indexes support different access patterns.

| Query pattern | Potentially better leading column |
|---|---|
| Filter by vehicle | `vehicle_id` |
| Filter by date range | `maintenance_date` |
| Filter by vehicle and date | Depends on selectivity and query pattern |
| Filter only by maintenance date | `maintenance_date` |

The correct design should be based on actual query patterns and measurements.

### 7.4 Important principle

Do not create every possible column combination.

Each additional index consumes storage and increases write-maintenance work.

---

## 8. Unique Indexes

A unique index ensures that indexed key values are unique according to the index's uniqueness rules.

Example:

```sql
CREATE UNIQUE INDEX idx_vehicles_unique_plate
ON vehicles (plate_number);
```

This prevents duplicate non-NULL plate values.

For a normal PostgreSQL unique index, multiple NULL values are permitted by default because NULL values are treated as distinct for uniqueness purposes.

PostgreSQL also supports `NULLS NOT DISTINCT` for unique indexes in supported versions when NULL values should be treated as equal for uniqueness.

### 8.1 Primary keys and unique indexes

A primary key automatically creates a unique index to enforce uniqueness.

Example:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT PRIMARY KEY,
    plate_number VARCHAR(15) UNIQUE
);
```

PostgreSQL creates the required unique indexes for these constraints.

Therefore, you generally should not create a duplicate unique index on the same primary-key column.

### 8.2 Unique constraint vs. unique index

A unique constraint expresses a logical integrity rule.

A unique index is the physical structure used to enforce uniqueness.

For ordinary cases, prefer declaring the business rule as a constraint:

```sql
ALTER TABLE vehicles
ADD CONSTRAINT vehicles_plate_number_unique
UNIQUE (plate_number);
```

PostgreSQL creates the supporting unique index.

---

## 9. Partial Indexes

A partial index includes only rows that satisfy a specified condition.

Example:

```sql
CREATE INDEX idx_active_vehicles_plate
ON vehicles (plate_number)
WHERE operational_status = 'ACTIVE';
```

This index contains entries only for active vehicles.

### 9.1 Why use partial indexes?

Imagine a fleet table containing:

- 200 active vehicles.
- 1,500 inactive or retired records.

If applications frequently search only active vehicles, a partial index may be smaller and more efficient than indexing every record.

Example:

```sql
SELECT *
FROM vehicles
WHERE operational_status = 'ACTIVE'
  AND plate_number = 'ABC-123';
```

The planner may use the partial index when it can establish that the query condition implies the index predicate.

### 9.2 Partial index considerations

A partial index is useful when:

- A meaningful subset of records is frequently queried.
- The predicate is stable enough to provide value.
- The indexed subset is significantly smaller.
- Query predicates are compatible with the index condition.

It is not appropriate when queries regularly need records outside the indexed subset.

---

## 10. Expression Indexes

An expression index indexes the result of an expression rather than only a direct column reference.

Example:

```sql
CREATE INDEX idx_employees_lower_email
ON employees (
    LOWER(email)
);
```

This can support:

```sql
SELECT *
FROM employees
WHERE LOWER(email) = LOWER('USER@EXAMPLE.COM');
```

Without an appropriate index, PostgreSQL may need to evaluate the expression across many rows.

### 10.1 Common use cases

Expression indexes can help with:

- Case-insensitive searches.
- Normalized values.
- Computed search keys.
- Frequently used deterministic expressions.

The query expression must match the indexed expression closely enough for the planner to recognize its applicability.

### 10.2 Design consideration

An expression index adds maintenance cost because PostgreSQL must calculate and maintain the indexed expression when relevant rows change.

Use it when the query pattern justifies the additional cost.

---

## 11. Covering Indexes and Index-Only Scans

A covering index contains the information required to satisfy a particular query without retrieving additional column values from the heap.

PostgreSQL supports this pattern through B-tree indexes and the `INCLUDE` clause.

Example:

```sql
CREATE INDEX idx_vehicles_plate_covering
ON vehicles (plate_number)
INCLUDE (brand, model);
```

Consider:

```sql
SELECT
    plate_number,
    brand,
    model
FROM vehicles
WHERE plate_number = 'ABC-123';
```

The index contains:

- The search key: `plate_number`.
- Additional included values: `brand`, `model`.

### 11.1 What is an index-only scan?

An index-only scan is an execution strategy where PostgreSQL can retrieve the required column values from the index without accessing the heap for every matching tuple.

Example plan:

```text
Index Only Scan using idx_vehicles_plate_covering
```

### 11.2 The Visibility Map connection

Even if all requested columns exist in the index, PostgreSQL must still ensure that the row version is visible to the current transaction.

The Visibility Map helps with this.

When a heap page is marked all-visible, PostgreSQL can often avoid visiting that heap page to check tuple visibility.

If the visibility information is insufficient, heap access may still be necessary.

### 11.3 Why INCLUDE is useful

The `INCLUDE` clause adds columns to the index payload without making them part of the search key.

For example:

```sql
CREATE INDEX idx_maintenance_vehicle_covering
ON maintenance_records (vehicle_id)
INCLUDE (
    maintenance_date,
    maintenance_type
);
```

This can help certain queries that search by vehicle and retrieve the included values.

### 11.4 Trade-offs

Covering indexes can:

- Reduce heap accesses.
- Improve some read-heavy queries.
- Increase index size.
- Increase write-maintenance overhead.

They should be designed around measured query patterns.

---

## 12. Index Selectivity and Cardinality

An important factor in index design is **selectivity**.

Selectivity describes how effectively a condition narrows down the number of matching rows.

### 12.1 High selectivity

Consider a table containing one million vehicles.

A query searches for a unique inventory identifier.

```sql
SELECT *
FROM vehicles
WHERE inventory_number = 'INV-001250';
```

If the condition returns one row out of one million, it is highly selective.

### 12.2 Low selectivity

Now consider:

```sql
SELECT *
FROM vehicles
WHERE operational_status = 'ACTIVE';
```

If 90% of the records are active, the condition returns a large portion of the table.

This is less selective.

### 12.3 Selectivity example

| Condition | Rows returned | Selectivity |
|---|---:|---|
| Unique inventory number | 1 / 1,000,000 | Very high |
| Vehicle brand | 100,000 / 1,000,000 | Moderate |
| Operational status | 900,000 / 1,000,000 | Low |

These figures are illustrative.

### 12.4 Why selectivity matters

An index is often more useful when it reduces the number of rows that must be accessed.

If a query returns most of the table, using an index may be more expensive than performing a sequential scan.

This explains why an index can exist but remain unused by the query planner.

### 12.5 Cardinality

In database discussions, cardinality can refer to different concepts depending on context.

For indexing and query planning, it commonly describes the number of distinct values in a column or expression.

For example:

| Column | Approximate distinct values |
|---|---:|
| vehicle_id | 1,000,000 |
| brand | 25 |
| operational_status | 4 |

A column with many distinct values often has higher cardinality.

High cardinality can make a column useful for selective searches, but it does not automatically guarantee that an index will be beneficial.

The actual data distribution and query conditions matter.

---

## 13. Index Costs and Trade-Offs

Indexes improve certain read operations, but they are not free.

Every index creates additional work.

### 13.1 Storage overhead

An index occupies disk space.

A table with multiple indexes may consume significantly more storage than its heap data alone.

For example:

```text
vehicles
│
├── Heap table
│
├── Primary-key index
│
├── Plate-number index
│
├── Brand index
│
└── Composite index
```

Each structure must be maintained.

### 13.2 Insert overhead

When a new record is inserted, PostgreSQL may need to add entries to several indexes.

The more indexes a table has, the more index-maintenance work can be required.

### 13.3 Update overhead

If an indexed column changes, the relevant index structures must be maintained.

Updates to non-indexed columns may also have storage consequences, depending on the update and whether PostgreSQL can use a HOT update.

### 13.4 Delete overhead

Deleting a row affects the visibility and eventual cleanup of corresponding index entries and heap tuples.

Vacuum maintenance helps remove obsolete index entries when appropriate.

### 13.5 Maintenance overhead

Indexes may require:

- Disk space.
- Buffer-cache space.
- Vacuum maintenance.
- Rebuilding or reorganization in some situations.
- Monitoring.

### 13.6 The balance

| Too few indexes | Too many indexes |
|---|---|
| More table scans | More storage consumption |
| Potentially slower reads | More insert overhead |
| Expensive filtering | More update overhead |
| More work for joins | More maintenance |

The objective is not to maximize the number of indexes.

The objective is to provide efficient access paths for important queries while keeping write and storage costs acceptable.

---

## 14. Indexes and Foreign Keys

A foreign key enforces referential integrity.

However, PostgreSQL does not automatically create an index on the referencing columns of a foreign key.

Consider:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT PRIMARY KEY
);

CREATE TABLE maintenance_records (
    maintenance_id BIGINT PRIMARY KEY,
    vehicle_id BIGINT NOT NULL,
    maintenance_date DATE NOT NULL,

    CONSTRAINT fk_maintenance_vehicle
        FOREIGN KEY (vehicle_id)
        REFERENCES vehicles (vehicle_id)
);
```

The referenced primary key has a supporting unique index.

However, PostgreSQL does not automatically create an index on:

```sql
maintenance_records.vehicle_id
```

You may need:

```sql
CREATE INDEX idx_maintenance_vehicle
ON maintenance_records (vehicle_id);
```

### 14.1 Why might this matter?

Consider:

```sql
SELECT *
FROM maintenance_records
WHERE vehicle_id = 1001;
```

An index can help retrieve maintenance records for a particular vehicle.

It can also improve certain operations involving updates or deletes on the referenced table, because PostgreSQL may need to check referencing rows.

### 14.2 Important design rule

Foreign-key columns are common index candidates, but not every foreign key requires a separate index.

Evaluate:

- Query frequency.
- Number of referencing records.
- Join patterns.
- Delete and update behavior.
- Existing composite indexes.

A composite index may already support the required access pattern.

---

## 15. Reading Execution Plans

Indexes should be evaluated using execution plans rather than assumptions.

PostgreSQL provides:

```sql
EXPLAIN
```

and:

```sql
EXPLAIN ANALYZE
```

### 15.1 EXPLAIN

Displays the planner's estimated execution plan.

Example:

```sql
EXPLAIN
SELECT *
FROM vehicles
WHERE plate_number = 'ABC-123';
```

Possible output:

```text
Index Scan using idx_vehicles_plate_number
on vehicles
```

This indicates that the planner selected an index scan.

### 15.2 EXPLAIN ANALYZE

Executes the query and reports actual execution information.

```sql
EXPLAIN ANALYZE
SELECT *
FROM vehicles
WHERE plate_number = 'ABC-123';
```

It provides information such as:

- Actual execution time.
- Actual rows returned.
- Number of loops.
- Estimated rows.
- Actual rows.
- Planning time.
- Execution time.

### 15.3 Example comparison

Sequential scan:

```text
Seq Scan on vehicles
  Filter: (plate_number = 'ABC-123')
  Rows Removed by Filter: 9999
```

Index scan:

```text
Index Scan using idx_vehicles_plate_number
  Index Cond: (plate_number = 'ABC-123')
```

These are illustrative fragments, not guaranteed output.

### 15.4 What should you evaluate?

When comparing plans, inspect:

| Element | What to examine |
|---|---|
| Scan type | Sequential, index, bitmap, index-only |
| Estimated rows | Planner estimate |
| Actual rows | Rows actually processed or returned at the node |
| Cost | Estimated planner cost |
| Execution time | Measured execution time |
| Buffers | Memory and disk block activity |
| Rows removed | Rows rejected by filters |

Do not evaluate an index only by whether the plan contains the word `Index`.

The full execution behavior matters.

### 15.5 A warning about EXPLAIN ANALYZE

`EXPLAIN ANALYZE` executes the statement.

For a `SELECT`, it runs the read query.

For a data-modifying statement, it actually performs the modification unless you use an appropriate transaction and roll it back.

Use caution when analyzing production data.

---

## 16. Creating, Inspecting, and Removing Indexes

### 16.1 Create an index

```sql
CREATE INDEX idx_vehicles_brand
ON vehicles (brand);
```

### 16.2 Create a unique index

```sql
CREATE UNIQUE INDEX idx_vehicles_plate_unique
ON vehicles (plate_number);
```

Only do this if duplicate values are not allowed and existing data satisfies the uniqueness requirement.

### 16.3 Create a composite index

```sql
CREATE INDEX idx_maintenance_vehicle_date
ON maintenance_records (
    vehicle_id,
    maintenance_date
);
```

### 16.4 Create a partial index

```sql
CREATE INDEX idx_active_vehicles
ON vehicles (plate_number)
WHERE operational_status = 'ACTIVE';
```

### 16.5 Create an expression index

```sql
CREATE INDEX idx_employees_lower_email
ON employees (LOWER(email));
```

### 16.6 Create an index concurrently

For an existing table that must remain available for ordinary writes during index creation, PostgreSQL supports:

```sql
CREATE INDEX CONCURRENTLY idx_vehicles_model
ON vehicles (model);
```

This avoids taking a lock that blocks normal inserts, updates, and deletes for the duration of the build in the same way as a regular index build.

However:

- It takes longer.
- It performs additional work.
- It has restrictions.
- It cannot run inside a transaction block.
- A failed concurrent build can leave an invalid index that requires investigation.

This is an important operational tool for production database maintenance.

### 16.7 Inspect indexes

Use the system catalog view:

```sql
SELECT
    schemaname,
    tablename,
    indexname,
    indexdef
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename, indexname;
```

### 16.8 Inspect index sizes

```sql
SELECT
    indexrelname AS index_name,
    pg_size_pretty(
        pg_relation_size(indexrelid)
    ) AS index_size
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;
```

### 16.9 Drop an index

```sql
DROP INDEX idx_vehicles_brand;
```

For a production environment where concurrent index removal is appropriate:

```sql
DROP INDEX CONCURRENTLY idx_vehicles_brand;
```

The concurrent form has restrictions and cannot run inside a transaction block.

Do not remove indexes without understanding their purpose and dependencies.

---

## 17. Practical Lab — Building and Evaluating Indexes

This laboratory demonstrates how different indexes affect PostgreSQL query plans.

Use a development database or a disposable environment.

### 17.1 Create a schema

```sql
CREATE SCHEMA IF NOT EXISTS indexing_lab;
```

### 17.2 Create a sample table

```sql
CREATE TABLE indexing_lab.vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    inventory_number VARCHAR(30) NOT NULL,
    plate_number VARCHAR(15),
    brand VARCHAR(50),
    model VARCHAR(50),
    operational_status VARCHAR(20),
    manufacture_year INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 17.3 Generate sample records

```sql
INSERT INTO indexing_lab.vehicles (
    inventory_number,
    plate_number,
    brand,
    model,
    operational_status,
    manufacture_year
)
SELECT
    'INV-' || LPAD(g::TEXT, 8, '0'),
    'PLT-' || LPAD(g::TEXT, 7, '0'),
    CASE
        WHEN g % 5 = 0 THEN 'Nissan'
        WHEN g % 5 = 1 THEN 'Ford'
        WHEN g % 5 = 2 THEN 'Toyota'
        WHEN g % 5 = 3 THEN 'Chevrolet'
        ELSE 'Volkswagen'
    END,
    CASE
        WHEN g % 3 = 0 THEN 'Frontier'
        WHEN g % 3 = 1 THEN 'Ranger'
        ELSE 'Hilux'
    END,
    CASE
        WHEN g % 10 = 0 THEN 'INACTIVE'
        ELSE 'ACTIVE'
    END,
    2015 + (g % 12)
FROM generate_series(1, 100000) AS g;
```

This creates 100,000 sample records.

### 17.4 Update planner statistics

```sql
ANALYZE indexing_lab.vehicles;
```

The planner uses statistics to estimate query costs and row counts.

### 17.5 Examine a query without a secondary index

```sql
EXPLAIN ANALYZE
SELECT *
FROM indexing_lab.vehicles
WHERE plate_number = 'PLT-0005000';
```

The primary key has an index, but `plate_number` does not yet have a dedicated index.

Observe the execution plan.

### 17.6 Create a B-tree index

```sql
CREATE INDEX idx_lab_vehicles_plate
ON indexing_lab.vehicles (plate_number);
```

Run the same query:

```sql
EXPLAIN ANALYZE
SELECT *
FROM indexing_lab.vehicles
WHERE plate_number = 'PLT-0005000';
```

Compare:

- Scan type.
- Estimated cost.
- Actual rows.
- Execution time.
- Buffer activity, if included.

For this small and controlled example, the planner may still choose a sequential scan under some conditions. Do not assume an index must always be selected.

### 17.7 Create a composite index

```sql
CREATE INDEX idx_lab_vehicles_status_year
ON indexing_lab.vehicles (
    operational_status,
    manufacture_year
);
```

Test:

```sql
EXPLAIN ANALYZE
SELECT *
FROM indexing_lab.vehicles
WHERE operational_status = 'ACTIVE'
  AND manufacture_year = 2020;
```

Then test:

```sql
EXPLAIN ANALYZE
SELECT *
FROM indexing_lab.vehicles
WHERE manufacture_year = 2020;
```

Compare the plans and explain the significance of index column order.

### 17.8 Create a partial index

```sql
CREATE INDEX idx_lab_active_vehicles
ON indexing_lab.vehicles (inventory_number)
WHERE operational_status = 'ACTIVE';
```

Test:

```sql
EXPLAIN ANALYZE
SELECT *
FROM indexing_lab.vehicles
WHERE operational_status = 'ACTIVE'
  AND inventory_number = 'INV-00001000';
```

Observe whether the planner considers the partial index.

### 17.9 Create an expression index

```sql
CREATE INDEX idx_lab_vehicles_brand_lower
ON indexing_lab.vehicles (
    LOWER(brand)
);
```

Test:

```sql
EXPLAIN ANALYZE
SELECT *
FROM indexing_lab.vehicles
WHERE LOWER(brand) = 'nissan';
```

Compare the plan with and without the expression index.

### 17.10 Inspect index storage

```sql
SELECT
    indexrelname AS index_name,
    pg_size_pretty(
        pg_relation_size(indexrelid)
    ) AS index_size
FROM pg_stat_user_indexes
WHERE schemaname = 'indexing_lab'
ORDER BY pg_relation_size(indexrelid) DESC;
```

Observe how each additional index contributes to storage consumption.

### 17.11 Inspect table and total storage

```sql
SELECT
    pg_size_pretty(
        pg_relation_size(
            'indexing_lab.vehicles'
        )
    ) AS heap_size,

    pg_size_pretty(
        pg_indexes_size(
            'indexing_lab.vehicles'
        )
    ) AS indexes_size,

    pg_size_pretty(
        pg_total_relation_size(
            'indexing_lab.vehicles'
        )
    ) AS total_size;
```

### 17.12 Clean up

```sql
DROP SCHEMA indexing_lab CASCADE;
```

Execute this only after completing the exercises and saving any results you want to keep.

---

## 18. Common Indexing Mistakes

| Mistake | Why it is problematic |
|---|---|
| Creating an index on every column | Increases storage and write overhead |
| Assuming an index is always faster | Sequential scans may be more efficient |
| Ignoring composite index order | Queries may not benefit from the intended access path |
| Duplicating existing indexes | Adds unnecessary maintenance |
| Indexing low-selectivity columns without analysis | May provide little benefit |
| Ignoring foreign-key access patterns | Joins and referential checks may become expensive |
| Creating indexes without measuring queries | Optimizes assumptions rather than actual workloads |
| Forgetting index maintenance | Storage and performance can degrade |
| Using the wrong index type | The structure may not support the required operators |
| Ignoring write frequency | Read improvements may come at an unacceptable write cost |

---

## 19. Applying Indexes to SIAPV

Consider the SIAPV vehicle and machinery management application.

Its database contains operational information that users need to retrieve through searches, reports, and administrative workflows.

Possible tables include:

```text
vehicles
maintenance_records
meter_readings
insurance_records
accidents
operators
workshops
```

### 19.1 Vehicle inventory searches

A user may search for a unit by its unique inventory number.

```sql
SELECT *
FROM vehicles
WHERE inventory_number = 'INV-000123';
```

A B-tree index can support this search.

If `inventory_number` is a candidate key and duplicates are not permitted, a unique constraint may be appropriate.

### 19.2 License plate searches

```sql
SELECT *
FROM vehicles
WHERE plate_number = 'ABC-123';
```

A B-tree index may help retrieve the corresponding unit.

If plates can be NULL or change over time, the business rules should determine whether uniqueness is appropriate.

### 19.3 Maintenance history

A common query may retrieve maintenance events for a specific unit, ordered by date.

```sql
SELECT *
FROM maintenance_records
WHERE vehicle_id = 1001
ORDER BY maintenance_date DESC;
```

A possible index is:

```sql
CREATE INDEX idx_maintenance_vehicle_date
ON maintenance_records (
    vehicle_id,
    maintenance_date DESC
);
```

The usefulness of the index depends on the actual query patterns and data distribution.

### 19.4 Historical meter readings

For reports involving readings over a date range:

```sql
SELECT *
FROM meter_readings
WHERE reading_date >= DATE '2026-01-01'
  AND reading_date < DATE '2027-01-01';
```

A B-tree index on `reading_date` may be appropriate.

For very large historical tables where dates correlate with physical row order, BRIN may also be worth evaluating.

### 19.5 Insurance and operational status

A query may frequently retrieve currently active insurance records.

```sql
SELECT *
FROM insurance_records
WHERE status = 'ACTIVE';
```

A partial index could be considered if the workload frequently searches a small active subset and the query predicate matches the index condition.

### 19.6 Index design questions

For every potential SIAPV index, ask:

1. Which query needs this index?
2. How frequently is the query executed?
3. How many rows are expected to match?
4. Is the condition selective?
5. Is the column used in a join?
6. Is the column used for sorting?
7. Does another index already support the query?
8. How often does the indexed data change?
9. What is the index's storage cost?
10. Does `EXPLAIN ANALYZE` demonstrate a measurable benefit?

This is the difference between creating indexes mechanically and designing an indexing strategy.

---

## 20. Key Architectural Principles

1. **An index is a separate access structure.** It helps the DBMS locate records without requiring a full table scan in every situation.

2. **B-tree is the general-purpose default.** It supports equality, range, and ordered operations.

3. **Different index types solve different problems.** GIN, GiST, SP-GiST, BRIN, and Hash have distinct use cases.

4. **The query planner chooses the access path.** Creating an index does not guarantee its use.

5. **Selectivity matters.** Indexes are often most useful when they substantially reduce the number of candidate rows.

6. **Composite index order matters.** The leading columns influence which query patterns can use the index efficiently.

7. **Indexes have costs.** They consume storage and require maintenance during data modifications.

8. **Foreign keys do not automatically create indexes on referencing columns in PostgreSQL.** Evaluate these columns as potential index candidates.

9. **Covering indexes can reduce heap access.** Index-only scans also depend on visibility information.

10. **Measure before optimizing.** Use `EXPLAIN`, `EXPLAIN ANALYZE`, and storage statistics.

11. **Avoid redundant indexes.** More indexes do not automatically mean better performance.

12. **Index design is workload design.** A good strategy reflects the application's real read, write, and reporting patterns.

---

## 21. Knowledge Check

Answer these questions without looking at the explanations above.

1. What is a database index?
2. Why are indexes stored separately from ordinary heap tables?
3. What is the difference between a sequential scan and an index scan?
4. Why can a sequential scan be faster than an index scan?
5. What is a B-tree?
6. What are root, internal, and leaf pages?
7. Why is a B-tree useful for range queries?
8. What is the main difference between B-tree and Hash indexes?
9. When is a GIN index useful?
10. What types of workloads may benefit from BRIN?
11. What is a composite index?
12. Why does column order matter in a composite index?
13. What is a partial index?
14. What is an expression index?
15. What is the purpose of the INCLUDE clause?
16. What is an index-only scan?
17. What is selectivity?
18. What does cardinality mean in indexing discussions?
19. Why can an index increase the cost of INSERT operations?
20. Does PostgreSQL automatically index foreign-key referencing columns?
21. What is the difference between a unique constraint and a unique index?
22. What does EXPLAIN ANALYZE provide?
23. Why should index design be based on actual query workloads?
24. What does CREATE INDEX CONCURRENTLY accomplish?
25. Why should redundant indexes be avoided?

---

## 22. Practical Exercises

### Exercise 1 — Sequential vs. index scan

Create a table with at least 100,000 records.

Execute a selective query before and after creating an index.

Compare the execution plans.

Explain the differences.

### Exercise 2 — Composite index design

Create a table containing:

- vehicle_id
- maintenance_date
- maintenance_type
- estimated_cost

Create a composite index using:

```sql
(vehicle_id, maintenance_date)
```

Test queries that filter by:

1. vehicle_id.
2. maintenance_date.
3. vehicle_id and maintenance_date.
4. maintenance_type only.

Explain which queries may benefit and why.

### Exercise 3 — Index type selection

For each workload, identify a possible index type and justify your choice:

- Searching for an inventory number.
- Searching for JSONB document attributes.
- Filtering a large historical table by date.
- Performing spatial searches.
- Searching for exact equality on a value.

### Exercise 4 — Index cost

Create several indexes on a test table.

Measure:

- Heap size.
- Index size.
- Total relation size.

Insert and update additional records.

Explain why the indexing strategy introduces additional write work.

### Exercise 5 — SIAPV indexing proposal

Choose three SIAPV query patterns.

For each one, document:

1. The SQL query.
2. The table involved.
3. The columns used in filtering.
4. The expected selectivity.
5. The proposed index.
6. The reason for choosing that index type.
7. The expected read benefit.
8. The potential write and storage cost.
9. The execution plan before and after indexing.

Your objective is to justify an indexing decision, not simply to create an index.

---

## 23. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Database index | Índice de base de datos |
| Indexing | Indexación |
| Index scan | Exploración mediante índice |
| Sequential scan | Exploración secuencial |
| B-tree | Árbol B |
| Root page | Página raíz |
| Internal page | Página interna |
| Leaf page | Página hoja |
| Search key | Clave de búsqueda |
| Composite index | Índice compuesto |
| Unique index | Índice único |
| Partial index | Índice parcial |
| Expression index | Índice de expresión |
| Covering index | Índice que cubre una consulta |
| Index-only scan | Exploración únicamente mediante índice |
| Selectivity | Selectividad |
| Cardinality | Cardinalidad |
| Query planner | Planificador de consultas |
| Query optimizer | Optimizador de consultas |
| Access path | Ruta de acceso |
| Index maintenance | Mantenimiento de índices |
| Index overhead | Sobrecosto de los índices |
| Range query | Consulta por rango |
| Equality comparison | Comparación de igualdad |
| Foreign key | Clave foránea |
| Included column | Columna incluida |
| Concurrent index creation | Creación concurrente de índices |
| Execution plan | Plan de ejecución |
| Heap access | Acceso a la tabla heap |
| Index fragmentation | Fragmentación de índices |

---

## 24. Related Notes

### Previous concepts

- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[03 - Storage Pages and Tables]]
- [[03 - Keys and Constraints]]
- [[05 - Normalization]]

### Next concepts

- [[05 - Transactions and ACID]]
- [[06 - Concurrency and Locks]]
- [[02 - Database Scalability]]
- [[01 - PostgreSQL Architecture Overview]]
- [[05 - PostgreSQL Performance Fundamentals]]

---

## 25. Final Understanding

Indexes are fundamental structures in database performance.

They allow PostgreSQL to find records through organized access paths instead of always scanning every row.

However, an index is not automatically beneficial.

Every index introduces additional storage, write operations, and maintenance requirements.

A database architect must understand the relationship between:

- Query patterns.
- Data distribution.
- Selectivity.
- Index structures.
- Execution plans.
- Read performance.
- Write performance.
- Storage consumption.

The objective is to design an indexing strategy that supports the actual workload.

**The next step is understanding how databases guarantee reliable changes to stored data.**

That leads to transactions and the ACID properties.
