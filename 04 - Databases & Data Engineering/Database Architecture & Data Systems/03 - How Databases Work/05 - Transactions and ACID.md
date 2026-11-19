
---
title: "Transactions and ACID"
aliases:
  - "Database Transactions"
  - "ACID Properties"
  - "Transaction Management"
  - "PostgreSQL Transactions"
type: learning-note
domain: technology
category: database-systems
subject: transaction-management
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - database-transactions
  - acid
  - transaction-management
  - postgresql
  - data-integrity
  - concurrency
  - recovery
  - sql

concepts:
  - transaction
  - transaction-boundary
  - atomicity
  - consistency
  - isolation
  - durability
  - commit
  - rollback
  - savepoint
  - autocommit
  - transaction-state
  - write-ahead-log
  - checkpoint
  - crash-recovery
  - isolation-level
  - deadlock
  - two-phase-commit

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Database Management Systems]]"
  - "[[02 - How a Query is Executed]]"
  - "[[03 - Storage Pages and Tables]]"
  - "[[04 - Indexes]]"
  - "[[03 - Keys and Constraints]]"

related:
  - "[[06 - Concurrency and Locks]]"
  - "[[06 - High Availability]]"
  - "[[07 - Backup and Disaster Recovery]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[02 - PostgreSQL Processes]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"

tools:
  - PostgreSQL
  - SQL
  - pgAdmin
  - psql
  - BEGIN
  - COMMIT
  - ROLLBACK
  - SAVEPOINT
  - EXPLAIN

learning_objectives:
  - "Explain what a database transaction is and why it is necessary."
  - "Understand the four ACID properties."
  - "Use BEGIN, COMMIT, and ROLLBACK in PostgreSQL."
  - "Understand transaction boundaries and autocommit behavior."
  - "Use SAVEPOINT to manage partial rollback."
  - "Explain how PostgreSQL uses Write-Ahead Logging."
  - "Understand the relationship between transactions and crash recovery."
  - "Distinguish transaction isolation from data integrity."
  - "Recognize common transaction errors and operational risks."
  - "Connect transaction design with real application workflows."

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 05 — Transactions and ACID

## 1. Introduction

A database application performs many operations:

- Inserting records.
- Updating information.
- Deleting obsolete data.
- Registering payments.
- Modifying inventory.
- Recording maintenance events.
- Generating operational information.

Some operations are independent.

Others must be executed together.

Consider a fleet management application that registers a vehicle maintenance event.

The process may require:

1. Creating a maintenance record.
2. Updating the vehicle's operational status.
3. Registering a new meter reading.
4. Recording the employee responsible for the operation.

What happens if the first three operations succeed but the fourth fails?

The database could be left with incomplete or inconsistent information.

A **transaction** provides a mechanism for treating related operations as a single logical unit of work.

The central principle is:

> A transaction groups database operations so that they can be committed together or rolled back when the operation cannot be completed successfully.

To understand reliable database systems, we must understand transactions and their four fundamental properties: ACID.

---

## 2. What Is a Database Transaction?

A database transaction is a logical unit of work consisting of one or more database operations.

A transaction can include:

- SQL statements.
- Data insertion.
- Data modification.
- Data deletion.
- Queries that participate in a larger operation.
- Changes to multiple related tables.

A transaction has a beginning, an execution phase, and a completion decision.

Conceptually:

```text
Transaction
    |
    v
BEGIN
    |
    v
Execute database operations
    |
    +---- All operations succeed
    |             |
    |             v
    |           COMMIT
    |             |
    |             v
    |       Changes committed
    |
    +---- Operation fails
                  |
                  v
               ROLLBACK
                  |
                  v
          Changes undone
```

This is a simplified representation of transaction control.

### 2.1 A simple example

Imagine transferring money between two bank accounts.

Account A contains:

```text
Balance: 5,000
```

Account B contains:

```text
Balance: 2,000
```

A transfer of 1,000 requires two operations:

1. Subtract 1,000 from Account A.
2. Add 1,000 to Account B.

The expected result is:

```text
Account A: 4,000
Account B: 3,000
```

Both changes must succeed together.

If only the first operation succeeds, money appears to disappear.

If only the second succeeds, money appears to be created.

A transaction protects the logical integrity of the transfer.

### 2.2 Transaction vs. SQL statement

A SQL statement is an individual command.

A transaction can contain multiple SQL statements.

| SQL statement | Transaction |
|---|---|
| Individual database command | Logical unit of work |
| Can be part of a transaction | Groups one or more commands |
| Example: UPDATE | Example: transfer between accounts |
| Has its own execution result | Has a final commit or rollback decision |

A single SQL statement is normally executed transactionally by PostgreSQL, even when the user does not explicitly issue `BEGIN`.

### 2.3 Transaction boundaries

A transaction boundary defines where a transaction begins and where it ends.

For example:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 2;

COMMIT;
```

The `BEGIN` statement starts the transaction.

The `COMMIT` statement completes it successfully.

The two updates belong to the same transaction.

---

## 3. The ACID Properties

ACID is a set of four properties that describe important guarantees of reliable database transactions.

The acronym means:

| Property | Meaning |
|---|---|
| A | Atomicity |
| C | Consistency |
| I | Isolation |
| D | Durability |

Each property addresses a different reliability concern.

```text
ACID
│
├── Atomicity
│   └── All or nothing
│
├── Consistency
│   └── Preserve valid database states
│
├── Isolation
│   └── Control concurrent transaction effects
│
└── Durability
    └── Preserve committed changes
