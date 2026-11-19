
---
title: "Data Pipelines"
aliases:
  - "Data Pipeline Fundamentals"
  - "Data Pipeline Architecture"
  - "Data Workflow Engineering"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: data-pipelines
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - data-pipelines
  - data-engineering
  - data-architecture
  - etl
  - elt
  - workflow-orchestration
  - data-ingestion
  - data-processing
  - data-automation
  - data-quality
  - batch-processing
  - stream-processing
concepts:
  - pipeline
  - workflow
  - task
  - dependency
  - orchestration
  - scheduling
  - data-ingestion
  - data-transformation
  - data-loading
  - batch-processing
  - stream-processing
  - incremental-processing
  - checkpoint
  - idempotency
  - observability
  - fault-tolerance
  - retry-policy
  - backfill
  - data-lineage
  - service-level-objective
prerequisites:
  - "[[01 - Data Architecture Overview]]"
  - "[[02 - Data Integration]]"
  - "[[03 - ETL and ELT]]"
  - "[[04 - OLTP vs OLAP]]"
related:
  - "[[05 - Data Warehouses]]"
  - "[[06 - Data Lakes]]"
  - "[[07 - Lakehouse Architecture]]"
  - "[[03 - Data Quality]]"
  - "[[04 - Data Governance]]"
  - "[[01 - PostgreSQL Architecture Overview]]"
tools:
  - Python
  - SQL
  - PostgreSQL
  - Pandas
  - Apache Airflow
  - Prefect
  - Dagster
  - Apache Kafka
  - Docker
  - dbt
learning_objectives:
  - "Define a data pipeline and explain its architectural components."
  - "Distinguish data pipelines from individual ETL and ELT operations."
  - "Understand tasks, dependencies, workflows, and directed acyclic graphs."
  - "Compare batch and streaming pipelines."
  - "Understand scheduling and event-based execution."
  - "Design reliable pipelines with retries, checkpoints, and idempotency."
  - "Explain backfills, recovery, and failure handling."
  - "Understand pipeline monitoring and observability."
  - "Identify common orchestration platforms."
  - "Design a conceptual data pipeline using SIAPV information."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Data Pipelines

## 1. Introduction

A data pipeline is a structured process that moves data from one or more sources through a sequence of operations until it reaches a destination where it can be stored, analyzed, or consumed.

A pipeline may perform:

- Data extraction.
- Data ingestion.
- Data validation.
- Data transformation.
- Data integration.
- Data enrichment.
- Data loading.
- Data publication.

The operations can run automatically according to a schedule, in response to an event, or continuously as new information arrives.

A data pipeline is not simply a SQL query or a Python script. It is an organized workflow that includes processing logic, dependencies, execution rules, error handling, and monitoring.

### 1.1 Basic example

Imagine an organization that needs a daily report of vehicle maintenance.

The information comes from:

- A PostgreSQL fleet database.
- An external maintenance application.
- A collection of CSV files.

A pipeline can automate the process:

```text
Source Systems
      |
      v
Data Extraction
      |
      v
Data Validation
      |
      v
Data Integration
      |
      v
Data Transformation
      |
      v
Data Warehouse
      |
      v
Daily Report
```

Without a pipeline, an employee might have to download files, combine spreadsheets, correct values, execute queries, and prepare the report manually.

With a pipeline, those operations can become a repeatable and monitored process.

---

## 2. Data Pipeline vs. ETL vs. ELT

These concepts are related but represent different levels of an architecture.

### 2.1 Data pipeline

A data pipeline represents the complete workflow through which information travels.

It includes the sequence of tasks and the mechanisms that coordinate them.

### 2.2 ETL

ETL is a processing approach:

1. Extract.
2. Transform.
3. Load.

It describes the order of these operations.

### 2.3 ELT

ELT is another processing approach:

1. Extract.
2. Load.
3. Transform.

It changes where transformation occurs in relation to loading.

### 2.4 Relationship

```text
                 DATA PIPELINE
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
    Extraction     Processing       Delivery
                       |
                +------+------+
                |             |
                v             v
               ETL           ELT
```

A pipeline can implement ETL, ELT, or a combination of both.

For example, a pipeline might extract data from an API, validate the response using Python, load the raw records into a data lake, and transform them later using SQL.

---

## 3. Main Components of a Data Pipeline

A complete pipeline usually contains several logical components.

### 3.1 Data sources

Sources provide the original information.

Examples:

- PostgreSQL.
- MySQL.
- SQL Server.
- REST APIs.
- CSV files.
- Excel workbooks.
- ERP systems.
- CRM systems.
- Message brokers.
- IoT devices.

A pipeline may have one source or many.

### 3.2 Ingestion layer

The ingestion layer retrieves information from the sources and introduces it into the processing environment.

Common mechanisms include:

- SQL extraction.
- API requests.
- File transfers.
- Event subscriptions.
- CDC.
- Scheduled exports.

### 3.3 Processing layer

The processing layer performs operations on the incoming data.

Examples:

- Validation.
- Deduplication.
- Standardization.
- Filtering.
- Joining.
- Aggregation.
- Calculations.

Processing can occur through SQL, Python, Spark, or other technologies.

### 3.4 Storage layer

