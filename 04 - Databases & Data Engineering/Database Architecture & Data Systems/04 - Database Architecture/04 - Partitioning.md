
---
title: "Database Partitioning"
aliases:
  - "Table Partitioning"
  - "PostgreSQL Partitioning"
  - "Partitioned Tables"
type: learning-note
domain: technology
category: database-architecture
subject: database-partitioning
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - partitioning
  - postgresql
  - database-performance
  - data-management
  - scalability
  - partition-pruning
  - range-partitioning
  - list-partitioning
  - hash-partitioning

concepts:
  - database-partitioning
  - partitioned-table
  - partition-key
  - partition-bound
  - range-partitioning
  - list-partitioning
  - hash-partitioning
  - partition-pruning
  - partition-constraint
  - partition-maintenance
  - partition-index
  - partitioned-index
  - partition-management

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Client Server Architecture]]"
  - "[[02 - Database Scalability]]"
  - "[[03 - Replication]]"
  - "[[04 - Indexes]]"
  - "[[03 - Storage Pages and Tables]]"

related:
  - "[[05 - Sharding]]"
  - "[[06 - High Availability]]"
  - "[[07 - Backup and Disaster Recovery]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"
  - "[[06 - Schema Design Principles]]"
  - "[[05 - Normalization]]"

tools:
  - PostgreSQL
  - SQL
  - psql
  - pgAdmin
  - Docker
  - EXPLAIN
  - EXPLAIN ANALYZE

learning_objectives:
  - "Explain database partitioning and its architectural purpose."
  - "Distinguish partitioning from replication and sharding."
  - "Understand partitioned tables and partition keys."
  - "Compare range, list, and hash partitioning."
  - "Understand partition pruning and query planning."
  - "Create and manage partitioned tables in PostgreSQL."
  - "Evaluate partitioning strategies and their limitations."
  - "Recognize when partitioning is useful in an operational database."

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 04 — Database Partitioning

## 1. What Is Database Partitioning?

**Database partitioning** is a technique that divides a large table into smaller, more manageable parts called *partitions*.

Instead of storing every row in one physical table structure, the database distributes rows across multiple partitions according to a defined rule.

From the application's perspective, a partitioned table can still behave like one logical table.

Consider a table containing millions of vehicle maintenance records.

Without partitioning:

```text
MAINTENANCE_RECORDS
|
+-- Record 1
+-- Record 2
+-- Record 3
+-- ...
+-- Record 5,000,000
```

With partitioning:

```text
MAINTENANCE_RECORDS
|
+-- MAINTENANCE_2024
|
+-- MAINTENANCE_2025
|
+-- MAINTENANCE_2026
|
+-- MAINTENANCE_2027
```

Each partition contains only the records that satisfy its partitioning rule.

For example, the database may store records according to their maintenance date.

### 1.1 Why Is Partitioning Used?

Partitioning can help with:

- Managing very large tables.
- Improving certain query patterns.
- Reducing the amount of data scanned by queries.
- Simplifying data retention.
- Organizing historical records.
- Managing indexes on large datasets.
- Moving or removing groups of records efficiently.

However, partitioning is not automatically a performance optimization.

It is useful when the partitioning strategy matches the application's data access patterns and operational requirements.

### 1.2 A Practical Example

Imagine that SIAPV eventually stores millions of maintenance records.

