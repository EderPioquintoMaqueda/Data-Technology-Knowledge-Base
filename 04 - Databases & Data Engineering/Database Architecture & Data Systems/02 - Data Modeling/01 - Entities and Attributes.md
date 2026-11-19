
---
title: "Entities and Attributes"
aliases:
  - Database Entities
  - Database Attributes
  - Entity Identification
  - Data Modeling Fundamentals

type: learning-note
domain: technology
category: data-engineering
subject: data-modeling

level: beginner
difficulty: foundational
status: in-progress

created: 2026-11-12
updated: 2026-11-12

tags:
  - data-modeling
  - entities
  - attributes
  - database-design
  - relational-databases
  - database-architecture
  - requirements-analysis

concepts:
  - entity
  - entity-instance
  - attribute
  - attribute-domain
  - entity-identification
  - data-model
  - business-entity
  - entity-integrity
  - data-requirements

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - What is a Database System]]"
  - "[[03 - Types of Data]]"
  - "[[05 - Database Architecture Fundamentals]]"

related:
  - "[[02 - Relationships and Cardinality]]"
  - "[[03 - Keys and Constraints]]"
  - "[[04 - Conceptual Logical and Physical Models]]"
  - "[[05 - Normalization]]"
  - "[[06 - Schema Design Principles]]"
  - "[[01 - Database Management Systems]]"
  - "[[01 - SIAPV Data Architecture]]"
  - "[[02 - SIAPV Database Design Review]]"

tools:
  - PostgreSQL
  - pgAdmin
  - SQL
  - Visual Studio Code
  - diagrams.net

learning_objectives:
  - Define an entity in database modeling.
  - Understand the difference between an entity and an entity instance.
  - Define an attribute and its role in a data model.
  - Identify entities from business requirements.
  - Distinguish attributes from entities.
  - Understand attribute domains and data types.
  - Recognize simple, composite, single-valued, and multivalued attributes.
  - Translate a real-world information requirement into a preliminary entity structure.

review:
  status: pending
  confidence:
  last_reviewed:
---

# Entities and Attributes

## 1. Introduction

Before creating a database, it is necessary to understand what information the system must represent.

This process begins with analyzing the real-world environment that the database will support.

Consider a vehicle management application.

The organization may need to store information about:

- Vehicles.
- Employees.
- Departments.
- Maintenance operations.
- Workshops.
- Equipment assignments.

These concepts represent objects, people, or events that the organization needs to identify and manage.

In database modeling, many of these concepts are represented as **Entities**.

The individual characteristics describing them are represented as **Attributes**.

Understanding entities and attributes is the first step toward designing a structured database.

## 2. What Is an Entity?

An **Entity** is a distinguishable object, person, place, event, or concept about which an organization needs to store information.

An entity represents something that can be identified and described within a particular system.

Examples:

| Business Concept | Entity |
|---|---|
| A vehicle owned by an organization | Vehicle |
| An employee working in a department | Employee |
| An organizational unit | Department |
| A maintenance operation | Maintenance |
| A supplier providing equipment | Supplier |
| A registered customer | Customer |
| A product available for sale | Product |

An entity does not necessarily represent a physical object.

For example:

- A payment is an entity.
- A reservation is an entity.
- A contract is an entity.
- A transaction is an entity.

These concepts represent business information that must be managed.

### 2.1 Entity vs Entity Instance

It is important to distinguish an entity from an individual instance of that entity.

An **Entity Type** defines a category of objects.

An **Entity Instance** represents one specific occurrence of that category.

Example:

**Entity Type:** Vehicle

**Entity Instances:**

- Vehicle 1001.
- Vehicle 1002.
- Vehicle 1003.

In a relational database, an entity type is commonly represented by a table, while its instances are represented by rows.

Example:

| vehicle_id | brand | model |
|---|---|---|
| 1001 | Nissan | Frontier |
| 1002 | Ford | Ranger |
| 1003 | Toyota | Hilux |

Here:

- `Vehicle` is the entity type.
- Each row is an entity instance.
- Each column represents an attribute.

### 2.2 Entity Identification

An entity must be distinguishable from other instances of the same entity type.

For example, two vehicles may have:

- The same brand.
- The same model.
- The same manufacturing year.

However, the database must still be able to distinguish between them.

This is why database models use identifiers.

Identifiers will be studied in detail in:

[[03 - Keys and Constraints]].

## 3. What Is an Attribute?

An **Attribute** is a characteristic or property that describes an entity.

For example, a vehicle may have the following attributes:

- Vehicle identifier.
- Inventory number.
- Brand.
- Model.
- Manufacturing year.
- License plate.
- Mileage.
- Current status.

An attribute describes one aspect of an entity.

### 3.1 Example: Vehicle Entity

```text
ENTITY: VEHICLE

Attributes:

- vehicle_id
- inventory_number
- brand
- model
- manufacturing_year
- license_plate
- mileage
- status
```

A simplified relational representation:

| vehicle_id | inventory_number | brand | model | year |
|---|---|---|---|---|
| 1001 | PK-001 | Nissan | Frontier | 2025 |
| 1002 | PK-002 | Ford | Ranger | 2024 |
| 1003 | PK-003 | Toyota | Hilux | 2026 |

The columns describe the characteristics of each vehicle.

### 3.2 Entity and Attribute Relationship

An entity groups related attributes.

For example:

```text
                  VEHICLE
                     |
          +----------+----------+
          |          |          |
       BRAND       MODEL       YEAR
          |          |          |
          +----------+----------+
                     |
                  MILEAGE
                     |
                   STATUS
```

This is a simplified conceptual representation.

A complete entity model must also consider identifiers, attribute domains, relationships, and business rules.

## 4. How to Identify Entities in Business Requirements

Entity identification begins with understanding the business problem.

Imagine that an organization requests a system with the following description:

"The application must manage vehicles, employees, departments, maintenance records, and equipment assignments."

The first step is to identify the important business concepts.

### Step 1: Read the Requirements

Extract the main concepts from the description.

Potential entities:

- Vehicle.
- Employee.
- Department.
- Maintenance.
- Equipment Assignment.

### Step 2: Understand What Must Be Stored

For each concept, ask:

- Does the system need to store information about it?
- Does it have characteristics that must be recorded?
- Does it need to be identified?
- Does it participate in business operations?
- Does it need to exist independently in the system?

### Step 3: Identify Candidate Attributes

For the Vehicle entity:

- Inventory number.
- Brand.
- Model.
- Year.
- License plate.
- Status.

For the Employee entity:

- Employee identifier.
- First name.
- Last name.
- Position.
- Contact information.

For the Department entity:

- Department identifier.
- Department name.
- Department code.

### Step 4: Validate the Model

Not every noun in a business requirement should automatically become an entity.

Consider the following questions:

- Is this concept important enough to store independently?
- Is it merely an attribute of another entity?
- Does it represent an event or a persistent object?
- Does it have its own identifier?
- Does the business need to query it independently?

Entity identification requires understanding the business context.

## 5. Distinguishing Entities from Attributes

One of the most common mistakes in data modeling is confusing entities with attributes.

Consider the following examples.

### Example A: Vehicle

A vehicle may have:

- Brand.
- Model.
- Year.
- Color.

These are generally attributes because they describe the vehicle.

### Example B: Department

A department may have:

- Department name.
- Department code.
- Location.

However, a department may also be an independent entity because the organization needs to manage it separately and associate multiple employees or vehicles with it.

### Example C: Manufacturer

A manufacturer may initially appear to be a simple vehicle attribute.

However, suppose the system must store:

- Manufacturer name.
- Headquarters.
- Contact information.
- Country of origin.
- Multiple vehicle models.

In that situation, Manufacturer may deserve its own entity.

### Comparison

