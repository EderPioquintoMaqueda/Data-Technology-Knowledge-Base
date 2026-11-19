---

title: "Lakehouse Architecture"  
aliases:

- "Data Lakehouse"
    
- "Lakehouse Fundamentals"  
    type: learning-note  
    domain: technology  
    category: data-engineering-architecture  
    subject: lakehouse-architecture  
    level: intermediate  
    difficulty: intermediate  
    status: in-progress  
    created: 2026-11-12  
    updated: 2026-11-12  
    tags:
    
- lakehouse
    
- data-engineering
    
- data-architecture
    
- data-lake
    
- data-warehouse
    
- analytics
    
- big-data
    
- business-intelligence  
    concepts:
    
- lakehouse
    
- data-lake
    
- data-warehouse
    
- open-table-format
    
- transactional-data-lake
    
- acid-transactions
    
- schema-enforcement
    
- time-travel
    
- medallion-architecture
    
- bronze-silver-gold
    
- data-governance
    
- unified-analytics  
    prerequisites:
    
- "[[01 - Data Architecture Overview]]"
    
- "[[03 - ETL and ELT]]"
    
- "[[04 - Data Pipelines]]"
    
- "[[05 - Data Warehouses]]"
    
- "[[06 - Data Lakes]]"  
    related:
    
- "[[08 - Cloud Data Architecture]]"
    
- "[[Data Quality]]"
    
- "[[Data Governance]]"
    
- "[[09 - Distributed Data Processing]]"  
    tools:
    
- Python
    
- SQL
    
- Apache Spark
    
- Delta Lake
    
- Apache Iceberg
    
- Apache Hudi
    
- PostgreSQL
    
- Power BI  
    learning_objectives:
    
- "Define a Lakehouse architecture."
    
- "Explain why Lakehouse architectures emerged."
    
- "Compare Data Lakes, Data Warehouses, and Lakehouses."
    
- "Understand open table formats."
    
- "Understand ACID transactions in analytical storage."
    
- "Explain schema enforcement and schema evolution."
    
- "Understand time travel and versioned data."
    
- "Understand the role of the Medallion Architecture."
    
- "Identify Lakehouse use cases."
    
- "Design a conceptual Lakehouse architecture."  
    review:  
    frequency: weekly  
    last_reviewed: null  
    next_review: null
    

---

# Lakehouse Architecture

## 1. Introduction

A **Lakehouse** is a data architecture that combines characteristics of a **Data Lake** and a **Data Warehouse**.

The central objective is to create a unified data platform that can support:

- Data engineering.
    
- Business Intelligence.
    
- Analytics.
    
- Machine learning.
    
- Historical analysis.
    

A simplified idea is:

```
DATA LAKE
Flexibility
Multiple Data Types
Low-Cost Storage

        +

DATA WAREHOUSE
Structure
Reliability
Analytical Performance

        =

LAKEHOUSE
```

A Lakehouse attempts to reduce the separation between systems used for storing raw data and systems used for analytical consumption.

---

# 2. Why Did Lakehouse Architecture Emerge?

Traditional data architectures often separate data into multiple platforms.

For example:

```
Operational Systems
        |
        v
    Data Lake
        |
        | Data Engineering
        v
 Data Warehouse
        |
        | BI
        v
 Dashboards
```

This architecture can work well, but it may introduce:

- Multiple copies of data.
    
- Complex pipelines.
    
- Data synchronization challenges.
    
- Higher maintenance requirements.
    
- Delays between raw data and analytical availability.
    

A Lakehouse attempts to create a more unified architecture.

```
Operational Systems
        |
        v
     LAKEHOUSE
        |
   +----+----+
   |    |    |
   v    v    v
  BI   ML  Analytics
```

The same platform can support multiple types of workloads.

---

# 3. The Core Idea

A Lakehouse combines two important concepts.

## Data Lake capabilities

- Large-scale storage.
    
- Multiple data formats.
    
- Structured data.
    
- Semi-structured data.
    
- Unstructured data.
    

## Data Warehouse capabilities

- Reliable tables.
    
- Structured analytical access.
    
- Data governance.
    
- Transactional consistency.
    
- Optimized analytical workloads.
    

The Lakehouse architecture aims to provide both.

---

# 4. Data Lake vs. Data Warehouse vs. Lakehouse