A common query might be:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2027-01-01';
```

If the table is partitioned by year, PostgreSQL may only need to access the partition containing records from 2026.

This can reduce the amount of data that needs to be examined.

The optimization is known as **partition pruning**.

---

## 2. Partitioned Tables and Partitions

A partitioned table has a logical structure that defines how its rows are distributed.

It consists of two important elements.

### 2.1 Parent Table

The **parent table**, also called the partitioned table, defines:

- Column names.
- Data types.
- Constraints.
- Partitioning strategy.
- Partition key.

It represents the logical table used by the application.

Example:

```sql
CREATE TABLE maintenance_records (
    id BIGINT NOT NULL,
    vehicle_id BIGINT NOT NULL,
    maintenance_date DATE NOT NULL,
    description TEXT
)
PARTITION BY RANGE (maintenance_date);
```

This creates a partitioned table.

It defines a range-based partitioning strategy using `maintenance_date`.

At this stage, the parent table does not contain the actual records in ordinary storage.

The rows are stored in its partitions.

### 2.2 Child Partitions

A **partition** is an individual table associated with the parent partitioned table.

Each partition accepts only rows that satisfy its partition bounds.

Example:

```sql
CREATE TABLE maintenance_records_2026
PARTITION OF maintenance_records
FOR VALUES FROM ('2026-01-01')
TO ('2027-01-01');
```

This creates a partition for records from 2026.

The upper boundary is exclusive.

Therefore:

- January 1, 2026 belongs to the partition.
- December 31, 2026 belongs to the partition.
- January 1, 2027 does not belong to the partition.

### 2.3 Logical Structure

```text
                 PARENT TABLE
              MAINTENANCE_RECORDS
                       |
          +------------+------------+
          |            |            |
          v            v            v
      PARTITION     PARTITION    PARTITION
        2024          2025         2026
          |            |            |
          v            v            v
      RECORDS       RECORDS      RECORDS
```

Applications can query the parent table without specifying the individual partitions.

PostgreSQL determines which partitions are relevant.

---

## 3. Partition Keys

A **partition key** is the column or set of columns used to determine which partition receives a row.

For example:

```sql
PARTITION BY RANGE (maintenance_date)
```

Here, `maintenance_date` is the partition key.

The value of this column determines the destination partition.

### 3.1 Characteristics of a Good Partition Key

A useful partition key should:

- Match common query filters.
- Support a meaningful data distribution strategy.
- Be stable enough for the chosen design.
- Allow clear partition boundaries.
- Support the application's retention and maintenance requirements.

Examples of possible partition keys:

| Column | Possible use |
|---|---|
| `created_at` | Organize records by time. |
| `transaction_date` | Organize financial transactions by period. |
| `region_id` | Organize data by geographic region. |
| `department_id` | Organize records by department. |
| `customer_id` | Distribute data according to a hash strategy. |

The correct key depends on the workload.

### 3.2 Partition Key and Query Patterns

Suppose an application frequently executes:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2026-04-01';
```

Partitioning by `maintenance_date` can be useful because the query filters directly on the partition key.

Now consider:

```sql
SELECT *
FROM maintenance_records
WHERE vehicle_id = 13995;
```

If the table is partitioned only by date, PostgreSQL may need to inspect multiple partitions to find records for that vehicle.

This does not mean the query cannot be optimized.

Indexes on individual partitions can help.

However, the partitioning strategy should be evaluated against the most important query patterns.

---

## 4. Types of Partitioning

PostgreSQL supports three principal declarative partitioning strategies:

1. Range partitioning.
2. List partitioning.
3. Hash partitioning.

Each strategy distributes records according to a different rule.

### 4.1 Range Partitioning

**Range partitioning** distributes rows according to intervals of values.

It is commonly used for:

- Dates.
- Timestamps.
- Numeric ranges.
- Sequential identifiers.

Example:

```sql
CREATE TABLE sales (
    sale_id BIGINT NOT NULL,
    sale_date DATE NOT NULL,
    amount NUMERIC(12,2)
)
PARTITION BY RANGE (sale_date);
```

Create yearly partitions:

```sql
CREATE TABLE sales_2025
PARTITION OF sales
FOR VALUES FROM ('2025-01-01')
TO ('2026-01-01');

CREATE TABLE sales_2026
PARTITION OF sales
FOR VALUES FROM ('2026-01-01')
TO ('2027-01-01');

CREATE TABLE sales_2027
PARTITION OF sales
FOR VALUES FROM ('2027-01-01')
TO ('2028-01-01');
```

The ranges are inclusive at the lower boundary and exclusive at the upper boundary.