```

Let's examine each property.

---

## 4. Atomicity

### 4.1 Definition

Atomicity means that a transaction is treated as an indivisible logical unit.

Its operations are committed together or rolled back together.

The common expression is:

**All or nothing.**

A transaction should not leave only part of its intended changes committed.

### 4.2 Example: bank transfer

Suppose we execute:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 2;

COMMIT;
```

If both operations succeed, PostgreSQL commits the transaction.

If the second operation fails and the transaction is rolled back, the first update is also undone.

### 4.3 Atomicity diagram

```text
Transaction: Transfer 1,000

Operation 1:
Subtract 1,000 from Account A
        |
        v
Operation 2:
Add 1,000 to Account B
        |
        v
Commit transaction
        |
        v
Both changes become committed
```

If the transaction fails:

```text
Transaction: Transfer 1,000

Operation 1:
Subtract 1,000 from Account A
        |
        v
Operation 2:
FAILS
        |
        v
ROLLBACK
        |
        v
Previous database state restored
```

### 4.4 Atomicity in PostgreSQL

PostgreSQL uses transaction management mechanisms and Write-Ahead Logging to support reliable transactional behavior.

If a transaction is rolled back, its changes are not visible as committed data.

PostgreSQL does not need to physically erase every modified byte immediately. It manages transaction visibility and recovery using its internal storage mechanisms.

### 4.5 When atomicity matters

Atomicity is important when:

- Transferring money.
- Processing orders.
- Updating inventory.
- Registering maintenance events.
- Creating related records.
- Changing multiple tables in one workflow.

### 4.6 Architectural principle

If several database operations represent one business action, evaluate whether they should belong to the same transaction.

---

## 5. Consistency

### 5.1 Definition

Consistency means that a transaction must preserve the database's defined integrity rules.

A successful transaction moves the database from one valid state to another valid state.

It must not leave the database violating the constraints and rules enforced by the database.

### 5.2 What defines a valid database state?

Database consistency can be supported by:

- Primary keys.
- Foreign keys.
- Unique constraints.
- NOT NULL constraints.
- CHECK constraints.
- Data types.
- Referential integrity.
- Additional business rules implemented through application logic or database mechanisms.

Example:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT PRIMARY KEY,
    plate_number VARCHAR(15) UNIQUE,
    manufacture_year INTEGER
        CHECK (manufacture_year BETWEEN 1900 AND 2100)
);
```

These constraints define some conditions that valid records must satisfy.

### 5.3 Example of a consistency violation

Consider:

```sql
INSERT INTO vehicles (
    vehicle_id,
    plate_number,
    manufacture_year
)
VALUES (
    1,
    'ABC-123',
    1750
);
```

The CHECK constraint rejects the operation because the year is outside the allowed range.

The database prevents a transaction from successfully committing a change that violates this enforced rule.

### 5.4 Consistency and business rules

Suppose a vehicle must not have a negative mileage reading.

A database constraint can enforce this:

```sql
CREATE TABLE meter_readings (
    reading_id BIGINT PRIMARY KEY,
    vehicle_id BIGINT NOT NULL,
    reading_value NUMERIC(12, 2) NOT NULL
        CHECK (reading_value >= 0)
);
```

The database can reject negative values.

However, some business rules require more complex logic.

For example:

- A maintenance date cannot precede the vehicle's registration date.
- A new meter reading should not be lower than the previous reading, except when a meter replacement is documented.
- A vehicle cannot be assigned to two incompatible operational activities simultaneously.

Such rules may require application logic, database triggers, additional constraints, or carefully designed transactions.

### 5.5 Consistency is not the same as correctness of business logic

ACID consistency does not automatically mean that every business decision is correct.

For example, if an application incorrectly records a maintenance cost of 50,000 instead of 5,000 and that value satisfies all database constraints, PostgreSQL may commit it successfully.

The database can enforce defined rules, but it cannot automatically know every intended business requirement.

### 5.6 Architectural principle

Important integrity rules should be enforced as close to the data as practical.

Do not rely exclusively on the user interface to protect database consistency.

---

## 6. Isolation

### 6.1 Definition

Isolation controls how concurrent transactions interact.

A database may have many users executing operations at the same time.

For example:

- One employee updates a vehicle.
- Another employee reads its information.
- A reporting process queries historical records.
- An administrative process updates related information.

Without concurrency control, these operations could interfere with one another.

Isolation helps control which changes one transaction can observe from another.

### 6.2 Concurrent transactions

Imagine two transactions reading the same account balance.

Initial balance:

```text
1,000
```

Transaction A wants to subtract 100.

Transaction B wants to subtract 200.

Both transactions begin at approximately the same time.

A simplified problematic sequence could be:

```text
Transaction A              Transaction B
      |                           |
Read balance = 1,000       Read balance = 1,000
      |                           |
Calculate 900              Calculate 800
      |                           |
Write 900                   Write 800
      |                           |
