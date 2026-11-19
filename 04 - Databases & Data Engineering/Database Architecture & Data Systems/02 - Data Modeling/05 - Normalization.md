
---
title: "Database Normalization"
aliases:
  - "Normalization"
  - "Relational Database Normalization"
  - "Database Normal Forms"
  - "1NF 2NF 3NF and BCNF"
type: learning-note
domain: technology
category: data-engineering
subject: data-modeling
level: beginner-to-intermediate
difficulty: foundational-to-intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - database-architecture
  - data-modeling
  - normalization
  - relational-databases
  - functional-dependencies
  - database-design
  - data-integrity
  - postgresql
concepts:
  - database-normalization
  - data-redundancy
  - insertion-anomaly
  - update-anomaly
  - deletion-anomaly
  - functional-dependency
  - candidate-key
  - first-normal-form
  - second-normal-form
  - third-normal-form
  - boyce-codd-normal-form
  - denormalization
  - lossless-decomposition
  - dependency-preservation
prerequisites:
  - "[[01 - Entities and Attributes]]"
  - "[[02 - Relationships and Cardinality]]"
  - "[[03 - Keys and Constraints]]"
  - "[[04 - Conceptual Logical and Physical Models]]"
related:
  - "[[06 - Schema Design Principles]]"
  - "[[01 - Database Management Systems]]"
  - "[[02 - How a Query is Executed]]"
  - "[[04 - Indexes]]"
  - "[[05 - Transactions and ACID]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[01 - SIAPV Data Architecture]]"
  - "[[02 - SIAPV Database Design Review]]"
tools:
  - PostgreSQL
  - SQL
  - pgAdmin
  - Database Modeling Tools
learning_objectives:
  - "Understand the purpose of database normalization."
  - "Identify data redundancy and modification anomalies."
  - "Understand functional dependencies."
  - "Apply First, Second, and Third Normal Forms."
  - "Understand Boyce-Codd Normal Form."
  - "Evaluate lossless decomposition and dependency preservation."
  - "Distinguish normalization from denormalization."
  - "Apply normalization principles to operational database scenarios."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 05 — Database Normalization

## 1. Introduction

A relational database should organize information in a way that minimizes unnecessary duplication and protects consistency.

Imagine a table that stores vehicles, departments, employees, and maintenance information together.

Every time a vehicle receives maintenance, the system repeats the department name, employee information, vehicle brand, and other attributes.

This may seem convenient initially.

However, repeated information creates problems when records are inserted, updated, or deleted.

**Database normalization** is the process of organizing relational data into well-structured tables according to defined rules called normal forms.

Its main objectives are:

- Reduce unnecessary data redundancy.
- Improve data consistency.
- Prevent modification anomalies.
- Represent functional dependencies correctly.
- Create a more reliable relational structure.

### Central principle

Each fact should be stored in an appropriate place, and relationships between facts should be represented through keys.

Normalization is not simply about creating more tables.

It is about ensuring that each table represents a coherent set of facts.

---

# 2. What Is Data Redundancy?

**Data redundancy** occurs when the same information is unnecessarily repeated in multiple records or locations.

Consider this table:

| maintenance_id | inventory_number | brand | department_name | maintenance_date |
|---|---|---|---|---|
| 501 | INV-001 | Nissan | Administration | 2026-01-10 |
| 502 | INV-001 | Nissan | Administration | 2026-03-15 |
| 503 | INV-002 | John Deere | Operations | 2026-02-20 |
| 504 | INV-001 | Nissan | Administration | 2026-05-08 |

The vehicle information appears repeatedly.

For example:

- `INV-001` appears three times.
- `Nissan` appears three times.
- `Administration` appears three times.

Some repetition is necessary because each maintenance record must identify the vehicle involved.

However, repeating descriptive vehicle and department attributes introduces unnecessary dependencies.

### Why can redundancy be problematic?

Suppose the vehicle is transferred from Administration to another department.

If the department name is stored directly in every maintenance record, several rows may need to be updated.

If one row is missed, the database becomes inconsistent.

Normalization helps separate the facts according to the entities they describe.

---

# 3. Database Anomalies

Poorly structured tables can create three important problems.

These are known as **modification anomalies**.

## 3.1 Insertion anomaly

An insertion anomaly occurs when information cannot be inserted without introducing unrelated information or creating incomplete records.

Example:

A department is created, but it has not received any vehicles yet.

If department information exists only inside the vehicle table, it may be impossible to register the department without creating a vehicle record.

The database structure unnecessarily couples two independent facts.

