
---
title: "Types of Data"
aliases:
  - Data Classification
  - Structured and Unstructured Data
  - Data Structures and Formats

type: learning-note
domain: technology
category: data-engineering
subject: data-classification

level: beginner
difficulty: foundational
status: in-progress

created: 2026-11-12
updated: 2026-11-12

tags:
  - types-of-data
  - structured-data
  - semi-structured-data
  - unstructured-data
  - data-modeling
  - database-architecture
  - data-engineering
  - data-formats
  - fundamentals

concepts:
  - structured-data
  - semi-structured-data
  - unstructured-data
  - data-schema
  - data-format
  - data-type
  - metadata
  - data-classification

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - What is a Database System]]"
  - "[[02 - The Data Lifecycle]]"

related:
  - "[[01 - Entities and Attributes]]"
  - "[[04 - OLTP vs OLAP]]"
  - "[[05 - Database Architecture Fundamentals]]"
  - "[[02 - Data Integration]]"
  - "[[05 - Data Warehouses]]"
  - "[[06 - Data Lakes]]"
  - "[[07 - Lakehouse Architecture]]"

tools:
  - PostgreSQL
  - SQL
  - Python
  - JSON
  - CSV
  - XML

learning_objectives:
  - Understand the main categories of data.
  - Differentiate structured, semi-structured, and unstructured data.
  - Understand schemas and data formats.
  - Recognize common data representation formats.
  - Understand how data types influence database design.
  - Identify appropriate storage technologies for different data structures.

review:
  status: pending
  confidence:
  last_reviewed:
---

# Types of Data

## 1. Introduction

Not all information has the same structure.

A financial transaction, a customer profile, an image, a PDF document, and a machine sensor reading are all examples of data. However, they differ in how they are organized, represented, stored, and processed.

These differences are important when designing database systems.

For example, a relational database is particularly suitable for managing structured records with defined relationships. However, a collection of images or documents may require a different storage approach.

Understanding data classification helps architects determine:

- How information should be represented.
- Which database technology may be appropriate.
- How information should be validated.
- How data should be processed.
- How different systems can exchange information.
- What storage infrastructure is required.

One of the most common classifications divides data into three categories:

1. Structured Data.
2. Semi-Structured Data.
3. Unstructured Data.

## 2. Structured Data

**Structured Data** is information organized according to a predefined structure or schema.

A schema defines how information is organized, including fields, data types, relationships, and constraints.

Structured data is commonly represented in relational databases using tables, rows, and columns.

### 2.1 Main Characteristics

Structured data typically has:

- Clearly defined fields.
- Consistent data types.
- Predictable organization.
- Defined relationships.
- Formal validation rules.
- Well-established query mechanisms.

### 2.2 Example

Imagine a database used to manage vehicles.

A table named `vehicles` could contain:

| vehicle_id | brand | model | year |
|---|---|---|---:|
| 1001 | Nissan | Frontier | 2024 |
| 1002 | Ford | Ranger | 2023 |
| 1003 | Toyota | Hilux | 2025 |

Each column has a specific meaning.

The `vehicle_id` field identifies the vehicle, while `brand`, `model`, and `year` describe its characteristics.

A simplified SQL definition could be:

```sql
CREATE TABLE vehicles (
    vehicle_id INTEGER PRIMARY KEY,
    brand VARCHAR(50) NOT NULL,
    model VARCHAR(50) NOT NULL,
    year INTEGER
);
```

This definition establishes the structure that records must follow.

### 2.3 Common Examples

- Customer databases.
- Financial transactions.
- Inventory systems.
- Employee records.
- Vehicle inventories.
- Banking accounts.
- Sales records.

### 2.4 Advantages

- Easy to query using SQL.
- Clear organization.
- Strong data integrity mechanisms.
- Predictable relationships.
- Suitable for transactional applications.
- Efficient for many structured reporting operations.

### 2.5 Limitations

- Requires a defined schema.
- Structural changes may require database modifications.
- Complex or highly variable information may be difficult to represent naturally in rigid table structures.

### 2.6 Typical Technologies

- PostgreSQL.
- MySQL.
- Microsoft SQL Server.
- Oracle Database.
- IBM Db2.

**Key concept:** Structured data is organized according to an explicitly defined schema.

## 3. Semi-Structured Data

**Semi-Structured Data** is information that does not necessarily follow a rigid tabular structure but contains organizational elements that make it possible to identify and interpret its contents.

It may use:

- Keys.
- Values.
- Tags.
- Nested objects.
- Hierarchical structures.
- Metadata.

Unlike traditional relational tables, semi-structured data can contain different attributes across individual records.

