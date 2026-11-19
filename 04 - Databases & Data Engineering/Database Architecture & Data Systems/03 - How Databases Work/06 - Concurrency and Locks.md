
---
title: "Concurrency and Locks"
aliases:
  - "Database Concurrency Control"
  - "PostgreSQL Locking"
  - "MVCC and Lock Management"
type: learning-note
domain: technology
category: database-systems
subject: concurrency-control
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - database-architecture
  - concurrency-control
  - database-locks
  - mvcc
  - postgresql
  - transactions
  - deadlocks
  - isolation-levels
concepts:
  - concurrency
  - parallelism
  - transaction-isolation
  - multiversion-concurrency-control
  - row-level-locks
  - table-level-locks
  - lock-compatibility
  - blocking
  - deadlocks
  - optimistic-concurrency-control
  - pessimistic-concurrency-control
  - advisory-locks
  - vacuum
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Database Management Systems]]"
  - "[[05 - Transactions and ACID]]"
  - "[[03 - Storage Pages and Tables]]"
related:
  - "[[02 - How a Query is Executed]]"
  - "[[04 - Indexes]]"
  - "[[02 - Database Scalability]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"
  - "[[01 - Database Security Fundamentals]]"
tools:
  - PostgreSQL
  - SQL
  - psql
  - pgAdmin
learning_objectives:
  - Explain why databases require concurrency control.
  - Distinguish concurrency from parallelism.
  - Understand PostgreSQL MVCC and row visibility.
  - Identify common concurrency anomalies.
  - Understand table-level and row-level locks.
  - Diagnose blocking sessions and deadlocks.
  - Apply locking strategies to transactional workloads.
  - Explain how long-running transactions affect vacuum.
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 06 — Concurrency and Locks

## 1. Introduction to Database Concurrency

A database system rarely serves only one user at a time.

In a production environment, many users, applications, services, and automated processes may access the same database simultaneously.

For example, consider a vehicle management system:

- One employee consults a vehicle's information.
- Another employee updates its current mileage.
- A maintenance employee registers a service.
- A reporting process retrieves information about the vehicle fleet.
- An automated process imports new records.

All these operations may happen at nearly the same time.

The database must coordinate these operations to prevent inconsistent results.

This problem is known as **concurrency control**.

### 1.1 What is concurrency?

Concurrency is the ability of a system to manage multiple operations whose execution periods overlap.

These operations do not necessarily execute at exactly the same instant.

They may alternate execution while maintaining the appearance of simultaneous activity.

Consider three transactions:

```text
Transaction A: Read vehicle information
Transaction B: Update vehicle mileage
Transaction C: Register maintenance
```

Their execution may overlap:

```text
Time →

Transaction A: |---- READ --------|
Transaction B:     |--- UPDATE -------|
Transaction C:          |---- INSERT -----|
```

The database must determine how these transactions interact.

### 1.2 Concurrency vs. parallelism

These concepts are related but different.

| Concept | Meaning |
|---|---|
| Concurrency | Multiple operations make progress during overlapping periods. |
| Parallelism | Multiple operations execute at the same instant using separate processing resources. |

A database can support concurrency even when operations are not executing in parallel.

For example, a single CPU core can switch between multiple database sessions.

With multiple CPU cores, some operations may also execute in parallel.

**Important distinction:**

Concurrency is primarily about coordinating operations. Parallelism is about executing operations simultaneously.

## 2. Why Databases Need Concurrency Control

Without concurrency control, simultaneous transactions could produce incorrect results.

Consider a vehicle with the following mileage:

```text
Current mileage: 25,000 km
```

Two employees independently retrieve this value.

Employee A records an increase of 500 km.

Employee B records an increase of 300 km.

Both calculate a new value using the original mileage:

```text
Employee A: 25,000 + 500 = 25,500
Employee B: 25,000 + 300 = 25,300
```

If both updates overwrite the same record, the final result might be:

```text
25,300 km
```

The correct result should be:

```text
25,800 km
```

One update has effectively been lost.

This is called a **lost update anomaly**.

Concurrency control helps prevent this and other inconsistencies.

### 2.1 Main objectives

A database concurrency-control mechanism aims to:

1. Preserve data integrity.
2. Coordinate simultaneous transactions.
3. Prevent incompatible operations from interfering.
4. Maintain transaction isolation.
5. Reduce unnecessary blocking.
6. Detect and resolve deadlocks.
7. Preserve acceptable system performance.

Concurrency control is not simply about preventing simultaneous access.

It is about allowing as much simultaneous work as possible while preserving correctness.

## 3. Database Sessions and Concurrent Transactions

A database session represents an established interaction between a client and a database server.

For example:

```text
Application A ── Session A ──┐
                             │
Application B ── Session B ──┼── PostgreSQL
                             │
Application C ── Session C ──┘
```

Each session can execute transactions independently.

A transaction is a logical unit of work.

```sql
BEGIN;

UPDATE vehicles
SET mileage = mileage + 500
WHERE vehicle_id = 101;

COMMIT;
```

Another session may execute a different transaction at the same time.

The DBMS coordinates these operations through mechanisms such as:

- Transaction isolation.
- Multi-Version Concurrency Control.
- Locks.
- Deadlock detection.
- Transaction logging.

These mechanisms work together rather than operating independently.

## 4. Multi-Version Concurrency Control (MVCC)

One of the most important concepts in PostgreSQL is **Multi-Version Concurrency Control (MVCC)**.

MVCC is a concurrency-control mechanism that allows transactions to access different visible versions of a row.

Instead of requiring every reader to wait for every writer, PostgreSQL can preserve older row versions so that readers can continue working with a consistent view of the data.

