
---
title: "Database Backup and Disaster Recovery"
aliases:
  - "Database Backup"
  - "Database Recovery"
  - "Disaster Recovery Architecture"
  - "Point-in-Time Recovery"
type: learning-note
domain: technology
category: database-architecture
subject: backup-and-disaster-recovery
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - database-architecture
  - backup
  - recovery
  - disaster-recovery
  - data-protection
  - postgresql
  - point-in-time-recovery
  - wal
  - rto
  - rpo
  - business-continuity
concepts:
  - backup
  - restore
  - recovery
  - disaster-recovery
  - full-backup
  - incremental-backup
  - differential-backup
  - logical-backup
  - physical-backup
  - wal-archiving
  - point-in-time-recovery
  - recovery-target
  - retention-policy
  - backup-validation
  - offsite-backup
  - immutable-backup
  - rto
  - rpo
  - business-continuity
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[05 - Transactions and ACID]]"
  - "[[03 - Storage Pages and Tables]]"
  - "[[04 - Indexes]]"
  - "[[03 - Replication]]"
  - "[[06 - High Availability]]"
related:
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[03 - PostgreSQL Storage]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"
  - "[[01 - Database Security Fundamentals]]"
  - "[[03 - Data Quality]]"
tools:
  - PostgreSQL
  - pg_dump
  - pg_restore
  - pg_basebackup
  - pg_wal
  - psql
  - pgAdmin
  - Docker
  - Linux
learning_objectives:
  - "Explain the purpose of database backups and disaster recovery."
  - "Distinguish backup, replication, high availability, and disaster recovery."
  - "Compare logical and physical backups."
  - "Understand full, incremental, and differential backup strategies."
  - "Explain PostgreSQL WAL archiving and point-in-time recovery."
  - "Interpret RTO and RPO in a recovery strategy."
  - "Design backup retention and storage policies."
  - "Understand off-site, immutable, and encrypted backups."
  - "Validate backups through restoration tests."
  - "Design a practical recovery strategy for an application."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Database Backup and Disaster Recovery

## 1. Introduction

A database may contain years of operational information:

- Customers and suppliers.
- Financial transactions.
- Inventory records.
- Maintenance history.
- User accounts and permissions.
- Business reports.
- Operational and administrative records.

Protecting this information requires more than keeping the database server running.

Consider the following situation:

An administrator executes an incorrect SQL statement:

```sql
DELETE FROM equipment;
```

The statement removes every record from the `equipment` table.

The database server is still running. PostgreSQL is responding normally. The application is accessible.

However, important information has disappeared.

High availability alone does not solve this problem. If the database uses replication, the deletion may also be replicated to the standby.

The organization needs a way to recover a previous valid version of its data.

This is the purpose of database backup and recovery.

**A backup is a recoverable copy of data. Recovery is the process of using that copy, together with other recovery information when necessary, to restore a database to a usable state.**

## 2. Backup, Recovery, and Disaster Recovery

These concepts are closely related but have different purposes.

### 2.1 Backup

A backup is a copy of database information stored independently from the active database.

It can be used to recover from:

- Accidental deletion.
- Incorrect updates.
- Database corruption.
- Hardware failure.
- Software errors.
- Security incidents.
- Loss of the original database files.

### 2.2 Recovery

Recovery is the process of restoring a database or its information to a valid state.

It may involve:

- Restoring a backup.
- Replaying transaction logs.
- Recovering individual records.
- Rebuilding database files.
- Reconfiguring a database server.
- Validating the restored information.

### 2.3 Disaster Recovery (DR)

Disaster Recovery is the broader strategy for restoring IT services after a major disruptive event.

Examples include:

- Complete data center failure.
- Regional infrastructure outage.
- Destruction of storage systems.
- Major ransomware attack.
- Loss of an entire production environment.
- Severe operational or security incidents.

DR considers more than database files. It may include applications, networks, infrastructure, credentials, configurations, and operational procedures.

### 2.4 Comparison

| Concept | Main purpose |
|---|---|
| Backup | Preserve recoverable copies of information. |
| Restore | Copy backup data into a target environment. |
| Recovery | Bring the database back to a valid operational state. |
| High Availability | Maintain service during component failures. |
| Disaster Recovery | Restore services after major disruptions. |
| Business Continuity | Maintain essential business operations during and after disruptions. |

These capabilities complement each other.

A mature architecture needs to consider all of them.

## 3. Why Replication Is Not a Backup

This distinction is essential.

Replication maintains additional copies of database changes.

A backup preserves a recoverable state from an earlier point in time.

Imagine a primary database and a standby database.

An administrator accidentally executes:

```sql
DELETE FROM maintenance_records;
```

If replication is operating normally, the deletion may be transmitted to the standby.

Both database instances can now contain the same incorrect state.

Replication has successfully copied the changes, but the organization still needs to recover the deleted records.