### 3.1 Main Characteristics

- Flexible structure.
- Explicit field names or identifiers.
- Nested information.
- Potentially variable attributes.
- Often represented using hierarchical formats.

### 3.2 JSON Example

JSON stands for JavaScript Object Notation.

It is a widely used format for exchanging and representing information between applications.

Example:

```json
{
  "vehicle_id": 1001,
  "brand": "Nissan",
  "model": "Frontier",
  "year": 2024,
  "maintenance": {
    "last_service": "2026-10-15",
    "mileage": 35000
  },
  "features": [
    "diesel",
    "automatic"
  ]
}
```

Notice that the information is organized using keys and values.

It also contains nested objects and arrays.

This structure allows related information to be grouped together.

### 3.3 Other Common Formats

**XML — Extensible Markup Language**

XML represents information through hierarchical elements and tags.

Example:

```xml
<vehicle>
    <vehicle_id>1001</vehicle_id>
    <brand>Nissan</brand>
    <model>Frontier</model>
    <year>2024</year>
</vehicle>
```

**YAML — YAML Ain't Markup Language**

YAML is a human-readable format frequently used for configuration files and structured information.

Example:

```yaml
vehicle:
  vehicle_id: 1001
  brand: Nissan
  model: Frontier
  year: 2024
```

**Important:** JSON, XML, and YAML are data representation formats. Their use does not automatically determine which database technology must store them.

### 3.4 Common Applications

- REST APIs.
- Application configuration.
- Event messages.
- System integrations.
- Document-oriented databases.
- Application metadata.
- Nested records.

### 3.5 Advantages

- Flexible representation.
- Supports nested structures.
- Suitable for exchanging information.
- Can accommodate variable attributes.
- Useful in distributed applications.

### 3.6 Limitations

- Data validation may require additional rules.
- Inconsistent structures can complicate analysis.
- Complex nested information may require specialized queries.
- Changes in field names or formats can affect dependent applications.

### 3.7 Typical Technologies

- MongoDB.
- PostgreSQL JSONB.
- Elasticsearch.
- Apache Kafka for event-based data exchange.

**Key concept:** Semi-structured data contains organizational information without necessarily following a rigid relational schema.

## 4. Unstructured Data

**Unstructured Data** is information that does not follow a predefined tabular or consistently identifiable data model.

It may contain meaningful information, but its internal organization is not necessarily represented through conventional rows and columns.

Examples include:

- Images.
- Videos.
- Audio recordings.
- PDF documents.
- Text documents.
- Emails.
- Scanned records.

### 4.1 Main Characteristics

- No fixed relational schema.
- Complex internal representation.
- May require specialized processing.
- Often stored as files or objects.
- Can contain large volumes of information.

### 4.2 Examples

Consider an organization that maintains vehicle documentation.

Its information could include:

- Vehicle photographs.
- Insurance policy PDFs.
- Maintenance invoices.
- Inspection reports.
- Scanned documents.
- Operational videos.

These files contain valuable information, but they are not naturally represented as conventional relational records.

### 4.3 How Is Unstructured Data Stored?

A common approach is to use file systems or object storage.

Examples:

- Local file storage.
- Network Attached Storage (NAS).
- Amazon S3.
- Azure Blob Storage.
- Google Cloud Storage.

A relational database may still store metadata or references associated with these files.

For example:

| document_id | vehicle_id | file_name | storage_location |
|---|---|---|---|
| 501 | 1001 | insurance.pdf | /documents/501 |
| 502 | 1001 | inspection.jpg | /documents/502 |

The actual files can remain in external storage.

This separates the metadata management from the physical storage of the files.

### 4.4 Advantages

- Supports rich information formats.
- Suitable for images, videos, and documents.
- Flexible storage approaches.
- Can preserve original files without forcing them into relational structures.

### 4.5 Limitations

- More difficult to query using traditional SQL.
- Often requires specialized processing.
- May need additional metadata.
- Storage and indexing strategies can be more complex.
- Searching inside files may require OCR, full-text indexing, or other processing technologies.

**Key concept:** Unstructured data does not mean meaningless or completely disorganized data. It means the information does not follow a predefined conventional data structure.

## 5. Comparison of the Three Data Categories

| Characteristic | Structured | Semi-Structured | Unstructured |
|---|---|---|---|
| Fixed schema | Yes | Flexible | Not necessarily |
| Organization | Tables | Keys, tags, hierarchy | Variable |
| Relationships | Explicitly modeled | Can be represented | Usually require additional interpretation |
| Typical format | SQL tables | JSON, XML, YAML | PDF, images, audio, video |
| Querying | SQL | Specialized queries or processing | Specialized search and processing |
| Common storage | Relational DBMS | Document DBMS or relational JSON support | File or object storage |

