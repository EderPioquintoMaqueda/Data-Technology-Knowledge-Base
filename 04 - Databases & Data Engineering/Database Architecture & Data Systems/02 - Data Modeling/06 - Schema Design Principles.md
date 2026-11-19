
---
title: "Schema Design Principles"
aliases:
  - "Database Schema Design"
  - "Relational Schema Design"
  - "Principles of Database Design"
type: learning-note
domain: technology
category: data-engineering
subject: data-modeling
level: intermediate
difficulty: foundational-to-intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - database-design
  - data-modeling
  - relational-databases
  - schema-design
  - postgresql
  - sql
  - data-integrity
  - database-development

concepts:
  - relational-schema
  - database-objects
  - naming-conventions
  - data-types
  - nullability
  - constraints
  - referential-integrity
  - historical-data
  - indexing
  - database-migrations
  - schema-versioning
  - access-control

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - What is a Database System]]"
  - "[[01 - Entities and Attributes]]"
  - "[[02 - Relationships and Cardinality]]"
  - "[[03 - Keys and Constraints]]"
  - "[[04 - Conceptual Logical and Physical Models]]"
  - "[[05 - Normalization]]"

related:
  - "[[05 - Database Architecture Fundamentals]]"
  - "[[01 - Database Management Systems]]"
  - "[[04 - Indexes]]"
  - "[[05 - Transactions and ACID]]"
  - "[[PostgreSQL Architecture Overview]]"
  - "[[SIAPV Data Architecture]]"

tools:
  - PostgreSQL
  - SQL
  - pgAdmin
  - Git

learning_objectives:
  - Define a relational database schema and distinguish it from a database and a table.
  - Translate business requirements into maintainable database structures.
  - Apply consistent naming conventions and appropriate data types.
  - Design tables, columns, relationships, and constraints.
  - Understand how historical records should be preserved.
  - Select indexes based on actual query patterns.
  - Understand schema migrations and version control.
  - Evaluate maintainability, security, and performance trade-offs.

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 06 — Schema Design Principles

## 1. Introduction

Database schema design is the process of defining how information is organized, represented, related, constrained, and accessed inside a database.

A well-designed schema does more than store information. It establishes a structure that allows an application to:

- Store reliable information.
- Maintain relationships between records.
- Prevent invalid data.
- Retrieve information efficiently.
- Preserve historical events.
- Support future changes.
- Enforce business rules.
- Control access to sensitive information.

Schema design connects business requirements with the technical implementation of a database.

For example, a vehicle management application may need to answer questions such as:

- Which vehicles belong to a particular department?
- Which maintenance operations have been performed?
- How many kilometers has a vehicle traveled?
- Which machinery has an active assignment?
- What was the maintenance history of a vehicle?
- Which vehicles are currently available?

These questions influence the structure of the database.

The central principle is:

**A database schema should represent the real-world rules of the system, not merely reproduce the appearance of its user interface.**

## 2. What Is a Database Schema?

A database schema is the logical structure that defines the organization of database objects.

Depending on the DBMS, a schema may contain:

- Tables.
- Columns.
- Data types.
- Primary keys.
- Foreign keys.
- Constraints.
- Indexes.
- Views.
- Functions.
- Procedures.
- Triggers.
- Sequences.

In PostgreSQL, a schema is also a namespace used to organize objects inside a database.

### 2.1 Database, schema, table, and record

These concepts represent different levels of organization.

| Concept | Meaning |
|---|---|
| Database | An organized collection of database objects and information. |
| Schema | A namespace that groups related database objects. |
| Table | A structure that stores records using rows and columns. |
| Row | A single record in a table. |
| Column | An attribute defined for every record in a table. |
| Constraint | A rule that restricts which values or relationships are valid. |
| Index | A structure that helps retrieve records efficiently. |

Example:

```text
Database: vehicle_management
│
├── Schema: fleet
│   ├── departments
│   ├── vehicles
│   ├── assignments
│   └── meter_readings
│
└── Schema: maintenance
    ├── maintenance_records
    └── maintenance_types
```

The database contains schemas, and schemas contain database objects.

A schema is not the same thing as a database.

### 2.2 PostgreSQL schema example

PostgreSQL creates a default schema named `public` in a new database.

You can create additional schemas to organize objects:

```sql
CREATE SCHEMA fleet;

CREATE SCHEMA maintenance;
```

You can then create tables inside a specific schema:

```sql
CREATE TABLE fleet.vehicles (
    vehicle_id BIGINT PRIMARY KEY,
    brand TEXT NOT NULL,
    model TEXT
);
```

The fully qualified table name is:

```text
fleet.vehicles
```

Here:

- `fleet` is the schema.
- `vehicles` is the table.

This organization becomes useful when applications grow and contain multiple functional areas.

## 3. Schema Design as a Translation Process

A database schema should originate from requirements and business rules.

It should not begin with arbitrary table creation.

Consider this requirement:

"A department must be able to register vehicles, record maintenance operations, and track mileage and engine hours."

This requirement contains several concepts:

| Requirement | Possible database element |
|---|---|
| Register vehicles | `vehicles` table |
| Organize departments | `departments` table |
| Associate vehicles with departments | Foreign key |
| Record maintenance | `maintenance_records` table |
| Track mileage | `meter_readings` records |
| Track engine hours | `meter_readings` records |
| Preserve operational history | Historical records |