### 4.1 The basic idea

Imagine a table containing a vehicle:

| Vehicle ID | Mileage |
|---|---:|
| 101 | 25,000 |

A transaction updates its mileage:

```sql
UPDATE vehicles
SET mileage = 25,500
WHERE vehicle_id = 101;
```

Conceptually, PostgreSQL creates a new row version.

```text
Previous version:
Vehicle 101 → 25,000 km

New version:
Vehicle 101 → 25,500 km
```

Different transactions may observe different versions depending on their snapshots and isolation levels.

This does not mean PostgreSQL stores an unlimited collection of permanent copies.

Old row versions become eligible for cleanup when they are no longer needed by active transactions.

### 4.2 How MVCC helps readers and writers

Consider two sessions.

**Session A**

```sql
BEGIN;

SELECT mileage
FROM vehicles
WHERE vehicle_id = 101;
```

Session A reads:

```text
25,000 km
```

**Session B**

```sql
BEGIN;

UPDATE vehicles
SET mileage = 25,500
WHERE vehicle_id = 101;

COMMIT;
```

Under PostgreSQL's MVCC model, Session A can generally continue reading its previously visible version without waiting for Session B's update to finish.

The exact version visible depends on the transaction isolation level and snapshot.

This is one of the reasons PostgreSQL can support many concurrent readers and writers.

### 4.3 MVCC does not eliminate locks

A common misconception is that MVCC means PostgreSQL does not use locks.

That is incorrect.

PostgreSQL uses both MVCC and locks.

MVCC primarily helps determine which row versions are visible to a transaction.

Locks coordinate operations that must not conflict.

For example:

- Two transactions attempting to update the same row may block each other.
- A transaction executing certain DDL operations may block other operations.
- A transaction explicitly requesting a row lock can prevent another transaction from acquiring an incompatible lock.

**Remember:**

MVCC reduces unnecessary blocking, but it does not eliminate locking.

### 4.4 Transaction identifiers and row visibility

PostgreSQL uses transaction identifiers (XIDs) as part of its MVCC visibility mechanism.

Each stored row version contains internal metadata, including:

- `xmin`: the transaction identifier associated with the creation of the row version.
- `xmax`: the transaction identifier associated with its deletion or replacement, when applicable.

These fields help PostgreSQL determine whether a row version is visible to a particular transaction.

A simplified conceptual example:

```text
Row version A
Mileage: 25,000
xmin: 100
xmax: 105

Row version B
Mileage: 25,500
xmin: 105
xmax: 0
```

This illustrates a previous version and a newer version.

The actual visibility rules also depend on transaction status and snapshot information. The values should not be interpreted as a simple universal rule that every transaction with a larger identifier sees the newest version.

### 4.5 Snapshots

A snapshot represents the transaction visibility information PostgreSQL uses to determine which committed row versions a transaction can see.

Different isolation levels use snapshots differently.

| Isolation level | Snapshot behavior in PostgreSQL |
|---|---|
| READ COMMITTED | A new snapshot is generally taken for each statement. |
| REPEATABLE READ | The transaction uses a stable snapshot. |
| SERIALIZABLE | Uses a stable snapshot plus additional checks for serialization conflicts. |

Snapshots are central to understanding why two transactions may observe different results even when querying the same table.

## 5. Concurrency Anomalies

A concurrency anomaly is an unexpected or undesirable result caused by interactions between transactions.

Understanding these anomalies helps determine which isolation level or locking strategy is appropriate.

### 5.1 Dirty read

A dirty read occurs when a transaction reads data written by another transaction that has not yet committed.

Example:

```text
Transaction A:
UPDATE account balance to 500
(Not committed)

Transaction B:
Reads balance = 500

Transaction A:
ROLLBACK
```

Transaction B has read a value that was never committed.

PostgreSQL prevents dirty reads.

Even when a transaction requests READ UNCOMMITTED, PostgreSQL behaves as READ COMMITTED.

### 5.2 Non-repeatable read

A non-repeatable read occurs when a transaction reads the same row twice and obtains different committed values.

Example:

```text
Transaction A:
SELECT mileage → 25,000

Transaction B:
UPDATE mileage → 25,500
COMMIT

Transaction A:
SELECT mileage → 25,500
```

Transaction A observes different values for the same row.

This can occur under READ COMMITTED in PostgreSQL.

### 5.3 Phantom read

A phantom read occurs when a transaction repeats a query and obtains a different set of rows because another transaction has inserted, deleted, or modified rows that satisfy the query condition.

Example:

```sql
SELECT *
FROM vehicles
WHERE mileage > 100000;
```

The first execution returns:

```text
Vehicle 101
Vehicle 205
```

Another transaction inserts a vehicle with mileage of 150,000 and commits.

The second execution may return:

```text
Vehicle 101
Vehicle 205
Vehicle 310
```

The additional row is called a phantom.

PostgreSQL's REPEATABLE READ prevents phantom reads through its snapshot behavior, although SERIALIZABLE provides stronger protection against broader serialization anomalies.

### 5.4 Lost update

A lost update happens when two transactions modify the same data and one modification overwrites the other.

Conceptually:

```text
Initial value: 100

Transaction A reads 100
Transaction B reads 100

Transaction A calculates 100 + 20
Transaction B calculates 100 + 30

Transaction A writes 120
Transaction B writes 130

Final value: 130
```

The update from Transaction A is lost.

PostgreSQL's row-level locking and isolation behavior prevent many direct lost-update scenarios involving concurrent updates to the same row. However, application-level read-modify-write workflows can still be incorrect if they use separate reads and unconditional writes without appropriate safeguards.

An atomic update is often safer:

```sql
UPDATE accounts
SET balance = balance + 20
WHERE account_id = 1;
```

The database performs the increment as part of one statement rather than relying on an application to calculate and overwrite a previously read value.

### 5.5 Write skew

Write skew occurs when two transactions read overlapping information and then update different rows in ways that jointly violate a business rule.

Example:

A department must always have at least one employee on duty.

Initially:

| Employee | On duty |
|---|---|
| A | Yes |
| B | Yes |

Two transactions execute:

```text
Transaction A:
Checks that Employee B is on duty.
Sets Employee A to off duty.

Transaction B:
Checks that Employee A is on duty.
Sets Employee B to off duty.
```

If both transactions proceed using snapshots that show the other employee as on duty, both may commit under an isolation level that allows this anomaly.

The final state is:

| Employee | On duty |
|---|---|
| A | No |
| B | No |

The business rule has been violated.

This is one reason SERIALIZABLE isolation is important for certain business operations.

## 6. PostgreSQL Isolation Levels and Concurrency

PostgreSQL implements four SQL transaction isolation-level names, but READ UNCOMMITTED behaves like READ COMMITTED.

| Isolation level | Main behavior |
|---|---|
| READ UNCOMMITTED | Behaves like READ COMMITTED in PostgreSQL. |
| READ COMMITTED | Each statement receives a new snapshot. |
| REPEATABLE READ | The transaction uses a stable snapshot. |
| SERIALIZABLE | Attempts to ensure transactions behave as if executed sequentially. |

### 6.1 READ COMMITTED

This is PostgreSQL's default isolation level.

```sql
BEGIN ISOLATION LEVEL READ COMMITTED;

SELECT *
FROM vehicles
WHERE vehicle_id = 101;

COMMIT;
```

Each statement generally sees data committed before that statement began.

Advantages:

- Good concurrency.
- Lower overhead than stronger isolation in many workloads.
- Suitable for many ordinary transactional applications.

Considerations:

- A repeated query may return different committed data.
- Application logic must account for changes between statements.

### 6.2 REPEATABLE READ

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;

SELECT *
FROM vehicles
WHERE vehicle_id = 101;

-- Other transactions may commit changes.

SELECT *
FROM vehicles
WHERE vehicle_id = 101;

COMMIT;
```

Both reads use the transaction's stable snapshot.

Advantages:

- Stable view of committed data during the transaction.
- Prevents non-repeatable and phantom reads in PostgreSQL.

Considerations:

- Concurrent modifications can cause serialization-related errors.
- Long-running transactions can retain old row versions and affect maintenance.

### 6.3 SERIALIZABLE

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;

SELECT *
FROM vehicles
WHERE vehicle_id = 101;

UPDATE vehicles
SET status = 'IN_MAINTENANCE'
WHERE vehicle_id = 101;

COMMIT;
```

SERIALIZABLE provides the strongest standard isolation level.

PostgreSQL uses Serializable Snapshot Isolation (SSI) to detect dangerous dependency patterns.

It may abort a transaction if allowing all transactions to commit could produce a result inconsistent with any serial execution.

Advantages:

- Strong protection against serialization anomalies.
- Useful for complex business rules involving multiple records.

Considerations:

- Transactions may fail with serialization errors.
- Applications must be prepared to retry certain transactions.

**Important:** SERIALIZABLE does not mean that PostgreSQL executes every transaction one at a time. It uses concurrency while checking whether the resulting behavior is serializable.

## 7. Understanding Database Locks

A lock is a mechanism that controls access to a database resource.

Locks help coordinate operations that cannot safely proceed simultaneously.

A database can acquire locks automatically or an application can request certain locks explicitly.

### 7.1 Why locks are necessary

Imagine two employees editing the same vehicle record.

```text
Employee A:
Changes vehicle status to IN_MAINTENANCE.

Employee B:
Changes vehicle status to AVAILABLE.
```

If both operations modify the same record simultaneously, the database must coordinate them.

A row-level lock can prevent incompatible modifications from occurring at the same time.

### 7.2 Main lock categories

PostgreSQL has several lock mechanisms.

| Lock category | Purpose |
|---|---|
| Table-level locks | Coordinate operations on database relations such as tables. |
| Row-level locks | Coordinate operations affecting individual rows. |
| Advisory locks | Allow applications to coordinate custom operations. |
| Predicate locks | Help SERIALIZABLE transactions detect conflicts involving sets of rows. |
| Lightweight locks | Protect internal shared-memory structures and operations. |

These mechanisms serve different purposes.

They should not be treated as interchangeable.

## 8. Table-Level Locks

PostgreSQL uses table-level lock modes to coordinate operations on tables and other relations.

Some operations acquire stronger locks than others.

### 8.1 Main table-level lock modes

| Lock mode | General purpose |
|---|---|
| ACCESS SHARE | Allows ordinary reads. |
| ROW SHARE | Used by operations such as SELECT FOR UPDATE. |
| ROW EXCLUSIVE | Used by INSERT, UPDATE, and DELETE. |
| SHARE UPDATE EXCLUSIVE | Used by certain maintenance operations, including VACUUM. |
| SHARE | Allows reads while restricting conflicting modifications. |
| SHARE ROW EXCLUSIVE | A stronger lock that conflicts with several other modes. |
| EXCLUSIVE | Prevents many concurrent operations while allowing ACCESS SHARE. |
| ACCESS EXCLUSIVE | Strongest table lock; blocks ordinary reads and writes. |

### 8.2 Examples of operations and locks