### Important Architectural Observation

These categories are not completely isolated.

A modern information system can combine all three.

For example, a vehicle management platform may use:

- Structured data for vehicle records.
- Semi-structured data for API responses.
- Unstructured data for photographs and PDF documents.

The architecture must determine how these different forms of information are connected and managed.

## 6. Data Types vs. Data Categories

It is important to distinguish between two concepts:

**Data Category** describes how information is organized.

Examples:
- Structured.
- Semi-structured.
- Unstructured.

**Data Type** defines the kind of value a particular field can contain.

Examples:
- Integer.
- Decimal.
- Text.
- Boolean.
- Date.
- Timestamp.

For example:

```sql
CREATE TABLE vehicles (
    vehicle_id INTEGER,
    brand VARCHAR(50),
    manufacturing_year INTEGER,
    acquisition_cost NUMERIC(12, 2),
    active BOOLEAN,
    registration_date DATE
);
```

Here:

- The table represents structured data.
- `INTEGER` defines an integer field.
- `VARCHAR` defines a variable-length text field.
- `NUMERIC` defines a numeric value with precision.
- `BOOLEAN` represents a true or false value.
- `DATE` represents a calendar date.

A data category and a data type answer different questions.

## 7. Understanding Data Formats

A **Data Format** defines how information is represented or encoded so that applications can store, exchange, or interpret it.

Different formats are designed for different purposes.

| Format | Typical Purpose |
|---|---|
| CSV | Tabular data exchange |
| JSON | Application communication |
| XML | Structured document exchange |
| YAML | Configuration and structured representation |
| Parquet | Columnar analytical storage |
| Avro | Data serialization |
| ORC | Columnar analytical storage |
| PDF | Document representation |

### CSV — Comma-Separated Values

CSV represents tabular information using delimited values.

Example:

```csv
vehicle_id,brand,model,year
1001,Nissan,Frontier,2024
1002,Ford,Ranger,2023
1003,Toyota,Hilux,2025
```

CSV is commonly used to exchange information between spreadsheets, databases, and data processing tools.

### Parquet

Apache Parquet is a columnar file format designed for efficient analytical storage and processing.

Instead of organizing information primarily as a sequence of complete records, Parquet stores data in a column-oriented structure.

This can improve analytical workloads that access only selected columns from large datasets.

### Avro

Apache Avro is a data serialization system that uses schemas to describe data structures.

It is frequently used in distributed data systems and event-based architectures.

### Important Distinction

A file format is not necessarily a database.

For example:

- CSV is a file format.
- Parquet is a file format.
- PostgreSQL is a DBMS.
- MongoDB is a DBMS.

A DBMS may import, export, or process data represented in different formats.

## 8. What Is a Data Schema?

A **Data Schema** is the formal definition of how information is organized.

It can specify:

- Fields.
- Data types.
- Relationships.
- Constraints.
- Required attributes.
- Naming conventions.

Consider this example:

```sql
CREATE TABLE departments (
    department_id INTEGER PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

CREATE TABLE vehicles (
    vehicle_id INTEGER PRIMARY KEY,
    brand VARCHAR(50) NOT NULL,
    model VARCHAR(50) NOT NULL,
    department_id INTEGER REFERENCES departments(department_id)
);
```

The schema defines two tables and establishes a relationship between them.

This means a vehicle can reference an existing department through `department_id`.

### Why Is Schema Design Important?

A well-designed schema helps:

- Maintain data consistency.
- Prevent invalid information.
- Establish relationships.
- Simplify queries.
- Support future changes.
- Improve system maintainability.

Schema design will be studied in greater depth in the Data Modeling section.

Related topic: [[06 - Schema Design Principles]].

## 9. Metadata: Data About Data

**Metadata** is information that describes other data.

It helps identify, interpret, manage, and understand stored information.

Examples include:

- File name.
- Creation date.
- Author.
- File size.
- Data format.
- Database column name.
- Data type.
- Last modification date.
- Data source.

Consider a vehicle photograph.

The photograph itself is the data.

Its file name, creation timestamp, size, and storage location are examples of metadata.

Metadata is important because it helps organizations manage large collections of information.

It also plays a major role in data catalogs, data governance, and data discovery.

Related topic: [[04 - Data Governance]].

## 10. How Data Classification Influences Architecture

Choosing a storage technology requires understanding the characteristics of the information.

Consider three different requirements:

### Scenario A — Operational Vehicle Records

Requirement:

Store vehicle identification, mileage, departments, and maintenance records.