Commit                      Commit
```

The final balance could become 800 even though the expected result is 700.

This is a simplified lost-update scenario.

Proper transaction design and concurrency control are necessary to prevent such problems.

### 6.3 Concurrency anomalies

Common anomalies include:

| Anomaly | Description |
|---|---|
| Dirty read | Reading uncommitted changes from another transaction |
| Non-repeatable read | Reading the same row twice and observing different committed values |
| Phantom read | Repeating a predicate query and observing additional or missing matching rows |
| Lost update | One transaction overwrites another transaction's change |
| Write skew | Concurrent transactions make individually valid changes that jointly violate a business rule |

The exact anomalies permitted depend on the database engine and isolation level.

PostgreSQL's MVCC implementation prevents dirty reads, even when the transaction requests the READ UNCOMMITTED isolation level. PostgreSQL treats READ UNCOMMITTED as READ COMMITTED.

### 6.4 Isolation and MVCC

PostgreSQL uses MVCC to manage concurrent access to row versions.

Instead of requiring every reader to wait for every writer, MVCC allows transactions to work with versions of records according to their visibility rules.

This helps support concurrent reads and writes.

However, MVCC does not eliminate every concurrency problem.

Locks and transaction isolation levels remain important.

We will study these mechanisms in more detail in:

- [[06 - Concurrency and Locks]]

### 6.5 Architectural principle

Isolation is not simply about preventing all transactions from interacting.

It is about defining which interactions are permitted while maintaining correct results.

---

## 7. Durability

### 7.1 Definition

Durability means that once a transaction is successfully committed, its changes are preserved against failures covered by the database's durability guarantees.

For example, if PostgreSQL confirms a successful commit, the transaction should not simply disappear because the server process crashes immediately afterward.

### 7.2 Why durability matters

Imagine an application registering an important maintenance event.

The user receives confirmation that the operation was successful.

If the database server crashes and the committed record disappears after recovery, the application has failed to provide reliable persistence.

Durability protects committed changes.

### 7.3 Write-Ahead Logging

PostgreSQL uses **Write-Ahead Logging (WAL)** to support durability and crash recovery.

The central WAL principle is:

> The log records describing a change must be safely written before the corresponding modified data pages are required to be written to their final data files.

This allows PostgreSQL to recover committed changes after a crash.

### 7.4 Simplified WAL process

```text
Transaction modifies data
          |
          v
WAL records generated
          |
          v
WAL records written
          |
          v
COMMIT requested
          |
          v
Required WAL is flushed
          |
          v
Commit acknowledged
          |
          v
Data pages may be written later
```

This is a simplified conceptual sequence.

PostgreSQL can group commit operations, and the exact timing depends on configuration and execution conditions.

### 7.5 Why not write every modified page immediately?

Writing every modified database page to its final location at every commit would be inefficient.

Instead, PostgreSQL uses WAL to record changes and can write modified data pages later.

This reduces the need to force all affected data pages to disk at each commit.

### 7.6 Durability and configuration

PostgreSQL durability behavior depends on configuration and the environment.

Relevant settings include:

- `fsync`
- `synchronous_commit`
- `wal_level`
- `synchronous_standby_names`

For example, disabling `fsync` can compromise crash safety.

Setting `synchronous_commit` to a less strict mode can allow PostgreSQL to acknowledge a commit before the corresponding WAL record is flushed to durable storage, creating a possible loss window after a crash.

These settings involve trade-offs between performance and durability guarantees.

Do not change them in production without understanding the consequences.

### 7.7 Durability is not the same as backup

WAL and crash recovery help preserve database consistency after failures.

However, they do not replace backups.

A committed transaction can still contain incorrect information.

If a user accidentally deletes important records and commits the transaction, durability means the deletion is preserved.

Recovering the previous business state may require:

- Backups.
- WAL archiving.
- Point-in-time recovery.
- Other recovery procedures.

Review:

- [[07 - Backup and Disaster Recovery]]

---

## 8. ACID Summary

| Property | Main question | Main guarantee |
|---|---|---|
| Atomicity | Do all operations succeed together? | All or nothing |
| Consistency | Are integrity rules preserved? | Valid database state |
| Isolation | How do concurrent transactions interact? | Controlled visibility and concurrency |
| Durability | Are committed changes preserved? | Persistence after commit |

### 8.1 One scenario, four properties

Imagine registering a maintenance event.

**Atomicity**

The maintenance record and related status changes are committed together or rolled back.

**Consistency**

The record must reference an existing vehicle and satisfy defined data rules.

**Isolation**

Other transactions must not observe inconsistent intermediate states.

**Durability**

Once the operation is committed, the database must preserve it according to its durability guarantees.

ACID is not four unrelated features.

The properties work together to provide reliable transaction processing.

---

## 9. Transaction Control in PostgreSQL

PostgreSQL provides SQL commands for controlling transactions.

The most important are:

| Command | Purpose |
|---|---|
| `BEGIN` | Starts an explicit transaction |
| `COMMIT` | Commits the current transaction |
| `ROLLBACK` | Aborts the current transaction |
| `SAVEPOINT` | Defines a rollback point |
| `ROLLBACK TO SAVEPOINT` | Reverts changes after a savepoint |
| `RELEASE SAVEPOINT` | Removes a savepoint |

### 9.1 BEGIN

Starts an explicit transaction.

```sql
BEGIN;
```

### 9.2 COMMIT

Makes the transaction's changes committed.

```sql
COMMIT;
```

After a successful commit, the transaction is complete.

### 9.3 ROLLBACK

Aborts the transaction and cancels its uncommitted changes.

```sql
ROLLBACK;
```

### 9.4 Complete example

```sql
BEGIN;