The storage layer preserves the data during or after processing.

Possible destinations include:

- Relational databases.
- Data warehouses.
- Data lakes.
- Object storage.
- Lakehouse platforms.

Some pipelines use intermediate storage locations to separate tasks.

### 3.5 Orchestration layer

The orchestration layer coordinates when and how tasks execute.

It manages:

- Task dependencies.
- Scheduling.
- Execution order.
- Retries.
- Failure handling.
- Workflow status.

### 3.6 Consumption layer

The final information is delivered to its consumers.

Examples:

- Power BI dashboards.
- Business reports.
- Analytical queries.
- Machine learning models.
- Applications.
- APIs.

### 3.7 Monitoring layer

Monitoring provides visibility into pipeline behavior.

It can track:

- Execution duration.
- Successful and failed tasks.
- Number of records processed.
- Data freshness.
- Resource utilization.
- Error frequency.

---

## 4. Tasks and Workflows

A pipeline is generally composed of smaller units of work called tasks.

A task performs a specific operation.

Examples:

- Extract vehicle records.
- Download a maintenance file.
- Validate inventory identifiers.
- Remove duplicates.
- Calculate monthly totals.
- Load a warehouse table.

### 4.1 Task

A task should have a clearly defined responsibility.

For example:

```text
Task: Validate Vehicle Records

Input:
Extracted vehicle dataset

Operations:
- Check required fields
- Validate inventory identifiers
- Detect duplicate records
- Check allowed status values

Output:
Validated dataset
```

A task should ideally be understandable, testable, and reusable.

### 4.2 Workflow

A workflow organizes several tasks into a coordinated process.

Example:

```text
Extract Vehicles
       |
       v
Validate Vehicles
       |
       v
Transform Vehicles
       |
       v
Load Warehouse
       |
       v
Generate Report
```

The workflow defines the relationship between tasks.

### 4.3 Task dependencies

A dependency indicates that one task requires another task to complete before it can execute.

For example:

- Transformation depends on extraction.
- Loading depends on transformation.
- Reporting depends on successful loading.

Dependencies prevent tasks from executing in an invalid order.

### 4.4 Parallel tasks

Not every task must execute sequentially.

Suppose vehicle and maintenance data come from independent sources.

Both extraction tasks can run at the same time.

```text
           Start
             |
       +-----+-----+
       |           |
       v           v
Extract         Extract
Vehicles        Maintenance
       |           |
       v           v
Validate        Validate
Vehicles        Maintenance
       |           |
       +-----+-----+
             |
             v
       Integrate Data
             |
             v
        Load Results
```

Parallel execution can reduce total processing time.

However, tasks should run in parallel only when their dependencies and resource requirements allow it.

---

## 5. Directed Acyclic Graphs (DAGs)

A **Directed Acyclic Graph (DAG)** is a structure commonly used to represent workflow dependencies.

It contains:

- Nodes: individual tasks.
- Directed edges: dependency relationships.
- No directed cycles.

### 5.1 Directed

A dependency has a direction.

If Task B depends on Task A, the relationship is:

```text
Task A ---> Task B
```

### 5.2 Acyclic

A valid DAG does not contain a directed cycle.

For example, this is invalid:

```text
Task A ---> Task B
   ^           |
   |           v
   +-------- Task C
```

The tasks depend on one another in a circular way.

A workflow cannot determine a valid execution order if its dependencies form a cycle.

### 5.3 Example DAG

```text
              START
                |
        +-------+-------+
        |               |
        v               v
 Extract Fleet    Extract Maintenance
        |               |
        v               v
 Validate Fleet   Validate Maintenance
        |               |
        +-------+-------+
                |
                v
          Integrate Data
                |
                v
          Calculate KPIs
                |
                v
           Load Warehouse
                |
                v
          Publish Report
```

### 5.4 Topological ordering

A topological ordering is a sequence of tasks that respects all dependency relationships.

For the previous example, one valid order is:

1. Extract Fleet.
2. Extract Maintenance.
3. Validate Fleet.
4. Validate Maintenance.
5. Integrate Data.
6. Calculate KPIs.
7. Load Warehouse.
8. Publish Report.

The first two extraction tasks may run simultaneously.

A DAG does not necessarily prescribe one single execution sequence. It defines the dependency constraints that valid execution orders must respect.

---

## 6. Batch Data Pipelines

A batch pipeline processes data in groups at scheduled or triggered intervals.

For example, a company may process its operational records every night.

### 6.1 Batch architecture

```text
Source Systems
      |
      v
Scheduled Extraction
      |
      v
Batch Dataset
      |
      v
Validation
      |
      v
Transformation
      |
      v
Warehouse Loading
      |
      v
Daily Reports
```

### 6.2 Common characteristics

- Processes a defined collection of records.
- Executes at specific intervals.
- Often handles large datasets.
- Can use full or incremental loading.
- Usually has a clear start and completion time.

### 6.3 Example schedules

| Frequency | Example |
|---|---|
| Every 5 minutes | Frequent operational synchronization |
| Every hour | Periodic application integration |
| Every night | Daily analytical refresh |
| Weekly | Management reporting |
| Monthly | Financial consolidation |

The appropriate frequency depends on how quickly the business needs the information.

