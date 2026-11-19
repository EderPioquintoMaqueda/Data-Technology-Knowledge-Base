
---
title: "Keys and Constraints"
aliases:
  - "Database Keys"
  - "Relational Keys"
  - "Database Constraints"
  - "Keys and Integrity Constraints"
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
  - relational-databases
  - database-keys
  - primary-key
  - foreign-key
  - constraints
  - data-integrity
  - postgresql
concepts:
  - candidate-key
  - superkey
  - primary-key
  - alternate-key
  - natural-key
  - surrogate-key
  - composite-key
  - foreign-key
  - unique-constraint
  - not-null-constraint
  - check-constraint
  - default-constraint
  - referential-integrity
  - entity-integrity
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - What is a Database System]]"
  - "[[01 - Entities and Attributes]]"
  - "[[02 - Relationships and Cardinality]]"
related:
  - "[[04 - Conceptual Logical and Physical Models]]"
  - "[[05 - Normalization]]"
  - "[[06 - Schema Design Principles]]"
  - "[[01 - Database Management Systems]]"
  - "[[05 - Transactions and ACID]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
  - "[[01 - SIAPV Data Architecture]]"
tools:
  - PostgreSQL
  - SQL
  - pgAdmin
learning_objectives:
  - "Understand the purpose of keys in relational databases."
  - "Distinguish between candidate, primary, alternate, natural, surrogate, and composite keys."
  - "Understand how foreign keys establish relationships between tables."
  - "Apply integrity constraints to protect data quality."
  - "Design primary keys and foreign keys using PostgreSQL."
  - "Understand referential actions and their consequences."
  - "Evaluate key and constraint decisions in a practical database."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 03 — Keys and Constraints

## 1. Introduction

A relational database needs more than tables and columns.

It also needs rules that determine:

- How each record is identified.
- How records relate to other records.
- Which values are allowed.
- Which values must be unique.
- Which fields cannot be empty.
- What happens when related records are updated or deleted.

These rules are implemented through **keys and constraints**.

Keys identify records and connect tables.

Constraints enforce rules that protect the integrity of stored information.

Without these mechanisms, a database can contain duplicate records, invalid references, missing identifiers, and values that violate business requirements.

### Central principle

A database should not depend exclusively on application code or users to maintain correct information.

The database itself should enforce essential integrity rules.

---

# 2. What Is a Database Key?

A **key** is an attribute or combination of attributes used to identify records or establish relationships between tables.

Consider a vehicle table:

| vehicle_id | inventory_number | brand | model |
|---|---|---|---|
| 1 | INV-001 | Nissan | Frontier |
| 2 | INV-002 | John Deere | 310L |
| 3 | INV-003 | Shantui | SG18 |

Each row represents a different vehicle.

The database needs a reliable way to distinguish one vehicle from another.

The `vehicle_id` column can provide that identification.

A key helps the database:

1. Identify individual records.
2. Prevent duplicate identifiers.
3. Establish relationships between tables.
4. Maintain consistency when information changes.
5. Support queries and data retrieval.

### Why are keys necessary?

Imagine a database without a reliable identifier.

Two vehicles could have:

- The same brand.
- The same model.
- The same year.
- The same color.

How would the database distinguish them?

A reliable key solves this problem.

---

# 3. Superkeys and Candidate Keys

Before understanding primary keys, it is useful to understand two related concepts.

## 3.1 Superkey

A **superkey** is any attribute or combination of attributes that uniquely identifies a record.

For example, consider:

| employee_id | email | full_name |
|---|---|---|
| 101 | ana@example.com | Ana López |
| 102 | luis@example.com | Luis Pérez |
| 103 | carlos@example.com | Carlos Ruiz |

Assuming employee IDs and email addresses are unique:

Possible superkeys include:

- `employee_id`
- `email`
- `employee_id + full_name`
- `email + full_name`
- `employee_id + email`

All these combinations can identify a record uniquely.

However, some combinations contain unnecessary attributes.

For example, `employee_id + full_name` is unique because `employee_id` is already unique.

The additional `full_name` does not improve identification.

## 3.2 Candidate key

A **candidate key** is a minimal superkey.

It uniquely identifies a record without unnecessary attributes.

For the previous example:

- `employee_id` is a candidate key.
- `email` is another candidate key.

But `employee_id + full_name` is not a candidate key because it contains an unnecessary attribute.

### Difference

| Concept | Meaning |
|---|---|
| Superkey | Any combination that uniquely identifies a record |
| Candidate key | A minimal combination that uniquely identifies a record |

A table can have multiple candidate keys.

The database designer evaluates them before selecting the primary key.

---

# 4. Primary Key

A **primary key (PK)** is the candidate key selected as the main identifier for a table.

Every relational table should have a clearly defined primary key.

Example:

```sql
CREATE TABLE employees (
    employee_id INTEGER PRIMARY KEY,
    full_name VARCHAR(120) NOT NULL,
    email VARCHAR(150) UNIQUE
);
```

In this example:

- `employee_id` identifies each employee.
- `PRIMARY KEY` enforces uniqueness.
- A primary key cannot contain NULL values.
- The table can have only one primary key definition.

A primary key can contain one column or multiple columns.

### Primary key characteristics

A primary key must:

1. Uniquely identify every record.
2. Never contain NULL values.
3. Remain stable over time whenever possible.
4. Be referenced by other tables when relationships require it.

### Why should a primary key remain stable?

Suppose a vehicle's inventory number changes because the organization updates its administrative identification system.

If the inventory number is also the primary key, changing it may affect every table that references it.

Using a separate internal identifier can prevent this dependency.

---

# 5. Alternate Keys

An **alternate key** is a candidate key that was not selected as the primary key.

Consider:

```sql
CREATE TABLE employees (
    employee_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(150) NOT NULL UNIQUE,
    employee_code VARCHAR(30) NOT NULL UNIQUE,
    full_name VARCHAR(120) NOT NULL
);
```

Here:

- `employee_id` is the primary key.
- `email` is an alternate key.
- `employee_code` is another alternate key.

The database can use these alternate identifiers to locate employees.

In practice, alternate keys are frequently implemented through `UNIQUE` constraints.

### Important distinction

The primary key is the principal identifier.

Alternate keys are other attributes or combinations that must also remain unique according to business rules.

---

# 6. Natural Keys

A **natural key** is an identifier that already exists in the real-world domain.

Examples include:

- A government-issued identification number.
- An official product code.
- An institutional employee number.
- A registration number.
- An inventory number.

Consider a vehicle inventory system:

| inventory_number | brand | model |
|---|---|---|
| INV-001 | Nissan | Frontier |
| INV-002 | John Deere | 310L |

The inventory number may be a natural key because it has meaning outside the database.

### Advantages

- It has business meaning.
- Users may already know the identifier.
- It can avoid creating an additional identifier in simple systems.
- It can represent an existing organizational rule.

### Disadvantages

Natural keys can be problematic when they:

- Change over time.
- Are too long.
- Contain multiple attributes.
- Depend on external organizations.
- Are not consistently assigned.
- Have different formats across systems.

### Example: License plates

A vehicle's license plate may appear to be a suitable primary key.

However, plates can change.

Vehicles can also have administrative circumstances that make their registration information temporarily unavailable.

Therefore, using a license plate as the only permanent database identifier may create unnecessary dependencies.

A natural identifier can still be stored and protected with a `UNIQUE` constraint when the business rules require it.

---

# 7. Surrogate Keys

A **surrogate key** is an artificial identifier created specifically for database identification.

It has no direct business meaning.

Example:

| vehicle_id | inventory_number | brand |
|---|---|---|
| 1 | INV-001 | Nissan |
| 2 | INV-002 | John Deere |
| 3 | INV-003 | Shantui |

The `vehicle_id` is a surrogate key.

The `inventory_number` is a natural identifier.

### PostgreSQL identity columns

PostgreSQL supports identity columns for generating identifiers automatically.