INSERT INTO vehicles (
    vehicle_id,
    plate_number
)
VALUES (
    1001,
    'ABC-123'
);

UPDATE vehicles
SET plate_number = 'DEF-456'
WHERE vehicle_id = 1001;

COMMIT;
```

Both operations belong to one transaction.

If an error occurs before the commit, the transaction can be rolled back.

---

## 10. Autocommit Behavior

Many SQL clients operate with autocommit enabled by default.

Autocommit means that individual statements are automatically committed when they complete successfully, unless an explicit transaction is active.

For example:

```sql
INSERT INTO vehicles (
    vehicle_id,
    plate_number
)
VALUES (
    1001,
    'ABC-123'
);
```

When executed outside an explicit transaction, PostgreSQL executes this statement as its own transaction.

The successful statement is committed automatically.

### 10.1 Explicit transaction vs. autocommit

| Autocommit | Explicit transaction |
|---|---|
| Each statement is normally its own transaction | Multiple statements can share one transaction |
| No manual COMMIT required for each successful statement | Requires a commit or rollback decision |
| Useful for independent operations | Useful for related operations |
| Cannot automatically group separate statements into one business unit | Provides explicit transaction boundaries |

### 10.2 Important distinction

Autocommit does not mean that PostgreSQL executes SQL without transactions.

Every database statement executes transactionally.

Autocommit determines whether the client automatically completes the transaction after each statement.

---

## 11. SAVEPOINT: Partial Rollback

Sometimes a transaction contains multiple steps, and only part of the operation needs to be reversed.

PostgreSQL supports savepoints.

A savepoint marks a position within a transaction.

### 11.1 Example

```sql
BEGIN;

INSERT INTO vehicles (
    vehicle_id,
    plate_number
)
VALUES (
    1001,
    'ABC-123'
);

SAVEPOINT after_vehicle_insert;

INSERT INTO maintenance_records (
    maintenance_id,
    vehicle_id,
    maintenance_date
)
VALUES (
    5001,
    1001,
    CURRENT_DATE
);

ROLLBACK TO SAVEPOINT after_vehicle_insert;

COMMIT;
```

The second insert is rolled back.

The first insert remains part of the transaction and is committed.

### 11.2 Savepoint diagram

```text
BEGIN
  |
  v
Operation 1
  |
  v
SAVEPOINT A
  |
  v
Operation 2
  |
  v
Operation 3
  |
  v
ROLLBACK TO A
  |
  v
Operation 1 remains
  |
  v
COMMIT
```

### 11.3 RELEASE SAVEPOINT

A savepoint can be explicitly removed:

```sql
RELEASE SAVEPOINT after_vehicle_insert;
```

Releasing a savepoint does not commit the transaction.

It only removes that rollback marker.

### 11.4 When to use savepoints

Savepoints can be useful when:

- Processing a group of related operations.
- Handling recoverable errors.
- Executing optional steps.
- Implementing complex transaction workflows.

However, they should not be used to hide poorly designed transaction boundaries.

---

## 12. Transaction States and Errors

A transaction does not always finish successfully.

It can encounter:

- Constraint violations.
- Deadlocks.
- Lock timeouts.
- Connection failures.
- Serialization failures.
- Application errors.

### 12.1 Failed transaction state

In PostgreSQL, when a statement causes an error inside an explicit transaction, the transaction generally enters an aborted state.

Further ordinary SQL commands are rejected until the transaction is rolled back, or rolled back to a suitable savepoint.

Example:

```sql
BEGIN;

INSERT INTO vehicles (
    vehicle_id,
    plate_number
)
VALUES (
    1001,
    'ABC-123'
);

INSERT INTO vehicles (
    vehicle_id,
    plate_number
)
VALUES (
    1001,
    'DEF-456'
);
```

The second statement violates the primary-key constraint if the first insert has succeeded in the same transaction.

The transaction enters an aborted state.

A later command such as:

```sql
SELECT *
FROM vehicles;
```

will not execute normally in that failed transaction.

The transaction must be recovered:

```sql
ROLLBACK;
```

Or, if a savepoint was established before the failed operation:

```sql
ROLLBACK TO SAVEPOINT savepoint_name;
```

### 12.2 Why this matters in applications

An application must handle database errors correctly.

A common pattern is:

```text
Start transaction
      |
      v
Execute operations
      |
      +---- Success
      |       |
      |       v
      |     Commit
      |
      +---- Failure
              |
              v
           Rollback
              |
              v
         Return error
```

The application should not assume that a partially executed transaction can simply continue.

---

## 13. Isolation Levels

Isolation levels define the visibility guarantees and concurrency behavior of transactions.

The SQL standard defines four commonly discussed isolation levels.

| Isolation level | General description |
|---|---|
| READ UNCOMMITTED | Allows the weakest visibility guarantees |
| READ COMMITTED | Reads committed data according to statement-level snapshots |
| REPEATABLE READ | Maintains a stable transaction-level snapshot |
| SERIALIZABLE | Provides the strongest serial-equivalence guarantee |

PostgreSQL supports:

- READ COMMITTED.
- REPEATABLE READ.
- SERIALIZABLE.

PostgreSQL treats READ UNCOMMITTED as READ COMMITTED.

### 13.1 READ COMMITTED

This is PostgreSQL's default isolation level.

Each statement generally obtains a new snapshot of committed data.

Example:

```sql
BEGIN;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT *
FROM vehicles
WHERE vehicle_id = 1001;

