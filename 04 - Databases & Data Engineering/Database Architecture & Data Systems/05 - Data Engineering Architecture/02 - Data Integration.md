
---
title: "Data Integration"
aliases:
  - "Data Integration Fundamentals"
  - "Data Integration Architecture"
  - "Integrating Data Across Systems"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: data-integration
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - data-integration
  - data-engineering
  - data-architecture
  - data-ingestion
  - data-transformation
  - api-integration
  - database-integration
  - data-synchronization
  - change-data-capture
concepts:
  - source-system
  - target-system
  - schema-mapping
  - data-mapping
  - data-standardization
  - entity-resolution
  - batch-integration
  - real-time-integration
  - event-driven-architecture
  - change-data-capture
  - idempotency
  - data-contract
  - data-lineage
  - data-reconciliation
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - Data Architecture Overview]]"
  - "[[02 - The Data Lifecycle]]"
  - "[[03 - Types of Data]]"
  - "[[04 - OLTP vs OLAP]]"
related:
  - "[[03 - ETL and ELT]]"
  - "[[04 - Data Pipelines]]"
  - "[[05 - Data Warehouses]]"
  - "[[03 - Data Quality]]"
  - "[[04 - Data Governance]]"
tools:
  - PostgreSQL
  - SQL
  - Python
  - Pandas
  - REST APIs
  - JSON
  - Apache Kafka
  - Apache Airflow
learning_objectives:
  - "Explain the purpose and challenges of data integration."
  - "Distinguish ingestion, integration, transformation, and synchronization."
  - "Identify common data integration patterns."
  - "Understand schema mapping and entity resolution."
  - "Compare batch, API-based, event-driven, and CDC integration."
  - "Design reliable integration workflows."
  - "Explain idempotency, data contracts, and reconciliation."
  - "Evaluate security and governance considerations."
  - "Design a conceptual integration solution for SIAPV."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Data Integration

## 1. Introduction

Modern organizations rarely store all their information in a single system.

A company may use:

- An operational database to manage customers and transactions.
- An ERP system to manage financial and administrative processes.
- A CRM platform to manage customer relationships.
- APIs to communicate with external services.
- Spreadsheets to exchange information between departments.
- Cloud storage to preserve files and historical records.
- Message brokers to distribute events between applications.

Each system may contain information that is relevant to other systems.

**Data integration** is the process of connecting data from different sources and making it available in a consistent, meaningful, and usable form.

The objective is not simply to move records from one location to another. It is to preserve their meaning, resolve differences, maintain consistency, and make the resulting information useful.

### 1.1 A simple example

Imagine an organization that stores vehicle information in three systems:

| System | Information |
|---|---|
| Fleet database | Vehicle inventory number and characteristics |
| Maintenance application | Maintenance dates and service history |
| Financial system | Acquisition value and operating expenses |

Each system describes part of the same real-world vehicle.

Data integration allows these records to be associated through a common identifier and presented as a unified representation.

```text
Fleet Database
      |
      | Vehicle ID
      |
      v
Integration Layer
      ^
      |
      | Vehicle ID
      |
Maintenance System

      |
      | Vehicle ID
      v
Financial System
      |
      v
Unified Vehicle Information
```

The result is a more complete view of the vehicle.

---

## 2. Data Integration vs. Related Concepts

Several concepts are closely related but do not describe exactly the same activity.

### 2.1 Data ingestion

**Data ingestion** is the process of collecting data from a source and bringing it into a target environment.

Examples:

- Importing a CSV file.
- Reading records from PostgreSQL.
- Receiving messages from Kafka.
- Downloading information from an API.

Ingestion answers:

> How does data enter the system?

### 2.2 Data integration

**Data integration** combines information from different sources into a unified representation.

Examples:

- Matching vehicle records from two databases.
- Combining customer information from CRM and billing systems.
- Reconciling different representations of the same product.

Integration answers:

> How do different sources become a coherent information system?

### 2.3 Data transformation

**Data transformation** modifies the structure, format, or values of data.

Examples:

- Converting dates into a standard format.
- Changing currency values.
- Normalizing text.
- Calculating derived fields.
- Converting measurement units.

Transformation answers:

> How must data change to become suitable for its intended use?

### 2.4 Data synchronization

**Data synchronization** keeps information aligned between systems over time.

For example, if a vehicle's administrative status changes in one application, another application may need to receive that update.

Synchronization answers:

> How do systems remain aligned when information changes?

### 2.5 Comparison

| Concept | Main purpose |
|---|---|
| Data ingestion | Bring data into an environment |
| Data integration | Combine information from different sources |
| Data transformation | Modify data representation |
| Data synchronization | Keep systems aligned over time |

These activities often appear together in the same architecture.

---

## 3. The Main Components of Data Integration

A typical integration process involves several components.

### 3.1 Source systems

A source system produces or stores information.

Common sources include:

- Relational databases.
- NoSQL databases.
- REST APIs.
- CSV and Excel files.
- ERP and CRM applications.
- Message brokers.
- Sensors and IoT devices.
- Legacy applications.
- Cloud storage.

Every source has its own data model, access method, and update behavior.

### 3.2 Extraction or collection

The integration process retrieves information from its source.

Possible mechanisms include:

- SQL queries.
- API requests.
- File transfers.
- Event subscriptions.
- Database logs.
- Scheduled exports.

The collection method depends on the source system and the required freshness of the data.

### 3.3 Integration layer

The integration layer coordinates the exchange of information.

It may perform:

- Schema mapping.
- Identifier matching.
- Data validation.
- Deduplication.
- Error handling.
- Format conversion.
- Routing.
- Conflict resolution.

The integration layer may be implemented as a separate service, a collection of pipelines, middleware, or functionality distributed across applications.

### 3.4 Target systems

The target system receives and stores or consumes the integrated information.

Examples:

- Data warehouses.
- Operational databases.
- Reporting platforms.
- Data lakes.
- Analytical applications.
- Machine learning systems.

### 3.5 Monitoring and control

Reliable integration requires visibility into the process.

Monitoring can include:

- Number of records processed.
- Number of rejected records.
- Processing duration.
- Failed API requests.
- Duplicate records.
- Data freshness.
- Synchronization delays.

Without monitoring, an integration process may silently stop working while users continue relying on outdated information.

---

## 4. Common Data Integration Patterns

Different architectures solve different integration problems.

### 4.1 Point-to-point integration

In point-to-point integration, two systems communicate directly.

```text
System A <------> System B
```

For example:

- An inventory application sends vehicle updates directly to a maintenance application.
- A billing system requests customer information directly from a CRM.

**Advantages**

- Simple to understand.
- Suitable for a small number of systems.
- Can be implemented quickly.

**Disadvantages**

- Connections increase as systems are added.
- Business logic may become duplicated.
- Changes can affect multiple integrations.
- Monitoring becomes more complicated.

If every system must connect directly to every other system, the number of possible connections grows rapidly.

For \(n\) systems, the maximum number of bidirectional point-to-point connections is:

\[
C = \frac{n(n-1)}{2}
\]

For example, with six systems:

\[
C = \frac{6(6-1)}{2} = 15
\]

This illustrates why direct integration can become difficult to maintain at scale.

### 4.2 Hub-and-spoke integration

A central hub coordinates communication between multiple systems.

```text
             System A
                |
                |
System B --- Integration Hub --- System C
                |
                |
             System D
```

The hub may handle:

- Routing.
- Transformation.
- Validation.
- Message delivery.
- Monitoring.

**Advantages**

- Centralized integration logic.
- Reduced number of direct connections.
- Easier monitoring.
- Consistent transformation rules.

**Disadvantages**

- The hub can become a bottleneck.
- It may become a single point of failure.
- Centralized logic can become difficult to maintain.

The hub should therefore be designed with appropriate scalability and availability.

### 4.3 Enterprise Service Bus (ESB)

An **Enterprise Service Bus** is an integration architecture that provides shared communication capabilities between enterprise applications.

Typical responsibilities include:

- Message routing.
- Protocol conversion.
- Data transformation.
- Service mediation.
- Centralized integration policies.

An ESB is particularly associated with traditional enterprise integration environments.

It should not be confused with a simple database connection or an ordinary API.

### 4.4 API-led integration

API-led integration organizes communication around well-defined interfaces.

An API exposes selected functionality or data to other applications.

For example:

```http
GET /api/vehicles/13990
```

A client can request vehicle information without directly accessing the database.

The API controls:

- Authentication.
- Authorization.
- Input validation.
- Response structure.
- Error handling.
- Access policies.

API-led integration is useful when systems need controlled and reusable access to information.

### 4.5 Event-driven integration

In event-driven integration, systems communicate through events.

An event represents something that has happened.

Examples:

- VehicleRegistered.
- MaintenanceCompleted.
- InsurancePolicyUpdated.
- InventoryStatusChanged.

A producer publishes an event, and interested consumers receive it.

```text
Producer
   |
   v
Message Broker
   |
   +------> Consumer A
   |
   +------> Consumer B
   |
   +------> Consumer C
```

This approach reduces direct dependencies between systems.

The producer does not need to know every consumer.

### 4.6 Batch integration

Batch integration processes groups of records at scheduled intervals.

For example:

- Every night at 1:00 AM.
- Every hour.
- At the end of each business day.
- Whenever a new file arrives.

Batch processing is appropriate when immediate updates are unnecessary.

### 4.7 Change Data Capture (CDC)

**Change Data Capture** identifies changes occurring in a database and makes those changes available to other systems.

Instead of repeatedly extracting an entire table, CDC can capture:

- Inserts.
- Updates.
- Deletes.

This is useful for incremental data integration.

