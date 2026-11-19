
---
title: "Database Scalability"
aliases:
  - "Database Scaling"
  - "Horizontal and Vertical Scaling"
  - "Database Performance and Scalability"
type: learning-note
domain: technology
category: database-architecture
subject: database-scalability
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - database-architecture
  - scalability
  - performance
  - horizontal-scaling
  - vertical-scaling
  - distributed-systems
  - postgresql
  - load-balancing
  - connection-pooling
concepts:
  - scalability
  - vertical-scaling
  - horizontal-scaling
  - scale-up
  - scale-out
  - workload
  - throughput
  - latency
  - bottleneck
  - resource-utilization
  - load-balancing
  - caching
  - connection-pooling
  - read-scaling
  - write-scaling
  - elasticity
  - capacity-planning
  - performance-optimization
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Client Server Architecture]]"
  - "[[05 - Database Architecture Fundamentals]]"
  - "[[06 - Concurrency and Locks]]"
related:
  - "[[03 - Replication]]"
  - "[[04 - Partitioning]]"
  - "[[05 - Sharding]]"
  - "[[06 - High Availability]]"
  - "[[07 - Backup and Disaster Recovery]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"
  - "[[02 - How a Query is Executed]]"
tools:
  - PostgreSQL
  - SQL
  - Python
  - Docker
  - PgBouncer
  - Linux
  - Monitoring tools
learning_objectives:
  - Define scalability in database architecture.
  - Distinguish vertical and horizontal scaling.
  - Understand throughput, latency, and workload.
  - Identify common database bottlenecks.
  - Explain read scaling and write scaling.
  - Understand the role of caching and connection pooling.
  - Evaluate scalability trade-offs.
  - Explain capacity planning and elasticity.
  - Apply scaling strategies to a relational database application.
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 02 — Database Scalability

## 1. Introduction to Database Scalability

A database application may begin with a small number of users and a relatively limited amount of information.

As the application grows, its requirements change.

For example:

- More users access the application.
- More transactions are executed.
- More records are stored.
- Queries become more complex.
- Reports process larger datasets.
- More applications depend on the same database.

A system that performs well with ten users may experience performance problems with ten thousand users.

The ability to accommodate increasing demand is known as **scalability**.

### 1.1 What is scalability?

Scalability is the ability of a system to handle increasing workloads by adding or improving resources while maintaining acceptable performance and reliability.

In database architecture, scalability involves the capacity to support growth in:

- Concurrent users.
- Database connections.
- Read operations.
- Write operations.
- Stored data.
- Transaction volume.
- Query complexity.
- Data-processing requirements.

Scalability is not simply about making a server more powerful.

It involves architectural decisions about how workloads are distributed and managed.

### 1.2 A simple example

Imagine a vehicle management application.

Initially:

```text
Users: 5
Vehicles: 200
Daily transactions: 30
Database size: 50 MB
```

After several years:

```text
Users: 500
Vehicles: 50,000
Daily transactions: 20,000
Database size: 100 GB
```

The original architecture may no longer be sufficient.

The system may need:

- More processing power.
- Additional memory.
- Faster storage.
- Better indexing.
- Connection pooling.
- Caching.
- Read replicas.
- Data partitioning.
- Distributed database architecture.

The appropriate solution depends on which resource or component has become the limiting factor.

## 2. Scalability vs. Performance

Scalability and performance are related, but they are not the same.

### 2.1 Performance

Performance describes how efficiently a system performs under a particular workload.

Examples include:

- How long a query takes.
- How many requests are processed per second.
- How much memory is consumed.
- How much CPU time is required.
- How quickly a transaction completes.

### 2.2 Scalability

Scalability describes how system performance changes as workload or resources increase.

A system can have excellent performance with a small workload but poor scalability when demand increases.

For example:

```text
Workload: 10 users
Average response time: 50 ms

Workload: 1,000 users
Average response time: 2,000 ms
```

The system performs well initially but degrades significantly as demand grows.

### 2.3 Comparison

| Characteristic | Performance | Scalability |
|---|---|---|
| Main question | How fast does the system operate? | How well does it handle growth? |
| Focus | Efficiency under a workload. | Behavior as workload or resources increase. |
| Typical metric | Latency, throughput, CPU time. | Throughput and latency as load increases. |
| Optimization | Query tuning, indexes, memory. | Resource expansion, distribution, architecture. |

A scalable architecture should maintain acceptable performance as demand grows, although perfect linear scaling is rarely achievable.

