
---
title: "ETL and ELT"
aliases:
  - "ETL Fundamentals"
  - "ELT Architecture"
  - "Extract Transform Load"
  - "Extract Load Transform"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: etl-elt
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - etl
  - elt
  - data-engineering
  - data-integration
  - data-transformation
  - data-ingestion
  - data-warehouse
  - data-lake
  - data-pipeline
  - batch-processing
  - data-quality
concepts:
  - extract
  - transform
  - load
  - staging-area
  - data-cleaning
  - data-validation
  - data-transformation
  - data-orchestration
  - incremental-loading
  - full-loading
  - change-data-capture
  - data-warehouse
  - data-lake
  - data-lakehouse
  - medallion-architecture
  - idempotency
  - data-lineage
  - schema-on-write
  - schema-on-read
prerequisites:
  - "[[01 - Data Architecture Overview]]"
  - "[[02 - Data Integration]]"
  - "[[04 - OLTP vs OLAP]]"
  - "[[02 - The Data Lifecycle]]"
related:
  - "[[04 - Data Pipelines]]"
  - "[[05 - Data Warehouses]]"
  - "[[06 - Data Lakes]]"
  - "[[07 - Lakehouse Architecture]]"
  - "[[03 - Data Quality]]"
  - "[[04 - Data Governance]]"
tools:
  - SQL
  - PostgreSQL
  - Python
  - Pandas
  - Apache Spark
  - dbt
  - Apache Airflow
  - Docker
learning_objectives:
  - "Explain the Extract, Transform, and Load stages."
  - "Distinguish ETL from ELT architectures."
  - "Understand staging areas and intermediate storage."
  - "Compare full and incremental loading."
  - "Understand batch and streaming transformations."
  - "Design data validation and error-handling procedures."
  - "Explain schema-on-write and schema-on-read."
  - "Identify the role of orchestration tools."
  - "Design a conceptual ETL and ELT workflow using SIAPV data."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# ETL and ELT

## 1. Introduction

Organizations collect information from many different sources, but raw data is not always ready for analysis.

It may contain:

- Missing values.
- Duplicate records.
- Inconsistent formats.
- Incorrect data types.
- Different measurement units.
- Invalid identifiers.
- Historical information that requires consolidation.
- Records that do not follow a common structure.

Before information can be used reliably, it often needs to be extracted, prepared, and stored in an appropriate destination.

Two of the most important approaches for performing these operations are:

- **ETL:** Extract, Transform, Load.
- **ELT:** Extract, Load, Transform.

Both approaches move data from source systems into target environments. Their main difference is the order in which transformation and loading occur.

---

## 2. What Is ETL?

ETL stands for:

- **Extract**
- **Transform**
- **Load**

It is a data integration process in which information is extracted from source systems, transformed before reaching its final destination, and then loaded into that destination.

The traditional ETL architecture performs most data preparation outside the final analytical repository.

### 2.1 ETL workflow

```text
+----------------------+
|    Source Systems    |
|                      |
| Databases            |
| APIs                 |
| CSV / Excel          |
| Business Applications|
+----------+-----------+
           |
           v
+----------------------+
|       EXTRACT        |
|                      |
| Retrieve source data |
+----------+-----------+
           |
           v
+----------------------+
|      TRANSFORM       |
|                      |
| Clean                 |
| Validate              |
| Standardize           |
| Join                  |
| Calculate             |
+----------+-----------+
           |
           v
+----------------------+
|         LOAD         |
|                      |
| Store prepared data  |
+----------+-----------+
           |
           v
+----------------------+
|    Data Warehouse    |
|                      |
| Analytical datasets  |
| Reporting tables     |
+----------------------+
```

### 2.2 Extract

Extraction retrieves data from one or more sources.

Possible sources include:

- Relational databases.
- REST APIs.
- CSV files.
- Excel workbooks.
- Enterprise applications.
- Legacy systems.
- Cloud storage.

Example:

```sql
SELECT
    inventory_number,
    manufacturer,
    equipment_type,
    status,
    updated_at
FROM vehicles;
```

This query extracts selected vehicle information from a source database.

Extraction should consider:

- Source availability.
- Permissions.
- Query performance.
- Data volume.
- Extraction frequency.
- Consistency of the extracted dataset.

### 2.3 Transform

Transformation prepares the extracted data for its intended use.

Common operations include:

**Cleaning**

- Removing unnecessary whitespace.
- Handling missing values.
- Detecting invalid records.
- Identifying duplicates.

**Standardization**