Example:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    inventory_number VARCHAR(30) NOT NULL UNIQUE,
    brand VARCHAR(80) NOT NULL,
    model VARCHAR(80) NOT NULL
);
```

The database generates the value of `vehicle_id` when a record is inserted.

Two identity generation modes are available:

**GENERATED ALWAYS AS IDENTITY**

The database normally generates the identifier. Explicit values require special handling.

**GENERATED BY DEFAULT AS IDENTITY**

The database generates a value when one is not explicitly supplied. An explicit value can be inserted when necessary.

### Advantages of surrogate keys

- They are independent of business attributes.
- They are usually short and simple.
- They remain stable when descriptive information changes.
- They simplify relationships between tables.
- They are useful when natural identifiers are complicated.

### Disadvantages

- They do not communicate business meaning.
- They require an additional column.
- They do not automatically prevent duplicate business identifiers.

For example, generating a unique `vehicle_id` does not prevent two vehicles from receiving the same inventory number.

That is why both identifiers may be necessary.

### Natural key vs surrogate key

| Characteristic | Natural key | Surrogate key |
|---|---|---|
| Origin | Real-world business domain | Database |
| Business meaning | Yes | Usually no |
| Stability | Depends on the domain | Usually high |
| Example | Inventory number | vehicle_id |
| Main purpose | Business identification | Internal record identification |

### Practical recommendation

A common relational design uses:

- A surrogate primary key for internal identification.
- A natural key protected by a `UNIQUE` constraint when the business requires uniqueness.

This is not an absolute rule. The correct choice depends on the stability, meaning, and requirements of the data.

---

# 8. Composite Keys

A **composite key** is a key formed by two or more columns.

It is useful when no single column can uniquely identify a record, but a combination can.

Consider a table that associates employees with maintenance activities.

| maintenance_id | employee_id | role |
|---|---|---|
| 501 | 101 | Technician |
| 501 | 102 | Supervisor |
| 502 | 101 | Technician |

An employee can participate in multiple maintenance activities.

A maintenance activity can involve multiple employees.

The combination of `maintenance_id` and `employee_id` can uniquely identify each assignment.

Example:

```sql
CREATE TABLE maintenance_employees (
    maintenance_id BIGINT NOT NULL,
    employee_id BIGINT NOT NULL,
    role VARCHAR(50),

    PRIMARY KEY (maintenance_id, employee_id)
);
```

The primary key is composed of:

```text
maintenance_id + employee_id
```

This prevents the same employee from being assigned to the same maintenance activity more than once.

### When are composite keys useful?

- Junction tables.
- Many-to-many relationships.
- Historical records with multiple identifying dimensions.
- Business records whose uniqueness depends on a combination of attributes.

### Important consideration

Composite keys can make relationships more complicated when other tables need to reference them.

For that reason, database designers should evaluate whether a composite primary key or a surrogate primary key with an additional `UNIQUE` constraint is more appropriate.

---

# 9. Foreign Keys

A **foreign key (FK)** is an attribute or combination of attributes that references a candidate key, usually the primary key, in another table.

Foreign keys establish relationships between tables.

Consider:

**Departments**

| department_id | department_name |
|---|---|
| 1 | Administration |
| 2 | Operations |

**Employees**

| employee_id | full_name | department_id |
|---|---|---|
| 101 | Ana López | 1 |
| 102 | Luis Pérez | 2 |
| 103 | Carlos Ruiz | 1 |

The `department_id` column in the employees table references the `department_id` column in departments.

Example:

```sql
CREATE TABLE departments (
    department_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE employees (
    employee_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name VARCHAR(120) NOT NULL,
    department_id BIGINT NOT NULL,

    CONSTRAINT fk_employee_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

### What does the foreign key guarantee?

It prevents an employee from referencing a department that does not exist.

For example:

```sql
INSERT INTO employees (full_name, department_id)
VALUES ('Ana López', 999);
```

If department `999` does not exist, PostgreSQL rejects the insertion.

This is called **referential integrity**.

### Important distinction

A foreign key does not mean that the referenced value must be unique in the referencing table.

Many employees can reference the same department.

This is how a one-to-many relationship is implemented.

---

# 10. What Are Database Constraints?

A **constraint** is a rule enforced by the database to restrict which data can be stored.

Constraints protect data integrity.

The most common relational constraints include:

| Constraint | Purpose |
|---|---|
| PRIMARY KEY | Uniquely identifies records |
| FOREIGN KEY | Maintains valid references |
| UNIQUE | Prevents duplicate values or combinations |
| NOT NULL | Requires a value |
| CHECK | Enforces a logical condition |
| DEFAULT | Supplies a value when one is omitted |

Constraints are part of the database schema.

They are not merely suggestions for application developers.

---

# 11. PRIMARY KEY Constraint

A primary key combines two important rules:

- Uniqueness.
- Non-nullability.

Example:

```sql
CREATE TABLE products (
    product_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_name VARCHAR(120) NOT NULL
);
```

The following operations are rejected:

- Inserting a duplicate `product_id`.
- Inserting NULL into `product_id`.

A primary key is essential for reliable record identification.

---

# 12. UNIQUE Constraint

The `UNIQUE` constraint prevents duplicate values in a column or combination of columns.

Example:

```sql
CREATE TABLE users (
    user_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(150) UNIQUE
);
```

Two users cannot have the same non-NULL email value.

### UNIQUE vs PRIMARY KEY

| Characteristic | PRIMARY KEY | UNIQUE |
|---|---|---|
| Uniqueness | Required | Required |
| NULL values | Not allowed | Allowed by default |
| Number per table | One primary key definition | Multiple unique constraints |
| Main purpose | Main record identifier | Enforce additional uniqueness rules |

### PostgreSQL and NULL values

By default, PostgreSQL treats NULL values as distinct for a regular `UNIQUE` constraint.

This means multiple rows can contain NULL in a column protected by `UNIQUE`.

Example:

```sql
CREATE TABLE users (
    user_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(150) UNIQUE
);
```

Multiple users may have no email address.

However, two users cannot have the same non-NULL email.

PostgreSQL also supports:

```sql
UNIQUE NULLS NOT DISTINCT (email)
```

This treats NULL values as equal for uniqueness purposes, so only one NULL is allowed in that constrained key.

Use this behavior only when it matches the business rule.

---

# 13. NOT NULL Constraint

`NOT NULL` requires a column to contain a value.

Example:

```sql
CREATE TABLE employees (
    employee_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name VARCHAR(120) NOT NULL,
    hire_date DATE NOT NULL
);
```

The database rejects an insertion when `full_name` or `hire_date` is NULL.

### NULL is not the same as an empty string

These values are different:

| Value | Meaning |
|---|---|
| NULL | Missing or unknown value |
| `''` | Empty string |
| `' '` | String containing a space |
| `'Unknown'` | Literal text |

A `NOT NULL` constraint does not reject an empty string.

If empty strings must also be rejected, an additional `CHECK` constraint may be needed.

Example:

```sql
full_name VARCHAR(120) NOT NULL
    CHECK (length(trim(full_name)) > 0)
```

---

# 14. CHECK Constraint

A `CHECK` constraint enforces a logical condition on a column or a combination of columns.

Example:

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    inventory_number VARCHAR(30) NOT NULL UNIQUE,
    mileage NUMERIC(12, 2) NOT NULL CHECK (mileage >= 0),
    acquisition_year INTEGER CHECK (acquisition_year >= 1900)
);
```

The database rejects negative mileage values.

### More examples

```sql
CHECK (maintenance_cost >= 0)
```

```sql
CHECK (quantity > 0)
```

```sql
CHECK (end_date >= start_date)
```

```sql
CHECK (status IN ('ACTIVE', 'INACTIVE', 'RETIRED'))
```

### Important NULL behavior

A PostgreSQL `CHECK` constraint rejects a row when its condition evaluates to FALSE.

It accepts a condition that evaluates to TRUE or NULL.

For example:

```sql
CHECK (mileage >= 0)
```

does not prevent `mileage` from being NULL.

To require both a non-null value and a non-negative value, use:

```sql
mileage NUMERIC(12, 2) NOT NULL
    CHECK (mileage >= 0)
```

### CHECK constraints and business rules

A constraint should express a rule that must always be true for the stored record.

For example:

- Mileage cannot be negative.
- Maintenance cost cannot be negative.
- A quantity must be greater than zero.
- An end date cannot precede a start date.

However, CHECK constraints are not designed to enforce arbitrary rules involving other rows or tables.

Those requirements may need foreign keys, unique indexes, triggers, or application-level transaction logic.

---

# 15. DEFAULT Constraint

A `DEFAULT` value is automatically supplied when an INSERT statement omits that column.

Example:

```sql
CREATE TABLE maintenance (
    maintenance_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    status VARCHAR(30) NOT NULL DEFAULT 'PENDING',
    created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

When a maintenance record is inserted without a status, PostgreSQL assigns `PENDING`.

When `created_at` is omitted, PostgreSQL uses the current timestamp.

### Important distinction

A default value is not a validation rule.

It does not prevent a user from explicitly inserting a different value.

Also, explicitly inserting NULL does not activate the default.

For example:

```sql
INSERT INTO maintenance (status)
VALUES (NULL);
```

This statement is rejected because `status` is `NOT NULL`.

The default is only applied when the column is omitted or when the SQL statement explicitly requests `DEFAULT`.

---

# 16. Referential Actions

A foreign key can define what happens when the referenced record is updated or deleted.

These behaviors are called **referential actions**.

Consider:

```sql
FOREIGN KEY (department_id)
REFERENCES departments(department_id)
ON DELETE RESTRICT
```

The action determines how PostgreSQL handles a deletion when related records exist.

## 16.1 NO ACTION

This is the default behavior.

The database rejects an operation that would violate the foreign key when the constraint is checked.

```sql
ON DELETE NO ACTION
```

It is similar to RESTRICT in ordinary immediate-constraint situations, but it can differ when constraints are deferrable.

## 16.2 RESTRICT

Prevents deleting a referenced record while dependent records exist.

```sql
ON DELETE RESTRICT
```

Useful when deleting the parent record must not proceed while references remain.

## 16.3 CASCADE

Automatically deletes dependent records when the referenced parent record is deleted.

```sql
ON DELETE CASCADE
```

Example:

If a department is deleted, all employees referencing that department may also be deleted.

This can be useful for dependent records such as certain junction-table assignments.

However, it can be dangerous when applied to important business records.

A single deletion can affect many related records.

## 16.4 SET NULL

Sets the foreign key to NULL when the referenced record is deleted.

```sql
ON DELETE SET NULL
```

Useful when the relationship is optional.

The foreign key column must allow NULL.

## 16.5 SET DEFAULT

Sets the foreign key to its default value when the referenced record is deleted.

```sql
ON DELETE SET DEFAULT
```

The default value must still satisfy the foreign key constraint.

### Referential actions comparison

| Action | Result |
|---|---|
| NO ACTION | Rejects the operation if the constraint is violated when checked |
| RESTRICT | Immediately prevents the referenced record from being deleted |
| CASCADE | Propagates deletion or update to dependent records |
| SET NULL | Sets the referencing column to NULL |
| SET DEFAULT | Assigns the referencing column its default value |

### ON UPDATE

Similar actions can be configured for updates:

```sql
ON UPDATE CASCADE
```

This propagates changes to the referenced key into referencing records.

For stable surrogate primary keys, changing the referenced key should be unusual.

### Design recommendation

Choose referential actions according to the business meaning of the relationship.

Do not use `CASCADE` automatically.

Deleting an employee should not necessarily delete maintenance records that document work already performed.

Historical and administrative information often needs to remain available.

---

# 17. Types of Data Integrity

Keys and constraints protect several dimensions of database integrity.

## 17.1 Entity integrity

Entity integrity guarantees that each row can be uniquely identified.

It is primarily enforced through primary keys.

Requirements:

- A primary key must be unique.
- A primary key cannot contain NULL.
- Every record must have a valid identifier.

## 17.2 Referential integrity

Referential integrity guarantees that relationships between tables remain valid.

It is enforced through foreign keys.

Example:

An employee cannot reference a department that does not exist.

## 17.3 Domain integrity

Domain integrity guarantees that values follow the rules defined for a particular attribute.

It can be enforced through:

- Data types.
- NOT NULL.
- CHECK.
- DEFAULT.
- UNIQUE.

Examples:

- Mileage must be non-negative.
- An email must be unique when provided.
- A maintenance status must belong to an allowed set.
- A date column must contain a valid date.

### Integrity overview

| Integrity type | Main concern | Typical mechanism |
|---|---|---|
| Entity | Record identity | PRIMARY KEY |
| Referential | Valid relationships | FOREIGN KEY |
| Domain | Valid attribute values | Data types and constraints |

These mechanisms work together to keep a database consistent.

---

# 18. Indexes Created by Keys and Constraints in PostgreSQL

Keys and indexes are related, but they are not the same concept.

An **index** is a data structure that helps PostgreSQL locate records more efficiently.

A key is a logical identifier or relationship rule.

Some constraints automatically create indexes.

### PRIMARY KEY

When a primary key is defined, PostgreSQL automatically creates a unique B-tree index to enforce it.

### UNIQUE

A `UNIQUE` constraint also creates a unique index.

### FOREIGN KEY

A foreign key does not automatically create an index on the referencing columns in PostgreSQL.

This is an important design consideration.

Example:

```sql
CREATE TABLE employees (
    employee_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_id BIGINT NOT NULL REFERENCES departments(department_id)
);
```

PostgreSQL protects referential integrity, but it does not automatically create an index on `employees.department_id`.

Depending on query patterns and data volume, an index may be useful:

```sql
CREATE INDEX idx_employees_department_id
ON employees(department_id);
```

### Why index foreign key columns?

Indexes can improve:

- JOIN operations.
- Searches by foreign key.
- Updates to referenced keys.
- Deletes from referenced tables when PostgreSQL checks dependent records.

### Important distinction

Adding an index to every foreign key is not an automatic requirement.

Indexes consume storage and add overhead to INSERT, UPDATE, and DELETE operations.

The decision should consider actual query patterns and workload.

Related note: [[04 - Indexes]].

---

# 19. Complete PostgreSQL Example

The following example combines the concepts covered in this document.

It represents a simplified vehicle and maintenance management system.

The schema contains:

1. Departments.
2. Employees.
3. Vehicles.
4. Maintenance records.
5. Employee assignments to maintenance activities.

The example is illustrative and does not assume that SIAPV uses this exact database structure.

## 19.1 Create departments

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

Key concepts:

- Surrogate primary key.
- Identity column.
- NOT NULL.
- UNIQUE.

## 19.2 Create employees

```sql
CREATE TABLE employees (
    employee_id BIGINT GENERATED ALWAYS AS IDENTITY,

    employee_code VARCHAR(30) NOT NULL,

    full_name VARCHAR(120) NOT NULL,

    email VARCHAR(150),

    department_id BIGINT NOT NULL,

    hire_date DATE NOT NULL DEFAULT CURRENT_DATE,

    CONSTRAINT pk_employees
        PRIMARY KEY (employee_id),

    CONSTRAINT uq_employees_code
        UNIQUE (employee_code),

    CONSTRAINT uq_employees_email
        UNIQUE (email),

    CONSTRAINT fk_employees_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE RESTRICT
);
```

Key concepts:

- Primary key.
- Alternate candidate keys.
- Foreign key.
- Referential integrity.
- Default date.
- Optional email.

The email column allows NULL because it is not declared NOT NULL.

## 19.3 Create vehicles

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY,

    inventory_number VARCHAR(30) NOT NULL,

    brand VARCHAR(80) NOT NULL,

    model VARCHAR(80) NOT NULL,

    acquisition_year INTEGER,

    mileage NUMERIC(12, 2) NOT NULL DEFAULT 0,

    department_id BIGINT NOT NULL,

    status VARCHAR(30) NOT NULL DEFAULT 'ACTIVE',

    CONSTRAINT pk_vehicles
        PRIMARY KEY (vehicle_id),

    CONSTRAINT uq_vehicles_inventory
        UNIQUE (inventory_number),

    CONSTRAINT chk_vehicles_mileage
        CHECK (mileage >= 0),

    CONSTRAINT chk_vehicles_year
        CHECK (
            acquisition_year IS NULL
            OR acquisition_year >= 1900
        ),

    CONSTRAINT chk_vehicles_status
        CHECK (
            status IN ('ACTIVE', 'INACTIVE', 'RETIRED')
        ),

    CONSTRAINT fk_vehicles_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE RESTRICT
);
```

Key concepts:

- Surrogate primary key.
- Natural business identifier.
- Unique constraint.
- CHECK constraints.
- DEFAULT values.
- Foreign key.

The inventory number is unique, but it is not the primary key.

This allows the organization to change an inventory number without changing the internal vehicle identifier.

## 19.4 Create maintenance records

```sql
CREATE TABLE maintenance (
    maintenance_id BIGINT GENERATED ALWAYS AS IDENTITY,

    vehicle_id BIGINT NOT NULL,

    maintenance_date DATE NOT NULL DEFAULT CURRENT_DATE,

    description TEXT NOT NULL,

    maintenance_cost NUMERIC(12, 2) NOT NULL DEFAULT 0,

    status VARCHAR(30) NOT NULL DEFAULT 'PENDING',

    CONSTRAINT pk_maintenance
        PRIMARY KEY (maintenance_id),

    CONSTRAINT chk_maintenance_cost
        CHECK (maintenance_cost >= 0),

    CONSTRAINT chk_maintenance_status
        CHECK (
            status IN ('PENDING', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED')
        ),

    CONSTRAINT fk_maintenance_vehicle
        FOREIGN KEY (vehicle_id)
        REFERENCES vehicles(vehicle_id)
        ON DELETE RESTRICT
);
```

Key concepts:

- Maintenance primary key.
- Vehicle foreign key.
- Non-negative cost.
- Allowed status values.
- Default date and status.

A vehicle can have multiple maintenance records.

## 19.5 Create the junction table

```sql
CREATE TABLE maintenance_employees (
    maintenance_id BIGINT NOT NULL,

    employee_id BIGINT NOT NULL,

    role VARCHAR(50) NOT NULL DEFAULT 'TECHNICIAN',

    CONSTRAINT pk_maintenance_employees
        PRIMARY KEY (maintenance_id, employee_id),

    CONSTRAINT fk_maintenance_employees_maintenance
        FOREIGN KEY (maintenance_id)
        REFERENCES maintenance(maintenance_id)
        ON DELETE CASCADE,

    CONSTRAINT fk_maintenance_employees_employee
        FOREIGN KEY (employee_id)
        REFERENCES employees(employee_id)
        ON DELETE RESTRICT
);
```

Key concepts:

- Many-to-many relationship.
- Composite primary key.
- Two foreign keys.
- Referential actions.

The `CASCADE` action is applied to assignments when a maintenance record is deleted.

The employee deletion is restricted while assignments still exist.

In a real operational system, maintenance history may need to be preserved permanently or archived rather than deleted. The referential action should reflect that requirement.

## 19.6 Insert sample data

```sql
INSERT INTO departments (department_name)
VALUES
    ('Administration'),
    ('Operations');
```

```sql
INSERT INTO employees (
    employee_code,
    full_name,
    email,
    department_id
)
VALUES
    ('EMP-001', 'Ana Lopez', 'ana@example.com', 1),
    ('EMP-002', 'Luis Perez', 'luis@example.com', 2);
```

```sql
INSERT INTO vehicles (
    inventory_number,
    brand,
    model,
    acquisition_year,
    department_id
)
VALUES
    ('INV-001', 'Nissan', 'Frontier', 2022, 1),
    ('INV-002', 'John Deere', '310L', 2021, 2);
```

```sql
INSERT INTO maintenance (
    vehicle_id,
    description,
    maintenance_cost
)
VALUES
    (1, 'Preventive maintenance', 3500.00),
    (2, 'Hydraulic system inspection', 5200.00);
```

```sql
INSERT INTO maintenance_employees (
    maintenance_id,
    employee_id,
    role
)
VALUES
    (1, 1, 'Technician'),
    (1, 2, 'Supervisor'),
    (2, 2, 'Technician');
```

## 19.7 Validate the relationships

```sql
SELECT
    m.maintenance_id,
    v.inventory_number,
    v.brand,
    m.maintenance_date,
    m.description,
    e.full_name,
    me.role

FROM maintenance AS m

INNER JOIN vehicles AS v
    ON m.vehicle_id = v.vehicle_id

INNER JOIN maintenance_employees AS me
    ON m.maintenance_id = me.maintenance_id

INNER JOIN employees AS e
    ON me.employee_id = e.employee_id

ORDER BY m.maintenance_id;
```

This query demonstrates how primary keys and foreign keys allow data from different tables to be connected.

---

# 20. Applying Keys and Constraints to SIAPV

A vehicle and machinery management system needs reliable identifiers and integrity rules.

Consider a simplified inventory record.

| Attribute | Possible role |
|---|---|
| vehicle_id | Surrogate primary key |
| inventory_number | Natural business identifier |
| brand | Descriptive attribute |
| model | Descriptive attribute |
| mileage | Operational attribute |
| department_id | Foreign key |
| status | Controlled domain |

The following design decisions are worth evaluating.

## 20.1 Internal identifier

A stable internal identifier can distinguish each registered unit.

Example:

```sql
vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

This identifier should not depend on a mutable administrative field.

## 20.2 Unique inventory number

If each inventory number must identify only one active database record, a uniqueness rule can be applied.

```sql
inventory_number VARCHAR(30) NOT NULL UNIQUE
```

The exact rule must reflect how the organization handles retired equipment, transferred units, and historical identifiers.

## 20.3 Mileage validation

A vehicle should not have negative mileage.

```sql
mileage NUMERIC(12, 2) NOT NULL
    CHECK (mileage >= 0)
```

## 20.4 Maintenance cost validation

A maintenance cost should not be negative.

```sql
maintenance_cost NUMERIC(12, 2) NOT NULL
    CHECK (maintenance_cost >= 0)
```

## 20.5 Department relationships

A vehicle can reference the department responsible for its administrative or operational assignment.

```sql
FOREIGN KEY (department_id)
REFERENCES departments(department_id)
```

The correct relationship depends on the real organizational model.

An administrative assignment and an operational location may be separate concepts. They should not be represented by one foreign key if they have different meanings.

## 20.6 Historical information

Maintenance records, insurance information, mileage reports, and incidents may need to remain available even when a vehicle is retired or transferred.

Therefore, deletion rules should protect historical information.

A status change or an archival process may be more appropriate than deleting a vehicle.

### SIAPV design principle

The database should distinguish between:

- The permanent identity of a unit.
- Its administrative inventory number.
- Its current organizational assignment.
- Its operational status.
- Its historical events.

These concepts may be related, but they are not interchangeable.

---

# 21. Common Database Design Mistakes

## Mistake 1: Creating tables without primary keys

Without a reliable identifier, records can become difficult to reference and manage.

**Better approach:** Define a primary key for every entity table.

## Mistake 2: Using mutable business information as the only primary key

An email, license plate, or administrative code may change.

**Better approach:** Evaluate whether a stable surrogate identifier is more appropriate.

## Mistake 3: Assuming a surrogate key prevents business duplicates

A generated ID does not prevent duplicate inventory numbers or email addresses.

**Better approach:** Add UNIQUE constraints for business identifiers that must remain unique.

## Mistake 4: Using foreign keys without understanding their meaning

A foreign key should represent a genuine relationship.

**Better approach:** Define the business rule before creating the relationship.

## Mistake 5: Applying CASCADE everywhere

A parent deletion may unintentionally remove important historical records.

**Better approach:** Choose referential actions according to the lifecycle and ownership of the data.

## Mistake 6: Assuming CHECK prevents NULL

A CHECK condition can evaluate to NULL and still pass.

**Better approach:** Combine CHECK with NOT NULL when both rules are necessary.

## Mistake 7: Forgetting indexes on foreign key columns

PostgreSQL does not automatically index the referencing side of a foreign key.

**Better approach:** Evaluate indexes based on joins, filtering, updates, deletes, and workload.

## Mistake 8: Enforcing every rule only in the application

An application may contain validation errors or another application may access the database directly.

**Better approach:** Enforce essential integrity rules in the database itself.

---

# 22. Practical Design Checklist

Before creating a relational table, evaluate the following:

- [ ] Does the table represent a clearly defined entity?
- [ ] Does it have a primary key?
- [ ] Is the primary key stable?
- [ ] Are there natural identifiers that require UNIQUE constraints?
- [ ] Are foreign keys defined for relationships?
- [ ] Are required attributes declared NOT NULL?
- [ ] Are valid value ranges enforced with CHECK?
- [ ] Are appropriate default values defined?
- [ ] Are referential actions consistent with business requirements?
- [ ] Are indexes needed for foreign key columns?
- [ ] Are historical records protected from accidental deletion?
- [ ] Can the database reject invalid data independently of the application?

---

# 23. Knowledge Check

Answer these questions without looking at the explanations.

1. What is the difference between a superkey and a candidate key?
2. What makes an attribute a primary key?
3. What is an alternate key?
4. What is the difference between a natural key and a surrogate key?
5. When would a composite key be appropriate?
6. What problem does a foreign key solve?
7. What is the difference between PRIMARY KEY and UNIQUE?
8. Why can a UNIQUE column contain multiple NULL values in PostgreSQL by default?
9. What is the purpose of a CHECK constraint?
10. Why should CHECK sometimes be combined with NOT NULL?
11. What is the difference between CASCADE and RESTRICT?
12. Does PostgreSQL automatically create an index on a foreign key column?
13. Why might a vehicle inventory number be better as an alternate key than as the primary key?
14. What is entity integrity?
15. What is referential integrity?

---

# 24. Practical Exercises

## Exercise 1 — Identify the keys

Consider the following table:

| student_id | institutional_email | full_name | enrollment_code |
|---|---|---|---|
| 1 | ana@university.edu | Ana Lopez | ENR-001 |
| 2 | luis@university.edu | Luis Perez | ENR-002 |
| 3 | carlos@university.edu | Carlos Ruiz | ENR-003 |

Assume all identifiers are unique.

Identify:

- The candidate keys.
- The primary key.
- The alternate keys.
- Possible superkeys.

Explain why some combinations are not minimal candidate keys.

## Exercise 2 — Design a vehicle table

Create a PostgreSQL table named `equipment` with:

- A generated primary key.
- A unique inventory number.
- A required brand.
- A required model.
- A non-negative operating-hour value.
- An optional acquisition date.
- A default status of `ACTIVE`.

Use appropriate data types and constraints.

## Exercise 3 — Design a relationship

Create two tables:

- `workshops`
- `equipment`

Each equipment record must belong to one workshop.

Define:

- A primary key for each table.
- A foreign key in equipment.
- A suitable deletion rule.

Explain why you selected that rule.

## Exercise 4 — Identify integrity problems

Evaluate these proposed values:

| Field | Value |
|---|---|
| inventory_number | INV-001 |
| mileage | -150 |
| maintenance_cost | -2000 |
| department_id | 999 |
| status | UNKNOWN |

Assume department `999` does not exist and the allowed statuses are ACTIVE, INACTIVE, and RETIRED.

Identify which constraints should reject these values.

## Exercise 5 — Architecture reasoning

A system uses an employee email address as its primary key.

Employees can change their email addresses.

Explain:

1. What design risk exists?
2. How could a surrogate key help?
3. Which constraint should protect email uniqueness?
4. What happens to related tables when the email changes?
5. Why is stable identity important in database architecture?

---

# 25. Key Takeaways

1. Keys identify records and establish relationships.
2. A superkey uniquely identifies a record; a candidate key is a minimal superkey.
3. A primary key is the selected candidate key.
4. Alternate keys preserve other candidate identifiers.
5. Natural keys come from the business domain.
6. Surrogate keys are generated for internal identification.
7. Composite keys use multiple columns.
8. Foreign keys enforce valid relationships between tables.
9. Constraints protect database integrity.
10. PRIMARY KEY, UNIQUE, NOT NULL, CHECK, and DEFAULT enforce different rules.
11. Referential actions determine what happens when referenced records change or are deleted.
12. PostgreSQL automatically creates indexes for primary keys and UNIQUE constraints, but not for foreign key columns.
13. Good database design separates stable identity from mutable business information.
14. Constraints should represent real business rules, not arbitrary technical restrictions.

---

# 26. Technical Vocabulary

| English term | Spanish equivalent |
|---|---|
| Key | Clave |
| Superkey | Superclave |
| Candidate key | Clave candidata |
| Primary key | Clave primaria |
| Alternate key | Clave alternativa |
| Natural key | Clave natural |
| Surrogate key | Clave sustituta |
| Composite key | Clave compuesta |
| Foreign key | Clave foránea |
| Constraint | Restricción |
| Unique constraint | Restricción de unicidad |
| Referential integrity | Integridad referencial |
| Entity integrity | Integridad de entidad |
| Domain integrity | Integridad de dominio |
| Data integrity | Integridad de datos |
| Nullability | Admisión de valores nulos |
| Referential action | Acción referencial |
| Identity column | Columna de identidad |
| Junction table | Tabla intermedia |
| Business rule | Regla de negocio |
| Data consistency | Consistencia de datos |

---

# 27. Related Learning Notes

## Data Modeling

- [[01 - Entities and Attributes]]
- [[02 - Relationships and Cardinality]]
- [[04 - Conceptual Logical and Physical Models]]
- [[05 - Normalization]]
- [[06 - Schema Design Principles]]

## Database Internals

- [[01 - Database Management Systems]]
- [[04 - Indexes]]
- [[05 - Transactions and ACID]]
- [[06 - Concurrency and Locks]]

## Applied Architecture

- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]
- [[03 - Architecture Decision Framework]]

---

# 28. Next Learning Step

The next topic is **Conceptual, Logical, and Physical Data Models**.

The objective is to understand how database requirements evolve from business concepts into structured relational schemas and finally into physical database implementations.

This transition connects data modeling with database architecture and provides the foundation for designing complete database systems.