## 3.2 Update anomaly

An update anomaly occurs when the same fact must be modified in multiple records.

Example:

The department name changes from:

```text
Administration
```

to:

```text
Administrative Services
```

If the name appears in 100 maintenance records, all 100 records may need to be updated.

If some remain unchanged, the database contains contradictory information.

## 3.3 Deletion anomaly

A deletion anomaly occurs when deleting one record unintentionally removes information that should have been preserved.

Example:

A department has only one registered vehicle.

If the database stores department information exclusively in the vehicle table, deleting the last vehicle record may also remove the only stored information about that department.

### Anomaly comparison

| Anomaly | Problem | Example |
|---|---|---|
| Insertion | Cannot register an independent fact | Department without vehicles |
| Update | Same fact must be changed in multiple places | Department name changes |
| Deletion | Removing one record destroys another fact | Deleting the last vehicle removes department information |

These anomalies are among the main reasons for normalization.

---

# 4. Functional Dependencies

Before studying normal forms, it is necessary to understand **functional dependencies**.

A functional dependency describes a relationship in which one attribute or combination of attributes determines another attribute.

It is written as:

\[
X \rightarrow Y
\]

This means:

> X functionally determines Y.

In other words, for each valid value of X, there is only one corresponding value of Y within the relation.

## 4.1 Simple example

Consider:

| employee_id | employee_name | email |
|---|---|---|
| 101 | Ana Lopez | ana@example.com |
| 102 | Luis Perez | luis@example.com |
| 103 | Carlos Ruiz | carlos@example.com |

If `employee_id` uniquely identifies an employee:

\[
employee\_id \rightarrow employee\_name
\]

\[
employee\_id \rightarrow email
\]

The employee ID determines the employee name and email.

## 4.2 Functional dependency does not mean causation

A functional dependency is a structural property of the data.

It does not mean that one attribute causes another to exist.

For example:

\[
inventory\_number \rightarrow brand
\]

means that each inventory number determines one brand in the modeled domain.

It does not mean that the inventory number causes the brand to be assigned.

## 4.3 Determinant

The left-hand side of a functional dependency is called the **determinant**.

Example:

\[
vehicle\_id \rightarrow inventory\_number
\]

Here:

- Determinant: `vehicle_id`.
- Dependent attribute: `inventory_number`.

## 4.4 Full functional dependency

An attribute is fully functionally dependent on a composite key when it depends on the entire key, not merely part of it.

Example:

```text
(maintenance_id, employee_id) -> role
```

If the role describes an employee's participation in a particular maintenance activity, the combination determines the role.

Neither identifier alone necessarily determines it.

## 4.5 Partial dependency

A partial dependency occurs when an attribute depends on only part of a composite candidate key.

Example:

```text
(maintenance_id, employee_id) -> maintenance_date
```

But:

```text
maintenance_id -> maintenance_date
```

If `maintenance_id` is unique for each maintenance activity, the date depends only on part of the composite key.

This becomes important when studying Second Normal Form.

## 4.6 Transitive dependency

A transitive dependency occurs when one attribute determines another through an intermediate attribute.

Example:

\[
employee\_id \rightarrow department\_id
\]

\[
department\_id \rightarrow department\_name
\]

Therefore:

\[
employee\_id \rightarrow department\_name
\]

The employee determines the department name through the department identifier.

This is important when studying Third Normal Form.

---

# 5. What Are Normal Forms?

A **normal form** is a set of structural requirements used to evaluate and organize relational tables.

The most commonly discussed normal forms are:

1. First Normal Form (1NF).
2. Second Normal Form (2NF).
3. Third Normal Form (3NF).
4. Boyce-Codd Normal Form (BCNF).

Each form introduces additional conditions.

A relation that satisfies a higher normal form must also satisfy the lower normal forms in that sequence.

### General progression

```text
UNSTRUCTURED DATA
       |
       v
FIRST NORMAL FORM (1NF)
       |
       v
SECOND NORMAL FORM (2NF)
       |
       v
THIRD NORMAL FORM (3NF)
       |
       v
BOYCE-CODD NORMAL FORM (BCNF)
```

Most conventional transactional relational databases aim for at least Third Normal Form, although the appropriate design depends on the application.

---

# 6. First Normal Form (1NF)

## 6.1 Definition

A relation is in **First Normal Form** when each attribute contains a single value from its defined domain for each row.

A column should not store a collection of independent values in one field when those values need to be treated as separate data elements.

This is commonly described as requiring atomic values.