## 3. Understanding Workload

Before selecting a scaling strategy, it is necessary to understand the workload.

A workload describes the operations performed by a system over a period of time.

For databases, workloads may contain:

- SELECT statements.
- INSERT statements.
- UPDATE statements.
- DELETE statements.
- Analytical queries.
- Background jobs.
- Data imports.
- Scheduled reports.

### 3.1 Read-heavy workloads

A read-heavy workload performs substantially more reads than writes.

Example:

```text
SELECT: 95%
INSERT: 2%
UPDATE: 2%
DELETE: 1%
```

Possible characteristics:

- Many users consult information.
- Dashboards retrieve records frequently.
- Reports are generated repeatedly.
- Data changes relatively infrequently.

Common scaling strategies include:

- Read replicas.
- Caching.
- Query optimization.
- Indexing.
- Materialized views.

### 3.2 Write-heavy workloads

A write-heavy workload performs a large number of data modifications.

Example:

```text
SELECT: 30%
INSERT: 40%
UPDATE: 25%
DELETE: 5%
```

Possible characteristics:

- Frequent transaction registration.
- Continuous event ingestion.
- Large numbers of updates.
- High-volume application activity.

Common scaling strategies include:

- Efficient transaction design.
- Appropriate indexing.
- Batch processing.
- Optimized storage.
- Partitioning.
- Write distribution where supported by the architecture.

Write scaling is generally more complex than read scaling because multiple writes must preserve consistency and coordinate changes to shared data.

### 3.3 Mixed workloads

Most business applications combine reads and writes.

For example, a vehicle management system may simultaneously:

- Display inventory records.
- Register mileage.
- Update maintenance status.
- Generate reports.
- Import information from external sources.

The architecture must accommodate these different operations without allowing one workload to excessively interfere with another.

## 4. Core Metrics for Scalability

Scalability decisions should be based on measurable indicators.

The most important metrics include latency, throughput, concurrency, resource utilization, and error rates.

### 4.1 Latency

Latency measures the time required to complete an operation or respond to a request.

Example:

```text
Query execution time: 120 ms
API response time: 250 ms
```

These values measure different parts of the request lifecycle.

Database query latency does not necessarily represent total application response time.

### 4.2 Throughput

Throughput measures how much work a system completes within a given period.

Examples:

- Queries per second.
- Transactions per second.
- Requests per second.
- Records processed per minute.

A database processing 500 transactions per second has a throughput of:

```text
500 transactions/second
```

Throughput depends on the workload and operating conditions.

### 4.3 Concurrency

Concurrency describes the number of operations that are active or overlapping during a period.

For example:

```text
Active database sessions: 100
Concurrent transactions: 40
```

These values are not necessarily equal.

A database session can exist without executing an active transaction.

### 4.4 Resource utilization

Resource utilization measures how much of a system resource is being consumed.

Important resources include:

| Resource | What to monitor |
|---|---|
| CPU | Processing utilization and saturation. |
| RAM | Memory consumption and pressure. |
| Disk | Capacity, IOPS, throughput, and latency. |
| Network | Bandwidth, latency, and packet loss. |
| Connections | Active, idle, and waiting connections. |
| Storage | Database size and growth rate. |

### 4.5 Error rate

Error rate measures the proportion of operations that fail.

Examples:

- Failed database connections.
- Query timeouts.
- Deadlocks.
- Serialization failures.
- Application errors.
- Request failures.

A system that processes many requests but frequently fails is not necessarily scaling successfully.

### 4.6 Percentile latency

Average response time can hide important performance problems.

Consider these request times:

```text
10 ms
15 ms
20 ms
25 ms
900 ms
```

The average may appear acceptable, but one request experienced a significant delay.

Percentiles provide additional insight.

| Metric | Meaning |
|---|---|
| p50 | 50% of requests complete within this time. |
| p90 | 90% of requests complete within this time. |
| p95 | 95% of requests complete within this time. |
| p99 | 99% of requests complete within this time. |

For example:

```text
p50 = 40 ms
p95 = 180 ms
p99 = 900 ms
```

This indicates that most requests are relatively fast, but a small percentage experience substantial delays.

Percentile metrics are especially useful for systems with variable workloads.

## 5. Vertical Scaling (Scale Up)

Vertical scaling means increasing the resources of an existing server.

Instead of adding more database servers, the existing database server receives more computing capacity.

### 5.1 How it works

