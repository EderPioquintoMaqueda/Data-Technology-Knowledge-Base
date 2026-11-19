
---
title: "Database Architecture Fundamentals"
aliases:
  - Database Architecture
  - Database System Architecture
  - Fundamentals of Database Design

type: learning-note
domain: technology
category: data-engineering
subject: database-architecture

level: beginner-to-intermediate
difficulty: foundational
status: in-progress

created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - database-design
  - data-engineering
  - systems-architecture
  - databases
  - infrastructure
  - fundamentals

concepts:
  - database-architecture
  - architecture-layers
  - application-layer
  - data-access-layer
  - database-layer
  - infrastructure-layer
  - system-components
  - scalability
  - reliability
  - maintainability
  - performance

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - What is a Database System]]"
  - "[[02 - The Data Lifecycle]]"
  - "[[03 - Types of Data]]"
  - "[[04 - OLTP vs OLAP]]"

related:
  - "[[01 - Entities and Attributes]]"
  - "[[04 - Conceptual Logical and Physical Models]]"
  - "[[06 - Schema Design Principles]]"
  - "[[01 - Database Management Systems]]"
  - "[[02 - How a Query is Executed]]"
  - "[[01 - Client Server Architecture]]"
  - "[[02 - Database Scalability]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[01 - Data Architecture Overview]]"
  - "[[01 - SIAPV Data Architecture]]"

tools:
  - PostgreSQL
  - pgAdmin
  - SQL
  - Python
  - Docker
  - Git
  - Visual Studio Code

learning_objectives:
  - Define database architecture.
  - Identify the main components of a database system.
  - Understand the responsibilities of architectural layers.
  - Explain how applications communicate with databases.
  - Understand the relationship between logical and physical architecture.
  - Identify fundamental architecture quality attributes.
  - Evaluate basic architectural decisions.
  - Connect database architecture concepts with a practical application.

review:
  status: pending
  confidence:
  last_reviewed:
---

# Database Architecture Fundamentals

## 1. Introduction

A database is not simply a collection of tables.

In a real organization, databases operate as part of larger information systems that include applications, servers, networks, users, security mechanisms, storage, and data processing components.

All these elements must work together.

**Database Architecture** describes how the components of a database system are organized, how they interact, and how they fulfill operational and business requirements.

Understanding database architecture requires moving beyond individual SQL statements.

Instead of asking only:

- How do I create a table?
- How do I retrieve a record?
- How do I update information?

An architect must also ask:

- Where should the data be stored?
- Which application components can access it?
- How will the system handle simultaneous requests?
- How will information remain consistent?
- What happens if a server fails?
- How will the system grow?
- How will access be controlled?
- How will performance be monitored?

These questions define the architectural perspective.

## 2. What Is Database Architecture?

Database architecture is the structural design of a database environment.

It defines:

- System components.
- Component responsibilities.
- Communication mechanisms.
- Data organization.
- Storage strategies.
- Access mechanisms.
- Security controls.
- Operational requirements.
- Relationships between system components.

A database architecture establishes how information moves from an application to persistent storage and how it is retrieved when needed.

### 2.1 Architecture vs Database Design

These concepts are related, but they are not identical.

**Database Design** focuses primarily on how data is structured.

It includes:

- Tables.
- Columns.
- Data types.
- Relationships.
- Keys.
- Constraints.
- Normalization.

**Database Architecture** considers the broader system.

It includes database design, but also:

- Database servers.
- Applications.
- Network communication.
- Infrastructure.
- Replication.
- Backups.
- Security.
- Performance.
- Availability.
- Scalability.

A useful way to remember the distinction:

> Database design defines how information is organized. Database architecture defines how the complete database environment operates.

## 3. Main Components of a Database Architecture

A database system generally contains several interconnected components.

### 3.1 Users

Users interact with an information system through applications, interfaces, APIs, or administrative tools.

Examples:

- Employees.
- Customers.
- Administrators.
- Developers.
- Data analysts.
- Database administrators.

Different users have different responsibilities and permissions.

### 3.2 Applications

Applications provide the functionality through which users interact with information.

