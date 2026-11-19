
---
title: "Database High Availability"
aliases:
  - "Database HA"
  - "High Availability Architecture"
  - "Database Failover and Redundancy"
type: learning-note
domain: technology
category: database-architecture
subject: high-availability
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - database-architecture
  - high-availability
  - fault-tolerance
  - failover
  - switchover
  - replication
  - disaster-recovery
  - postgresql
  - rto
  - rpo
concepts:
  - availability
  - uptime
  - downtime
  - sla
  - slo
  - sli
  - rto
  - rpo
  - redundancy
  - active-passive
  - active-active
  - health-check
  - heartbeat
  - quorum
  - failover
  - switchover
  - split-brain
  - fencing
  - failure-domain
  - multi-az
  - multi-region
  - observability
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Client Server Architecture]]"
  - "[[02 - Database Scalability]]"
  - "[[03 - Replication]]"
  - "[[05 - Sharding]]"
  - "[[05 - Transactions and ACID]]"
related:
  - "[[07 - Backup and Disaster Recovery]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[02 - PostgreSQL Processes]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"
  - "[[01 - Database Security Fundamentals]]"
tools:
  - PostgreSQL
  - SQL
  - psql
  - pg_ctl
  - Patroni
  - repmgr
  - HAProxy
  - PgBouncer
  - Docker
learning_objectives:
  - "Define database high availability and explain its objectives."
  - "Calculate availability and interpret common uptime targets."
  - "Distinguish RTO, RPO, durability, reliability, and availability."
  - "Explain redundancy, failure domains, and single points of failure."
  - "Compare active-passive and active-active architectures."
  - "Explain failover, switchover, split-brain, and fencing."
  - "Understand how PostgreSQL replication contributes to HA."
  - "Identify monitoring and operational requirements for HA."
  - "Distinguish HA from backup and disaster recovery."
  - "Evaluate an appropriate HA design for a practical application."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Database High Availability

## 1. Introduction

A database can be correctly designed, normalized, indexed, secured, and optimized while still being unavailable when users need it.

Consider an application that manages vehicle inventories, maintenance records, equipment assignments, and operational reports.

The application depends on several components:

- Application server.
- Database server.
- Storage system.
- Network connectivity.
- Authentication services.
- Operating system.
- Database management system.

If the database server suddenly stops working, the application may become unable to retrieve or save information.

The database may contain perfectly intact records, but users cannot access them.

This introduces an important architectural requirement:

**A database system must not only store data correctly; it must also remain accessible when failures occur.**

High availability addresses this requirement.

## 2. What Is High Availability?

**High Availability (HA)** is the ability of a system to remain operational and accessible for an agreed proportion of time, despite failures in individual components.

A high-availability architecture reduces downtime through mechanisms such as:

- Redundant infrastructure.
- Database replication.
- Health monitoring.
- Failure detection.
- Automatic or manual failover.
- Load balancing.
- Recovery procedures.
- Operational testing.

The objective is not necessarily to prevent every failure.

Instead, the architecture must be prepared to tolerate failures and restore service within acceptable limits.

### 2.1 Availability versus fault tolerance

These concepts are related but different.

| Concept | Meaning |
|---|---|
| Availability | The proportion of time a service is operational and accessible. |
| Reliability | The probability that a system performs correctly over a specified period. |
| Fault tolerance | The ability to continue operating when one or more components fail. |
| Resilience | The ability to withstand failures, recover, and adapt. |
| Durability | The ability to preserve committed data over time. |

A database can be highly available but have weak durability if committed data can be lost.

Likewise, a database can preserve every record but remain unavailable for several hours.

**Example:**

A database server fails at 10:00 AM.

- Its data files remain intact.
- A standby server is promoted at 10:02 AM.
- The application reconnects at 10:03 AM.

The system experienced downtime, but its architecture limited the interruption.

This is the central objective of HA.

## 3. Measuring Availability

Availability is commonly expressed as a percentage.

The basic formula is:

\[
Availability =
\frac{Uptime}{Uptime + Downtime}
\times 100
\]

Where:

- **Uptime:** Time during which the service is operational.
- **Downtime:** Time during which the service is unavailable.

### 3.1 Availability targets

Common availability targets include:

| Availability | Approximate annual downtime |
|---|---:|
| 99% | 3 days, 15 hours, 36 minutes |
| 99.9% | 8 hours, 45 minutes, 36 seconds |
| 99.95% | 4 hours, 22 minutes, 48 seconds |
| 99.99% | 52 minutes, 34 seconds |
| 99.999% | 5 minutes, 15 seconds |

These estimates assume a 365-day year.

The difference between 99.9% and 99.99% may appear small, but it represents a significant reduction in acceptable downtime.

### 3.2 The cost of higher availability

Higher availability usually requires more infrastructure and operational complexity.

For example:

**Architecture A**
- One database server.
- Local storage.
- Manual recovery.
- Regular backups.

