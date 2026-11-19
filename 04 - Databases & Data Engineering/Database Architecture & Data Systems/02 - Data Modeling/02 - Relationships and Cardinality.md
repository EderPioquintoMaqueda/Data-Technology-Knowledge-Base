
---
title: "Relationships and Cardinality"
aliases:
  - Database Relationships
  - Relationship Cardinality
  - Entity Relationships
  - ER Modeling Relationships

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
  - data-modeling
  - database-relationships
  - cardinality
  - entity-relationship-model
  - relational-databases
  - database-design
  - foreign-keys

concepts:
  - entity-relationship
  - cardinality
  - one-to-one
  - one-to-many
  - many-to-many
  - participation
  - optionality
  - foreign-key
  - junction-table
  - referential-integrity
  - entity-relationship-diagram

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Entities and Attributes]]"

related:
  - "[[03 - Keys and Constraints]]"
  - "[[04 - Conceptual Logical and Physical Models]]"
  - "[[05 - Normalization]]"
  - "[[06 - Schema Design Principles]]"
  - "[[01 - Database Management Systems]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[01 - SIAPV Data Architecture]]"
  - "[[02 - SIAPV Database Design Review]]"

tools:
  - PostgreSQL
  - SQL
  - pgAdmin
  - diagrams.net
  - dbdiagram.io

learning_objectives:
  - Define relationships between database entities.
  - Understand relationship cardinality.
  - Differentiate one-to-one, one-to-many, and many-to-many relationships.
  - Understand optional and mandatory participation.
  - Identify foreign keys and their role in relational databases.
  - Understand why many-to-many relationships require an intermediate structure.
  - Interpret basic entity-relationship diagrams.
  - Translate business rules into relational structures.

review:
  status: pending
  confidence:
  last_reviewed:
---

# Relationships and Cardinality

## 1. Introduction

In the previous document, we learned how to identify entities and their attributes.

For example, a vehicle management system may contain:

- Vehicle.
- Employee.
- Department.
- Maintenance.
- Workshop.

However, identifying these entities is not enough.

We must also understand how they interact.

Consider the following questions:

- Can a department manage multiple vehicles?
- Can a vehicle have multiple maintenance records?
- Can an employee be responsible for multiple vehicles?
- Can a vehicle be assigned to different employees over time?
- Can multiple employees participate in the same maintenance operation?

These questions describe relationships between entities.

**Relationships define how information is connected within a database model.**

Cardinality describes how many instances of one entity can be associated with instances of another entity.

Together, relationships and cardinality help transform a collection of independent entities into a coherent data model.

## 2. What Is a Relationship?

A **Relationship** describes an association between two or more entities.

For example:

- An employee belongs to a department.
- A department manages vehicles.
- A vehicle receives maintenance.
- A supplier provides products.
- A customer places orders.

A relationship represents a business rule.

Consider:

```text
DEPARTMENT -------- manages -------- VEHICLE
```

This expresses that departments manage vehicles.

However, it does not yet specify how many vehicles a department can manage or whether a vehicle can belong to multiple departments.

That information is defined through cardinality.

## 3. What Is Cardinality?

**Cardinality** describes the number of entity instances that can participate in a relationship.

It helps answer questions such as:

- Can one department have several vehicles?
- Can one vehicle belong to several departments?
- Can one employee manage several maintenance operations?
- Can one maintenance operation involve several vehicles?

Three fundamental relationship types are:

1. One-to-One (1:1).
2. One-to-Many (1:N).
3. Many-to-Many (M:N).

These relationships are essential in relational database modeling.

## 4. One-to-One Relationship (1:1)

A one-to-one relationship means that one instance of an entity can be associated with at most one instance of another entity, and vice versa.

### Example: Employee and Employee Profile

Imagine that an organization stores additional personal information in a separate profile structure.

```text
EMPLOYEE  1 -------- 1  EMPLOYEE_PROFILE
```

Each employee has at most one profile.

Each profile belongs to one employee.

### Example Tables

**EMPLOYEE**

| employee_id | first_name |
|---|---|
| 101 | Daniel |
| 102 | Laura |