### Example: Not in 1NF

Consider:

| vehicle_id | inventory_number | assigned_employees |
|---|---|---|
| 1 | INV-001 | Ana, Luis |
| 2 | INV-002 | Carlos |
| 3 | INV-003 | Ana, Carlos |

The `assigned_employees` column contains multiple employee values.

This makes it difficult to:

- Search for one employee.
- Count assignments.
- Validate employee references.
- Update one assignment independently.
- Establish reliable relationships.

## 6.2 Transforming the structure

Separate vehicle information from employee assignments.

**Vehicles**

| vehicle_id | inventory_number |
|---|---|
| 1 | INV-001 |
| 2 | INV-002 |
| 3 | INV-003 |

**Vehicle employees**

| vehicle_id | employee_id |
|---|---|
| 1 | 101 |
| 1 | 102 |
| 2 | 103 |
| 3 | 101 |
| 3 | 103 |

Now each row represents one vehicle-employee association.

### SQL implementation

```sql
CREATE TABLE vehicle_employees (
    vehicle_id BIGINT NOT NULL,
    employee_id BIGINT NOT NULL,

    PRIMARY KEY (vehicle_id, employee_id)
);
```

In a complete database, both columns would also reference their respective parent tables through foreign keys.

## 6.3 Characteristics of 1NF

- Each row represents one record.
- Each column represents one attribute.
- Each field contains one value from its domain.
- Repeating groups are represented through separate rows or related tables.
- Records can be distinguished through a key.

### Important clarification

Atomicity depends on the intended meaning and operations of the data.

A full name stored in one column may be perfectly acceptable if the system treats it as one attribute.

However, if the system must independently search, validate, or analyze first names and surnames, separating them may be more appropriate.

Similarly, PostgreSQL supports arrays and JSON values, but using them does not automatically make a relational design incorrect. The question is whether those values need independent relational management.

---

# 7. Second Normal Form (2NF)

## 7.1 Definition

A relation is in **Second Normal Form** when:

1. It satisfies First Normal Form.
2. Every non-prime attribute is fully functionally dependent on every candidate key.

A non-prime attribute is an attribute that is not part of any candidate key.

In practical terms, 2NF eliminates partial dependencies on part of a composite candidate key.

### Important consideration

If a table has only single-column candidate keys, it cannot have a partial dependency on part of a composite key.

Therefore, once 1NF is satisfied, a relation with only single-column candidate keys also satisfies 2NF.

## 7.2 Example: Not in 2NF

Consider this table:

**Maintenance assignments**

| maintenance_id | employee_id | maintenance_date | employee_name | role |
|---|---|---|---|---|
| 501 | 101 | 2026-01-10 | Ana Lopez | Technician |
| 501 | 102 | 2026-01-10 | Luis Perez | Supervisor |
| 502 | 101 | 2026-03-15 | Ana Lopez | Technician |

The composite primary key is:

```text
(maintenance_id, employee_id)
```

Functional dependencies:

```text
maintenance_id -> maintenance_date

employee_id -> employee_name

(maintenance_id, employee_id) -> role
```

The problem is that:

- `maintenance_date` depends only on `maintenance_id`.
- `employee_name` depends only on `employee_id`.
- `role` depends on the combination of both identifiers.

The first two are partial dependencies.

Therefore, the table does not satisfy 2NF.

## 7.3 Transforming the structure

Separate the information into three tables.

**Maintenance**

| maintenance_id | maintenance_date |
|---|---|
| 501 | 2026-01-10 |
| 502 | 2026-03-15 |

**Employees**

| employee_id | employee_name |
|---|---|
| 101 | Ana Lopez |
| 102 | Luis Perez |

**Maintenance assignments**

| maintenance_id | employee_id | role |
|---|---|---|
| 501 | 101 | Technician |
| 501 | 102 | Supervisor |
| 502 | 101 | Technician |

Now:

- Maintenance dates depend on maintenance identifiers.
- Employee names depend on employee identifiers.
- Assignment roles depend on the complete composite key.

### SQL implementation

```sql
CREATE TABLE maintenance (
    maintenance_id BIGINT PRIMARY KEY,
    maintenance_date DATE NOT NULL
);
```

```sql
CREATE TABLE employees (
    employee_id BIGINT PRIMARY KEY,
    employee_name VARCHAR(120) NOT NULL
);
```