**Architecture B**
- Primary database.
- Standby database.
- Continuous replication.
- Automated failure detection.
- Failover orchestration.
- Redundant networking.

Architecture B can provide better availability, but it also requires additional resources, monitoring, testing, and maintenance.

The appropriate target depends on business requirements.

A small internal inventory application may not justify the same architecture as a financial transaction platform.

## 4. SLA, SLO, and SLI

Availability targets are often defined using three operational concepts.

### 4.1 Service Level Indicator (SLI)

An SLI is a measurable indicator of service performance.

Examples:

- Percentage of successful database connections.
- Percentage of successful queries.
- Database response latency.
- Percentage of time the database endpoint is reachable.
- Number of failed requests.

An SLI measures actual behavior.

### 4.2 Service Level Objective (SLO)

An SLO is an internal or operational target for an SLI.

Example:

"The database service should achieve 99.95% availability per month."

This defines the performance level the organization intends to maintain.

### 4.3 Service Level Agreement (SLA)

An SLA is a formal agreement that defines expected service levels and may include consequences if those levels are not achieved.

For example, a cloud database provider may offer an availability commitment under specific contractual conditions.

| Term | Main purpose |
|---|---|
| SLI | Measures actual service behavior. |
| SLO | Defines the target performance. |
| SLA | Establishes a formal service commitment. |

These concepts are useful when evaluating managed database services and enterprise infrastructure.

## 5. Recovery Objectives: RTO and RPO

Availability architecture must define how quickly a service should recover and how much data loss is acceptable.

Two critical concepts are:

- Recovery Time Objective (RTO).
- Recovery Point Objective (RPO).

### 5.1 Recovery Time Objective (RTO)

**RTO is the maximum acceptable amount of time required to restore a service after an interruption.**

Example:

An organization establishes an RTO of 15 minutes.

If the database becomes unavailable at 10:00 AM, the service should be restored by 10:15 AM at the latest.

RTO answers:

"How long can the service remain unavailable?"

### 5.2 Recovery Point Objective (RPO)

**RPO is the maximum acceptable amount of data loss, measured in time.**

Example:

An organization establishes an RPO of 5 minutes.

If the database fails at 10:00 AM, the organization must recover to a point no earlier than approximately 9:55 AM.

RPO answers:

"How much recent data can the organization afford to lose?"

### 5.3 RTO and RPO comparison

| Dimension | RTO | RPO |
|---|---|---|
| Focus | Service restoration | Data recovery point |
| Measures | Time without service | Potential data loss |
| Example | 15 minutes | 5 minutes |
| Main concern | Operational continuity | Data preservation |

### 5.4 Practical scenario

Imagine a maintenance application that records equipment repairs.

A database failure occurs at 14:00.

The organization has:

- RTO: 20 minutes.
- RPO: 5 minutes.

The recovery objectives are:

1. Restore database access before 14:20.
2. Recover the database with no more than approximately five minutes of lost transactions.

A system that recovers at 14:10 but loses 30 minutes of recent records satisfies the RTO but violates the RPO.

A system that preserves every transaction but takes two hours to restore violates the RTO.

Both objectives must be considered independently.

## 6. Failure Domains

A failure domain is a group of components that may become unavailable because of the same failure event.

Understanding failure domains is essential when designing redundancy.

### 6.1 Common failure domains

| Failure domain | Example |
|---|---|
| Process | PostgreSQL process crashes. |
| Host | Physical or virtual server becomes unavailable. |
| Storage | Disk or storage subsystem fails. |
| Network | Database server loses network connectivity. |
| Availability Zone | A data center zone experiences an outage. |
| Region | A complete geographic cloud region becomes unavailable. |
| Application | A deployment introduces a critical software bug. |
| Human operation | An administrator accidentally deletes data. |
| External dependency | Authentication or DNS service becomes unavailable. |

A system with two database instances on the same physical host does not provide strong protection against host failure.

Both instances may fail simultaneously.

Similarly, two servers connected to the same vulnerable storage system may share a failure domain.

### 6.2 Independent failure domains

A stronger architecture separates redundant components.

For example:

- Primary database in Availability Zone A.
- Standby database in Availability Zone B.
- Separate network paths where practical.
- Independent storage.
- Monitoring outside the database hosts.

The objective is to reduce the probability that one failure event disables every component.

## 7. Redundancy and Single Points of Failure

**Redundancy** means providing additional components or resources so that the system can continue operating when one component fails.

A Single Point of Failure (SPOF) is a component whose failure can interrupt the entire service.

### 7.1 Example of a single point of failure

Consider this architecture:

```text
Users
  |
  v
Application Server
  |
  v
Database Server
  |
  v
Storage
```

If the database server fails, the application cannot access its data.

The database server is a single point of failure.

### 7.2 Redundant architecture

A more resilient design could look like this:

```text
                  Users
                    |
                    v
             Load Balancer
                    |
                    v
             Application Layer
                    |
                    v
              Database Router
                    |
              +-----+-----+
              |           |
              v           v
          Primary      Standby
          Database     Database
              |           ^
              +-----------+
                Replication
```