### 3.1 Replication versus backup

| Characteristic | Replication | Backup |
|---|---|---|
| Main objective | Maintain synchronized copies | Preserve recoverable data |
| Typical operation | Continuous | Scheduled or continuous log archiving |
| Accidental deletion | May replicate the deletion | Can preserve an earlier state |
| Historical recovery | Not its primary purpose | Primary purpose |
| Failover support | Yes, when combined with HA mechanisms | Not generally immediate |
| Recovery after corruption | Depends on the corruption | Can provide an earlier valid copy |

**Important principle:**

A replica is not an independent historical backup.

A database can have multiple replicas and still have no reliable recovery strategy.

## 4. Backup Classification

Backups can be classified according to the amount of data copied and the method used to create the copy.

Two important classifications are:

1. Backup strategy: full, incremental, or differential.
2. Backup format: logical or physical.

These classifications describe different dimensions of backup design.

## 5. Full Backup

A full backup copies all the selected data required by the backup operation.

For a database, this may include the entire database or an entire database cluster, depending on the backup method.

Example:

A database contains 100 GB of data.

A full backup copies the complete 100 GB dataset, subject to the backup tool and compression settings.

### 5.1 Advantages

- Simple recovery model.
- Provides a complete baseline.
- Easier to manage independently.
- Useful for establishing recovery points.

### 5.2 Disadvantages

- Requires more storage.
- May take longer to complete.
- Can generate significant I/O.
- Repeated full backups can consume substantial resources.

### 5.3 Example

```text
Monday:    Full backup
Tuesday:   Full backup
Wednesday: Full backup
Thursday:  Full backup
```

Each backup contains a complete copy of the selected data.

This is straightforward but can be expensive for large databases.

## 6. Incremental Backup

An incremental backup stores only the changes made since the previous backup.

Example:

```text
Monday:    Full backup       100 GB
Tuesday:   Incremental         5 GB
Wednesday: Incremental         3 GB
Thursday:  Incremental         7 GB
```

To recover Thursday's state, the recovery process may need:

1. Monday's full backup.
2. Tuesday's incremental backup.
3. Wednesday's incremental backup.
4. Thursday's incremental backup.

The exact process depends on the backup technology.

### 6.1 Advantages

- Reduces backup storage consumption.
- Can reduce backup duration.
- Suitable for large datasets with relatively small daily changes.

### 6.2 Disadvantages

- Recovery may require multiple backup sets.
- Missing or corrupted incremental files can complicate restoration.
- Backup management becomes more complex.

## 7. Differential Backup

A differential backup stores changes made since the most recent full backup.

Example:

```text
Monday:    Full backup       100 GB
Tuesday:   Differential        5 GB
Wednesday: Differential        8 GB
Thursday:  Differential       15 GB
```

To recover Thursday's state, the process generally requires:

1. Monday's full backup.
2. Thursday's differential backup.

The differential backup contains the accumulated changes since Monday.

### 7.1 Comparison

| Strategy | Copies | Recovery requirements |
|---|---|---|
| Full | All selected data | Latest valid full backup |
| Incremental | Changes since previous backup | Full backup plus required incremental chain |
| Differential | Changes since latest full backup | Full backup plus latest differential |

### 7.2 Visual comparison

```text
FULL BACKUP
    |
    +---- Incremental 1
    |          |
    |          +---- Incremental 2
    |                     |
    |                     +---- Incremental 3
    |
    +---- Differential 1
    |
    +---- Differential 2
    |
    +---- Differential 3
```

The choice depends on:

- Database size.
- Change rate.
- Storage capacity.
- Backup window.
- Recovery time requirements.
- Backup software capabilities.

**PostgreSQL note:** PostgreSQL's built-in backup utilities provide logical dumps and physical backup capabilities. Its standard physical backup and WAL recovery mechanisms should not be confused with every generic incremental or differential backup scheme. Incremental physical backup capabilities depend on PostgreSQL version and the specific tooling used.

## 8. Logical Backups

A logical backup exports database objects and their data in a logical representation.

It can include:

- Database schemas.
- Tables.
- Views.
- Sequences.
- Functions.
- Stored procedures.
- Data.
- Other supported database objects.

Logical backups are useful when migrating data, moving selected objects, or restoring specific database components.

### 8.1 PostgreSQL pg_dump

PostgreSQL provides `pg_dump` to create logical backups of individual databases.

Example:

```bash
pg_dump -U postgres -d siapv -F c -f siapv_backup.dump
```

Explanation:

| Argument | Meaning |
|---|---|
| `pg_dump` | PostgreSQL logical backup utility. |
| `-U postgres` | Database user. |
| `-d siapv` | Database to export. |
| `-F c` | Custom archive format. |
| `-f` | Output file. |

The resulting file contains a logical representation of the selected database.

### 8.2 Backup in plain SQL format