Conceptually:

```text
SALES
|
+-- SALES_2025
|      2025-01-01 <= date < 2026-01-01
|
+-- SALES_2026
|      2026-01-01 <= date < 2027-01-01
|
+-- SALES_2027
       2027-01-01 <= date < 2028-01-01
```

**When to use it:**

When records are naturally organized by periods or numeric intervals.

### 4.2 List Partitioning

**List partitioning** distributes records according to specific values.

It is useful for categorical information.

Example:

```sql
CREATE TABLE service_requests (
    request_id BIGINT NOT NULL,
    department TEXT NOT NULL,
    description TEXT
)
PARTITION BY LIST (department);
```

Create partitions:

```sql
CREATE TABLE requests_administration
PARTITION OF service_requests
FOR VALUES IN ('ADMINISTRATION');

CREATE TABLE requests_operations
PARTITION OF service_requests
FOR VALUES IN ('OPERATIONS');

CREATE TABLE requests_maintenance
PARTITION OF service_requests
FOR VALUES IN ('MAINTENANCE');
```

Conceptually:

```text
SERVICE_REQUESTS
|
+-- ADMINISTRATION
|
+-- OPERATIONS
|
+-- MAINTENANCE
```

Each partition accepts only its assigned category.

**When to use it:**

When data naturally belongs to a limited set of categories, such as regions, departments, or business units.

### 4.3 Hash Partitioning

**Hash partitioning** distributes rows according to a hash function applied to the partition key.

Unlike range and list partitioning, it does not use explicit value intervals or manually assigned categories.

Example:

```sql
CREATE TABLE customer_events (
    event_id BIGINT NOT NULL,
    customer_id BIGINT NOT NULL,
    event_description TEXT
)
PARTITION BY HASH (customer_id);
```

Create four partitions:

```sql
CREATE TABLE customer_events_0
PARTITION OF customer_events
FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE customer_events_1
PARTITION OF customer_events
FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE customer_events_2
PARTITION OF customer_events
FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE customer_events_3
PARTITION OF customer_events
FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

PostgreSQL calculates a hash value from `customer_id` and uses the partition modulus and remainder to determine the destination.

Conceptually:

```text
CUSTOMER_EVENTS
|
+-- PARTITION 0
|
+-- PARTITION 1
|
+-- PARTITION 2
|
+-- PARTITION 3
```

**When to use it:**

When a reasonably even distribution across partitions is desirable and there is no natural range or category-based organization.

Hash partitioning does not guarantee perfectly equal partition sizes.

---

## 5. Comparing Partitioning Strategies

| Characteristic | Range | List | Hash |
|---|---|---|---|
| Distribution rule | Value intervals | Explicit values | Hash function |
| Typical key | Date or number | Category | Identifier |
| Common use | Historical data | Regions or departments | Even distribution |
| Boundaries | Defined intervals | Defined values | Modulus and remainder |
| Natural time organization | Excellent | Limited | Not suitable |
| Manual category assignment | No | Yes | No |

### 5.1 Choosing a Strategy

Consider the following scenarios.

| Scenario | Possible strategy |
|---|---|
| Millions of transactions organized by date | Range |
| Records grouped by business region | List |
| Large event table distributed by customer identifier | Hash |
| Historical logs organized by month | Range |
| Requests assigned to a fixed number of departments | List |

The strategy should reflect the structure of the data and the queries performed against it.

---

## 6. Partition Pruning

**Partition pruning** is the process of eliminating partitions that cannot contain rows satisfying a query's conditions.

It is one of the most important performance benefits of partitioning.

### 6.1 Without Partition Pruning

Imagine a table with records distributed across ten yearly partitions.

A query requests records from 2026.

Without effective partition pruning, the database might need to consider many partitions unnecessarily.

### 6.2 With Partition Pruning

If the query filters using the partition key, PostgreSQL can identify the relevant partition.

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2027-01-01';
```

If the table has yearly partitions, PostgreSQL can eliminate partitions outside 2026.