| Concept | Possible Representation |
|---|---|
| Vehicle brand | Attribute |
| Vehicle model | Attribute or separate entity |
| Department | Entity |
| Employee name | Attribute |
| Manufacturer | Entity or attribute, depending on requirements |
| Maintenance operation | Entity |
| Maintenance date | Attribute |
| Vehicle status | Attribute or reference entity |

The correct decision depends on the information requirements and business rules.

There is no universal rule that every repeated value must become an independent entity.

## 6. Attribute Characteristics

Attributes can have different structural properties.

Understanding these properties helps create better data models.

### 6.1 Simple Attributes

A simple attribute cannot be meaningfully divided into smaller components within the current model.

Examples:

- Manufacturing year.
- Vehicle status.
- Mileage.
- Employee identifier.

Example:

```text
VEHICLE
- manufacturing_year
- mileage
- status
```

### 6.2 Composite Attributes

A composite attribute can be divided into meaningful components.

For example, a person's full name may be represented as:

```text
FULL NAME
   |
   +-- First Name
   |
   +-- Middle Name
   |
   +-- Last Name
```

An address may contain:

- Street.
- Building number.
- Neighborhood.
- City.
- State.
- Postal code.
- Country.

In a relational database, these components are often stored in separate columns when the application needs to filter, validate, or analyze them independently.

### 6.3 Single-Valued Attributes

A single-valued attribute contains one value for a particular entity instance.

Examples:

- Manufacturing year.
- Current vehicle status.
- Employee identifier.

A vehicle may have one current status at a given moment.

### 6.4 Multivalued Attributes

A multivalued attribute can contain multiple values for a single entity instance.

For example, an employee may have multiple telephone numbers.

A vehicle may have several historical license plates.

In relational modeling, these values are commonly represented through a separate related table rather than storing multiple values in one column.

Example:

```text
EMPLOYEE
--------
employee_id
first_name
last_name

EMPLOYEE_PHONE
--------------
phone_id
employee_id
phone_number
phone_type
```

This approach allows each phone number to be stored and managed independently.

Multivalued attributes will become easier to understand when studying relationships and normalization.

### 6.5 Derived Attributes

A derived attribute is calculated from other stored information.

Example:

A vehicle's age may be calculated using its manufacturing year.

Conceptually:

```text
vehicle_age = current_year - manufacturing_year
```

Other examples:

- Total maintenance cost.
- Employee tenure.
- Average monthly mileage.
- Total number of maintenance operations.

Derived values do not always need to be stored permanently.

Storing them may introduce synchronization problems if the underlying information changes.

The decision depends on performance, historical requirements, and consistency rules.

## 7. Attribute Domains and Data Types

An attribute must have a defined domain.

An **Attribute Domain** describes the set of valid values that an attribute can accept.

For example:

| Attribute | Possible Domain |
|---|---|
| Manufacturing year | Valid integer years |
| Mileage | Non-negative numeric values |
| Vehicle status | Approved status values |
| Email address | Text following defined validation rules |
| Registration date | Valid dates |
| Inventory number | Text following an organizational format |

A database uses data types and constraints to enforce part of these requirements.

### 7.1 Common SQL Data Types

| Data Type | Purpose | Example |
|---|---|---|
| INTEGER | Whole numbers | 125 |
| BIGINT | Large whole numbers | 9000000000 |
| NUMERIC | Exact decimal values | 1250.75 |
| VARCHAR | Variable-length text | 'Nissan' |
| TEXT | General text | 'Vehicle description' |
| DATE | Calendar date | '2026-11-12' |
| TIMESTAMP | Date and time | '2026-11-12 09:30:00' |
| BOOLEAN | True or false | TRUE |
| UUID | Universally unique identifier | A UUID value |

Selecting appropriate data types is important for:

- Data integrity.
- Storage efficiency.
- Query behavior.
- Application compatibility.
- Validation.

### 7.2 Example: PostgreSQL Table

Consider a preliminary Vehicle entity.

