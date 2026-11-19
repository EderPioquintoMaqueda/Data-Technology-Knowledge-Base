
---
title: "Database Sharding"
aliases:
  - "Database Sharding Architecture"
  - "Horizontal Database Scaling"
  - "Distributed Database Sharding"
type: learning-note
domain: technology
category: database-architecture
subject: database-sharding
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - sharding
  - distributed-systems
  - horizontal-scaling
  - database-scalability
  - data-distribution
  - shard-key
  - postgresql
  - query-routing
  - distributed-databases

concepts:
  - database-sharding
  - shard
  - shard-key
  - horizontal-partitioning
  - horizontal-scaling
  - data-locality
  - shard-routing
  - shard-rebalancing
  - cross-shard-query
  - distributed-transaction
  - data-skew
  - hotspot
  - consistent-hashing
  - shard-topology
  - distributed-join

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Client Server Architecture]]"
  - "[[02 - Database Scalability]]"
  - "[[03 - Replication]]"
  - "[[04 - Partitioning]]"
  - "[[05 - Transactions and ACID]]"

related:
  - "[[06 - High Availability]]"
  - "[[07 - Backup and Disaster Recovery]]"
  - "[[01 - Data Architecture Overview]]"
  - "[[04 - Data Pipelines]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"

tools:
  - PostgreSQL
  - SQL
  - psql
  - Docker
  - Distributed SQL
  - Citus
  - EXPLAIN

learning_objectives:
  - "Explain database sharding and its role in horizontal scaling."
  - "Distinguish sharding from partitioning and replication."
  - "Understand shard keys and data distribution strategies."
  - "Compare range-based, hash-based, and directory-based sharding."
  - "Understand query routing and cross-shard queries."
  - "Identify data skew, hotspots, and rebalancing challenges."
  - "Understand the impact of sharding on transactions and consistency."
  - "Evaluate when a database architecture should use sharding."

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 05 — Database Sharding

## 1. What Is Database Sharding?

**Database sharding** is a horizontal scaling technique that distributes subsets of a dataset across multiple independent database nodes.

Each node stores only part of the complete dataset.

These nodes are called **shards**.

Instead of placing all records on one database server, the system distributes them across multiple servers.

Consider a database containing customer information.

Without sharding:

```text
                 APPLICATION
                      |
                      v
               DATABASE SERVER
                      |
                      v
                ALL CUSTOMERS
```

With sharding:

```text
                  APPLICATION
                       |
                SHARD ROUTER
                       |
          +------------+------------+
          |            |            |
          v            v            v
       SHARD 1      SHARD 2      SHARD 3
       SERVER A     SERVER B     SERVER C
          |            |            |
          v            v            v
      CUSTOMERS    CUSTOMERS    CUSTOMERS
       1 - 1000    1001 - 2000  2001 - 3000
```

Each shard stores a subset of the records.

Together, the shards represent the distributed dataset.

### 1.1 Why Is Sharding Used?

Sharding is used when a single database node can no longer efficiently support the system's storage or workload requirements.

Common objectives include:

- Distributing storage across multiple servers.
- Increasing total processing capacity.
- Distributing write workloads.
- Supporting large datasets.
- Reducing the workload handled by individual nodes.
- Scaling systems across different infrastructure environments.

Sharding is particularly relevant to systems with very large datasets and high transaction volumes.

### 1.2 The Main Idea

The fundamental principle is:

**Instead of making one database server increasingly powerful, distribute the workload across multiple database servers.**

This is horizontal scaling.

However, horizontal scaling introduces additional architectural complexity.

The application or database infrastructure must determine where each record belongs and where each query should be executed.

---

## 2. Understanding a Shard

A **shard** is an individual database node or database partition in a distributed database architecture.

Each shard stores part of the overall dataset.

For example, an application could distribute customer records across three database servers.

| Shard | Stored customers |
|---|---|
| Shard A | Customers 1–1,000 |
| Shard B | Customers 1,001–2,000 |
| Shard C | Customers 2,001–3,000 |

The complete dataset is distributed across the three nodes.

### 2.1 Shard Responsibilities

A shard may be responsible for:

- Storing a subset of records.
- Processing queries involving its data.
- Executing transactions on locally stored records.
- Maintaining local indexes.
- Participating in distributed operations.
- Replicating its data to another node for availability.