### 6.4 Advantages

- Relatively simple operational model.
- Efficient for large collections of records.
- Easier to reproduce a defined processing period.
- Can be scheduled during low-activity periods.

### 6.5 Limitations

- Information may become outdated between executions.
- Large batches can require substantial resources.
- A failed execution can delay the entire reporting cycle.

---

## 7. Streaming Data Pipelines

A streaming pipeline processes data continuously or as events arrive.

Instead of waiting for a scheduled batch, the system processes incoming events with low latency.

### 7.1 Streaming architecture

```text
Event Producers
      |
      v
Message Broker
      |
      v
Stream Processor
      |
      v
Operational Storage
      |
      v
Real-Time Dashboard
```

### 7.2 Example

Imagine a fleet management system receiving telemetry from equipment.

Each device generates events containing:

- Vehicle identifier.
- Timestamp.
- Operating hours.
- Location.
- Engine temperature.
- Other sensor measurements.

A streaming pipeline could process these events and identify abnormal operating conditions.

### 7.3 Common technologies

- Apache Kafka.
- Apache Flink.
- Spark Structured Streaming.
- Amazon Kinesis.
- Azure Event Hubs.

### 7.4 Advantages

- Low-latency processing.
- Continuous data availability.
- Immediate reaction to events.
- Useful for monitoring and alerting.

### 7.5 Limitations

- More complex operational requirements.
- Event ordering can be difficult.
- Duplicate delivery must be handled.
- Late-arriving events require special treatment.
- Continuous infrastructure may increase costs.

---

## 8. Batch vs. Streaming

| Characteristic | Batch | Streaming |
|---|---|---|
| Processing | Groups of records | Continuous or event-based |
| Latency | Usually higher | Usually lower |
| Execution | Scheduled or triggered | Continuous or event-triggered |
| Data volume | Often large collections | Continuous event flow |
| Complexity | Often simpler | Often more complex |
| Common tools | Airflow, dbt, Spark batch | Kafka, Flink, Spark Streaming |
| Common use | Daily reporting | Monitoring and alerts |

Neither approach is universally superior.

A system may use batch for historical analytics and streaming for immediate operational monitoring.

---

## 9. Pipeline Execution Strategies

A pipeline can be started in different ways.

### 9.1 Time-based scheduling

The pipeline runs according to a predefined schedule.

Examples:

- Every day at 02:00.
- Every hour.
- Every Monday.
- On the first day of each month.

Scheduling is useful when data is expected at predictable intervals.

### 9.2 Event-based execution

The pipeline starts when a specific event occurs.

Examples:

- A new file is uploaded.
- A database change is detected.
- A message arrives.
- An API receives a notification.
- An upstream process completes.

### 9.3 Manual execution

An authorized operator starts the pipeline manually.

This is useful for:

- Testing.
- Recovery.
- Historical processing.
- Administrative operations.

### 9.4 Continuous execution

The pipeline remains active and processes incoming data continuously.

This is common in streaming architectures.

---

## 10. Pipeline Reliability

A pipeline must continue operating correctly despite failures.

A successful architecture anticipates interruptions rather than assuming that every task will always work.

### 10.1 Common failure scenarios

| Failure | Possible cause |
|---|---|
| Extraction failure | Source database unavailable |
| API failure | Timeout or rate limit |
| Transformation failure | Invalid data or programming error |
| Loading failure | Database unavailable |
| Scheduling failure | Orchestration service interruption |
| Data inconsistency | Incorrect mapping or missing records |
| Resource failure | Insufficient memory or storage |

### 10.2 Retry policies

A retry policy defines how the system responds to temporary failures.

A task may be configured to retry a limited number of times.

Example:

```text
Attempt 1
   |
   v
Failure
   |
   v
Wait 10 seconds
   |
   v
Attempt 2
   |
   v
Failure
   |
   v
Wait 20 seconds
   |
   v
Attempt 3
```

A retry policy should distinguish temporary failures from permanent ones.

For example:

- A network timeout may be temporary.
- An invalid inventory identifier may require data correction.
- An unauthorized request may require permission changes.

Repeatedly retrying a permanent error wastes resources.

### 10.3 Exponential backoff

Exponential backoff increases the waiting time between attempts.

\[
t_n = \min(t_{\max}, t_0 \times 2^n)
\]

Where:

- \(t_0\) is the initial delay.
- \(n\) is the retry number.
- \(t_{\max}\) is the maximum delay.

Random jitter can reduce synchronized retry attempts from multiple workers.

### 10.4 Idempotency

A task is idempotent if repeating it does not create additional unintended effects.

For example, loading a vehicle using an upsert based on a unique inventory number can be safer than blindly inserting the same record every time.

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

This example assumes that `inventory_number` has a unique constraint.

Idempotency is especially important when a task fails after completing part of its work.

### 10.5 Checkpoints

A checkpoint records the progress of a pipeline.

It may contain:

- Last successfully processed timestamp.
- Last source transaction position.
- Last event offset.
- Last completed batch identifier.

Checkpoints help pipelines resume from a known position instead of processing everything again.

### 10.6 Dead-letter handling

Records that repeatedly fail processing should be isolated.

A rejected-data repository or dead-letter queue can preserve:

- Original record.
- Error description.
- Processing timestamp.
- Source system.
- Retry count.
- Processing status.

This enables investigation and controlled reprocessing.

---

## 11. Backfills and Historical Reprocessing

A **backfill** is the process of executing a pipeline for historical data that was not previously processed or must be recalculated.

### 11.1 Example

Suppose a pipeline began processing maintenance records in January.

In April, the organization discovers that a transformation rule was incorrect.

The historical records from January through March may need to be processed again.

This is a backfill.

### 11.2 Why backfills are necessary

- Correcting transformation logic.
- Recovering from pipeline failures.
- Loading historical information.
- Rebuilding analytical tables.
- Adding new business metrics.
- Reprocessing late-arriving data.

### 11.3 Backfill risks

Backfills can create:

- Duplicate records.
- Excessive resource consumption.
- Conflicts with current processing.
- Incorrect historical calculations.

### 11.4 Safe backfill design

A controlled backfill should define:

1. Historical date range.
2. Target datasets.
3. Processing version.
4. Duplicate prevention strategy.
5. Resource limits.
6. Validation criteria.
7. Publication procedure.

Historical reprocessing should not silently overwrite correct data without a documented policy.

---

## 12. Pipeline Monitoring and Observability

A pipeline needs monitoring to determine whether it is functioning correctly.

Monitoring focuses on collecting information about system behavior.

Observability goes further by using logs, metrics, and traces to investigate why a system behaves in a particular way.

### 12.1 Logs

Logs record individual events during execution.

Example:

```text
2026-11-12 02:00:00 INFO Pipeline started
2026-11-12 02:00:03 INFO Extraction completed
2026-11-12 02:00:08 INFO 200 records extracted
2026-11-12 02:00:10 INFO Validation completed
2026-11-12 02:00:15 INFO Loading started
2026-11-12 02:00:18 INFO Pipeline completed
```

Logs should include enough context to identify the task and execution.

### 12.2 Metrics

Metrics are numerical measurements of system behavior.

Examples:

- Execution duration.
- Records extracted.
- Records loaded.
- Error count.
- Retry count.
- Data freshness.
- Processing throughput.

### 12.3 Traces

Traces follow an operation across multiple components.

For example, a record may pass through:

```text
API Request
    |
    v
Ingestion Service
    |
    v
Validation Service
    |
    v
Transformation Service
    |
    v
Warehouse
```

Tracing can help identify where delays or failures occur.

### 12.4 Pipeline health indicators

| Indicator | What it measures |
|---|---|
| Success rate | Percentage of successful executions |
| Failure rate | Percentage of failed executions |
| Duration | Time required to complete |
| Throughput | Records processed per unit of time |
| Freshness | Age of the most recent available data |
| Completeness | Whether expected records arrived |
| Retry count | Frequency of repeated attempts |

### 12.5 Data freshness

A pipeline may complete successfully while delivering outdated data.

Freshness measures how recently the dataset was updated.

For example:

```text
Expected refresh: Every 60 minutes
Last successful refresh: 08:00
Current time: 11:00
Data delay: 3 hours
```

The pipeline is not meeting its expected freshness requirement.

---

## 13. Service-Level Objectives for Data Pipelines

A **Service-Level Objective (SLO)** defines a measurable reliability or performance target.

For data pipelines, an SLO may describe:

- Maximum processing duration.
- Minimum successful execution rate.
- Maximum acceptable data delay.
- Required data completeness.

Example:

```text
Pipeline: Daily Fleet Reporting

Freshness SLO:
The dataset must be available by 06:00 every business day.

Completeness SLO:
At least 99.5% of expected valid records must be processed.

Execution SLO:
95% of scheduled runs must complete successfully.
```

An SLO should be measurable and tied to business requirements.

### 13.1 SLA vs. SLO vs. SLI

These terms are related.

| Term | Meaning |
|---|---|
| SLI | Service-Level Indicator: measured performance |
| SLO | Service-Level Objective: desired performance target |
| SLA | Service-Level Agreement: formal commitment between parties |

Example:

- SLI: Actual pipeline completion time.
- SLO: Complete the pipeline within 30 minutes.
- SLA: Contractual commitment that defines consequences if agreed service requirements are not met.

---

## 14. Pipeline Security

Pipelines frequently move information across databases, APIs, storage systems, and applications.

Security must be applied throughout the workflow.

### 14.1 Least privilege

Each pipeline component should have only the permissions required for its task.

For example:

- Extraction service: read access to source tables.
- Loading service: write access to target tables.
- Monitoring service: read access to execution metadata.

### 14.2 Credential management

Credentials should not be stored directly in scripts or source control.

Use:

- Environment variables.
- Secret managers.
- Managed identities.
- Restricted service accounts.

### 14.3 Encryption

Use secure network communication such as TLS when transmitting sensitive data.

Apply appropriate protection to stored information as well.

### 14.4 Data minimization

Avoid transferring fields that are not necessary for the pipeline's purpose.

### 14.5 Auditability

Maintain logs of important operations, including:

- Execution identity.
- Source and destination.
- Processing time.
- Records affected.
- Failure details.

---

## 15. Pipeline Architecture Patterns