COMMIT;
```

A later statement in the same transaction may observe changes committed by other transactions after the earlier statement began.

### 13.2 REPEATABLE READ

This level provides a stable snapshot for the transaction.

Example:

```sql
BEGIN;

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT *
FROM vehicles
WHERE vehicle_id = 1001;

COMMIT;
```

Repeated reads within the transaction use the same transaction snapshot.

PostgreSQL's REPEATABLE READ implementation prevents phantom reads under its MVCC model, although serialization anomalies such as write skew can still occur.

### 13.3 SERIALIZABLE

SERIALIZABLE provides the strongest isolation level.

It ensures that successfully committed concurrent transactions produce an outcome equivalent to some serial execution order.

PostgreSQL uses Serializable Snapshot Isolation (SSI) to detect dangerous dependency patterns.

Some transactions may be aborted with a serialization failure and need to be retried by the application.

Example:

```sql
BEGIN;

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT *
FROM vehicles
WHERE vehicle_id = 1001;

COMMIT;
```

### 13.4 Isolation level comparison

| Level | PostgreSQL support | Main behavior |
|---|---|---|
| READ UNCOMMITTED | Treated as READ COMMITTED | No dirty reads |
| READ COMMITTED | Yes | New snapshot per statement |
| REPEATABLE READ | Yes | Stable transaction snapshot |
| SERIALIZABLE | Yes | Serializable execution guarantee |

### 13.5 Choosing an isolation level

A stronger isolation level can provide stronger guarantees, but it may increase contention or cause transactions to be aborted and retried.

The correct choice depends on:

- Business requirements.
- Read and write patterns.
- Concurrency.
- Transaction duration.
- Error-handling capabilities.
- Performance requirements.

The default READ COMMITTED level is suitable for many applications.

Do not select SERIALIZABLE simply because it sounds safer.

Use the weakest level that correctly satisfies the application's consistency requirements, and test the actual workload.

---

## 14. Write-Ahead Logging (WAL)

WAL is central to PostgreSQL transaction durability and recovery.

### 14.1 What is WAL?

Write-Ahead Logging is a mechanism that records database changes in a sequential log before modified data pages are required to be written to their final storage locations.

PostgreSQL stores WAL records in the database cluster's WAL directory.

The directory is commonly named:

```text
pg_wal
```

### 14.2 Why WAL exists

Imagine PostgreSQL modifies a page in memory.

The system crashes before that page is written to the table's data file.

Without an appropriate recovery mechanism, the committed change could be lost or the database could be left inconsistent.

WAL provides information needed to recover database changes.

### 14.3 Simplified operation

```text
SQL Transaction
      |
      v
Modify database page in memory
      |
      v
Generate WAL records
      |
      v
Write WAL records
      |
      v
Flush required WAL at commit
      |
      v
Acknowledge successful commit
      |
      v
Write dirty data pages later
```

### 14.4 WAL and data files

It is important to distinguish:

- WAL files: contain records describing changes.
- Data files: contain the actual database relation structures and pages.

They serve different purposes.

WAL is not a permanent replacement for table storage.

It is part of the mechanism used for durability and recovery.

### 14.5 Checkpoints

A checkpoint is a recovery-related operation that establishes a known point in the database's WAL history.

PostgreSQL ensures that relevant dirty pages are written to data files as part of checkpoint processing.

This helps limit how much WAL must be replayed during crash recovery.

A simplified relationship:

```text
Transaction changes
       |
       v
WAL records
       |
       v
Checkpoint
       |
       v
Data pages synchronized
       |
       v
Future crash recovery
```

Checkpoints are important for:

- Crash recovery.
- WAL management.
- I/O behavior.
- Database performance.

### 14.6 WAL and replication

WAL also plays an important role in PostgreSQL replication.

A standby server can receive and replay WAL records generated by a primary server.

This helps maintain a synchronized copy of the database.

We will examine this in:

- [[03 - Replication]]

### 14.7 Inspecting WAL configuration

You can inspect relevant settings:

```sql
SHOW wal_level;

SHOW synchronous_commit;

SHOW fsync;

SHOW checkpoint_timeout;

SHOW max_wal_size;
```

Some settings require elevated privileges to change.

For learning purposes, inspecting their current values is sufficient.

---

## 15. Transactions and Locks

Transactions and locks are closely related, but they are not the same concept.

A transaction defines a unit of work.

A lock coordinates access to shared database resources.

For example:

```sql
BEGIN;

UPDATE vehicles
SET operational_status = 'IN_MAINTENANCE'
WHERE vehicle_id = 1001;