```bash
pg_dump -U postgres -d siapv -F p -f siapv_backup.sql
```

This generates a plain SQL script.

The file can be inspected with a text editor and executed through PostgreSQL tools.

### 8.3 Restoring a custom-format backup

Custom-format backups are commonly restored with `pg_restore`.

```bash
createdb -U postgres siapv_recovered
```

Then:

```bash
pg_restore -U postgres -d siapv_recovered siapv_backup.dump
```

The target database must exist before this example is executed.

### 8.4 Restoring a plain SQL backup

A plain SQL dump can be restored using `psql`.

```bash
createdb -U postgres siapv_recovered
```

Then:

```bash
psql -U postgres -d siapv_recovered -f siapv_backup.sql
```

### 8.5 Backing up roles and global objects

`pg_dump` exports one database. It does not automatically include every cluster-wide object.

PostgreSQL provides `pg_dumpall` for exporting global objects such as roles and tablespaces, in addition to its ability to produce SQL dumps of databases.

Example:

```bash
pg_dumpall -U postgres --globals-only -f globals.sql
```

This is important because a database restore may depend on roles and permissions that exist outside the individual database.

### 8.6 Advantages of logical backups

- Portable between compatible PostgreSQL environments.
- Useful for selective restoration.
- Suitable for database migrations.
- Can be inspected in plain SQL format.
- Can restore selected schemas or tables with appropriate options.

### 8.7 Limitations

- Restoration can be slower for large databases.
- Does not automatically preserve the complete physical cluster state.
- Requires consideration of roles and global objects.
- Does not independently provide point-in-time recovery.

## 9. Physical Backups

A physical backup copies the database's physical files and related information needed to reconstruct the database cluster.

It may include:

- Database files.
- Relation files.
- System catalogs.
- Transaction-related storage.
- Required configuration or tablespace information, depending on the procedure.

Physical backups are especially useful for large databases and full-cluster recovery.

### 9.1 PostgreSQL pg_basebackup

PostgreSQL provides `pg_basebackup` to create a base backup of a running database cluster.

Example:

```bash
pg_basebackup \
  -h localhost \
  -p 5432 \
  -U replication_user \
  -D /backup/postgres/base \
  -Fp \
  -Xs \
  -P
```

Explanation:

| Argument | Meaning |
|---|---|
| `-h` | Database server host. |
| `-p` | Database port. |
| `-U` | Replication user. |
| `-D` | Destination directory. |
| `-Fp` | Plain directory output format. |
| `-Xs` | Stream required WAL during backup. |
| `-P` | Display progress. |

The user must have the necessary replication permissions and the server must be configured to permit the connection.

### 9.2 Logical versus physical backups

| Characteristic | Logical backup | Physical backup |
|---|---|---|
| Representation | SQL or logical archive | Database files |
| Typical PostgreSQL tool | `pg_dump` | `pg_basebackup` |
| Scope | Individual database or selected objects | Database cluster |
| Selective restore | More flexible | Usually cluster-oriented |
| Large database restoration | May be slower | Often more efficient |
| Portability | Generally more flexible | More dependent on compatible PostgreSQL environments |
| Point-in-time recovery | Requires additional mechanisms | Can be combined with WAL archiving |

Neither method replaces the other in every situation.

Organizations may use both to cover different recovery scenarios.

## 10. PostgreSQL Write-Ahead Logging (WAL)

To understand advanced PostgreSQL recovery, it is necessary to understand Write-Ahead Logging.

**Write-Ahead Logging (WAL)** records changes that can be used to recover database consistency after failures.

The fundamental principle is:

**Changes must be recorded in the transaction log before the corresponding modified data pages are considered safely written to their final database files.**

This allows PostgreSQL to reconstruct database state after a crash.

### 10.1 Simplified WAL flow

```text
Application
     |
     v
SQL Transaction
     |
     v
PostgreSQL Processing
     |
     v
WAL Records Generated
     |
     v
WAL Written and Flushed
     |
     v
Transaction Commit
     |
     v
Data Pages Written Later
```

The exact timing of physical data-page writes depends on PostgreSQL's storage and checkpoint mechanisms.

### 10.2 Why WAL matters for recovery

Imagine:

1. A transaction modifies an equipment record.
2. PostgreSQL writes the necessary WAL information.
3. The transaction commits.
4. The server crashes before all modified data pages are written.

When PostgreSQL restarts, it can use WAL to recover the database to a consistent state.

WAL is therefore essential to crash recovery.

### 10.3 WAL and backup

WAL can also be archived and replayed after restoring a physical base backup.

This makes it possible to recover the database to a particular point in time.

That capability is called Point-in-Time Recovery.

## 11. WAL Archiving

WAL archiving stores completed WAL segments in an external archive location.

Instead of relying only on the WAL files currently available in the database's local storage, the organization preserves copies for later recovery.

A simplified architecture:

```text
              PostgreSQL
                  |
                  v
              WAL Files
                  |
                  v
             WAL Archive
                  |
                  v
          External Storage
                  |
                  v
        Recovery when required
```

### 11.1 Why archive WAL files?

A base backup provides a starting point.

Archived WAL records provide the changes needed to advance the restored database from that starting point to a later recovery target.

For example:

- Base backup created at 01:00.
- WAL records archived throughout the day.
- Accidental deletion occurs at 15:30.
- Recovery can potentially reconstruct the database to 15:29, provided the required backup and WAL files are available and valid.

### 11.2 PostgreSQL configuration

A simplified configuration might include:

```conf
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'
```

Explanation:

| Setting | Purpose |
|---|---|
| `wal_level` | Determines the amount of information written to WAL. |
| `archive_mode` | Enables WAL archiving. |
| `archive_command` | Defines how completed WAL segments are copied to the archive. |

This is an illustrative local-filesystem example, not a complete production configuration.

Production environments must consider:

- Archive permissions.
- Reliable storage.
- Monitoring of failed archive commands.
- Storage capacity.
- Retention and cleanup.
- Secure transfer.
- Recovery testing.

The archive destination must be independent enough to survive failures affecting the database server.

## 12. Point-in-Time Recovery (PITR)

Point-in-Time Recovery allows a PostgreSQL database to be restored to a specific point in time by combining a physical base backup with the necessary WAL records.

This is particularly useful when an error occurs after a known valid recovery point.

### 12.1 Example scenario

A maintenance database contains thousands of historical records.

At 14:35, an administrator accidentally executes:

```sql
DELETE FROM maintenance_records;
```

The organization has:

- A valid physical base backup from the previous night.
- Archived WAL files covering the period since that backup.

The recovery objective is to restore the database to 14:34:59.

### 12.2 Recovery process

A simplified PITR workflow:

1. Identify a valid base backup.
2. Prepare a separate recovery environment.
3. Restore the base backup.
4. Configure PostgreSQL to use the required WAL archive.
5. Specify the recovery target.
6. Start PostgreSQL in recovery mode.
7. Allow WAL records to be replayed.
8. Stop recovery at the requested point.
9. Validate the recovered database.
10. Promote or finalize the recovered instance according to the recovery procedure.

### 12.3 Recovery target

PostgreSQL supports several recovery targets, including:

- Timestamp.
- Transaction ID.
- Named restore point.
- Log sequence number (LSN).

For example, a recovery configuration may include:

```conf
restore_command = 'cp /archive/%f %p'

recovery_target_time = '2026-11-12 14:34:59'

recovery_target_action = 'promote'
```

This is an illustrative configuration. A complete recovery setup also requires a valid base backup, the correct recovery signal and configuration for the PostgreSQL version, and all required WAL segments.

### 12.4 Recovery timeline

```text
01:00             14:34:59       14:35
  |                   |             |
  v                   v             v
Base Backup       Recovery Point   Error
  |-------------------|-------------|
           WAL Replay
```

The objective is to reconstruct the database at the last known valid point before the unwanted operation.

### 12.5 Why PITR is powerful

Without PITR, the organization may only be able to restore the most recent scheduled backup.

With PITR, it can recover to a more precise point in time, provided the necessary backup and WAL chain is complete.

This significantly improves recovery flexibility.

## 13. Backup Retention Policies

A backup retention policy defines how long backups are preserved and when they can be deleted.

Without a retention policy, storage can grow indefinitely.

A retention policy must balance:

- Recovery requirements.
- Storage costs.
- Legal obligations.
- Security requirements.
- Operational complexity.
- Historical recovery needs.

### 13.1 Example retention policy

| Backup type | Frequency | Retention |
|---|---|---|
| Full backup | Weekly | 8 weeks |
| Incremental backup | Daily | 14 days |
| WAL archive | Continuous | 30 days |
| Monthly archive | Monthly | 12 months |

This is only an example. Actual retention must be determined by business and regulatory requirements.

### 13.2 Backup rotation

Backup rotation is the practice of managing multiple generations of backups.

A common strategy is the Grandfather-Father-Son (GFS) approach.

- Son: Daily backups.
- Father: Weekly backups.
- Grandfather: Monthly backups.

The strategy preserves recovery points at different time scales.

### 13.3 Why retention matters

Imagine a database becomes corrupted today.

The corruption may have started three weeks ago.

If the organization only keeps backups from the previous 48 hours, all available backups may already contain the corruption.

A longer retention period may provide a valid recovery point.

However, retaining more backups does not automatically guarantee recoverability. Backup integrity and restoration testing remain necessary.

## 14. Backup Storage Strategies

A backup should not depend entirely on the infrastructure it is intended to protect.

If the database server and its backup files are stored on the same physical disk, a disk failure may destroy both.

### 14.1 Local backups

Stored on the same server or local infrastructure.