```sql
CREATE TABLE maintenance_assignments (
    maintenance_id BIGINT NOT NULL,
    employee_id BIGINT NOT NULL,
    role VARCHAR(50) NOT NULL,

    PRIMARY KEY (maintenance_id, employee_id),

    FOREIGN KEY (maintenance_id)
        REFERENCES maintenance(maintenance_id),

    FOREIGN KEY (employee_id)
        REFERENCES employees(employee_id)
);
```

### Characteristics of 2NF

- The table satisfies 1NF.
- Non-prime attributes depend on the entire candidate key.
- Partial dependencies are eliminated.
- Composite keys are especially important when evaluating 2NF.

---

# 8. Third Normal Form (3NF)

## 8.1 Definition

A relation is in **Third Normal Form** when:

1. It satisfies Second Normal Form.
2. It contains no transitive dependency of non-prime attributes on a candidate key.

A more formal definition states that for every non-trivial functional dependency:

\[
X \rightarrow A
\]

at least one of the following must hold:

- X is a superkey.
- A is a prime attribute.

A prime attribute is an attribute that belongs to at least one candidate key.

The practical goal is to prevent non-key attributes from depending on other non-key attributes in a way that creates redundancy.

## 8.2 Example: Not in 3NF

Consider:

**Employees**

| employee_id | employee_name | department_id | department_name |
|---|---|---|---|
| 101 | Ana Lopez | 1 | Administration |
| 102 | Luis Perez | 2 | Operations |
| 103 | Carlos Ruiz | 1 | Administration |

Functional dependencies:

```text
employee_id -> employee_name

employee_id -> department_id

department_id -> department_name
```

The employee ID determines the department ID.

The department ID determines the department name.

Therefore:

```text
employee_id -> department_name
```

through an intermediate attribute.

This creates a transitive dependency.

The department name is repeated for every employee in the same department.

## 8.3 Transforming the structure

Separate departments from employees.

**Departments**

| department_id | department_name |
|---|---|
| 1 | Administration |
| 2 | Operations |

**Employees**

| employee_id | employee_name | department_id |
|---|---|---|
| 101 | Ana Lopez | 1 |
| 102 | Luis Perez | 2 |
| 103 | Carlos Ruiz | 1 |

Now the department name is stored in one appropriate location.

### SQL implementation

```sql
CREATE TABLE departments (
    department_id BIGINT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE
);
```

```sql
CREATE TABLE employees (
    employee_id BIGINT PRIMARY KEY,
    employee_name VARCHAR(120) NOT NULL,
    department_id BIGINT NOT NULL,

    FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

### Characteristics of 3NF

- The table satisfies 2NF.
- Non-key attributes do not depend transitively on candidate keys, subject to the formal 3NF definition.
- Each fact is stored in an appropriate relation.
- Relationships are represented through keys.

---

# 9. Boyce-Codd Normal Form (BCNF)

## 9.1 Definition

**Boyce-Codd Normal Form (BCNF)** is a stronger form of Third Normal Form.

A relation is in BCNF when, for every non-trivial functional dependency:

\[
X \rightarrow Y
\]

X is a superkey.

In simpler terms:

Every determinant of a non-trivial functional dependency must be a superkey.

## 9.2 Why does BCNF exist?

A relation can satisfy 3NF while still containing certain dependencies that create redundancy.

BCNF addresses these cases by imposing a stricter condition on determinants.

## 9.3 Example

Consider a simplified table:

**Employee workshop assignments**

| employee_id | workshop_id | supervisor_id |
|---|---|---|
| 101 | 1 | 900 |
| 102 | 1 | 900 |
| 103 | 2 | 901 |

Assume the business rules are:

1. An employee can work in multiple workshops.
2. Each workshop has one supervisor.
3. A supervisor can supervise multiple workshops.
4. The supervisor is determined by the workshop.

The relevant functional dependencies are:

```text
(employee_id, workshop_id) -> supervisor_id