A shard is not necessarily a single physical machine.

It can represent a database instance or a logical database group.

### 2.2 Shard vs Replica

These concepts must be distinguished.

A shard stores a different subset of the overall dataset.

A replica stores another copy of the same shard's data.

Example:

```text
                 DISTRIBUTED DATABASE
                          |
             +------------+------------+
             |                         |
             v                         v
          SHARD A                   SHARD B
             |                         |
        +----+----+               +----+----+
        |         |               |         |
        v         v               v         v
      PRIMARY   REPLICA         PRIMARY   REPLICA
```

Sharding distributes data.

Replication duplicates data.

They can be combined in the same architecture.

---

## 3. Sharding vs Partitioning vs Replication

These three concepts are related but solve different problems.

### 3.1 Partitioning

Partitioning divides a table into smaller sections within a database system.

Example:

```text
             DATABASE CLUSTER
                    |
                    v
             PARTITIONED TABLE
                    |
          +---------+---------+
          |         |         |
          v         v         v
       PARTITION PARTITION PARTITION
          1         2         3
```

### 3.2 Sharding

Sharding distributes subsets of a dataset across separate database nodes.

Example:

```text
              DISTRIBUTED DATABASE
                       |
          +------------+------------+
          |            |            |
          v            v            v
       SHARD A       SHARD B      SHARD C
       SERVER A      SERVER B     SERVER C
```

### 3.3 Replication

Replication maintains copies of data across database instances.

Example:

```text
                 SHARD A PRIMARY
                        |
                        v
                  SHARD A REPLICA
```

### 3.4 Comparison

| Characteristic | Partitioning | Sharding | Replication |
|---|---|---|---|
| Main objective | Organize large tables | Distribute data and workload | Maintain additional copies |
| Data distribution | Within a database system | Across database nodes | Across database instances |
| Duplicate data | No, normally | No, normally | Yes |
| Horizontal scaling | Not by itself | Yes | Can support read scaling |
| Typical use | Large tables | Very large distributed datasets | Availability and recovery |

A distributed database architecture may use all three techniques.

---

## 4. The Shard Key

The **shard key** is the column or combination of columns used to determine where a record is stored.

It is one of the most important decisions in a sharded architecture.

For example, consider a customer table:

```sql
CREATE TABLE customers (
    customer_id BIGINT PRIMARY KEY,
    full_name TEXT NOT NULL,
    region TEXT NOT NULL
);
```

A system could distribute customers according to `customer_id`.

```text
customer_id
     |
     v
SHARDING RULE
     |
     +----------------+
     |                |
     v                v
 SHARD A           SHARD B
```

Alternatively, it could use `region`.

The choice affects:

- Data distribution.
- Query routing.
- Data locality.
- Transaction boundaries.
- Performance.
- Future scalability.

### 4.1 Characteristics of a Good Shard Key

A good shard key should ideally:

1. Distribute data reasonably evenly.
2. Support common query patterns.
3. Avoid concentrating too much activity on one shard.
4. Remain sufficiently stable.
5. Minimize cross-shard operations.
6. Support future growth.
7. Reflect the application's data access patterns.

There is no universally optimal shard key.

The correct choice depends on the system.

### 4.2 Example: Customer ID

A customer identifier may be useful because it can distribute records across multiple shards.

For example:

```text
customer_id = 1001 -> SHARD A
customer_id = 1002 -> SHARD B
customer_id = 1003 -> SHARD C
```

The actual destination depends on the distribution strategy.

### 4.3 Example: Region

A geographic region can also be used.

```text
REGION
  |
  +-- NORTH  -> SHARD A
  |
  +-- CENTER -> SHARD B
  |
  +-- SOUTH  -> SHARD C
```

This can improve data locality when users primarily access records from their own region.

However, uneven regional growth can produce imbalanced shards.

---

## 5. Sharding Strategies

There are several ways to distribute data across shards.

The three common conceptual strategies are:

1. Range-based sharding.
2. Hash-based sharding.
3. Directory-based sharding.

### 5.1 Range-Based Sharding

**Range-based sharding** assigns records to shards according to intervals of values.

Example:

```text
CUSTOMER ID

1 - 1000       -> SHARD A
1001 - 2000    -> SHARD B
2001 - 3000    -> SHARD C
```