```text
BEFORE

+-------------------------+
|     DATABASE SERVER     |
|                         |
|  4 CPU Cores            |
|  8 GB RAM               |
|  100 GB Storage         |
+-------------------------+


AFTER

+-------------------------+
|     DATABASE SERVER     |
|                         |
|  16 CPU Cores           |
|  64 GB RAM              |
|  1 TB Storage           |
+-------------------------+
```

The database remains on one server, but the available resources increase.

### 5.2 Resources that can be increased

Vertical scaling may involve:

- More CPU cores.
- Faster processors.
- More RAM.
- Faster SSD storage.
- Higher storage throughput.
- More network capacity.

### 5.3 Advantages

1. Relatively simple architecture.
2. No immediate need to distribute database writes.
3. Fewer network communication paths.
4. Existing application connections may remain largely unchanged.
5. Often easier to operate than a distributed database.

### 5.4 Disadvantages

1. Hardware capacity has limits.
2. More powerful resources can be expensive.
3. Upgrades may require downtime.
4. A single database server can remain a failure point.
5. Performance improvements eventually become less cost-effective.

### 5.5 When is vertical scaling appropriate?

It is often useful when:

- The database is relatively small or medium-sized.
- The workload is concentrated on one database.
- A server is approaching resource limits.
- The application does not yet require distributed database architecture.
- Operational simplicity is important.

Vertical scaling is frequently a practical first step before introducing more complex distribution mechanisms.

## 6. Horizontal Scaling (Scale Out)

Horizontal scaling means adding additional machines or service instances to distribute workload.

Instead of making one server more powerful, the architecture uses multiple servers.

### 6.1 How it works

```text
BEFORE

             Application
                  |
                  v
          +---------------+
          | Database      |
          | Server        |
          +---------------+


AFTER

             Application
                  |
          +-------+-------+
          |       |       |
          v       v       v
       Server  Server  Server
          1       2       3
```

The architecture distributes operations across multiple resources.

However, horizontal scaling can mean different things depending on the component being scaled.

For example:

- Multiple application servers.
- Multiple database read replicas.
- Multiple database shards.
- Distributed database nodes.

Adding application servers does not automatically distribute database writes.

### 6.2 Advantages

- More capacity through additional machines.
- Potentially better fault isolation.
- Independent resource expansion.
- Greater flexibility for large workloads.
- Can support geographically distributed systems.

### 6.3 Disadvantages

- More architectural complexity.
- Additional network communication.
- Data consistency challenges.
- More difficult monitoring.
- More complicated deployment and recovery.
- Potential coordination overhead.

### 6.4 When is horizontal scaling appropriate?

It may be appropriate when:

- One server cannot meet workload requirements.
- Application demand changes significantly.
- Read traffic needs distribution.
- Data volume exceeds practical single-server capacity.
- High availability requires redundant components.
- Workloads need independent scaling.

## 7. Vertical vs. Horizontal Scaling

| Characteristic | Vertical scaling | Horizontal scaling |
|---|---|---|
| Strategy | Increase resources on one machine. | Add machines or instances. |
| Common term | Scale up. | Scale out. |
| Complexity | Usually lower. | Usually higher. |
| Hardware limit | Limited by individual machine capacity. | Capacity can grow by adding resources. |
| Data distribution | Not inherently required. | Often required for distributed database workloads. |
| Cost | Depends on hardware and upgrade path. | Depends on number and type of additional resources. |
| Operational effort | Often simpler. | Usually requires more coordination. |

Neither approach is universally superior.

Many production architectures combine both.

For example, a system may use a powerful primary database server and multiple read replicas.

## 8. Scaling Application Servers vs. Scaling Databases

An important architectural distinction is that the application layer and database layer can be scaled independently.

### 8.1 Application scaling

Suppose an application has one backend server.

As the number of users increases, the application server becomes overloaded.

The architecture may add additional application instances:

```text
             Clients
                |
                v
         Load Balancer
                |
       +--------+--------+
       |        |        |
       v        v        v
    Backend Backend Backend
       1        2        3
                |
                v
           PostgreSQL
```

The load balancer distributes requests across backend instances.

The database may remain on one server.

### 8.2 Database scaling

The database may independently become the bottleneck.

For example:

- Too many queries.
- Excessive write activity.
- Insufficient memory.
- Slow storage.
- Lock contention.
- Connection exhaustion.

The solution must address the database bottleneck rather than simply adding more application servers.

### 8.3 Important principle

Scaling one layer does not automatically scale every other layer.

A system may have:

- Ten application servers.
- One overloaded database server.

In that situation, the database remains the limiting component.