Advantages:

- Fast access.
- Simple configuration.
- Low network overhead.

Disadvantages:

- Vulnerable to local hardware failure.
- May share the production failure domain.
- Limited protection against site-wide disasters.

### 14.2 Network backups

Stored on a separate network storage system.

Advantages:

- Separates backup storage from the database host.
- Can provide larger storage capacity.
- Supports centralized backup management.

Disadvantages:

- Depends on network availability.
- May still share a data center or administrative failure domain.

### 14.3 Off-site backups

Stored in a different physical location or geographic region.

Advantages:

- Better protection against local disasters.
- Supports geographic recovery strategies.
- Reduces dependence on the original site.

Disadvantages:

- Network transfer costs.
- Higher recovery latency.
- Additional storage and security considerations.

### 14.4 Cloud object storage

Backup files can be stored in object storage services.

Examples include:

- Amazon S3.
- Azure Blob Storage.
- Google Cloud Storage.
- Compatible object-storage platforms.

Object storage can provide scalable storage, lifecycle management, versioning, and retention capabilities, depending on the service.

### 14.5 Immutable backups

An immutable backup is protected against modification or deletion for a defined period.

This can help defend against:

- Ransomware.
- Malicious administrators.
- Accidental deletion.
- Compromised production credentials.

Immutability is especially important when attackers may have access to production systems.

However, immutability must be combined with access controls, encryption, monitoring, and tested recovery procedures.

## 15. The 3-2-1 Backup Rule

The 3-2-1 rule is a widely used backup principle.

It recommends:

- 3 copies of important data.
- 2 different types of storage media.
- 1 copy stored off-site.

Example:

```text
Original Database
       |
       +------ Local Backup
       |
       +------ Network Backup
       |
       +------ Off-site Backup
```

The objective is to avoid depending on a single storage system or physical location.

Some organizations extend the model with additional requirements:

- At least one immutable copy.
- Regular recovery testing.
- Separate administrative credentials.
- Offline or isolated backup copies.

The specific strategy should reflect the threat model and recovery objectives.

## 16. Backup Security

Backups can contain the same sensitive information as production databases.

Therefore, they require strong security controls.

### 16.1 Encryption

Encryption protects backup data from unauthorized access.

Two important stages are:

**Encryption at rest**

Protects stored backup files.

**Encryption in transit**

Protects backups while being transferred across networks.

### 16.2 Access control

Only authorized users and systems should be able to:

- Create backups.
- Read backup files.
- Delete backups.
- Restore backups.
- Modify retention policies.

### 16.3 Credential separation

Backup credentials should not automatically provide unrestricted access to production infrastructure.

Separating administrative privileges can reduce the impact of compromised accounts.

### 16.4 Audit logging

Backup operations should be recorded.

Important events include:

- Backup creation.
- Backup failure.
- Backup deletion.
- Restore operations.
- Retention-policy changes.
- Access to backup repositories.

Backup security is part of the broader subject of:

[[01 - Database Security Fundamentals]]

## 17. Backup Validation and Restoration Testing

A successful backup operation does not necessarily mean that the backup can be restored.

A file may exist but be:

- Incomplete.
- Corrupted.
- Missing required dependencies.
- Incompatible with the intended recovery environment.
- Missing necessary WAL segments.

Therefore, backups must be tested.

### 17.1 Backup validation

Validation can include:

- Checking file existence.
- Verifying file size.
- Checking checksums.
- Inspecting backup metadata.
- Reviewing backup logs.
- Confirming WAL archive continuity.

### 17.2 Restoration testing

A restoration test verifies that the backup can actually reconstruct a usable database.

A typical procedure:

1. Select a backup.
2. Create an isolated recovery environment.
3. Restore the database.
4. Start the database server.
5. Verify database objects.
6. Execute representative queries.
7. Validate record counts and relationships.
8. Confirm permissions and roles.
9. Measure restoration time.
10. Document the result.

### 17.3 Why restoration time matters

An organization may have an RTO of 30 minutes.

However, its most recent restoration test takes three hours.

The architecture does not currently demonstrate that the RTO can be achieved.

Recovery performance must be measured, not assumed.

## 18. Recovery Procedures and Runbooks

A recovery runbook is a documented sequence of steps for responding to a specific incident.

It helps administrators act consistently under pressure.

A database recovery runbook may include:

1. Incident identification.
2. Impact assessment.
3. Authorization to begin recovery.
4. Selection of recovery strategy.
5. Identification of backup and WAL files.
6. Preparation of recovery infrastructure.
7. Restoration.
8. Database validation.
9. Application reconnection.
10. Communication with stakeholders.
11. Documentation of the incident.

A runbook should also define responsibilities.

For example:

| Role | Responsibility |
|---|---|
| Database administrator | Restore and validate the database. |
| Infrastructure team | Prepare servers, storage, and network access. |
| Application team | Reconnect and validate application behavior. |
| Security team | Investigate potential compromise. |
| Business owner | Approve recovery objectives and operational decisions. |

