
---
title: "Data Architecture Overview"
aliases:
  - "Data Architecture Fundamentals"
  - "Enterprise Data Architecture"
  - "Data Engineering Architecture Overview"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: data-architecture
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - data-architecture
  - data-engineering
  - data-platform
  - data-management
  - data-integration
  - data-pipeline
  - data-warehouse
  - data-lake
  - data-governance
  - analytics
concepts:
  - data-architecture
  - data-platform
  - data-ecosystem
  - data-source
  - data-ingestion
  - data-integration
  - data-transformation
  - data-storage
  - data-serving
  - data-consumption
  - data-pipeline
  - data-flow
  - data-lineage
  - metadata
  - data-quality
  - data-governance
  - batch-processing
  - stream-processing
  - operational-system
  - analytical-system
  - data-warehouse
  - data-lake
  - lakehouse
  - data-mesh
prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[02 - The Data Lifecycle]]"
  - "[[03 - Types of Data]]"
  - "[[04 - OLTP vs OLAP]]"
  - "[[05 - Database Architecture Fundamentals]]"
  - "[[02 - Database Scalability]]"
  - "[[07 - Backup and Disaster Recovery]]"
related:
  - "[[02 - Data Integration]]"
  - "[[03 - ETL and ELT]]"
  - "[[04 - Data Pipelines]]"
  - "[[05 - Data Warehouses]]"
  - "[[06 - Data Lakes]]"
  - "[[07 - Lakehouse Architecture]]"
  - "[[03 - Data Quality]]"
  - "[[04 - Data Governance]]"
tools:
  - PostgreSQL
  - SQL
  - Python
  - Pandas
  - Apache Spark
  - Apache Kafka
  - Apache Airflow
  - dbt
  - Docker
  - Power BI
learning_objectives:
  - "Define data architecture and explain its role in an organization."
  - "Distinguish data architecture, data engineering, and data management."
  - "Identify the main components of a data ecosystem."
  - "Understand how data moves from operational sources to analytical systems."
  - "Compare batch processing and stream processing."
  - "Understand data ingestion, integration, transformation, storage, and serving."
  - "Explain the role of metadata, lineage, quality, and governance."
  - "Identify common data architecture patterns."
  - "Understand the responsibilities of a data engineer."
  - "Design a conceptual data architecture for an application."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Data Architecture Overview

## 1. Introduction

Modern organizations generate and consume information through many different systems.

A company may use:

- Transactional databases.
- Enterprise applications.
- Financial systems.
- Inventory management platforms.
- Customer relationship management systems.
- Web applications.
- Mobile applications.
- External APIs.
- Spreadsheets.
- Sensors and industrial equipment.
- Cloud services.

Each system generates or stores information for a particular purpose.

However, organizations frequently need to combine information from multiple sources to answer broader questions.

For example, an organization may want to understand:

- How much inventory is available?
- Which vehicles require maintenance?
- How much money was spent on repairs?
- Which suppliers provide the most services?
- How have operating costs changed over time?
- Which equipment has the highest utilization?
- What operational trends can be identified from historical records?

The information required to answer these questions may be distributed across several systems.

This creates an architectural challenge:

**How can an organization collect, integrate, organize, store, process, and distribute data so that it becomes reliable and useful?**

This is the central problem addressed by Data Architecture.

## 2. What Is Data Architecture?

**Data Architecture is the discipline of designing and organizing the structures, technologies, processes, and rules through which an organization manages its data.**

It defines how data is:

- Generated.
- Collected.
- Transferred.
- Integrated.
- Stored.
- Transformed.
- Protected.
- Governed.
- Distributed.
- Consumed.

Data architecture establishes the overall structure of an organization's data environment.

It connects business requirements with technical systems.

### 2.1 A simple example

Imagine an organization with three independent systems:

1. A vehicle inventory application.
2. A maintenance management application.
3. A financial management system.

Each system contains information about different aspects of organizational operations.

The organization wants to create a consolidated report showing maintenance expenses by vehicle.

To achieve this, the systems must exchange information.

A data architecture defines:

- Which systems are the sources.
- How their information is extracted.
- How records are matched.
- How inconsistent values are standardized.
- Where integrated information is stored.
- How reports access the resulting data.
- How data quality and security are maintained.

Without an architecture, these connections may become isolated scripts and manually maintained spreadsheets.

With an architecture, data movement becomes an organized and maintainable process.

## 3. Data Architecture vs. Data Engineering vs. Data Management

These terms are closely related, but they describe different responsibilities.

### 3.1 Data Architecture

Data Architecture defines the overall structure and design of the data environment.

