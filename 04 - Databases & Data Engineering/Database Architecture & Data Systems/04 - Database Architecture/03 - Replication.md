
---
title: "Database Replication"
aliases:
  - "PostgreSQL Replication"
  - "Primary and Standby Databases"
  - "Streaming Replication"
type: learning-note
domain: technology
category: database-architecture
subject: database-replication
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - replication
  - postgresql
  - streaming-replication
  - logical-replication
  - physical-replication
  - wal
  - high-availability
  - disaster-recovery
  - distributed-systems

concepts:
  - database-replication
  - primary-server
  - standby-server
  - synchronous-replication
  - asynchronous-replication
  - physical-replication
  - logical-replication
  - write-ahead-log
  - replication-lag
  - replication-slot
  - failover
  - switchover
  - log-sequence-number
  - read-replica

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Client Server Architecture]]"
  - "[[02 - Database Scalability]]"
  - "[[05 - Transactions and ACID]]"

related:
  - "[[04 - Partitioning]]"
  - "[[05 - Sharding]]"
  - "[[06 - High Availability]]"
  - "[[07 - Backup and Disaster Recovery]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[02 - PostgreSQL Processes]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"

tools:
  - PostgreSQL
  - SQL
  - psql
  - pgAdmin
  - Docker
  - pg_basebackup
  - pg_ctl

learning_objectives:
  - "Explain database replication and its architectural purposes."
  - "Distinguish primary and standby database roles."
  - "Compare synchronous and asynchronous replication."
  - "Explain physical and logical replication in PostgreSQL."
  - "Understand WAL, LSN, replication slots, and replication lag."
  - "Interpret basic replication monitoring queries."
  - "Understand failover, switchover, and split-brain risks."
  - "Evaluate replication trade-offs in a database architecture."

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 03 — Database Replication

## 1. What Is Database Replication?

**Database replication** is the process of maintaining copies of data across multiple database servers or instances.

Instead of storing all database information in a single server, an architecture can maintain additional servers that receive and reproduce changes from another database.

The objective is to make data available in more than one location.

A simplified architecture looks like this:

```text
                APPLICATION
                     |
                     v
              PRIMARY DATABASE
                     |
             Replication Process
                     |
          +----------+----------+
          |                     |
          v                     v
    STANDBY SERVER A      STANDBY SERVER B
```

The primary database usually handles write operations, while standby servers receive changes and can serve other purposes depending on their configuration.

For example:

- The primary processes transactions.
- A standby maintains a synchronized copy.
- Another standby serves analytical read queries.
- A standby in another location provides an additional recovery option.

Replication is especially important in systems where database availability and continuity matter.

### 1.1 Why Is Replication Used?

Replication can support several architectural objectives.

| Objective | Explanation |
|---|---|
| High availability | Provides another database instance that can take over after a failure. |
| Read scalability | Distributes read workloads across multiple database servers. |
| Disaster recovery | Maintains a database copy in another server or location. |
| Geographic distribution | Places database copies closer to different application regions. |
| Migration | Helps move data between database environments. |
| Operational flexibility | Allows certain reporting or integration workloads to run on separate instances. |

However, replication does not automatically guarantee all these benefits.

Its effectiveness depends on configuration, monitoring, network reliability, recovery procedures, and application design.

---

## 2. Primary and Standby Architecture

A common replication architecture uses two different server roles.

### 2.1 Primary Database

The **primary database** is the main database instance responsible for accepting write operations.

Typical responsibilities:

- Insert records.
- Update existing records.
- Delete records.
- Execute transactions.
- Generate changes that can be replicated.
- Coordinate replication with standby servers.

Example:

```sql
INSERT INTO vehicles (
    inventory_number,
    brand,
    model
)
VALUES (
    'PK-001',
    'Nissan',
    'Frontier'
);
```

The transaction is executed on the primary database.

### 2.2 Standby Database

A **standby database** maintains a copy of the primary database by receiving and applying changes.

A physical standby in PostgreSQL operates in recovery mode and is generally read-only.

It can:

- Receive replicated changes.
- Replay database changes.
- Serve read-only queries when hot standby is enabled.
- Become the new primary through promotion.

Example architecture:

```text
                 APPLICATION
                      |
                      v
               PRIMARY DATABASE
                 READ / WRITE
                      |
                      | Replication
                      |
              +-------+-------+
              |               |
              v               v
          STANDBY 1        STANDBY 2
          READ ONLY        READ ONLY
```

A standby is not necessarily an independent writable database.

In PostgreSQL, a physical standby normally follows the primary's database state.

### 2.3 Common Terminology

Different technologies use different terms for similar roles.

| Term | Meaning |
|---|---|
| Primary | Main database instance that accepts writes. |
| Standby | Database instance receiving replicated changes. |
| Leader | Node coordinating write operations. |
| Follower | Node following changes from the leader. |
| Source | Database from which changes originate. |
| Replica | Copy of a database or selected data. |
| Read replica | Replica intended to serve read workloads. |