CDC can support event-driven architectures and data replication workflows.

### 4.8 Data virtualization and federation

Data virtualization allows users or applications to query information from multiple sources through a unified access layer without necessarily copying all the underlying data into one physical repository.

Federated queries may access multiple databases or external systems.

**Advantages**

- Less physical data duplication.
- Potentially faster access to distributed information.
- Unified query interfaces.

**Limitations**

- Query performance depends on source systems.
- Network latency can affect execution.
- Source availability becomes important.
- Complex joins across systems may be expensive.

### 4.9 Pattern comparison

| Pattern | Primary characteristic | Common use |
|---|---|---|
| Point-to-point | Direct connections | Small integrations |
| Hub-and-spoke | Central integration hub | Multiple enterprise systems |
| ESB | Shared enterprise messaging and mediation | Complex legacy environments |
| API-led | Controlled service interfaces | Application interoperability |
| Event-driven | Asynchronous event communication | Reactive distributed systems |
| Batch | Scheduled data exchange | Periodic reporting |
| CDC | Capture database changes | Incremental synchronization |
| Federation | Unified access to distributed sources | Distributed querying |

These patterns are not mutually exclusive. An enterprise architecture may combine APIs, CDC, event brokers, and batch pipelines.

---

## 5. Schema Mapping

One of the most important integration challenges is that different systems represent the same information differently.

A **schema** defines how information is structured.

Consider two systems.

**System A**

```json
{
  "vehicle_id": 13990,
  "registration_date": "2026-01-15",
  "status": "ACTIVE"
}
```

**System B**

```json
{
  "inventory_number": "13990",
  "date_registered": "15/01/2026",
  "operational_status": "In Service"
}
```

Both systems describe the same vehicle, but their field names and value formats differ.

Schema mapping establishes how fields correspond.

| Source field | Target field | Transformation |
|---|---|---|
| vehicle_id | inventory_number | Convert to string |
| registration_date | date_registered | Convert date format |
| status | operational_status | Map status values |

### 5.1 Field mapping

Field mapping establishes relationships between individual attributes.

Example:

```text
Source: vehicle_id
Target: inventory_number
```

### 5.2 Data type conversion

Different systems may use different data types.

Examples:

- Integer to string.
- Timestamp to date.
- Decimal to integer.
- Boolean to text.

Conversions must preserve meaning and avoid unintended precision loss.

For example, an inventory identifier may contain leading zeros. Converting it into a numeric type could remove those zeros.

### 5.3 Value mapping

The same concept may use different categorical values.

| Source value | Target value |
|---|---|
| ACTIVE | In Service |
| INACTIVE | Out of Service |
| REPAIR | Under Maintenance |

A controlled mapping prevents inconsistent interpretations.

### 5.4 Missing values

Systems may represent missing information differently:

- NULL.
- Empty string.
- Zero.
- Unknown.
- Not applicable.

These values are not always equivalent.

For example, a missing acquisition date should not automatically become January 1, 1970, or a zero value.

The integration process must define how missing information is interpreted.

---

## 6. Entity Resolution and Master Data

Two systems may contain records describing the same real-world entity but use different identifiers.

**Entity resolution** is the process of determining whether different records refer to the same entity.

### 6.1 Example

Consider these records:

| System | Identifier | Description |
|---|---|---|
| Fleet | 13990 | Shantui Motor Grader |
| Maintenance | EQ-13990 | Motor Grader Shantui |
| Finance | ASSET-00482 | Road Grader |

The descriptions differ, but the records may represent the same physical machine.

An integration process needs a reliable way to associate them.

### 6.2 Matching strategies

Possible matching strategies include:

1. Exact identifier matching.
2. Matching multiple attributes.
3. Reference tables.
4. Business rules.
5. Probabilistic matching.
6. Manual review for ambiguous records.

Exact matching is generally preferable when a stable shared identifier exists.

### 6.3 Master Data Management (MDM)

**Master Data Management** is the discipline of maintaining consistent, authoritative records for important business entities.

Common master data entities include:

- Customers.
- Employees.
- Products.
- Suppliers.
- Vehicles.
- Locations.
- Organizational units.

An organization may define one authoritative source for each entity.

For example:

```text
Vehicle Master Record
        |
        +---- Inventory Information
        |
        +---- Maintenance History
        |
        +---- Insurance Information
        |
        +---- Financial Information
```

MDM helps prevent different systems from maintaining conflicting versions of important business information.

---

## 7. Data Standardization

Data standardization ensures that information follows consistent formats, conventions, and representations.

### 7.1 Date standardization

Different systems may use:

```text
12/11/2026
2026-11-12
11-12-2026
```

These formats can be ambiguous.

A common machine-readable format is ISO 8601:

```text
2026-11-12
```

For timestamps:

```text
2026-11-12T14:30:00Z
```

The `Z` indicates UTC.

Systems should preserve timezone information when it is relevant.