```sql
CREATE TABLE vehicles (
    vehicle_id BIGINT,
    inventory_number VARCHAR(30),
    brand VARCHAR(80),
    model VARCHAR(80),
    manufacturing_year INTEGER,
    mileage NUMERIC(12, 2),
    status VARCHAR(30)
);
```

This statement creates a table with columns representing attributes.

However, this is only an initial structure.

A production-quality design should also consider:

- Primary keys.
- NOT NULL constraints.
- UNIQUE constraints.
- CHECK constraints.
- Foreign keys.
- Appropriate default values.

These topics will be developed in [[03 - Keys and Constraints]] and [[06 - Schema Design Principles]].

## 8. Naming Attributes Correctly

Good attribute names make a database easier to understand and maintain.

Consider these examples:

| Poor Name | Better Name |
|---|---|
| `x` | `vehicle_id` |
| `data1` | `registration_date` |
| `val` | `maintenance_cost` |
| `desc` | `vehicle_description` |
| `num` | `inventory_number` |
| `stat` | `vehicle_status` |

### Naming Guidelines

1. Use descriptive names.
2. Keep naming conventions consistent.
3. Avoid ambiguous abbreviations.
4. Use singular or plural conventions consistently.
5. Avoid names that conflict with SQL reserved keywords.
6. Make the meaning of each attribute clear.
7. Use a consistent language throughout the schema.

For this learning vault, examples use English database identifiers because English naming is common in technical environments.

## 9. A Complete Preliminary Entity Inventory

Before building relationships or creating SQL tables, it is useful to prepare an entity inventory.

Consider a simplified vehicle management system.

| Entity | Purpose | Example Attributes |
|---|---|---|
| Vehicle | Stores vehicle information | vehicle_id, inventory_number, brand, model |
| Employee | Stores employee information | employee_id, first_name, last_name |
| Department | Stores organizational units | department_id, department_name |
| Maintenance | Records maintenance operations | maintenance_id, maintenance_date, cost |
| Workshop | Stores workshop information | workshop_id, workshop_name, location |
| Assignment | Records vehicle assignments | assignment_id, start_date, end_date |

This inventory establishes the initial information structure.

It does not yet define how entities are connected.

That is the next step in data modeling.

## 10. Common Modeling Mistakes

### Mistake 1: Creating One Large Table

Putting all information into a single table may create:

- Repeated information.
- Difficult updates.
- Inconsistent values.
- Data duplication.
- Complex queries.

The solution begins with identifying independent entities and their relationships.

### Mistake 2: Treating Every Attribute as an Entity

Not every characteristic needs its own table.

For example, a manufacturing year does not normally require an independent entity.

An independent entity should have a meaningful role in the information model.

### Mistake 3: Storing Multiple Values in One Column

Example of a problematic structure:

```text
employee_id | phone_numbers
------------|-----------------------
101         | 5551111, 5552222
```

This makes individual values harder to validate, search, and update.

A separate related table is generally a better relational design.

### Mistake 4: Using Ambiguous Attribute Names

Names such as `data`, `value`, or `info` do not clearly explain what information is stored.

Descriptive names improve maintainability.

### Mistake 5: Ignoring Business Requirements

A technically valid model may still be incorrect if it does not represent the organization's actual processes.

Data modeling must begin with understanding the business.

## 11. Practical Example: SIAPV Entity Identification

Imagine reviewing the information requirements of SIAPV.

The system must manage:

- Vehicles and machinery.
- Inventory identifiers.
- Vehicle characteristics.
- Mileage and operating hours.
- Maintenance records.
- Organizational assignments.
- Operational status.

A preliminary model could contain the following entities.

### Entity: Vehicle

| Attribute | Description |
|---|---|
| vehicle_id | Internal identifier |
| inventory_number | Organizational inventory identifier |
| brand | Vehicle manufacturer |
| model | Vehicle model |
| manufacturing_year | Manufacturing year |
| license_plate | Vehicle registration plate |
| status | Current operational status |

### Entity: Maintenance