It answers questions such as:

- Which systems store the data?
- How should systems communicate?
- Where should analytical information be stored?
- Which technologies should be used?
- How should data be protected?
- How should the architecture scale?

It focuses on the design of the complete data ecosystem.

### 3.2 Data Engineering

Data Engineering implements and operates the technical systems that move, transform, and prepare data.

It involves:

- Building data pipelines.
- Extracting information from sources.
- Integrating datasets.
- Transforming data.
- Automating workflows.
- Managing storage systems.
- Monitoring pipeline execution.
- Improving performance and reliability.

A Data Engineer translates architectural requirements into working data infrastructure.

### 3.3 Data Management

Data Management concerns the policies, practices, responsibilities, and processes used to manage data throughout its lifecycle.

It includes:

- Data governance.
- Data quality.
- Metadata management.
- Security.
- Privacy.
- Data ownership.
- Retention.
- Compliance.

### 3.4 Comparison

| Discipline | Main question | Primary focus |
|---|---|---|
| Data Architecture | How should the data environment be designed? | Structure and technical decisions |
| Data Engineering | How do we build and operate data systems? | Pipelines and infrastructure |
| Data Management | How should data be controlled and maintained? | Policies, quality, and responsibilities |

These disciplines work together.

A data architecture without implementation remains a design.

Data engineering without architectural direction can produce disconnected and difficult-to-maintain systems.

Data management without technical implementation may fail to enforce organizational rules.

## 4. The Data Ecosystem

A data ecosystem is the collection of systems, technologies, people, processes, and information that interact to generate and consume data.

A typical enterprise data ecosystem may contain:

1. Data sources.
2. Data ingestion.
3. Data integration.
4. Data processing.
5. Data storage.
6. Data serving.
7. Data consumption.
8. Governance and security.

### 4.1 High-level architecture

```text
                  DATA SOURCES
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
    Databases       Applications    External APIs
        |              |              |
        +--------------+--------------+
                       |
                       v
                DATA INGESTION
                       |
                       v
                DATA INTEGRATION
                       |
                       v
                DATA PROCESSING
                       |
                       v
                 DATA STORAGE
                       |
                       v
                  DATA SERVING
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Reports   Analytics    ML / AI
```

Governance, security, quality, metadata, and monitoring apply across the entire architecture.

This diagram represents a conceptual architecture. Real systems may combine, separate, or repeat these components.

## 5. Data Sources

A data source is any system, application, device, or repository from which data originates or can be collected.

### 5.1 Operational databases

Operational databases store information required by business applications.

Examples:

- PostgreSQL.
- MySQL.
- Microsoft SQL Server.
- Oracle Database.
- IBM Db2.

They commonly support transactional workloads.

See:

- [[04 - OLTP vs OLAP]]
- [[01 - Database Management Systems]]

### 5.2 Enterprise applications

Organizations may use specialized systems for:

- Enterprise Resource Planning (ERP).
- Customer Relationship Management (CRM).
- Human Resources Management Systems (HRMS).
- Inventory management.
- Accounting.
- Logistics.
- Maintenance.

These applications may expose their data through databases, APIs, exports, or integration services.

### 5.3 External APIs

An API allows one system to request information or operations from another system.

For example, a financial application may obtain exchange rates from an external service.

A data engineering system can periodically retrieve this information and store it for analysis.

### 5.4 Files

Data may also originate from:

- CSV files.
- Excel workbooks.
- JSON documents.
- XML files.
- Parquet files.
- Text files.
- PDF documents.

File-based integration is common in organizations where systems do not provide direct database access or APIs.

### 5.5 Events and sensors

Some systems generate information continuously.

Examples:

- Industrial sensors.
- Vehicle telemetry.
- Application logs.
- Clickstream events.
- IoT devices.
- Network monitoring systems.

These sources can produce high-volume or time-sensitive data.

### 5.6 Source characteristics

Different sources have different technical characteristics.

| Source | Data format | Typical access method |
|---|---|---|
| PostgreSQL | Relational tables | SQL connection |
| REST API | JSON or XML | HTTP requests |
| Excel | Worksheets and cells | File processing |
| CSV | Delimited text | File ingestion |
| IoT sensor | Events or measurements | Message broker |
| Application logs | Text or structured events | Log collector |

An important architectural responsibility is understanding the source before designing its integration.

## 6. Data Ingestion

Data ingestion is the process of collecting data from source systems and bringing it into another environment for further processing or storage.

It is one of the first stages of a data architecture.

### 6.1 Common ingestion methods

**Database extraction**