| SQL operation | Typical table-level lock |
|---|---|
| SELECT | ACCESS SHARE |
| INSERT | ROW EXCLUSIVE |
| UPDATE | ROW EXCLUSIVE |
| DELETE | ROW EXCLUSIVE |
| SELECT FOR UPDATE | ROW SHARE |
| TRUNCATE | ACCESS EXCLUSIVE |
| Many ALTER TABLE operations | ACCESS EXCLUSIVE |

The exact lock requirements can vary by operation and PostgreSQL version.

### 8.3 Lock compatibility

Not every lock conflicts with every other lock.

For example:

- ACCESS SHARE is compatible with many ordinary operations.
- ROW EXCLUSIVE locks can coexist with other ROW EXCLUSIVE locks.
- ACCESS EXCLUSIVE conflicts with all other table-level lock modes.

This is important because two transactions can often modify different rows in the same table without blocking each other at the table-lock level.

The row-level locks determine whether their individual row modifications conflict.

**Key distinction:**

A table-level ROW EXCLUSIVE lock does not mean that the entire table is exclusively locked.

## 9. Row-Level Locks

Row-level locks coordinate access to individual rows.

They are particularly important in transactional applications.

### 9.1 Common row-lock modes

PostgreSQL supports four principal row-level lock modes:

| Row lock mode | General purpose |
|---|---|
| FOR UPDATE | Strongest ordinary row lock; prevents conflicting updates and row-lock requests. |
| FOR NO KEY UPDATE | Similar to FOR UPDATE, but weaker when key columns are not being changed. |
| FOR SHARE | Allows shared row locking while preventing conflicting modifications. |
| FOR KEY SHARE | Protects key-related values from certain updates and deletes. |

The exact conflict behavior depends on the lock modes involved.

### 9.2 Locking a specific row

```sql
BEGIN;

SELECT *
FROM vehicles
WHERE vehicle_id = 101
FOR UPDATE;

UPDATE vehicles
SET status = 'IN_MAINTENANCE'
WHERE vehicle_id = 101;

COMMIT;
```

The `FOR UPDATE` clause requests row-level locks on the selected rows.

Another transaction attempting an incompatible operation on those rows may have to wait.

### 9.3 NOWAIT

Sometimes an application should not wait for a lock.

It can request immediate failure instead:

```sql
SELECT *
FROM vehicles
WHERE vehicle_id = 101
FOR UPDATE NOWAIT;
```

If the required row lock cannot be acquired immediately, PostgreSQL returns an error.

This can be useful when the application prefers to inform the user that the record is currently being modified.

### 9.4 SKIP LOCKED

`SKIP LOCKED` allows a transaction to ignore rows that are already locked by another transaction.

Example:

```sql
SELECT *
FROM maintenance_jobs
WHERE status = 'PENDING'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 1;
```

This is useful for queue-processing systems.

Imagine multiple workers processing maintenance jobs.

```text
Worker A → Locks Job 1
Worker B → Skips Job 1 and locks Job 2
Worker C → Skips Jobs 1 and 2 and locks Job 3
```

Workers can process separate jobs without waiting for one another.

However, `SKIP LOCKED` does not provide a generally consistent view of all rows. It is intended for workloads such as queues where skipping currently locked records is acceptable.

## 10. Blocking and Lock Waits

Blocking occurs when a transaction must wait because another transaction holds an incompatible lock.

Example:

**Session A**

```sql
BEGIN;

UPDATE vehicles
SET mileage = 25500
WHERE vehicle_id = 101;

-- Keep the transaction open.
```

**Session B**

```sql
UPDATE vehicles
SET mileage = 26000
WHERE vehicle_id = 101;
```

Session B may wait because Session A holds a conflicting row lock.

If Session A commits:

```sql
COMMIT;
```

Session B can continue, subject to the transaction's isolation rules.

### 10.1 Blocking is not necessarily a problem

Some blocking is normal.

For example:

- Two users editing the same record.
- A maintenance operation requiring a stronger table lock.
- A transaction waiting for another transaction to finish.

The problem occurs when blocking becomes excessive or prolonged.

Common causes include:

- Long-running transactions.
- Transactions left open accidentally.
- Large updates or deletes.
- DDL operations.
- Poorly designed transaction boundaries.
- Missing indexes that cause operations to process more rows than necessary.

## 11. Monitoring Locks in PostgreSQL

PostgreSQL provides system views that help administrators inspect sessions, locks, and waiting processes.

The most useful views and functions include:

| Resource | Purpose |
|---|---|
| `pg_stat_activity` | Displays information about database sessions and queries. |
| `pg_locks` | Displays locks currently held or awaited. |
| `pg_blocking_pids()` | Returns process IDs blocking a specified backend. |
| `pg_stat_database` | Provides database-level statistics. |

### 11.1 Inspect active sessions

```sql
SELECT
    pid,
    usename,
    application_name,
    state,
    wait_event_type,
    wait_event,
    query_start,
    query
FROM pg_stat_activity
WHERE datname = current_database()
ORDER BY query_start;
```

Important columns:

| Column | Meaning |
|---|---|
| pid | Process identifier of the database backend. |
| usename | Connected database user. |
| application_name | Name of the connecting application. |
| state | Session state. |
| wait_event_type | Category of the current wait event. |
| wait_event | Specific event being waited on. |
| query_start | Time when the current query began. |
| query | Current or most recent query text. |

A session with `wait_event_type = 'Lock'` is waiting for a lock.

A session in `idle in transaction` state has an open transaction but is not currently executing a query.

That state deserves attention because an abandoned transaction can retain locks and old row versions.

### 11.2 Identify blocking sessions

