
---
title: "Conceptual, Logical, and Physical Data Models"
aliases:
  - "Three Levels of Data Modeling"
  - "Conceptual Data Model"
  - "Logical Data Model"
  - "Physical Data Model"
  - "Data Modeling Levels"
type: learning-note
domain: technology
category: data-engineering
subject: data-modeling
level: beginner-to-intermediate
difficulty: foundational
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - database-architecture
  - data-modeling
  - conceptual-model
  - logical-model
  - physical-model
  - relational-databases
  - database-design
  - postgresql
concepts:
  - conceptual-data-model
  - logical-data-model
  - physical-data-model
  - entity-relationship-diagram
  - relational-schema
  - database-schema
  - data-abstraction
  - data-model-transformation
  - physical-storage
  - design-constraints
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Entities and Attributes]]"
  - "[[02 - Relationships and Cardinality]]"
  - "[[03 - Keys and Constraints]]"
related:
  - "[[05 - Normalization]]"
  - "[[06 - Schema Design Principles]]"
  - "[[01 - Database Management Systems]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[01 - SIAPV Data Architecture]]"
  - "[[02 - SIAPV Database Design Review]]"
tools:
  - Entity Relationship Diagrams
  - SQL
  - PostgreSQL
  - pgAdmin
  - Database Modeling Tools
learning_objectives:
  - "Distinguish conceptual, logical, and physical data models."
  - "Understand how business requirements become database structures."
  - "Identify the level of abstraction used by each model."
  - "Understand entities, attributes, relationships, keys, and implementation details across modeling levels."
  - "Transform a conceptual model into a logical relational model."
  - "Transform a logical model into a physical PostgreSQL schema."
  - "Recognize the importance of documentation and traceability in database design."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 04 — Conceptual, Logical, and Physical Data Models

## 1. Introduction

Database design is not simply the process of creating tables.

Before implementing a database, architects and developers need to understand the business problem, identify the information that must be managed, define relationships, and determine how the information will be stored.

A structured design process commonly uses three modeling levels:

1. **Conceptual Data Model**
2. **Logical Data Model**
3. **Physical Data Model**

Each level describes the same information system from a different perspective.

The main difference is the level of technical detail.

### The central idea

A business requirement should not be translated directly into SQL without first understanding the data and its relationships.

A model provides a structured representation of the information before implementation.

---

# 2. The Three Levels of Data Modeling

Consider an organization that needs a system to manage vehicles, employees, departments, and maintenance activities.

The organization has a business requirement:

> The system must register vehicles, assign them to departments, record maintenance activities, and identify the employees involved in each activity.

This requirement can be represented at three different levels.

## 2.1 Conceptual Data Model

The conceptual model describes the business information at a high level.

It answers questions such as:

- What information does the organization need?
- Which business entities exist?
- How are those entities related?
- What concepts are important to the business?

At this level, technical implementation details are usually excluded.

Example:

- A vehicle belongs to a department.
- A vehicle can have maintenance activities.
- Employees participate in maintenance activities.
- A department manages employees.

The conceptual model focuses on business meaning.

## 2.2 Logical Data Model

The logical model describes the structure of the data in greater detail.

It defines:

- Entities.
- Attributes.
- Primary keys.
- Foreign keys.
- Relationships.
- Cardinality.
- Data integrity requirements.
- Normalization rules.

The logical model is more precise than the conceptual model.

However, it is generally independent of a specific database management system.

For example, the model can describe a vehicle identifier as a primary key without deciding whether PostgreSQL will implement it using `BIGINT`, an identity column, or another physical mechanism.

## 2.3 Physical Data Model

The physical model describes how the database will actually be implemented in a specific DBMS.

It includes:

- Tables.
- Columns.
- Data types.
- Indexes.
- Constraints.
- Sequences or identity generation.
- Storage considerations.
- Partitioning decisions.
- Database-specific implementation details.

