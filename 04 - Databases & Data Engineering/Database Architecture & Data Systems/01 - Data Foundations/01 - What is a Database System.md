
---
title: "What is a Database System?"
aliases:
  - Database System Fundamentals
  - Introduction to Database Systems
  - Database Systems Overview

type: learning-note
domain: technology
category: data-engineering
subject: database-architecture

level: beginner
difficulty: foundational
status: in-progress

created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - database-systems
  - dbms
  - data-engineering
  - sql
  - fundamentals
  - learning-path

concepts:
  - database
  - dbms
  - database-system
  - data-management
  - data-persistence

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"

related:
  - "[[02 - The Data Lifecycle]]"
  - "[[03 - Types of Data]]"
  - "[[04 - OLTP vs OLAP]]"
  - "[[05 - Database Architecture Fundamentals]]"

tools:
  - PostgreSQL
  - SQL

learning_objectives:
  - Understand what a database is.
  - Understand what a DBMS is.
  - Differentiate a database from a database system.
  - Identify the main components of a database system.
  - Understand how applications interact with databases.
  - Recognize the role of databases in information systems.

review:
  status: pending
  confidence:
  last_reviewed:
---

# What is a Database System?

## 1. Introduction

Modern applications generate, process, store, and retrieve enormous amounts of information.

Every time a user logs into an application, makes a purchase, searches for a product, registers a vehicle, or updates a personal record, information may need to be stored and managed.

This creates an important technological requirement:

**How can information be organized, stored, accessed, modified, and protected in a reliable way?**

Database systems exist to solve this problem.

A database system provides the mechanisms required to manage information throughout its lifecycle, ensuring that applications and users can interact with data in a structured and controlled manner.

Understanding database systems is the first step toward understanding database architecture.

## 2. What is a Database?

A **Database** is an organized collection of data that can be stored, accessed, managed, and updated.

Its purpose is to represent information in a structured way so that it can be retrieved and used efficiently.

For example, imagine an organization responsible for managing a vehicle fleet.

It may need to store information such as:

- Vehicle identification numbers.
- License plates.
- Vehicle brands and models.
- Manufacturing years.
- Assigned departments.
- Maintenance records.
- Mileage.
- Insurance information.

Instead of keeping this information in disconnected documents, a database allows it to be organized into related structures.

A simplified example:

| Vehicle ID | Brand | Model | Year |
|---|---|---|---|
| 1001 | Nissan | Frontier | 2024 |
| 1002 | Ford | Ranger | 2023 |
| 1003 | Toyota | Hilux | 2025 |

This table represents a small collection of structured information.

However, a real database can contain multiple tables, relationships, constraints, indexes, and other structures.

### Key idea

A database is not simply a spreadsheet or a collection of files.

It is a structured data repository designed to support controlled and consistent data operations.

## 3. What is a DBMS?

A **Database Management System (DBMS)** is software designed to create, manage, access, and maintain databases.

The database stores the information, while the DBMS provides the mechanisms used to interact with and manage that information.

A DBMS typically provides:

- Database creation and management.
- Data insertion, modification, and deletion.
- Data retrieval through queries.
- Access control and permissions.
- Transaction management.
- Data integrity enforcement.
- Backup and recovery mechanisms.
- Concurrency management.

For example, PostgreSQL is a DBMS.

Other examples include:

| DBMS | Developer / Organization |
|---|---|
| PostgreSQL | PostgreSQL Global Development Group |
| MySQL | Oracle |
| Microsoft SQL Server | Microsoft |
| Oracle Database | Oracle |
| IBM Db2 | IBM |
| SQLite | SQLite Consortium |

### Important distinction

SQL is a language used to communicate with relational database systems.

PostgreSQL is a DBMS that can interpret and execute SQL instructions.

They are not the same thing.

## 4. What is a Database System?

A **Database System** is the complete environment responsible for managing and interacting with stored information.

It includes more than the database itself.

A database system generally involves:

1. The database: the stored information and its structures.
2. The DBMS: the software that manages the database.
3. Applications: software that interacts with the database.
4. Users: people or systems that request information.
5. Infrastructure: hardware, operating systems, storage, and network resources.
6. Procedures: operational rules, security policies, backup strategies, and maintenance processes.

The exact components depend on the architecture.

### Database vs. DBMS vs. Database System

| Concept | Meaning |
|---|---|
| Database | Organized collection of data |
| DBMS | Software that manages the database |
| Database System | Complete environment for managing and accessing data |

This distinction is fundamental when discussing database architecture.

## 5. Main Components of a Database System

### 5.1 Data

Data represents the information stored and processed by the system.

Examples:

- Customer records.
- Product information.
- Financial transactions.
- Vehicle inventories.
- Maintenance histories.

Data is the central resource managed by the database system.

### 5.2 Database Management System

The DBMS controls how data is stored, accessed, modified, and protected.

It acts as the intermediary between applications and the stored information.

### 5.3 Applications

Applications provide interfaces through which users and other systems interact with information.

Examples:

- Web applications.
- Mobile applications.
- Enterprise software.
- Internal management platforms.
- Data analysis tools.

An application usually sends requests to the DBMS instead of directly modifying database files.

### 5.4 Users

Different users may interact with the system according to their responsibilities.

Examples:

- End users.
- Application developers.
- Database administrators (DBAs).
- Data engineers.
- System administrators.
- Data analysts.

Each role may require different permissions and responsibilities.

### 5.5 Infrastructure

Infrastructure provides the computing resources required to operate the database system.

It may include:

- Servers.
- CPU and RAM.
- Storage devices.
- Operating systems.
- Network connections.
- Virtual machines.
- Cloud infrastructure.

The infrastructure influences database performance, availability, and scalability.

## 6. How Does a Database System Work?

Consider a user who wants to retrieve information about a vehicle.

The interaction can be represented as follows:

User
  ↓
Application Interface
  ↓
Application Logic
  ↓
Database Driver / Connection
  ↓
DBMS
  ↓
Database Storage

The process generally follows these steps:

1. The user requests information through an application.
2. The application receives the request.
3. The application processes the request and prepares a database operation.
4. The database driver sends the operation to the DBMS.
5. The DBMS interprets and processes the request.
6. The DBMS accesses the required data.
7. The result is returned to the application.
8. The application presents the information to the user.

This is a simplified representation. Actual database architectures may include additional services, network layers, caches, authentication mechanisms, and distributed components.

### Example using SQL

Suppose a database contains a table called `vehicles`.

A user wants to retrieve the information of vehicle 1001.

```sql
SELECT *
FROM vehicles
WHERE vehicle_id = 1001;
```

The SQL statement expresses the requested operation.

The DBMS is responsible for processing the query and returning the matching record.

The application then uses the returned information.

**Important:** Writing a SQL query is only one part of the process. Understanding how the DBMS interprets and executes that query is a separate subject that we will study later.

Related topic: [[02 - How a Query is Executed]].

## 7. Why Do We Need Database Systems?

Database systems solve several problems associated with managing information.

### 7.1 Data Organization

They provide structures for organizing information into tables, records, documents, or other data models.

### 7.2 Data Consistency

They help ensure that information follows defined rules and remains consistent.

For example, a vehicle record should not reference a nonexistent department if the system enforces that relationship.

### 7.3 Data Integrity

Integrity mechanisms help prevent invalid or inconsistent information from entering the database.

Examples:

- Primary keys.
- Foreign keys.
- Unique constraints.
- NOT NULL constraints.
- CHECK constraints.

### 7.4 Data Security

Database systems provide mechanisms for controlling who can access information and what operations they can perform.

### 7.5 Concurrent Access

Multiple users and applications may access the same database simultaneously.

The DBMS coordinates these operations to preserve consistency.

### 7.6 Data Recovery

Databases can use backup and recovery mechanisms to restore information after failures or accidental data loss.

### 7.7 Performance

Database systems provide mechanisms such as indexing and query optimization to improve data retrieval and processing.

## 8. Database System vs. File-Based Systems