The standby provides an alternative database instance.

However, simply having two database servers does not guarantee high availability.

The architecture must also determine:

- Which server is currently primary?
- How is failure detected?
- How is the standby promoted?
- How does the application find the new primary?
- How are duplicate primary instances prevented?
- How is the old primary safely reintroduced?

These questions are part of HA engineering.

### 7.3 N+1 redundancy

N+1 redundancy means providing one additional component beyond the number required for normal operation.

For example:

- Three application servers are required.
- Four application servers are deployed.
- One server can fail while the remaining three continue handling the required workload.

The same principle can apply to infrastructure capacity, although database replication requires additional considerations.

## 8. Active-Passive Architecture

An active-passive architecture uses one primary component to handle the workload while another component remains available as a standby.

In database systems, the common arrangement is:

- Primary: accepts writes and usually serves reads.
- Standby: receives replicated data and waits to take over.

```text
                 Application
                      |
                      v
                Primary DB
                      |
                Replication
                      |
                      v
                 Standby DB
                 (Passive)
```

### 8.1 Normal operation

1. The application connects to the primary.
2. The primary executes transactions.
3. Changes are recorded in the transaction log.
4. Changes are transmitted to the standby.
5. The standby applies the replicated changes.

### 8.2 Primary failure

If the primary becomes unavailable:

1. The monitoring system detects the failure.
2. The HA mechanism evaluates whether promotion is safe.
3. The standby is promoted to primary.
4. The application is redirected.
5. Clients reconnect and resume operations.

This process is called **failover**.

### 8.3 Advantages

- Clear write ownership.
- Simpler transaction management than multi-primary architectures.
- Reduced risk of write conflicts.
- Standby can provide read capacity when configured appropriately.

### 8.4 Limitations

- The standby may not be immediately ready.
- Asynchronous replication can lose recent transactions.
- Failover may require client reconnections.
- The system requires monitoring and orchestration.
- A standby that shares the same failure domain may not provide sufficient protection.

## 9. Active-Active Architecture

An active-active architecture allows multiple instances to serve traffic simultaneously.

For example:

```text
                 Application
                      |
                Load Balancer
                 /         \
                v           v
           Database A   Database B
                \           /
                 \---------/
             Data Synchronization
```

Both database instances may actively process requests.

### 9.1 Advantages

- Workload distribution.
- Potentially higher read capacity.
- Better utilization of available infrastructure.
- Reduced dependence on a single active instance.

### 9.2 Challenges

Active-active database architectures are significantly more complex.

Consider two instances modifying the same record:

- Database A changes an equipment status to "Under Maintenance".
- Database B changes the same equipment status to "Available".

The system must determine how to resolve the conflict.

Other challenges include:

- Write conflicts.
- Transaction ordering.
- Data consistency.
- Distributed locking.
- Network partitions.
- Conflict resolution.
- Duplicate operations.

Active-active architectures are not automatically more available or more consistent.

PostgreSQL's standard streaming replication is primarily a primary-standby architecture. General multi-primary writes are not provided by PostgreSQL's built-in streaming replication.

Specialized distributed databases and additional technologies can support different multi-writer models.

## 10. Replication as an HA Mechanism

Replication is one of the foundations of database high availability.

It maintains additional database copies that can potentially take over when the primary fails.

The details of replication were covered in:

[[03 - Replication]]

For HA, the most important distinction is between synchronous and asynchronous replication.

### 10.1 Synchronous replication

In synchronous replication, a transaction may wait for confirmation from one or more standby servers before being acknowledged as committed.

Simplified flow:

```text
Application
    |
    v
Primary Database
    |
    | Send transaction changes
    v
Standby Database
    |
    | Confirmation
    v
Primary acknowledges commit
```

**Advantages:**

- Can reduce the amount of acknowledged data lost during failover.
- Provides stronger durability guarantees when configured correctly.

**Disadvantages:**

- Higher transaction latency.
- Dependence on standby availability.
- Network problems can affect write availability.

Synchronous replication does not automatically guarantee zero data loss in every possible failure scenario. The exact guarantee depends on configuration, acknowledgment mode, promotion policy, and failure conditions.

### 10.2 Asynchronous replication

In asynchronous replication, the primary can acknowledge a transaction before the standby confirms that it has received and applied the changes.

```text
Application
    |
    v
Primary Database
    |
    +------> Acknowledge commit
    |
    v
Standby receives changes later
```

**Advantages:**

- Lower transaction latency.
- Less dependence on immediate standby acknowledgment.
- Suitable for geographically distant replicas.

**Disadvantages:**

- Replication lag may occur.
- Recent transactions may be lost during failover.
- The standby may not contain the latest committed changes.

### 10.3 Replication lag

Replication lag is the delay between changes occurring on the primary and becoming available on a replica.

Example:

- A transaction commits at 10:00:00.
- The standby receives the corresponding WAL records at 10:00:02.
- The standby applies them at 10:00:04.

The standby is approximately four seconds behind the primary in this example.

Replication lag is important because it affects recovery objectives.

A system with asynchronous replication and 30 seconds of lag may lose recent transactions if the primary fails before those changes reach the standby.

### 10.4 Replication mode and RPO

| Replication model | Potential RPO |
|---|---|
| Asynchronous | May be greater than zero. |
| Synchronous | Can approach zero for acknowledged transactions under the configured failure assumptions. |
| Backup-based recovery | Depends on backup frequency and log archiving. |

The selected replication strategy must reflect the organization's acceptable data-loss tolerance.

## 11. Failover and Switchover

These terms describe different ways of changing the active database server.

### 11.1 Failover

Failover is the process of moving service to a standby system after an unexpected failure.

Example:

- Primary database crashes.
- Standby is promoted.
- Application traffic moves to the standby.

Failover is generally a response to an incident.

### 11.2 Switchover

Switchover is a planned transition from one database instance to another.

Example:

- The primary needs maintenance.
- Administrators verify that the standby is synchronized.
- The primary role is transferred deliberately.
- The application continues operating through the new primary.

A switchover should minimize disruption and reduce the risk of data loss.

### 11.3 Comparison

| Characteristic | Failover | Switchover |
|---|---|---|
| Trigger | Unexpected failure | Planned operation |
| Initiation | Automatic or manual | Usually controlled |
| Main purpose | Restore service | Change the active node safely |
| Typical scenario | Primary server crashes | Scheduled maintenance |

## 12. Split-Brain

Split-brain is a dangerous condition in which two database instances incorrectly believe they are the primary.

Imagine a primary and standby connected through a network.

A network partition occurs:

- The primary cannot communicate with the standby.
- The standby cannot determine whether the primary has failed or is merely unreachable.
- Both instances may accept writes if the HA system is poorly designed.

The result can be two independent histories of data.

```text
                 Network Partition

             Primary A       Standby B
                 |               |
             Accepts writes  Accepts writes
                 |               |
                 v               v
              History A       History B
```

When connectivity returns, the system may have conflicting records.

This can cause data corruption, inconsistent business operations, or difficult recovery.

### 12.1 Why split-brain happens

Possible causes include:

- Network partitions.
- Incorrect failure detection.
- Poorly configured automated failover.
- Multiple systems attempting promotion.
- Lack of coordination between nodes.

### 12.2 Preventing split-brain

Architectures use mechanisms such as:

- Quorum-based decisions.
- Distributed consensus.
- Leader election.
- Fencing.
- Strict primary-role management.

A standby should not become primary merely because it cannot contact the existing primary.

The system must establish that the promotion is safe.

## 13. Quorum and Leader Election

### 13.1 Quorum

A quorum is the minimum number of participating members required to make a valid decision.

In a cluster with three voting members, a majority is two.

If one member becomes unavailable, the remaining two can still form a quorum.

If only one member remains, it cannot form a majority.

Quorum helps prevent isolated nodes from making conflicting decisions.

The exact quorum rules depend on the clustering and consensus system.

### 13.2 Leader election

Leader election is the process by which a cluster selects one member to coordinate operations.

In a primary-standby database architecture, this may help determine which instance should act as primary.

A reliable leader-election process must account for:

- Node failures.
- Network partitions.
- Membership changes.
- Timeouts.
- Voting rules.
- Safe promotion.

Leader election and quorum are especially important in distributed systems.

## 14. Fencing

Fencing is a mechanism that prevents a failed or isolated node from continuing to operate when it should no longer have control of a resource.

For example:

A primary database becomes unreachable from the cluster.

Before promoting a standby, the HA system must prevent the old primary from accepting new writes if it later becomes reachable again.

Possible fencing approaches include:

- Powering off the failed host through infrastructure management.
- Disabling its access to shared storage.
- Revoking its access to a resource.
- Using a cluster manager to enforce exclusive ownership.

Fencing is important because failure detection alone does not prove that a server has stopped operating.

A server can be alive but isolated from the rest of the system.

## 15. Failover Architecture: The Complete Process

A database failover is not simply a command that starts another server.

It is an operational sequence involving several components.

### 15.1 Typical sequence

1. **Failure detection:** Monitoring detects that the primary is unhealthy.
2. **Validation:** The HA manager determines whether the failure is genuine and whether promotion is safe.
3. **Fencing:** The old primary is prevented from accepting conflicting writes when necessary.
4. **Promotion:** A suitable standby becomes the new primary.
5. **Routing:** The application is redirected to the new primary.
6. **Reconnection:** Application connections are re-established.
7. **Verification:** Monitoring confirms that the service is operating.
8. **Recovery:** The former primary is repaired and reintegrated as a standby when safe.

### 15.2 Simplified architecture