## 9. Load Balancing

A load balancer distributes incoming requests across multiple available servers.

It is commonly used when an application has several backend instances.

### 9.1 Basic architecture

```text
               Clients
                  |
                  v
           +-------------+
           | Load        |
           | Balancer    |
           +------+------+
                  |
        +---------+---------+
        |         |         |
        v         v         v
     Server A  Server B  Server C
```

### 9.2 Common load-balancing strategies

| Strategy | Description |
|---|---|
| Round Robin | Distributes requests sequentially across servers. |
| Least Connections | Selects a server with fewer active connections. |
| Weighted Round Robin | Assigns more requests to servers with greater capacity. |
| IP Hash | Uses client IP information to select a server. |
| Least Response Time | Considers response time when selecting a server. |

The availability of these strategies depends on the load-balancing technology.

### 9.3 Database load balancing

Database load balancing requires additional care.

A system cannot always distribute every database operation across servers arbitrarily.

For example:

- A read may be served by a replica.
- A write generally must go to the appropriate writable node.
- A replica may contain slightly older data.
- Transaction consistency may require reading from the primary.

Database routing must understand the consistency requirements of each operation.

## 10. Read Scaling and Write Scaling

One of the most important challenges in database architecture is handling read and write workloads separately.

### 10.1 Read scaling

Read scaling increases the system's ability to handle SELECT operations.

Common techniques include:

- Read replicas.
- Caching.
- Materialized views.
- Query optimization.
- Indexing.
- Read-oriented database models.

Example:

```text
                 Application
                      |
               Read/Write Router
                 /          \
                /            \
               v              v
         Primary DB       Read Replica
         Reads/Writes      Reads
```

The primary database handles modifications.

The read replica handles selected read operations.

Replication is explored in the next note.

### 10.2 Write scaling

Write scaling increases the system's ability to handle INSERT, UPDATE, and DELETE operations.

It is more challenging because writes may affect shared data and must preserve transactional consistency.

Possible strategies include:

- Improving query efficiency.
- Reducing unnecessary indexes.
- Batch processing.
- Partitioning.
- Sharding.
- Distributing independent workloads.
- Separating transactional and analytical processing.

A write-heavy workload may require architectural changes beyond simply adding more database connections.

### 10.3 Read vs. write scaling

| Characteristic | Read scaling | Write scaling |
|---|---|---|
| Main objective | Handle more read requests. | Handle more modifications. |
| Common technique | Replication and caching. | Partitioning and sharding. |
| Complexity | Often lower. | Often higher. |
| Main challenge | Data freshness and consistency. | Coordination and transaction consistency. |

## 11. Caching as a Scalability Strategy

Caching stores frequently requested information in a faster-access location.

Instead of retrieving the same information repeatedly from the primary database, an application can reuse a cached result.

### 11.1 How caching works

Without caching:

```text
Request 1 → Database Query
Request 2 → Database Query
Request 3 → Database Query
Request 4 → Database Query
```

With caching:

```text
Request 1 → Database → Cache
Request 2 → Cache
Request 3 → Cache
Request 4 → Cache
```

The cache reduces repeated database access when cached information can safely be reused.

### 11.2 Cache hit and cache miss

A cache hit occurs when the requested information is already available in the cache.

A cache miss occurs when the information is not available and must be retrieved from another source.

Example:

```text
Application requests vehicle 101
              |
              v
         Check Cache
              |
        +-----+-----+
        |           |
        v           v
      HIT         MISS
        |           |
        v           v
   Return data   Query database
                    |
                    v
               Update cache
                    |
                    v
               Return data
```

### 11.3 Common caching technologies

Examples include:

- Redis.
- Memcached.
- In-memory application caches.
- HTTP caching mechanisms.
- CDN caching for suitable content.

### 11.4 Cache invalidation

One of the most difficult aspects of caching is keeping cached information synchronized with the source of truth.

Suppose a vehicle's status changes from:

```text
AVAILABLE
```

to:

```text
IN_MAINTENANCE
```

If the cache still contains the previous value, users may see outdated information.

Common cache-management strategies include:

- Time-to-live (TTL).
- Explicit invalidation.
- Write-through caching.
- Cache-aside.
- Event-driven invalidation.

### 11.5 Cache-aside pattern

A common strategy is cache-aside.

The application:

1. Checks the cache.
2. Retrieves the information from the database if necessary.
3. Stores the result in the cache.
4. Returns the information.

The application controls when the cache is read and updated.