A pipeline connects to a database and retrieves records using SQL.

**API ingestion**

A program sends HTTP requests to an API and receives structured responses.

**File ingestion**

A process reads CSV, Excel, JSON, or other files.

**Event ingestion**

A system receives events through a message broker or streaming platform.

### 6.2 Batch ingestion

Batch ingestion collects and transfers data in groups at scheduled intervals.

Example:

An application exports all new maintenance records every night at 02:00.

The data is processed and stored in an analytical database.

```text
Source Database
      |
      v
Scheduled Extraction
      |
      v
Batch File
      |
      v
Data Processing
      |
      v
Analytical Storage
```

Batch ingestion is useful when immediate processing is unnecessary.

### 6.3 Streaming ingestion

Streaming ingestion collects and processes data continuously or in near real time.

Example:

A fleet monitoring system sends vehicle location events every few seconds.

A streaming platform receives and distributes these events to downstream systems.

```text
Vehicle Sensors
      |
      v
Event Producer
      |
      v
Message Broker
      |
      v
Stream Processing
      |
      v
Data Storage
```

Streaming architectures are useful when information must be processed quickly.

### 6.4 Batch vs. streaming

| Characteristic | Batch ingestion | Streaming ingestion |
|---|---|---|
| Data collection | Groups of records | Continuous events |
| Processing | Scheduled or periodic | Continuous or event-driven |
| Latency | Usually higher | Usually lower |
| Complexity | Often simpler | Often more complex |
| Example | Daily financial report | Real-time vehicle monitoring |

The correct approach depends on business requirements.

Not every dataset needs real-time processing.

## 7. Data Integration

Data integration combines information from different sources into a unified and useful representation.

Imagine that three systems identify the same vehicle differently:

| System | Vehicle identifier |
|---|---|
| Inventory | INV-1025 |
| Maintenance | VEH-1025 |
| Finance | 1025 |

The organization must determine whether these identifiers represent the same physical vehicle.

This requires integration logic.

### 7.1 Integration activities

Common activities include:

- Connecting to multiple sources.
- Mapping fields.
- Standardizing formats.
- Matching records.
- Resolving duplicates.
- Combining datasets.
- Reconciling inconsistent values.
- Preserving source information.

### 7.2 Schema mapping

Schema mapping defines how fields from one data structure correspond to fields in another.

Example:

```text
Source A                    Target
--------                    ------
vehicle_code       ------>  vehicle_id
equipment_name     ------>  name
current_status     ------>  operational_status
```

The source and target do not need identical structures.

The integration process defines the transformation.

### 7.3 Data standardization

Consider these values:

```text
Available
available
AVAILABLE
A
```

They may represent the same business state.

An integration process may standardize them into a single controlled value:

```text
AVAILABLE
```

Standardization improves consistency across systems.

### 7.4 Data integration vs. data ingestion

These concepts should not be confused.

| Ingestion | Integration |
|---|---|
| Collects data from sources | Combines and reconciles data |
| Moves data into an environment | Establishes relationships and consistent representations |
| Focuses on acquisition | Focuses on unification |

A system can ingest data without fully integrating it.

For example, it may collect three CSV files and store them separately without reconciling their identifiers.

See:

[[02 - Data Integration]]

## 8. Data Processing and Transformation

Data processing applies operations to collected information to prepare it for storage, analysis, or consumption.

Common operations include:

- Filtering.
- Sorting.
- Aggregation.
- Type conversion.
- Validation.
- Deduplication.
- Joining datasets.
- Calculating derived fields.
- Applying business rules.

### 8.1 Example

Suppose a maintenance dataset contains:

| Vehicle | Maintenance cost | Date |
|---|---:|---|
| V-001 | 1200 | 2026-01-10 |
| V-001 | 800 | 2026-02-12 |
| V-002 | 1500 | 2026-02-15 |
| V-002 | 500 | 2026-03-03 |

A transformation may calculate the total maintenance cost per vehicle.

```sql
SELECT
    vehicle,
    SUM(maintenance_cost) AS total_maintenance_cost
FROM maintenance
GROUP BY vehicle;
```

Result:

| Vehicle | Total maintenance cost |
|---|---:|
| V-001 | 2000 |
| V-002 | 2000 |

The original records remain the source of the calculation.

The transformed result provides a more useful analytical representation.

### 8.2 Transformation requirements

Transformations should be:

- Consistent.
- Documented.
- Testable.
- Repeatable.
- Traceable.
- Appropriate for the business meaning of the data.

A transformation that changes a value without documenting the rule can introduce analytical errors.

See:

[[03 - ETL and ELT]]