A technically correct recovery is not sufficient if the organization cannot coordinate the process.

## 19. Disaster Recovery Architecture

A disaster recovery architecture defines how critical systems will be restored after a major incident.

Several approaches are common.

### 19.1 Backup and restore

The organization maintains backups and restores the system when necessary.

Characteristics:

- Lower infrastructure cost.
- Longer recovery time.
- Requires restoration capacity.
- Suitable for systems with less demanding RTO requirements.

### 19.2 Pilot light

A minimal version of the infrastructure remains available in another location.

Critical components are maintained, but the complete production environment is not continuously active.

When a disaster occurs, additional infrastructure is started and the system is restored.

Characteristics:

- Moderate cost.
- Faster recovery than backup-only strategies.
- Requires deployment and recovery automation.

### 19.3 Warm standby

A partially or fully configured recovery environment remains running and receives data from the primary environment.

During a disaster, the standby environment is activated or promoted.

Characteristics:

- Faster recovery.
- Additional infrastructure cost.
- Requires replication and operational coordination.

### 19.4 Hot standby

A fully operational secondary environment remains ready to take over.

Depending on the architecture, it may continuously receive replicated data and be prepared for rapid promotion.

Characteristics:

- Potentially very low RTO.
- Higher infrastructure cost.
- More demanding operational requirements.

### 19.5 Comparison

| Strategy | Recovery speed | Infrastructure cost |
|---|---|---|
| Backup and restore | Usually slower | Lower |
| Pilot light | Moderate | Low to moderate |
| Warm standby | Faster | Moderate to high |
| Hot standby | Usually fastest | High |

These are general tendencies, not guaranteed recovery times.

Actual RTO depends on the complete architecture and tested procedures.

## 20. Disaster Recovery and Geographic Resilience

A local backup may protect against database corruption but fail to protect against a complete site disaster.

For this reason, important recovery copies should be stored outside the primary failure domain.

A geographically distributed architecture may include:

```text
                  Production Region
                         |
                         v
                   Primary Database
                         |
                         v
                   WAL Archiving
                         |
                         v
                 Remote Backup Storage
                         |
                         v
                   Recovery Region
                         |
                         v
                 Restored Database
                         |
                         v
                  Application Recovery
```

The recovery region must have enough infrastructure to operate the restored service.

This includes:

- Database servers.
- Application servers.
- Network connectivity.
- DNS or routing.
- Security credentials.
- Monitoring.
- Required external dependencies.

A backup stored in another region is not enough if the organization has no practical way to start the application there.

## 21. RTO and RPO in Backup Architecture

Backup strategies must be selected according to recovery objectives.

### 21.1 Example

An organization defines:

- RTO: 2 hours.
- RPO: 15 minutes.

This means:

- The service must be restored within two hours.
- The organization can tolerate no more than approximately 15 minutes of data loss.

A daily full backup alone may not satisfy the RPO.

If the database fails 20 hours after the last backup, the recovery point may be too old.

The organization may need:

- More frequent backups.
- Continuous WAL archiving.
- A standby replica.
- A combination of these mechanisms.

### 21.2 Recovery objectives influence architecture

| Requirement | Possible architectural response |
|---|---|
| Low RTO | Pre-provisioned recovery infrastructure, automation, standby systems |
| Low RPO | Frequent backups, continuous WAL archiving, synchronous replication where appropriate |
| Long historical retention | Backup rotation and archival storage |
| Regional disaster protection | Off-site or cross-region copies |
| Ransomware protection | Immutable backups and separated credentials |
| Fast restoration of large databases | Physical backups and optimized recovery infrastructure |

The best solution is not necessarily the most expensive one.

It is the solution that meets the required objectives and can be operated reliably.

## 22. Practical PostgreSQL Backup Lab

This exercise introduces logical backup and restoration using PostgreSQL.

Perform it only in a development environment.

### 22.1 Create a test database

```sql
CREATE DATABASE backup_lab;
```

Connect to it:

```bash
psql -U postgres -d backup_lab
```

### 22.2 Create a test table

```sql
CREATE TABLE equipment (
    equipment_id INTEGER PRIMARY KEY,
    equipment_name VARCHAR(100) NOT NULL,
    equipment_status VARCHAR(30) NOT NULL,
    operating_hours NUMERIC(10, 2)
);
```

### 22.3 Insert sample records

```sql
INSERT INTO equipment (
    equipment_id,
    equipment_name,
    equipment_status,
    operating_hours
)
VALUES
    (1, 'Vibratory Roller 01', 'Available', 1250.50),
    (2, 'Motor Grader 01', 'Under Maintenance', 2300.75),
    (3, 'Backhoe Loader 01', 'Available', 980.25);
```

Verify:

```sql
SELECT * FROM equipment;
```

### 22.4 Create a logical backup

Exit `psql` and execute:

```bash
pg_dump -U postgres -d backup_lab -F c -f backup_lab.dump
```

### 22.5 Create a recovery database

```bash
createdb -U postgres backup_lab_recovered
```

### 22.6 Restore the backup

```bash
pg_restore -U postgres -d backup_lab_recovered backup_lab.dump
```

### 22.7 Verify restored information

Connect to the recovered database:

```bash
psql -U postgres -d backup_lab_recovered
```

Execute:

```sql
SELECT * FROM equipment;
```

Compare the restored records with the original database.

### 22.8 Test data independence

In the original database, execute:

```sql
DELETE FROM equipment
WHERE equipment_id = 1;
```

Then compare:

- Original database.
- Recovered database.

The original database no longer contains the deleted record, but the restored database should still contain the version preserved in the backup.

This demonstrates why a backup provides historical recovery capabilities that replication alone cannot guarantee.

## 23. Designing a Recovery Strategy for SIAPV

Consider a future institutional version of SIAPV.

The application stores:

- Vehicle inventory.
- Machinery information.
- Maintenance history.
- Mileage and hour-meter readings.
- Insurance documentation.
- Administrative records.
- Operational reports.

The organization must protect both the database and the information required to operate the application.

### 23.1 Identify critical data

Not all information has the same recovery importance.

For example:

| Information | Recovery importance |
|---|---|
| Equipment inventory | Critical |
| Maintenance history | Critical |
| Insurance records | High |
| User permissions | High |
| Operational reports | Medium to high |
| Temporary report exports | Lower |

This classification helps define backup and retention requirements.

### 23.2 Define recovery objectives

A hypothetical strategy might establish:

- RTO: 2 hours.
- RPO: 15 minutes.
- Daily full backup.
- Continuous WAL archiving.
- Weekly off-site backup verification.
- Monthly restoration exercise.

These are illustrative objectives, not claims about SIAPV's existing configuration.

### 23.3 Proposed architecture

```text
                 SIAPV Application
                         |
                         v
                  PostgreSQL
                         |
              +----------+----------+
              |                     |
              v                     v
         Physical Backup        WAL Archive
              |                     |
              v                     v
         Local Storage         Remote Storage
              |                     |
              +----------+----------+
                         |
                         v
                  Backup Repository
                         |
                         v
                 Recovery Environment
```

### 23.4 Operational considerations

The organization should document:

- Who is authorized to restore information.
- Where backups are stored.
- How encryption keys are protected.
- How long backups are retained.
- How backup integrity is verified.
- How recovery is performed.
- How recovered information is validated.
- How the application is reconnected.
- How incidents are documented.

### 23.5 Architectural lesson

The objective is not simply to create backup files.

The objective is to demonstrate that the organization can recover its information and resume operations within an acceptable period.

That distinction separates a backup routine from a real disaster recovery capability.

## 24. Common Backup and Recovery Mistakes

### Mistake 1: Assuming a successful backup command guarantees recovery

A backup may be incomplete, corrupted, or missing dependencies.

Test restoration.

### Mistake 2: Storing every backup on the production server

A hardware failure or security incident may destroy both production data and backups.

Use independent storage.

### Mistake 3: Keeping only one backup generation

The latest backup may already contain corruption or unwanted changes.

Maintain historical recovery points.

### Mistake 4: Confusing replication with backup

Replication can reproduce unwanted changes.

Maintain independent backups.

### Mistake 5: Ignoring WAL archiving

A physical base backup without the necessary WAL files cannot provide the same point-in-time recovery capability.

### Mistake 6: Never measuring restoration time

A backup strategy may fail to meet the organization's RTO.

Measure recovery performance.

### Mistake 7: Ignoring backup security

Backup files may contain sensitive information.

Protect them with encryption and access controls.

### Mistake 8: Forgetting database roles and configuration

Restoring database records alone may not restore the complete application environment.

Document dependencies, roles, permissions, and configuration.

### Mistake 9: Assuming cloud storage guarantees recovery

Cloud object storage is a storage mechanism, not a complete disaster recovery plan.

Recovery infrastructure and procedures are still required.

## 25. Knowledge Check

Answer these questions without consulting the explanations above.

1. What is the purpose of a database backup?
2. What is the difference between backup and recovery?
3. Why is replication not a substitute for backup?
4. What is a full backup?
5. What is the difference between incremental and differential backups?
6. What is a logical backup?
7. What is a physical backup?
8. What is the difference between `pg_dump` and `pg_basebackup`?
9. What is Write-Ahead Logging?
10. Why is WAL important for database recovery?
11. What is WAL archiving?
12. What is Point-in-Time Recovery?
13. What information is needed to perform PITR?
14. What is a backup retention policy?
15. Why should backups be stored off-site?
16. What are immutable backups?
17. What does the 3-2-1 backup rule recommend?
18. Why must backups be tested?
19. What is the difference between pilot light and warm standby?
20. How do RTO and RPO influence backup architecture?