### 7.2 Measurement standardization

Different sources may report measurements in different units.

For example:

```text
Source A: 150 kilometers
Source B: 93.2 miles
```

A conversion can standardize the representation.

\[
1 \text{ mile} \approx 1.60934 \text{ kilometers}
\]

The architecture should also document the unit used for each measurement.

### 7.3 Currency standardization

Financial systems may store different currencies.

A conversion requires:

- Original amount.
- Original currency.
- Target currency.
- Exchange rate.
- Exchange rate date.
- Conversion policy.

The exchange rate date matters because exchange rates change over time.

### 7.4 Text standardization

Text fields may differ because of:

- Uppercase and lowercase letters.
- Extra spaces.
- Accents.
- Abbreviations.
- Different naming conventions.

Standardization can improve matching and reporting.

However, text normalization should not destroy meaningful distinctions.

---

## 8. Integration Methods

### 8.1 Database-to-database integration

One database transfers information to another.

Possible techniques include:

- SQL queries.
- Database links.
- Scheduled exports.
- Replication.
- CDC.
- Application services.

Direct database access should be controlled carefully.

Applications should not receive unrestricted permissions simply because they need to read a few records.

### 8.2 File-based integration

Systems exchange files such as:

- CSV.
- JSON.
- XML.
- Excel workbooks.
- Parquet files.

A typical process is:

```text
Source Application
       |
       v
Export File
       |
       v
File Transfer
       |
       v
Validation
       |
       v
Transformation
       |
       v
Target Database
```

File-based integration is common when applications have limited connectivity or when users exchange information manually.

### 8.3 API integration

An **Application Programming Interface (API)** defines how software systems communicate.

REST APIs commonly use HTTP methods.

| Method | Typical purpose |
|---|---|
| GET | Retrieve information |
| POST | Create a resource or submit data |
| PUT | Replace a resource |
| PATCH | Partially update a resource |
| DELETE | Remove a resource |

Example:

```http
GET /api/vehicles/13990
Accept: application/json
Authorization: Bearer <access_token>
```

Possible response:

```json
{
  "inventory_number": "13990",
  "manufacturer": "Shantui",
  "equipment_type": "Motor Grader",
  "status": "In Service"
}
```

### 8.4 HTTP status codes

An integration client should interpret HTTP responses correctly.

| Status code | Meaning |
|---|---|
| 200 | Request successful |
| 201 | Resource created |
| 204 | Successful request with no response body |
| 400 | Invalid request |
| 401 | Authentication required or invalid |
| 403 | Access forbidden |
| 404 | Resource not found |
| 409 | Conflict |
| 429 | Rate limit exceeded |
| 500 | Internal server error |
| 503 | Service unavailable |

A failed request should not always be retried immediately.

For example, a `400 Bad Request` usually requires correcting the request rather than repeating it.

### 8.5 API pagination

Large APIs often divide responses into pages.

Example:

```http
GET /api/vehicles?page=1&limit=100
```

The client must retrieve subsequent pages until the dataset is complete.

Some APIs use:

- Page numbers.
- Offset and limit.
- Cursor-based pagination.
- Continuation tokens.

The integration process must handle pagination correctly to avoid silently omitting records.

### 8.6 Rate limits

APIs may restrict the number of requests a client can make during a specific period.

For example:

```text
Maximum: 100 requests per minute
```

A client that exceeds the limit may receive HTTP `429`.

Common strategies include:

- Request batching.
- Caching.
- Exponential backoff.
- Respecting `Retry-After`.
- Reducing unnecessary requests.

---

## 9. Event-Driven Integration in More Detail

Event-driven architectures are especially useful when systems need to react to changes without constantly polling databases.

### 9.1 Events

An event describes something that has already happened.

Example:

```json
{
  "event_id": "evt-001",
  "event_type": "MaintenanceCompleted",
  "occurred_at": "2026-11-12T15:30:00Z",
  "vehicle_id": "13990",
  "maintenance_record_id": "MNT-2026-014"
}
```

The event should contain enough information for consumers to understand what happened.

### 9.2 Producers and consumers

**Producer:** creates and publishes an event.

**Consumer:** receives and processes an event.

Example:

```text
Maintenance Application
       |
       | MaintenanceCompleted
       v
Message Broker
       |
       +------> Fleet Reporting
       |
       +------> Maintenance Analytics
       |
       +------> Notification Service
```

### 9.3 Message brokers

A message broker receives, stores, and distributes messages.

Examples include:

- Apache Kafka.
- RabbitMQ.
- Amazon SQS.
- Azure Service Bus.

A broker helps separate producers from consumers.

However, each platform has different delivery, persistence, ordering, and scalability characteristics.

### 9.4 Synchronous vs. asynchronous communication

**Synchronous communication**

The caller waits for a response.

```text
Application A ---- Request ----> Application B
Application A <--- Response ---- Application B
```

**Asynchronous communication**