COMMIT;
```

The update can acquire row-level locks while the transaction is active.

These locks help coordinate concurrent modifications.

### 15.1 Why locks matter

Without appropriate concurrency control, two transactions could modify the same data in incompatible ways.

Locks help PostgreSQL coordinate these operations.

### 15.2 Transaction duration

Long-running transactions can create operational problems.

They may:

- Hold locks for extended periods.
- Delay other transactions.
- Prevent cleanup of old row versions.
- Increase the risk of contention.
- Contribute to storage growth.

### 15.3 Detailed study

We will study:

- Shared locks.
- Exclusive locks.
- Row-level locks.
- Table-level locks.
- Deadlocks.
- Lock waits.
- Blocking transactions.
- MVCC interactions.

In:

- [[06 - Concurrency and Locks]]

---

## 16. Distributed Transactions and Two-Phase Commit

So far, we have focused on transactions within one PostgreSQL database.

However, enterprise systems sometimes need coordinated changes across multiple systems.

Examples include:

- Two different databases.
- A database and an external transaction manager.
- Multiple services participating in one business process.

A distributed transaction is a transaction that coordinates work across multiple independent transactional resources.

### 16.1 The coordination problem

Imagine a business process that needs to:

1. Update a central inventory database.
2. Update a financial database.
3. Register an operation in another system.

A failure between these operations could leave the systems inconsistent.

Coordinating these changes is more complex than managing a transaction inside one database.

### 16.2 Two-Phase Commit (2PC)

Two-Phase Commit is a protocol used to coordinate a distributed transaction across multiple participants.

It has two main phases.

**Phase 1: Prepare**

The coordinator asks each participant whether it can commit.

Each participant prepares the transaction and reports its status.

**Phase 2: Commit or Abort**

If all participants successfully prepare, the coordinator instructs them to commit.

If a participant cannot prepare, the coordinator can instruct them to abort.

### 16.3 Conceptual diagram

```text
Coordinator
    |
    v
PHASE 1: PREPARE
    |
    +------> Database A
    |             |
    |             v
    |          Prepared
    |
    +------> Database B
    |             |
    |             v
    |          Prepared
    |
    +------> Database C
                  |
                  v
               Prepared

                  |
                  v

PHASE 2: COMMIT
    |
    +------> Database A: COMMIT
    |
    +------> Database B: COMMIT
    |
    +------> Database C: COMMIT
```

This is a simplified representation of the protocol.

### 16.4 Advantages and limitations

| Advantages | Limitations |
|---|---|
| Coordinates multiple transactional participants | Requires coordination infrastructure |
| Helps maintain atomic outcomes across participants | Can introduce latency |
| Supports distributed transactional workflows | Prepared transactions may remain unresolved |
| Provides a structured commit protocol | Coordinator failures can complicate recovery |

### 16.5 PostgreSQL prepared transactions

PostgreSQL supports prepared transactions through commands such as:

```sql
PREPARE TRANSACTION 'transaction_001';
```

And later:

```sql
COMMIT PREPARED 'transaction_001';
```

Or:

```sql
ROLLBACK PREPARED 'transaction_001';
```

These commands are intended for coordinated transaction management.

They are not equivalent to ordinary SQL savepoints.

Prepared transactions can hold locks and other resources while awaiting a final decision.

They should not be enabled or used casually.

### 16.6 Distributed transactions vs. distributed application workflows

Not every application involving multiple services needs two-phase commit.

Many modern distributed systems use patterns such as:

- Saga pattern.
- Transactional outbox.
- Idempotent operations.
- Compensating transactions.
- Event-driven architecture.

These approaches coordinate business workflows without requiring one global database transaction.

They are important in microservices and distributed application architecture.

---

## 17. Practical Lab — PostgreSQL Transaction Management

The purpose of this laboratory is to practice transaction boundaries, commits, rollbacks, savepoints, and isolation.

Use a development database.

### 17.1 Create a laboratory schema

```sql
CREATE SCHEMA IF NOT EXISTS transaction_lab;
```

### 17.2 Create accounts

```sql
CREATE TABLE transaction_lab.accounts (
    account_id BIGINT PRIMARY KEY,
    account_name VARCHAR(100) NOT NULL,
    balance NUMERIC(12, 2) NOT NULL
        CHECK (balance >= 0)
);
```

### 17.3 Insert initial data

```sql
INSERT INTO transaction_lab.accounts (
    account_id,
    account_name,
    balance
)
VALUES
    (1, 'Account A', 5000.00),
    (2, 'Account B', 2000.00);
```

Inspect:

```sql
SELECT *
FROM transaction_lab.accounts
ORDER BY account_id;
```

### 17.4 Execute a successful transaction

```sql
BEGIN;

UPDATE transaction_lab.accounts
SET balance = balance - 1000
WHERE account_id = 1;

UPDATE transaction_lab.accounts
SET balance = balance + 1000
WHERE account_id = 2;

COMMIT;
```

Verify:

```sql
SELECT *
FROM transaction_lab.accounts
ORDER BY account_id;
```

Expected result:

| Account | Balance |
|---|---:|
| Account A | 4000.00 |
| Account B | 3000.00 |

### 17.5 Test rollback

```sql
BEGIN;

UPDATE transaction_lab.accounts
SET balance = balance - 500
WHERE account_id = 1;

UPDATE transaction_lab.accounts
SET balance = balance + 500
WHERE account_id = 2;

ROLLBACK;
```

Verify:

```sql
SELECT *
FROM transaction_lab.accounts
ORDER BY account_id;
```

The balances should remain unchanged from the previous committed state.

### 17.6 Test a constraint violation

```sql
BEGIN;