A simplified architecture:

```text
             CUSTOMER DATA
                   |
              SHARD KEY
                   |
          +--------+--------+
          |        |        |
          v        v        v
       RANGE 1  RANGE 2  RANGE 3
          |        |        |
          v        v        v
       SHARD A  SHARD B  SHARD C
```

**Advantages:**

- Simple to understand.
- Useful for ordered data.
- Can support range-based queries.
- Allows clear allocation of value intervals.

**Disadvantages:**

- Uneven data distribution is possible.
- Some ranges may receive more activity than others.
- New records may concentrate on the latest range.
- Reassigning ranges can be operationally expensive.

Range-based sharding is useful when the data has a natural ordering and the workload can be distributed effectively.

### 5.2 Hash-Based Sharding

**Hash-based sharding** applies a hash function to the shard key.

The resulting value determines the destination shard.

Conceptually:

```text
customer_id
     |
     v
HASH FUNCTION
     |
     v
HASH VALUE
     |
     v
SHARD ASSIGNMENT
     |
 +---+---+---+
 |   |   |   |
 v   v   v   v
 A   B   C   D
```

A simplified rule might be:

```text
shard_number = hash(customer_id) % number_of_shards
```

This is a conceptual formula.

Production systems may use more sophisticated distribution mechanisms.

**Advantages:**

- Can distribute records more evenly.
- Reduces dependence on sequential value ranges.
- Useful when identifiers are distributed across many values.

**Disadvantages:**

- Range queries can require access to multiple shards.
- Changing the number of shards can require significant redistribution.
- Data locality based on ordered values is reduced.
- Poor hash-key selection can still produce uneven workloads.

### 5.3 Directory-Based Sharding

**Directory-based sharding** uses a mapping structure to determine where records are stored.

Example:

```text
CUSTOMER ID     SHARD
---------------------
1001            A
1002            C
1003            B
1004            A
1005            C
```

The application or routing service consults the directory before accessing the database.

```text
              APPLICATION
                   |
                   v
             SHARD DIRECTORY
                   |
          +--------+--------+
          |        |        |
          v        v        v
       SHARD A  SHARD B  SHARD C
```

**Advantages:**

- Flexible distribution.
- Records can be assigned according to custom rules.
- Shards can be reorganized without relying exclusively on a fixed hash or range.

**Disadvantages:**

- The directory must remain available.
- Mapping information must be maintained.
- Routing introduces additional complexity.
- Directory consistency becomes important.

### 5.4 Strategy Comparison

| Characteristic | Range | Hash | Directory |
|---|---|---|---|
| Distribution | Value intervals | Hash function | Explicit mapping |
| Flexibility | Moderate | Moderate | High |
| Range queries | Often favorable | Often less favorable | Depends on mapping |
| Even distribution | Not guaranteed | Often more balanced | Depends on allocation |
| Mapping complexity | Relatively low | Moderate | Higher |
| Main challenge | Hot ranges | Resharding | Directory management |

---

## 6. Consistent Hashing

A related concept is **consistent hashing**.

It is a distribution technique designed to reduce the number of records that need to be reassigned when nodes are added or removed.

### 6.1 The Problem with Simple Modulo Hashing

Consider this conceptual rule:

```text
shard = hash(customer_id) % 3
```

There are three shards.

If the system increases to four shards:

```text
shard = hash(customer_id) % 4
```

Many records may now map to different shards.

This can require extensive data movement.

### 6.2 Consistent Hashing

Consistent hashing organizes nodes and keys in a logical hash space.

A key is assigned to a node according to its position in that space.

When a node is added or removed, the system aims to minimize the number of keys that need reassignment.

Conceptually:

```text
             HASH SPACE

                 NODE A
                    |
           +--------+--------+
           |                 |
        KEYS A            KEYS B
           |                 |
           v                 v
         NODE B            NODE C
```

Real implementations commonly use additional techniques such as virtual nodes to improve distribution.

### 6.3 Why It Matters

Consistent hashing can help with:

- Adding database nodes.
- Removing failed nodes.
- Rebalancing distributed workloads.
- Reducing unnecessary data movement.

It is not a complete sharding architecture by itself.

The system still needs data routing, replication, failure handling, and operational coordination.

---

## 7. Query Routing

In a sharded architecture, queries must reach the correct shard.