Conceptually:

```text
AVAILABLE PARTITIONS

2022  -> EXCLUDED
2023  -> EXCLUDED
2024  -> EXCLUDED
2025  -> EXCLUDED
2026  -> SCANNED
2027  -> EXCLUDED
```

This reduces the amount of data that must be examined.

### 6.3 Why It Improves Performance

Partition pruning can reduce:

- Number of partitions accessed.
- Amount of data scanned.
- I/O operations.
- Query execution work.
- Resource consumption for relevant queries.

However, actual performance depends on:

- Number of partitions.
- Data distribution.
- Query conditions.
- Available indexes.
- Table statistics.
- Execution plan.
- Storage performance.

Partitioning does not guarantee that every query becomes faster.

### 6.4 Inspecting Partition Pruning

Use:

```sql
EXPLAIN
SELECT *
FROM maintenance_records
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2027-01-01';
```

You can also execute:

```sql
EXPLAIN ANALYZE
SELECT *
FROM maintenance_records
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2027-01-01';
```

Inspect which partitions appear in the execution plan.

`EXPLAIN ANALYZE` executes the query, so use it carefully with statements that modify data.

### 6.5 Planning-Time and Execution-Time Pruning

PostgreSQL can eliminate partitions at different stages.

**Planning-time pruning:**

The planner determines which partitions are unnecessary while creating the execution plan.

**Execution-time pruning:**

The executor can eliminate additional partitions when certain parameter values become available during execution.

This can be relevant for prepared statements and parameterized queries.

---

## 7. Partitioning and Indexes

Partitioning and indexing solve related but different problems.

An index helps locate rows efficiently within a table.

Partitioning divides the table into smaller structures.

They can be used together.

### 7.1 Indexes on Partitioned Tables

PostgreSQL supports partitioned indexes.

For example:

```sql
CREATE INDEX idx_maintenance_vehicle
ON maintenance_records (vehicle_id);
```

When created on the partitioned parent table, PostgreSQL creates or manages corresponding indexes for its partitions.

The parent index represents the logical index across the partitioned table.

The physical index structures exist on individual partitions.

### 7.2 Why Combine Partitioning and Indexing?

Suppose a query searches for maintenance records from 2026 for a specific vehicle.

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2027-01-01'
  AND vehicle_id = 13995;