Possible solution:

A relational database such as PostgreSQL.

Reason:

The information has clearly defined fields and relationships, and the system requires data integrity and structured queries.

### Scenario B — API Integration

Requirement:

Receive vehicle information from an external application through JSON messages.

Possible solution:

An application integration layer that validates JSON and stores the relevant information in PostgreSQL or another suitable database.

Reason:

JSON allows flexible information exchange between systems.

### Scenario C — Document Management

Requirement:

Store thousands of vehicle photographs, invoices, and inspection documents.

Possible solution:

Object storage combined with a relational database for metadata and document references.

Reason:

Large files can be managed independently while their relationships with vehicles remain available in the database.

### Architectural Principle

**The nature of the data influences the storage and processing architecture.**

However, the choice must also consider:

- Query patterns.
- Data volume.
- Performance requirements.
- Consistency requirements.
- Security.
- Operational complexity.
- Cost.
- Scalability.

There is no single storage technology that is ideal for every data category.

## 11. Practical Example: Designing a Hybrid Data System

Imagine that SIAPV must eventually support vehicle records, maintenance information, documents, and external integrations.

A possible conceptual architecture could be:

```text
                 SIAPV Application
                         |
                Application Layer
                         |
              +----------+----------+
              |                     |
              v                     v
       PostgreSQL DB          Object Storage
              |                     |
              v                     v
       Structured Data       PDF / Images
              |
              v
       JSONB Metadata
              |
              v
       External Integrations
```

This is an illustrative design, not a description of SIAPV's confirmed internal implementation.

The architecture separates responsibilities:

- PostgreSQL manages structured records and relationships.
- Object storage manages files.
- JSONB can represent suitable semi-structured information.
- The application coordinates interactions between components.
- External integrations exchange information using defined formats.

The important lesson is that a single application can work with multiple data categories and storage technologies.

## 12. Knowledge Check

Answer these questions without reviewing the previous sections.

1. What distinguishes structured data from semi-structured data?
2. Why is JSON considered semi-structured?
3. Does unstructured data mean that information has no value or organization?
4. What is the difference between a data category and a data type?
5. What is a data schema?
6. What is metadata?
7. Why might an organization store photographs outside a relational database?
8. What is the difference between CSV and PostgreSQL?
9. Why is Parquet useful in analytical environments?
10. Why can a modern architecture combine several storage technologies?

## 13. Practical Exercise

### Exercise: Classify the Information in a Vehicle Management System

Classify the following information:

| Information | Data Category | Possible Format or Technology |
|---|---|---|
| Vehicle identification number | | |
| Vehicle photograph | | |
| Maintenance invoice PDF | | |
| API response containing vehicle details | | |
| Monthly mileage report in CSV | | |
| Inspection video | | |
| Vehicle database table | | |
| Application configuration file | | |

Then answer:

1. Which information would you store in PostgreSQL?
2. Which information could be stored in object storage?
3. Which information could be represented using JSON?
4. What metadata would you maintain for each document?
5. How would you connect a stored photograph to its corresponding vehicle?

### Additional Challenge

Design a simple conceptual architecture that combines:

- A relational database.
- JSON-based data exchange.
- File or object storage.
- An application interface.

Explain the responsibility of each component.

## 14. Key Takeaways

- Data can be classified as structured, semi-structured, or unstructured.
- Structured data commonly uses predefined schemas and relational tables.
- Semi-structured data uses flexible organizational elements such as keys, values, and nested objects.
- Unstructured data includes formats such as images, videos, and documents.
- Data formats define how information is represented or exchanged.
- Data types define the kind of values that individual fields can contain.
- Schemas describe how information is organized.
- Metadata provides information about other data.
- Modern architectures frequently combine different data categories and storage technologies.
- Data classification is an important input when selecting database and storage solutions.

## 15. Related Notes

### Previous Topics
- [[00 - Database Architecture Roadmap]]
- [[01 - What is a Database System]]
- [[02 - The Data Lifecycle]]

### Next Topics
- [[04 - OLTP vs OLAP]]
- [[05 - Database Architecture Fundamentals]]

### Related Modeling Topics
- [[01 - Entities and Attributes]]
- [[02 - Relationships and Cardinality]]
- [[06 - Schema Design Principles]]

### Related Data Engineering Topics
- [[02 - Data Integration]]
- [[03 - ETL and ELT]]
- [[05 - Data Warehouses]]
- [[06 - Data Lakes]]
- [[07 - Lakehouse Architecture]]

---

**Learning Objective:** Identify different data categories, understand their structures and formats, and evaluate how their characteristics influence database design and storage architecture.