This is known as **query routing**.

A query router or application-level routing component determines which database node contains the requested information.

### 7.1 Query by Shard Key

Suppose customers are distributed according to `customer_id`.

An application requests:

```sql
SELECT *
FROM customers
WHERE customer_id = 1002;
```

The router determines the corresponding shard and sends the query there.

```text
             APPLICATION
                  |
                  v
             QUERY ROUTER
                  |
           customer_id = 1002
                  |
                  v
               SHARD C
                  |
                  v
               RESULT
```

This is an efficient access pattern when the shard key is included in the query.

### 7.2 Query Without Shard Key

Now consider:

```sql
SELECT *
FROM customers
WHERE full_name = 'Alex Johnson';
```

If `full_name` is not the shard key, the router may not know which shard contains the record.

It may need to send the query to multiple shards.

```text
              QUERY ROUTER
                   |
         +---------+---------+
         |         |         |
         v         v         v
      SHARD A   SHARD B   SHARD C
         |         |         |
         +---------+---------+
                   |
                   v
             MERGE RESULTS
```

This is called a **scatter-gather query**.

It can increase network traffic and response time.

### 7.3 Query Routing Responsibilities

A routing component may need to:

- Identify the shard key.
- Determine the destination shard.
- Forward queries.
- Manage database connections.
- Collect results from multiple shards.
- Handle node failures.
- Apply routing rules.
- Support retries and timeouts.

Query routing is one of the central responsibilities of a distributed database architecture.

---

## 8. Cross-Shard Queries and Joins

A **cross-shard query** accesses information stored on multiple shards.

These queries can be significantly more complicated than queries executed on one database node.

### 8.1 Local Query

Suppose an order and its order items are stored on the same shard.

```sql
SELECT *
FROM orders
JOIN order_items
    ON orders.order_id = order_items.order_id
WHERE orders.customer_id = 1002;
```

If all relevant records are colocated, the database can execute the join locally.

### 8.2 Distributed Join

Now suppose the required information is distributed across multiple shards.

The system may need to:

1. Send queries to different shards.
2. Retrieve intermediate results.
3. Transfer data across the network.
4. Combine the results.
5. Return the final dataset.

This introduces additional costs.

```text
                APPLICATION
                     |
                QUERY ROUTER
                     |
          +----------+----------+
          |                     |
          v                     v
       SHARD A               SHARD B
       ORDERS                CUSTOMERS
          |                     |
          +----------+----------+
                     |
                     v
               RESULT MERGE
                     |
                     v
                APPLICATION
```

### 8.3 Why Data Colocation Matters

**Data colocation** means storing related records on the same shard whenever possible.

For example, orders and order items could be distributed using a common customer or order key.

This reduces the need for cross-shard joins.

A good sharding strategy considers relationships between entities, not just the size of individual tables.

---

## 9. Distributed Transactions

A normal database transaction may execute entirely within one database node.

A distributed transaction involves multiple nodes.

This introduces additional coordination.

### 9.1 Local Transaction

```text
APPLICATION
    |
    v
SHARD A
    |
    v
BEGIN
UPDATE
COMMIT
```

The transaction is executed within one shard.

### 9.2 Distributed Transaction

```text
              APPLICATION
                   |
            +------+------+
            |             |
            v             v
          SHARD A       SHARD B
            |             |
            +------+------+
                   |
              COORDINATION
```

The system must coordinate the transaction across multiple nodes.

A failure during this process can leave participants in different states unless the transaction protocol handles it correctly.

### 9.3 Two-Phase Commit

One mechanism for coordinating distributed transactions is **Two-Phase Commit (2PC)**.

It has two main phases.

**Phase 1 — Prepare**

The coordinator asks participants whether they can commit.

Each participant prepares its transaction and reports its status.

**Phase 2 — Commit or Abort**

If all participants are prepared, the coordinator instructs them to commit.

If the operation cannot proceed, the coordinator can instruct them to abort.

Conceptually:

```text
             COORDINATOR
                  |
             PREPARE PHASE
                  |
          +-------+-------+
          |               |
          v               v
       SHARD A         SHARD B
          |               |
          +-------+-------+
                  |
            COMMIT PHASE
                  |
          +-------+-------+
          |               |
          v               v
       COMMIT          COMMIT
```