```

Two optimizations may work together:

1. Partition pruning eliminates partitions outside 2026.
2. An index on `vehicle_id` helps locate matching records within the remaining partition.

The combination can be more effective than either technique alone for the right workload.

### 7.3 Index Considerations

A partitioned table may have many indexes across its partitions.

This creates additional operational considerations:

- Index storage requirements.
- Index creation time.
- Index maintenance.
- Write overhead.
- Partition-level index consistency.
- Query plan evaluation.

Do not create indexes without understanding the queries they are intended to support.

See also:

- [[04 - Indexes]]
- [[05 - PostgreSQL Performance Fundamentals]]

---

## 8. Partition Constraints and Data Integrity

Each partition has boundaries that determine which rows it can contain.

For example:

```sql
CREATE TABLE sales_2026
PARTITION OF sales
FOR VALUES FROM ('2026-01-01')
TO ('2027-01-01');
```

PostgreSQL enforces the partition bounds.

A row with a date outside that interval cannot be stored in this partition.

### 8.1 Partition Constraints

Partition constraints describe the conditions that rows must satisfy.

For a range partition, the condition is based on its defined interval.

For a list partition, the condition is based on its assigned values.

For a hash partition, the condition is based on the hash partitioning rule.

PostgreSQL manages these constraints for declaratively created partitions.

### 8.2 Primary Keys and Unique Constraints

There is an important PostgreSQL design consideration.

A primary key or unique constraint declared on a partitioned table generally needs to include all partition key columns.

This is because PostgreSQL cannot enforce uniqueness across all partitions using a local unique index alone when the partition key is absent.

For example:

```sql
CREATE TABLE sales (
    sale_id BIGINT NOT NULL,
    sale_date DATE NOT NULL,
    amount NUMERIC(12,2),
    PRIMARY KEY (sale_id, sale_date)
)
PARTITION BY RANGE (sale_date);
```

Here, the primary key includes `sale_date`, which is also the partition key.

If the business requires `sale_id` to be globally unique by itself, the architecture must account for that requirement separately.

Possible approaches include:

- A globally coordinated identifier-generation strategy.
- A separate unpartitioned registry table.
- A data model that includes the partition key in the unique identifier.
- Another design that guarantees the required uniqueness.

This is a significant consideration when designing partitioned schemas.

---

## 9. Managing Partitions

Partitioning introduces a lifecycle of its own.

A database administrator or automated process may need to:

- Create future partitions.
- Monitor partition sizes.
- Inspect partition distribution.
- Archive historical partitions.
- Detach or remove obsolete partitions.
- Maintain indexes.
- Verify constraints.

### 9.1 Creating Future Partitions

Suppose a table is partitioned by month.

The database should have partitions available before new records arrive.

Example:

```sql
CREATE TABLE events_2027_01
PARTITION OF events
FOR VALUES FROM ('2027-01-01')
TO ('2027-02-01');
```

A production system may automate this process.

### 9.2 Default Partitions

PostgreSQL supports a default partition that stores rows not matching any explicitly defined partition.

Example:

```sql
CREATE TABLE events_default
PARTITION OF events
DEFAULT;
```

This can prevent inserts from failing when a value does not match an existing explicit partition.

However, it should not replace partition planning.

A default partition can accumulate unexpected records and requires monitoring.

When adding a new explicit partition, PostgreSQL may need to verify that existing default-partition rows do not belong in the new partition.

This can require additional work and may involve locking considerations.

### 9.3 Detaching a Partition

A partition can be detached from its parent table.

```sql
ALTER TABLE maintenance_records
DETACH PARTITION maintenance_records_2024;
```

This separates the table from the partitioned parent.

The exact locking and operational behavior depends on the detach option and PostgreSQL version.

### 9.4 Dropping a Partition

A partition can be removed:

```sql
DROP TABLE maintenance_records_2024;
```

This removes the partition and its data.

This operation is potentially destructive.

It should only be performed after verifying retention requirements and backup or archival policies.

### 9.5 Why Partition Maintenance Matters

Poor partition management can produce:

- Missing partitions.
- Failed inserts.
- Excessive numbers of partitions.
- Unexpected default-partition growth.
- Inconsistent retention practices.
- More complicated maintenance operations.

Partitioning is not a one-time configuration.

It is an ongoing data-management responsibility.

---

## 10. Partitioning vs Sharding

Partitioning and sharding both divide data, but their architectural meaning differs.

### 10.1 Partitioning

Partitioning organizes a table into smaller partitions within a database system.

In PostgreSQL declarative partitioning, the partitions are managed by the same PostgreSQL database cluster.

```text
             POSTGRESQL CLUSTER
                     |
                     v
              PARTITIONED TABLE
                     |
          +----------+----------+
          |          |          |
          v          v          v
      PARTITION  PARTITION  PARTITION
          1          2          3
```

### 10.2 Sharding

Sharding distributes data across separate database nodes or independent database instances.

```text
             APPLICATION
                  |
          +-------+-------+
          |       |       |
          v       v       v
        SHARD 1 SHARD 2 SHARD 3
        SERVER  SERVER  SERVER