Examples:

- Web applications.
- Desktop applications.
- Mobile applications.
- Enterprise resource planning systems.
- Internal management platforms.

Applications receive requests, validate information, execute business logic, and communicate with databases.

### 3.3 Application Logic

Application logic implements the business rules of the system.

For example, a vehicle management application may enforce rules such as:

- Every vehicle must have a unique inventory number.
- A maintenance record must reference an existing vehicle.
- Only authorized users can modify certain fields.
- A vehicle cannot be assigned to two active locations simultaneously, if that is a defined business rule.

Not every business rule belongs exclusively in application code. Important integrity rules may also be enforced by the database.

### 3.4 Database Management System

The **DBMS (Database Management System)** manages the database.

Its responsibilities include:

- Receiving queries.
- Managing transactions.
- Enforcing constraints.
- Controlling access.
- Managing concurrency.
- Optimizing query execution.
- Reading and writing data.
- Managing database storage.

Examples include:

- PostgreSQL.
- MySQL.
- Microsoft SQL Server.
- Oracle Database.
- IBM Db2.

### 3.5 Storage Infrastructure

Storage infrastructure provides the physical or virtual resources used to persist information.

It may include:

- Solid-state drives.
- Hard disk drives.
- Storage arrays.
- Cloud block storage.
- Network-attached storage.
- Object storage.

Database systems use storage infrastructure to preserve data beyond the lifetime of an individual application process.

### 3.6 Network Infrastructure

Network infrastructure allows components to communicate.

For example:

- A browser communicates with a web server.
- A web server communicates with an application service.
- An application service connects to a database server.
- A database server accesses storage.

Network performance, reliability, and security can influence the overall behavior of the system.

## 4. Architectural Layers

A useful way to understand database architecture is to organize the system into logical layers.

The exact number of layers depends on the design.

For learning purposes, consider this four-layer model.

### Layer 1: Presentation Layer

This is the interface used by people or external systems.

Examples:

- Web pages.
- Forms.
- Dashboards.
- Mobile interfaces.
- Desktop applications.

Its responsibility is to present information and receive user interactions.

### Layer 2: Application Layer

This layer contains the business logic.

Responsibilities include:

- Validating requests.
- Applying business rules.
- Managing workflows.
- Coordinating operations.
- Preparing database requests.
- Handling application-level errors.

### Layer 3: Data Access Layer

This layer manages communication between application logic and the database.

It may contain:

- Database drivers.
- Connection management.
- SQL statements.
- Object-relational mapping tools.
- Repository patterns.
- Data access services.

Its objective is to organize and control how applications interact with persistent data.

### Layer 4: Database Layer

This layer contains the DBMS and database structures.

Responsibilities include:

- Storing information.
- Executing queries.
- Managing transactions.
- Enforcing constraints.
- Managing indexes.
- Controlling concurrent access.

### Simplified Architecture

```text
+----------------------------------+
|         PRESENTATION LAYER       |
|                                  |
|      Web Interface / Dashboard   |
+----------------------------------+
                 |
                 v
+----------------------------------+
|          APPLICATION LAYER       |
|                                  |
|       Business Rules / Logic     |
+----------------------------------+
                 |
                 v
+----------------------------------+
|          DATA ACCESS LAYER       |
|                                  |
|       SQL / Drivers / APIs       |
+----------------------------------+
                 |
                 v
+----------------------------------+
|           DATABASE LAYER         |
|                                  |
|       DBMS / Tables / Indexes     |
+----------------------------------+
                 |
                 v
+----------------------------------+
|         STORAGE INFRASTRUCTURE   |
|                                  |
|        Persistent Storage        |
+----------------------------------+
```

This model is conceptual.

In an actual implementation, some responsibilities may be combined, distributed across services, or implemented through additional layers.

## 5. How Do These Components Communicate?

Consider a user searching for a vehicle using its inventory number.

The process may look like this:

1. The user enters an inventory number.
2. The presentation layer sends a request.
3. The application layer validates the request.
4. The data access layer prepares a database query.
5. The DBMS receives the query.
6. The database engine executes it.
7. The DBMS returns the result.
8. The application processes the response.
9. The presentation layer displays the vehicle information.