```sql
SELECT
    pid AS blocked_pid,
    pg_blocking_pids(pid) AS blocking_pids,
    query AS blocked_query,
    wait_event_type,
    wait_event
FROM pg_stat_activity
WHERE cardinality(pg_blocking_pids(pid)) > 0;
```

This query identifies sessions currently waiting for other sessions.

### 11.3 Inspect lock information

```sql
SELECT
    pid,
    locktype,
    relation::regclass AS relation,
    mode,
    granted,
    transactionid,
    virtualxid
FROM pg_locks
ORDER BY pid;
```

Important columns:

- `pid`: session process identifier.
- `locktype`: type of resource being locked.
- `relation`: associated relation, when applicable.
- `mode`: requested lock mode.
- `granted`: indicates whether the lock has been acquired.
- `transactionid`: associated transaction identifier, when applicable.

A lock with `granted = false` represents a lock request that is waiting.

These monitoring queries are useful when diagnosing performance problems in production systems.

## 12. Deadlocks

A deadlock occurs when two or more transactions wait for one another in a cycle.

Neither transaction can continue without another transaction releasing a resource.

### 12.1 Simple deadlock example

Suppose two transactions update two different vehicles.

```text
Transaction A:
Locks Vehicle 101.

Transaction B:
Locks Vehicle 202.
```

Then:

```text
Transaction A:
Attempts to lock Vehicle 202.
Waits for Transaction B.

Transaction B:
Attempts to lock Vehicle 101.
Waits for Transaction A.
```

The result:

```text
Transaction A → waits for B
Transaction B → waits for A
```

Neither transaction can proceed.

### 12.2 How PostgreSQL handles deadlocks

PostgreSQL detects deadlocks.

When it detects a deadlock cycle, it aborts one of the transactions to break the cycle.

The application receives an error.

The SQLSTATE code for a deadlock is:

```text
40P01
```

The application should generally handle this error and retry the complete transaction when appropriate.

### 12.3 How to prevent deadlocks

Several practices reduce deadlock risk:

1. Access records in a consistent order.
2. Keep transactions short.
3. Avoid unnecessary locks.
4. Do not wait for user interaction while a transaction remains open.
5. Use appropriate indexes.
6. Retry transactions after recoverable deadlock errors.

For example, if two transactions must update vehicles 101 and 202, both should acquire locks in the same order:

```text
First: Vehicle 101
Second: Vehicle 202
```

Consistent ordering reduces the chance of circular waiting.

## 13. Lock Timeouts and Statement Timeouts

PostgreSQL provides settings to limit how long operations can wait or execute.

### 13.1 lock_timeout

```sql
SET lock_timeout = '3s';
```

This limits how long a statement waits to acquire a lock.

If the required lock is not acquired within the configured period, PostgreSQL cancels the statement with an error.

### 13.2 statement_timeout

```sql
SET statement_timeout = '30s';
```

This limits the total execution time of a statement.

It is not limited to lock waits.

A statement can exceed this limit because of:

- A slow query plan.
- Large amounts of data.
- Resource contention.
- Lock waits.
- Other execution delays.

### 13.3 idle_in_transaction_session_timeout

```sql
SET idle_in_transaction_session_timeout = '60s';
```

This setting terminates sessions that remain idle while holding an open transaction for longer than the configured interval.

It can help prevent abandoned transactions from retaining resources indefinitely.

### 13.4 Comparison

| Setting | Controls |
|---|---|
| lock_timeout | Time spent waiting for a lock. |
| statement_timeout | Total statement execution time. |
| idle_in_transaction_session_timeout | Time a session can remain idle inside an open transaction. |

These settings should be selected according to application requirements rather than assigned arbitrary values.

## 14. Optimistic and Pessimistic Concurrency Control

Applications can use different strategies to coordinate concurrent modifications.

Two important approaches are optimistic and pessimistic concurrency control.

### 14.1 Pessimistic concurrency control

Pessimistic concurrency control assumes that conflicts are sufficiently likely to justify acquiring locks before performing critical work.

Example:

```sql
BEGIN;

SELECT *
FROM vehicles
WHERE vehicle_id = 101
FOR UPDATE;

UPDATE vehicles
SET status = 'IN_MAINTENANCE'
WHERE vehicle_id = 101;

COMMIT;
```

The transaction locks the selected row before modifying it.

Advantages:

- Prevents conflicting operations from modifying the locked row simultaneously.
- Useful when contention is expected.
- Provides explicit coordination.

Disadvantages:

- Can create blocking.
- Requires careful transaction management.
- Long transactions can reduce concurrency.

### 14.2 Optimistic concurrency control

Optimistic concurrency control assumes that conflicts are relatively uncommon.

The application reads a record and later updates it only if the record has not changed since it was read.

A common implementation uses a version column.

Example table:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT PRIMARY KEY,
    mileage INTEGER NOT NULL,
    version INTEGER NOT NULL DEFAULT 1
);
```

The application reads:

```sql
SELECT
    vehicle_id,
    mileage,
    version
FROM vehicles
WHERE vehicle_id = 101;
```

Suppose it receives:

```text
vehicle_id: 101
mileage: 25000
version: 1
```

It later attempts:

```sql
UPDATE vehicles
SET
    mileage = 25500,
    version = version + 1
WHERE vehicle_id = 101
  AND version = 1;