**EMPLOYEE_PROFILE**

| profile_id | employee_id | emergency_contact |
|---|---|---|
| 501 | 101 | 5551001 |
| 502 | 102 | 5551002 |

The `employee_id` in `EMPLOYEE_PROFILE` references the corresponding employee.

### SQL Implementation

```sql
CREATE TABLE employees (
    employee_id BIGINT PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL
);

CREATE TABLE employee_profiles (
    profile_id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL UNIQUE,
    emergency_contact VARCHAR(30),

    CONSTRAINT fk_profile_employee
        FOREIGN KEY (employee_id)
        REFERENCES employees(employee_id)
);
```

The `UNIQUE` constraint ensures that an employee cannot appear more than once in `employee_profiles`.

The foreign key ensures that the referenced employee exists.

### Important Observation

A one-to-one relationship is not always necessary.

Sometimes both sets of attributes can be stored in one table.

Separating them may be justified by:

- Different access permissions.
- Optional information.
- Different lifecycle requirements.
- Security considerations.
- Organizational boundaries.

The model should reflect a real requirement, not simply create extra tables.

## 5. One-to-Many Relationship (1:N)

A one-to-many relationship means that one instance of an entity can be associated with multiple instances of another entity.

However, each instance on the many side is associated with at most one instance on the one side.

### Example: Department and Employees

One department may have many employees.

Each employee belongs to one department in this simplified model.

```text
DEPARTMENT  1 -------- N  EMPLOYEE
```

### Example Tables

**DEPARTMENT**

| department_id | department_name |
|---|---|
| 10 | Administration |
| 20 | Maintenance |

**EMPLOYEE**

| employee_id | first_name | department_id |
|---|---|---|
| 101 | Daniel | 10 |
| 102 | Laura | 10 |
| 103 | Carlos | 20 |

The `department_id` attribute in the Employee table connects each employee to a department.

### SQL Implementation

```sql
CREATE TABLE departments (
    department_id BIGINT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    employee_id BIGINT PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    department_id BIGINT NOT NULL,

    CONSTRAINT fk_employee_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

### Where Does the Foreign Key Go?

In a one-to-many relationship, the foreign key is generally placed on the many side.

For this example:

- Department is the one side.
- Employee is the many side.
- `department_id` is stored in Employee.

This is one of the most important implementation rules in relational database design.

### Another Example: Vehicle and Maintenance

One vehicle may have multiple maintenance records.

Each maintenance record refers to one vehicle.

```text
VEHICLE  1 -------- N  MAINTENANCE
```

A simplified implementation:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT PRIMARY KEY,
    inventory_number VARCHAR(30) UNIQUE NOT NULL
);

CREATE TABLE maintenance (
    maintenance_id BIGINT PRIMARY KEY,
    vehicle_id BIGINT NOT NULL,
    maintenance_date DATE NOT NULL,
    maintenance_cost NUMERIC(12, 2),

    CONSTRAINT fk_maintenance_vehicle
        FOREIGN KEY (vehicle_id)
        REFERENCES vehicles(vehicle_id)
);
```

A maintenance record cannot reference a vehicle that does not exist.

## 6. Many-to-Many Relationship (M:N)

A many-to-many relationship means that multiple instances of one entity can be associated with multiple instances of another entity.

### Example: Employees and Projects

An employee may participate in several projects.

A project may involve several employees.

```text
EMPLOYEE  N -------- M  PROJECT
```

Neither entity can represent the complete relationship using only one foreign key.

### Why Is This a Problem?

Consider this hypothetical structure:

| employee_id | employee_name | project_ids |
|---|---|---|
| 101 | Daniel | 1, 2, 3 |
| 102 | Laura | 2, 3 |

This design stores multiple project identifiers inside a single field.

It creates difficulties when searching, updating, validating, and relating individual records.

Relational database design normally resolves this problem using an intermediate table.

## 7. Junction Tables

A **Junction Table**, also called an associative table or bridge table, converts a many-to-many relationship into two one-to-many relationships.

### Example: Employee and Project