Example SQL:

```sql
SELECT
    vehicle_id,
    inventory_number,
    brand,
    model,
    year
FROM vehicles
WHERE inventory_number = 'PK-00139';
```

The SQL statement is only one part of the complete operation.

The application, connection, DBMS, execution engine, storage, and network all participate in the process.

Related topic: [[02 - How a Query is Executed]].

## 6. Three-Schema Architecture

Database theory also describes a three-schema architecture.

This model separates the database into three levels of abstraction.

### 6.1 External Level

The external level describes how individual users or applications view the data.

Different users may require different representations of the same database.

For example:

- A fleet administrator sees vehicle information.
- A maintenance employee sees maintenance records.
- A manager sees aggregated operational information.

Each view exposes the information relevant to a particular role.

### 6.2 Conceptual Level

The conceptual level describes the overall logical structure of the database.

It defines:

- Entities.
- Attributes.
- Relationships.
- Constraints.
- Logical data organization.

It represents the database as a coherent information model without focusing on the physical storage details.

### 6.3 Internal Level

The internal level describes how data is physically stored and accessed.

It includes:

- Storage structures.
- Data pages.
- Indexes.
- File organization.
- Physical access methods.

### Conceptual Representation

```text
             EXTERNAL LEVEL
          User and Application Views
                     |
                     v
             CONCEPTUAL LEVEL
          Logical Database Structure
                     |
                     v
              INTERNAL LEVEL
           Physical Storage Details
                     |
                     v
                DATA STORAGE
```

### 6.4 Data Independence

An important objective of this architecture is data independence.

Data independence means that changes at one level should not necessarily require changes at another level.

Two important forms are:

**Physical Data Independence**

Changes to physical storage should not require changes to the logical database model.

For example, changing an indexing strategy should not normally require rewriting every application query.

**Logical Data Independence**

Changes to the conceptual schema should not necessarily require changes to every external user view.

For example, adding an attribute to a table should not automatically invalidate every application interface.

Perfect independence is difficult to achieve in practice, but it is an important architectural objective.

## 7. Common Database Architecture Models

Database environments can be organized in different ways.

### 7.1 One-Tier Architecture

The application, database, and user interface operate within the same environment.

This approach is common in:

- Local development.
- Small standalone applications.
- Educational environments.
- Some desktop database tools.

It is simple but generally limited for multi-user enterprise systems.

### 7.2 Two-Tier Architecture

The client communicates directly with the database server.

```text
+---------------------+
|    Client           |
|                     |
|  Application Logic  |
|  User Interface     |
+---------------------+
          |
          | Database Connection
          |
          v
+---------------------+
|   Database Server   |
|                     |
|       DBMS          |
|       Data          |
+---------------------+
```

Advantages:

- Relatively simple architecture.
- Direct communication.
- Fewer architectural components.

Limitations:

- Client applications may contain too much business logic.
- Database access must be carefully controlled.
- Scaling large numbers of clients can become difficult.
- Database credentials and connection management require special attention.

### 7.3 Three-Tier Architecture

The client communicates with an application server, which communicates with the database server.

```text
+---------------------+
|   Presentation      |
|      Client         |
+---------------------+
          |
          v
+---------------------+
|   Application      |
|      Server         |
+---------------------+
          |
          v
+---------------------+
|   Database Server   |
|                     |
|       DBMS          |
|       Data          |
+---------------------+
```

Advantages:

- Clear separation of responsibilities.
- Centralized business logic.
- Better control over database access.
- Easier maintenance.
- Greater flexibility for scaling application services.

Three-tier architecture is common in enterprise applications.

### 7.4 N-Tier Architecture

N-tier architecture extends the separation of responsibilities into additional services.

A system may include:

- Web frontend.
- API gateway.
- Authentication service.
- Application services.
- Data access services.
- Database servers.
- Caching systems.
- Messaging infrastructure.
- Analytical platforms.

This approach is useful for complex enterprise environments.