workshop_id -> supervisor_id
```

The candidate key is:

```text
(employee_id, workshop_id)
```

The determinant `workshop_id` is not a superkey of the original relation because multiple employees can work in the same workshop.

This violates BCNF.

It can still satisfy 3NF because `supervisor_id` is assumed to be a prime attribute under the stated candidate-key structure.

## 9.4 Decomposition

Separate workshop supervision from employee assignments.

**Workshop supervisors**

| workshop_id | supervisor_id |
|---|---|
| 1 | 900 |
| 2 | 901 |

**Employee workshops**

| employee_id | workshop_id |
|---|---|
| 101 | 1 |
| 102 | 1 |
| 103 | 2 |

The new structure separates two independent facts.

The workshop determines its supervisor.

The employee-workshop table records participation.

### Important distinction

BCNF decomposition may not preserve every original functional dependency directly within an individual decomposed table.

Therefore, architects must evaluate both:

- Lossless decomposition.
- Dependency preservation.

These properties are discussed later.

---

# 10. Comparing Normal Forms

| Normal form | Main requirement | Problem addressed |
|---|---|---|
| 1NF | Atomic attribute values and no repeating groups | Repeating values and non-atomic structures |
| 2NF | 1NF and no partial dependency of non-prime attributes on candidate keys | Partial dependencies |
| 3NF | 2NF and no disallowed transitive dependencies | Transitive dependencies |
| BCNF | Every determinant of a non-trivial dependency is a superkey | Remaining determinant-related redundancy |

### A practical way to remember them

- 1NF: Organize individual values.
- 2NF: Depend on the whole candidate key.
- 3NF: Depend on keys, not on other non-key attributes.
- BCNF: Every determinant must be a superkey.

This is a learning shortcut. The formal definitions remain important when evaluating complex candidate keys and functional dependencies.

---

# 11. Lossless Decomposition

Normalization frequently involves dividing one relation into multiple relations.

However, decomposition must preserve the ability to reconstruct the original information.

A **lossless decomposition** is a decomposition in which joining the resulting relations reconstructs the original relation without introducing spurious tuples or losing information.

Consider an original relation:

**Employees**

| employee_id | department_id | department_name |
|---|---|---|
| 101 | 1 | Administration |
| 102 | 2 | Operations |
| 103 | 1 | Administration |

It is decomposed into:

**Employee departments**

| employee_id | department_id |
|---|---|
| 101 | 1 |
| 102 | 2 |
| 103 | 1 |

**Departments**

| department_id | department_name |
|---|---|
| 1 | Administration |
| 2 | Operations |

The original information can be reconstructed with:

```sql
SELECT
    e.employee_id,
    e.department_id,
    d.department_name

FROM employee_departments AS e

INNER JOIN departments AS d
    ON e.department_id = d.department_id;
```

The join restores the original employee-department information.

### Why does losslessness matter?

A decomposition that loses information can make the new database structurally cleaner but functionally incorrect.

A correct normalization process should preserve the information represented by the original relation.

---

# 12. Dependency Preservation

A decomposition is **dependency-preserving** when the original functional dependencies can be enforced by checking the decomposed relations individually, without requiring joins to validate them.

Consider the original dependencies:

```text
employee_id -> department_id

department_id -> department_name
```

After decomposition:

**Employees**

```text
employee_id -> department_id
```

**Departments**

```text
department_id -> department_name
```

Both dependencies can be enforced in their respective tables.

This is dependency preservation.

### Why is it important?

It makes integrity rules easier to enforce.

If a dependency can only be validated by joining several tables, enforcing it may require more complex mechanisms.

### Architectural trade-off

Some decompositions, particularly those used to achieve BCNF, may not preserve every functional dependency.

Database designers sometimes need to balance:

- Strong normalization.
- Dependency preservation.
- Lossless decomposition.
- Query complexity.
- Operational requirements.

---

# 13. Denormalization

Normalization is important, but not every database should remain maximally normalized in every part of its architecture.

**Denormalization** is the intentional introduction of redundancy or precomputed information to improve particular read or reporting workloads.

It is a design decision, not necessarily a mistake.

## 13.1 Why denormalize?

A normalized structure may require several JOIN operations to retrieve information.

For some workloads, repeated joins can increase query complexity or execution cost.

Denormalization can reduce that cost by storing selected information in a more directly accessible form.

Possible reasons include:

- Read-heavy workloads.
- Reporting requirements.
- Analytical queries.
- Precomputed aggregates.
- Performance optimization.
- Simplified access patterns.

## 13.2 Example

Normalized structure:

**Vehicles**

| vehicle_id | inventory_number |
|---|---|
| 1 | INV-001 |
| 2 | INV-002 |

**Maintenance**

| maintenance_id | vehicle_id | maintenance_cost |
|---|---|---|
| 501 | 1 | 3500 |
| 502 | 1 | 1200 |
| 503 | 2 | 5000 |

To calculate total maintenance cost per vehicle:

```sql
SELECT
    v.vehicle_id,
    v.inventory_number,
    SUM(m.maintenance_cost) AS total_maintenance_cost

FROM vehicles AS v

LEFT JOIN maintenance AS m
    ON v.vehicle_id = m.vehicle_id

GROUP BY
    v.vehicle_id,
    v.inventory_number;