Before database systems became common, many applications stored information in independent files.

Although files remain useful, managing complex information through disconnected files introduces several challenges.

| Characteristic | File-Based System | Database System |
|---|---|---|
| Organization | Independent files | Managed data structures |
| Relationships | Usually managed manually | Can be formally defined |
| Data integrity | Application-dependent | DBMS-supported |
| Concurrent access | More difficult to coordinate | Transaction mechanisms |
| Security | File-level controls | Database-level permissions |
| Querying | Requires custom processing | Query languages such as SQL |
| Recovery | Often application-dependent | Backup and recovery mechanisms |

A database system does not automatically eliminate every problem, but it provides specialized mechanisms for managing information more reliably.

## 9. Practical Example: SIAPV

SIAPV is a useful example of a database system because it combines application logic, PostgreSQL, and structured information.

Its main components can be understood as follows:

| Component | SIAPV Example |
|---|---|
| Database | PostgreSQL database |
| DBMS | PostgreSQL |
| Application | SIAPV interface |
| Application logic | Backend operations and application code |
| Data | Vehicle and machinery records |
| Infrastructure | Server environment and database storage |
| Users | Authorized system users |

This example helps distinguish between the application and the database.

SIAPV is not PostgreSQL.

PostgreSQL is the DBMS used by SIAPV to manage its information.

The application provides the functionality through which users interact with that information.

### Architectural observation

When designing a database system, it is important to understand the responsibilities of each component.

An application should not be confused with its database, and a database should not be confused with the DBMS managing it.

This separation is essential for understanding larger technological systems.

## 10. Key Technical Vocabulary

| English Term | Meaning |
|---|---|
| Database | Base de datos |
| DBMS | Sistema gestor de bases de datos |
| Database System | Sistema de bases de datos |
| Data Storage | Almacenamiento de datos |
| Data Retrieval | Recuperación de datos |
| Data Manipulation | Manipulación de datos |
| Data Integrity | Integridad de datos |
| Data Consistency | Consistencia de datos |
| Query | Consulta |
| Transaction | Transacción |
| Database Administrator | Administrador de bases de datos |
| Application Layer | Capa de aplicación |
| Infrastructure | Infraestructura |
| Concurrent Access | Acceso concurrente |
| Data Persistence | Persistencia de datos |

## 11. Knowledge Check

Try to answer these questions without looking at the previous sections.

1. What is the difference between a database and a DBMS?
2. Why is a database system more than a database?
3. What responsibilities does a DBMS have?
4. How does an application communicate with a database?
5. Why is data integrity important?
6. What happens when multiple users access the same database?
7. Why should application logic and database management be understood as separate responsibilities?

## 12. Practical Exercise

### Exercise: Identify the Components of a Database System

Imagine that you are designing a system to manage a company's vehicle fleet.

Identify the following components:

- What information needs to be stored?
- What database structures would be necessary?
- Which DBMS could manage that information?
- What application would users interact with?
- What infrastructure would be required?
- Which users would need access?
- What security and integrity mechanisms would be necessary?

Do not worry about designing the complete database yet.

The objective is to recognize the different components of the system.

## 13. Key Takeaways

- A database is an organized collection of information.
- A DBMS is software that manages databases.
- A database system is the complete environment used to manage and access information.
- Applications communicate with databases through defined interfaces and database connections.
- The DBMS manages operations such as querying, transactions, security, and data integrity.
- Infrastructure provides the resources needed to operate the system.
- Understanding the distinction between these components is essential for database architecture.

## 14. Related Notes

### Previous Knowledge
- [[00 - Database Architecture Roadmap]]

### Next Topics
- [[02 - The Data Lifecycle]]
- [[03 - Types of Data]]
- [[04 - OLTP vs OLAP]]
- [[05 - Database Architecture Fundamentals]]

### Advanced Topics
- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[01 - PostgreSQL Architecture Overview]]

---

**Learning Objective:** Understand the fundamental components of a database system and explain how applications, DBMS software, stored data, users, and infrastructure interact.