The schema is the technical representation of these requirements.

### 3.1 Recommended design workflow

A disciplined schema design process follows several stages:

1. Understand business requirements.
2. Identify entities and their attributes.
3. Define relationships and cardinalities.
4. Select primary and foreign keys.
5. Choose appropriate data types.
6. Define constraints and business rules.
7. Normalize the data.
8. Identify expected queries and workloads.
9. Design indexes where justified.
10. Review security and access requirements.
11. Test the schema with realistic data.
12. Document and version the implementation.

This workflow connects several previous concepts:

- [[01 - Entities and Attributes]]
- [[02 - Relationships and Cardinality]]
- [[03 - Keys and Constraints]]
- [[04 - Conceptual Logical and Physical Models]]
- [[05 - Normalization]]

Schema design is not an isolated activity. It integrates all of them.

## 4. Naming Conventions

Naming conventions establish consistency across database objects.

Good names make a database easier to understand, query, maintain, and document.

### 4.1 Use meaningful names

Prefer names that communicate the purpose of an object.

Good examples:

```text
vehicles
maintenance_records
departments
inventory_number
maintenance_date
```

Avoid unclear names:

```text
tbl1
data2
reg
info
x
```

A database may contain hundreds of objects. Meaningful naming reduces the effort required to understand them.

### 4.2 Use a consistent naming style

In PostgreSQL, `snake_case` is a common convention.

Examples:

```text
vehicle_id
inventory_number
maintenance_date
department_id
created_at
```

Avoid mixing styles:

```text
vehicleId
InventoryNumber
maintenance_date
DepartmentID
```

A consistent convention improves readability.

### 4.3 Singular or plural table names?

Both conventions are used in professional database systems.

Singular:

```text
vehicle
department
maintenance_record
```

Plural:

```text
vehicles
departments
maintenance_records
```

Neither is universally correct.

The important principle is consistency throughout the schema.

For this learning environment, we will use plural table names because they clearly represent collections of records.

### 4.4 Primary and foreign key naming

A practical convention is:

```text
vehicle_id
department_id
maintenance_id
```

The primary key in `vehicles` is `vehicle_id`.

A foreign key referencing that table should normally use the same column name:

```text
vehicles.vehicle_id
maintenance_records.vehicle_id
```

This makes relationships easier to identify.

### 4.5 Avoid reserved words

Some names conflict with SQL keywords or commonly used functions.

Avoid names such as:

```text
user
order
group
table
select
```

Instead, use more descriptive names:

```text
system_users
sales_orders
user_groups
```

### 4.6 Naming recommendations

| Object | Recommended convention |
|---|---|
| Tables | `plural_snake_case` |
| Columns | `snake_case` |
| Primary keys | `entity_id` |
| Foreign keys | Referenced entity's primary key name |
| Indexes | `idx_table_column` |
| Unique constraints | `uq_table_column` |
| Check constraints | `chk_table_rule` |
| Foreign key constraints | `fk_table_reference` |

These are conventions, not mandatory SQL rules.

## 5. Table Design Principles

A table should represent a coherent entity, relationship, or event.

The structure should make it clear what each row represents.

### 5.1 One table, one main responsibility

Consider a `vehicles` table.

It should contain attributes that describe a vehicle:

```text
vehicles
├── vehicle_id
├── inventory_number
├── brand
├── model
├── manufacturing_year
└── department_id
```

Maintenance operations should not be stored as repeated columns inside this table.

An unsuitable design might look like:

```text
vehicles
├── vehicle_id
├── brand
├── maintenance_1
├── maintenance_2
├── maintenance_3
├── maintenance_4
└── maintenance_5
```

This design creates problems:

- The number of maintenance operations is limited.
- Queries become more complicated.
- Adding new maintenance records requires structural changes.
- Historical information becomes difficult to manage.
- Data redundancy increases.

A better design separates maintenance into its own table:

```text
vehicles
    |
    | 1:N
    |
maintenance_records
```

Each maintenance operation becomes a separate record.

### 5.2 Avoid excessively large tables

A table containing unrelated information from several business processes becomes difficult to maintain.

For example, combining vehicles, employees, insurance policies, maintenance operations, and department information into one table creates unnecessary dependencies.

Separate tables allow each entity to evolve independently.

### 5.3 Avoid unnecessary fragmentation

The opposite problem also exists.

Creating a separate table for every small attribute can make the schema unnecessarily complicated.

For example, a separate table for a simple fixed category may not always be necessary.

A lookup table is useful when the category:

- Has additional attributes.
- Changes over time.
- Needs independent administration.
- Is referenced by many records.
- Requires stronger referential control.

The design should reflect actual requirements.

### 5.4 Associative tables

Many-to-many relationships require an associative table.

For example, a maintenance operation may involve several employees, and an employee may participate in multiple maintenance operations.

A possible design is:

```text
maintenance_records
        |
        | 1:N
        |
maintenance_participants
        |
        | N:1
        |
employees
```

The associative table may contain:

```text
maintenance_participants
├── maintenance_id
├── employee_id
├── participation_role
└── assigned_at
```

This structure can represent both the relationship and additional information about participation.

## 6. Choosing Appropriate Data Types

A data type determines which kind of values a column can store and which operations can be performed on those values.