```

The total is calculated when the query executes.

A denormalized structure might maintain a separate summary:

**Vehicle maintenance summary**

| vehicle_id | total_maintenance_cost |
|---|---|
| 1 | 4700 |
| 2 | 5000 |

The summary makes certain reports easier to retrieve.

However, the total must be updated whenever relevant maintenance records change.

This creates an additional consistency responsibility.

## 13.3 Denormalization trade-offs

| Benefit | Cost |
|---|---|
| Faster access for selected queries | More duplicated data |
| Fewer JOIN operations | More complex updates |
| Precomputed aggregates | Additional maintenance logic |
| Read optimization | Potential inconsistencies |
| Reporting convenience | More storage |

### Important recommendation

Start with a logically correct normalized design.

Introduce denormalization when a measured performance or operational requirement justifies it.

Do not duplicate information simply because fewer tables appear easier to understand.

---

# 14. Normalization and Different Database Workloads

Normalization is especially relevant to transactional systems.

However, different database architectures may use different modeling approaches.

## 14.1 OLTP systems

Online Transaction Processing systems typically handle:

- Frequent INSERT operations.
- Frequent UPDATE operations.
- Frequent DELETE operations.
- Small, focused transactions.
- Multiple concurrent users.
- Operational consistency requirements.

Normalized relational structures are commonly suitable for these systems.

Examples:

- Banking transactions.
- Inventory management.
- Employee administration.
- Vehicle management.
- Order processing.

## 14.2 OLAP systems

Online Analytical Processing systems focus on:

- Large analytical queries.
- Aggregations.
- Historical analysis.
- Reporting.
- Trend identification.

Analytical data warehouses often use dimensional models, such as:

- Star schemas.
- Snowflake schemas.

These structures may intentionally organize data differently from a normalized transactional database.

Related note: [[04 - OLTP vs OLAP]].

### Important distinction

A star schema is not simply an incorrectly normalized OLTP database.

It is a deliberate analytical modeling approach designed for a different workload.

---

# 15. Applying Normalization to SIAPV

Consider a simplified operational system for managing vehicles and machinery.

The system needs to store:

- Equipment identification.
- Inventory information.
- Organizational assignments.
- Maintenance records.
- Mileage or operating-hour readings.
- Employee participation.

A poorly designed structure could combine everything into one large table.

Example:

| inventory_number | brand | department | maintenance_date | employee_name | maintenance_cost |
|---|---|---|---|---|---|
| INV-001 | Nissan | Administration | 2026-01-10 | Ana Lopez | 3500 |
| INV-001 | Nissan | Administration | 2026-03-15 | Luis Perez | 1200 |
| INV-002 | John Deere | Operations | 2026-02-20 | Ana Lopez | 5000 |

This structure repeats information and mixes multiple entities.

## 15.1 Identify the entities

A possible entity list:

- Vehicles.
- Departments.
- Employees.
- Maintenance.
- Maintenance assignments.
- Operational readings.

## 15.2 Identify functional dependencies

Possible dependencies:

```text
vehicle_id -> inventory_number, brand, model

department_id -> department_name

employee_id -> employee_name

maintenance_id -> vehicle_id, maintenance_date, maintenance_cost

(maintenance_id, employee_id) -> participation_role
```

These dependencies indicate which attributes belong together.

## 15.3 Separate the relations

A possible normalized structure:

```text
DEPARTMENTS
-----------
department_id (PK)
department_name

VEHICLES
--------
vehicle_id (PK)
inventory_number (AK)
brand
model
department_id (FK)

EMPLOYEES
---------
employee_id (PK)
employee_code (AK)
employee_name

MAINTENANCE
-----------
maintenance_id (PK)
vehicle_id (FK)
maintenance_date
maintenance_cost

MAINTENANCE_EMPLOYEES
---------------------
maintenance_id (PK, FK)
employee_id (PK, FK)
participation_role