|Characteristic|Data Lake|Data Warehouse|Lakehouse|
|---|---|---|---|
|Primary storage|Raw files|Curated tables|Open data storage and managed tables|
|Data types|Multiple|Primarily structured|Multiple|
|Schema|Often schema-on-read|Schema-on-write|Flexible with controls|
|Analytics|Requires additional processing|Strong|Strong|
|Machine Learning|Strong|More limited|Strong|
|Data governance|Can be challenging|Strong|Integrated|
|Historical versions|Depends on implementation|Depends on implementation|Common capability|
|Main goal|Flexible storage|Trusted analytics|Unified data platform|

The Lakehouse is not simply a Data Lake with a new name.

The key difference is the addition of stronger data management capabilities.

---

# 5. The Storage Foundation

A Lakehouse commonly uses scalable storage similar to a Data Lake.

Conceptually:

```
+----------------------------------+
|          LAKEHOUSE STORAGE       |
|                                  |
|   Structured Data                |
|   Semi-structured Data           |
|   Unstructured Data              |
|                                  |
+----------------------------------+
```

Data may be stored using open and efficient file formats.

Examples include:

- Parquet.
    
- ORC.
    
- Avro.
    

These formats can support large-scale processing.

---

# 6. Open Table Formats

One of the most important concepts in modern Lakehouse architectures is the **table format**.

Raw files alone do not provide all the behavior expected from database tables.

For example, a collection of files may not automatically provide:

- Transaction control.
    
- Version history.
    
- Schema enforcement.
    
- Reliable concurrent updates.
    

Open table formats add management capabilities on top of data files.

Examples include:

- Delta Lake.
    
- Apache Iceberg.
    
- Apache Hudi.
    

Conceptually:

```
+-----------------------------+
|       Table Management      |
|                             |
| Transactions                |
| Metadata                    |
| Schema Rules                |
| Version History             |
+-------------+---------------+
              |
              v
+-----------------------------+
|         Data Files          |
|                             |
| Parquet Files               |
+-----------------------------+
```

The table format acts as a management layer for the underlying data.

---

# 7. ACID Transactions

A major feature associated with modern Lakehouse tables is support for **ACID transactions**.

ACID represents:

- Atomicity.
    
- Consistency.
    
- Isolation.
    
- Durability.
    

## 7.1 Atomicity

A transaction is completed entirely or not completed at all.

```
Update begins
     |
     +--> Success → Commit
     |
     +--> Failure → Rollback
```

Partial changes should not leave the table in an inconsistent state.

---

## 7.2 Consistency

Data should move from one valid state to another valid state.

Example:

A pipeline should not produce records that violate defined constraints or schema rules.

---

## 7.3 Isolation

Multiple processes should not interfere with each other in a way that produces incorrect results.

Example:

```
Process A ──┐
            ├──> Managed Table
Process B ──┘
```

The system manages concurrent operations.

---

## 7.4 Durability

Once a transaction is successfully committed, the change should persist.

---

# 8. Schema Enforcement

A Lakehouse can apply rules to incoming data.

Suppose a table expects:

```
vehicle_id    INTEGER
mileage       INTEGER
event_date    DATE
```

A new record arrives with:

```
vehicle_id = "ABC"
mileage = "unknown"
event_date = "tomorrow"
```

Schema enforcement can detect incompatible data before it affects trusted analytical tables.

This helps improve data quality.

---

# 9. Schema Evolution

Data structures can change over time.

For example:

```
Version 1:

vehicle_id
mileage
event_date
```

Later:

```
Version 2:

vehicle_id
mileage
event_date
fuel_consumption
```

**Schema evolution** provides controlled mechanisms for adapting to these changes.

The objective is to balance:

- Flexibility.
    
- Compatibility.
    
- Data quality.
    

Schema evolution should be intentional rather than uncontrolled.

---

# 10. Time Travel

Some Lakehouse technologies support **time travel**.

Time travel allows users to access previous versions of a dataset.

Conceptually:

```
Version 1 ─── Version 2 ─── Version 3
    |              |              |
 January         February        March
```

A user may need to answer:

> What did this table contain before the latest update?

Time travel can support:

- Auditing.
    
- Debugging.
    
- Reproducibility.
    
- Historical analysis.
    
- Recovery from incorrect updates.
    

---

# 11. The Medallion Architecture in a Lakehouse