- Standardizing dates.
- Converting units.
- Normalizing categorical values.
- Applying consistent naming conventions.

**Integration**

- Joining datasets.
- Matching identifiers.
- Combining information from multiple sources.

**Calculation**

- Deriving metrics.
- Calculating totals.
- Computing averages.
- Creating analytical indicators.

**Validation**

- Checking required fields.
- Verifying data types.
- Enforcing business rules.
- Detecting invalid relationships.

Example using Python and Pandas:

```python
import pandas as pd

# Load the extracted dataset.
vehicles = pd.read_csv("vehicles.csv")

# Remove leading and trailing whitespace.
vehicles["manufacturer"] = (
    vehicles["manufacturer"].str.strip()
)

# Standardize status values.
status_mapping = {
    "ACTIVE": "In Service",
    "INACTIVE": "Out of Service",
    "REPAIR": "Under Maintenance"
}

vehicles["status"] = (
    vehicles["status"].replace(status_mapping)
)

# Convert the update field into a datetime value.
vehicles["updated_at"] = pd.to_datetime(
    vehicles["updated_at"],
    errors="coerce"
)

# Remove records without an inventory identifier.
vehicles = vehicles.dropna(
    subset=["inventory_number"]
)

# Export the transformed dataset.
vehicles.to_csv(
    "vehicles_clean.csv",
    index=False
)
```

The transformation stage creates a dataset that follows the expected analytical structure.

### 2.4 Load

Loading writes the transformed information into its destination.

Common destinations include:

- Data warehouses.
- Analytical databases.
- Reporting repositories.
- Data marts.
- Other structured storage systems.

Example:

```sql
INSERT INTO warehouse_vehicles (
    inventory_number,
    manufacturer,
    equipment_type,
    status,
    updated_at
)
VALUES (
    '13990',
    'Shantui',
    'Motor Grader',
    'In Service',
    '2026-11-12 14:30:00'
);
```

In a real pipeline, records are generally loaded in batches or through a controlled bulk-loading process rather than one manually written statement per record.

---

## 3. What Is ELT?

ELT stands for:

- **Extract**
- **Load**
- **Transform**

In ELT, extracted data is loaded into the target environment before the main transformations occur.

The target platform performs the transformation using its own processing capabilities.

### 3.1 ELT workflow

```text
+----------------------+
|    Source Systems    |
|                      |
| Databases            |
| APIs                 |
| CSV / Excel          |
| Business Applications|
+----------+-----------+
           |
           v
+----------------------+
|       EXTRACT        |
|                      |
| Retrieve source data |
+----------+-----------+
           |
           v
+----------------------+
|         LOAD         |
|                      |
| Store raw data       |
+----------+-----------+
           |
           v
+----------------------+
|    Raw Data Layer    |
|                      |
| Original records     |
+----------+-----------+
           |
           v
+----------------------+
|      TRANSFORM       |
|                      |
| SQL / Spark / dbt    |
| Cleaning             |
| Joining              |
| Aggregation          |
+----------+-----------+
           |
           v
+----------------------+
| Analytical Layer     |
|                      |
| Prepared datasets    |
+----------------------+
```

### 3.2 Extract

The extraction stage retrieves data from source systems.

The extraction process may preserve the original structure and values to avoid losing information before loading.

### 3.3 Load

The extracted information is loaded into the target platform.

The raw data may be stored in:

- Cloud object storage.
- A data lake.
- A data warehouse.
- A lakehouse platform.
- A raw database schema.

The objective is to make source data available for subsequent transformations.

### 3.4 Transform

Transformations occur inside the target environment.

For example, SQL can be used to standardize and aggregate records.

```sql
CREATE TABLE analytics_vehicles AS
SELECT
    inventory_number,
    TRIM(manufacturer) AS manufacturer,
    equipment_type,
    CASE
        WHEN status = 'ACTIVE'
            THEN 'In Service'
        WHEN status = 'INACTIVE'
            THEN 'Out of Service'
        WHEN status = 'REPAIR'
            THEN 'Under Maintenance'
        ELSE 'Unknown'
    END AS operational_status
FROM raw_vehicles;
```

The target platform executes the transformation using its available processing resources.

---

## 4. ETL vs. ELT

The two approaches share the same general objective but organize processing differently.