UPDATE transaction_lab.accounts
SET balance = -100
WHERE account_id = 1;
```

The CHECK constraint rejects the operation.

The transaction is now in an aborted state.

Recover:

```sql
ROLLBACK;
```

Verify the balances again.

### 17.7 Test a savepoint

```sql
BEGIN;

UPDATE transaction_lab.accounts
SET balance = balance - 100
WHERE account_id = 1;

SAVEPOINT after_debit;

UPDATE transaction_lab.accounts
SET balance = balance + 100
WHERE account_id = 2;

ROLLBACK TO SAVEPOINT after_debit;

COMMIT;
```

Only the first update is committed.

This is intentionally an incomplete business operation to demonstrate savepoint behavior.

Inspect the balances and then restore the initial state if necessary.

### 17.8 Test isolation configuration

```sql
SHOW transaction_isolation;
```

The default is normally:

```text
read committed
```

You can also inspect:

```sql
BEGIN;

SHOW transaction_isolation;

COMMIT;
```

To set an isolation level for a transaction:

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;

SELECT *
FROM transaction_lab.accounts;

COMMIT;
```

### 17.9 Inspect WAL-related settings

```sql
SHOW wal_level;

SHOW synchronous_commit;

SHOW fsync;

SHOW checkpoint_timeout;
```

Record the values and investigate their purposes.

Do not modify these settings as part of this introductory lab.

### 17.10 Clean up

```sql
DROP SCHEMA transaction_lab CASCADE;
```

Execute this only after completing the exercises.

---

## 18. Common Transaction Mistakes

| Mistake | Consequence |
|---|---|
| Forgetting COMMIT | Changes remain uncommitted until the transaction ends |
| Forgetting ROLLBACK after an error | The transaction remains aborted |
| Keeping transactions open too long | Locks and old row versions may remain |
| Grouping unrelated operations | Unnecessarily large transaction boundaries |
| Splitting one business operation into independent commits | Partial business outcomes become possible |
| Assuming ACID guarantees correct business decisions | Invalid business logic can still be committed |
| Disabling durability settings without analysis | Increased risk of losing acknowledged transactions |
| Ignoring transaction retries | Serialization failures may cause application errors |
| Confusing SAVEPOINT with COMMIT | A savepoint does not make changes permanent |
| Treating WAL as a backup | WAL alone is not a complete backup strategy |

---

## 19. Applying Transactions to SIAPV

Consider a workflow that registers a maintenance event for a vehicle.

The process may involve several related database operations.

### 19.1 Example business operation

An employee registers:

- A maintenance record.
- A new vehicle operational status.
- A meter reading.
- Supporting information about the operation.

A simplified transaction could be:

```sql
BEGIN;

INSERT INTO maintenance_records (
    maintenance_id,
    vehicle_id,
    maintenance_date,
    maintenance_type
)
VALUES (
    5001,
    1001,
    CURRENT_DATE,
    'Preventive'
);

UPDATE vehicles
SET operational_status = 'IN_MAINTENANCE'
WHERE vehicle_id = 1001;

INSERT INTO meter_readings (
    reading_id,
    vehicle_id,
    reading_date,
    reading_value
)
VALUES (
    8001,
    1001,
    CURRENT_DATE,
    1250.50
);

COMMIT;
```

This example assumes the referenced tables and columns exist.

### 19.2 Atomicity

If one operation fails, the transaction can be rolled back so that the application does not preserve only part of the workflow.

### 19.3 Consistency

Foreign keys, constraints, and business rules protect the validity of the maintenance and reading records.

### 19.4 Isolation

Other users should not observe an incomplete operation as if it were fully registered.

### 19.5 Durability

After a successful commit, the operation should survive failures covered by PostgreSQL's durability guarantees.

### 19.6 Transaction design questions

For a real SIAPV workflow, ask:

1. Which records must be created together?
2. Which updates depend on the successful creation of another record?
3. Which integrity rules must be enforced?
4. What should happen if one operation fails?
5. How long should the transaction remain open?
6. Could another employee modify the same vehicle simultaneously?
7. Does the workflow require a particular isolation level?
8. How should the application report failures?
9. Are retries safe?
10. Which operations can be performed independently?

These questions transform a sequence of SQL commands into a properly designed business transaction.

---

## 20. Key Architectural Principles

1. **A transaction is a logical unit of work.** It groups related operations under one completion decision.

2. **Atomicity means all or nothing.** Related operations should be committed together or rolled back together.

3. **Consistency preserves defined integrity rules.** Constraints and business logic determine which database states are valid.

4. **Isolation controls concurrent interactions.** Transactions must not interfere in ways that violate required guarantees.

5. **Durability protects committed changes.** WAL and recovery mechanisms help preserve transaction results.

6. **ACID properties complement each other.** Reliable transaction processing depends on all four.

7. **Autocommit does not eliminate transactions.** It automatically completes individual statements when appropriate.

8. **Savepoints support partial rollback.** They do not commit transactions.

9. **Isolation levels involve trade-offs.** Stronger guarantees may introduce additional contention or retries.

10. **WAL is central to recovery.** It is not a replacement for backups.

11. **Long-running transactions create operational risks.** They can retain locks and prevent cleanup of obsolete tuple versions.

12. **Application code must handle transaction failures.** Failed transactions often require rollback or retry.

13. **Distributed transactions are more complex than local transactions.** They require coordination among independent participants.

14. **Transaction boundaries should reflect business operations.** The design must preserve the intended meaning of each workflow.