The **Medallion Architecture** is commonly used to organize data processing in a Lakehouse.

The three main layers are:

```
BRONZE
   |
   v
SILVER
   |
   v
GOLD
```

Each layer represents a different level of refinement.

---

## 11.1 Bronze: Raw Data

The Bronze layer stores raw or minimally transformed information.

Examples:

- API responses.
    
- Database extracts.
    
- Log files.
    
- Sensor events.
    

The main objective is to preserve source information.

```
Sources
   |
   v
BRONZE
Raw Data
```

---

## 11.2 Silver: Clean and Standardized Data

The Silver layer improves data quality.

Typical processes include:

- Removing duplicates.
    
- Standardizing names.
    
- Validating data types.
    
- Resolving inconsistent values.
    
- Combining related datasets.
    

```
BRONZE
   |
   v
Validation
Cleaning
Standardization
   |
   v
SILVER
```

The Silver layer often becomes an important reusable foundation for multiple analytical workloads.

---

## 11.3 Gold: Business-Ready Data

The Gold layer contains curated information designed for consumption.

Examples:

- KPI tables.
    
- Analytical models.
    
- Aggregated metrics.
    
- Reporting datasets.
    

```
SILVER
   |
   v
Business Rules
Aggregations
Metrics
   |
   v
GOLD
```

Business Intelligence tools can consume these datasets.

---

# 12. Example: Fleet Lakehouse

A conceptual Lakehouse could integrate multiple fleet data sources.

```
Database ───┐
API ────────┤
CSV Files ──┤
Telemetry ──┤
            v
       BRONZE
            |
            v
       SILVER
            |
            v
        GOLD
            |
     +------+------+ 
     |      |      |
     v      v      v
    BI     SQL     ML
```

---

## Bronze Layer

Possible raw datasets:

```
bronze/
├── vehicles/
├── maintenance/
├── mileage/
├── incidents/
└── telemetry/
```

The focus is data preservation.

---

## Silver Layer

Possible cleaned datasets:

```
silver/
├── vehicles_clean/
├── maintenance_clean/
├── mileage_clean/
├── incidents_clean/
└── telemetry_clean/
```

The focus is standardization and reliability.

---

## Gold Layer

Possible analytical datasets:

```
gold/
├── fleet_performance/
├── maintenance_kpis/
├── equipment_availability/
└── incident_analysis/
```

The focus is business consumption.

---

# 13. Lakehouse Data Flow

A simplified data flow can look like this:

```
+---------------------+
|    Source Systems   |
+----------+----------+
           |
           v
+---------------------+
|       BRONZE        |
| Raw Data            |
+----------+----------+
           |
           v
+---------------------+
|       SILVER        |
| Clean Data          |
+----------+----------+
           |
           v
+---------------------+
|        GOLD         |
| Business Data       |
+----------+----------+
           |
     +-----+-----+
     |           |
     v           v
Analytics      Machine
& BI          Learning
```

The important principle is **progressive data refinement**.

---

# 14. Batch and Streaming Workloads

A Lakehouse can support different ingestion patterns.

## Batch

```
Daily Extraction
       |
       v
    BRONZE
       |
       v
Transformation
```

Useful for:

- Daily reports.
    
- Historical datasets.
    
- Scheduled processes.
    

---

## Streaming

```
Continuous Events
       |
       v
    BRONZE
       |
       v
Near-Real-Time Processing
```

Useful for:

- Telemetry.
    
- Sensors.
    
- Application events.
    
- Operational monitoring.
    

A modern architecture can combine both approaches.

---

# 15. Data Quality in a Lakehouse

Data quality should improve as information moves through the architecture.

```
BRONZE
Original Data
     |
     v
SILVER
Validated Data
     |
     v
GOLD
Trusted Business Data
```

Possible quality checks include:

- Required fields.
    
- Valid data types.
    
- Duplicate detection.
    
- Range validation.
    
- Referential consistency.
    
- Business rules.
    

---

# 16. Governance

A unified data platform still requires governance.

Important areas include:

## Data Ownership

Who is responsible for each dataset?

## Access Control

Who can read or modify data?

## Metadata

What does the dataset contain?

## Lineage

Where did the data originate?

## Classification

How should the data be handled?

## Retention

How long should the information be preserved?