2PC introduces coordination and operational overhead.

It can also create blocking situations when failures occur at critical stages.

Not every sharded architecture uses distributed transactions for every operation.

Many systems deliberately design transactions to remain within one shard.

---

## 10. Data Skew and Hotspots

A major sharding challenge is uneven data distribution.

Two important problems are **data skew** and **hotspots**.

### 10.1 Data Skew

Data skew occurs when the amount of data stored across shards is significantly unbalanced.

Example:

| Shard | Number of records |
|---|---:|
| Shard A | 900,000 |
| Shard B | 100,000 |
| Shard C | 120,000 |

Shard A stores substantially more records.

This can lead to:

- Higher storage consumption.
- Greater query workload.
- Uneven resource utilization.
- Increased maintenance costs.

A distribution strategy should aim for a reasonable balance.

Perfect equality is not always possible or necessary.

### 10.2 Hotspots

A hotspot occurs when a particular shard receives a disproportionately high amount of activity.

For example:

```text
SHARD A -> 20% of requests
SHARD B -> 25% of requests
SHARD C -> 55% of requests
```

Even if record counts are balanced, workload distribution may not be.

A shard can become a bottleneck if many users access the same subset of records.

### 10.3 Example: Geographic Distribution

Suppose a system uses region as its shard key.

```text
NORTH  -> SHARD A
CENTER -> SHARD B
SOUTH  -> SHARD C
```

If the central region has far more users and transactions, Shard B may become overloaded.

The data distribution appears logically reasonable but does not necessarily match the actual workload.

### 10.4 Reducing Skew and Hotspots

Possible strategies include:

- Choosing a different shard key.
- Using a hash-based distribution.
- Splitting overloaded ranges.
- Adding capacity.
- Rebalancing data.
- Separating high-volume tenants.
- Distributing frequently accessed records more effectively.

The key lesson is that a good shard key must account for both data volume and access patterns.

---

## 11. Rebalancing and Resharding

As an application grows, the original shard distribution may become inefficient.

**Rebalancing** is the process of redistributing data or workload across shards.

**Resharding** involves changing the shard allocation or distribution structure.

### 11.1 Why Rebalancing Is Needed

Consider a system that initially has two shards.

```text
INITIAL ARCHITECTURE

SHARD A -> 500,000 records
SHARD B -> 500,000 records
```

The system grows and one shard becomes overloaded.

```text
AFTER GROWTH

SHARD A -> 2,000,000 records
SHARD B -> 700,000 records
```

The distribution is no longer balanced.

The system may need to move records or create additional shards.

### 11.2 Rebalancing Process

A simplified rebalancing process may involve:

1. Evaluating current shard sizes and workload.
2. Identifying overloaded nodes.
3. Defining a new distribution.
4. Moving data.
5. Updating routing metadata.
6. Verifying data consistency.
7. Monitoring application behavior.

### 11.3 Rebalancing Challenges

Rebalancing can introduce:

- Network traffic.
- Temporary performance degradation.
- Additional storage requirements.
- Data synchronization challenges.
- Routing changes.
- Operational risk.

The system must be designed to support growth without requiring disruptive migrations every time a new shard is added.

---

## 12. Replication and High Availability Within Shards

Sharding does not automatically provide high availability.

If a shard becomes unavailable and no replica exists, part of the dataset may become inaccessible.

A common architecture combines sharding and replication.

```text
                  DISTRIBUTED DATABASE
                           |
             +-------------+-------------+
             |                           |
             v                           v
          SHARD A                     SHARD B
             |                           |
        +----+----+                 +----+----+
        |         |                 |         |
        v         v                 v         v
      PRIMARY   REPLICA           PRIMARY   REPLICA
```

Each shard can maintain its own replica.

This provides separate availability mechanisms for each subset of data.

The architecture must also coordinate:

- Failover.
- Replica monitoring.
- Data consistency.
- Connection routing.
- Backup and recovery.
- Node replacement.

Sharding distributes data.

Replication protects additional copies of that data.

They complement each other.

---

## 13. Security and Operational Considerations

A distributed database architecture increases the number of components that must be protected and maintained.

Important considerations include:

### 13.1 Security

- Authenticate applications and database nodes.
- Use encrypted network connections.
- Apply least-privilege permissions.
- Protect shard-routing metadata.
- Secure replication credentials.
- Monitor access across all shards.