```

Each shard stores a subset of the overall dataset.

This can distribute storage and processing across multiple servers.

### 10.3 Comparison

| Characteristic | Partitioning | Sharding |
|---|---|---|
| Main purpose | Organize a large table | Distribute data across nodes |
| Data location | Within a database cluster | Across database nodes |
| Management | Database partitioning features | Distributed architecture |
| Query complexity | Usually transparent through parent table | May require routing and coordination |
| Scaling | Can improve management and query efficiency | Can distribute storage and workload horizontally |
| Operational complexity | Moderate | Generally higher |

Partitioning does not automatically distribute data across multiple independent servers.

That is an important distinction.

See also:

- [[05 - Sharding]]
- [[02 - Database Scalability]]

---

## 11. Partitioning and Data Lifecycle Management

Partitioning is particularly useful when data has a defined lifecycle.

Consider a system that stores maintenance records indefinitely.

The organization may have different policies for:

- Current operational records.
- Recent historical records.
- Older records.
- Archived records.
- Records eligible for deletion.

Partitioning can simplify these operations.

### 11.1 Example: Historical Maintenance Data

```text
MAINTENANCE_RECORDS
|
+-- 2023 PARTITION
|      |
|      +-- ARCHIVE
|
+-- 2024 PARTITION
|      |
|      +-- ARCHIVE
|
+-- 2025 PARTITION
|      |
|      +-- HISTORICAL ACCESS
|
+-- 2026 PARTITION
|      |
|      +-- ACTIVE REPORTING
|
+-- 2027 PARTITION
       |
       +-- CURRENT OPERATIONS
```

The exact storage or archival approach depends on business requirements.

### 11.2 Retention Policies

A retention policy defines how long information must be preserved.

For example:

- Operational records: retained in the active database.
- Older records: moved to archival storage.
- Records beyond the legal retention period: deleted according to approved procedures.

Partitioning can simplify the movement or removal of groups of records.

However, partitioning does not decide which records should be retained.

That is a data governance decision.

See also:

- [[02 - The Data Lifecycle]]
- [[04 - Data Governance]]
- [[07 - Backup and Disaster Recovery]]

---

## 12. When Should You Use Partitioning?

Partitioning is worth evaluating when:

- Tables contain very large amounts of data.
- Queries frequently filter by a consistent partition key.
- Historical data has clear retention requirements.
- Individual data ranges need separate maintenance.
- Removing or archiving large groups of records is operationally important.
- Query execution plans can benefit from partition pruning.

### 12.1 When Partitioning May Not Be Necessary

Partitioning may be unnecessary when:

- Tables are relatively small.
- Queries do not benefit from the partition key.
- The workload is already efficient.
- Partition management would introduce more complexity than value.
- A suitable index is sufficient.
- Data distribution does not justify multiple partitions.

A database with a few thousand records does not automatically need partitioning.

The design should be justified by actual requirements and measured workload.

---

## 13. Practical PostgreSQL Laboratory

In this laboratory, you will create a partitioned table and observe how PostgreSQL distributes records.

### 13.1 Create a Range-Partitioned Table

```sql
CREATE TABLE maintenance_lab (
    record_id BIGINT NOT NULL,
    vehicle_id BIGINT NOT NULL,
    maintenance_date DATE NOT NULL,
    description TEXT,
    PRIMARY KEY (record_id, maintenance_date)
)
PARTITION BY RANGE (maintenance_date);
```

The table is partitioned by maintenance date.

The primary key includes the partition key.

### 13.2 Create Partitions

```sql
CREATE TABLE maintenance_lab_2025
PARTITION OF maintenance_lab
FOR VALUES FROM ('2025-01-01')
TO ('2026-01-01');

CREATE TABLE maintenance_lab_2026
PARTITION OF maintenance_lab
FOR VALUES FROM ('2026-01-01')
TO ('2027-01-01');

CREATE TABLE maintenance_lab_2027
PARTITION OF maintenance_lab
FOR VALUES FROM ('2027-01-01')
TO ('2028-01-01');
```

### 13.3 Insert Records

```sql
INSERT INTO maintenance_lab (
    record_id,
    vehicle_id,
    maintenance_date,
    description
)
VALUES
    (1, 13995, '2025-03-15', 'Preventive maintenance'),
    (2, 13995, '2026-05-20', 'Hydraulic inspection'),
    (3, 13996, '2026-08-10', 'Engine maintenance'),
    (4, 13990, '2027-02-05', 'Mechanical inspection');