```text
EMPLOYEE  1 ----- N  EMPLOYEE_PROJECT  N ----- 1  PROJECT
```

The intermediate table contains the identifiers of both related entities.

### Example Structure

**EMPLOYEE**

| employee_id | employee_name |
|---|---|
| 101 | Daniel |
| 102 | Laura |

**PROJECT**

| project_id | project_name |
|---|---|
| 1 | Database Migration |
| 2 | Reporting Platform |
| 3 | Inventory Automation |

**EMPLOYEE_PROJECT**

| employee_id | project_id |
|---|---|
| 101 | 1 |
| 101 | 2 |
| 102 | 2 |
| 102 | 3 |

Each row in the junction table represents one association.

### SQL Implementation

```sql
CREATE TABLE employees (
    employee_id BIGINT PRIMARY KEY,
    employee_name VARCHAR(100) NOT NULL
);

CREATE TABLE projects (
    project_id BIGINT PRIMARY KEY,
    project_name VARCHAR(150) NOT NULL
);

CREATE TABLE employee_projects (
    employee_id BIGINT NOT NULL,
    project_id BIGINT NOT NULL,

    PRIMARY KEY (employee_id, project_id),

    CONSTRAINT fk_assignment_employee
        FOREIGN KEY (employee_id)
        REFERENCES employees(employee_id),

    CONSTRAINT fk_assignment_project
        FOREIGN KEY (project_id)
        REFERENCES projects(project_id)
);
```

The composite primary key prevents the same employee-project association from being registered twice.

### 7.1 Junction Tables Can Have Their Own Attributes

An intermediate table may contain additional information about the relationship.

For example:

- Assignment date.
- Completion date.
- Project role.
- Hours assigned.
- Participation status.

```sql
CREATE TABLE employee_projects (
    employee_id BIGINT NOT NULL,
    project_id BIGINT NOT NULL,
    assignment_date DATE NOT NULL,
    project_role VARCHAR(80),

    PRIMARY KEY (employee_id, project_id),

    FOREIGN KEY (employee_id)
        REFERENCES employees(employee_id),

    FOREIGN KEY (project_id)
        REFERENCES projects(project_id)
);
```

This is important because the relationship itself may contain business information.

## 8. Cardinality vs Optionality

Cardinality and optionality are related, but they are not exactly the same concept.

**Cardinality** describes the possible number of related instances.

**Optionality** describes whether participation in a relationship is mandatory or optional.

Consider the relationship between Department and Employee.

Possible rules:

- A department may have zero or many employees.
- An employee must belong to one department.

The minimum and maximum participation can be represented as:

```text
DEPARTMENT (0..N) -------- (1..1) EMPLOYEE
```

Interpretation:

- `0..N`: A department can have zero, one, or many employees.
- `1..1`: Each employee must belong to exactly one department.

### Common Participation Ranges

| Notation | Meaning |
|---|---|
| 0..1 | Zero or one |
| 1..1 | Exactly one |
| 0..N | Zero or many |
| 1..N | One or many |

These ranges help express business rules more precisely than simply writing 1:N.

### Example: Vehicle and Maintenance

A vehicle may not have any maintenance records yet.

A vehicle may also have many maintenance records.

Each maintenance record must refer to one vehicle.

```text
VEHICLE (0..N) -------- (1..1) MAINTENANCE
```

This distinction becomes important when implementing `NOT NULL`, foreign keys, and other constraints.

## 9. Understanding ER Diagrams

An **Entity-Relationship Diagram (ERD)** is a visual representation of entities, attributes, and relationships.

ER diagrams help developers and architects understand the structure of a database before implementing it.

### 9.1 Basic Representation

```text
+-------------------+
|     DEPARTMENT    |
+-------------------+
| PK department_id  |
|    department_name|
+-------------------+
          |
          | 1
          |
          | N
+-------------------+
|      EMPLOYEE     |
+-------------------+
| PK employee_id    |
| FK department_id  |
|    first_name     |
+-------------------+
```

In this diagram:

- `PK` means Primary Key.
- `FK` means Foreign Key.
- The relationship is one-to-many.