However, more components also introduce additional operational complexity.

## 8. Architectural Quality Attributes

A database architecture should not be evaluated only by whether it works.

It should also be evaluated by how well it satisfies operational requirements.

These requirements are often called **Quality Attributes** or **Non-Functional Requirements**.

### 8.1 Performance

Performance describes how efficiently the system responds to requests.

Factors include:

- Query execution time.
- CPU consumption.
- Memory usage.
- Storage I/O.
- Network latency.
- Index design.

### 8.2 Scalability

Scalability describes the ability of a system to handle increasing workloads.

A system may need to support:

- More users.
- More transactions.
- Larger datasets.
- More complex queries.
- Additional applications.

Related topic: [[02 - Database Scalability]].

### 8.3 Availability

Availability describes the ability of a system to remain accessible when users or applications need it.

A highly available system is designed to reduce service interruptions.

Related topic: [[06 - High Availability]].

### 8.4 Reliability

Reliability describes the ability of a system to operate correctly and consistently over time.

A reliable database environment must protect data integrity and recover appropriately from failures.

### 8.5 Security

Security protects data and system resources against unauthorized access, modification, disclosure, or destruction.

It includes:

- Authentication.
- Authorization.
- Encryption.
- Auditing.
- Access policies.
- Network protection.

Related topic: [[01 - Database Security Fundamentals]].

### 8.6 Maintainability

Maintainability describes how easily a system can be modified, corrected, monitored, and improved.

Good architecture should make it possible to:

- Understand system components.
- Identify problems.
- Introduce changes.
- Upgrade infrastructure.
- Maintain documentation.

### 8.7 Recoverability

Recoverability describes the ability to restore a system and its information after a failure.

It depends on:

- Backups.
- Transaction logs.
- Replication.
- Recovery procedures.
- Disaster recovery planning.

Related topic: [[07 - Backup and Disaster Recovery]].

## 9. Architectural Decisions and Trade-Offs

There is rarely a single architecture that is optimal for every situation.

Architectural decisions involve trade-offs.

For example:

| Decision | Potential Benefit | Potential Cost |
|---|---|---|
| Add indexes | Faster data retrieval | More storage and write overhead |
| Replicate databases | Improved read distribution and redundancy | Replication complexity |
| Partition large tables | Better data organization and targeted access | More design and maintenance complexity |
| Separate application and database servers | Better separation of responsibilities | Additional infrastructure |
| Use a distributed architecture | Horizontal scaling and service independence | More operational complexity |
| Introduce caching | Faster repeated reads | Cache invalidation and consistency challenges |

An architect must understand the business requirements before choosing a solution.

### Example: A Performance Problem

Imagine that a vehicle management application becomes slow when users search for records.

A poor approach would be to immediately migrate the entire database to a different technology.

A better process would be:

1. Identify the affected operation.
2. Measure query execution time.
3. Inspect the execution plan.
4. Review table size and indexes.
5. Evaluate application and network latency.
6. Identify the actual bottleneck.
7. Propose an appropriate solution.
8. Test and measure the result.

Architecture requires evidence-based decisions.

## 10. Architecture Requirements

Before designing a database environment, requirements must be identified.

### Functional Requirements

Functional requirements describe what the system must do.

Examples:

- Register vehicles.
- Update inventory information.
- Search for equipment.
- Record maintenance activities.
- Generate operational reports.
- Export information.

### Non-Functional Requirements

Non-functional requirements describe how the system should operate.

Examples:

- Support 500 concurrent users.
- Respond to common searches within two seconds.
- Maintain defined availability targets.
- Protect sensitive information.
- Recover data after infrastructure failure.
- Support future growth.

The numerical examples are illustrative requirements, not universal standards.

### Why This Distinction Matters

A system may satisfy all functional requirements but still fail operationally.

For example, an application may correctly register vehicles but take 30 seconds to retrieve a record.

It works functionally, but its performance may be unacceptable.

Architecture must consider both types of requirements.

## 11. Applying Architecture Thinking to SIAPV

SIAPV is a useful practical reference for understanding these concepts.