### 15.1 Linear pipeline

Tasks execute sequentially.

```text
A ---> B ---> C ---> D
```

Suitable for simple workflows.

### 15.2 Parallel pipeline

Independent tasks execute simultaneously and later converge.

```text
        +---> B ---+
        |          |
A ------+          +----> D
        |          |
        +---> C ---+
```

Suitable for independent source extraction or validation.

### 15.3 Fan-out and fan-in

One task distributes work to multiple downstream tasks, which later converge.

```text
             Start
               |
        +------+------+ 
        |      |      |
        v      v      v
       A1     A2     A3
        |      |      |
        +------+------+ 
               |
               v
            Combine
```

Useful when data can be processed independently by category, region, or partition.

### 15.4 Parent-child workflows

A main workflow triggers smaller workflows.

```text
Main Pipeline
      |
      +----> Customer Pipeline
      |
      +----> Product Pipeline
      |
      +----> Finance Pipeline
```

This can improve modularity and reuse.

### 15.5 Micro-batch pipeline

A continuous stream is divided into small processing windows.

For example, events may be processed every 30 seconds.

This approach offers a compromise between traditional batch processing and continuous streaming.

---

## 16. Data Pipeline Tools

### 16.1 Apache Airflow

Apache Airflow is a platform for developing, scheduling, and monitoring workflows.

Its workflows are commonly represented as DAGs.

Typical responsibilities include:

- Scheduling.
- Task dependencies.
- Execution tracking.
- Retries.
- Logs.
- Workflow monitoring.

Airflow is not itself a general-purpose distributed data processing engine. Tasks commonly delegate computation to databases, Spark, Python services, or other platforms.

### 16.2 Prefect

Prefect is a workflow orchestration platform focused on defining and operating data workflows.

It supports:

- Task and flow definitions.
- Retries.
- Scheduling.
- State tracking.
- Execution monitoring.

### 16.3 Dagster

Dagster is a data orchestrator that emphasizes data assets, dependencies, testing, and observability.

It helps organize workflows around the datasets they produce and maintain.

### 16.4 Apache Kafka

Kafka is a distributed event streaming platform.

It supports:

- Event publishing.
- Event consumption.
- Persistent event storage.
- Partitioning.
- Consumer groups.
- Stream-based architectures.

Kafka is not a direct replacement for an orchestration platform such as Airflow. The tools address different architectural responsibilities.

### 16.5 Apache Spark

Spark is a distributed processing engine.

It can perform:

- Batch processing.
- Large-scale transformations.
- Distributed joins.
- Aggregations.
- Stream processing.

### 16.6 dbt

dbt manages SQL-based transformations in analytical data platforms.

It supports:

- Model dependencies.
- Data tests.
- Documentation.
- Incremental models.
- Version-controlled transformation logic.

### 16.7 Tool comparison

| Tool | Main responsibility |
|---|---|
| Airflow | Workflow orchestration |
| Prefect | Workflow orchestration |
| Dagster | Data orchestration and asset management |
| Kafka | Event streaming |
| Spark | Distributed data processing |
| dbt | SQL-based data transformation |
| Pandas | In-memory data manipulation |
| PostgreSQL | Relational data storage and SQL processing |

A complete architecture may combine several of these tools.

---

## 17. Practical Example: A Python Data Pipeline

Consider a small pipeline that:

1. Reads vehicle records from a CSV file.
2. Validates required fields.
3. Standardizes text.
4. Detects duplicate identifiers.
5. Exports valid records.
6. Saves rejected records separately.

### 17.1 Example input

`vehicles.csv`

```csv
inventory_number,manufacturer,status
13990, Shantui ,ACTIVE
13994,Caterpillar,REPAIR
13995,Komatsu,ACTIVE
,Unknown,ACTIVE
13990,Shantui,ACTIVE
```

### 17.2 Pipeline implementation