A Lakehouse does not eliminate governance requirements. It provides an architecture where governance can be integrated across analytical workloads.

---

# 17. Data Lakehouse Users

Different teams can interact with the same platform.

|User|Typical Activity|
|---|---|
|Data Engineer|Builds pipelines|
|Data Analyst|Queries and analyzes data|
|BI Developer|Creates reports and dashboards|
|Data Scientist|Builds analytical and ML models|
|Business User|Consumes reports and metrics|

This shared architecture can reduce unnecessary data duplication.

---

# 18. Advantages of a Lakehouse

## Unified Platform

Different analytical workloads can use the same data foundation.

## Scalable Storage

Large volumes of data can be stored efficiently.

## Multiple Data Types

Structured and semi-structured data can coexist.

## Historical Versioning

Previous data versions can support auditing and reproducibility.

## Data Quality Controls

Schema and transactional features can protect trusted datasets.

## Advanced Analytics

The architecture can support both BI and machine learning.

---

# 19. Challenges

A Lakehouse also introduces challenges.

## Architecture Complexity

The platform can involve:

- Storage systems.
    
- Table formats.
    
- Processing engines.
    
- Metadata systems.
    
- Governance tools.
    

## Skills Requirements

Teams may need knowledge of:

- SQL.
    
- Python.
    
- Distributed processing.
    
- Data engineering.
    
- Cloud architecture.
    

## Governance

Flexible storage requires disciplined management.

## Cost Management

Large-scale processing can become expensive if poorly designed.

---

# 20. When Should You Consider a Lakehouse?

A Lakehouse can be useful when an organization needs:

- Large-scale data storage.
    
- Multiple data formats.
    
- Business Intelligence.
    
- Machine learning.
    
- Shared analytical infrastructure.
    
- Historical data management.
    

For a smaller application, a traditional relational database may be sufficient.

Architecture should match the actual requirements.

> More technology does not automatically mean better architecture.

---

# 21. Comparison Example

Imagine a fleet management organization.

## Operational Database

Used for:

- Registering vehicles.
    
- Updating current mileage.
    
- Recording maintenance events.
    

Primary concern:

> Operational consistency.

---

## Data Warehouse

Used for:

- Historical reports.
    
- Maintenance KPIs.
    
- Cost analysis.
    

Primary concern:

> Reliable business analytics.

---

## Data Lake

Used for:

- Raw telemetry.
    
- API responses.
    
- PDF reports.
    
- Sensor data.
    

Primary concern:

> Flexible data storage.

---

## Lakehouse

Used for:

- Raw data storage.
    
- Data engineering.
    
- Curated analytical tables.
    
- BI datasets.
    
- Machine learning.
    

Primary concern:

> A unified analytical data platform.

---

# 22. Lakehouse Design Process

A practical design process can follow these steps.

## Step 1: Identify Data Sources

Examples:

- Databases.
    
- APIs.
    
- Files.
    
- Event streams.
    

## Step 2: Define the Bronze Layer

Determine what raw information will be preserved.

## Step 3: Define the Silver Layer

Define:

- Cleaning rules.
    
- Standardization.
    
- Validation.
    
- Integration.
    

## Step 4: Define the Gold Layer

Determine which datasets support:

- Reports.
    
- KPIs.
    
- Analytics.
    
- Machine learning.
    

## Step 5: Define Governance

Determine:

- Ownership.
    
- Access.
    
- Metadata.
    
- Lineage.
    
- Retention.
    

## Step 6: Define Quality Controls

Determine validation rules for each layer.

---

# 23. Practical Exercises

## Exercise 1: Compare the Architectures

Explain the main difference between:

- Data Lake.
    
- Data Warehouse.
    
- Lakehouse.
    

Focus on:

- Storage.
    
- Data structure.
    
- Processing.
    
- Analytical use.
    

---

## Exercise 2: Design a Fleet Lakehouse

Create a conceptual architecture for fleet data.

Include:

### Bronze

- Raw vehicle records.
    
- Maintenance files.
    
- Telemetry events.
    
- Incident reports.
    

### Silver

- Cleaned vehicle records.
    
- Standardized maintenance data.
    
- Validated telemetry.
    

### Gold

- Maintenance KPIs.
    
- Fleet availability metrics.
    
- Cost analysis.
    

Draw the complete flow.

---

## Exercise 3: Schema Evolution