Choosing the correct type is important for:

- Data integrity.
- Precision.
- Storage.
- Query behavior.
- Application compatibility.
- Validation.

A column should use a type that reflects the meaning of the information.

### 6.1 Integer types

PostgreSQL provides several integer types:

| Type | Storage | Approximate range |
|---|---:|---|
| `SMALLINT` | 2 bytes | -32 thousand to 32 thousand |
| `INTEGER` | 4 bytes | -2.1 billion to 2.1 billion |
| `BIGINT` | 8 bytes | -9.2 quintillion to 9.2 quintillion |

Examples:

```sql
quantity INTEGER;

number_of_employees SMALLINT;

vehicle_id BIGINT;
```

Use integer types for values that represent whole numbers.

### 6.2 Numeric types

`NUMERIC` or `DECIMAL` stores exact decimal values.

It is especially useful for financial information.

Example:

```sql
maintenance_cost NUMERIC(12, 2);
```

The definition means:

- 12 total digits.
- 2 digits after the decimal point.
- Up to 10 digits before the decimal point.

Example value:

```text
125430.75
```

For financial amounts, avoid using floating-point types such as `REAL` or `DOUBLE PRECISION` when exact decimal representation is required.

### 6.3 Text types

PostgreSQL provides:

- `CHAR(n)`
- `VARCHAR(n)`
- `TEXT`

Examples:

```sql
brand VARCHAR(80);

model VARCHAR(100);

description TEXT;
```

`VARCHAR(n)` establishes a maximum length.

`TEXT` stores variable-length text without an explicit maximum length.

In PostgreSQL, `VARCHAR` and `TEXT` have similar performance characteristics for ordinary use.

Choose a length restriction when it represents a real business rule, not simply because a column contains text.

### 6.4 Date and time types

Date and time fields should reflect the meaning of the information.

| Type | Purpose |
|---|---|
| `DATE` | Calendar date |
| `TIME` | Time of day |
| `TIMESTAMP` | Date and time without time zone |
| `TIMESTAMPTZ` | Date and time with time zone semantics |

Examples:

```sql
maintenance_date DATE;

recorded_at TIMESTAMPTZ;

scheduled_time TIME;
```

For events that occur at a specific moment, `TIMESTAMPTZ` is generally a strong choice.

PostgreSQL converts `TIMESTAMPTZ` values internally and displays them according to the session time zone.

It does not preserve the original time zone name entered by the user.

### 6.5 Boolean types

Use `BOOLEAN` for values with true/false semantics.

Example:

```sql
is_active BOOLEAN NOT NULL DEFAULT TRUE;
```

Do not use arbitrary strings such as:

```text
"YES"
"NO"
"ACTIVE"
"INACTIVE"
```

when a Boolean value accurately represents the business rule.

However, Boolean values can also be `NULL` unless prohibited by a constraint.

That means a nullable Boolean can represent three states:

- `TRUE`
- `FALSE`
- `NULL` (unknown or not recorded)

Use this behavior intentionally.

### 6.6 UUID types

A UUID is a 128-bit identifier.

Example:

```sql
vehicle_id UUID PRIMARY KEY;
```

UUIDs can be useful when identifiers must be generated independently across distributed systems.

They are not automatically better than integer identifiers.

Consider:

- Human readability.
- Index size.
- Distributed generation requirements.
- Integration with external systems.
- Identifier exposure.

### 6.7 JSON and JSONB

PostgreSQL supports JSON data through:

```sql
JSON
JSONB
```

Example:

```sql
additional_data JSONB;
```

`JSONB` stores JSON data in a decomposed binary representation and supports indexing and efficient querying for many JSON operations.

It is useful for flexible attributes or semi-structured information.

However, it should not replace relational modeling when information has:

- Stable structure.
- Important relationships.
- Strong validation requirements.
- Frequent filtering or joins.
- Independent business meaning.

For example, storing every vehicle and maintenance record inside one large JSON document would make relational operations more difficult.

Use JSONB as a complementary structure, not as an excuse to avoid schema design.

## 7. NULL, DEFAULT, and Required Information

NULL is one of the most important concepts in relational databases.

It represents the absence of a value.

It is not equivalent to:

- Zero.
- An empty string.
- False.
- A missing database row.

### 7.1 NULL semantics

Consider:

```sql
engine_hours NUMERIC(12, 2);
```

A NULL value may indicate that the engine-hour reading has not been recorded.

It does not mean that the engine has zero hours.

These values have different meanings:

| Value | Interpretation |
|---|---|
| `0` | The recorded value is zero. |
| `NULL` | No value is available or known. |
| `''` | An empty text value. |
| `FALSE` | A Boolean false value. |

This distinction matters when calculating totals and filtering records.

### 7.2 NOT NULL

Use `NOT NULL` when the information is mandatory.

```sql
brand TEXT NOT NULL;
```

The database will reject an insertion that omits the required value or supplies NULL.

### 7.3 DEFAULT