```python
from pathlib import Path

import pandas as pd


# --------------------------------------------------
# Configuration
# --------------------------------------------------

INPUT_FILE = Path("vehicles.csv")

OUTPUT_DIRECTORY = Path("output")

VALID_OUTPUT = OUTPUT_DIRECTORY / "vehicles_valid.csv"

REJECTED_OUTPUT = OUTPUT_DIRECTORY / "vehicles_rejected.csv"


# --------------------------------------------------
# Task 1: Extract
# --------------------------------------------------

def extract_data(file_path: Path) -> pd.DataFrame:
    """Read vehicle records from a CSV file."""

    if not file_path.exists():
        raise FileNotFoundError(
            f"Input file not found: {file_path}"
        )

    return pd.read_csv(
        file_path,
        dtype={"inventory_number": "string"}
    )


# --------------------------------------------------
# Task 2: Transform
# --------------------------------------------------

def transform_data(
    data: pd.DataFrame
) -> pd.DataFrame:
    """Standardize selected vehicle attributes."""

    transformed = data.copy()

    transformed["inventory_number"] = (
        transformed["inventory_number"]
        .str.strip()
    )

    transformed["manufacturer"] = (
        transformed["manufacturer"]
        .astype("string")
        .str.strip()
    )

    transformed["status"] = (
        transformed["status"]
        .astype("string")
        .str.strip()
        .str.upper()
    )

    status_mapping = {
        "ACTIVE": "In Service",
        "REPAIR": "Under Maintenance",
        "INACTIVE": "Out of Service"
    }

    transformed["status"] = (
        transformed["status"]
        .map(status_mapping)
        .fillna("Unknown")
    )

    return transformed


# --------------------------------------------------
# Task 3: Validate
# --------------------------------------------------

def validate_data(
    data: pd.DataFrame
) -> tuple[pd.DataFrame, pd.DataFrame]:
    """Separate valid records from rejected records."""

    valid_identifier = (
        data["inventory_number"].notna()
        & data["inventory_number"].ne("")
    )

    valid_manufacturer = (
        data["manufacturer"].notna()
        & data["manufacturer"].ne("")
    )

    valid_status = data["status"].isin([
        "In Service",
        "Under Maintenance",
        "Out of Service"
    ])

    valid_mask = (
        valid_identifier
        & valid_manufacturer
        & valid_status
    )

    valid = data.loc[valid_mask].copy()

    rejected = data.loc[~valid_mask].copy()

    # Reject duplicate identifiers.
    duplicate_mask = valid.duplicated(
        subset=["inventory_number"],
        keep="first"
    )

    duplicates = valid.loc[duplicate_mask].copy()

    valid = valid.loc[~duplicate_mask].copy()

    rejected = pd.concat(
        [rejected, duplicates],
        ignore_index=True
    )

    return valid, rejected


# --------------------------------------------------
# Task 4: Load
# --------------------------------------------------

def load_data(
    valid: pd.DataFrame,
    rejected: pd.DataFrame
) -> None:
    """Write valid and rejected records to separate files."""

    OUTPUT_DIRECTORY.mkdir(
        parents=True,
        exist_ok=True
    )

    valid.to_csv(
        VALID_OUTPUT,
        index=False
    )

    rejected.to_csv(
        REJECTED_OUTPUT,
        index=False
    )


# --------------------------------------------------
# Main workflow
# --------------------------------------------------

def main() -> None:
    """Execute the complete pipeline."""

    extracted = extract_data(INPUT_FILE)

    transformed = transform_data(extracted)

    valid, rejected = validate_data(transformed)

    load_data(valid, rejected)

    print(f"Extracted records: {len(extracted)}")
    print(f"Valid records: {len(valid)}")
    print(f"Rejected records: {len(rejected)}")

    print("Pipeline completed successfully.")


if __name__ == "__main__":
    main()
```

### 17.3 Pipeline structure

```text
vehicles.csv
     |
     v
extract_data()
     |
     v
transform_data()
     |
     v
validate_data()
     |
     +------> Valid Records
     |              |
     |              v
     |        vehicles_valid.csv
     |
     +------> Rejected Records
                    |
                    v
             vehicles_rejected.csv
```

### 17.4 Important design observations

This example demonstrates a simple pipeline, but it is not yet a production-grade implementation.

A production system should also consider:

- Structured logging.
- Automated tests.
- Configuration management.
- Database transactions.
- Error classification.
- Monitoring.
- Data lineage.
- Idempotent loading.
- Recovery procedures.
- Secure credential management.

The example also uses a simple duplicate policy: keep the first occurrence and reject subsequent occurrences. A real business process must define whether the first, latest, or authoritative record should be retained.

---

## 18. Practical Example: Orchestrating a Pipeline with Airflow

Apache Airflow allows tasks to be organized into DAGs.

The following simplified example defines a daily workflow.

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.python import PythonOperator


def extract_vehicles():
    print("Extracting vehicle records")


def validate_vehicles():
    print("Validating vehicle records")


def transform_vehicles():
    print("Transforming vehicle records")


def load_warehouse():
    print("Loading warehouse tables")


with DAG(
    dag_id="daily_vehicle_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    tags=["vehicles", "data-engineering"],
) as dag:

    extract = PythonOperator(
        task_id="extract_vehicles",
        python_callable=extract_vehicles,
    )

    validate = PythonOperator(
        task_id="validate_vehicles",
        python_callable=validate_vehicles,
    )

    transform = PythonOperator(
        task_id="transform_vehicles",
        python_callable=transform_vehicles,
    )

    load = PythonOperator(
        task_id="load_warehouse",
        python_callable=load_warehouse,
    )

    extract >> validate >> transform >> load