Caching should be used when its benefits justify the complexity of managing stale data.

## 12. Connection Pooling and Scalability

Connection pooling is especially important when the number of application requests grows.

A database connection requires server resources.

If every incoming request creates a new connection, the database may become overloaded.

### 12.1 Connection exhaustion

Suppose an application has 1,000 simultaneous requests.

If every request attempts to create its own database connection, the database may receive far more connections than it can efficiently manage.

Possible consequences:

- High memory consumption.
- Increased process overhead.
- Connection failures.
- Reduced query performance.
- Increased context switching.

### 12.2 Connection pooling

A pool limits the number of reusable connections.

```text
           1,000 Application Requests
                      |
                      v
             Connection Pool
                      |
            +---------+---------+
            |         |         |
            v         v         v
         Conn. 1   Conn. 2   Conn. 3
            |         |         |
            +---------+---------+
                      |
                      v
                 PostgreSQL
```

The application requests a connection when it needs one.

When the operation finishes, the connection returns to the pool.

### 12.3 Pool sizing

Pool size should consider:

- Available database resources.
- Number of application instances.
- Expected concurrency.
- Query duration.
- Transaction duration.
- Other applications using the same database.

For example:

```text
Application instances: 4
Maximum pool per instance: 20

Potential application connections:
4 × 20 = 80
```

This illustrates why pool limits must be considered across the entire deployment, not just within one application instance.

More connections do not always produce better performance.

## 13. Bottlenecks and the Bottleneck Principle

A bottleneck is a component that limits the overall performance or capacity of a system.

A system can have powerful hardware but still perform poorly because one component cannot handle the workload.

### 13.1 Common database bottlenecks

| Bottleneck | Possible symptoms |
|---|---|
| CPU | High processor utilization and slow query execution. |
| Memory | Memory pressure and increased disk activity. |
| Storage | High I/O latency and slow reads or writes. |
| Network | Slow communication between components. |
| Connections | Requests waiting for database connections. |
| Locks | Transactions blocked by concurrent operations. |
| Poor query plans | Excessive scans or inefficient joins. |
| Missing indexes | Large amounts of unnecessary data access. |

### 13.2 Example

Consider this application:

```text
Browser
   |
   v
Load Balancer
   |
   v
Four Application Servers
   |
   v
One PostgreSQL Server
```

The application servers are operating normally.

However, PostgreSQL is experiencing:

- High CPU usage.
- Many slow queries.
- Frequent lock waits.
- Excessive connection activity.

Adding more application servers may make the problem worse because they can generate even more database requests.

The correct response is to identify and address the database bottleneck.

### 13.3 Measure before scaling

Before adding infrastructure, investigate:

1. Which resource is saturated?
2. Which queries consume the most resources?
3. Are indexes appropriate?
4. Are transactions unnecessarily long?
5. Are connections being reused?
6. Is the workload read-heavy or write-heavy?
7. Can requests be cached?
8. Can the workload be separated?

Scaling without diagnosing the bottleneck can increase cost without solving the problem.

## 14. Elasticity and Capacity Planning

Scalability is closely related to elasticity and capacity planning.

### 14.1 Elasticity

Elasticity is the ability to increase or decrease resources according to changing demand.

For example:

```text
Normal demand:
2 application instances

Peak demand:
8 application instances

After peak:
2 application instances
```

An elastic system can adjust resources as demand changes.

This is especially useful in cloud environments.

### 14.2 Capacity planning

Capacity planning is the process of estimating the resources needed to support expected workloads.

It considers:

- Current utilization.
- Historical workload.
- Growth forecasts.
- Peak demand.
- Storage growth.
- Transaction volume.
- Hardware limitations.
- Budget constraints.

### 14.3 Example: Database storage growth

Suppose a database currently occupies 100 GB.

Its average monthly growth is 10 GB.

A simple linear projection:

```text
Current size: 100 GB
Monthly growth: 10 GB
Period: 12 months

Projected size:
100 + (10 × 12) = 220 GB
```

This is a basic estimate.

Actual growth may vary due to workload changes, maintenance, indexes, deleted records, and storage-management behavior.

### 14.4 Capacity planning is continuous

Capacity planning should be reviewed regularly.

A database architecture that meets today's requirements may become insufficient as:

- The number of users increases.
- New application modules are introduced.
- Data retention requirements change.
- Reporting workloads become heavier.
- New integrations generate additional traffic.

## 15. Scaling Strategies in PostgreSQL

PostgreSQL can support several scaling strategies.