The sender publishes a message and continues its work without waiting for every consumer.

```text
Application A ---> Message Broker ---> Application B
```

Synchronous communication is useful when an immediate response is required.

Asynchronous communication is useful when systems can process work independently.

---

## 10. Change Data Capture (CDC)

CDC captures changes made to a database and makes them available to other systems.

### 10.1 Why CDC is useful

Imagine a table containing one million vehicle records.

A traditional integration process might extract all one million records every hour, even if only ten records changed.

CDC can identify and transmit only the changes.

This reduces unnecessary data movement.

### 10.2 CDC techniques

**Log-based CDC**

Reads database transaction logs to identify changes.

Advantages:

- Can capture inserts, updates, and deletes.
- Often has low impact on source queries.
- Can provide detailed change information.

Limitations:

- Depends on database-specific capabilities.
- Requires careful log and retention management.
- Requires operational monitoring.

**Timestamp-based polling**

Uses a modification timestamp to identify changed records.

Example:

```sql
SELECT *
FROM vehicles
WHERE updated_at > :last_successful_sync;
```

Advantages:

- Relatively simple.
- Works with many databases.
- Does not necessarily require specialized infrastructure.

Limitations:

- Deletes may be difficult to detect.
- Timestamp accuracy matters.
- Updates can be missed if the checkpoint is poorly managed.
- Repeated extraction may occur.

**Trigger-based CDC**

Database triggers record changes in separate tables.

Advantages:

- Can capture changes directly at the database level.
- Can support custom logic.

Limitations:

- Adds work to database transactions.
- Can increase complexity.
- Poorly designed triggers may affect performance.

### 10.3 Incremental extraction

An incremental process retrieves only records that changed after a previous successful checkpoint.

Conceptually:

```text
Last Successful Checkpoint
          |
          v
Read Changes
          |
          v
Validate Records
          |
          v
Apply Changes to Target
          |
          v
Confirm Successful Processing
          |
          v
Save New Checkpoint
```

The checkpoint must advance only after the corresponding changes have been processed successfully.

Otherwise, records may be skipped.

---

## 11. Reliability: Delivery, Duplicates, and Ordering

Distributed systems cannot always guarantee that every message is delivered exactly once at every stage.

Integration architectures must account for failures and repeated delivery.

### 11.1 At-most-once delivery

A message is delivered zero or one times.

It is not retried after certain failures.

Possible consequence:

- A message may be lost.

### 11.2 At-least-once delivery

A message is delivered one or more times.

The system retries until delivery is acknowledged according to its delivery policy.

Possible consequence:

- The same message may be processed more than once.

### 11.3 Exactly-once processing

Exactly-once processing aims to ensure that a logical operation affects the final result only once.

This is difficult to guarantee across multiple independent systems.

Many practical architectures combine at-least-once delivery with idempotent processing and transactional mechanisms.

### 11.4 Idempotency

An operation is **idempotent** when repeating it produces the same final state as performing it once.

Mathematically:

\[
f(f(x)) = f(x)
\]

Example:

Setting a vehicle status to `In Service` is idempotent if repeated execution does not create additional effects.

By contrast, adding 100 to a balance every time an event is received is not idempotent.

### 11.5 Idempotency keys

An idempotency key identifies a logical operation.

Example:

```json
{
  "idempotency_key": "maintenance-13990-2026-014",
  "vehicle_id": "13990",
  "maintenance_record_id": "MNT-2026-014"
}
```

Before applying the operation, the target system checks whether that key has already been processed.

If it has, the operation can be safely recognized as a duplicate.

### 11.6 Message ordering

Messages may arrive in a different order from the order in which events occurred.

Example:

```text
Event 1: Vehicle status changed to Under Maintenance
Event 2: Maintenance completed
```

If Event 2 arrives before Event 1, a consumer may temporarily or permanently calculate an incorrect state.

Possible strategies include:

- Sequence numbers.
- Event timestamps.
- Partitioning by entity identifier.
- Version numbers.
- State validation.
- Reconciliation.

Timestamps alone may not always be sufficient because clocks can differ between systems.

### 11.7 Retries and exponential backoff

A retry repeats an operation after a temporary failure.

Exponential backoff increases the waiting period between attempts.

A simplified model is:

\[
t_n = \min(t_{\max}, t_0 \times 2^n)
\]

Where:

- \(t_n\) = waiting time before attempt \(n\).
- \(t_0\) = initial delay.
- \(t_{\max}\) = maximum delay.
- \(n\) = retry number.

Random jitter can be added to prevent many clients from retrying simultaneously.

### 11.8 Dead-letter queues

A **Dead-Letter Queue (DLQ)** stores messages that cannot be processed successfully after the configured retry policy.

A DLQ supports:

- Failure investigation.
- Controlled reprocessing.
- Error classification.
- Operational monitoring.

Messages should not simply remain in a DLQ indefinitely without review.

---