## 9. Data Storage Layers

Data architectures often separate storage according to the purpose of the information.

### 9.1 Operational storage

Operational databases support application transactions.

Examples:

- Creating an equipment record.
- Updating a maintenance status.
- Registering a payment.
- Assigning a vehicle to a department.

These systems are generally optimized for transactional workloads.

### 9.2 Staging area

A staging area is an intermediate location where incoming data is temporarily stored before further processing.

It can be used to:

- Preserve extracted source data.
- Validate incoming records.
- Compare source and target datasets.
- Handle failed ingestion.
- Prepare transformations.

Example:

```text
External API
     |
     v
Staging Table
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

A staging area can simplify the integration process and make failures easier to investigate.

### 9.3 Data warehouse

A data warehouse is a storage system designed primarily for analytical workloads.

It integrates and organizes information for reporting and analysis.

Examples of analytical questions:

- How much did maintenance cost each year?
- Which vehicle categories require the most repairs?
- How has equipment utilization changed?
- Which departments generate the highest operational expenses?

Data warehouses often use dimensional modeling and structures such as fact and dimension tables.

See:

[[05 - Data Warehouses]]

### 9.4 Data lake

A data lake stores large volumes of data in its original or minimally transformed form.

It can contain:

- Structured data.
- Semi-structured data.
- Unstructured data.
- Logs.
- Images.
- Documents.
- Sensor data.

Data lakes are often built using scalable object storage.

See:

[[06 - Data Lakes]]

### 9.5 Lakehouse

A lakehouse combines characteristics of data lakes and data warehouses.

It aims to support flexible storage while providing capabilities such as:

- Structured table management.
- Schema enforcement.
- Transactional consistency.
- Analytical querying.
- Data governance.

See:

[[07 - Lakehouse Architecture]]

## 10. Data Serving and Consumption

Data serving makes processed information available to systems and users.

The data must be accessible in a format appropriate for its intended use.

### 10.1 Business Intelligence

Business Intelligence (BI) systems present data through:

- Dashboards.
- Reports.
- Charts.
- KPIs.
- Aggregated metrics.

Examples of BI tools include:

- Power BI.
- Tableau.
- Looker.

### 10.2 Analytical applications

Applications may query integrated datasets to support:

- Operational planning.
- Financial analysis.
- Resource allocation.
- Performance monitoring.
- Forecasting.

### 10.3 Machine Learning

Machine Learning systems consume prepared data to train models and generate predictions.

Examples:

- Predicting equipment failures.
- Forecasting maintenance costs.
- Detecting anomalous transactions.
- Estimating demand.
- Classifying documents.

Machine learning depends heavily on data quality and reliable data preparation.

### 10.4 APIs and data services

Data can also be exposed through APIs so other applications can consume it.

For example:

An internal application may request an aggregated equipment utilization metric from a data service rather than connecting directly to an analytical database.

This creates a controlled access layer.

## 11. Batch and Stream Processing

Data processing architectures can be organized according to how quickly information must be handled.

### 11.1 Batch processing

Batch processing executes operations on a collection of records.

Example:

Every night, an organization calculates the total maintenance expenditure for the previous day.

Typical characteristics:

- Scheduled execution.
- Large datasets.
- Periodic results.
- Easier reproducibility in many workflows.

### 11.2 Stream processing

Stream processing handles events continuously as they arrive.

Example:

A vehicle monitoring system receives telemetry and detects when an engine temperature exceeds a defined threshold.

Typical characteristics:

- Continuous input.
- Low-latency processing.
- Event-driven execution.
- State and time-window management.

### 11.3 Architecture comparison

| Dimension | Batch | Stream |
|---|---|---|
| Input | Bounded dataset | Continuous event stream |
| Execution | Scheduled | Continuous |
| Latency | Seconds to hours, depending on workflow | Often milliseconds to seconds |
| Processing model | Process a collected group | Process incoming events |
| Typical use | Historical reports | Monitoring and alerts |

The distinction is conceptual. Real architectures can combine both processing models.

### 11.4 Lambda and Kappa architectures

Two architectural patterns are associated with large-scale data processing.

**Lambda architecture**

Uses separate processing paths:

- Batch layer.
- Speed or streaming layer.
- Serving layer.

The batch layer processes historical data, while the streaming layer handles recent events.

**Kappa architecture**

Uses a streaming-oriented processing model in which historical data can be replayed through the same processing pipeline.

Both patterns have tradeoffs involving complexity, replay, consistency, and operational maintenance.

They are not mandatory designs for every data platform.

## 12. Data Pipelines

A data pipeline is a sequence of processes that moves data from one or more sources to a destination.

A pipeline may include:

1. Extraction.
2. Ingestion.
3. Validation.
4. Transformation.
5. Storage.
6. Publication.

Example:

```text
PostgreSQL
    |
    v