```

The application checks the affected-row count.

If one row was updated, the operation succeeded.

If zero rows were updated, the record may have changed since it was read.

The application can then reload the record and resolve the conflict.

### 14.3 Comparison

| Characteristic | Pessimistic | Optimistic |
|---|---|---|
| Main strategy | Lock before modifying. | Detect conflicts during update. |
| Expected conflicts | Relatively frequent. | Relatively uncommon. |
| Blocking | More likely. | Usually less explicit blocking at application level. |
| Conflict handling | Wait, fail, or coordinate through locks. | Detect and retry or ask for user resolution. |
| Typical use | Critical shared records. | Applications with relatively low contention. |

Neither approach is universally better.

The correct choice depends on the workload, business rules, and expected contention.

## 15. Advisory Locks

PostgreSQL also supports advisory locks.

Unlike ordinary table and row locks, advisory locks are application-defined coordination mechanisms.

They are not automatically associated with a specific table row.

For example:

```sql
SELECT pg_advisory_lock(12345);
```

This acquires a session-level advisory lock identified by a key.

To release it:

```sql
SELECT pg_advisory_unlock(12345);
```

PostgreSQL also supports transaction-level advisory locks:

```sql
SELECT pg_advisory_xact_lock(12345);
```

A transaction-level advisory lock is automatically released when the transaction ends.

### 15.1 When are advisory locks useful?

Examples include:

- Preventing two workers from processing the same logical task.
- Coordinating a scheduled job.
- Protecting a custom application workflow.
- Serializing operations that do not correspond to a single database row.

### 15.2 Important considerations

Advisory locks require application-level coordination.

PostgreSQL does not automatically enforce a business rule merely because an advisory lock exists.

All processes that need to respect the rule must follow the same locking convention.

Use advisory locks carefully and document their keys and purpose.

## 16. MVCC, VACUUM, and Long-Running Transactions

MVCC requires PostgreSQL to manage multiple row versions.

When rows are updated or deleted, older versions may remain physically present for some time.

These versions cannot be removed while they might still be needed by active transactions.

PostgreSQL uses VACUUM to help reclaim space occupied by obsolete row versions and maintain database health.

### 16.1 Why old row versions remain

Consider:

```text
Transaction A:
Begins and establishes a snapshot.

Transaction B:
Updates a row several times.

Transaction B:
Commits.
```

Transaction A may still need an older version depending on its snapshot.

PostgreSQL must preserve row versions that could still be visible to active transactions.

A long-running transaction can therefore delay cleanup.

### 16.2 What is table bloat?

Table bloat refers to excessive storage occupied by obsolete or inefficiently organized data structures.

It can result from frequent updates and deletes combined with delayed cleanup.

Potential consequences include:

- Increased disk usage.
- More pages to scan.
- Additional I/O.
- Reduced query performance.

### 16.3 Why VACUUM matters

PostgreSQL's VACUUM process helps:

- Remove dead row versions when they are no longer needed.
- Make space available for reuse.
- Maintain visibility information.
- Prevent transaction ID wraparound problems.

PostgreSQL normally uses autovacuum to perform maintenance automatically.

However, autovacuum effectiveness can be affected by long-running transactions and other operational conditions.

**Architecture lesson:**

Transaction duration affects more than locks. It can also affect storage maintenance and long-term database performance.

## 17. Practical Laboratory — Understanding Concurrency

This laboratory uses PostgreSQL and two separate database sessions.

You can open two query tabs in pgAdmin or two terminal connections through psql.

The goal is to observe how transactions interact.

### 17.1 Prepare the environment

Run this script once:

```sql
DROP TABLE IF EXISTS concurrency_lab;

CREATE TABLE concurrency_lab (
    vehicle_id INTEGER PRIMARY KEY,
    mileage INTEGER NOT NULL,
    status VARCHAR(30) NOT NULL,
    version INTEGER NOT NULL DEFAULT 1
);

INSERT INTO concurrency_lab (
    vehicle_id,
    mileage,
    status
)
VALUES
    (101, 25000, 'AVAILABLE'),
    (102, 40000, 'AVAILABLE'),
    (103, 55000, 'AVAILABLE');
```

Verify the data:

```sql
SELECT *
FROM concurrency_lab
ORDER BY vehicle_id;
```

### 17.2 Laboratory A — Observe a blocking update

**Session A**

```sql
BEGIN;

UPDATE concurrency_lab
SET mileage = 25500
WHERE vehicle_id = 101;

-- Do not commit yet.
```

**Session B**

```sql
UPDATE concurrency_lab
SET mileage = 26000
WHERE vehicle_id = 101;
```

Session B should wait because Session A holds a conflicting row lock.

Now return to Session A:

```sql
COMMIT;
```

Session B can continue.

Check the final value:

```sql
SELECT *
FROM concurrency_lab
WHERE vehicle_id = 101;
```

Observe:

- Which session acquired the lock first?
- Which session waited?
- What was the final mileage?
- Why did PostgreSQL coordinate the updates?

### 17.3 Laboratory B — Observe MVCC

Reset the record:

```sql
UPDATE concurrency_lab
SET mileage = 25000
WHERE vehicle_id = 101;
```

**Session A**

```sql
BEGIN ISOLATION LEVEL READ COMMITTED;

SELECT mileage
FROM concurrency_lab
WHERE vehicle_id = 101;
```

Expected result:

```text
25000
```

**Session B**

```sql
UPDATE concurrency_lab
SET mileage = 25500
WHERE vehicle_id = 101;

COMMIT;
```

Return to Session A:

```sql
SELECT mileage
FROM concurrency_lab
WHERE vehicle_id = 101;
```

Expected result:

```text
25500
```

Commit Session A:

```sql
COMMIT;
```

Repeat the experiment with REPEATABLE READ.

In that case, Session A should continue seeing the same snapshot for both reads.

### 17.4 Laboratory C — Observe NOWAIT

**Session A**

```sql
BEGIN;

SELECT *
FROM concurrency_lab
WHERE vehicle_id = 102
FOR UPDATE;
```

Keep the transaction open.

**Session B**

```sql
BEGIN;