```text
                 Application
                      |
                      v
                Database Endpoint
                      |
                      v
                HA Router / Proxy
                      |
               +------+------+
               |             |
               v             v
           Primary        Standby
           (Leader)       (Replica)
               |
               | Replication
               +------------->
```

During failover, the standby becomes the new primary.

The database endpoint or proxy must then direct new connections to the promoted instance.

### 15.3 Connection handling

Existing connections to the failed primary usually cannot continue normally.

Applications may need to:

- Detect connection failures.
- Retry requests safely.
- Re-establish database connections.
- Use a connection pool that can discard stale connections.
- Avoid repeating non-idempotent operations without checking their outcome.

This is why application architecture is part of database HA.

A highly available database does not automatically make the entire application highly available.

## 16. Failback and Reintegration

Failback is the process of returning to the preferred or original infrastructure after the failure has been resolved.

Example:

1. Database A was originally primary.
2. Database A fails.
3. Database B is promoted.
4. Database A is repaired.
5. Database A is synchronized with Database B.
6. The architecture decides whether to return the primary role to Database A.

The original server must not simply restart and begin accepting writes.

It may contain an outdated data history.

It must first be safely reintegrated.

Depending on the technology, this can involve rebuilding the replica or using a tool that safely reconciles its state.

Failback may be performed automatically or manually.

In many environments, keeping the new primary active is safer until a planned switchover is appropriate.

## 17. PostgreSQL High Availability

PostgreSQL provides important building blocks for HA, but a complete HA architecture generally requires additional orchestration and routing mechanisms.

### 17.1 PostgreSQL streaming replication

PostgreSQL supports physical streaming replication.

A primary sends WAL records to one or more standby servers.

The standby continuously receives and replays those records.

This provides a foundation for primary-standby architectures.

Relevant concepts:

- WAL.
- Primary server.
- Standby server.
- Replication connection.
- Synchronous replication.
- Asynchronous replication.
- Replication slots.
- Recovery and promotion.

See:

- [[02 - PostgreSQL Processes]]
- [[03 - PostgreSQL Storage]]
- [[05 - Transactions and ACID]]

### 17.2 Promoting a standby with pg_ctl

PostgreSQL includes the `pg_ctl` utility.

A standby can be promoted using:

```bash
pg_ctl promote -D /path/to/standby/data
```

This requests promotion of the standby to a primary server.

The exact directory and command execution environment depend on the installation.

### 17.3 Promoting with SQL

PostgreSQL also provides:

```sql
SELECT pg_promote();
```

This requests promotion of a standby server.

Promotion changes the server's recovery state so it can operate as a primary.

It does not automatically configure the entire application routing or cluster-management process.

### 17.4 pg_rewind

`pg_rewind` is a PostgreSQL utility that can synchronize a former primary with a new timeline after certain failover scenarios.

It can reduce the need to copy an entire database cluster again.

However, its use has prerequisites and operational constraints, including the required data checks and configuration.

It is not a universal repair command.

### 17.5 HA orchestration tools

Several tools help coordinate PostgreSQL HA.

| Tool | Primary role |
|---|---|
| Patroni | PostgreSQL HA orchestration, leader election, and failover management. |
| repmgr | PostgreSQL replication and failover management. |
| HAProxy | TCP/HTTP proxying and traffic routing. |
| PgBouncer | PostgreSQL connection pooling. |
| Keepalived | Virtual IP and high-availability networking mechanisms. |

These tools solve different problems.

For example:

- Patroni helps manage database roles and cluster state.
- HAProxy can route client connections.
- PgBouncer reduces connection overhead.
- Keepalived can help provide a highly available virtual IP.

They are not interchangeable.

A complete architecture may use several of them together.

### 17.6 Managed database services

Cloud providers also offer managed database services with varying levels of HA support.

These services may provide:

- Multi-zone deployment.
- Automated failover.
- Managed replication.
- Health monitoring.
- Backup integration.
- Infrastructure maintenance.

However, availability guarantees depend on the selected service tier, region, configuration, and provider agreement.

A managed database does not remove the need to understand RTO, RPO, backups, application reconnection, and failure scenarios.

## 18. Monitoring and Observability

High availability requires continuous visibility into the system.

Without monitoring, an organization may discover a problem only after users report that the application is unavailable.

### 18.1 Important metrics

| Metric | Why it matters |
|---|---|
| Database availability | Indicates whether the database endpoint is accessible. |
| Replication lag | Shows how far a standby is behind. |
| Connection count | Detects connection exhaustion. |
| Query latency | Identifies slow database operations. |
| CPU utilization | Reveals processing bottlenecks. |
| Memory utilization | Helps identify resource pressure. |
| Disk utilization | Detects storage capacity problems. |
| WAL generation | Helps understand transaction-log activity. |
| WAL storage usage | Prevents disk exhaustion. |
| Failed connections | Indicates connection or service problems. |
| Failover events | Tracks role transitions and incidents. |

### 18.2 Health checks

A health check is a test used to determine whether a component is functioning correctly.