| Characteristic | ETL | ELT |
|---|---|---|
| Sequence | Extract, Transform, Load | Extract, Load, Transform |
| Transformation location | Before final loading | Inside target environment |
| Raw data storage | Optional or intermediate | Commonly retained |
| Processing | External engine or integration platform | Target platform |
| Typical destination | Traditional data warehouse | Modern warehouse or lakehouse |
| Flexibility | Depends on transformed outputs | Often greater access to raw data |
| Infrastructure | Requires transformation infrastructure | Uses target processing resources |
| Common tools | SSIS, Informatica, Talend | dbt, SQL, Spark, warehouse-native tools |

### 4.1 The most important distinction

ETL transforms data before loading it into the final destination.

ELT loads data first and transforms it afterward.

This difference affects:

- Processing architecture.
- Storage requirements.
- Infrastructure costs.
- Data retention.
- Governance.
- Transformation flexibility.
- Performance.

### 4.2 When should ETL be used?

ETL may be appropriate when:

- Data must be cleaned before entering the destination.
- The target system accepts only prepared records.
- Transformation must happen in a separate processing environment.
- Sensitive information must be filtered before loading.
- The target platform has limited transformation capabilities.
- Strict pre-load validation is required.

### 4.3 When should ELT be used?

ELT may be appropriate when:

- The target platform has substantial processing capacity.
- Large amounts of raw data must be retained.
- Different teams need to create different transformations.
- Analytical models change frequently.
- SQL-based transformations are preferred.
- A data warehouse or lakehouse provides scalable processing.

### 4.4 Neither approach is universally better

ETL is not obsolete, and ELT is not automatically superior.

The correct choice depends on:

- Data volume.
- Data velocity.
- Processing requirements.
- Security policies.
- Infrastructure.
- Budget.
- Operational skills.
- Business requirements.

Some architectures combine both approaches.

For example, an organization may filter sensitive information before loading and then perform analytical transformations inside a warehouse.

---

## 5. Staging Areas

A staging area is an intermediate location where data is temporarily stored during an integration process.

It separates extraction from subsequent processing.

### 5.1 Why staging areas exist

A staging area can help:

- Preserve extracted records temporarily.
- Validate incoming data.
- Separate source systems from target systems.
- Recover failed processes.
- Compare source and destination records.
- Process data in controlled batches.

### 5.2 Example architecture

```text
Source Database
      |
      v
Extraction Process
      |
      v
Staging Area
      |
      +---- Validation
      |
      +---- Standardization
      |
      +---- Deduplication
      |
      v
Target Warehouse
```

### 5.3 Staging database

A staging database may contain temporary tables.

Example:

```sql
CREATE SCHEMA staging;

CREATE TABLE staging.vehicles (
    inventory_number VARCHAR(30),
    manufacturer TEXT,
    equipment_type TEXT,
    status TEXT,
    source_updated_at TIMESTAMPTZ
);
```

The staging schema separates incoming data from production application tables.

### 5.4 Raw vs. staging vs. curated data

These concepts are related but not identical.

| Layer | Purpose |
|---|---|
| Raw | Preserve data close to its original form |
| Staging | Prepare data for processing and validation |
| Curated | Store cleaned, integrated, and structured datasets |

A raw layer emphasizes preservation.

A staging layer emphasizes intermediate processing.

A curated layer emphasizes usability and consistency.

Not every architecture implements these layers as separate physical systems.

---

## 6. Full Loading and Incremental Loading

A loading strategy determines how much data is transferred during each execution.

Two common approaches are full loading and incremental loading.

### 6.1 Full loading

Full loading transfers the complete dataset during each execution.

Example:

```sql
SELECT *
FROM vehicles;
```

The entire source table is extracted.

**Advantages**

- Simple implementation.
- Easier to reason about.
- Useful for initial loads.
- Can reconstruct the target dataset from the source.

**Disadvantages**

- Transfers unnecessary records.
- Can consume significant network bandwidth.
- Increases processing time.
- Can place additional load on source systems.

Full loading may be acceptable for small datasets or infrequent refreshes.

### 6.2 Incremental loading

Incremental loading transfers only new or modified records.

Example:

```sql
SELECT
    inventory_number,
    manufacturer,
    status,
    updated_at
FROM vehicles
WHERE updated_at > :last_successful_timestamp;
```

The process retrieves records modified after the last successful checkpoint.

**Advantages**

- Less data movement.
- Faster processing for large datasets.
- Reduced source workload.
- More frequent updates can become practical.

**Disadvantages**

- More complex implementation.
- Requires reliable change detection.
- Must handle deleted records.
- Requires checkpoint management.
- Can miss changes if the extraction logic is incorrect.

### 6.3 Initial load and subsequent loads