## 26. Practical Exercises

### Exercise 1: Select a backup strategy

A database contains 500 GB of information and changes approximately 10 GB per day.

Compare:

- Daily full backups.
- Weekly full backups with daily incremental backups.
- Weekly full backups with daily differential backups.

Discuss:

1. Storage requirements.
2. Backup duration.
3. Recovery complexity.
4. Operational risk.

### Exercise 2: Calculate the recovery point

A database receives WAL archives every five minutes.

The last successfully archived WAL segment corresponds to 11:55.

The database fails at 12:00.

Discuss the possible recovery point and why the archive schedule alone does not guarantee a precise five-minute RPO.

### Exercise 3: Design a retention policy

Design a backup retention policy for an institutional application.

Define:

- Daily backup retention.
- Weekly backup retention.
- Monthly backup retention.
- WAL retention.
- Off-site storage.
- Immutable copies.
- Restoration testing frequency.

Explain your decisions.

### Exercise 4: Analyze a ransomware incident

An attacker gains administrative access to the production database and deletes the database files.

The local backup directory is also deleted.

Answer:

1. Why did the backup architecture fail?
2. How could off-site storage help?
3. How could immutable backups help?
4. Why should backup credentials be separated?
5. What recovery infrastructure is required?

### Exercise 5: Plan PostgreSQL PITR

An organization has:

- A physical base backup from 01:00.
- WAL archives from 01:00 to 16:00.
- An accidental update at 15:30.

Develop a conceptual recovery plan to restore the database to 15:29.

Identify the required files, recovery configuration, validation steps, and operational precautions.

### Exercise 6: SIAPV recovery architecture

Design a recovery strategy for SIAPV.

Document:

- Critical data.
- Backup type.
- Backup frequency.
- WAL archiving.
- Retention.
- Storage location.
- Encryption.
- RTO.
- RPO.
- Recovery procedure.
- Restoration testing.

Justify every decision.

## 27. Key Vocabulary

| English term | Spanish equivalent | Meaning |
|---|---|---|
| Backup | Copia de seguridad | Recoverable copy of data. |
| Restore | Restaurar | Copy backup information into a target environment. |
| Recovery | Recuperación | Process of returning a system to a valid state. |
| Disaster Recovery | Recuperación ante desastres | Strategy for restoring systems after major disruptions. |
| Full Backup | Respaldo completo | Copy of all selected data. |
| Incremental Backup | Respaldo incremental | Copy of changes since the previous backup. |
| Differential Backup | Respaldo diferencial | Copy of changes since the last full backup. |
| Logical Backup | Respaldo lógico | Export of database objects and data in logical form. |
| Physical Backup | Respaldo físico | Copy of physical database files. |
| Write-Ahead Logging | Registro anticipado de escritura | Transaction logging mechanism used for recovery. |
| WAL Archiving | Archivado de WAL | Preservation of completed WAL segments. |
| Point-in-Time Recovery | Recuperación a un punto en el tiempo | Recovery to a specific historical point. |
| Retention Policy | Política de retención | Rules defining how long backups are preserved. |
| Off-site Backup | Respaldo externo | Backup stored in a separate location. |
| Immutable Backup | Respaldo inmutable | Backup protected from modification or deletion. |
| Backup Rotation | Rotación de respaldos | Management of backup generations. |
| Restoration Testing | Pruebas de restauración | Verification that backups can be restored. |
| Recovery Runbook | Procedimiento de recuperación | Documented recovery instructions. |
| Business Continuity | Continuidad del negocio | Ability to maintain essential business operations. |
| Recovery Environment | Entorno de recuperación | Infrastructure used to restore services. |

## 28. Connections to Other Notes

### Previous knowledge

- [[05 - Transactions and ACID]]
- [[03 - Storage Pages and Tables]]
- [[03 - Replication]]
- [[06 - High Availability]]

### Related concepts

- [[01 - PostgreSQL Architecture Overview]]
- [[03 - PostgreSQL Storage]]
- [[05 - PostgreSQL Performance Fundamentals]]
- [[01 - Database Security Fundamentals]]
- [[03 - Data Quality]]

### Conceptual relationship

```text
Database Protection
        |
        +------> High Availability
        |              |
        |              v
        |          Failover
        |
        +------> Backup
        |              |
        |              v
        |           Restore
        |
        +------> WAL Archiving
        |              |
        |              v
        |             PITR
        |
        +------> Disaster Recovery
                       |
                       v
                Service Restoration
```

A reliable database architecture must consider both service continuity and historical data recovery.

High availability helps keep the system running when components fail. Backups preserve recoverable information. Disaster recovery combines infrastructure, data, procedures, and people to restore operations after major incidents.