Python Extraction
    |
    v
Raw Dataset
    |
    v
Data Validation
    |
    v
Transformation
    |
    v
Data Warehouse
    |
    v
Power BI
```

### 12.1 Pipeline orchestration

Orchestration coordinates the execution of tasks.

For example:

- Run extraction at 01:00.
- Wait for successful completion.
- Execute data validation.
- Run transformations.
- Refresh analytical tables.
- Publish a completion notification.

Apache Airflow is an example of a workflow orchestration platform.

### 12.2 Pipeline reliability

A production pipeline should consider:

- Retries.
- Failure handling.
- Logging.
- Monitoring.
- Data validation.
- Scheduling.
- Dependency management.
- Idempotency.
- Recovery from partial execution.

A pipeline is not reliable merely because it works once.

It must operate predictably over time.

See:

[[04 - Data Pipelines]]

## 13. Metadata and Data Lineage

Data architecture must describe not only the data itself but also its context.

### 13.1 Metadata

Metadata is information that describes other data.

Examples:

| Data element | Metadata example |
|---|---|
| Table | Table name and description |
| Column | Data type and business meaning |
| Dataset | Owner and creation date |
| File | Format and encoding |
| Pipeline | Execution schedule |
| Database | Version and environment |

Metadata helps users and systems understand how information is structured and maintained.

### 13.2 Technical metadata

Technical metadata describes implementation details.

Examples:

- Column names.
- Data types.
- Index definitions.
- Table relationships.
- File formats.
- Database schemas.
- Pipeline configurations.

### 13.3 Business metadata

Business metadata describes the meaning and organizational context of information.

Examples:

- Definition of an active vehicle.
- Meaning of a maintenance expense.
- Department responsible for a dataset.
- Official definition of operating hours.

### 13.4 Data lineage

Data lineage describes the origin, movement, and transformation history of data.

Example:

```text
Vehicle Inventory
       |
       v
Data Extraction
       |
       v
Vehicle Data Staging
       |
       v
Standardization
       |
       v
Analytical Warehouse
       |
       v