For example:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    inventory_number VARCHAR(30) NOT NULL UNIQUE,
    brand VARCHAR(80) NOT NULL,
    model VARCHAR(80) NOT NULL
);
```

This is a physical implementation for PostgreSQL.

### Comparison

| Characteristic | Conceptual | Logical | Physical |
|---|---|---|---|
| Main focus | Business meaning | Data structure | Technical implementation |
| Audience | Business stakeholders and architects | Analysts, designers, developers | DBAs, developers, architects |
| Entities | Yes | Yes | Represented as tables |
| Attributes | High-level | Detailed | Columns |
| Relationships | Business relationships | Cardinality and keys | Foreign keys and constraints |
| Data types | Usually omitted | Logical types may be described | DBMS-specific types |
| Indexes | No | Usually no | Yes |
| DBMS dependency | No | Usually no | Yes |

### Important distinction

These models are not three separate databases.

They are three levels of abstraction describing the same information requirements.

---

# 3. Level of Abstraction

**Abstraction** means representing essential characteristics while hiding details that are not relevant at a particular level.

In database modeling, abstraction allows different stakeholders to understand the same system without requiring everyone to understand SQL or storage mechanisms.

Consider the concept of a vehicle.

At each level:

**Conceptual:**

A vehicle is an asset managed by the organization.

**Logical:**

A vehicle has an identifier, inventory number, brand, model, acquisition date, and relationship with a department.

**Physical:**

The vehicle is implemented as a PostgreSQL table with columns such as:

```sql
vehicle_id BIGINT
inventory_number VARCHAR(30)
brand VARCHAR(80)
model VARCHAR(80)
acquisition_date DATE
department_id BIGINT
```

The business concept remains the same.

The representation becomes progressively more technical.

### Why abstraction matters

It helps teams:

- Separate business decisions from technical decisions.
- Communicate with different stakeholders.
- Detect missing requirements before implementation.
- Evaluate alternative database technologies.
- Reduce unnecessary complexity.
- Maintain traceability between requirements and implementation.

---

# 4. Conceptual Data Modeling

## 4.1 Purpose

The conceptual model establishes the main information concepts of a system.

It is concerned with the business domain rather than the technical structure.

A conceptual model should answer:

1. What entities exist?
2. What do they represent?
3. How are they related?
4. What business rules govern those relationships?

## 4.2 Identifying entities

Consider a vehicle management system.

Potential entities include:

- Vehicle.
- Department.
- Employee.
- Maintenance.
- Workshop.

Each entity represents an identifiable concept in the business domain.

An entity should not be created simply because a word appears in a requirement.

It should represent something the system needs to identify, manage, or relate to other information.

## 4.3 Identifying relationships

Possible business relationships:

- A department manages multiple vehicles.
- A department has multiple employees.
- A vehicle has multiple maintenance activities.
- Employees participate in maintenance activities.

These relationships establish how the entities interact.

## 4.4 Conceptual representation

A simplified conceptual model can be represented as:

```text
DEPARTMENT
    |
    | manages
    |
    +------ VEHICLE
    |
    +------ EMPLOYEE

VEHICLE
    |
    | receives
    |
    +------ MAINTENANCE

EMPLOYEE
    |
    | participates in
    |
    +------ MAINTENANCE
```

This representation communicates the business concepts.

It does not yet define PostgreSQL tables, data types, or indexes.

## 4.5 Business rules

A business rule is a statement that defines how an organization operates or how its information must behave.

Examples:

- Every registered vehicle must have an inventory identifier.
- A vehicle must be associated with an organizational department.
- A maintenance activity must refer to a registered vehicle.
- An employee can participate in multiple maintenance activities.
- A maintenance activity can involve multiple employees.

Business rules guide the conceptual and logical models.

### Deliverables of conceptual modeling

Common deliverables include:

- Conceptual entity-relationship diagram.
- Entity descriptions.
- Relationship descriptions.
- Business rules.
- Scope and assumptions.

---

# 5. Logical Data Modeling

## 5.1 Purpose

The logical model translates business concepts into a precise data structure.

It introduces the elements needed to represent the information consistently.

A logical model usually includes:

- Entities.
- Attributes.
- Primary keys.
- Candidate keys.
- Foreign keys.
- Relationships.
- Cardinality.
- Optionality.
- Integrity rules.
- Normalization.

It does not necessarily specify the exact SQL syntax or DBMS-specific implementation.

## 5.2 Transforming entities into logical structures

Consider the conceptual entity:

**Vehicle**

At the logical level, it becomes:

```text
VEHICLE