A default provides a value when an insertion does not specify one.

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP;
```

The database automatically assigns the current timestamp when the column is omitted.

A default does not prevent an explicit NULL unless the column also has `NOT NULL`.

### 7.4 Avoid meaningless defaults

Consider:

```sql
manufacturing_year INTEGER DEFAULT 0;
```

If the manufacturing year is unknown, zero is misleading.

A better design may be:

```sql
manufacturing_year INTEGER;
```

The database can represent unknown information with NULL.

The principle is:

**A default value should represent a valid business meaning, not hide missing information.**

## 8. Constraints and Business Rules

Constraints enforce rules directly inside the database.

They prevent invalid information from being stored, regardless of which application submits the data.

### 8.1 Primary key

Identifies each record uniquely.

```sql
vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY;
```

### 8.2 Foreign key

Maintains a valid relationship between tables.

```sql
department_id BIGINT
    REFERENCES fleet.departments(department_id);
```

### 8.3 UNIQUE

Prevents duplicate values.

```sql
inventory_number VARCHAR(40) NOT NULL UNIQUE;
```

This ensures that two vehicle records cannot share the same inventory number.

### 8.4 CHECK

Enforces a logical condition.

```sql
CHECK (maintenance_cost >= 0);
```

This prevents negative maintenance costs.

A more explicit nullable version is:

```sql
CHECK (
    maintenance_cost IS NULL
    OR maintenance_cost >= 0
);
```

The second version allows NULL while rejecting negative values.

### 8.5 Referential actions

Foreign keys can define what happens when a referenced record is deleted or updated.

Common options include:

| Action | Behavior |
|---|---|
| `CASCADE` | Propagates the operation to dependent records. |
| `RESTRICT` | Prevents deletion when dependent records exist. |
| `NO ACTION` | Checks the constraint according to its timing. |
| `SET NULL` | Sets the foreign key to NULL. |
| `SET DEFAULT` | Sets the foreign key to its default value. |

Example:

```sql
department_id BIGINT
    REFERENCES fleet.departments(department_id)
    ON DELETE RESTRICT;
```

For operational and historical information, destructive cascading deletion requires particular care.

Deleting a vehicle should not automatically erase its maintenance history.

### 8.6 Database rules versus application rules

Some business rules can be enforced directly through constraints.

Others require more complex logic.

Examples:

| Rule | Possible implementation |
|---|---|
| Inventory number must be unique | UNIQUE |
| Maintenance cost cannot be negative | CHECK |
| Every maintenance record must reference a vehicle | NOT NULL + FOREIGN KEY |
| A maintenance operation must be approved by an authorized user | Application logic and authorization controls |
| A vehicle cannot have overlapping active assignments | Exclusion constraint, transaction logic, or another appropriate design |
| A maintenance operation must follow a multi-step workflow | Application logic, potentially supported by database constraints |

Important rules should not exist exclusively in the user interface if the database can enforce them reliably.

## 9. Modeling Historical Information

One of the most important schema design decisions is determining whether information represents:

1. The current state of an entity.
2. An event that occurred.
3. A historical version of an entity's state.

These concepts should not be confused.

### 9.1 Current state

A vehicle may have a current department:

```text
vehicles
├── vehicle_id
├── inventory_number
└── department_id
```

This tells us where the vehicle is currently assigned.

However, it does not tell us where the vehicle was assigned last year.

### 9.2 Historical events

A separate assignment history table can preserve changes.

```text
vehicle_assignments
├── assignment_id
├── vehicle_id
├── department_id
├── assigned_at
├── released_at
└── assignment_status
```

Each record represents an assignment period.

Example:

| Vehicle | Department | Assigned | Released |
|---|---|---|---|
| 1001 | Department A | 2025-01-10 | 2025-08-15 |
| 1001 | Department B | 2025-08-16 | NULL |

The second record represents the current assignment if the system defines NULL `released_at` as an active assignment.

This history allows the system to answer:

- Where was the vehicle assigned?
- When did the assignment begin?
- When did it end?
- Which department was responsible during a specific period?

### 9.3 Do not overwrite historical facts

Consider a maintenance record.

If the maintenance date changes because someone edits an old record, the original event information may be lost.

A system with audit requirements may need to preserve:

- Original values.
- Updated values.
- Who performed the change.
- When the change occurred.
- Why the change was made.

This can be implemented through audit tables, event records, or other controlled mechanisms.

The appropriate approach depends on regulatory, operational, and security requirements.

### 9.4 Event time and recording time

These two timestamps can represent different things.

```text
event_at
recorded_at
```

- `event_at`: When the real-world event occurred.
- `recorded_at`: When the system registered the information.

Example:

A maintenance operation occurred on November 10 but was entered into the application on November 12.

```text
event_at:    2026-11-10 14:30
recorded_at: 2026-11-12 09:15
```

This distinction is useful for operational traceability.

## 10. Deletion, Status, and Archiving

Not every record should be physically deleted when it is no longer active.

Different strategies serve different purposes.

### 10.1 Physical deletion

The record is removed from the database.

```sql
DELETE FROM fleet.vehicles
WHERE vehicle_id = 1001;
```

This may be appropriate for erroneous test data or records that must be removed according to an established policy.

However, deleting records referenced by historical information can damage traceability.

### 10.2 Soft deletion

A record remains in the database but is marked as deleted.

Example:

```sql
is_deleted BOOLEAN NOT NULL DEFAULT FALSE;
```

A query can exclude deleted records:

```sql
SELECT *
FROM fleet.vehicles
WHERE is_deleted = FALSE;
```

Advantages:

- Records can remain available for historical purposes.
- Accidental deletion can be reversible.
- Some audit workflows become easier.

Disadvantages:

- Every relevant query must consider the deletion status.
- Unique constraints may require special treatment.
- Data retention becomes more complicated.
- Sensitive information is not automatically protected merely because it is marked deleted.

Soft deletion is not universally the best solution.

### 10.3 Status-based lifecycle

Sometimes a record should remain but change its operational status.

For example:

```text
ACTIVE
IN_MAINTENANCE
OUT_OF_SERVICE
RETIRED
```

A status represents the lifecycle of the entity.

It is not necessarily equivalent to deletion.

### 10.4 Archiving

Archiving moves older information into a different storage or management strategy.

It may be useful when historical data grows significantly.

Archiving should preserve the ability to retrieve information required for audits, reporting, or legal obligations.

## 11. Indexing as Part of Schema Design

Indexes are structures that help the database locate records without scanning every row.

They are important, but creating too many indexes can reduce write performance and increase storage consumption.

### 11.1 Primary key indexes

In PostgreSQL, defining a primary key automatically creates a unique B-tree index.

A UNIQUE constraint also creates a unique index.

You do not need to create another identical index manually.

### 11.2 Foreign key indexes

PostgreSQL does not automatically create an index on the referencing side of a foreign key.

For example:

```sql
vehicle_id BIGINT REFERENCES fleet.vehicles(vehicle_id);
```

The referenced primary key is indexed, but the `vehicle_id` column in the referencing table is not automatically indexed.

Depending on the workload, an additional index may be useful.

### 11.3 Index based on queries

Suppose the application frequently retrieves maintenance records for a particular vehicle, ordered by date.

Query:

```sql
SELECT *
FROM fleet.maintenance_records
WHERE vehicle_id = 1001
ORDER BY maintenance_date DESC;
```

A suitable index may be:

```sql
CREATE INDEX idx_maintenance_vehicle_date
ON fleet.maintenance_records (
    vehicle_id,
    maintenance_date DESC
);
```

This index reflects the filtering and ordering pattern.

### 11.4 Composite indexes

An index can contain multiple columns.

```sql
CREATE INDEX idx_meter_readings_vehicle_time
ON fleet.meter_readings (
    vehicle_id,
    recorded_at DESC
);
```

The order of columns matters.

A composite B-tree index is generally most useful when queries constrain its leading columns.

Do not assume that every combination of columns deserves an index.

### 11.5 Indexing principles

- Identify frequently executed queries.
- Consider filtering conditions.
- Consider join columns.
- Consider sorting and grouping patterns.
- Evaluate selectivity.
- Measure performance before and after adding indexes.
- Monitor write overhead and storage consumption.

Index design belongs to the broader performance discussion in [[04 - Indexes]].

## 12. Views and Derived Information

A schema can provide different ways to access information without duplicating all the underlying records.

### 12.1 Views

A view is a stored SQL query that can be accessed like a table.

Example:

```sql
CREATE VIEW fleet.vehicle_maintenance_summary AS
SELECT
    v.vehicle_id,
    v.inventory_number,
    COUNT(m.maintenance_id) AS maintenance_count