| Attribute | Description |
|---|---|
| maintenance_id | Maintenance record identifier |
| maintenance_date | Date of the operation |
| maintenance_type | Type of maintenance |
| maintenance_cost | Recorded cost |
| description | Details of the work |

### Entity: Department

| Attribute | Description |
|---|---|
| department_id | Department identifier |
| department_name | Official department name |
| department_code | Organizational code |

### Entity: Employee

| Attribute | Description |
|---|---|
| employee_id | Employee identifier |
| first_name | Employee first name |
| last_name | Employee last name |
| position | Organizational position |

These are preliminary examples, not a claim that they reproduce SIAPV's actual database schema.

The next modeling step is to determine how these entities relate to one another.

For example:

- Can a department manage multiple vehicles?
- Can a vehicle have multiple maintenance records?
- Can an employee be responsible for multiple vehicles?
- Can a vehicle change departments over time?

These questions lead directly to relationships and cardinality.

## 12. Knowledge Check

Answer the following questions without reviewing the explanations.

1. What is an entity?
2. What is the difference between an entity type and an entity instance?
3. What is an attribute?
4. What is the difference between an entity and an attribute?
5. Why might Manufacturer be modeled as an independent entity?
6. What is a composite attribute?
7. What is a multivalued attribute?
8. What is a derived attribute?
9. What is an attribute domain?
10. Why is selecting an appropriate SQL data type important?
11. Why should multiple telephone numbers not normally be stored in one text column?
12. Why must entity identification begin with business requirements?

## 13. Practical Exercises

### Exercise A: Identify Entities

An organization wants to develop an application to manage its procurement process.

The system must manage:

- Suppliers.
- Purchase orders.
- Products.
- Employees.
- Departments.
- Deliveries.
- Payments.

Identify the candidate entities.

For each entity, explain why it should be represented independently.

### Exercise B: Identify Attributes

For the following entities, propose at least six attributes.

1. Supplier.
2. Product.
3. Purchase Order.
4. Employee.
5. Delivery.

For every attribute, specify:

- Attribute name.
- Description.
- Proposed SQL data type.
- Whether it is simple, composite, multivalued, or derived.

### Exercise C: Distinguish Entities and Attributes

Classify each concept according to the procurement system described above.

| Concept | Entity or Attribute? | Explanation |
|---|---|---|
| Supplier name | | |
| Supplier | | |
| Product price | | |
| Product | | |
| Delivery date | | |
| Delivery | | |
| Employee email | | |
| Department | | |
| Purchase order number | | |
| Payment | | |

Some concepts may require additional business context. Explain your assumptions.

### Exercise D: SIAPV Modeling

Prepare a preliminary entity inventory for a vehicle management application.

Include at least:

- Vehicle.
- Department.
- Employee.
- Maintenance.
- Workshop.

For each entity, identify its attributes and explain what information the system needs to preserve.

Do not create relationships yet.

The next document will address that subject.

## 14. Key Takeaways

- An entity represents an identifiable business object, person, event, or concept.
- An entity type defines a category; an entity instance represents an individual occurrence.
- Attributes describe the characteristics of entities.
- Entities and attributes must be identified from business requirements.
- Attributes may be simple, composite, single-valued, multivalued, or derived.
- Attribute domains define valid values.
- SQL data types help control how information is represented and stored.
- Independent entities should represent meaningful business concepts.
- Good naming conventions improve database readability and maintainability.
- A preliminary entity inventory is an important step before defining relationships and implementing tables.

## 15. Related Notes

### Previous Learning
- [[00 - Database Architecture Roadmap]]
- [[01 - What is a Database System]]
- [[02 - The Data Lifecycle]]
- [[03 - Types of Data]]
- [[04 - OLTP vs OLAP]]
- [[05 - Database Architecture Fundamentals]]

### Data Modeling
- [[02 - Relationships and Cardinality]]
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

**Learning Objective:** Identify the entities and attributes required to represent a real-world business environment and prepare a structured foundation for relational database modeling.