The terminology depends on the database technology and replication architecture.

---

## 3. Synchronous vs Asynchronous Replication

One of the most important replication decisions concerns when the primary considers a transaction committed.

There are two fundamental approaches.

### 3.1 Asynchronous Replication

With asynchronous replication, the primary does not wait for the standby to confirm that the latest transaction has been received or applied before acknowledging the commit to the client.

Conceptually:

```text
APPLICATION
    |
    | INSERT / UPDATE
    v
PRIMARY DATABASE
    |
    | Commit transaction
    |
    +----------------------> Client receives confirmation
    |
    | Send changes
    v
STANDBY DATABASE
    |
    | Receive and replay changes
    v
UPDATED REPLICA
```

The primary can continue processing transactions without waiting for the standby.

**Advantages:**

- Lower transaction latency.
- Less dependency on standby response time.
- Better tolerance of temporary standby or network delays.
- Useful for read replicas and geographically distant replicas.

**Disadvantages:**

- The standby may be behind the primary.
- A primary failure can cause the loss of recently committed transactions that were not replicated.
- Applications may read outdated information from a standby.

### 3.2 Synchronous Replication

With synchronous replication, the primary waits for acknowledgment from one or more configured synchronous standbys before completing a commit according to the selected acknowledgment level.

Conceptually:

```text
APPLICATION
    |
    | INSERT / UPDATE
    v
PRIMARY DATABASE
    |
    | Send WAL records
    v
SYNCHRONOUS STANDBY
    |
    | Acknowledge required stage
    v
PRIMARY DATABASE
    |
    | Commit acknowledgment
    v
APPLICATION
```

**Advantages:**

- Reduces the risk of losing recently committed transactions during certain primary failures.
- Provides stronger coordination between primary and standby.
- Can support recovery objectives requiring very low data loss.

**Disadvantages:**

- Higher transaction latency.
- Commit operations depend on standby responsiveness.
- Network failures can affect write availability.
- Configuration requires careful consideration of the number of synchronous standbys.

### 3.3 Comparison

| Characteristic | Asynchronous | Synchronous |
|---|---|---|
| Waits for standby acknowledgment | No | Yes, according to configuration |
| Commit latency | Usually lower | Usually higher |
| Risk of losing recent commits | Higher | Lower |
| Dependency on network response | Lower | Higher |
| Typical use | Read scaling and remote replicas | Stronger replication guarantees |

Neither approach is universally better.

The correct choice depends on the application's tolerance for data loss and transaction latency.

### 3.4 Understanding RPO

**Recovery Point Objective (RPO)** defines how much data loss an organization can tolerate after a failure.

For example:

- RPO = 0: the recovery strategy aims to avoid losing committed transactions.
- RPO = 5 minutes: the organization accepts that up to five minutes of data may be lost.

Asynchronous replication can have a non-zero RPO because changes may not have reached the standby before a failure.

Synchronous replication can reduce this risk, but the exact guarantee depends on what acknowledgment is required and the failure scenario.

### 3.5 PostgreSQL Synchronous Commit Settings

PostgreSQL provides the `synchronous_commit` parameter to control the acknowledgment requirements for synchronous replication.

Important values include:

| Setting | General behavior |
|---|---|
| `off` | Does not wait for local WAL flush before reporting commit success. |
| `local` | Waits for local WAL flush but does not wait for synchronous standby acknowledgment. |
| `remote_write` | Waits until the synchronous standby reports that WAL has been written to its operating system. |
| `on` | Waits for the synchronous standby to flush WAL to durable storage. |
| `remote_apply` | Waits until the synchronous standby has replayed the transaction. |

These settings have different durability and visibility implications.

The `synchronous_standby_names` parameter identifies which standby servers participate in synchronous replication.

For example, a configuration can require acknowledgment from one synchronous standby.

```ini
synchronous_standby_names = 'FIRST 1 (standby_a, standby_b)'
```

This example means that PostgreSQL selects one standby from the listed candidates according to the configured priority.

The exact configuration should be designed around the number of standbys, failure tolerance, and required commit guarantees.

---

## 4. Physical Replication

**Physical replication** reproduces changes at the database storage level.

In PostgreSQL, physical replication is based on the Write-Ahead Log (WAL).

Instead of transmitting individual SQL statements, PostgreSQL sends WAL records that allow another server to reproduce the corresponding database changes.

### 4.1 How Physical Replication Works

The general process is:

1. A transaction modifies data on the primary.
2. PostgreSQL generates WAL records.
3. The primary sends WAL information to the standby.
4. The standby receives the WAL records.
5. The standby replays the changes.
6. The standby database advances toward the primary's state.

```text
                 PRIMARY
                    |
                    v
              SQL TRANSACTION
                    |
                    v
               WAL RECORDS
                    |
                    v
              WAL SENDER
                    |
              NETWORK STREAM
                    |
                    v
              WAL RECEIVER
                    |
                    v
               WAL REPLAY
                    |
                    v
                 STANDBY
```