Some improve the performance of a single database server.

Others introduce additional components or distribute data.

### 15.1 Optimize before distributing

A sensible optimization sequence often begins with:

1. Analyze query execution plans.
2. Identify slow queries.
3. Create appropriate indexes.
4. Improve SQL statements.
5. Review transaction design.
6. Configure memory and storage appropriately.
7. Monitor connection usage.
8. Evaluate connection pooling.
9. Consider caching.
10. Evaluate architectural distribution.

The sequence is not mandatory for every system, but it encourages solving simpler bottlenecks before introducing more complex infrastructure.

### 15.2 Vertical scaling PostgreSQL

A PostgreSQL server can benefit from:

- More CPU resources.
- More available memory.
- Faster storage.
- Higher I/O capacity.
- Appropriate configuration.

PostgreSQL settings such as `shared_buffers`, `work_mem`, and `max_connections` influence resource management.

They must be configured according to the actual environment and workload.

Increasing a configuration value without understanding its impact can make performance worse.

### 15.3 Read replicas

PostgreSQL supports physical and logical replication mechanisms.

Read replicas can serve selected queries, reducing the read workload on a primary server.

The following note will examine replication in detail.

### 15.4 Partitioning

Partitioning divides a large logical table into smaller physical partitions.

It can improve data organization and make certain queries and maintenance operations more efficient.

Partitioning does not automatically distribute data across multiple servers.

It can operate within a single PostgreSQL database instance.

### 15.5 Sharding

Sharding distributes data across multiple independent database nodes.

Each node stores a portion of the total dataset.

It can support large-scale horizontal data distribution, but it introduces additional complexity.

The relationship between replication, partitioning, and sharding is important:

| Technique | Main purpose |
|---|---|
| Replication | Maintain copies of data across database instances. |
| Partitioning | Divide a table into smaller parts. |
| Sharding | Distribute portions of a dataset across multiple database nodes. |

These techniques solve different problems and may be combined.

## 16. Practical Laboratory — Understanding Database Workload

The objective is to measure how a database behaves under a simple workload.

We will use PostgreSQL and SQL.

### 16.1 Create a test table

```sql
DROP TABLE IF EXISTS scalability_lab;

CREATE TABLE scalability_lab (
    record_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    category INTEGER NOT NULL,
    description TEXT NOT NULL,
    amount NUMERIC(12,2) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 16.2 Insert test records

```sql
INSERT INTO scalability_lab (
    category,
    description,
    amount
)
SELECT
    (random() * 20)::INTEGER,
    'Test record ' || generate_series,
    (random() * 10000)::NUMERIC(12,2)
FROM generate_series(1, 100000);
```

This generates 100,000 test records.

### 16.3 Measure table size

```sql
SELECT
    pg_size_pretty(
        pg_total_relation_size('scalability_lab')
    ) AS total_size;
```

This returns the total storage associated with the table, including indexes and related storage.

### 16.4 Analyze a query without an index

```sql
EXPLAIN ANALYZE
SELECT *
FROM scalability_lab
WHERE category = 10;
```

Observe:

- Execution time.
- Scan method.
- Number of rows returned.
- Estimated versus actual rows.

The query may use a sequential scan depending on the data distribution and planner estimates.

### 16.5 Create an index

```sql
CREATE INDEX idx_scalability_lab_category
ON scalability_lab(category);
```

Run the query again:

```sql
EXPLAIN ANALYZE
SELECT *
FROM scalability_lab
WHERE category = 10;
```

Compare the execution plans.

An index does not guarantee that PostgreSQL will use an index scan.

The optimizer chooses a plan based on estimated cost and available statistics.

For a query returning a large proportion of the table, a sequential scan may still be more efficient.

### 16.6 Examine query statistics

If `pg_stat_statements` is installed and configured, it can help identify queries that consume substantial database resources.

Example:

```sql
SELECT
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

The extension must be enabled and configured before this view is available.

### 16.7 What should you learn?

Observe how:

- Table size changes as records are inserted.
- Indexes affect query plans.
- Execution time varies.
- PostgreSQL chooses between scan methods.
- Query optimization can improve capacity without adding hardware.

The main objective is to understand that database scalability begins with understanding the workload and the resources it consumes.

## 17. Applying Scalability to SIAPV

A vehicle management system with approximately 200 units does not necessarily require distributed database architecture.

However, understanding scalability helps design the system so it can grow without unnecessary redesign.

### 17.1 Current workload characteristics

A system like SIAPV may involve:

- Inventory searches.
- Vehicle detail consultations.
- Mileage updates.
- Maintenance registrations.
- Operational reports.
- Administrative record changes.

These operations create a mixed workload.

Some are read-heavy, while others modify important operational records.

### 17.2 A possible growth scenario

Imagine the application expands:

```text
Initial system:
200 units
10 users
100 daily requests

Expanded system:
10,000 units
500 users
50,000 daily requests
```

This is a hypothetical scenario, not a measurement of SIAPV.

The architecture may need to evolve.

### 17.3 Potential evolution

**Stage 1 — Optimize the existing database**

- Review indexes.
- Improve SQL queries.
- Validate constraints.
- Optimize transaction boundaries.
- Monitor PostgreSQL.

**Stage 2 — Improve application infrastructure**

- Introduce a dedicated backend API if necessary.
- Implement connection pooling.
- Improve request handling.
- Add monitoring and logging.

**Stage 3 — Reduce repeated reads**

- Introduce caching for suitable information.
- Optimize reports.
- Use materialized views where appropriate.
- Evaluate read replicas if read traffic justifies them.

**Stage 4 — Distribute the workload**

- Evaluate partitioning for large tables.
- Introduce replication.
- Consider sharding only when justified by actual capacity requirements.

### 17.4 Architectural decision

For a relatively small fleet management system, a well-optimized PostgreSQL instance may be sufficient.

The goal is not to use the most complex architecture.

The goal is to select the simplest architecture that satisfies current and reasonably expected requirements.

## 18. Common Scalability Mistakes

Avoid these assumptions:

### Mistake 1: More servers always mean better performance

Additional servers help only when the workload can be distributed effectively.

### Mistake 2: More connections mean more throughput

Too many database connections can increase resource consumption and reduce performance.

### Mistake 3: Every read should go to a replica

Replication introduces data-freshness considerations.

Some reads require the latest committed data from the primary.

### Mistake 4: Caching solves every performance problem

Caching can reduce repeated reads, but it does not fix inefficient writes, poor transaction design, or incorrect query plans.

### Mistake 5: Sharding should be introduced early

Sharding introduces routing, distribution, operational, and consistency challenges.

It should be justified by workload and capacity requirements.

### Mistake 6: Scaling eliminates the need for optimization

A poorly designed query remains inefficient even on powerful hardware.

### Mistake 7: Average latency is sufficient

Average values can hide slow requests.

Percentiles and workload-specific measurements provide a better picture.

## 19. Knowledge Check

Answer these questions without consulting the previous sections.

1. What is database scalability?
2. How does scalability differ from performance?
3. What is a workload?
4. What characterizes a read-heavy workload?
5. What characterizes a write-heavy workload?
6. What is latency?
7. What is throughput?
8. What does p95 latency represent?
9. What is vertical scaling?
10. What is horizontal scaling?
11. Why can horizontal scaling introduce additional complexity?
12. How does application scaling differ from database scaling?
13. What is the role of a load balancer?
14. Why is read scaling generally easier than write scaling?
15. What is caching?
16. What is the difference between a cache hit and a cache miss?
17. Why is cache invalidation difficult?
18. How does connection pooling improve resource utilization?
19. What is a database bottleneck?
20. Why should workload measurements precede infrastructure changes?
21. What is elasticity?
22. What is capacity planning?
23. How can PostgreSQL benefit from vertical scaling?
24. What is the difference between replication, partitioning, and sharding?
25. Why is sharding not always the best solution?

## 20. Practical Exercises

### Exercise 1 — Choose a scaling strategy

For each scenario, recommend a strategy and explain your reasoning.

A. A PostgreSQL server has sufficient memory but insufficient CPU capacity.

B. A reporting application performs thousands of repeated reads.

C. An application has many backend instances but only one overloaded database.

D. A system stores billions of records and needs to distribute data across multiple database nodes.

E. A system receives unpredictable demand during certain periods.

### Exercise 2 — Analyze a workload

Consider this workload:

```text
SELECT: 80%
INSERT: 10%
UPDATE: 8%
DELETE: 2%
```

Answer:

1. Is the workload read-heavy or write-heavy?
2. Which scaling techniques might be appropriate?
3. Would read replicas help?
4. Could caching reduce database demand?
5. What consistency concerns should be considered?

### Exercise 3 — Identify the bottleneck

A database system has these characteristics:

```text
CPU utilization: 95%
RAM utilization: 55%
Disk utilization: 40%
Database connections: 80% of configured maximum
Average query latency: 900 ms
```