Examples:

- Can the database accept a connection?
- Can the application execute a simple query?
- Is the primary responding?
- Is the standby receiving WAL records?
- Is replication lag within acceptable limits?

A basic PostgreSQL connectivity check can be performed with:

```bash
pg_isready -h localhost -p 5432
```

This checks whether PostgreSQL is accepting connections.

However, a successful connectivity check does not guarantee that every application operation is functioning correctly.

### 18.3 Heartbeats

A heartbeat is a periodic signal exchanged between components to indicate that they are responsive.

If a cluster member stops sending heartbeats, the system may suspect that it has failed.

Heartbeats are useful, but network interruptions can make a healthy node appear unavailable.

Therefore, heartbeat failures must be interpreted carefully.

### 18.4 Alerting

Monitoring should generate alerts when conditions exceed defined thresholds.

Examples:

- Standby replication lag exceeds 30 seconds.
- Database connections reach 90% of the configured limit.
- Disk space falls below 15%.
- Database endpoint becomes unreachable.
- WAL storage grows unexpectedly.
- A failover occurs.

Alerts should be actionable and connected to an incident-response process.

## 19. High Availability, Backup, and Disaster Recovery

These concepts are related, but they solve different problems.

### 19.1 High availability

HA protects service continuity when infrastructure components fail.

Example:

The primary database server crashes, and a standby takes over.

### 19.2 Backup

A backup provides a recoverable copy of data from an earlier point in time.

It protects against situations such as:

- Accidental deletion.
- Logical corruption.
- Incorrect updates.
- Human errors.
- Some ransomware incidents.
- Database loss.

A standby replica is not a substitute for a backup.

If an administrator accidentally deletes a record on the primary, that deletion may also be replicated to the standby.

### 19.3 Disaster recovery

Disaster Recovery (DR) is the broader strategy for restoring systems after major disruptive events.

Examples:

- Complete data center failure.
- Regional cloud outage.
- Destruction of infrastructure.
- Large-scale security incident.
- Loss of critical business systems.

DR can involve:

- Off-site backups.
- Cross-region replication.
- Recovery infrastructure.
- Documented procedures.
- Alternative operating environments.
- Business continuity plans.

### 19.4 Comparison

| Capability | HA | Backup | DR |
|---|---|---|---|
| Component failure | Primary focus | Not primary focus | May be included |
| Accidental deletion | Not sufficient | Primary protection | May be included |
| Logical corruption | Not sufficient | Important protection | May be included |
| Regional disaster | Limited unless geographically distributed | Depends on backup location | Primary focus |
| Fast service restoration | Primary focus | Depends on restore process | Depends on recovery strategy |
| Historical recovery | Not its main purpose | Primary focus | Often included |

A robust architecture usually needs all three.

See:

[[07 - Backup and Disaster Recovery]]

## 20. Multi-AZ and Multi-Region Architectures

High availability can be implemented across different infrastructure locations.

### 20.1 Multi-AZ

An Availability Zone (AZ) is an isolated infrastructure location within a cloud region.

A multi-AZ database architecture distributes components across different zones.

Example:

```text
             Cloud Region
             /           \
            /             \
           v               v
     Availability       Availability
       Zone A              Zone B
          |                  |
          v                  v
       Primary             Standby
       Database            Database
            \              /
             \------------/
               Replication
```

The objective is to tolerate the failure of an individual availability zone.

Multi-AZ architectures often provide lower network latency than cross-region architectures.

### 20.2 Multi-Region

A multi-region architecture distributes infrastructure across geographically separate regions.

Example:

```text
             Region A
                |
                v
           Primary DB
                |
           Replication
                |
                v
             Region B
                |
                v
           Standby DB
```

This provides protection against larger regional failures.

However, geographic distance introduces challenges:

- Higher network latency.
- Greater replication delay.
- More complicated failover.
- Potentially higher infrastructure costs.
- More complex data consistency requirements.

### 20.3 Multi-AZ versus multi-region

| Characteristic | Multi-AZ | Multi-region |
|---|---|---|
| Geographic scope | Same region | Different regions |
| Network latency | Usually lower | Usually higher |
| Failure protection | Zone-level failures | Regional failures |
| Operational complexity | Moderate | Higher |
| Typical use | High availability | Disaster recovery and geographic resilience |

A multi-region design is not automatically better for every application.

The organization must evaluate cost, latency, data consistency, and recovery objectives.

## 21. Testing High Availability

An HA architecture should never be considered reliable simply because it was configured successfully.

It must be tested.

### 21.1 Failover testing

A controlled failover test can verify:

- Whether failure detection works.
- Whether the standby can be promoted.
- Whether applications reconnect.
- Whether data loss remains within the RPO.
- Whether service restoration meets the RTO.
- Whether monitoring and alerts work.
- Whether the former primary can be safely reintegrated.

### 21.2 Game days

A game day is a planned exercise in which an organization simulates failures to evaluate its operational readiness.