### 4.2 Important Components

| Component | Responsibility |
|---|---|
| WAL | Records changes needed for database recovery and replication. |
| WAL Sender | Sends WAL information from the primary. |
| WAL Receiver | Receives WAL information on the standby. |
| WAL Replay | Applies received changes to the standby. |
| LSN | Identifies positions within the WAL stream. |
| Replication Slot | Tracks the WAL position required by a replication consumer. |

These components are part of PostgreSQL's replication architecture.

### 4.3 Advantages of Physical Replication

- Efficient replication of database changes.
- Maintains a physical copy of the database.
- Suitable for standby servers and disaster recovery.
- Can replicate the entire database cluster.
- Supports read-only queries through hot standby.

### 4.4 Limitations

- It generally requires compatible PostgreSQL major-version configurations.
- It is not designed to replicate only selected tables.
- It reproduces physical database changes rather than providing a table-level integration interface.
- A physical standby follows the primary's database state.

Physical replication is commonly used for PostgreSQL streaming replication and high-availability architectures.

---

## 5. Logical Replication

**Logical replication** transfers data changes using a logical representation of database operations.

Instead of reproducing the entire physical database state, it allows selected tables and their changes to be replicated.

PostgreSQL implements logical replication through a publication and subscription model.

### 5.1 Publication

A **publication** defines which tables and operations are made available for replication.

For example:

```sql
CREATE PUBLICATION vehicle_publication
FOR TABLE vehicles;
```

This creates a publication for the `vehicles` table.

A publication can include multiple tables and can define which operations are published.

### 5.2 Subscription

A **subscription** defines how another PostgreSQL database receives changes from a publication.

Example:

```sql
CREATE SUBSCRIPTION vehicle_subscription
CONNECTION 'host=192.168.1.20 port=5432 dbname=fleet_db user=replication_user password=replace_me'
PUBLICATION vehicle_publication;
```

This example assumes:

- The source database has the publication.
- The destination database has a compatible table structure.
- The replication connection is permitted.
- The user has the necessary permissions.
- Network connectivity and authentication are correctly configured.

Never place production credentials directly in shared scripts or source-control repositories.

### 5.3 Logical Replication Architecture

```text
             SOURCE DATABASE
                    |
                    v
              PUBLICATION
                    |
                    v
             LOGICAL CHANGES
                    |
                    v
              NETWORK STREAM
                    |
                    v
              SUBSCRIPTION
                    |
                    v
           DESTINATION DATABASE
                    |
                    v
             REPLICATED TABLES
```

### 5.4 Advantages

- Selective replication of tables.
- Useful for data integration.
- Supports certain database migration strategies.
- Allows replication between compatible PostgreSQL environments with different major versions.
- Can support separate analytical or operational databases.

### 5.5 Limitations

Logical replication does not automatically replicate every database object or administrative state.

Important considerations:

- Table schemas must be prepared appropriately.
- DDL changes are not automatically replicated as ordinary row changes.
- Sequence state requires separate handling.
- Large objects are not automatically covered by ordinary logical replication.
- Conflicting writes at the subscriber require careful handling.
- Replication identity and primary-key design matter.

Logical replication is therefore not a complete replacement for physical replication.

### 5.6 Physical vs Logical Replication

| Characteristic | Physical | Logical |
|---|---|---|
| Replication level | Physical database changes | Logical row changes |
| Main mechanism | WAL streaming | Publications and subscriptions |
| Selective table replication | No, not as a normal physical standby | Yes |
| Entire database copy | Yes | Not automatically |
| Typical purpose | Standby and recovery | Integration and migration |
| Destination behavior | Physical standby follows source state | Subscriber maintains selected replicated data |

---

## 6. Understanding WAL and LSN

To understand PostgreSQL replication, two concepts are essential.

### 6.1 Write-Ahead Log (WAL)

The **Write-Ahead Log** is a transaction log used by PostgreSQL to support durability and crash recovery.

Before modified database pages are considered safely persisted, the corresponding WAL records must be written according to PostgreSQL's durability rules.

WAL also provides the information required by physical replication.

A simplified sequence:

```text
TRANSACTION
    |
    v
WAL RECORDS GENERATED
    |
    v
WAL WRITTEN
    |
    v
COMMIT
    |
    v
DATA PAGES WRITTEN LATER
```

WAL is important because database changes can be recovered by replaying the log.

See also:

- [[05 - Transactions and ACID]]
- [[01 - PostgreSQL Architecture Overview]]
- [[03 - PostgreSQL Storage]]

### 6.2 Log Sequence Number (LSN)

An **LSN** identifies a position in the WAL stream.

It helps PostgreSQL determine how far WAL generation, transmission, writing, flushing, and replay have progressed.

Example:

```text
0/30000A0
0/3000128
0/3000190
```

These values represent positions in the WAL stream.

The primary and standby can be compared using their respective LSN positions.

For example:

- Primary has generated WAL up to position A.
- Standby has received WAL up to position B.
- Standby has replayed WAL up to position C.