SELECT *
FROM concurrency_lab
WHERE vehicle_id = 102
FOR UPDATE NOWAIT;
```

Expected behavior:

Session B immediately receives an error if it cannot acquire the required lock.

Close both transactions:

```sql
ROLLBACK;
```

Run the rollback in each session.

### 17.5 Laboratory D — Observe SKIP LOCKED

**Session A**

```sql
BEGIN;

SELECT *
FROM concurrency_lab
WHERE vehicle_id = 103
FOR UPDATE;
```

**Session B**

```sql
BEGIN;

SELECT *
FROM concurrency_lab
WHERE vehicle_id IN (102, 103)
ORDER BY vehicle_id
FOR UPDATE SKIP LOCKED;
```

Session B should skip the locked row and may acquire the lock on vehicle 102.

Finish both transactions:

```sql
ROLLBACK;
```

### 17.6 Laboratory E — Inspect active locks

While one session is holding a transaction open, execute this query from another session:

```sql
SELECT
    pid,
    state,
    wait_event_type,
    wait_event,
    query
FROM pg_stat_activity
WHERE datname = current_database();
```

Inspect the locks:

```sql
SELECT
    pid,
    locktype,
    relation::regclass AS relation,
    mode,
    granted
FROM pg_locks
ORDER BY pid;
```

Identify:

- The session holding a lock.
- The session waiting for a lock.
- The lock mode.
- Whether the lock has been granted.

Clean up after the experiment:

```sql
ROLLBACK;
```

Use this in any session that still has an open transaction.

## 18. Applying Concurrency Control to SIAPV

Consider a system managing approximately 200 vehicles and machinery units.

Multiple employees may use the system during the same working period.

The architecture must protect important records while allowing normal operations to continue.

### 18.1 Scenario: Two employees edit the same vehicle

Employee A changes the vehicle status.

Employee B changes the assigned department.

These changes affect the same record but may represent independent business operations.

A well-designed application should:

- Define which fields each operation can modify.
- Avoid overwriting unrelated fields.
- Use transactions for logically related changes.
- Detect conflicting updates where necessary.

An update that modifies only the intended column is safer than replacing an entire record using stale application data.

### 18.2 Scenario: Simultaneous mileage updates

Suppose two employees register separate mileage increases.

A direct increment can avoid an application-level read-modify-write race:

```sql
UPDATE vehicles
SET mileage = mileage + 500
WHERE vehicle_id = 101;
```

However, real mileage records usually require additional validation.

For example:

- The new reading should not be lower than an accepted previous reading.
- The reading should be associated with the correct vehicle.
- The operator and timestamp should be recorded.
- Duplicate submissions should be controlled.

A transaction can combine the validation and insertion of the mileage event.

### 18.3 Scenario: Assigning machinery to a work crew

Suppose a machine cannot be assigned to two active crews simultaneously.

This is a business rule.

A transaction should verify availability and create the assignment in a way that prevents conflicting assignments.

Possible strategies include:

- A unique constraint or partial unique index for active assignments.
- Row-level locking on the machinery record.
- SERIALIZABLE isolation for more complex cross-record rules.
- Application-level conflict detection.

The database constraint is especially important because it protects the rule even if two application requests arrive simultaneously.

### 18.4 Scenario: Maintenance registration

A maintenance operation may involve:

1. Creating a maintenance record.
2. Updating the vehicle's status.
3. Recording the current mileage.
4. Registering the responsible workshop.
5. Saving the supporting documentation reference.

These changes should generally be handled in one transaction when they represent a single logical operation.

```sql
BEGIN;

-- Register maintenance.
-- Update vehicle status.
-- Record relevant operational information.