Maintenance Dashboard
```

Lineage helps answer:

- Where did this metric originate?
- Which source tables contributed to it?
- Which transformations were applied?
- Which reports depend on this dataset?
- What could be affected if a source changes?

Data lineage is important for debugging, auditing, compliance, and trust.

## 14. Data Quality

Data quality describes how suitable data is for its intended use.

A dataset may be technically accessible but still be unreliable.

For example:

- A vehicle has no inventory identifier.
- A maintenance date is invalid.
- A cost is stored as text.
- Two records represent the same equipment.
- A department is associated with the wrong vehicle.

Common data quality dimensions include:

| Dimension | Meaning |
|---|---|
| Accuracy | Values correctly represent reality. |
| Completeness | Required information is present. |
| Consistency | Values agree across systems. |
| Validity | Values follow defined rules. |
| Uniqueness | Duplicate records are controlled. |
| Timeliness | Data is available when needed. |

Data quality must be considered throughout the architecture, not only when generating reports.

See:

[[03 - Data Quality]]

## 15. Data Governance and Security

Data architecture also establishes how information is controlled.

### 15.1 Data governance

Data governance defines responsibilities, policies, standards, and decision-making processes for organizational data.

It addresses:

- Data ownership.
- Data stewardship.
- Data definitions.
- Quality standards.
- Access policies.
- Retention rules.
- Compliance.
- Data lifecycle responsibilities.

### 15.2 Data security

Data security protects information from unauthorized access, modification, disclosure, or destruction.

Important controls include:

- Authentication.
- Authorization.
- Encryption.
- Network security.
- Audit logging.
- Backup protection.
- Role-based access control.

### 15.3 Why governance matters

Imagine two departments calculate maintenance expenses differently.

One includes spare parts but excludes labor.

The other includes both.

Their reports may be technically correct according to their own definitions, but the organization cannot compare them consistently.

Data governance establishes shared definitions and rules.

A data architecture should make these rules enforceable where possible.

See:

- [[04 - Data Governance]]
- [[01 - Database Security Fundamentals]]
- [[05 - Privacy and Data Protection]]

## 16. Common Data Architecture Patterns

Different organizations use different architectural patterns according to their needs.

### 16.1 Centralized data architecture

Data from multiple sources is collected and managed through a central platform.

Advantages:

- Centralized control.
- Consistent standards.
- Easier consolidated reporting.
- Shared infrastructure.

Challenges:

- Potential central bottlenecks.
- Organizational dependencies.
- Platform complexity.
- Risk of becoming a central team bottleneck.

### 16.2 Distributed data architecture

Data remains distributed across multiple systems or domains.

Advantages:

- Local autonomy.
- Domain-specific optimization.
- Reduced dependence on one central system.

Challenges:

- Integration complexity.
- Inconsistent definitions.
- Distributed governance.
- Data discovery difficulties.

### 16.3 Data warehouse architecture

Focuses on integrating structured information into an analytical environment.

It is particularly useful for:

- Reporting.
- Historical analysis.
- Business intelligence.
- Structured analytical queries.

### 16.4 Data lake architecture

Focuses on storing diverse data types at scale.

It is useful for:

- Raw data preservation.
- Large-scale processing.
- Machine learning.
- Semi-structured and unstructured data.

### 16.5 Data lakehouse architecture

Combines aspects of data lakes and warehouses.

It attempts to support flexible storage and structured analytical management within a unified architecture.

### 16.6 Data mesh

Data mesh is an organizational and architectural approach that emphasizes:

- Domain-oriented data ownership.
- Data as a product.
- Self-service data infrastructure.
- Federated computational governance.

Instead of treating all organizational data as the responsibility of one central team, data mesh distributes ownership among business domains while maintaining shared standards.

Data mesh is not simply a database technology or storage product.

It is a broader approach to organizing data responsibilities.

## 17. Core Technologies in a Data Architecture

Different tools solve different problems.

| Technology | Main role |
|---|---|
| PostgreSQL | Relational database management |
| SQL | Querying and manipulating structured data |
| Python | Data processing and automation |
| Pandas | Tabular data manipulation |
| Apache Spark | Distributed data processing |
| Apache Kafka | Event streaming and message distribution |
| Apache Airflow | Workflow orchestration |
| dbt | SQL-based data transformation |
| Power BI | Business intelligence and visualization |
| Docker | Application and service containerization |
| Object Storage | Scalable file and data storage |

### 17.1 Understanding the tool categories

**Database systems**

Store and manage information.

**Programming languages and libraries**

Implement extraction, processing, transformation, and automation.

**Distributed processing frameworks**

Handle large-scale datasets across multiple computing resources.

**Message brokers and streaming platforms**

Transport events between producers and consumers.

**Orchestration tools**

Coordinate tasks and dependencies.

**BI tools**

Present data through reports and dashboards.

An effective data engineer understands the problem each category solves instead of treating every tool as interchangeable.

## 18. The Role of a Data Engineer

A Data Engineer designs, builds, and maintains systems that make data available and usable.

Typical responsibilities include:

### 18.1 Data ingestion

Develop processes to retrieve information from databases, APIs, files, and streaming sources.

### 18.2 Data transformation

Clean, standardize, validate, and transform datasets.

### 18.3 Data storage

Design and maintain storage structures suitable for different workloads.

### 18.4 Pipeline development

Build automated workflows that move data between systems.

### 18.5 Performance optimization

Improve processing speed, query performance, and infrastructure efficiency.

### 18.6 Reliability

Implement retries, logging, monitoring, error handling, and recovery mechanisms.

### 18.7 Collaboration

Work with:

- Data architects.
- Database administrators.
- Data analysts.
- Data scientists.
- Software engineers.
- Business stakeholders.

The role requires technical skills and the ability to understand the meaning and operational context of information.

## 19. Data Architecture Quality Attributes

A good data architecture must satisfy more than functional requirements.

It must also consider quality attributes.

| Attribute | Architectural concern |
|---|---|
| Scalability | Can the architecture support growing data volumes and workloads? |
| Performance | Can data be processed and accessed efficiently? |
| Reliability | Can the system operate consistently? |
| Availability | Can users access data when required? |
| Security | Is data protected against unauthorized access? |
| Maintainability | Can the architecture be modified without excessive complexity? |
| Interoperability | Can different systems exchange information? |
| Observability | Can operators understand system behavior? |
| Data quality | Is the information suitable for its intended purpose? |
| Governance | Are responsibilities and rules clearly defined? |
| Cost efficiency | Are infrastructure and operational costs justified? |

These attributes often compete.

For example:

- Lower latency may require more infrastructure.
- Stronger consistency may increase coordination costs.
- Longer retention increases storage requirements.
- More centralized control can reduce local autonomy.

Architectural decisions require tradeoffs.

## 20. A Complete Example: SIAPV Data Architecture

Consider SIAPV as the starting point for a broader institutional data platform.

The application currently manages approximately 200 vehicles and machinery units.

Its information includes:

- Unique inventory identifiers.
- Equipment descriptions.
- Operational status.
- Mileage.
- Hour-meter readings.
- Maintenance records.
- Insurance information.
- Administrative documentation.

Now imagine that the organization wants to combine SIAPV information with financial records and external data.

### 20.1 Sources

```text
SIAPV Database
Financial System
Maintenance Records
External APIs
Excel Files
```

Each source provides different information.

### 20.2 Ingestion

Python processes retrieve information from:

- PostgreSQL.
- External APIs.
- Excel files.
- CSV exports.

### 20.3 Integration

The system standardizes:

- Vehicle identifiers.
- Department names.
- Dates.
- Monetary values.
- Equipment categories.
- Maintenance statuses.

### 20.4 Transformation

The pipeline calculates:

- Maintenance costs per vehicle.
- Annual expenditure.
- Equipment utilization.
- Maintenance frequency.
- Operational availability indicators.

### 20.5 Storage

Integrated information is stored in an analytical database or warehouse.

Historical records can be preserved independently from operational transactions.

### 20.6 Consumption

Power BI presents:

- Maintenance dashboards.
- Financial reports.
- Equipment utilization.
- Historical trends.
- Department-level comparisons.

### 20.7 Conceptual architecture

```text
                 DATA SOURCES
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
     SIAPV        Financial       External
   PostgreSQL      System          Files
       |              |              |
       +--------------+--------------+
                      |
                      v
                DATA INGESTION
                   Python
                      |
                      v
                STAGING LAYER
                      |
                      v
              DATA INTEGRATION
                      |
                      v
             DATA TRANSFORMATION
                Python / SQL
                      |
                      v
             ANALYTICAL STORAGE
                Data Warehouse
                      |
                      v
                DATA CONSUMPTION
                      |
              +-------+-------+
              |               |
              v               v
           Power BI        Analytics