A common architecture combines both strategies.

```text
First Execution
      |
      v
Full Load
      |
      v
Target Initialized
      |
      v
Subsequent Executions
      |
      v
Incremental Load
```

### 6.4 Handling deletions

A simple timestamp query may detect inserted or updated records but may not detect deleted records.

Possible solutions include:

- Soft deletes.
- Deletion flags.
- CDC.
- Tombstone records.
- Periodic full reconciliation.

A soft delete marks a record as deleted without physically removing it.

Example:

```sql
ALTER TABLE vehicles
ADD COLUMN deleted_at TIMESTAMPTZ;
```

A record with a non-null `deleted_at` value is considered deleted according to the application's business rules.

---

## 7. Transformation Categories

Transformations can be classified according to the operation performed.

### 7.1 Structural transformations

Modify the organization of data.

Examples:

- Renaming columns.
- Changing data types.
- Splitting fields.
- Combining fields.
- Pivoting and unpivoting tables.

### 7.2 Semantic transformations

Modify the meaning or interpretation of values.

Examples:

- Mapping `ACTIVE` to `In Service`.
- Converting miles to kilometers.
- Standardizing equipment categories.
- Resolving different status definitions.

### 7.3 Data cleansing

Identifies and corrects data problems.

Examples:

- Removing invalid values.
- Handling missing attributes.
- Correcting formatting inconsistencies.
- Detecting duplicates.

### 7.4 Aggregation

Combines multiple records into summarized values.

Example:

```sql
SELECT
    equipment_type,
    COUNT(*) AS total_units
FROM vehicles
GROUP BY equipment_type;
```

### 7.5 Enrichment

Adds additional information to existing records.

For example, a vehicle record may be enriched with:

- Department information.
- Geographic classification.
- Equipment category.
- Reference data.
- External identifiers.

### 7.6 Filtering

Selects only records relevant to a particular process.

Example:

```sql
SELECT *
FROM vehicles
WHERE status <> 'Out of Service';
```

Filtering should be based on documented business requirements. Excluding records without understanding their meaning can introduce analytical bias.

---

## 8. Schema-on-Write and Schema-on-Read

These concepts describe when data structure is applied or interpreted.

### 8.1 Schema-on-write

A schema is defined and enforced when data is written into the destination.

Typical characteristics:

- Structured target tables.
- Defined columns and data types.
- Validation before or during loading.
- Consistent storage representation.

Traditional ETL architectures commonly use this approach.

### 8.2 Schema-on-read

Data is stored before a complete analytical structure is imposed.

The structure is interpreted when the data is read or processed.

Typical characteristics:

- Flexible raw storage.
- Multiple possible interpretations.
- Useful for semi-structured and unstructured data.
- Transformations may occur during analysis.

This approach is common in data lakes.

### 8.3 Comparison

| Characteristic | Schema-on-write | Schema-on-read |
|---|---|---|
| Schema definition | Before or during writing | During reading |
| Data validation | Often earlier | May occur during processing |
| Flexibility | More controlled | More flexible |
| Common environment | Traditional warehouses | Data lakes |
| Main risk | Rigid schema changes | Inconsistent interpretation |

Modern platforms may support both approaches.

---

## 9. Batch and Streaming Transformations

ETL and ELT are often associated with batch processing, but transformation can also occur in streaming architectures.

### 9.1 Batch processing

Batch processing handles a collection of records as a group.

Example:

```text
Every day at 02:00 AM
        |
        v
Extract daily records
        |
        v
Transform dataset
        |
        v
Load analytical tables
```

Useful when:

- Data does not need immediate availability.
- Processing occurs at scheduled intervals.
- Large groups of records are processed together.

### 9.2 Stream processing

Stream processing handles events continuously or in small increments.

Example:

```text
Vehicle Sensor Events
        |
        v
Message Broker
        |
        v
Stream Processing
        |
        v
Operational Dashboard
```

Useful when:

- Data must be processed with low latency.
- Events arrive continuously.
- Systems must react to recent changes.

### 9.3 Important distinction

Batch and streaming describe processing modes.

ETL and ELT describe the order and location of extraction, transformation, and loading.

They are related but independent architectural dimensions.

A system can perform batch ETL, batch ELT, or streaming transformations.

---

## 10. Orchestration

A transformation process may consist of many dependent tasks.

For example:

1. Extract vehicle records.
2. Extract maintenance records.
3. Validate identifiers.
4. Standardize dates.
5. Join datasets.
6. Calculate maintenance metrics.
7. Load analytical tables.
8. Run reconciliation checks.