## 12. Data Reconciliation

Data reconciliation compares information across systems to identify inconsistencies.

For example, a source system may report 200 vehicles while a target system contains 198.

The difference requires investigation.

### 12.1 Reconciliation checks

Common checks include:

- Record counts.
- Unique identifier comparisons.
- Missing records.
- Duplicate records.
- Value differences.
- Total financial amounts.
- Date ranges.
- Status distributions.

### 12.2 Record count comparison

```sql
SELECT COUNT(*) AS total_vehicles
FROM vehicles;
```

The result can be compared with the corresponding target table.

### 12.3 Missing-record detection

Suppose a source table is called `source_vehicles` and a target table is called `target_vehicles`.

```sql
SELECT s.inventory_number
FROM source_vehicles AS s
LEFT JOIN target_vehicles AS t
    ON s.inventory_number = t.inventory_number
WHERE t.inventory_number IS NULL;
```

This identifies source records that are missing from the target.

### 12.4 Value comparison

```sql
SELECT
    s.inventory_number,
    s.status AS source_status,
    t.status AS target_status
FROM source_vehicles AS s
JOIN target_vehicles AS t
    ON s.inventory_number = t.inventory_number
WHERE s.status IS DISTINCT FROM t.status;
```

`IS DISTINCT FROM` is useful in PostgreSQL because it handles `NULL` comparisons consistently.

### 12.5 Reconciliation frequency

Reconciliation may occur:

- After every batch.
- At the end of each day.
- After a migration.
- Following a system outage.
- Before financial reporting.
- Periodically as a control process.

---

## 13. Data Contracts and Schema Evolution

A data contract defines expectations between systems that exchange information.

It describes what a producer provides and what a consumer can expect.

A data contract may define:

- Field names.
- Data types.
- Required fields.
- Optional fields.
- Valid values.
- Constraints.
- Ownership.
- Update frequency.
- Compatibility rules.
- Quality expectations.

### 13.1 Example data contract

```yaml
dataset: vehicles
owner: fleet-management
version: 1.0.0

fields:
  - name: inventory_number
    type: string
    required: true
    unique: true

  - name: manufacturer
    type: string
    required: true

  - name: status
    type: string
    required: true
    allowed_values:
      - In Service
      - Under Maintenance
      - Out of Service

  - name: updated_at
    type: timestamp
    required: true
```

### 13.2 Schema evolution

Schemas change over time.

A system may introduce a new field:

```text
vehicle_category
```

Or rename an existing field:

```text
status -> operational_status
```

These changes can break consumers that depend on the original schema.

### 13.3 Backward compatibility

A change is backward-compatible when existing consumers can continue working without modification.

For example, adding an optional field is often backward-compatible if consumers ignore unknown fields.

Removing a required field is generally a breaking change.

### 13.4 Versioning

A contract may use explicit versions:

```text
/api/v1/vehicles
/api/v2/vehicles
```

Versioning helps manage changes, but it also introduces maintenance responsibilities.

A good architecture avoids unnecessary breaking changes and communicates them clearly.

---

## 14. Data Integration Security

Integration creates additional pathways through which data can be accessed or transferred.

Security must therefore be designed into the integration process.

### 14.1 Authentication

Authentication verifies the identity of a user, application, or service.

Common mechanisms include:

- API keys.
- OAuth 2.0.
- Mutual TLS.
- Service accounts.
- Client certificates.

### 14.2 Authorization

Authorization determines which resources an authenticated identity can access.

An integration service that only needs to read vehicle records should not automatically receive permission to delete every record in the database.

### 14.3 Encryption

Data should be protected while moving between systems.

TLS is commonly used to encrypt network communication.

Sensitive information should also be protected when stored.

### 14.4 Secret management

Credentials and API keys should not be embedded directly in source code.

Use appropriate secret-management mechanisms, environment configuration, or managed identity services.

### 14.5 Data minimization

Only transfer the information required by the receiving system.

For example, a maintenance application may need:

- Inventory number.
- Equipment type.
- Operational status.

It may not need unrelated financial or personal information.

### 14.6 Auditability

Maintain records of important integration activities:

- Which system initiated the transfer.
- When the transfer occurred.
- Which records were processed.
- Whether processing succeeded.
- Which identity performed the operation.

Audit logs should avoid exposing sensitive values unnecessarily.

---

## 15. Choosing an Integration Pattern

The correct pattern depends on business requirements and technical constraints.

Consider the following factors.

| Requirement | Possible approach |
|---|---|
| Data changes once per day | Batch integration |
| Immediate response is required | Synchronous API |
| Multiple systems need to react to events | Event-driven architecture |
| Large database changes must be captured incrementally | CDC |
| A few systems exchange simple data | Point-to-point |
| Many enterprise systems require coordinated integration | Hub-and-spoke or ESB |
| Distributed data must be queried without full replication | Federation or virtualization |

A pattern should not be selected merely because it is popular.