If A is ahead of C, the standby has not yet applied all generated changes.

### 6.3 Why LSN Matters

LSNs help answer questions such as:

- How much WAL has been generated?
- How much WAL has been sent?
- How much WAL has reached the standby?
- How much WAL has been replayed?
- How far behind is the standby?

LSN is one of the main tools for understanding replication progress.

---

## 7. Replication Lag

**Replication lag** is the delay or difference between the state of a primary database and the state of its replica.

A standby may receive changes after the primary commits them.

This means a query executed on the standby can return older information.

### 7.1 Example

Suppose an application updates a vehicle's status.

```sql
UPDATE vehicles
SET status = 'IN_MAINTENANCE'
WHERE inventory_number = 'PK-001';
```

The primary database commits the transaction.

However, the standby has not replayed the corresponding WAL records yet.

If the application immediately queries the standby:

```sql
SELECT status
FROM vehicles
WHERE inventory_number = 'PK-001';
```

The result may still be the previous status.

This is a replication-lag problem.

### 7.2 Why Replication Lag Occurs

Common causes include:

- Network latency.
- High write activity on the primary.
- Slow storage on the standby.
- Insufficient standby resources.
- Long-running queries interfering with WAL cleanup or replay.
- High WAL generation.
- Replication configuration problems.
- Standby disconnection.

### 7.3 Types of Lag

Replication progress can be considered at different stages:

| Stage | Meaning |
|---|---|
| Send lag | Delay in sending WAL from the primary. |
| Write lag | Delay until WAL is written on the standby. |
| Flush lag | Delay until WAL is flushed to durable storage. |
| Replay lag | Delay until WAL is applied on the standby. |

These distinctions help identify where replication is falling behind.

### 7.4 Read-After-Write Consistency

A common application problem occurs when a user writes data to the primary and immediately reads from a standby.

Example:

```text
1. User updates a vehicle.
2. Primary commits the update.
3. Application sends a read query to a standby.
4. Standby has not replayed the update.
5. User sees the previous value.
```

This is a consistency issue.

Possible solutions include:

- Route critical reads to the primary.
- Wait until the standby has replayed the required WAL position.
- Use an architecture with stronger synchronization guarantees.
- Design the application to tolerate temporarily stale reads.

The important lesson is that **read scaling can introduce consistency trade-offs**.

---

## 8. Replication Slots

A **replication slot** allows PostgreSQL to track how much WAL a replication consumer still needs.

Without appropriate retention, PostgreSQL might remove WAL records before a disconnected standby or logical replication consumer can process them.

A replication slot helps prevent this.

### 8.1 Why Replication Slots Matter

Consider this situation:

1. A standby becomes disconnected.
2. The primary continues generating WAL.
3. The standby still needs some of those WAL records.
4. A replication slot retains the required WAL.
5. The standby can reconnect and continue, provided the necessary WAL remains available.

However, replication slots introduce an operational risk.

If a consumer remains disconnected for too long, the primary can retain large amounts of WAL.

This can eventually fill the disk.

### 8.2 Monitoring Replication Slots

```sql
SELECT
    slot_name,
    slot_type,
    active,
    restart_lsn,
    wal_status
FROM pg_replication_slots;
```

The available columns can vary by PostgreSQL version.

Important fields:

| Field | Meaning |
|---|---|
| `slot_name` | Name of the replication slot. |
| `slot_type` | Physical or logical. |
| `active` | Indicates whether a consumer is currently connected. |
| `restart_lsn` | Oldest WAL position that may still be required. |
| `wal_status` | Information about the slot's WAL retention status. |

Replication slots must be monitored because excessive WAL retention can become a storage problem.

---

## 9. Configuring PostgreSQL Physical Replication

PostgreSQL supports streaming replication between a primary and one or more standby servers.

This section introduces the configuration concepts.

It is not a production deployment procedure. A real implementation requires careful configuration of networking, authentication, storage, permissions, security, and recovery procedures.

### 9.1 Primary Server Configuration

Some important parameters include:

```ini
wal_level = replica

max_wal_senders = 10

max_replication_slots = 10

hot_standby = on
```

Explanation:

| Parameter | Purpose |
|---|---|
| `wal_level` | Controls the amount of information written to WAL. |
| `max_wal_senders` | Sets the maximum number of concurrent WAL sender processes. |
| `max_replication_slots` | Sets the maximum number of replication slots. |
| `hot_standby` | Allows read-only queries during standby recovery. |

For synchronous replication, an additional setting may be used:

```ini
synchronous_standby_names = 'FIRST 1 (standby_a)'
```

The configuration must match the actual standby names and desired synchronization policy.

### 9.2 Create a Replication User

A dedicated replication account should be used instead of a general administrator account.

```sql
CREATE ROLE replicator
WITH REPLICATION LOGIN
PASSWORD 'replace_with_a_secure_secret';
```

The password shown is a placeholder.

In a real environment, use a securely managed credential and restrict the account's access.

### 9.3 Configure Client Authentication