### 9.2 Crow's Foot Notation

Crow's Foot notation is commonly used to represent entity relationships.

Its symbols indicate:

- A single instance.
- Zero or optional participation.
- Multiple instances.
- Mandatory participation.

Example:

```text
DEPARTMENT ||------o{ EMPLOYEE
```

Conceptually:

- A department can have zero or many employees.
- Each employee belongs to exactly one department.

The exact appearance of symbols may vary depending on the diagramming tool.

### 9.3 Why ER Diagrams Matter

They help teams:

- Visualize data structures.
- Identify missing relationships.
- Detect inconsistencies.
- Communicate design decisions.
- Review database requirements.
- Prepare relational schemas.

Tools such as diagrams.net and dbdiagram.io can help create these diagrams.

## 10. Translating Business Rules into Relationships

Database relationships should come from business requirements.

Consider these statements:

**Rule 1:** A department manages multiple vehicles.

Possible relationship:

```text
DEPARTMENT 1 -------- N VEHICLE
```

**Rule 2:** Each vehicle belongs to one department at a time.

This supports placing a department foreign key in the Vehicle table.

**Rule 3:** A vehicle can have multiple maintenance records.

Possible relationship:

```text
VEHICLE 1 -------- N MAINTENANCE
```

**Rule 4:** Each maintenance record refers to one vehicle.

This supports placing a vehicle foreign key in the Maintenance table.

**Rule 5:** A maintenance operation may involve several employees, and each employee may participate in multiple maintenance operations.

Possible relationship:

```text
EMPLOYEE N -------- M MAINTENANCE
```

This requires a junction table, such as `maintenance_employees`.

The database structure must reflect the actual business rules.

## 11. Applying Relationships to SIAPV

Imagine a simplified vehicle management system with the following entities:

- Department.
- Vehicle.
- Employee.
- Maintenance.
- Workshop.

Possible relationships include:

| Relationship | Cardinality | Explanation |
|---|---|---|
| Department → Vehicle | 1:N | A department may manage multiple vehicles |
| Vehicle → Maintenance | 1:N | A vehicle may have multiple maintenance records |
| Workshop → Maintenance | 1:N | A workshop may perform multiple maintenance operations |
| Employee → Maintenance | M:N | Multiple employees may participate in multiple maintenance operations |

These are illustrative modeling assumptions. The actual cardinality must be confirmed against organizational rules.

### 11.1 Preliminary Relationship Diagram

```text
+------------------+
|    DEPARTMENT    |
+------------------+
| PK department_id |
+------------------+
         |
         | 1:N
         |
         v
+------------------+
|      VEHICLE     |
+------------------+
| PK vehicle_id    |
| FK department_id|
+------------------+
         |
         | 1:N
         |
         v
+------------------+
|    MAINTENANCE   |
+------------------+
| PK maintenance_id|
| FK vehicle_id    |
| FK workshop_id   |
+------------------+
         ^
         |
         | N:1
         |
+------------------+
|     WORKSHOP     |
+------------------+
| PK workshop_id   |
+------------------+

+------------------+       +----------------------+
|     EMPLOYEE     | 1:N   | MAINTENANCE_EMPLOYEE  |
+------------------+-------+----------------------+
| PK employee_id   |       | PK, FK employee_id   |
+------------------+       | PK, FK maintenance_id|
                           +----------------------+
```

The diagram is a simplified logical representation.

It does not yet address all possible historical assignments, employee roles, multiple workshops, or operational constraints.

Those decisions require additional requirements analysis.

## 12. Common Modeling Mistakes

### Mistake 1: Choosing Cardinality Without Understanding the Business

A department may manage many vehicles, but the organization might also allow temporary vehicle assignments to other departments.

The correct model depends on whether the system must preserve only the current assignment or the complete assignment history.

### Mistake 2: Putting a Foreign Key on the Wrong Side

For a basic one-to-many relationship, the foreign key normally belongs on the many side.

### Mistake 3: Storing Multiple Identifiers in One Column

A relational design should not normally store a list of foreign identifiers inside a single text field.

Use a junction table for many-to-many relationships.