Its known components include:

- PostgreSQL as the database engine.
- Database administration through pgAdmin.
- SQL for database operations.
- An application interface built with HTML and related technologies.
- Application functionality for inventory, searches, records, and reports.

The important exercise is to understand the responsibilities of each component.

### 11.1 Simplified Logical Architecture

```text
             SIAPV USER
                  |
                  v
          APPLICATION INTERFACE
                  |
                  v
          APPLICATION LOGIC
                  |
                  v
            DATABASE ACCESS
                  |
                  v
              POSTGRESQL
                  |
                  v
             DATA STORAGE
```

This is a conceptual representation. The exact deployed arrangement should be documented separately rather than inferred from the technology list.

### 11.2 Architecture Questions

When evaluating SIAPV, an architect should ask:

**Data Organization**

- Are vehicle records modeled consistently?
- Are relationships between vehicles and maintenance records defined?
- Are unique identifiers enforced?
- Are duplicate records prevented?

**Application Design**

- Where are business rules implemented?
- How does the application validate input?
- How are database connections managed?
- How are errors handled?

**Security**

- Which users can access the system?
- Are permissions assigned according to responsibilities?
- Are database credentials protected?
- Are administrative operations restricted?

**Performance**

- Which queries are executed most frequently?
- Are appropriate indexes available?
- Are slow queries monitored?
- How does the application behave as records increase?

**Reliability**

- Are backups performed?
- Have restoration procedures been tested?
- What happens if the database server becomes unavailable?
- How is data loss prevented?

**Scalability**

- What happens if the number of vehicles increases?
- What happens if more users access the application simultaneously?
- Would reporting workloads affect operational queries?

### 11.3 The Architect's Perspective

The objective is not to assume that SIAPV needs a complex enterprise architecture.

The objective is to identify its requirements, understand its current structure, evaluate limitations, and justify possible improvements.

A simple architecture can be an excellent solution when it satisfies the actual requirements.

Complexity should be introduced only when it provides measurable value.

## 12. A Practical Architecture Evaluation Framework

When reviewing an existing database system, use the following sequence.

### Step 1: Understand the Business

Identify:

- What problem does the system solve?
- Who uses it?
- What decisions depend on its information?
- Which operations are critical?

### Step 2: Identify the Workload

Determine:

- Number of users.
- Transaction frequency.
- Read/write ratio.
- Data volume.
- Query complexity.
- Peak usage periods.

### Step 3: Understand the Data Model

Review:

- Entities.
- Attributes.
- Relationships.
- Primary keys.
- Foreign keys.
- Constraints.
- Normalization.

### Step 4: Understand the Technical Architecture

Identify:

- Application components.
- Database engine.
- Network connections.
- Storage infrastructure.
- External integrations.
- Deployment environment.

### Step 5: Evaluate Quality Attributes

Review:

- Performance.
- Availability.
- Reliability.
- Security.
- Scalability.
- Maintainability.
- Recoverability.

### Step 6: Identify Risks and Opportunities

Document:

- Existing limitations.
- Potential failure points.
- Security weaknesses.
- Performance bottlenecks.
- Missing documentation.
- Possible improvements.

### Step 7: Propose and Validate Changes

Every recommendation should include:

- The problem being addressed.
- The proposed solution.
- Expected benefits.
- Possible disadvantages.
- Implementation complexity.
- Validation criteria.

## 13. Technical Vocabulary

| Term | Definition |
|---|---|
| Database Architecture | Structural organization of a database environment |
| DBMS | Software that manages databases |
| Application Layer | Layer responsible for business logic |
| Data Access Layer | Component responsible for database communication |
| Presentation Layer | User-facing interface |
| Database Server | Server that hosts and operates database services |
| Schema | Logical organization of database objects |
| Data Independence | Ability to change one schema level without necessarily changing another |
| Scalability | Ability to accommodate increasing workloads |
| Availability | Ability to remain accessible |
| Reliability | Ability to operate correctly over time |
| Maintainability | Ease of modifying and supporting a system |
| Latency | Time required for an operation to receive a response |
| Throughput | Amount of work processed during a given period |
| Bottleneck | Component that limits overall system performance |
| Trade-Off | Compromise between competing design objectives |
| Functional Requirement | Description of what a system must do |
| Non-Functional Requirement | Description of how a system must operate |