Discuss:

- Which resource appears most constrained?
- What additional metrics would you inspect?
- Which queries should be investigated?
- Would adding RAM necessarily solve the problem?
- Would adding application servers help?

Do not assume one metric is sufficient to identify the root cause.

### Exercise 4 — Design a growth strategy

Imagine SIAPV expands from approximately 200 units to 20,000 units and becomes available to several administrative areas.

Design an incremental scalability strategy.

Include:

- Database optimization.
- Application architecture.
- Connection management.
- Read scaling.
- Storage organization.
- Monitoring.
- Criteria for introducing distributed architecture.

### Exercise 5 — Capacity estimation

A database currently occupies 80 GB and grows by an average of 6 GB per month.

Calculate its estimated size after 18 months using a simple linear projection.

Then explain why the actual storage requirement may differ.

## 21. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Scalability | Escalabilidad |
| Performance | Rendimiento |
| Workload | Carga de trabajo |
| Latency | Latencia |
| Throughput | Rendimiento por unidad de tiempo |
| Concurrency | Concurrencia |
| Resource utilization | Utilización de recursos |
| Vertical scaling | Escalabilidad vertical |
| Horizontal scaling | Escalabilidad horizontal |
| Scale up | Aumentar recursos de una máquina |
| Scale out | Añadir máquinas o instancias |
| Load balancer | Balanceador de carga |
| Read scaling | Escalamiento de lecturas |
| Write scaling | Escalamiento de escrituras |
| Cache | Caché |
| Cache hit | Acierto de caché |
| Cache miss | Fallo de caché |
| Cache invalidation | Invalidación de caché |
| Connection pooling | Agrupación de conexiones |
| Bottleneck | Cuello de botella |
| Resource saturation | Saturación de recursos |
| Elasticity | Elasticidad |
| Capacity planning | Planeación de capacidad |
| Read replica | Réplica de lectura |
| Query optimization | Optimización de consultas |
| Distributed system | Sistema distribuido |
| Fault isolation | Aislamiento de fallos |
| Response time | Tiempo de respuesta |
| Percentile latency | Latencia por percentil |
| Database growth | Crecimiento de la base de datos |
| Horizontal distribution | Distribución horizontal |

## 22. Final Mental Model

Scalability is the ability to accommodate increasing workloads while maintaining acceptable performance and reliability.

A database architecture can scale through several complementary strategies:

```text
                DATABASE SCALABILITY
                         |
          +--------------+--------------+
          |                             |
          v                             v
    VERTICAL SCALING             HORIZONTAL SCALING
          |                             |
          v                             v
    More CPU / RAM               More Servers
    Faster Storage               More Instances
                                      |
                         +------------+------------+
                         |                         |
                         v                         v
                    READ SCALING              DATA DISTRIBUTION
                         |                         |
                         v                         v
                    Replication          Partitioning / Sharding
                         |
                         v
                  Caching and Routing
```

A good scalability strategy begins by understanding the workload, measuring system behavior, identifying bottlenecks, and selecting an appropriate solution.

**The central architectural principle is to scale the component that limits the system, not simply add resources everywhere.**

---

## 23. Related Notes

### Previous concepts

- [[01 - Client Server Architecture]]
- [[05 - Database Architecture Fundamentals]]
- [[06 - Concurrency and Locks]]
- [[02 - How a Query is Executed]]

### Next topics

- [[03 - Replication]]
- [[04 - Partitioning]]
- [[05 - Sharding]]
- [[06 - High Availability]]
- [[07 - Backup and Disaster Recovery]]

### Practical technologies

- PostgreSQL
- PgBouncer
- Redis
- Docker
- Python
- SQL
- Monitoring and observability tools

---

## 24. Completion Criteria

- [ ] Define database scalability.
- [ ] Distinguish scalability from performance.
- [ ] Analyze read-heavy, write-heavy, and mixed workloads.
- [ ] Understand latency, throughput, concurrency, and percentiles.
- [ ] Explain vertical scaling and horizontal scaling.
- [ ] Distinguish application scaling from database scaling.
- [ ] Explain load balancing.
- [ ] Compare read scaling and write scaling.
- [ ] Explain caching and cache invalidation.
- [ ] Understand connection pooling.
- [ ] Identify common database bottlenecks.
- [ ] Explain elasticity and capacity planning.
- [ ] Identify PostgreSQL scaling strategies.
- [ ] Design an incremental growth strategy for a relational database application.