PostgreSQL uses `pg_hba.conf` to control which clients can connect and how they authenticate.

A conceptual entry might look like:

```text
hostssl replication replicator 192.168.1.30/32 scram-sha-256
```

This example permits a replication connection from a specific IP address using SSL and SCRAM authentication.

The actual network address, TLS configuration, authentication method, and server policy must be verified before use.

### 9.4 Create the Standby

A standby requires an initial copy of the primary database.

PostgreSQL provides `pg_basebackup` for this purpose.

Conceptual command:

```bash
pg_basebackup \
    -h 192.168.1.20 \
    -D /var/lib/postgresql/standby \
    -U replicator \
    -R \
    -P
```

Explanation:

| Option | Meaning |
|---|---|
| `-h` | Primary server hostname or IP address. |
| `-D` | Destination directory. |
| `-U` | Replication user. |
| `-R` | Creates standby configuration, including `standby.signal` and connection settings. |
| `-P` | Displays progress. |

The command requires an appropriately prepared environment.

Important considerations:

- The destination directory must be suitable and correctly permissioned.
- The primary must permit replication connections.
- Firewall and network rules must allow communication.
- The replication user must have the required permissions.
- Credentials must be supplied securely.
- PostgreSQL versions and configuration must be compatible.

In PostgreSQL 12 and later, a standby uses a `standby.signal` file to indicate standby mode.

The `-R` option creates this file and writes connection settings into the standby configuration.

### 9.5 Primary Connection Configuration

A standby uses connection information to connect to the primary.

A conceptual configuration is:

```ini
primary_conninfo = 'host=192.168.1.20 port=5432 user=replicator application_name=standby_a'
```

The `application_name` helps identify the standby in replication monitoring views.

Do not store credentials insecurely in configuration files.

---

## 10. Monitoring Replication

Replication must be monitored continuously.

A replica that silently stops receiving or applying changes can create serious operational problems.

PostgreSQL exposes system views and functions for inspecting replication status.

### 10.1 Monitor Replication from the Primary

Run:

```sql
SELECT
    application_name,
    state,
    sync_state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    write_lag,
    flush_lag,
    replay_lag
FROM pg_stat_replication;
```

This view provides information about connected standbys.

Important fields:

| Field | Meaning |
|---|---|
| `application_name` | Identifies the standby connection. |
| `state` | Replication state, such as `streaming`. |
| `sync_state` | Indicates the standby's synchronization role. |
| `sent_lsn` | WAL position sent by the primary. |
| `write_lsn` | WAL position written by the standby. |
| `flush_lsn` | WAL position flushed by the standby. |
| `replay_lsn` | WAL position replayed by the standby. |
| `write_lag` | Recent write acknowledgment delay. |
| `flush_lag` | Recent flush acknowledgment delay. |
| `replay_lag` | Recent replay acknowledgment delay. |

These lag fields can be `NULL`, and they should not be interpreted as a permanent historical measurement of replication delay.

### 10.2 Monitor the Standby

On the standby, run:

```sql
SELECT
    status,
    receive_start_lsn,
    written_lsn,
    flushed_lsn,
    latest_end_lsn,
    latest_end_time
FROM pg_stat_wal_receiver;
```

This view provides information about the WAL receiver process.

You can also inspect the latest received and replayed positions:

```sql
SELECT
    pg_last_wal_receive_lsn() AS received_lsn,
    pg_last_wal_replay_lsn() AS replayed_lsn;
```

The difference between received and replayed WAL can help identify replay backlog.

Example:

```sql
SELECT
    pg_wal_lsn_diff(
        pg_last_wal_receive_lsn(),
        pg_last_wal_replay_lsn()
    ) AS pending_replay_bytes;
```

This result represents the byte difference between the received and replayed WAL positions.

It does not directly represent elapsed time.

### 10.3 Check the Last Replayed Transaction

```sql
SELECT
    pg_last_xact_replay_timestamp();
```

This function reports the timestamp of the last transaction replayed on the standby.

It can help identify whether a standby has stopped applying new transactions.

However, it must be interpreted carefully.

If the primary has had no recent write activity, the timestamp can become old even when replication is functioning correctly.

### 10.4 Monitoring Strategy

A practical monitoring system should evaluate:

- Whether the standby is connected.
- Whether WAL is being received.
- Whether WAL is being replayed.
- Whether replication lag exceeds acceptable thresholds.
- Whether replication slots are retaining excessive WAL.
- Whether storage capacity is sufficient.
- Whether replication errors appear in server logs.

Monitoring should detect both technical failures and situations where the replica is operational but too far behind to meet business requirements.

---

## 11. Failover and Switchover

Replication provides the foundation for certain high-availability architectures.

However, a standby does not automatically become the primary when the primary server fails.

The transition requires a controlled process.

### 11.1 Switchover

A **switchover** is a planned transition from the current primary to a standby.

Example:

```text
BEFORE

PRIMARY A
    |
    v
STANDBY B


AFTER

PRIMARY B
    |
    v
STANDBY A
```