A telemetry dataset receives a new field:

```
engine_temperature
```

Explain how you would introduce this field while maintaining compatibility with existing data.

---

## Exercise 4: Time Travel

A transformation process accidentally removes valid records.

Explain how historical versions could help:

1. Identify the problem.
    
2. Compare versions.
    
3. Recover the correct data.
    

---

## Exercise 5: Choose an Architecture

For each situation, choose the most appropriate primary architecture:

1. Small transactional application.
    
2. Corporate financial reporting.
    
3. Large collection of raw sensor data.
    
4. Organization requiring BI and machine learning from a shared platform.
    

Explain your reasoning.

---

# 24. Knowledge Check

1. What is a Lakehouse?
    
2. Why did Lakehouse architectures emerge?
    
3. How does a Lakehouse differ from a traditional Data Lake?
    
4. How does it differ from a Data Warehouse?
    
5. What is an open table format?
    
6. Why are table formats important?
    
7. What does ACID mean?
    
8. What is schema enforcement?
    
9. What is schema evolution?
    
10. What is time travel?
    
11. What is the purpose of the Bronze layer?
    
12. What happens in the Silver layer?
    
13. What is the Gold layer designed for?
    
14. How does data quality evolve across the layers?
    
15. Can a Lakehouse support both BI and machine learning?
    
16. What governance capabilities are required?
    
17. What are the main advantages of a Lakehouse?
    
18. What are its main challenges?
    
19. When might a Lakehouse be unnecessary?
    
20. How would a Lakehouse support a fleet management organization?
    

---

# 25. Glossary

|Term|Definition|
|---|---|
|Lakehouse|Unified architecture combining Data Lake and Data Warehouse capabilities|
|Open Table Format|Technology that manages data files as reliable tables|
|ACID|Atomicity, Consistency, Isolation, Durability|
|Schema Enforcement|Validation of data against expected structures|
|Schema Evolution|Controlled modification of data structures|
|Time Travel|Accessing previous versions of data|
|Bronze|Raw or minimally transformed data layer|
|Silver|Cleaned and standardized data layer|
|Gold|Curated, business-ready data layer|
|Medallion Architecture|Bronze, Silver, and Gold data architecture|
|Data Lineage|Record of data origin and transformations|
|Batch Processing|Processing data at scheduled intervals|
|Streaming|Continuous or near-real-time processing|
|Transaction|Controlled operation that changes data|

---

# 26. Connections to Other Topics

## Previous notes

- [[01 - Data Architecture Overview]]
    
- [[03 - ETL and ELT]]
    
- [[04 - Data Pipelines]]
    
- [[05 - Data Warehouses]]
    
- [[06 - Data Lakes]]
    

## Related concepts

- [[Data Modeling]]
    
- [[Data Quality]]
    
- [[Data Governance]]
    
- [[08 - Cloud Data Architecture]]
    
- [[09 - Distributed Data Processing]]
    

## Practical applications

- [[SIAPV Data Architecture]]
    
- [[Fleet Data Analytics]]
    
- [[Business Intelligence Fundamentals]]
    
- [[Machine Learning Systems]]
    

---

# 27. Key Takeaways

1. A Lakehouse combines Data Lake flexibility with Data Warehouse reliability.
    
2. It can support multiple analytical workloads from a shared data foundation.
    
3. Open table formats add management capabilities to large-scale data files.
    
4. ACID transactions improve reliability.
    
5. Schema enforcement helps protect data quality.
    
6. Schema evolution allows controlled changes over time.
    
7. Time travel can support auditing and recovery.
    
8. The Medallion Architecture organizes data into Bronze, Silver, and Gold layers.
    
9. Bronze preserves raw data.
    
10. Silver provides cleaned and standardized data.
    
11. Gold provides trusted, business-ready datasets.
    
12. Lakehouses can support BI, analytics, and machine learning.
    
13. Governance remains essential.
    
14. A Lakehouse can reduce unnecessary data duplication across analytical platforms.
    
15. Architecture decisions should always be based on actual requirements.
    

---

> [!summary]  
> A **Lakehouse** is a unified data architecture that combines the scalability and flexibility of a **Data Lake** with the reliability and analytical capabilities traditionally associated with a **Data Warehouse**. Its strength comes from combining **open storage, managed tables, data quality controls, historical versioning, and progressive data refinement**.