It should address the actual requirements for latency, reliability, security, cost, and maintainability.

### 15.1 Important design questions

Before designing an integration, ask:

1. What are the source systems?
2. What is the target system?
3. Which system owns each field?
4. How frequently does data change?
5. How quickly must changes become available?
6. How are entities identified across systems?
7. What happens when a source is unavailable?
8. How are duplicate messages handled?
9. How are errors monitored and recovered?
10. How is data quality verified?

---

## 16. Practical Example: SIAPV Data Integration

SIAPV is an internal application for managing vehicle and machinery information.

Its PostgreSQL database stores information related to approximately 200 units.

Relevant information includes:

- Unique inventory numbers.
- Equipment characteristics.
- Mileage.
- Operating hours.
- Maintenance.
- Incidents.
- Insurance.
- Administrative records.
- Operational reports.

The application provides a centralized representation of information that may otherwise be distributed across administrative documents and operational records.

### 16.1 Conceptual integration scenario

Imagine that a separate maintenance system is introduced.

The maintenance system records service events, while SIAPV maintains the main vehicle inventory.

The integration objective is to associate maintenance records with the corresponding SIAPV vehicle.

```text
SIAPV
PostgreSQL
    |
    | Inventory number
    |
    v
Integration Process
    |
    +---- Validate identifier
    |
    +---- Validate date
    |
    +---- Validate maintenance data
    |
    +---- Detect duplicates
    |
    v
Maintenance Data
    |
    v
Integrated Reporting
```

This is a conceptual design example, not a claim that SIAPV currently exchanges data with an external maintenance system.

### 16.2 Define the shared identifier

The inventory number should act as the reference identifier when both systems use the same official inventory record.

Example:

```text
SIAPV inventory number: 13990
Maintenance equipment reference: 13990
```

If another system uses a different identifier, a mapping table may be necessary.

### 16.3 Example integration table

```sql
CREATE TABLE maintenance_integration (
    integration_id BIGINT GENERATED ALWAYS AS IDENTITY
        PRIMARY KEY,

    inventory_number VARCHAR(30) NOT NULL,

    external_record_id VARCHAR(100) NOT NULL,

    maintenance_date DATE NOT NULL,

    description TEXT,

    source_system VARCHAR(100) NOT NULL,

    received_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_external_maintenance
        UNIQUE (source_system, external_record_id)
);
```

The unique constraint prevents the same external maintenance record from being inserted repeatedly for the same source system.

A foreign key to the SIAPV vehicle table could also be introduced if the actual schema and integration requirements support it.

### 16.4 Example validation

```sql
SELECT
    m.inventory_number,
    m.external_record_id,
    m.maintenance_date
FROM maintenance_integration AS m
LEFT JOIN vehicles AS v
    ON v.inventory_number = m.inventory_number
WHERE v.inventory_number IS NULL;
```

This query identifies maintenance records that reference vehicles not found in the target vehicle table.

The actual table and column names must be adapted to the SIAPV schema.

### 16.5 Integration controls

A production-ready process should define:

- Which system owns the vehicle inventory.
- Which system owns maintenance records.
- How new vehicles are registered.
- How corrections are communicated.
- How duplicate records are detected.
- How failed imports are recovered.
- How record counts are reconciled.
- Who is responsible for data quality.

### 16.6 Architectural lesson

The main challenge is not simply connecting two databases.

The difficult questions concern ownership, identity, consistency, validation, reliability, and operational responsibility.

These are the foundations of dependable data integration.

---

## 17. Common Data Integration Mistakes

### 17.1 Assuming identical field names mean identical meaning

Two fields named `status` may represent different business concepts.

Always verify their definitions.

### 17.2 Using unstable identifiers

Names and descriptions can change.

Prefer stable identifiers when available.

### 17.3 Ignoring duplicate delivery

Distributed systems may deliver the same message more than once.

Design idempotent processing.

### 17.4 Advancing checkpoints too early

If a checkpoint advances before successful processing, records may be lost.

Update checkpoints only after the corresponding work is safely completed.

### 17.5 Ignoring schema changes

A source application may change its API or database structure.

Use contracts, versioning, and compatibility testing.

### 17.6 Retrying every error

Some failures are temporary; others require correcting the request or data.

Classify errors before retrying.

### 17.7 Ignoring data ownership

If two systems can independently modify the same field, conflicts may occur.

Define an authoritative source or an explicit conflict-resolution policy.

### 17.8 Integrating without monitoring

A process can fail silently.

Track freshness, failures, rejected records, processing volume, and reconciliation results.

---

## 18. Knowledge Check

Answer these questions without consulting the previous sections.