It is typically performed for:

- Planned maintenance.
- Hardware replacement.
- Database migration.
- Infrastructure upgrades.
- Controlled failover testing.

A properly coordinated switchover aims to avoid losing committed transactions.

### 11.2 Failover

A **failover** is an unplanned transition after the primary becomes unavailable or cannot safely continue operating.

Example:

```text
BEFORE

PRIMARY A
    |
    v
STANDBY B


PRIMARY A FAILS


AFTER

STANDBY B
    |
    v
PROMOTED TO PRIMARY
```

The standby is promoted to become the new primary.

PostgreSQL provides mechanisms such as:

```sql
SELECT pg_promote();
```

A promotion can also be initiated using:

```bash
pg_ctl promote
```

The exact process depends on the deployment and recovery strategy.

### 11.3 Why Promotion Is Not Enough

A complete high-availability solution must also address:

- Detecting primary failure.
- Determining whether the primary is truly unavailable.
- Preventing two writable primaries.
- Promoting the correct standby.
- Redirecting application connections.
- Managing DNS, virtual IPs, or connection endpoints.
- Rebuilding or reconfiguring the former primary.
- Verifying database consistency after recovery.

This is why replication alone is not equivalent to high availability.

### 11.4 Split-Brain Problem

**Split brain** occurs when two database servers operate as independent writable primaries when the architecture expects only one.

Example:

```text
       APPLICATION GROUP A
                |
                v
           PRIMARY A


       APPLICATION GROUP B
                |
                v
           PRIMARY B
```

Both databases may accept writes.

The result can include:

- Conflicting records.
- Divergent database states.
- Inconsistent transactions.
- Difficult reconciliation.
- Potential data loss.

High-availability architectures must use mechanisms such as fencing, leader coordination, and controlled promotion to prevent this situation.

### 11.5 Rejoining the Former Primary

After a failover, the previous primary cannot simply be assumed to be a valid standby.

Its state must be evaluated.

Depending on the situation, it may need to be:

- Rebuilt from the new primary.
- Reconfigured as a standby.
- Reconciled using an appropriate recovery procedure.

PostgreSQL provides `pg_rewind` for certain cases where an old primary can be synchronized with a new timeline.

However, `pg_rewind` has prerequisites and is not applicable to every recovery scenario.

---

## 12. Replication Topologies

Replication can be organized in different ways.

### 12.1 Single Primary with One Standby

```text
PRIMARY
   |
   v
STANDBY
```

A simple architecture for recovery or read scaling.

### 12.2 Single Primary with Multiple Standbys

```text
              PRIMARY
                 |
       +---------+---------+
       |         |         |
       v         v         v
   STANDBY A  STANDBY B  STANDBY C
```

Useful when multiple replicas are needed for different workloads or recovery locations.

### 12.3 Cascading Replication

A standby can send WAL to another standby.

```text
PRIMARY
   |
   v
STANDBY A
   |
   v
STANDBY B
```

This is known as cascading replication.

It can reduce the number of direct replication connections to the primary.

However, additional replication stages can introduce additional delay and operational dependencies.

### 12.4 Multi-Primary Architecture

A multi-primary architecture allows multiple database nodes to accept writes.

```text
DATABASE A <----> DATABASE B
     ^                 ^
     |                 |
     +---- APPLICATION-+
```

This model introduces additional complexity:

- Write conflicts.
- Conflict resolution.
- Transaction coordination.
- Consistency management.
- Data ownership.
- Network partition behavior.

PostgreSQL's core physical streaming replication is based on a single writable primary and does not provide general native multi-primary replication.

Other technologies and extensions can implement multi-writer approaches, but they require separate architectural evaluation.

---

## 13. Replication vs Backup vs Partitioning vs Sharding

These concepts are related to database architecture, but they solve different problems.

| Concept | Main purpose |
|---|---|
| Replication | Maintain additional copies of database data. |
| Backup | Preserve recoverable copies of data from earlier points in time. |
| Partitioning | Divide a large table into smaller logical or physical parts. |
| Sharding | Distribute data across multiple database nodes. |

### 13.1 Replication Is Not a Backup

Suppose a user accidentally deletes thousands of records.

If the deletion is replicated, the standby may also receive the deletion.

Replication can therefore reproduce mistakes, corruption, or unwanted changes.

A backup provides a separate recovery mechanism.

A robust architecture can use both:

```text
                 PRIMARY DATABASE
                       |
              +--------+--------+
              |                 |
              v                 v
          REPLICATION         BACKUPS
              |                 |
              v                 v
          STANDBY          RECOVERY COPIES
```

The replication copy helps maintain availability.

The backup helps recover historical data.

See also:

- [[07 - Backup and Disaster Recovery]]
- [[02 - Database Scalability]]
- [[04 - Partitioning]]
- [[05 - Sharding]]

---

## 14. Replication and Database Architecture Decisions

Replication is an architectural decision, not simply a configuration setting.