## 14. Knowledge Check

Answer these questions without reviewing the explanations.

1. What is database architecture?
2. How does database architecture differ from database design?
3. What are the main components of a database system?
4. What is the responsibility of the application layer?
5. What is the purpose of the data access layer?
6. What are the three levels of the three-schema architecture?
7. What is physical data independence?
8. How does two-tier architecture differ from three-tier architecture?
9. What is the difference between scalability and availability?
10. Why should architectural decisions consider trade-offs?
11. What is the difference between functional and non-functional requirements?
12. Why should an architect measure a performance problem before changing the infrastructure?

## 15. Practical Exercise

### Exercise A: Design a Basic Database Architecture

Imagine that an organization needs an application to manage:

- Vehicles.
- Employees.
- Departments.
- Maintenance records.
- Equipment assignments.

Propose a basic architecture.

Your design must identify:

1. The presentation layer.
2. The application layer.
3. The data access layer.
4. The database layer.
5. The storage infrastructure.

Explain the responsibility of each component.

### Exercise B: Evaluate Architectural Requirements

Consider the following requirements:

| Requirement | Classification |
|---|---|
| Register a new vehicle | |
| Support 300 concurrent users | |
| Generate maintenance reports | |
| Protect administrative operations | |
| Respond to searches within two seconds | |
| Restore information after a database failure | |
| Export vehicle information | |

Classify each requirement as functional or non-functional.

### Exercise C: Architecture Review

Imagine that an application has the following problems:

- Searches are slow.
- Several users overwrite information.
- Database credentials are shared.
- Backups exist but have never been restored in a test.
- Monthly reports affect application performance.

For each problem:

1. Identify the architectural area involved.
2. Explain the potential cause.
3. Propose an initial investigation.
4. Recommend a possible improvement.
5. Explain how you would validate the result.

Do not assume that every problem has only one solution.

## 16. Key Takeaways

- Database architecture defines how a complete database environment is organized and operates.
- Database design focuses primarily on data structures and relationships.
- Applications, business logic, data access, DBMS software, networks, and storage are interconnected components.
- Architectural layers separate responsibilities.
- The three-schema architecture distinguishes external, conceptual, and internal representations.
- Two-tier, three-tier, and N-tier models describe different ways of organizing system components.
- Performance, scalability, availability, reliability, security, maintainability, and recoverability are essential quality attributes.
- Functional and non-functional requirements must both be considered.
- Architectural decisions involve trade-offs.
- Good architecture begins with understanding the business and measuring actual requirements.
- Complexity is not automatically a sign of good architecture.

## 17. Related Notes

### Data Foundations
- [[00 - Database Architecture Roadmap]]
- [[01 - What is a Database System]]
- [[02 - The Data Lifecycle]]
- [[03 - Types of Data]]
- [[04 - OLTP vs OLAP]]

### Data Modeling
- [[01 - Entities and Attributes]]
- [[02 - Relationships and Cardinality]]
- [[03 - Keys and Constraints]]
- [[04 - Conceptual Logical and Physical Models]]
- [[05 - Normalization]]
- [[06 - Schema Design Principles]]

### Database Internals
- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[03 - Storage Pages and Tables]]
- [[04 - Indexes]]
- [[05 - Transactions and ACID]]
- [[06 - Concurrency and Locks]]

### Database Architecture
- [[01 - Client Server Architecture]]
- [[02 - Database Scalability]]
- [[03 - Replication]]
- [[04 - Partitioning]]
- [[05 - Sharding]]
- [[06 - High Availability]]
- [[07 - Backup and Disaster Recovery]]

### Applied Architecture
- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]
- [[03 - Architecture Decision Framework]]

---

**Learning Objective:** Develop a system-level understanding of database architecture, identify the responsibilities of its components, evaluate architectural requirements, and justify design decisions based on business and technical needs.