**Data orchestration** coordinates the execution of these tasks.

### 10.1 Why orchestration matters

Orchestration manages:

- Task dependencies.
- Execution schedules.
- Retries.
- Failure handling.
- Logging.
- Notifications.
- Workflow status.
- Recovery.

### 10.2 Example workflow

```text
Extract Vehicles
       |
       v
Extract Maintenance
       |
       v
Validate Sources
       |
       v
Transform Data
       |
       v
Load Warehouse
       |
       v
Reconciliation
       |
       v
Publish Results
```

If source validation fails, the workflow should not automatically publish incomplete analytical results.

### 10.3 Common orchestration tools

| Tool | Main purpose |
|---|---|
| Apache Airflow | Workflow orchestration |
| Prefect | Data workflow orchestration |
| Dagster | Data orchestration and asset management |
| Azure Data Factory | Cloud data integration and orchestration |
| AWS Glue | Managed data integration and processing |
| Google Cloud Dataflow | Managed stream and batch processing |

These tools differ in execution models, deployment options, integrations, and operational complexity.

---

## 11. Common ETL and ELT Tools

### 11.1 SQL

SQL is used to:

- Extract data.
- Join datasets.
- Filter records.
- Aggregate information.
- Validate constraints.
- Create analytical tables.

It is particularly important in database-centered transformation workflows.

### 11.2 Python

Python is useful for:

- API extraction.
- File processing.
- Data cleansing.
- Custom transformation logic.
- Automation.
- Integration with external systems.

### 11.3 Pandas

Pandas provides data structures and operations for working with tabular datasets.

Example:

```python
import pandas as pd

vehicles = pd.read_csv("vehicles.csv")
maintenance = pd.read_csv("maintenance.csv")

# Standardize the shared identifier.
vehicles["inventory_number"] = (
    vehicles["inventory_number"].astype("string").str.strip()
)

maintenance["inventory_number"] = (
    maintenance["inventory_number"].astype("string").str.strip()
)

# Combine the two datasets.
integrated = vehicles.merge(
    maintenance,
    on="inventory_number",
    how="left"
)

print(integrated.head())
```

Pandas is especially useful for small and medium-sized datasets that fit comfortably in memory.

For larger distributed workloads, frameworks such as Apache Spark may be more appropriate.

### 11.4 Apache Spark

Apache Spark is a distributed data processing engine.

It can process large datasets across multiple machines.

Typical applications include:

- Large-scale transformations.
- Distributed joins.
- Aggregations.
- Batch processing.
- Streaming workloads.

### 11.5 dbt

dbt is a tool for managing data transformations, particularly SQL-based transformations inside analytical platforms.

It supports:

- Model organization.
- Dependencies.
- Testing.
- Documentation.
- Incremental models.
- Version control integration.

dbt is commonly associated with ELT architectures because it executes transformations within the data platform.

### 11.6 Integration platforms

Examples include:

- Informatica.
- Talend.
- Microsoft SQL Server Integration Services (SSIS).
- Fivetran.
- Airbyte.

These platforms provide different combinations of extraction, loading, transformation, connectors, and orchestration.

---

## 12. Data Quality in ETL and ELT

A pipeline can execute successfully and still produce incorrect data.

Technical success does not automatically mean data quality.

A reliable process needs explicit quality checks.

### 12.1 Completeness

Are required fields present?

```sql
SELECT COUNT(*) AS missing_inventory_numbers
FROM staging.vehicles
WHERE inventory_number IS NULL;
```

### 12.2 Uniqueness

Are identifiers duplicated?

```sql
SELECT
    inventory_number,
    COUNT(*) AS occurrences
FROM staging.vehicles
GROUP BY inventory_number
HAVING COUNT(*) > 1;
```

### 12.3 Validity

Do values follow the expected rules?

```sql
SELECT *
FROM staging.vehicles
WHERE status NOT IN (
    'In Service',
    'Under Maintenance',
    'Out of Service'
);
```

### 12.4 Consistency

Do related datasets agree?

For example, does every maintenance record reference a valid vehicle?

### 12.5 Freshness

Is the dataset recent enough for its intended use?

A daily financial report may tolerate a daily refresh, while an operational dashboard may require more frequent updates.

### 12.6 Accuracy

Does the data correctly represent the real-world entity or event?

Accuracy may require comparing data with authoritative records or verified source documents.

---

## 13. Reliability and Recovery

ETL and ELT processes must handle interruptions.