- Vehicle ID
- Inventory Number
- Brand
- Model
- Acquisition Date
- Department ID
```

Now the designer can identify:

- The primary identifier.
- Required attributes.
- Business identifiers.
- The relationship with departments.

## 5.3 Defining keys

A possible logical design:

```text
VEHICLE

Primary Key:
    Vehicle ID

Candidate Key:
    Inventory Number

Foreign Key:
    Department ID -> DEPARTMENT.Department ID
```

This describes the identity and relationship rules without committing to a particular SQL data type.

## 5.4 Defining relationships

Consider:

```text
DEPARTMENT 1 -------- N VEHICLE
```

This means:

- One department can be associated with multiple vehicles.
- Each vehicle belongs to one department in this simplified model.

At the logical level, the relationship is expressed through cardinality and key references.

## 5.5 Resolving many-to-many relationships

Suppose employees participate in maintenance activities.

The relationship is:

```text
EMPLOYEE N -------- N MAINTENANCE
```

A relational database generally resolves this many-to-many relationship through an associative entity.

Example:

```text
MAINTENANCE_EMPLOYEE

- Maintenance ID
- Employee ID
- Role
```

The logical model now contains two one-to-many relationships:

```text
MAINTENANCE 1 -------- N MAINTENANCE_EMPLOYEE

EMPLOYEE 1 -------- N MAINTENANCE_EMPLOYEE
```

This structure prepares the model for relational implementation.

## 5.6 Logical model example

```text
DEPARTMENT
----------
Department ID (PK)
Department Name

EMPLOYEE
--------
Employee ID (PK)
Employee Code (AK)
Full Name
Email
Department ID (FK)

VEHICLE
-------
Vehicle ID (PK)
Inventory Number (AK)
Brand
Model
Acquisition Date
Department ID (FK)

MAINTENANCE
-----------
Maintenance ID (PK)
Vehicle ID (FK)
Maintenance Date
Description
Status

MAINTENANCE_EMPLOYEE
--------------------
Maintenance ID (PK, FK)
Employee ID (PK, FK)
Role
```

Legend:

- PK = Primary Key.
- FK = Foreign Key.
- AK = Alternate Key.

This is a logical representation of the relational structure.

It is not yet a complete physical schema.

### Deliverables of logical modeling

Common deliverables include:

- Logical entity-relationship diagram.
- Attribute definitions.
- Primary and foreign key definitions.
- Cardinality and optionality.
- Data dictionary.
- Normalized relational schema.
- Integrity requirements.

---

# 6. Physical Data Modeling

## 6.1 Purpose

The physical model translates the logical structure into an implementation for a particular database management system.

For example:

- PostgreSQL.
- MySQL.
- Microsoft SQL Server.
- Oracle Database.
- IBM Db2.

The physical design considers the capabilities and requirements of the selected DBMS.

## 6.2 Logical attributes become physical columns

Logical attribute:

```text
Vehicle ID
```

Possible PostgreSQL implementation:

```sql
vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY
```

Logical attribute:

```text
Inventory Number
```

Possible PostgreSQL implementation:

```sql
inventory_number VARCHAR(30) NOT NULL UNIQUE
```

Logical attribute:

```text
Acquisition Date
```

Possible PostgreSQL implementation:

```sql
acquisition_date DATE
```

The physical model must select appropriate data types and constraints.

## 6.3 Physical implementation

Example:

```sql
CREATE TABLE departments (
    department_id BIGINT GENERATED ALWAYS AS IDENTITY,

    department_name VARCHAR(100) NOT NULL,

    CONSTRAINT pk_departments
        PRIMARY KEY (department_id),

    CONSTRAINT uq_departments_name
        UNIQUE (department_name)
);
```

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY,

    inventory_number VARCHAR(30) NOT NULL,

    brand VARCHAR(80) NOT NULL,

    model VARCHAR(80) NOT NULL,

    acquisition_date DATE,

    department_id BIGINT NOT NULL,

    CONSTRAINT pk_vehicles
        PRIMARY KEY (vehicle_id),

    CONSTRAINT uq_vehicles_inventory
        UNIQUE (inventory_number),

    CONSTRAINT fk_vehicles_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE RESTRICT
);
```

The physical model includes the exact implementation choices:

- `BIGINT`.
- `VARCHAR(100)`.
- `DATE`.
- Identity generation.
- Primary key.
- Unique constraint.
- Foreign key.
- Referential action.

## 6.4 Physical indexes

Physical modeling also evaluates indexing requirements.

Example:

```sql
CREATE INDEX idx_vehicles_department_id
ON vehicles(department_id);
```

This index may improve queries that filter or join vehicles by department.

However, indexes should be selected according to workload and query patterns.

## 6.5 Physical storage considerations

Depending on the system, physical modeling may also consider:

- Tablespaces.
- Storage parameters.
- Partitioning.
- Clustering strategies.
- Compression.
- Index types.
- Data distribution.
- Archival policies.

These decisions become increasingly important as databases grow.

### Deliverables of physical modeling

Common deliverables include:

- Physical database diagram.
- Table definitions.
- Column definitions.
- SQL DDL scripts.
- Constraints.
- Index definitions.
- Storage configuration.
- DBMS-specific implementation documentation.

---

# 7. How the Three Models Connect

The three modeling levels form a progression.

```text
BUSINESS REQUIREMENTS
        |
        v
CONCEPTUAL MODEL
        |
        | Identify business entities
        | Define business relationships
        | Establish business rules
        |
        v
LOGICAL MODEL
        |
        | Define attributes
        | Define keys
        | Define cardinality
        | Resolve many-to-many relationships
        | Normalize structures
        |
        v
PHYSICAL MODEL
        |
        | Select data types
        | Create tables
        | Implement constraints
        | Define indexes
        | Configure storage
        |
        v
DATABASE IMPLEMENTATION
```

Each stage adds precision.

The conceptual model explains what the business needs.

The logical model explains how the information is structured.

The physical model explains how the selected DBMS implements that structure.

## 7.1 Example transformation

| Business requirement | Conceptual model | Logical model | Physical model |
|---|---|---|---|
| Register vehicles | Vehicle | Vehicle entity and attributes | `vehicles` table |
| Identify each vehicle | Vehicle identity | Primary key | `vehicle_id BIGINT` |
| Prevent duplicate inventory numbers | Unique business identifier | Alternate key | `UNIQUE` constraint |
| Assign vehicles to departments | Department manages vehicles | 1:N relationship | Foreign key |
| Store acquisition date | Vehicle acquisition | Date attribute | `DATE` column |
| Prevent negative mileage | Valid mileage | Domain rule | `CHECK (mileage >= 0)` |

This table illustrates how a single business requirement can generate decisions at all three levels.

---

# 8. Data Dictionary

A **data dictionary** is documentation describing the elements of a database.

It commonly includes:

- Table names.
- Column names.
- Definitions.
- Data types.
- Allowed values.
- Nullability.
- Keys.
- Constraints.
- Relationships.
- Business meaning.

It helps developers, analysts, architects, and administrators understand the database consistently.

## 8.1 Example data dictionary

| Attribute | Data type | Required | Description |
|---|---|---|---|
| vehicle_id | BIGINT | Yes | Internal vehicle identifier |
| inventory_number | VARCHAR(30) | Yes | Official inventory identifier |
| brand | VARCHAR(80) | Yes | Vehicle manufacturer |
| model | VARCHAR(80) | Yes | Vehicle model |
| acquisition_date | DATE | No | Date of acquisition |
| department_id | BIGINT | Yes | Department responsible for the vehicle |

This documentation is particularly useful when:

- Multiple developers work on the same database.
- The database is integrated with external systems.
- Requirements change.
- New applications need access to existing data.
- The architecture must be reviewed or audited.

### Important distinction

A data dictionary documents the database structure and meaning.

It does not replace the actual schema or enforce constraints.

The database schema defines the implemented structure; the data dictionary explains it.

---

# 9. Schema vs Data Model

These concepts are related but not identical.

## 9.1 Data model

A data model is an abstract representation of how information is organized and related.

It can exist independently of a particular DBMS.

Examples:

- Conceptual data model.
- Logical relational model.
- Entity-relationship model.

## 9.2 Database schema

A database schema is the formal structure defined in a database.

In PostgreSQL, a schema is also a namespace that organizes database objects such as:

- Tables.
- Views.
- Functions.
- Types.
- Sequences.

For example:

```sql
CREATE SCHEMA fleet;
```

A table can be created inside that schema:

```sql
CREATE TABLE fleet.vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    inventory_number VARCHAR(30) NOT NULL UNIQUE
);
```

The physical database schema is an implementation of the database design.

### Key distinction

| Concept | Meaning |
|---|---|
| Data model | Abstract representation of information |
| Database schema | Defined organization of database objects |
| Physical schema | DBMS-specific implementation structure |

---

# 10. Logical vs Physical Data Independence

Database architecture also considers the ability to change one level without unnecessarily affecting another.

This idea is related to **data independence**.

## 10.1 Logical data independence

Logical data independence is the ability to change the conceptual or logical structure without requiring all external applications and views to change.

For example:

An organization introduces a new attribute to represent an equipment classification.

A well-designed database may accommodate this change without requiring every application to be redesigned.

However, this depends on how applications and database interfaces are structured.

## 10.2 Physical data independence

Physical data independence is the ability to change physical storage details without changing the logical data structure or application behavior.

For example:

- Adding an index.
- Changing storage configuration.
- Moving a table to a different tablespace.
- Adjusting physical organization.

Applications should generally continue to use the same logical database interface.

### Why this matters

A database can evolve without forcing every dependent system to be rebuilt.

This is an important objective of good database architecture.

Related note: [[05 - Database Architecture Fundamentals]].

---

# 11. Common Modeling Approaches and Notations

Different tools and notations can represent database models.

## 11.1 Entity-Relationship Diagram (ERD)

An **Entity-Relationship Diagram** represents entities, attributes, and relationships.

It is frequently used during conceptual and logical modeling.

Example:

```text
+------------------+
|    DEPARTMENT    |
+------------------+
| department_id    |
| department_name  |
+------------------+
         |
         | 1
         |
         | N
+------------------+
|     VEHICLE      |
+------------------+
| vehicle_id       |
| inventory_number |
| brand            |
| model            |
| department_id    |
+------------------+
```

## 11.2 Crow's Foot Notation

Crow's Foot notation represents cardinality using visual symbols.

Common meanings:

- One.
- Zero or one.
- One or many.
- Zero or many.

It helps describe the minimum and maximum participation in a relationship.

Related note: [[02 - Relationships and Cardinality]].

## 11.3 UML Class Diagrams

UML class diagrams can also represent structural relationships.

Although they are commonly associated with object-oriented design, they can help describe entities and associations.

They are not interchangeable with every ER modeling notation, but some concepts overlap.

## 11.4 Modeling tools

Common database modeling tools include:

- pgModeler.
- DBeaver.
- MySQL Workbench.
- Oracle SQL Developer Data Modeler.
- ERwin Data Modeler.
- Visual Paradigm.

Tool selection depends on the DBMS, team workflow, licensing, and documentation requirements.

A modeling tool helps create diagrams and generate scripts, but it does not replace sound architectural decisions.

---

# 12. Reverse Engineering and Forward Engineering

Database modeling can move in two directions.

## 12.1 Forward engineering

Forward engineering transforms a model into a database implementation.

```text
Conceptual Model
       |
       v
Logical Model
       |
       v
Physical Model
       |
       v
SQL DDL
       |
       v
Database
```

Example:

A designer defines a logical vehicle entity, selects PostgreSQL data types, and generates a `CREATE TABLE` statement.

## 12.2 Reverse engineering

Reverse engineering starts with an existing database and reconstructs its structure.

```text
Existing Database
       |
       v
Inspect Tables
       |
       v
Identify Columns and Keys
       |
       v
Reconstruct Relationships
       |
       v
Generate Database Diagram
       |
       v
Document Existing Architecture
```

This is useful when:

- Documentation is missing.
- A system was inherited from another developer.
- A database must be migrated.
- An architecture review is required.
- Existing relationships need to be understood.

### Example

Suppose an organization has an existing PostgreSQL database.

An engineer inspects:

- Tables.
- Columns.
- Primary keys.
- Foreign keys.
- Indexes.
- Constraints.
- Views.

The engineer can reconstruct a logical model from the existing physical implementation.

However, some original business requirements may not be recoverable from the schema alone.

That is why reverse engineering often needs interviews, documentation, and application analysis.

---

# 13. Common Modeling Mistakes