OPERATIONAL_READINGS
--------------------
reading_id (PK)
vehicle_id (FK)
reading_date
reading_type
reading_value
```

## 15.4 Why separate operational readings?

Mileage and operating hours represent different types of measurements.

Depending on system requirements, they may be stored as:

- Separate attributes.
- Separate measurement tables.
- A common measurement table with a type and value.

The correct choice depends on whether the system needs a common measurement structure, different validation rules, historical tracking, or specialized reporting.

A single field should not mix unrelated units without a clear model.

## 15.5 Architecture considerations

When evaluating a system such as SIAPV, ask:

1. Is vehicle information stored independently of maintenance history?
2. Is department information maintained in one authoritative location?
3. Can multiple maintenance records reference the same vehicle?
4. Are employee assignments represented correctly?
5. Are operational readings stored with dates and measurement types?
6. Can historical information be preserved?
7. Are repeated values justified by the data model?
8. Would reporting requirements benefit from summary tables or views?

### Important principle

Normalization helps establish a reliable operational foundation.

Reporting and analytical needs can then be addressed through views, materialized views, reporting tables, or a separate analytical architecture when justified.

---

# 16. Common Normalization Mistakes

## Mistake 1: Thinking normalization means eliminating all repetition

Some repeated values are necessary to represent relationships.

For example, a foreign key value may appear in thousands of rows.

That does not automatically indicate a normalization problem.

**Better approach:** Evaluate whether the repeated value represents a necessary relationship or duplicated descriptive information.

## Mistake 2: Creating too many tables without understanding dependencies

Splitting every attribute into its own table does not automatically improve design.

**Better approach:** Use functional dependencies and business meaning to determine appropriate relations.

## Mistake 3: Confusing 1NF with one column per word

Atomicity depends on how an attribute is used.

A full name can be one valid attribute if the application treats it as one value.

**Better approach:** Define attribute boundaries according to the system's requirements.

## Mistake 4: Ignoring composite keys

2NF requires special attention to attributes that depend on only part of a composite candidate key.

**Better approach:** Identify all candidate keys and functional dependencies before evaluating 2NF.

## Mistake 5: Assuming every transitive dependency automatically violates 3NF

The formal definition of 3NF includes an exception for prime attributes.

**Better approach:** Evaluate candidate keys and prime attributes, especially in advanced examples.

## Mistake 6: Normalizing without considering queries

A theoretically clean design may still require expensive queries for a particular workload.

**Better approach:** Separate logical correctness from physical performance optimization.

## Mistake 7: Denormalizing without a consistency strategy

Duplicated values may become inconsistent.

**Better approach:** Define how duplicated information is updated and validated.

---

# 17. Practical Normalization Checklist

When reviewing a relational table, ask:

- [ ] Does each row represent one clearly defined record?
- [ ] Does every table have a candidate key?
- [ ] Are repeating groups represented appropriately?
- [ ] Are attributes atomic according to their intended use?
- [ ] Does the table satisfy 1NF?
- [ ] Are there partial dependencies on composite candidate keys?
- [ ] Does the table satisfy 2NF?
- [ ] Are non-key attributes transitively dependent on candidate keys?
- [ ] Does the table satisfy 3NF?
- [ ] Are all non-trivial determinants superkeys when evaluating BCNF?
- [ ] Does decomposition preserve the original information?
- [ ] Are functional dependencies enforceable?
- [ ] Is denormalization justified by an actual workload requirement?
- [ ] Are duplicated values maintained consistently?

---

# 18. Knowledge Check

1. What is database normalization?
2. What is data redundancy?
3. What are insertion, update, and deletion anomalies?
4. What does a functional dependency represent?
5. What is the determinant of a functional dependency?
6. What is the difference between a full and partial functional dependency?
7. What does First Normal Form require?
8. Why are repeating groups problematic?
9. What additional requirement does Second Normal Form introduce?
10. What is a transitive dependency?
11. What is the central requirement of Third Normal Form?
12. How is BCNF stricter than 3NF?
13. What is a lossless decomposition?
14. What does dependency preservation mean?
15. Why might an architect intentionally denormalize a database?
16. Why is normalization especially relevant to OLTP systems?
17. Why might an analytical data warehouse use a different modeling structure?

---

# 19. Practical Exercises

## Exercise 1 — Identify anomalies

Consider this table:

| order_id | customer_name | customer_email | product_name | quantity |
|---|---|---|---|---|
| 1001 | Ana Lopez | ana@example.com | Laptop | 1 |
| 1002 | Ana Lopez | ana@example.com | Mouse | 2 |
| 1003 | Luis Perez | luis@example.com | Monitor | 1 |

Identify:

- Repeated customer information.
- Possible insertion anomalies.
- Possible update anomalies.
- Possible deletion anomalies.

Explain which entities should be separated.

## Exercise 2 — Evaluate 1NF

Consider:

| vehicle_id | inventory_number | maintenance_types |
|---|---|---|
| 1 | INV-001 | Oil change, Tire inspection |
| 2 | INV-002 | Engine inspection |

Explain why this structure may violate 1NF.

Design a structure that represents maintenance activities as independent records.

## Exercise 3 — Evaluate 2NF

Consider a table with the following composite primary key:

```text
(order_id, product_id)
```

Attributes:

```text
order_date
product_name
quantity
unit_price
```

Assume these functional dependencies:

```text
order_id -> order_date