```

### 18.1 Understanding the code

**DAG**

Defines the workflow and its configuration.

**dag_id**

Identifies the workflow.

**start_date**

Defines the beginning of the scheduling period.

**schedule**

Defines the execution frequency.

**catchup**

Controls whether Airflow creates historical scheduled runs for periods between the start date and the current scheduling period.

**PythonOperator**

Defines a task that executes a Python callable.

**Task dependencies**

The `>>` operator establishes execution order.

The example prints messages rather than accessing a real database. Actual production tasks would contain extraction, validation, transformation, and loading logic.

---

## 19. Practical Example: SIAPV Data Pipeline Architecture

SIAPV is an internal application that manages information for approximately 200 vehicles and machinery units using PostgreSQL.

Its information includes:

- Unique inventory numbers.
- Equipment characteristics.
- Mileage.
- Operating hours.
- Maintenance.
- Incidents.
- Insurance.
- Administrative records.
- Operational reports.

A conceptual analytical pipeline could use this information to produce a consolidated fleet report.

This is an architectural learning example, not a claim that the application currently runs this automated workflow.

### 19.1 Business objective

Produce a reliable daily dataset that combines:

- Vehicle inventory.
- Latest mileage.
- Latest horometer reading.
- Operational status.
- Maintenance information.

### 19.2 Proposed architecture

```text
                 SIAPV
              PostgreSQL
                   |
                   v
             Data Extraction
                   |
                   v
              Staging Layer
                   |
          +--------+--------+
          |                 |
          v                 v
    Data Validation   Data Standardization
          |                 |
          +--------+--------+
                   |
                   v
             Data Integration
                   |
                   v
              Transformation
                   |
                   v
           Analytical PostgreSQL
                   |
                   v
             Reporting Layer
                   |
                   v
              Power BI / SQL
```

### 19.3 Task definition

| Task | Responsibility |
|---|---|
| Extract inventory | Retrieve vehicle information |
| Extract readings | Retrieve mileage and horometer records |
| Extract maintenance | Retrieve maintenance information |
| Validate identifiers | Verify inventory references |
| Standardize values | Apply consistent formats |
| Integrate records | Associate information by inventory number |
| Calculate metrics | Derive operational indicators |
| Load analytical tables | Store prepared datasets |
| Reconcile results | Compare expected and processed records |
| Publish report | Make the dataset available to consumers |

### 19.4 Scheduling

A daily batch execution may be sufficient if reports do not require real-time updates.

For example:

```text
02:00 - Extract source data
02:05 - Validate records
02:10 - Transform datasets
02:20 - Load analytical tables
02:25 - Reconcile records
02:30 - Publish report
```

These times are illustrative. Actual scheduling should be based on source availability, processing duration, and reporting requirements.

### 19.5 Quality controls

The pipeline should verify:

- Every inventory identifier is valid.
- No unexpected duplicate identifiers exist.
- Mileage and horometer values are valid.
- Dates follow the expected format.
- Maintenance records reference known units.
- Expected records have been processed.
- Analytical outputs are sufficiently recent.

### 19.6 Monitoring

A daily execution record could include:

```text
Pipeline: SIAPV Daily Fleet Report