Examples:

- Stop a database process.
- Disconnect a network interface.
- Simulate a standby failure.
- Exhaust connection capacity in a controlled environment.
- Simulate a storage interruption.

The objective is to discover weaknesses before a real incident occurs.

### 21.3 Chaos engineering

Chaos engineering is a disciplined approach to testing system resilience by introducing controlled failures.

It requires:

- Defined hypotheses.
- Controlled experiments.
- Monitoring.
- Safety limits.
- Recovery procedures.
- Documented results.

It should not be confused with randomly breaking production systems.

The objective is to learn how the architecture behaves under failure conditions.

## 22. Designing an HA Architecture for SIAPV

Consider a future version of SIAPV.

The current application manages approximately 200 vehicles and machinery units.

Its responsibilities include:

- Inventory management.
- Equipment identification.
- Maintenance records.
- Mileage and hour-meter tracking.
- Insurance information.
- Operational reports.
- Administrative records.

At this scale, a complex distributed database architecture may be unnecessary.

However, suppose the application becomes a critical institutional service.

It is now used by multiple departments, and database downtime prevents personnel from consulting or updating essential information.

The organization may decide that it needs a formal availability target.

### 22.1 Possible architecture

```text
                   Users
                     |
                     v
               Web Application
                     |
                     v
               Database Endpoint
                     |
                     v
                Primary DB
                     |
               WAL Replication
                     |
                     v
                Standby DB

             Independent Backups
                     |
                     v
             Backup Repository

             Monitoring System
                     |
             +-------+-------+
             |               |
          Primary          Standby
```

### 22.2 Architectural decisions

The design must answer:

**Availability target**

What percentage of time must the application remain operational?

**RTO**

How long can the application remain unavailable?

**RPO**

How much recent information can the institution afford to lose?

**Replication**

Should the standby receive changes synchronously or asynchronously?

**Failure detection**

How will the architecture determine that the primary has failed?

**Failover**

Will promotion be automatic or manual?

**Application routing**

How will the application connect to the new primary?

**Backup**

How will the organization recover records deleted accidentally or restore an earlier database state?

**Monitoring**

Who receives alerts when replication or connectivity fails?

### 22.3 A proportional design

For an application of this size, a reasonable initial design might include:

1. A primary PostgreSQL server.
2. A standby in a separate failure domain, if the availability requirement justifies it.
3. A defined promotion and reconnection procedure.
4. Automated backups.
5. WAL archiving if point-in-time recovery is required.
6. Monitoring of database availability and replication.
7. Documented recovery procedures.
8. Periodic restoration and failover tests.

The architecture should grow according to actual operational requirements.

The goal is not to use the largest possible infrastructure.

The goal is to provide the required level of service with an acceptable cost and complexity.

## 23. Common High Availability Mistakes

### Mistake 1: Assuming replication guarantees availability

Replication creates additional copies of data, but it does not automatically manage failover, routing, or application reconnection.

### Mistake 2: Treating a standby as a backup

A replica can reproduce accidental deletions and logical corruption.

Independent backups are still necessary.

### Mistake 3: Ignoring replication lag

An asynchronous standby may not contain the latest transactions.

Promotion decisions must account for this possibility.

### Mistake 4: Ignoring split-brain

Promoting a standby without ensuring that the old primary cannot continue accepting writes can create conflicting database histories.

### Mistake 5: Testing only the database server

The application, network, proxy, connection pool, authentication, and dependent services must also be considered.

### Mistake 6: Assuming automatic failover is always safer

An incorrect automatic failover decision can create additional problems.

Automation must be supported by reliable failure detection and safe promotion rules.

### Mistake 7: Ignoring recovery procedures

A system is not operationally resilient if administrators do not know how to recover it.

### Mistake 8: Confusing uptime with business continuity

A database can be online while the application remains unusable because another critical dependency has failed.

## 24. Knowledge Check

Answer these questions without looking at the explanations above.

1. What is database high availability?
2. How is availability calculated?
3. What is the difference between 99.9% and 99.99% availability?
4. What is the difference between reliability and availability?
5. What does RTO measure?
6. What does RPO measure?
7. Why are failure domains important?
8. What is a single point of failure?
9. How does active-passive architecture work?
10. What makes active-active architecture more complex?
11. What is the difference between failover and switchover?
12. Why can asynchronous replication cause data loss during failover?
13. What is replication lag?
14. What is split-brain?
15. How does quorum help prevent conflicting decisions?
16. What is fencing?
17. Why is application reconnection part of HA?
18. Why is a standby not a replacement for backups?
19. What is the difference between multi-AZ and multi-region?
20. Why must failover procedures be tested?

## 25. Practical Exercises

### Exercise 1: Availability calculation

A database service operates for 29 days and is unavailable for 1 day during a 30-day measurement period.

Calculate its availability.

Use:

\[
Availability =
\frac{Uptime}{Total\ Time}
\times 100
\]