```

The parent table automatically routes each row to the appropriate partition.

### 13.4 Query the Parent Table

```sql
SELECT *
FROM maintenance_lab;
```

The query returns rows from all partitions.

### 13.5 Query a Specific Period

```sql
SELECT *
FROM maintenance_lab
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2027-01-01';
```

PostgreSQL can eliminate partitions outside the requested date range.

### 13.6 Inspect the Execution Plan

```sql
EXPLAIN
SELECT *
FROM maintenance_lab
WHERE maintenance_date >= '2026-01-01'
  AND maintenance_date < '2027-01-01';
```

Observe which partition or partitions appear in the plan.

### 13.7 Inspect the Partitions

PostgreSQL provides system catalogs and information schema views that help identify partition relationships.

For example:

```sql
SELECT
    parent.relname AS parent_table,
    child.relname AS partition_name
FROM pg_inherits
JOIN pg_class parent
    ON pg_inherits.inhparent = parent.oid
JOIN pg_class child
    ON pg_inherits.inhrelid = child.oid
WHERE parent.relname = 'maintenance_lab';
```

This query lists the direct partitions associated with the parent table.

### 13.8 Inspect Partition Sizes

```sql
SELECT
    relname AS partition_name,
    pg_size_pretty(
        pg_total_relation_size(oid)
    ) AS total_size
FROM pg_class
WHERE relname LIKE 'maintenance_lab_%'
ORDER BY relname;
```

This helps compare the storage consumed by the partitions.

---

## 14. Applying Partitioning to SIAPV

Consider a future SIAPV database with a growing number of records.

Potential tables include:

- `vehicles`
- `maintenance_records`
- `fuel_records`
- `vehicle_assignments`
- `incident_reports`
- `inspection_records`
- `operational_events`

Not every table needs partitioning.

The decision should be based on expected growth, query patterns, and retention policies.

### 14.1 Maintenance Records

A maintenance table could use range partitioning by date.

```text
MAINTENANCE_RECORDS
|
+-- 2025
+-- 2026
+-- 2027
```

Possible benefits:

- Efficient historical queries.
- Easier archival.
- Better management of large datasets.
- Partition pruning for date-based reporting.

### 14.2 Operational Events

An operational events table could also use range partitioning by timestamp.

This may be useful if the system records a high volume of events.

### 14.3 Vehicle Inventory

The `vehicles` table may not need partitioning.

Even if the system grows to several thousand units, a well-designed table with appropriate indexes may remain sufficient.

This is an important architectural principle:

**Do not partition every table simply because partitioning exists.**

### 14.4 Example Architecture

```text
                 SIAPV DATABASE
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
     VEHICLES      MAINTENANCE      INCIDENTS
     REGULAR        PARTITIONED     PARTITIONED
     TABLE           BY DATE         BY DATE