FROM fleet.vehicles AS v
LEFT JOIN fleet.maintenance_records AS m
    ON m.vehicle_id = v.vehicle_id
GROUP BY
    v.vehicle_id,
    v.inventory_number;
```

You can query it:

```sql
SELECT *
FROM fleet.vehicle_maintenance_summary;
```

A regular view does not normally store the result as an independent physical dataset.

It provides a reusable query interface.

### 12.2 Materialized views

A materialized view stores the result of a query.

Example:

```sql
CREATE MATERIALIZED VIEW fleet.monthly_maintenance_summary AS
SELECT
    DATE_TRUNC('month', maintenance_date) AS maintenance_month,
    COUNT(*) AS maintenance_count
FROM fleet.maintenance_records
GROUP BY DATE_TRUNC('month', maintenance_date);
```

The stored result must be refreshed:

```sql
REFRESH MATERIALIZED VIEW
fleet.monthly_maintenance_summary;
```

Materialized views can improve reporting performance when recalculating the result for every request would be expensive.

However, they can become outdated between refreshes.

### 12.3 Derived attributes

Some values can be calculated from other information.

For example:

```text
maintenance_count
total_maintenance_cost
vehicle_age
```

Before storing a derived value permanently, consider:

- Can it be calculated when needed?
- How frequently does it change?
- Is calculation expensive?
- Can the stored value become inconsistent?
- Does the system require a historical snapshot?

Avoid storing duplicated information without a clear reason.

## 13. PostgreSQL Implementation Example

The following example brings together the design principles discussed in this note.

It is an illustrative schema for a vehicle and machinery management system.

It does not assume that it reproduces the exact internal SIAPV implementation.

### 13.1 Create a schema

```sql
CREATE SCHEMA IF NOT EXISTS fleet;
```

### 13.2 Departments table

```sql
CREATE TABLE fleet.departments (
    department_id BIGINT
        GENERATED ALWAYS AS IDENTITY
        PRIMARY KEY,

    department_name VARCHAR(120)
        NOT NULL
        UNIQUE,

    created_at TIMESTAMPTZ
        NOT NULL
        DEFAULT CURRENT_TIMESTAMP
);
```

This table stores departments.

The primary key identifies each department independently of its name.

### 13.3 Vehicles table

```sql
CREATE TABLE fleet.vehicles (
    vehicle_id BIGINT
        GENERATED ALWAYS AS IDENTITY
        PRIMARY KEY,

    inventory_number VARCHAR(40)
        NOT NULL
        UNIQUE,

    brand VARCHAR(80)
        NOT NULL,

    model VARCHAR(100),

    manufacturing_year SMALLINT,

    department_id BIGINT,

    is_active BOOLEAN
        NOT NULL
        DEFAULT TRUE,

    created_at TIMESTAMPTZ
        NOT NULL
        DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_vehicles_department
        FOREIGN KEY (department_id)
        REFERENCES fleet.departments(department_id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_vehicles_inventory_number
        CHECK (LENGTH(TRIM(inventory_number)) > 0),

    CONSTRAINT chk_vehicles_manufacturing_year
        CHECK (
            manufacturing_year IS NULL
            OR manufacturing_year BETWEEN 1900 AND 2100
        )
);
```

Important decisions:

- `vehicle_id` is an internal primary key.
- `inventory_number` is a unique business identifier.
- The inventory number is not used as the primary key.
- `department_id` establishes the relationship with departments.
- `ON DELETE RESTRICT` prevents deleting a department while vehicles reference it.
- The manufacturing year is optional but validated when present.

An internal identifier is useful because business identifiers may change according to administrative rules.

### 13.4 Maintenance records table

```sql
CREATE TABLE fleet.maintenance_records (
    maintenance_id BIGINT
        GENERATED ALWAYS AS IDENTITY
        PRIMARY KEY,

    vehicle_id BIGINT
        NOT NULL,

    maintenance_date DATE
        NOT NULL,

    description TEXT
        NOT NULL,

    maintenance_cost NUMERIC(12, 2),

    created_at TIMESTAMPTZ
        NOT NULL
        DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_maintenance_vehicle
        FOREIGN KEY (vehicle_id)
        REFERENCES fleet.vehicles(vehicle_id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_maintenance_cost
        CHECK (
            maintenance_cost IS NULL
            OR maintenance_cost >= 0
        )
);
```

Each row represents one maintenance event.

A vehicle may have multiple maintenance records.

The cost can be NULL when it has not been registered.

### 13.5 Meter readings table

```sql
CREATE TABLE fleet.meter_readings (
    reading_id BIGINT
        GENERATED ALWAYS AS IDENTITY
        PRIMARY KEY,

    vehicle_id BIGINT
        NOT NULL,

    reading_type VARCHAR(30)
        NOT NULL,

    reading_value NUMERIC(14, 2)
        NOT NULL,

    recorded_at TIMESTAMPTZ
        NOT NULL
        DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_meter_readings_vehicle
        FOREIGN KEY (vehicle_id)
        REFERENCES fleet.vehicles(vehicle_id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_meter_reading_type
        CHECK (
            reading_type IN (
                'odometer_km',
                'engine_hours'
            )
        ),

    CONSTRAINT chk_meter_reading_value
        CHECK (reading_value >= 0)
);
```

This structure allows the application to register different measurement types.

Each reading is stored as an independent historical record.

A more complex implementation could use separate measurement tables or a measurement-type catalog if additional attributes and validation rules become necessary.

### 13.6 Indexes

```sql
CREATE INDEX idx_maintenance_vehicle_date
ON fleet.maintenance_records (
    vehicle_id,
    maintenance_date DESC
);

CREATE INDEX idx_meter_readings_vehicle_time
ON fleet.meter_readings (
    vehicle_id,
    recorded_at DESC
);
```

These indexes support common queries that retrieve the maintenance or meter-reading history of a vehicle.

### 13.7 Example query

Retrieve the maintenance history of one vehicle:

```sql
SELECT
    v.inventory_number,
    m.maintenance_date,
    m.description,
    m.maintenance_cost
FROM fleet.vehicles AS v
INNER JOIN fleet.maintenance_records AS m
    ON v.vehicle_id = m.vehicle_id
WHERE v.inventory_number = '13990'
ORDER BY m.maintenance_date DESC;
```

This query demonstrates:

- Table relationships.
- Primary and foreign keys.
- Joins.
- Filtering.
- Sorting.
- Historical records.

### 13.8 Example data flow

```text
User registers a maintenance operation
                 |
                 v
        Application validation
                 |
                 v
         PostgreSQL connection
                 |
                 v
      INSERT maintenance_records
                 |
                 v
       Foreign key validation
                 |
                 v
        CHECK constraint validation
                 |
                 v
        Transaction committed
                 |
                 v
       Maintenance history updated
```

The application provides the user interface, but the database protects the integrity of the stored information.

## 14. Schema Migrations and Version Control

A database schema changes as an application evolves.

New requirements may introduce:

- New tables.
- New columns.
- Modified constraints.
- Additional indexes.
- New relationships.
- Changes to data types.
- New views.

These modifications should be managed systematically.

### 14.1 What is a migration?

A database migration is a controlled change to the database structure or data.

Example:

```sql
ALTER TABLE fleet.vehicles
ADD COLUMN serial_number VARCHAR(100);
```

This migration adds a new column.

Another example:

```sql
CREATE INDEX idx_vehicles_brand
ON fleet.vehicles(brand);
```

This migration creates an index.

### 14.2 Why migrations matter

Without migration control, different environments may have different database structures.

For example:

```text
Development database: Version 5
Testing database:     Version 4
Production database:  Version 3
```

This inconsistency can cause deployment failures.

A migration system tracks the sequence of changes applied to each environment.

### 14.3 Version control

SQL migration files should be maintained in a version control system such as Git.

Example:

```text
database/
│
├── migrations/
│   ├── 001_create_fleet_schema.sql
│   ├── 002_create_departments.sql
│   ├── 003_create_vehicles.sql
│   ├── 004_create_maintenance_records.sql
│   └── 005_add_meter_readings.sql
│
└── README.md
```

This provides:

- Change history.
- Traceability.
- Collaboration.
- Reproducible environments.
- Easier deployment review.

### 14.4 Development, testing, and production

A professional workflow normally separates environments.

| Environment | Purpose |
|---|---|
| Development | Build and modify the application and schema. |
| Testing | Validate functionality and changes. |
| Production | Operate with real business information. |

A schema change should be tested before reaching production.

### 14.5 Backward-compatible changes

Some database modifications can disrupt applications.

For example, removing a column that an application still uses may cause errors.

A safer strategy may be:

1. Add the new column.
2. Update the application to support it.
3. Migrate existing data if required.
4. Verify the new implementation.
5. Remove the old column in a later controlled migration.

This staged approach reduces deployment risk.

Never treat production schema changes as casual manual edits.

## 15. Security Considerations

Schema design also affects security.

A database may contain:

- Personal information.
- Financial information.
- Operational records.
- Employee information.
- Administrative documentation.

The structure should support controlled access.

### 15.1 Principle of least privilege

A database user should receive only the permissions required for its responsibilities.

For example:

```sql
CREATE ROLE fleet_readonly;

GRANT USAGE ON SCHEMA fleet
TO fleet_readonly;

GRANT SELECT ON ALL TABLES IN SCHEMA fleet
TO fleet_readonly;
```

This role receives read access to existing tables in the `fleet` schema.

Additional permissions may be needed for future tables, depending on the database's default privileges.

### 15.2 Separate responsibilities

Different application roles may need different permissions.

Examples:

```text
database_administrator
application_writer
reporting_reader
audit_reader
```

An administrator should not be the default database account used by every application component.

### 15.3 Sensitive information

Avoid exposing sensitive information unnecessarily.

Consider:

- Data minimization.
- Role-based permissions.
- Audit logging.
- Encryption requirements.
- Retention policies.
- Secure backups.

Database security is expanded in [[01 - Database Security Fundamentals]] and [[02 - Authentication and Authorization]].

## 16. Schema Design Trade-offs

There is no single schema design that optimizes every requirement.

Database architecture involves balancing competing priorities.

| Decision | Potential benefit | Potential cost |
|---|---|---|
| More normalization | Less redundancy and stronger consistency | More joins |
| Denormalization | Faster access for certain workloads | More maintenance and consistency risk |
| More indexes | Faster reads for supported queries | Slower writes and additional storage |
| More constraints | Stronger data integrity | Additional validation work |
| JSONB flexibility | Flexible semi-structured information | Less relational enforcement |
| Historical tables | Better traceability | More storage and query complexity |
| Separate schemas | Better organization and namespace control | Additional organizational complexity |

A design decision should be justified by requirements and measurable consequences.

The goal is not to create the most complicated schema.

The goal is to create a structure that supports the system reliably.

## 17. Common Schema Design Mistakes

| Mistake | Consequence | Better approach |
|---|---|---|
| Using one table for unrelated entities | Redundancy and confusing relationships | Separate entities by responsibility |
| Using vague column names | Difficult maintenance | Use meaningful names |
| Storing numbers as text | Incorrect sorting and calculations | Use numeric data types |
| Using zero instead of NULL for unknown values | Misleading information | Represent missing information correctly |
| Repeating groups in columns | Difficult querying and expansion | Create related records |
| Ignoring foreign keys | Invalid relationships | Enforce referential integrity |
| Using business identifiers as permanent primary keys | Changes can affect many relationships | Consider stable internal identifiers |
| Deleting historical records | Loss of traceability | Preserve history when required |
| Creating indexes without query analysis | Unnecessary overhead | Index based on workload |
| Duplicating calculated values | Inconsistent information | Calculate or maintain them deliberately |
| Making manual production changes | Environment inconsistency | Use controlled migrations |
| Giving every user administrator permissions | Security exposure | Apply least privilege |

## 18. Practical Application: Reviewing an Operational Database

Imagine that an application manages approximately 200 vehicles and machinery units.

The system needs to support:

- Vehicle inventory.
- Department assignments.
- Maintenance operations.
- Odometer readings.
- Engine-hour readings.
- Historical reports.
- Administrative control.

A possible logical structure is:

```text
departments
    |
    | 1:N
    |
vehicles
    |
    | 1:N
    |
    +----------------------+
    |                      |
    v                      v
maintenance_records   meter_readings
```

This design separates the main responsibilities.

### 18.1 Questions to evaluate

When reviewing an existing database, ask:

**Structure**
- Does each table represent a clear entity or event?
- Are unrelated attributes mixed together?
- Are relationships explicit?

**Integrity**
- Are primary keys defined?
- Are foreign keys enforced?
- Are required fields marked NOT NULL?
- Are unique business identifiers protected?

**Historical information**
- Can the system retrieve previous maintenance records?
- Can it identify historical department assignments?
- Are old values overwritten or preserved?

**Performance**
- Which queries are executed frequently?
- Are filtering and join columns indexed appropriately?
- Are expensive reports separated from operational transactions when necessary?

**Maintainability**
- Are names consistent?
- Are changes documented?
- Are SQL migrations version-controlled?
- Can the schema evolve without breaking existing functionality?

**Security**
- Do application users have only the required permissions?
- Are sensitive fields protected?
- Are administrative actions traceable?

These questions form the beginning of a database architecture review.

## 19. Knowledge Check

Answer the following questions without consulting the previous sections.

1. What is the difference between a database and a schema in PostgreSQL?
2. Why should a table have a clearly defined responsibility?
3. Why is an internal primary key often preferable to using an inventory number as the primary key?
4. What is the difference between NULL and zero?
5. When should `NUMERIC(12,2)` be preferred over `DOUBLE PRECISION`?
6. Why is a foreign key important?
7. What is the difference between current-state information and historical events?
8. Why should indexes be designed around query patterns?
9. What is a database migration?
10. Why should production schema changes be version-controlled?
11. When is JSONB appropriate?
12. Why is soft deletion not always the best solution?

## 20. Practical Exercises

### Exercise 1 — Design a basic schema

Design a relational schema for an equipment management system.

The system must manage:

- Equipment.
- Departments.
- Maintenance records.
- Meter readings.

For each table, define:

- Table name.
- Columns.
- Data types.
- Primary key.
- Foreign keys.
- Constraints.

### Exercise 2 — Identify design problems

Consider this table:

```text
equipment
├── equipment_id
├── department_name
├── department_phone
├── brand
├── model
├── maintenance_1
├── maintenance_2
├── maintenance_3
├── mileage
└── maintenance_cost
```

Identify at least five design problems.

Then propose a normalized structure.

### Exercise 3 — Implement the schema

Using PostgreSQL:

1. Create a schema named `equipment`.
2. Create a `departments` table.
3. Create an `equipment_items` table.
4. Create a `maintenance_records` table.
5. Create a `meter_readings` table.
6. Define primary keys and foreign keys.
7. Add appropriate CHECK and UNIQUE constraints.
8. Insert sample records.
9. Write queries to retrieve equipment and maintenance history.

### Exercise 4 — Review an existing system

Use the SIAPV concept as a reference.

Prepare a short schema review document answering:

- Which entities should exist?
- Which relationships are necessary?
- Which identifiers should be stable?
- Which records must preserve history?
- Which constraints should be enforced?
- Which queries will need indexes?
- Which information should be restricted?
- How should future schema changes be deployed?

Do not assume that an illustrative schema matches the actual internal implementation.

## 21. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Database schema | Esquema de base de datos |
| Schema design | Diseño del esquema |
| Database object | Objeto de base de datos |
| Namespace | Espacio de nombres |
| Naming convention | Convención de nomenclatura |
| Data type | Tipo de dato |
| Nullability | Posibilidad de admitir valores NULL |
| Default value | Valor predeterminado |
| Constraint | Restricción |
| Referential integrity | Integridad referencial |
| Business rule | Regla de negocio |
| Historical record | Registro histórico |
| Audit trail | Registro de auditoría |
| Soft deletion | Eliminación lógica |
| Archiving | Archivado |
| Derived attribute | Atributo derivado |
| Materialized view | Vista materializada |
| Composite index | Índice compuesto |
| Migration | Migración |
| Schema versioning | Control de versiones del esquema |
| Backward compatibility | Compatibilidad con versiones anteriores |
| Least privilege | Principio de mínimo privilegio |
| Maintainability | Mantenibilidad |
| Scalability | Escalabilidad |
| Trade-off | Compromiso entre alternativas |

## 22. Key Takeaways

- A database schema defines how database objects are organized and related.
- Schema design translates business requirements into technical structures.
- Tables should represent coherent entities, relationships, or events.
- Naming conventions improve readability and maintainability.
- Data types should reflect the meaning and expected range of information.
- NULL, zero, empty strings, and FALSE have different meanings.
- Constraints enforce important business rules and data integrity.
- Historical events should not be confused with current-state information.
- Deletion and archiving strategies must consider operational and audit requirements.
- Indexes should support real query patterns.
- Views and materialized views provide different ways to expose information.
- Schema migrations make database evolution controlled and reproducible.
- Security and access control should be considered from the design stage.

**A good database schema is not simply a collection of tables. It is a structured representation of the business domain, its relationships, its rules, and its history.**

## 23. Related Notes

### Previous concepts

- [[01 - Entities and Attributes]]
- [[02 - Relationships and Cardinality]]
- [[03 - Keys and Constraints]]
- [[04 - Conceptual Logical and Physical Models]]
- [[05 - Normalization]]

### Next learning block

- [[01 - Database Management Systems]]
- [[02 - How a Query is Executed]]
- [[03 - Storage Pages and Tables]]
- [[04 - Indexes]]
- [[05 - Transactions and ACID]]

### Architecture connections

- [[05 - Database Architecture Fundamentals]]
- [[02 - Database Scalability]]
- [[PostgreSQL Architecture Overview]]
- [[SIAPV Data Architecture]]