Before implementing it, consider the following questions.

| Question | Architectural concern |
|---|---|
| Why is replication needed? | Availability, read scaling, recovery, migration, or integration. |
| How much data loss is acceptable? | RPO. |
| How quickly must the system recover? | RTO. |
| How many replicas are required? | Capacity and availability. |
| Should replicas accept reads? | Workload distribution. |
| Is synchronous replication necessary? | Durability and latency trade-offs. |
| How will failures be detected? | Monitoring and orchestration. |
| How will applications reconnect? | Connection routing. |
| How will backups be maintained? | Independent recovery capability. |
| How will WAL retention be controlled? | Storage management. |

The goal is not to add replicas simply because they are available.

The goal is to select a replication architecture that satisfies measurable business and technical requirements.

---

## 15. Practical PostgreSQL Laboratory

This section introduces ways to inspect replication concepts.

A complete physical replication setup normally requires at least two PostgreSQL instances.

If you currently have only one local instance, you can still study the monitoring concepts and prepare a second instance later using Docker or another isolated environment.

### 15.1 Inspect the Current Database

```sql
SELECT
    version();
```

Inspect the WAL level:

```sql
SHOW wal_level;
```

Inspect replication-related settings:

```sql
SHOW max_wal_senders;

SHOW max_replication_slots;

SHOW synchronous_commit;

SHOW hot_standby;
```

These commands help identify the current configuration.

They do not establish that replication is already configured.

### 15.2 Inspect Replication Connections

On a primary database:

```sql
SELECT
    application_name,
    client_addr,
    state,
    sync_state
FROM pg_stat_replication;
```

If no standby is connected, the result may be empty.

That does not necessarily indicate a database failure.

### 15.3 Inspect Replication Slots

```sql
SELECT
    slot_name,
    slot_type,
    active,
    restart_lsn
FROM pg_replication_slots;
```

An empty result means there are currently no visible replication slots.

### 15.4 Inspect WAL Activity

```sql
SELECT
    pg_current_wal_lsn() AS current_wal_position;
```

This returns the current WAL position on the primary.

You can use it to understand how WAL positions change as transactions are executed.

### 15.5 Observe Transaction Activity

Create a small test table:

```sql
CREATE TABLE replication_lab (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    description TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert a record:

```sql
INSERT INTO replication_lab (description)
VALUES ('First replication test');
```

Inspect the WAL position again:

```sql
SELECT
    pg_current_wal_lsn();
```

In a two-instance replication environment, you could then compare the primary and standby positions.

The purpose is to connect transaction execution with WAL generation and replication progress.

---

## 16. Applying Replication to SIAPV

Consider the SIAPV application as a database system managing approximately 200 vehicles and machinery units.

Suppose the application grows and begins supporting:

- Multiple operational departments.
- More concurrent users.
- Historical reporting.
- Maintenance records.
- Vehicle assignments.
- Incident tracking.
- Management dashboards.

A possible future architecture could be:

```text
                    SIAPV APPLICATION
                            |
                            v
                    PRIMARY DATABASE
                      READ / WRITE
                            |
                   STREAMING REPLICATION
                            |
                +-----------+-----------+
                |                       |
                v                       v
           STANDBY A               STANDBY B
           REPORTING               RECOVERY
           READ ONLY               READ ONLY