Common failures include:

- Source database unavailable.
- API timeout.
- Invalid file.
- Network interruption.
- Transformation error.
- Storage capacity problems.
- Duplicate records.
- Target database unavailable.

### 13.1 Checkpoints

A checkpoint records how far a pipeline has progressed.

For incremental extraction, it may store the last successfully processed timestamp or source position.

### 13.2 Idempotent loading

A pipeline should ideally be safe to execute again after a failure.

A common technique is an upsert.

In PostgreSQL:

```sql
INSERT INTO warehouse_vehicles (
    inventory_number,
    manufacturer,
    status
)
VALUES (
    '13990',
    'Shantui',
    'In Service'
)
ON CONFLICT (inventory_number)
DO UPDATE SET
    manufacturer = EXCLUDED.manufacturer,
    status = EXCLUDED.status;
```

This inserts a new record or updates the existing record with the same unique identifier.

The unique constraint on `inventory_number` is essential for this example.

### 13.3 Error isolation

Invalid records should be handled separately rather than silently discarded.

A pipeline may separate:

```text
Incoming Data
      |
      +---- Valid Records
      |         |
      |         v
      |      Target
      |
      +---- Rejected Records
                |
                v
          Error Repository
```

Rejected records should retain enough diagnostic information to support correction and reprocessing.

### 13.4 Recovery strategy

A recovery plan should define:

- Which task can be restarted.
- Whether previous tasks must be repeated.
- How duplicates are prevented.
- How checkpoints are restored.
- How failed records are recovered.
- How the final result is reconciled.

---

## 14. Data Processing Layers and the Medallion Architecture

A common approach in modern data platforms is to organize datasets into progressively refined layers.

The **Medallion Architecture** commonly uses three layers:

- Bronze.
- Silver.
- Gold.

### 14.1 Bronze layer

Contains raw or minimally processed source data.

Characteristics:

- Preserves source information.
- Supports traceability.
- May contain inconsistent records.
- Retains ingestion metadata.

### 14.2 Silver layer

Contains cleaned and standardized data.

Typical operations:

- Deduplication.
- Type conversion.
- Validation.
- Standardization.
- Integration.
- Entity resolution.

### 14.3 Gold layer

Contains business-ready datasets.

Examples:

- Financial indicators.
- Fleet performance reports.
- Maintenance KPIs.
- Department-level summaries.
- Analytical data marts.

### 14.4 Architecture

```text
       Source Systems
             |
             v
       Bronze Layer
       Raw Data
             |
             v
       Silver Layer
   Cleaned and Integrated
             |
             v
        Gold Layer
   Business-Ready Datasets
             |
             v
       BI / Analytics
```

### 14.5 Relationship with ETL and ELT

Medallion architecture is a data organization pattern, not a replacement for ETL or ELT.

It is commonly implemented using ELT-like workflows:

1. Load source data into Bronze.
2. Transform and standardize it into Silver.
3. Aggregate and prepare business datasets in Gold.

The actual implementation can also include ETL operations before data enters the Bronze layer.

---

## 15. ETL and ELT Architecture Decision Framework

The following factors can help select an approach.

| Decision factor | ETL considerations | ELT considerations |
|---|---|---|
| Data volume | External processing may be suitable | Target platform can process large volumes |
| Data sensitivity | Filter before loading | Requires secure raw-data storage |
| Raw data retention | May retain only selected outputs | Commonly preserves source data |
| Transformation complexity | External tools can handle specialized logic | SQL and warehouse engines can handle many transformations |
| Flexibility | Changes may require upstream reprocessing | Raw data may support new transformations |
| Infrastructure | Separate processing infrastructure | Uses target platform resources |
| Governance | Control before loading | Strong controls needed across raw and refined layers |

### 15.1 Questions before choosing

1. How much data will be processed?
2. How frequently will it arrive?
3. Does the target platform have sufficient processing capacity?
4. Must raw data be retained?
5. Is sensitive information present?
6. How complex are the transformations?
7. Can the pipeline be safely rerun?
8. What is the expected recovery time?
9. Who owns the transformation logic?
10. How will the output be validated?

---

## 16. Practical Example: ETL Workflow for SIAPV

Consider a conceptual analytical process based on SIAPV information.

The objective is to produce a report showing vehicle inventory, operational status, mileage, and operating hours.

The information is stored in PostgreSQL.

This example describes a possible analytical workflow. It does not imply that SIAPV currently uses an external ETL platform.

### 16.1 Source data

Assume two source tables:

```text
vehicles
--------
inventory_number
manufacturer
equipment_type
status

operational_readings
--------------------
reading_id
inventory_number
mileage
horometer
reading_date
```

### 16.2 Extract

Retrieve vehicle information:

```sql
SELECT
    inventory_number,
    manufacturer,
    equipment_type,
    status
FROM vehicles;
```

Retrieve operational readings:

```sql
SELECT
    reading_id,
    inventory_number,
    mileage,
    horometer,
    reading_date
FROM operational_readings;
```

### 16.3 Transform

The transformation process could:

- Validate inventory numbers.
- Standardize status values.
- Check for negative readings.
- Validate dates.
- Identify duplicate readings.
- Associate readings with vehicles.
- Calculate the latest known operational values.

Example:

```sql
SELECT
    v.inventory_number,
    v.manufacturer,
    v.equipment_type,
    v.status,
    r.mileage,
    r.horometer,
    r.reading_date
FROM vehicles AS v
LEFT JOIN LATERAL (
    SELECT
        mileage,
        horometer,
        reading_date
    FROM operational_readings AS r
    WHERE r.inventory_number = v.inventory_number
    ORDER BY r.reading_date DESC, r.reading_id DESC
    LIMIT 1
) AS r ON TRUE;
```

The query selects the latest available operational reading for each vehicle.

The ordering assumes `reading_id` provides a deterministic tie-breaker when multiple records have the same date.

### 16.4 Load

The prepared dataset can be loaded into an analytical table.

```sql
CREATE TABLE IF NOT EXISTS analytics_vehicle_status (
    inventory_number VARCHAR(30) PRIMARY KEY,
    manufacturer TEXT,
    equipment_type TEXT,
    operational_status TEXT,
    latest_mileage NUMERIC,
    latest_horometer NUMERIC,
    latest_reading_date DATE,
    refreshed_at TIMESTAMPTZ DEFAULT NOW()
);
```

A production pipeline would populate this table using a controlled insert or upsert process.

### 16.5 Result

The analytical dataset can support questions such as:

- How many units are currently in service?
- Which vehicles have the highest mileage?
- Which machines have the highest operating hours?
- Which vehicles lack recent readings?
- How is the fleet distributed by equipment type?

The architecture separates operational data management from analytical consumption.

---

## 17. Common Mistakes

### 17.1 Transforming without preserving source context

If the original values are overwritten, it may become difficult to investigate incorrect transformations.

Retain appropriate source metadata and raw records when required.

### 17.2 Loading without validation

A technically successful load may still introduce invalid data.

Implement quality checks before publishing datasets.

### 17.3 Using full loads unnecessarily

Repeatedly transferring large datasets can waste processing resources.

Evaluate incremental loading when data volume increases.

### 17.4 Ignoring deleted records

Incremental pipelines must define how deletions are detected and propagated.

### 17.5 Designing non-idempotent pipelines

A failed workflow may need to be restarted.

If rerunning creates duplicate records or repeated financial effects, recovery becomes dangerous.

### 17.6 Mixing transformation logic across systems

When business rules are duplicated in several scripts and applications, inconsistencies become difficult to control.

Centralize or clearly document transformation logic.

### 17.7 Treating orchestration as transformation

Orchestration coordinates tasks; it does not automatically define the quality or correctness of the transformations.

### 17.8 Ignoring lineage

A report should be traceable to the source data and transformation steps that produced it.

---

## 18. Knowledge Check

1. What does ETL stand for?
2. What does ELT stand for?
3. What is the main architectural difference between ETL and ELT?
4. Why is a staging area useful?
5. What is the difference between full loading and incremental loading?
6. Why can timestamp-based extraction fail to detect deletions?
7. What is the difference between schema-on-write and schema-on-read?
8. Are batch processing and ETL the same concept?
9. What responsibilities does an orchestration tool perform?
10. Why should invalid records be isolated rather than silently discarded?
11. What does idempotent loading mean?
12. What are the Bronze, Silver, and Gold layers?
13. Why is retaining raw data useful?
14. What is the difference between a technically successful pipeline and a reliable data pipeline?

---

## 19. Practical Exercises

### Exercise 1: Compare ETL and ELT

An organization receives data from:

- A PostgreSQL database.
- A REST API.
- Several Excel workbooks.

The organization wants to build a centralized analytical repository.

Design one ETL architecture and one ELT architecture.

Explain:

- Where transformation occurs.
- Where raw data is stored.
- Which infrastructure is required.
- How data quality is controlled.
- Which architecture you would recommend and why.