COMMIT;
```

If one required operation fails, the transaction can be rolled back.

### 18.5 Architectural recommendations

| Situation | Possible strategy |
|---|---|
| Frequent vehicle consultations | Use MVCC and appropriate indexes. |
| Simultaneous updates to one vehicle | Use row-level locking or atomic updates. |
| Independent field updates | Update only the intended columns. |
| Conflicting user edits | Use a version column for optimistic concurrency. |
| Unique active machinery assignment | Enforce a database constraint. |
| Complex cross-record business rules | Evaluate SERIALIZABLE isolation. |
| Background job processing | Consider FOR UPDATE SKIP LOCKED. |
| Long-running operations | Keep transactions short and monitor sessions. |

The objective is not to lock the entire system.

The objective is to protect the smallest necessary resource for the shortest reasonable time.

## 19. Important Architecture Principles

When designing a concurrent database system, keep these principles in mind:

1. Concurrency is necessary for multi-user applications.
2. MVCC allows PostgreSQL to provide consistent row visibility without making every reader wait for writers.
3. MVCC does not eliminate locks.
4. Row-level locks coordinate conflicting modifications.
5. Table-level locks coordinate operations on database relations.
6. Isolation levels determine which concurrent effects a transaction can observe.
7. Deadlocks are cycles of waiting, not simply long lock waits.
8. PostgreSQL detects deadlocks and aborts one transaction.
9. Long-running transactions can interfere with VACUUM.
10. Database constraints should enforce critical business rules.
11. Applications must handle retryable transaction errors.
12. Short and well-defined transactions generally improve concurrency.

## 20. Knowledge Check

Answer these questions without consulting the previous sections.

1. What is the difference between concurrency and parallelism?
2. Why do database systems require concurrency control?
3. What problem does MVCC solve?
4. Does MVCC eliminate the need for locks?
5. What is a transaction snapshot?
6. What is a dirty read?
7. What is a non-repeatable read?
8. What is a phantom read?
9. What is a lost update?
10. What is write skew?
11. What is the default isolation level in PostgreSQL?
12. How does REPEATABLE READ differ from READ COMMITTED?
13. What does SERIALIZABLE attempt to guarantee?
14. What is the difference between table-level and row-level locks?
15. What does SELECT FOR UPDATE do?
16. When would NOWAIT be useful?
17. Why is SKIP LOCKED useful for job queues?
18. What causes a deadlock?
19. How does PostgreSQL resolve a deadlock?
20. What is the difference between lock_timeout and statement_timeout?
21. Why can long-running transactions affect VACUUM?
22. How does optimistic concurrency control work?
23. What is an advisory lock?
24. Why should applications implement retry logic?

## 21. Practical Exercises

### Exercise 1 — Identify the concurrency problem

For each situation, identify the most relevant concurrency issue.

- Two users overwrite each other's changes.
- A transaction reads a row twice and gets different values.
- Two transactions wait indefinitely for resources held by one another until the DBMS detects the cycle.
- A repeated query returns additional matching rows.
- Two transactions independently update different records but violate a shared business rule.

### Exercise 2 — Design a locking strategy

A vehicle can only be assigned to one active work crew.

Describe:

- The business rule.
- The database constraint that could enforce it.
- Whether a row-level lock is appropriate.
- Which operations should be part of the transaction.
- How the application should respond if another employee has already assigned the vehicle.

### Exercise 3 — Diagnose a production problem

A PostgreSQL application has become slow.

You discover:

- Several sessions are waiting for locks.
- One session has remained idle in transaction for 20 minutes.
- Some queries are taking longer than expected.
- Autovacuum is unable to clean certain old row versions.

Explain how these conditions may be related and propose a diagnostic and corrective strategy.

### Exercise 4 — Choose an isolation level

For each workload, recommend an isolation level and explain your reasoning:

- A dashboard that displays current vehicle information.
- A user updating a single vehicle.
- A financial operation involving multiple related records.
- A business rule involving several rows that must remain consistent under concurrent changes.

There may be more than one valid solution. Justify the trade-offs.

## 22. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Concurrency | Concurrencia |
| Parallelism | Paralelismo |
| Concurrency control | Control de concurrencia |
| Session | Sesión |
| Transaction | Transacción |
| Snapshot | Instantánea de visibilidad |
| Row version | Versión de una fila |
| MVCC | Control de concurrencia multiversión |
| Isolation level | Nivel de aislamiento |
| Dirty read | Lectura sucia |
| Non-repeatable read | Lectura no repetible |
| Phantom read | Lectura fantasma |
| Lost update | Actualización perdida |
| Write skew | Anomalía de actualización divergente |
| Lock | Bloqueo |
| Lock mode | Modo de bloqueo |
| Row-level lock | Bloqueo a nivel de fila |
| Table-level lock | Bloqueo a nivel de tabla |
| Lock wait | Espera por bloqueo |
| Blocking | Bloqueo entre sesiones |
| Deadlock | Interbloqueo |
| Lock contention | Contención por bloqueos |
| Lock timeout | Tiempo límite de espera por bloqueo |
| Advisory lock | Bloqueo asesor |
| Optimistic concurrency | Concurrencia optimista |
| Pessimistic concurrency | Concurrencia pesimista |
| Serialization failure | Error de serialización |
| Transaction retry | Reintento de transacción |
| Vacuum | Proceso de limpieza y mantenimiento |
| Table bloat | Crecimiento excesivo de almacenamiento |
| Predicate lock | Bloqueo de predicado |
| Transaction identifier | Identificador de transacción |

## 23. Final Mental Model

A concurrent database system must coordinate many independent transactions without sacrificing correctness.

Think about its responsibilities as follows:

```text
Multiple Users and Applications
              |
              v
       Database Sessions
              |
              v
          Transactions
              |
              v
       Concurrency Control
              |
       +------+------+
       |             |
       v             v
      MVCC          Locks
       |             |
       v             v
 Row Visibility   Conflict Control
       |             |
       +------+------+
              |
              v
      Consistent Database
```

MVCC determines which row versions transactions can observe.

Locks coordinate operations that conflict with one another.

Isolation levels define the guarantees provided to transactions.

Deadlock detection and retry strategies help the system recover from conflicts.

Together, these mechanisms allow a relational database to support many simultaneous users while preserving data integrity.

---

## 24. Related Notes

### Previous concepts

- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[03 - Storage Pages and Tables]]
- [[04 - Indexes]]
- [[05 - Transactions and ACID]]

### Next topics

- [[01 - Client Server Architecture]]
- [[02 - Database Scalability]]
- [[03 - Replication]]
- [[04 - Partitioning]]
- [[05 - Sharding]]

### Practical technologies

- PostgreSQL
- SQL
- psql
- pgAdmin
- Python database drivers
- Transaction management in application development

---

## 25. Completion Criteria

You should be able to:

- [ ] Explain concurrency and parallelism.
- [ ] Describe the purpose of MVCC.
- [ ] Explain how PostgreSQL determines row visibility.
- [ ] Recognize common concurrency anomalies.
- [ ] Compare PostgreSQL isolation levels.
- [ ] Distinguish table-level and row-level locks.
- [ ] Use SELECT FOR UPDATE, NOWAIT, and SKIP LOCKED.
- [ ] Inspect active sessions and locks.
- [ ] Identify blocking transactions.
- [ ] Explain how deadlocks occur and how PostgreSQL handles them.
- [ ] Compare optimistic and pessimistic concurrency control.
- [ ] Explain why long-running transactions affect VACUUM.
- [ ] Design a basic concurrency strategy for a multi-user application.