```

### 16.1 Primary Responsibilities

The primary database would handle:

- Vehicle registration.
- Inventory modifications.
- Maintenance updates.
- Assignment changes.
- Incident records.
- Operational transactions.

### 16.2 Reporting Standby

A read-only standby could support:

- Historical reports.
- Vehicle utilization analysis.
- Maintenance summaries.
- Monthly operational indicators.
- Management dashboards.

This could reduce the read workload on the primary.

However, reports must tolerate possible replication lag.

### 16.3 Recovery Standby

Another standby could be maintained for recovery purposes.

Its usefulness would depend on:

- Geographic separation.
- Network reliability.
- Replication mode.
- Monitoring.
- Promotion procedures.
- Recovery testing.

Independent backups would still be necessary.

### 16.4 Important Application Decision

Suppose an operator changes a vehicle's status from `AVAILABLE` to `IN_MAINTENANCE`.

The application should confirm the change using the primary database.

If the same operator immediately opens the vehicle record, the application should consider reading from the primary to avoid displaying an outdated value from a standby.

This is an example of designing application behavior around replication consistency.

---

## 17. Common Replication Mistakes

| Mistake | Why It Is a Problem |
|---|---|
| Treating replication as a backup | Replicated deletions and corruption can affect replicas. |
| Assuming replicas are always current | Asynchronous replication can introduce lag. |
| Assuming a standby automatically becomes primary | Promotion and connection routing require coordination. |
| Ignoring replication slots | Retained WAL can consume disk space. |
| Using synchronous replication without planning | Commit latency and availability can be affected. |
| Allowing unrestricted replication connections | Increases security exposure. |
| Ignoring failover testing | Recovery procedures may fail when actually needed. |
| Sending every read to replicas | Applications may observe stale data. |
| Assuming physical and logical replication are interchangeable | They have different capabilities and limitations. |

---

## 18. Knowledge Check

Answer these questions without looking at the explanations.

1. What is database replication?
2. What is the difference between a primary and a standby?
3. How does synchronous replication differ from asynchronous replication?
4. What is the main role of WAL in PostgreSQL replication?
5. What does an LSN represent?
6. What is replication lag?
7. Why can a read replica return outdated information?
8. What is the difference between physical and logical replication?
9. What is a replication slot?
10. Why can a replication slot cause storage problems?
11. What is the difference between failover and switchover?
12. What is split brain?
13. Why is replication not a replacement for backups?
14. Why might an application route certain reads to the primary?
15. What architectural requirements should be defined before implementing replication?

---

## 19. Practical Exercises

### Exercise 1 — Architecture Comparison

Design two replication architectures for a fictional fleet management system.

**Architecture A:**

- One primary.
- One asynchronous standby.
- Read-only reporting.

**Architecture B:**

- One primary.
- One synchronous standby.
- One independent backup strategy.
- Planned failover procedures.

Explain the advantages, disadvantages, and operational risks of each design.

### Exercise 2 — Replication Failure

Imagine that a primary database suddenly becomes unavailable.

The standby is connected but has not replayed the latest WAL records.

Answer:

1. What information would you inspect before promoting the standby?
2. What data-loss risk might exist?
3. How would you prevent split brain?
4. What would happen to application connections?
5. How would you prepare the former primary to rejoin the architecture?

### Exercise 3 — SIAPV Architecture

Design a replication architecture for a future SIAPV deployment with:

- 10,000 managed units.
- Multiple operational departments.
- Several concurrent users.
- Historical reports.
- Critical maintenance updates.
- Recovery requirements.

Define:

- Primary responsibilities.
- Standby responsibilities.
- Replication mode.
- Read-routing strategy.
- Backup strategy.
- Monitoring requirements.
- Failover procedure.

### Exercise 4 — PostgreSQL Investigation

Using your PostgreSQL environment, execute and explain:

```sql
SHOW wal_level;
```

```sql
SELECT pg_current_wal_lsn();
```

```sql
SELECT *
FROM pg_stat_replication;
```

```sql
SELECT *
FROM pg_replication_slots;
```

For each query, describe what it reveals and why it matters.

Do not assume that replication is configured simply because these queries execute successfully.

---

## 20. Technical Vocabulary

| English term | Meaning |
|---|---|
| Database replication | Database replication |
| Primary database | Primary database |
| Standby database | Standby database |
| Read replica | Read replica |
| Synchronous replication | Synchronous replication |
| Asynchronous replication | Asynchronous replication |
| Physical replication | Physical replication |
| Logical replication | Logical replication |
| Write-Ahead Log (WAL) | Write-ahead transaction log |
| Log Sequence Number (LSN) | Log sequence number |
| Replication lag | Replication delay |
| Replication slot | Replication slot |
| WAL sender | WAL sender process |
| WAL receiver | WAL receiver process |
| WAL replay | WAL replay |
| Failover | Unplanned transition to a standby |
| Switchover | Planned transition to a standby |
| Promotion | Standby promotion |
| Split brain | Multiple nodes operating as independent primaries |
| Hot standby | Standby that accepts read-only queries |
| Recovery Point Objective (RPO) | Maximum tolerable data loss |
| Recovery Time Objective (RTO) | Target recovery time |
| Cascading replication | Replication through another standby |
| Publication | Logical replication publication |
| Subscription | Logical replication subscription |

---

## 21. Key Takeaways

- Replication maintains additional copies of database data.
- A primary generally handles write operations.
- A standby receives and applies changes from the primary.
- Asynchronous replication usually provides lower latency but can expose the system to recent data loss during certain failures.
- Synchronous replication provides stronger acknowledgment guarantees at the cost of additional coordination and latency.
- PostgreSQL physical replication uses WAL to reproduce database changes.
- Logical replication publishes selected table changes to subscribers.
- LSN values help track replication progress.
- Replication lag can cause applications to read outdated information.
- Replication slots retain WAL required by replication consumers and must be monitored.
- Failover and switchover require operational coordination.
- Split brain must be prevented in high-availability architectures.
- Replication improves availability and can support read scaling, but it does not replace backups.

## 22. Next Learning Step

The next topic is **Database Partitioning**.

Replication creates copies of data across database instances.

Partitioning solves a different problem: it divides a large table into smaller, more manageable physical or logical sections while preserving the table as a unified database object.

We will study partitioning strategies, partition keys, range and list partitioning, hash partitioning, and partition pruning.

**Next file:** `04 - Database Architecture/04 - Partitioning.md`

Related notes:

- [[02 - Database Scalability]]
- [[05 - Sharding]]
- [[06 - High Availability]]
- [[07 - Backup and Disaster Recovery]]
- [[01 - PostgreSQL Architecture Overview]]