### 13.2 Monitoring

Monitor each shard independently.

Useful metrics include:

- CPU utilization.
- Memory utilization.
- Storage consumption.
- Query latency.
- Transaction throughput.
- Connection count.
- Replication lag.
- Error rates.
- Network utilization.
- Data distribution.

### 13.3 Backup and Recovery

Each shard needs a reliable backup and recovery strategy.

The overall system must also consider how to recover a consistent distributed dataset.

A backup process that captures different shards at unrelated points in time may require additional coordination depending on the application's consistency requirements.

### 13.4 Operational Complexity

A sharded system requires coordinated management of:

- Multiple database nodes.
- Connection pools.
- Query routing.
- Schema changes.
- Monitoring.
- Backups.
- Failover.
- Data migrations.
- Capacity planning.

This is why sharding should not be introduced casually.

---

## 14. PostgreSQL and Sharding

PostgreSQL provides native table partitioning, but its standard single-server architecture does not automatically distribute a partitioned table across independent PostgreSQL servers.

To implement distributed PostgreSQL architectures, additional technologies or application-level mechanisms may be used.

### 14.1 Application-Level Sharding

The application determines which database instance should receive each query.

Example:

```text
APPLICATION
    |
    v
SHARD ROUTING LOGIC
    |
    +-------> DATABASE A
    |
    +-------> DATABASE B
    |
    +-------> DATABASE C
```

The application is responsible for mapping records to database nodes.

Advantages:

- Explicit control.
- Flexible routing.
- Can be adapted to business rules.

Disadvantages:

- More application complexity.
- Routing logic must be maintained.
- Database migrations can be harder.
- Cross-shard operations require additional handling.

### 14.2 Distributed Database Extensions

Technologies such as <Entity value="Citus" category="software" disambig="Distributed PostgreSQL extension"/> extend PostgreSQL to support distributed database workloads.

Citus introduces concepts such as:

- Coordinator nodes.
- Worker nodes.
- Distributed tables.
- Distribution columns.
- Shard placement.

A simplified architecture:

```text
              APPLICATION
                   |
                   v
             COORDINATOR
                   |
          +--------+--------+
          |        |        |
          v        v        v
       WORKER 1 WORKER 2 WORKER 3
          |        |        |
          v        v        v
       SHARDS   SHARDS   SHARDS
```

The coordinator manages query planning and distribution across worker nodes.

The exact architecture and capabilities depend on the Citus version and deployment model.

### 14.3 Important Distinction

PostgreSQL partitioning and distributed PostgreSQL are not the same thing.

- Native partitioning divides tables within a PostgreSQL database cluster.
- Distributed PostgreSQL distributes data and execution across multiple nodes through additional architecture or tooling.

---

## 15. When Should You Use Sharding?

Sharding may be appropriate when:

- A single database server cannot meet storage requirements.
- Write throughput exceeds the capacity of one node.
- The dataset is too large for a single database instance.
- Workloads can be divided into independent subsets.
- The application has a suitable shard key.
- The organization can support distributed operations.
- Horizontal scaling is an explicit requirement.

### 15.1 When Sharding May Be Unnecessary

Sharding may not be appropriate when:

- The database is relatively small.
- Query performance can be improved through indexes.
- A larger server is sufficient.
- Connection management is the actual bottleneck.
- Poor SQL queries are causing performance problems.
- Partitioning is enough to address data-management needs.
- The organization lacks the operational capability to manage a distributed database.

A system should not be sharded simply because it is growing.

The actual bottleneck must be identified first.

### 15.2 Recommended Evaluation Sequence

Before considering sharding:

1. Measure database performance.
2. Inspect slow queries.
3. Evaluate indexes.
4. Review connection pooling.
5. Evaluate caching.
6. Consider vertical scaling.
7. Consider read replicas.
8. Evaluate partitioning.
9. Determine whether a single database node remains insufficient.
10. Design and test a sharded architecture if justified.

Sharding is usually a later architectural decision because it introduces significant complexity.

---

## 16. Practical Architecture Example

Consider a fictional application managing customers and transactions.

The application initially uses one PostgreSQL database.

As the system grows, it reaches:

- Millions of customer records.
- Hundreds of millions of transactions.
- High concurrent write activity.
- Multiple geographic regions.
- Large analytical workloads.

A possible architecture could be:

```text
                    APPLICATION
                         |
                         v
                    QUERY ROUTER
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
          SHARD A     SHARD B     SHARD C
             |           |           |
             v           v           v
          PRIMARY     PRIMARY     PRIMARY
             |           |           |
             v           v           v
          REPLICA     REPLICA     REPLICA
```

Each shard stores a subset of the overall customer and transaction dataset.

Each shard also has a replica for availability.

Analytical workloads may be handled through a separate data platform that receives information from the operational system.

This architecture separates operational transaction processing from large-scale analytical processing.

---

## 17. Applying Sharding to SIAPV

Consider a hypothetical future SIAPV system that expands to support:

- Multiple states.
- Thousands of managed vehicles and machinery units.
- Multiple government departments.
- High volumes of maintenance records.
- Historical operational events.
- Multiple application instances.

At that point, sharding could become an architectural option, but it would require evidence that a single database system is insufficient.

### 17.1 Possible Shard Keys

Potential shard keys could include:

| Candidate | Potential advantage | Potential risk |
|---|---|---|
| `vehicle_id` | Groups records around individual vehicles | Queries across many vehicles may touch multiple shards |
| `department_id` | Supports departmental data locality | Some departments may have much larger workloads |
| `region_id` | Supports geographic organization | Uneven regional activity |
| `event_id` | Can distribute event records | Related vehicle queries may require cross-shard operations |

The selection should be based on the application's most important transactions.

### 17.2 Example: Vehicle-Centered Distribution

Suppose the system uses `vehicle_id` as its distribution key.

```text
                  SIAPV
                    |
               SHARD ROUTER
                    |
          +---------+---------+
          |         |         |
          v         v         v
       SHARD A   SHARD B   SHARD C
          |         |         |
          v         v         v
       VEHICLES  VEHICLES  VEHICLES
       1 - 1000  1001-2000 2001-3000
```

Each shard stores vehicle-related information.

If maintenance records are distributed using the same key, the system may keep a vehicle's records on the same shard.

This can improve data locality.

### 17.3 Architectural Trade-Off

However, reports that aggregate all maintenance activity across every vehicle may require querying multiple shards.

The system would need to collect and combine results.

This is a trade-off between efficient local transactions and more complicated global reporting.

### 17.4 Should SIAPV Be Sharded Now?

Not necessarily.

A system managing approximately 200 units does not automatically require sharding.

The appropriate architecture may remain a well-designed PostgreSQL database with:

- Appropriate relational modeling.
- Indexes.
- Connection management.
- Backups.
- Monitoring.
- Query optimization.
- Selective partitioning when justified.

The objective is to understand sharding as a future scaling option, not to introduce unnecessary complexity into the current application.

---

## 18. Common Sharding Mistakes

| Mistake | Why It Is a Problem |
|---|---|
| Choosing a poor shard key | Produces uneven distribution or inefficient queries. |
| Ignoring query patterns | Causes frequent cross-shard operations. |
| Assuming all shards have equal workloads | Data volume and request volume can differ. |
| Ignoring data colocation | Makes related joins more expensive. |
| Forgetting replication | A failed shard can make its data unavailable. |
| Ignoring rebalancing | Growth can create overloaded nodes. |
| Using distributed transactions unnecessarily | Adds coordination overhead. |
| Treating sharding as a backup | Sharding does not preserve historical recovery copies. |
| Sharding too early | Creates complexity without sufficient benefit. |
| Ignoring global uniqueness | Local constraints may not guarantee system-wide uniqueness. |

---

## 19. Knowledge Check

Answer these questions without looking at the explanations.

1. What is database sharding?
2. What is a shard?
3. How is a shard different from a replica?
4. What is the purpose of a shard key?
5. What characteristics make a shard key useful?
6. How does range-based sharding work?
7. How does hash-based sharding work?
8. What is directory-based sharding?
9. What problem does consistent hashing attempt to reduce?
10. What is query routing?
11. Why can a query without a shard key become expensive?
12. What is a scatter-gather query?
13. Why are cross-shard joins complicated?
14. What is a distributed transaction?
15. What are the two phases of two-phase commit?
16. What is data skew?
17. What is a hotspot?
18. Why is rebalancing necessary?
19. How can sharding and replication work together?
20. Why should sharding generally be considered only after evaluating simpler scaling strategies?