product_id -> product_name

(order_id, product_id) -> quantity
```

Identify partial dependencies and propose a decomposition.

Consider whether `unit_price` depends on the product, the order line, or a historical pricing rule. State your assumption before designing the solution.

## Exercise 4 — Evaluate 3NF

Consider:

```text
employee_id -> department_id

department_id -> department_name
```

Explain why storing `department_name` in every employee record creates a transitive dependency.

Propose a normalized structure.

## Exercise 5 — Evaluate BCNF

Consider:

```text
(employee_id, workshop_id) -> supervisor_id

workshop_id -> supervisor_id
```

Assume each employee can work in multiple workshops and each workshop has one supervisor.

Identify the candidate key and explain why the original relation violates BCNF.

Propose a decomposition.

## Exercise 6 — SIAPV normalization review

Design a normalized structure for a system that manages:

- Vehicles.
- Departments.
- Workshops.
- Maintenance activities.
- Employees.
- Mileage readings.
- Operating-hour readings.

For each table, identify:

- Primary key.
- Foreign keys.
- Important functional dependencies.
- Potential candidate keys.
- Attributes that should not be duplicated unnecessarily.

Explain which tables may require historical records.

---

# 20. Key Takeaways

1. Normalization organizes relational data according to structural rules.
2. Data redundancy can create insertion, update, and deletion anomalies.
3. Functional dependencies describe how attributes determine other attributes.
4. 1NF focuses on atomic values and eliminating repeating groups.
5. 2NF eliminates partial dependencies on candidate keys.
6. 3NF addresses disallowed transitive dependencies.
7. BCNF requires every determinant of a non-trivial functional dependency to be a superkey.
8. Decomposition should preserve information.
9. Dependency preservation makes integrity rules easier to enforce.
10. Denormalization can improve selected workloads but introduces consistency responsibilities.
11. OLTP systems commonly benefit from normalized relational structures.
12. Analytical architectures may intentionally use different modeling strategies.
13. Normalization is a design method, not an objective to maximize without considering the system's requirements.

---

# 21. Technical Vocabulary

| English term | Spanish equivalent |
|---|---|
| Normalization | Normalización |
| Normal form | Forma normal |
| Data redundancy | Redundancia de datos |
| Insertion anomaly | Anomalía de inserción |
| Update anomaly | Anomalía de actualización |
| Deletion anomaly | Anomalía de eliminación |
| Functional dependency | Dependencia funcional |
| Determinant | Determinante |
| Dependent attribute | Atributo dependiente |
| Full functional dependency | Dependencia funcional completa |
| Partial dependency | Dependencia parcial |
| Transitive dependency | Dependencia transitiva |
| Prime attribute | Atributo primo |
| First Normal Form | Primera forma normal |
| Second Normal Form | Segunda forma normal |
| Third Normal Form | Tercera forma normal |
| Boyce-Codd Normal Form | Forma normal de Boyce-Codd |
| Decomposition | Descomposición |
| Lossless decomposition | Descomposición sin pérdida |
| Dependency preservation | Preservación de dependencias |
| Denormalization | Desnormalización |
| Read-heavy workload | Carga de trabajo orientada a lectura |
| Transactional workload | Carga de trabajo transaccional |
| Relational schema | Esquema relacional |

---

# 22. Related Learning Notes

## Data Modeling

- [[01 - Entities and Attributes]]
- [[02 - Relationships and Cardinality]]
- [[03 - Keys and Constraints]]
- [[04 - Conceptual Logical and Physical Models]]
- [[06 - Schema Design Principles]]

## Database Foundations

- [[04 - OLTP vs OLAP]]
- [[05 - Database Architecture Fundamentals]]

## Database Internals

- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[04 - Indexes]]
- [[05 - Transactions and ACID]]

## Applied Architecture

- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]
- [[03 - Architecture Decision Framework]]

---

# 23. Next Learning Step

The next topic is **Schema Design Principles**.

We will bring together the concepts from the Data Modeling section and study how to design complete relational schemas.

The next document will cover:

- Naming conventions.
- Table and column design.
- Choosing appropriate data types.
- Modeling historical information.
- Referential design.
- Constraints and validation.
- Schema organization.
- Maintainability and extensibility.
- Common architectural decisions.