---

## 21. Knowledge Check

Answer these questions without looking at the explanations above.

1. What is a database transaction?
2. What is the difference between a SQL statement and a transaction?
3. What does ACID stand for?
4. What does atomicity guarantee?
5. How does consistency relate to database constraints?
6. Why does consistency not automatically guarantee correct business decisions?
7. What is transaction isolation?
8. What is a dirty read?
9. What is a non-repeatable read?
10. What is a phantom read?
11. What is a lost update?
12. How does MVCC support concurrent access?
13. What does durability guarantee?
14. What is Write-Ahead Logging?
15. Why does PostgreSQL use WAL?
16. What is the relationship between WAL and data pages?
17. What is a checkpoint?
18. What is the difference between COMMIT and ROLLBACK?
19. What is autocommit?
20. What is a SAVEPOINT?
21. What happens when a statement fails inside an explicit PostgreSQL transaction?
22. Which isolation level is PostgreSQL's default?
23. What is the difference between REPEATABLE READ and SERIALIZABLE?
24. What is Two-Phase Commit?
25. Why are long-running transactions potentially problematic?

---

## 22. Practical Exercises

### Exercise 1 — Design an atomic operation

Design a transaction for a maintenance workflow.

It must:

- Insert a maintenance record.
- Update the vehicle status.
- Insert a meter reading.

Explain what should happen if any operation fails.

### Exercise 2 — Test rollback

Create a transaction with at least three operations.

Execute the first two successfully and deliberately make the third fail.

Roll back the transaction.

Verify that the first two operations were not committed.

### Exercise 3 — Use savepoints

Create a transaction with three operations.

Establish a savepoint after the first operation.

Roll back the second and third operations.

Commit the first operation.

Explain the difference between rolling back to a savepoint and rolling back the entire transaction.

### Exercise 4 — Compare isolation levels

Use two database sessions.

Test READ COMMITTED and REPEATABLE READ.

Observe how the same transaction can see different data depending on the isolation level.

Document the steps and results.

### Exercise 5 — Analyze a transaction failure

Consider this scenario:

An application registers a maintenance event, updates the vehicle status, and then loses its connection before receiving confirmation.

Explain:

1. What might have happened to the transaction?
2. Why is the application uncertain about the final result?
3. Why should it not blindly repeat the operation?
4. How could an idempotency key help?
5. How could the application verify the final state?

### Exercise 6 — SIAPV transaction design

Choose a real operational workflow from SIAPV.

Document:

- Business objective.
- Participating tables.
- SQL operations.
- Transaction boundary.
- Integrity constraints.
- Expected rollback behavior.
- Possible concurrency conflicts.
- Recovery and error-handling strategy.

---

## 23. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Database transaction | Transacción de base de datos |
| Transaction management | Gestión de transacciones |
| Atomicity | Atomicidad |
| Consistency | Consistencia |
| Isolation | Aislamiento |
| Durability | Durabilidad |
| Commit | Confirmar una transacción |
| Rollback | Revertir una transacción |
| Savepoint | Punto de restauración dentro de una transacción |
| Transaction boundary | Límite de transacción |
| Autocommit | Confirmación automática |
| Concurrent transaction | Transacción concurrente |
| Transaction isolation level | Nivel de aislamiento |
| Dirty read | Lectura sucia |
| Non-repeatable read | Lectura no repetible |
| Phantom read | Lectura fantasma |
| Lost update | Actualización perdida |
| Write skew | Anomalía de escritura |
| Write-Ahead Logging | Registro anticipado de escritura |
| WAL record | Registro WAL |
| Checkpoint | Punto de control |
| Crash recovery | Recuperación tras una caída |
| Durability guarantee | Garantía de durabilidad |
| Prepared transaction | Transacción preparada |
| Distributed transaction | Transacción distribuida |
| Two-Phase Commit | Confirmación en dos fases |
| Serialization failure | Fallo de serialización |
| Transaction retry | Reintento de transacción |
| Idempotency | Idempotencia |
| Lock contention | Contención de bloqueos |

---

## 24. Related Notes

### Previous concepts

- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[03 - Storage Pages and Tables]]
- [[04 - Indexes]]
- [[03 - Keys and Constraints]]
- [[05 - Normalization]]

### Next concepts

- [[06 - Concurrency and Locks]]
- [[01 - Client Server Architecture]]
- [[06 - High Availability]]
- [[07 - Backup and Disaster Recovery]]
- [[02 - PostgreSQL Processes]]
- [[05 - PostgreSQL Performance Fundamentals]]

---

## 25. Final Understanding

Transactions are the foundation of reliable database operations.

They ensure that related changes can be executed as one logical unit while preserving integrity, coordinating concurrent access, and protecting committed information.

The ACID properties describe the fundamental guarantees:

- Atomicity: all or nothing.
- Consistency: valid database states.
- Isolation: controlled concurrent behavior.
- Durability: committed changes are preserved.

PostgreSQL implements these guarantees through transaction management, MVCC, locking, WAL, and recovery mechanisms.

For a database architect, understanding transactions is essential because database reliability is not simply about storing information.

It is about ensuring that every operation preserves the intended meaning of the data, even when failures and concurrent activity occur.

**The next step is to study the mechanisms that coordinate simultaneous transactions.**

That leads directly to concurrency control and locking.