1. What is the difference between data ingestion and data integration?
2. Why can point-to-point integration become difficult to maintain?
3. What problem does schema mapping solve?
4. Why are stable identifiers important?
5. What is the difference between synchronous and asynchronous communication?
6. What does a message broker do?
7. How does CDC reduce unnecessary data extraction?
8. What is the difference between at-most-once and at-least-once delivery?
9. Why is idempotency important?
10. What information should a data contract define?
11. Why should an integration process include reconciliation?
12. How can schema evolution break downstream applications?

---

## 19. Practical Exercises

### Exercise 1: Identify integration requirements

Imagine an organization has:

- A PostgreSQL fleet database.
- An Excel maintenance register.
- A financial application.
- A reporting platform.

Identify:

1. The source systems.
2. The target systems.
3. The shared business entities.
4. The possible identifiers.
5. The information that must be standardized.
6. The expected integration frequency.

### Exercise 2: Design a field mapping

Create a mapping table for the following fields:

```text
Source:
vehicle_id
registration_date
status
operating_hours

Target:
inventory_number
date_registered
operational_status
horometer
```

Define:

- Source data type.
- Target data type.
- Conversion rule.
- Validation rule.

### Exercise 3: Select an integration pattern

Choose an appropriate integration pattern for each scenario.

**Scenario A**

A reporting database receives updated information every night.

**Scenario B**

A maintenance application must immediately request a vehicle's current status.

**Scenario C**

Three independent applications must react when a vehicle is registered.

**Scenario D**

A data warehouse needs to capture only database changes instead of extracting entire tables.

Explain your choices.

### Exercise 4: Design an idempotent process

A maintenance event can be delivered multiple times.

Define:

- A unique event identifier.
- A duplicate detection strategy.
- A database constraint.
- The behavior when an event is received again.

### Exercise 5: Reconciliation

Create two tables containing vehicle inventory numbers.

Introduce:

- One missing target record.
- One duplicate source record.
- One status mismatch.

Write SQL queries to detect each problem.

### Exercise 6: SIAPV integration design

Design a conceptual integration between SIAPV and an external maintenance application.

Your design must include:

- Source and target systems.
- Shared identifiers.
- Data mapping.
- Validation.
- Error handling.
- Duplicate prevention.
- Reconciliation.
- Security controls.

---

## 20. Glossary

| Term | Definition |
|---|---|
| Data Integration | Combining data from different sources into a coherent representation |
| Data Ingestion | Bringing data into a target environment |
| Data Transformation | Modifying data structure or values |
| Data Synchronization | Keeping information aligned across systems |
| Source System | System that provides data |
| Target System | System that receives or consumes data |
| Schema Mapping | Defining correspondences between different data structures |
| Entity Resolution | Identifying records that represent the same real-world entity |
| Master Data | Core business information shared across systems |
| MDM | Master Data Management |
| API | Interface that enables software communication |
| REST | Architectural style commonly used for web APIs |
| Message Broker | Middleware that receives and distributes messages |
| Event | Record of something that happened |
| CDC | Change Data Capture |
| Idempotency | Property of an operation that produces the same result when repeated |
| Checkpoint | Recorded processing position used to resume an integration workflow |
| Dead-Letter Queue | Storage for messages that cannot be processed successfully |
| Data Reconciliation | Comparing datasets to identify inconsistencies |
| Data Contract | Agreement defining expectations for exchanged data |
| Schema Evolution | Changes to a data structure over time |
| Data Lineage | Record of where data comes from and how it changes |
| Data Federation | Querying information across multiple sources through a unified layer |
| Data Standardization | Applying consistent formats and conventions |
| Data Ownership | Responsibility for the authoritative management of information |

---

## 21. Connections to Other Topics

### Previous concepts

- [[01 - Data Architecture Overview]]
- [[02 - The Data Lifecycle]]
- [[03 - Types of Data]]
- [[04 - OLTP vs OLAP]]
- [[05 - Database Architecture Fundamentals]]

### Related concepts

- [[03 - ETL and ELT]]
- [[04 - Data Pipelines]]
- [[05 - Data Warehouses]]
- [[03 - Data Quality]]
- [[04 - Data Governance]]
- [[01 - Database Security Fundamentals]]
- [[02 - Authentication and Authorization]]

### Practical applications

- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]

---

## 22. Key Takeaways

1. Data integration connects information distributed across different systems.
2. Ingestion, integration, transformation, and synchronization are related but distinct activities.
3. Integration patterns include point-to-point, hub-and-spoke, API-led, event-driven, batch, CDC, and federation.
4. Schema mapping and standardization preserve the meaning of information across systems.
5. Stable identifiers are essential for matching entities.
6. Distributed integration must account for retries, duplicates, ordering, and failures.
7. Idempotency makes repeated processing safer.
8. Data contracts help producers and consumers manage expectations.
9. Reconciliation verifies that integrated information remains consistent.
10. Security, monitoring, and data ownership are fundamental architectural requirements.

**Next topic:** [[03 - ETL and ELT]]

The next note will explain how data is extracted, transformed, and loaded into analytical environments, including the architectural differences between ETL and ELT.