## Mistake 1: Starting directly with SQL

Creating tables without understanding business requirements can produce inconsistent structures.

**Better approach:** Define the conceptual model first.

## Mistake 2: Mixing conceptual and physical details

A conceptual diagram should not become overloaded with SQL types, indexes, and storage parameters.

**Better approach:** Keep each modeling level focused on its purpose.

## Mistake 3: Ignoring business rules

A technically valid database can still represent the business incorrectly.

**Better approach:** Document business rules and validate them with stakeholders.

## Mistake 4: Skipping logical modeling

Moving directly from requirements to SQL makes it harder to detect missing relationships and normalization problems.

**Better approach:** Define attributes, keys, cardinality, and constraints before physical implementation.

## Mistake 5: Choosing data types without considering requirements

For example, storing dates as text makes validation and date operations more difficult.

**Better approach:** Select data types according to the meaning and expected use of each attribute.

## Mistake 6: Treating the physical model as permanent

Database implementations may evolve as data volume, query patterns, and operational requirements change.

**Better approach:** Review physical decisions as the workload changes.

## Mistake 7: Failing to document assumptions

A diagram may show a relationship without explaining the business rule behind it.

**Better approach:** Document assumptions, definitions, optionality, and exceptions.

---

# 14. Applying the Three Models to SIAPV

Consider a simplified scenario for a vehicle and machinery management system.

The objective is to manage:

- Vehicles and machinery.
- Inventory identifiers.
- Organizational assignments.
- Maintenance activities.
- Operational information.

The same system can be analyzed through three modeling levels.

## 14.1 Conceptual model

Business entities:

- Vehicle.
- Department.
- Maintenance.
- Employee.
- Workshop.

Business rules:

- Each registered unit must be identifiable.
- A unit can have maintenance records.
- Maintenance activities must refer to a registered unit.
- Employees may participate in maintenance activities.
- A unit may be associated with an organizational area.

The conceptual model explains the information required by the organization.

## 14.2 Logical model

A possible logical structure:

```text
DEPARTMENT
    Department ID (PK)
    Department Name

VEHICLE
    Vehicle ID (PK)
    Inventory Number (AK)
    Brand
    Model
    Department ID (FK)

MAINTENANCE
    Maintenance ID (PK)
    Vehicle ID (FK)
    Date
    Description
    Cost

EMPLOYEE
    Employee ID (PK)
    Employee Code (AK)
    Full Name

MAINTENANCE_EMPLOYEE
    Maintenance ID (PK, FK)
    Employee ID (PK, FK)
    Role
```

At this level, the designer evaluates whether the relationships accurately represent the real operational process.

For example, an administrative department and a physical workshop may need to be modeled as separate concepts.

## 14.3 Physical model

A PostgreSQL implementation could define:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    inventory_number VARCHAR(30) NOT NULL UNIQUE,

    brand VARCHAR(80) NOT NULL,

    model VARCHAR(80) NOT NULL,

    department_id BIGINT NOT NULL,

    CONSTRAINT fk_vehicle_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

The physical model adds implementation-specific details.

### Architectural questions to investigate

When reviewing an existing system such as SIAPV, consider:

1. Does every entity have a stable identifier?
2. Are business identifiers protected against duplication?
3. Are administrative and operational relationships represented separately?
4. Are historical maintenance records preserved?
5. Are foreign keys implemented consistently?
6. Are data types appropriate?
7. Are indexes aligned with actual queries?
8. Can the structure support future reports and integrations?

The objective is not simply to create more tables.

It is to ensure that the database represents the real information domain accurately and can evolve with the organization.

---

# 15. Practical Exercise — From Business Requirement to Database

Use the following scenario:

An organization needs a system to manage workshops and the equipment assigned to them.

Each workshop has:

- A name.
- A location.
- A responsible employee.

Each equipment record has:

- An inventory number.
- A brand.
- A model.
- An acquisition date.
- An operating-hour reading.

A workshop can manage multiple equipment records, but each equipment record belongs to one workshop.

## Exercise A — Conceptual model

Identify:

- Main entities.
- Business relationships.
- Business rules.
- Assumptions.

Do not define SQL data types yet.

## Exercise B — Logical model

For each entity, define:

- Attributes.
- Primary key.
- Candidate keys.
- Foreign keys.
- Cardinality.
- Required and optional attributes.

Represent the relationship between workshops and equipment.

## Exercise C — Physical model

Translate the logical model into PostgreSQL.

Include:

- `CREATE TABLE`.
- Appropriate data types.
- Primary keys.
- Foreign keys.
- Unique constraints.
- NOT NULL constraints.
- CHECK constraints where necessary.
- Appropriate indexes.

## Exercise D — Design justification

Explain:

1. Why did you select each primary key?
2. Which natural identifiers require uniqueness?
3. Why are certain attributes optional?
4. Which referential action should be used when a workshop is deleted?
5. What would need to change if equipment could be assigned to multiple workshops over time?

The final question introduces the need to represent historical relationships rather than simply storing one current assignment.

---

# 16. Knowledge Check

1. What is the main purpose of conceptual modeling?
2. What additional details does a logical model introduce?
3. Why is a physical model DBMS-dependent?
4. What is the difference between a data model and a database schema?
5. Where should primary keys and foreign keys be defined?
6. Why are data types usually associated with physical modeling?
7. What is a data dictionary?
8. What is forward engineering?
9. What is reverse engineering?
10. How does abstraction help database architects?
11. What is logical data independence?
12. What is physical data independence?
13. Why should business rules be documented before implementation?
14. Can a physical database diagram replace the original business requirements?
15. How can a logical model help when migrating from PostgreSQL to another DBMS?

---

# 17. Key Takeaways

1. Data modeling translates business requirements into structured information representations.
2. Conceptual modeling focuses on business entities and relationships.
3. Logical modeling defines attributes, keys, relationships, cardinality, and integrity rules.
4. Physical modeling implements the structure in a specific DBMS.
5. The three models describe the same system at different abstraction levels.
6. Business rules guide the design at every level.
7. A data dictionary documents the meaning and structure of database elements.
8. Forward engineering transforms models into implementations.
9. Reverse engineering reconstructs models from existing databases.
10. Data independence helps separate logical structures from physical implementation.
11. A good database design should reflect the actual business domain, not merely the structure that is easiest to code.

---

# 18. Technical Vocabulary

| English term | Spanish equivalent |
|---|---|
| Data model | Modelo de datos |
| Conceptual data model | Modelo conceptual de datos |
| Logical data model | Modelo lógico de datos |
| Physical data model | Modelo físico de datos |
| Abstraction | Abstracción |
| Business requirement | Requerimiento de negocio |
| Business rule | Regla de negocio |
| Entity | Entidad |
| Attribute | Atributo |
| Relationship | Relación |
| Cardinality | Cardinalidad |
| Optionality | Opcionalidad |
| Data dictionary | Diccionario de datos |
| Database schema | Esquema de base de datos |
| Entity-Relationship Diagram | Diagrama entidad-relación |
| Forward engineering | Ingeniería directa |
| Reverse engineering | Ingeniería inversa |
| Data independence | Independencia de datos |
| Logical data independence | Independencia lógica de datos |
| Physical data independence | Independencia física de datos |
| Database implementation | Implementación de base de datos |
| Storage configuration | Configuración de almacenamiento |
| Database migration | Migración de base de datos |
| Traceability | Trazabilidad |

---

# 19. Related Learning Notes

## Data Foundations

- [[05 - Database Architecture Fundamentals]]

## Data Modeling

- [[01 - Entities and Attributes]]
- [[02 - Relationships and Cardinality]]
- [[03 - Keys and Constraints]]
- [[05 - Normalization]]
- [[06 - Schema Design Principles]]

## Database Internals

- [[01 - Database Management Systems]]
- [[04 - Indexes]]
- [[01 - PostgreSQL Architecture Overview]]

## Applied Architecture

- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]
- [[03 - Architecture Decision Framework]]

---

# 20. Next Learning Step

The next topic is **Normalization**.

Normalization explains how to organize relational data to reduce unnecessary duplication, prevent update anomalies, and improve structural consistency.

We will study:

- Functional dependencies.
- First Normal Form (1NF).
- Second Normal Form (2NF).
- Third Normal Form (3NF).
- Boyce-Codd Normal Form (BCNF).
- Denormalization and its architectural trade-offs.