```

### 20.8 Architectural considerations

A real implementation would need to define:

- Which system is authoritative for each data element.
- How often information is synchronized.
- How duplicate records are identified.
- How historical changes are preserved.
- How data quality is validated.
- Who can access financial information.
- How pipeline failures are handled.
- How analytical results are reconciled with source systems.

This is the difference between simply combining files and designing a reliable data architecture.

## 21. Common Data Architecture Mistakes

### Mistake 1: Treating every database as an analytical platform

Operational databases and analytical systems have different workload requirements.

Separating them may improve performance and reduce operational risk.

### Mistake 2: Building pipelines without understanding source systems

Incorrect assumptions about identifiers, formats, and business rules can produce unreliable results.

### Mistake 3: Ignoring data quality

A pipeline can execute successfully while producing incorrect information.

Technical success does not guarantee business correctness.

### Mistake 4: Transforming data without documenting the rules

Undocumented transformations make results difficult to validate and maintain.

### Mistake 5: Ignoring metadata and lineage

Without context, users may not understand where information originated or how it was modified.

### Mistake 6: Choosing tools before defining requirements

A technology should solve an identified architectural problem.

Tool selection should follow requirements analysis.

### Mistake 7: Assuming real-time processing is always better

Streaming systems introduce complexity and operational overhead.

Batch processing is often sufficient.

### Mistake 8: Ignoring governance and security

Data platforms can centralize sensitive information.

Access control, ownership, privacy, and retention must be considered from the beginning.

## 22. Knowledge Check

Answer these questions without consulting the explanations above.

1. What is data architecture?
2. What is the difference between data architecture and data engineering?
3. What is a data ecosystem?
4. What are the main stages of a data architecture?
5. What is a data source?
6. What is data ingestion?
7. How does ingestion differ from integration?
8. What is schema mapping?
9. What is the purpose of a staging area?
10. What is the difference between operational and analytical storage?
11. What is a data warehouse?
12. What is a data lake?
13. What is a lakehouse?
14. What is batch processing?
15. What is stream processing?
16. What is a data pipeline?
17. What is orchestration?
18. What is metadata?
19. What is data lineage?
20. Why are data quality and governance important?
21. What is the main idea behind data mesh?
22. What responsibilities does a Data Engineer typically have?
23. Why should technology selection follow requirements analysis?
24. How could SIAPV become part of a broader data architecture?

## 23. Practical Exercises

### Exercise 1: Identify data sources

Choose an organization and identify at least five possible data sources.

For each source, document:

- Source name.
- Data format.
- Information generated.
- Access method.
- Update frequency.
- Potential integration challenges.

### Exercise 2: Design a data flow

Design a conceptual architecture that collects information from:

- A PostgreSQL database.
- An external REST API.
- An Excel workbook.

Define:

1. Ingestion method.
2. Staging location.
3. Integration rules.
4. Transformation process.
5. Target storage.
6. Consumption method.

### Exercise 3: Batch or streaming?

Choose the appropriate processing model for each scenario.

1. Daily financial reporting.
2. Vehicle location tracking.
3. Monthly inventory analysis.
4. Real-time fraud detection.
5. Historical maintenance analysis.
6. Continuous application monitoring.

Justify every decision.

### Exercise 4: Identify architectural risks

Consider an organization that collects information from five departments.

Each department uses different Excel templates and different vehicle identifiers.

Identify:

- Data quality problems.
- Integration challenges.
- Metadata requirements.
- Governance responsibilities.
- Potential standardization rules.

### Exercise 5: Design SIAPV's analytical architecture

Develop a conceptual design that integrates SIAPV with financial and maintenance information.

Your design must identify:

- Data sources.
- Ingestion mechanisms.
- Staging.
- Integration.
- Transformation.
- Analytical storage.
- Data serving.
- Governance.
- Security.
- Monitoring.

Explain why each component is necessary.

## 24. Key Vocabulary

| English term | Spanish equivalent | Meaning |
|---|---|---|
| Data Architecture | Arquitectura de datos | Overall design of an organization's data environment. |
| Data Ecosystem | Ecosistema de datos | Systems and processes that generate and consume data. |
| Data Engineering | Ingeniería de datos | Building and operating systems for data movement and processing. |
| Data Management | Gestión de datos | Policies and practices for managing information. |
| Data Source | Fuente de datos | Origin of information. |
| Data Ingestion | Ingesta de datos | Collection and transfer of data into a system. |
| Data Integration | Integración de datos | Combining information from different sources. |
| Data Transformation | Transformación de datos | Modifying data into a required structure or representation. |
| Staging Area | Área de preparación | Intermediate storage for incoming data. |
| Data Warehouse | Almacén de datos | System designed for analytical workloads. |
| Data Lake | Lago de datos | Storage environment for diverse data formats. |
| Lakehouse | Arquitectura lakehouse | Architecture combining lake and warehouse capabilities. |
| Data Pipeline | Canalización de datos | Sequence of processes that moves and transforms data. |
| Batch Processing | Procesamiento por lotes | Processing collected groups of records. |
| Stream Processing | Procesamiento en flujo | Continuous processing of incoming events. |
| Data Serving | Distribución de datos | Making prepared data available to consumers. |
| Data Consumption | Consumo de datos | Use of data by applications, analysts, and systems. |
| Metadata | Metadatos | Information describing other data. |
| Data Lineage | Linaje de datos | History of data origin and transformations. |
| Data Governance | Gobierno de datos | Rules and responsibilities for managing data. |
| Data Quality | Calidad de datos | Suitability of information for its intended use. |
| Data Orchestration | Orquestación de datos | Coordination of data-processing tasks. |
| Data Mesh | Malla de datos | Domain-oriented approach to data ownership and infrastructure. |
| Interoperability | Interoperabilidad | Ability of systems to exchange and use information. |

## 25. Connections to Other Notes

### Previous knowledge

- [[02 - The Data Lifecycle]]
- [[03 - Types of Data]]
- [[04 - OLTP vs OLAP]]
- [[05 - Database Architecture Fundamentals]]
- [[02 - Database Scalability]]
- [[07 - Backup and Disaster Recovery]]

### Next documents

- [[02 - Data Integration]]
- [[03 - ETL and ELT]]
- [[04 - Data Pipelines]]
- [[05 - Data Warehouses]]
- [[06 - Data Lakes]]
- [[07 - Lakehouse Architecture]]

### Supporting knowledge

- [[03 - Data Quality]]
- [[04 - Data Governance]]
- [[05 - Privacy and Data Protection]]

### Conceptual relationship

```text
                 DATA ARCHITECTURE
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       Sources       Integration     Governance
          |              |              |
          v              v              |
       Ingestion ---> Processing        |
                         |              |
                         v              |
                      Storage           |
                         |              |
                         v              |
                       Serving          |
                         |              |
                         v              |
                    Consumption         |
                         |              |
                         +--------------+
```

Data architecture establishes the structure through which information becomes a reliable organizational resource.

Data engineering builds and operates the mechanisms that make this architecture work.

Data management ensures that information remains understandable, trustworthy, secure, and properly controlled.