### Mistake 4: Ignoring Optionality

A vehicle may exist before receiving its first maintenance record.

The model should allow that situation.

### Mistake 5: Confusing Relationships with Attributes

A department is not simply an attribute if it must be managed independently.

Likewise, an employee's department relationship should normally be represented through a foreign key rather than duplicated department information.

### Mistake 6: Ignoring Relationship Attributes

Some relationships contain important information of their own.

For example, a vehicle assignment may require:

- Assignment start date.
- Assignment end date.
- Responsible employee.
- Assignment status.

A separate associative entity may be necessary.

## 13. Knowledge Check

Answer these questions without reviewing the explanations.

1. What is a relationship in database modeling?
2. What does cardinality describe?
3. What is a one-to-one relationship?
4. What is a one-to-many relationship?
5. What is a many-to-many relationship?
6. Why does a many-to-many relationship normally require a junction table?
7. Where is the foreign key generally placed in a one-to-many relationship?
8. What is the difference between cardinality and optionality?
9. What does the notation `0..N` represent?
10. What is an Entity-Relationship Diagram?
11. What is Crow's Foot notation?
12. Why must relationships be based on business requirements?

## 14. Practical Exercises

### Exercise A: Classify Relationships

Determine the cardinality for each relationship.

| Business Rule | Cardinality |
|---|---|
| One department manages multiple employees | |
| One vehicle has multiple maintenance records | |
| One employee has one employee profile | |
| Multiple employees participate in multiple projects | |
| One workshop performs multiple maintenance operations | |
| Multiple students enroll in multiple courses | |

Explain your reasoning.

### Exercise B: Design a Relational Structure

Consider these entities:

- Customer.
- Order.
- Product.
- Supplier.

Define the relationships between them.

Consider the following rules:

1. A customer can place multiple orders.
2. Each order belongs to one customer.
3. An order can contain multiple products.
4. A product can appear in multiple orders.
5. A supplier can provide multiple products.

Identify:

- Relationship cardinalities.
- Foreign key locations.
- Junction tables.
- Possible attributes of the junction tables.

### Exercise C: SIAPV Relationship Modeling

Design the relationships for:

- Department.
- Vehicle.
- Employee.
- Maintenance.
- Workshop.

Answer:

1. Can one department manage multiple vehicles?
2. Can one vehicle have multiple maintenance records?
3. Can a workshop perform multiple maintenance operations?
4. Can multiple employees participate in one maintenance operation?
5. Should the model preserve historical vehicle assignments?
6. Which relationships require junction tables?

Create an ER diagram using Crow's Foot notation.

### Exercise D: SQL Implementation

Implement the following simplified structure in PostgreSQL:

- `departments`
- `vehicles`
- `maintenance`

Requirements:

1. Every department must have a unique identifier.
2. Every vehicle must have a unique identifier.
3. Each vehicle must reference an existing department.
4. Every maintenance record must reference an existing vehicle.
5. A vehicle may have zero or many maintenance records.

Verify the relationships using SQL queries.

## 15. Key Takeaways

- Relationships connect entities according to business rules.
- Cardinality defines how many instances can participate in a relationship.
- One-to-one relationships associate individual instances.
- One-to-many relationships normally place a foreign key on the many side.
- Many-to-many relationships generally require a junction table.
- Junction tables can contain attributes describing the relationship.
- Optionality defines whether participation is mandatory.
- ER diagrams visualize entities and their relationships.
- Crow's Foot notation is commonly used to represent cardinality and optionality.
- Business requirements determine the correct relational structure.

## 16. Related Notes

### Previous Topic
- [[01 - Entities and Attributes]]

### Next Topics
- [[03 - Keys and Constraints]]
- [[04 - Conceptual Logical and Physical Models]]
- [[05 - Normalization]]
- [[06 - Schema Design Principles]]

### Database Implementation
- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[01 - PostgreSQL Architecture Overview]]

### Practical Application
- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]

---

**Learning Objective:** Model relationships between business entities, determine cardinality and optionality, and translate those rules into relational database structures using foreign keys and junction tables.