Execution Date: 2026-11-12
Source Records: 200
Processed Records: 200
Rejected Records: 0
Execution Status: Successful
Duration: 30 seconds
```

These are illustrative values, not actual SIAPV execution metrics.

### 19.7 Architectural lesson

The pipeline turns existing operational information into a controlled analytical workflow.

The important engineering work is not merely automating queries. It is defining dependencies, validation, reliability, recovery, and the conditions under which results can be trusted.

---

## 20. Common Data Pipeline Mistakes

### 20.1 Creating one large task

A single task that extracts, transforms, loads, and publishes everything can become difficult to test and recover.

Prefer modular tasks with clear responsibilities.

### 20.2 Ignoring dependencies

Tasks may execute before their required inputs are ready.

Model dependencies explicitly.

### 20.3 Retrying permanent errors

Repeatedly retrying invalid data or unauthorized requests does not solve the underlying problem.

Classify errors before retrying.

### 20.4 Failing to design for reruns

A failed task may need to be executed again.

Use idempotent operations and controlled checkpoints.

### 20.5 Ignoring data freshness

A successful execution does not guarantee that data is current enough for the business.

Monitor freshness separately.

### 20.6 Processing without reconciliation

A pipeline may load fewer records than expected without raising an execution error.

Compare source and destination results.

### 20.7 Overengineering simple workflows

Not every data process requires Kafka, Spark, Airflow, and a distributed architecture.

Choose tools according to actual requirements.

### 20.8 Confusing orchestration with execution

An orchestrator coordinates tasks, but the actual data processing may happen in PostgreSQL, Python, Spark, or another service.

### 20.9 Ignoring resource constraints

Large transformations can exhaust memory, CPU, storage, or network capacity.

Monitor resource utilization and design processing strategies accordingly.

---

## 21. Knowledge Check

1. What is a data pipeline?
2. How does a pipeline differ from an individual ETL process?
3. What is a task?
4. What is a workflow?
5. What does a task dependency represent?
6. What is a DAG?
7. Why must a DAG be acyclic?
8. What is the difference between batch and streaming pipelines?
9. What is the difference between scheduled and event-based execution?
10. Why are retries important?
11. What makes a task idempotent?
12. What is a checkpoint?
13. What is a backfill?
14. What is the difference between logs, metrics, and traces?
15. What does data freshness measure?
16. What is the difference between an SLI, an SLO, and an SLA?
17. What is the responsibility of an orchestration platform?
18. Why should failed records be isolated?
19. Why is reconciliation necessary?
20. Why should a pipeline be designed for recovery?

---

## 22. Practical Exercises

### Exercise 1: Design a basic pipeline

An organization receives a CSV file containing vehicle information every morning.

Design a pipeline with:

- Extraction.
- Validation.
- Transformation.
- Loading.
- Monitoring.

Define the responsibility of each task.

### Exercise 2: Build a DAG

Create a workflow with these tasks:

- Extract customers.
- Extract transactions.
- Validate customers.
- Validate transactions.
- Integrate information.
- Calculate monthly totals.
- Load analytical tables.
- Publish the report.

Identify which tasks can run in parallel.

Draw the DAG.

### Exercise 3: Design a retry strategy

An API occasionally returns HTTP 503.

Define:

- Maximum retry attempts.
- Initial delay.
- Backoff strategy.
- Conditions for stopping retries.
- Failure notification.
- Recovery procedure.

Explain why retrying HTTP 400 responses indefinitely would be inappropriate.

### Exercise 4: Implement a Python pipeline

Extend the Python pipeline from Section 17.

Add:

- Logging.
- Input schema validation.
- Automated tests.
- Execution timestamps.
- A summary of rejected records.
- Error handling.

### Exercise 5: Design pipeline monitoring

Define five metrics for a daily analytical pipeline.

For each metric, explain:

- What it measures.
- How it is calculated.
- What threshold should trigger an alert.
- Who should investigate the alert.

### Exercise 6: Plan a backfill

A transformation rule was incorrect for the previous 90 days.

Design a backfill procedure.

Include:

- Date range.
- Source data.
- Target tables.
- Reprocessing method.
- Duplicate prevention.
- Validation.
- Publication controls.

### Exercise 7: SIAPV pipeline design

Design a complete analytical pipeline using SIAPV information.

Your architecture must include:

- PostgreSQL as an operational source.
- Extraction tasks.
- Staging.
- Data quality validation.
- Transformations.
- Analytical storage.
- Scheduling.
- Orchestration.
- Logging.
- Monitoring.
- Recovery.
- Reconciliation.

Explain which components could be implemented with SQL and Python before introducing a dedicated orchestration platform.

---

## 23. Glossary

| Term | Definition |
|---|---|
| Data Pipeline | Automated or controlled workflow that moves and processes data |
| Task | Individual unit of work in a pipeline |
| Workflow | Organized sequence or graph of tasks |
| Dependency | Relationship defining which task must complete before another |
| DAG | Directed Acyclic Graph representing workflow dependencies |
| Orchestration | Coordination of workflow execution |
| Scheduler | Component that determines when workflows execute |
| Batch Processing | Processing data in groups |
| Stream Processing | Processing continuously arriving events |
| Event Trigger | Event that initiates a workflow or task |
| Retry | Repeated execution after a failure |
| Exponential Backoff | Retry strategy with progressively longer delays |
| Idempotency | Property that makes repeated execution safe |
| Checkpoint | Stored processing position used for recovery |
| Backfill | Reprocessing of historical data |
| Reconciliation | Comparison of source and target information |
| Observability | Ability to understand system behavior using operational evidence |
| Log | Record of an event or execution detail |
| Metric | Numerical measurement of system behavior |
| Trace | Record of an operation across multiple components |
| Data Freshness | How recently a dataset was updated |
| Throughput | Amount of data processed per unit of time |
| SLI | Service-Level Indicator |
| SLO | Service-Level Objective |
| SLA | Service-Level Agreement |
| Dead-Letter Queue | Repository for messages that cannot be processed successfully |
| Fan-out | Distribution of work into multiple downstream tasks |
| Fan-in | Combination of multiple upstream tasks |
| Micro-batch | Processing of small groups of events at short intervals |
| Data Lineage | Record of data origin and transformations |

---

## 24. Connections to Other Topics

### Previous notes

- [[01 - Data Architecture Overview]]
- [[02 - Data Integration]]
- [[03 - ETL and ELT]]
- [[02 - The Data Lifecycle]]

### Related notes

- [[05 - Data Warehouses]]
- [[06 - Data Lakes]]
- [[07 - Lakehouse Architecture]]
- [[03 - Data Quality]]
- [[04 - Data Governance]]
- [[01 - PostgreSQL Architecture Overview]]
- [[02 - PostgreSQL Processes]]

### Tools and technologies

- Apache Airflow
- Apache Kafka
- Apache Spark
- dbt
- Python
- Pandas
- PostgreSQL
- Docker

### Practical applications

- [[01 - SIAPV Data Architecture]]
- [[02 - SIAPV Database Design Review]]

---

## 25. Key Takeaways

1. A data pipeline is a structured workflow for moving and processing information.
2. Pipelines can implement ETL, ELT, or combinations of both.
3. Tasks represent individual operations, while workflows organize their execution.
4. DAGs model dependencies and allow valid parallel execution.
5. Batch pipelines process groups of records, while streaming pipelines process continuous events.
6. Pipelines can execute according to schedules, events, or manual triggers.
7. Reliability requires retries, checkpoints, idempotency, and controlled recovery.
8. Backfills allow historical data to be reprocessed.
9. Observability uses logs, metrics, and traces to understand pipeline behavior.
10. Data freshness and quality must be monitored independently of task execution status.
11. Orchestration platforms coordinate execution but do not replace the underlying processing engines.
12. A good pipeline is modular, testable, secure, observable, and recoverable.

**Next topic:** [[05 - Data Warehouses]]

The next note will explain the architecture of data warehouses, including analytical modeling, fact tables, dimension tables, star schemas, snowflake schemas, and the differences between operational databases and analytical repositories.