### Exercise 2: Recovery objectives

A company establishes:

- RTO: 10 minutes.
- RPO: 2 minutes.

A database failure occurs at 16:00.

Answer:

1. What is the latest acceptable restoration time?
2. What is the maximum acceptable data-loss window?
3. Would a recovery at 16:20 satisfy the RTO?
4. Would losing 15 minutes of transactions satisfy the RPO?

### Exercise 3: Identify single points of failure

Consider this infrastructure:

```text
Users
  |
Router
  |
Application Server
  |
Database Server
  |
Storage System
```

Identify every component that could independently interrupt the application.

Propose a redundancy strategy for each one.

### Exercise 4: Choose a replication strategy

A database supports financial transactions.

The business requires very low data loss, but the primary and standby servers are located in different regions.

Discuss:

1. Synchronous versus asynchronous replication.
2. Transaction latency.
3. Network interruptions.
4. RPO requirements.
5. The cost of the selected architecture.

### Exercise 5: Analyze split-brain

A primary database loses network connectivity with its standby.

The standby is automatically promoted, but the original primary continues accepting writes.

Explain:

1. Why this is dangerous.
2. What fencing mechanism could prevent it.
3. Why failure detection alone is insufficient.
4. How the system should reintegrate the former primary.

### Exercise 6: Design SIAPV availability

Define a hypothetical HA architecture for SIAPV.

Document:

- Availability target.
- RTO.
- RPO.
- Primary database.
- Standby database.
- Replication strategy.
- Failover mechanism.
- Application reconnection.
- Backup strategy.
- Monitoring.
- Recovery procedure.

Justify each decision according to the application's operational requirements.

## 26. Key Vocabulary

| English term | Spanish equivalent | Meaning |
|---|---|---|
| High Availability | Alta disponibilidad | Ability to maintain service availability. |
| Uptime | Tiempo operativo | Time during which a service is available. |
| Downtime | Tiempo de inactividad | Time during which a service is unavailable. |
| Reliability | Confiabilidad | Probability of correct operation over time. |
| Fault Tolerance | Tolerancia a fallos | Ability to continue operating despite failures. |
| Redundancy | Redundancia | Additional components for failure protection. |
| Single Point of Failure | Punto único de fallo | Component whose failure interrupts the service. |
| Failure Domain | Dominio de fallo | Group of components exposed to the same failure. |
| Recovery Time Objective | Objetivo de tiempo de recuperación | Maximum acceptable service restoration time. |
| Recovery Point Objective | Objetivo de punto de recuperación | Maximum acceptable data-loss window. |
| Failover | Conmutación por fallo | Transfer of service after an unexpected failure. |
| Switchover | Cambio planificado | Controlled transfer of the active role. |
| Failback | Retorno al sistema original | Return to a preferred infrastructure configuration. |
| Standby | Servidor en espera | Secondary instance ready to take over. |
| Replication Lag | Retraso de replicación | Delay between primary changes and replica application. |
| Split-Brain | Cerebro dividido | Condition where multiple nodes incorrectly act as primary. |
| Quorum | Cuórum | Minimum number of members required for a valid decision. |
| Leader Election | Elección de líder | Process for selecting a coordinating node. |
| Fencing | Aislamiento forzado | Mechanism preventing an isolated node from using a resource. |
| Health Check | Comprobación de estado | Test that evaluates component health. |
| Heartbeat | Señal periódica | Message used to indicate that a node is responsive. |
| Availability Zone | Zona de disponibilidad | Isolated infrastructure location within a region. |
| Disaster Recovery | Recuperación ante desastres | Strategy for restoring systems after major disruptions. |
| Observability | Observabilidad | Ability to understand system behavior through telemetry. |

## 27. Connections to Other Notes

### Previous knowledge

- [[01 - Client Server Architecture]]
- [[02 - Database Scalability]]
- [[03 - Replication]]
- [[05 - Sharding]]
- [[05 - Transactions and ACID]]

### Related concepts

- [[07 - Backup and Disaster Recovery]]
- [[01 - PostgreSQL Architecture Overview]]
- [[02 - PostgreSQL Processes]]
- [[03 - PostgreSQL Storage]]
- [[05 - PostgreSQL Performance Fundamentals]]
- [[01 - Database Security Fundamentals]]

### Core relationship

```text
Database Architecture
        |
        v
Replication
        |
        v
High Availability
        |
        +------> Failover
        |
        +------> Monitoring
        |
        +------> Redundancy
        |
        +------> Recovery Objectives
        |
        v
Backup and Disaster Recovery
```

High availability is not a single database feature. It is an architectural capability built from replication, infrastructure redundancy, failure detection, safe promotion, application behavior, monitoring, and operational procedures.

The next document will be:

**`04 - Database Architecture/07 - Backup and Disaster Recovery.md`**

There we will study how to protect historical data, restore databases after accidental changes or corruption, use point-in-time recovery, and design recovery strategies for major infrastructure failures.