```

This architecture applies partitioning selectively to tables with suitable growth and access patterns.

---

## 15. Common Partitioning Mistakes

| Mistake | Why It Is a Problem |
|---|---|
| Partitioning small tables unnecessarily | Adds complexity without meaningful benefit. |
| Choosing a poor partition key | Queries may still access many partitions. |
| Creating too many partitions | Increases planning and operational overhead. |
| Forgetting future partitions | Inserts can fail if no partition accepts the new row. |
| Ignoring indexes | Queries may remain inefficient within individual partitions. |
| Treating partitioning as sharding | Confuses table organization with distributed data architecture. |
| Ignoring unique constraints | Can produce incorrect assumptions about global uniqueness. |
| Never monitoring partition sizes | Creates uneven or unexpected data distribution. |
| Deleting partitions without retention review | Can cause irreversible data loss. |

---

## 16. Knowledge Check

Answer these questions without looking at the explanations.

1. What is database partitioning?
2. What is a partitioned table?
3. What is the difference between a parent table and a child partition?
4. What is a partition key?
5. How does range partitioning work?
6. When is list partitioning useful?
7. How does hash partitioning distribute records?
8. What is partition pruning?
9. How can partitioning and indexing work together?
10. Why must primary keys and unique constraints be considered carefully in PostgreSQL partitioned tables?
11. What is the purpose of a default partition?
12. Why does partition management require ongoing maintenance?
13. How is partitioning different from sharding?
14. How can partitioning support data retention policies?
15. Why should not every database table be partitioned?

---

## 17. Practical Exercises

### Exercise 1 — Choose a Partitioning Strategy

Select an appropriate strategy for each scenario.

| Scenario | Your decision |
|---|---|
| Financial transactions organized by date | ? |
| Requests grouped by department | ? |
| Customer events distributed by customer ID | ? |
| Historical sensor readings | ? |
| Records assigned to a fixed list of regions | ? |

Explain why you selected each strategy.

### Exercise 2 — Design a Partitioned Table

Design a table for storing vehicle maintenance records.

Requirements:

- Each record has an identifier.
- Each record references a vehicle.
- Each record contains a maintenance date.
- Each record has a description.
- Records are organized by year.

Write:

1. The parent table.
2. Three yearly partitions.
3. An insert statement.
4. A query that filters one year.
5. An execution-plan inspection query.

### Exercise 3 — Performance Analysis

Compare the execution plans of:

- A query that filters by the partition key.
- A query that filters only by another column.
- A query that filters by both columns.

Explain how partition pruning and indexes influence each plan.

### Exercise 4 — SIAPV Architecture

Evaluate which SIAPV tables could benefit from partitioning.

For each candidate, explain:

- Expected data growth.
- Candidate partition key.
- Appropriate partitioning strategy.
- Typical queries.
- Retention requirements.
- Possible operational risks.

Do not assume every table needs partitioning.

---

## 18. Technical Vocabulary

| English term | Meaning |
|---|---|
| Database partitioning | Database partitioning |
| Partitioned table | Partitioned table |
| Parent table | Parent table |
| Child partition | Child partition |
| Partition key | Partition key |
| Partition bound | Partition boundary |
| Range partitioning | Range partitioning |
| List partitioning | List partitioning |
| Hash partitioning | Hash partitioning |
| Partition pruning | Partition elimination |
| Partition constraint | Partition constraint |
| Partition maintenance | Partition maintenance |
| Partitioned index | Partitioned index |
| Default partition | Default partition |
| Data retention | Data retention |
| Historical data | Historical data |
| Query planner | Query planner |
| Execution plan | Execution plan |
| Storage overhead | Storage overhead |
| Data distribution | Data distribution |

---

## 19. Key Takeaways

- Partitioning divides a large table into smaller parts.
- A partitioned table provides a logical interface to its partitions.
- The partition key determines where each row is stored.
- PostgreSQL supports range, list, and hash partitioning.
- Range partitioning is especially useful for time-based data.
- List partitioning is useful for categorical values.
- Hash partitioning distributes data according to a hash function.
- Partition pruning eliminates partitions that cannot satisfy a query.
- Indexes can complement partitioning.
- PostgreSQL has important constraints concerning unique keys on partitioned tables.
- Partition maintenance is an ongoing operational responsibility.
- Partitioning does not automatically distribute data across independent servers.
- Sharding and partitioning solve different architectural problems.
- Partitioning should be justified by workload, data growth, and operational requirements.

## 20. Next Learning Step

The next topic is **Database Sharding**.

Partitioning organizes data into smaller structures within a database cluster.

Sharding takes horizontal data distribution further by placing subsets of a dataset across separate database nodes.

We will study shard keys, routing, distribution strategies, cross-shard queries, rebalancing, and the operational complexity of distributed databases.

**Next file:** `04 - Database Architecture/05 - Sharding.md`

Related notes:

- [[02 - Database Scalability]]
- [[03 - Replication]]
- [[05 - Sharding]]
- [[06 - High Availability]]
- [[05 - Data Warehouses]]
- [[01 - PostgreSQL Architecture Overview]]