---

## 20. Practical Exercises

### Exercise 1 — Choose a Shard Key

Imagine a database for a large fleet management platform.

It contains:

- Vehicles.
- Departments.
- Maintenance records.
- Operators.
- Incidents.
- Historical events.

Evaluate these potential shard keys:

- `vehicle_id`
- `department_id`
- `region_id`
- `event_id`

For each candidate, explain:

1. How records would be distributed.
2. Which queries would benefit.
3. Which queries might become expensive.
4. Whether the key could create data skew.
5. Whether related records could remain colocated.

### Exercise 2 — Compare Distribution Strategies

Design three conceptual architectures:

- Range-based sharding.
- Hash-based sharding.
- Directory-based sharding.

Use the same customer dataset in all three designs.

Explain how each architecture assigns records to shards and how it handles growth.

### Exercise 3 — Analyze a Hotspot

A distributed database has three shards:

| Shard | Storage | Daily requests |
|---|---:|---:|
| A | 500 GB | 1 million |
| B | 450 GB | 1.2 million |
| C | 300 GB | 8 million |

Answer:

1. Which shard is the main workload bottleneck?
2. Does storage distribution explain the workload distribution?
3. What could cause the hotspot?
4. Which architectural changes could help?
5. Would adding a fourth shard automatically solve the problem?

### Exercise 4 — Design a Future SIAPV Architecture

Design a hypothetical distributed SIAPV architecture.

Include:

- A proposed shard key.
- Distribution strategy.
- Number of initial shards.
- Query-routing mechanism.
- Replication strategy.
- Backup strategy.
- Monitoring requirements.
- Rebalancing approach.
- Cross-shard reporting strategy.

Justify every decision.

---

## 21. Technical Vocabulary

| English term | Meaning |
|---|---|
| Database sharding | Database sharding |
| Shard | Shard or data fragment |
| Shard key | Sharding key |
| Horizontal scaling | Horizontal scaling |
| Data distribution | Data distribution |
| Range-based sharding | Range-based sharding |
| Hash-based sharding | Hash-based sharding |
| Directory-based sharding | Directory-based sharding |
| Consistent hashing | Consistent hashing |
| Query routing | Query routing |
| Shard router | Shard router |
| Data locality | Data locality |
| Data colocation | Data colocation |
| Cross-shard query | Cross-shard query |
| Scatter-gather query | Scatter-gather query |
| Distributed join | Distributed join |
| Distributed transaction | Distributed transaction |
| Two-Phase Commit | Two-phase commit |
| Data skew | Data imbalance |
| Hotspot | Workload hotspot |
| Rebalancing | Rebalancing |
| Resharding | Resharding |
| Coordinator | Coordinator |
| Worker node | Worker node |
| Shard topology | Shard topology |

---

## 22. Key Takeaways

- Sharding distributes subsets of a dataset across multiple database nodes.
- A shard is an individual component of a distributed database.
- A shard key determines where records are stored.
- Range-based sharding distributes records according to value intervals.
- Hash-based sharding distributes records through a hash function.
- Directory-based sharding uses an explicit mapping structure.
- Consistent hashing aims to reduce data movement when nodes change.
- Query routing directs requests to the appropriate shards.
- Queries without the shard key may require scatter-gather execution.
- Cross-shard joins and distributed transactions introduce additional complexity.
- Data skew and hotspots can create performance bottlenecks.
- Rebalancing redistributes data as the system grows.
- Sharding and replication can work together.
- Sharding should be introduced only when simpler scaling approaches are insufficient.

## 23. Next Learning Step

The next topic is **High Availability**.

Sharding explains how data can be distributed across multiple database nodes.

High availability focuses on keeping the database service operational when servers, networks, storage systems, or other components fail.

We will study availability targets, redundancy, failure detection, failover, recovery, and the architectural mechanisms that keep database services running.

**Next file:** `04 - Database Architecture/06 - High Availability.md`

Related notes:

- [[02 - Database Scalability]]
- [[03 - Replication]]
- [[04 - Partitioning]]
- [[06 - High Availability]]
- [[07 - Backup and Disaster Recovery]]
- [[01 - Data Architecture Overview]]