### Exercise 2: Design a staging schema

Create a PostgreSQL schema called `staging`.

Define a table for vehicle records with:

- Inventory number.
- Manufacturer.
- Equipment type.
- Status.
- Source update timestamp.
- Ingestion timestamp.

Add appropriate data types and constraints.

### Exercise 3: Incremental extraction

Write a SQL query that extracts records modified after a given timestamp.

Then explain:

- How the timestamp is stored.
- When the checkpoint is updated.
- What happens if the pipeline fails.
- How deletions are detected.

### Exercise 4: Data transformation

Using Python and Pandas:

1. Load a CSV file.
2. Remove whitespace from text fields.
3. Standardize status values.
4. Convert a date column to datetime.
5. Detect duplicate inventory numbers.
6. Export the cleaned dataset.

### Exercise 5: Data quality

Write SQL queries that detect:

- Missing inventory numbers.
- Duplicate identifiers.
- Invalid statuses.
- Negative mileage.
- Missing operational readings.

### Exercise 6: SIAPV analytical architecture

Design a conceptual ETL or ELT workflow using SIAPV data.

Your design must include:

- Source tables.
- Extraction method.
- Staging layer.
- Transformation rules.
- Analytical destination.
- Incremental loading strategy.
- Error handling.
- Reconciliation.
- Monitoring.

Explain why you selected ETL or ELT.

---

## 20. Glossary

| Term | Definition |
|---|---|
| ETL | Extract, Transform, Load |
| ELT | Extract, Load, Transform |
| Extract | Retrieve data from source systems |
| Transform | Modify data into a required representation |
| Load | Store data in a target environment |
| Staging Area | Intermediate storage used during data processing |
| Raw Data | Data preserved close to its original source representation |
| Full Load | Transfer of an entire dataset |
| Incremental Load | Transfer of only new or modified data |
| Upsert | Insert a record or update it when a matching key exists |
| Checkpoint | Stored processing position used to resume a workflow |
| Schema-on-Write | Apply schema constraints when data is written |
| Schema-on-Read | Interpret data structure when it is read |
| Batch Processing | Processing records in groups |
| Stream Processing | Processing continuously arriving events |
| Data Orchestration | Coordinating the execution of data tasks |
| Data Cleansing | Identifying and correcting data quality problems |
| Data Enrichment | Adding useful information to existing records |
| Data Aggregation | Summarizing multiple records |
| Data Lineage | Tracking data origin and transformations |
| Medallion Architecture | Data organization pattern using Bronze, Silver, and Gold layers |
| Bronze Layer | Raw or minimally processed data |
| Silver Layer | Cleaned and standardized data |
| Gold Layer | Business-ready analytical data |
| dbt | Tool for managing analytical data transformations |
| Apache Spark | Distributed data processing engine |
| Data Warehouse | Repository optimized for analytical workloads |

---

## 21. Connections to Other Topics

### Previous notes

- [[01 - Data Architecture Overview]]
- [[02 - Data Integration]]
- [[04 - OLTP vs OLAP]]
- [[02 - The Data Lifecycle]]

### Related notes

- [[04 - Data Pipelines]]
- [[05 - Data Warehouses]]
- [[06 - Data Lakes]]
- [[07 - Lakehouse Architecture]]
- [[03 - Data Quality]]
- [[04 - Data Governance]]
- [[03 - PostgreSQL Storage]]

### Practical applications

- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]

---

## 22. Key Takeaways

1. ETL and ELT are approaches for moving and preparing data.
2. ETL transforms information before loading it into its final destination.
3. ELT loads information first and transforms it inside the target environment.
4. Staging areas separate extraction from subsequent processing.
5. Full loading transfers complete datasets, while incremental loading transfers changes.
6. Data transformations include cleansing, standardization, integration, enrichment, filtering, and aggregation.
7. Schema-on-write and schema-on-read define different approaches to data structure.
8. Batch and streaming describe processing modes, not ETL or ELT sequences.
9. Orchestration coordinates dependencies, execution, retries, and recovery.
10. Reliable pipelines require data quality checks, idempotency, monitoring, and lineage.
11. Bronze, Silver, and Gold layers organize data according to its level of refinement.
12. The correct architecture depends on business requirements, data characteristics, infrastructure, and governance.

**Next topic:** [[04 - Data Pipelines]]

The next note will explore how individual extraction, transformation, and loading tasks become complete automated workflows, including dependencies, scheduling, orchestration, monitoring, and recovery.
