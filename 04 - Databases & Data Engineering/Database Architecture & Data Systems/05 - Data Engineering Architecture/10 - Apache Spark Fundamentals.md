
---
title: "Apache Spark Fundamentals"
aliases:
  - "Apache Spark Architecture"
  - "Spark Fundamentals"
  - "Distributed Data Processing with Spark"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: apache-spark
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - apache-spark
  - distributed-computing
  - data-engineering
  - big-data
  - python
  - sql
  - data-processing
  - dataframes
concepts:
  - apache-spark
  - spark-architecture
  - driver
  - executor
  - cluster-manager
  - spark-session
  - spark-context
  - resilient-distributed-dataset
  - dataframe
  - transformation
  - action
  - lazy-evaluation
  - directed-acyclic-graph
  - job
  - stage
  - task
  - partition
  - shuffle
  - spark-sql
prerequisites:
  - "[[04 - Data Pipelines]]"
  - "[[07 - Lakehouse Architecture]]"
  - "[[08 - Cloud Data Architecture]]"
  - "[[09 - Distributed Data Processing]]"
related:
  - "[[11 - Apache Spark DataFrames]]"
  - "[[12 - Spark SQL]]"
  - "[[Distributed Systems]]"
  - "[[Data Partitioning]]"
  - "[[03 - ETL and ELT]]"
  - "[[Data Quality]]"
tools:
  - Apache Spark
  - PySpark
  - Python
  - SQL
  - Jupyter
  - Docker
  - Parquet
  - PostgreSQL
learning_objectives:
  - "Define Apache Spark and identify its main use cases."
  - "Explain the main components of Spark architecture."
  - "Differentiate the driver, executors, and cluster manager."
  - "Understand Spark applications, jobs, stages, and tasks."
  - "Explain transformations and actions."
  - "Understand lazy evaluation."
  - "Explain how Spark builds execution plans using DAGs."
  - "Understand partitions and parallel execution."
  - "Recognize the role of Spark SQL and DataFrames."
  - "Understand when Spark is appropriate and when it is unnecessary."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Apache Spark Fundamentals

## 1. Introduction

**Apache Spark** is an open-source distributed computing framework designed to process large datasets across multiple computing resources.

It provides tools for:

- Large-scale data processing.
- Data transformation.
- SQL analytics.
- Batch processing.
- Stream processing.
- Machine learning workflows.

Spark allows developers to write processing logic while the framework manages much of the distributed execution.

A simplified representation:

```text
                 APACHE SPARK
                       |
          +------------+------------+
          |            |            |
          v            v            v
       Batch         SQL         Streaming
     Processing   Analytics     Processing
          |            |            |
          +------------+------------+
                       |
                       v
               Distributed Data
```

Spark is particularly relevant to Data Engineering because it can process data from different sources and transform it into analytical datasets.

---

## 2. Why Was Apache Spark Created?

Traditional data processing frequently relied on a single machine or systems with limited processing capabilities.

Consider a dataset containing millions of maintenance records.

A simple Python script might process it:

```python
import pandas as pd

df = pd.read_csv("maintenance.csv")

result = (
    df.groupby("equipment_type")["cost"]
    .sum()
)

print(result)
```

For a sufficiently small dataset, this approach is practical.

However, as data volume increases, a single machine may become constrained by:

- Available RAM.
- CPU capacity.
- Processing time.
- Storage performance.

Spark addresses this problem by distributing computation.

```text
              LARGE DATASET
                    |
                    v
             APACHE SPARK
                    |
          +---------+---------+
          |         |         |
          v         v         v
        Node 1    Node 2    Node 3
          |         |         |
          +---------+---------+
                    |
                    v
               Final Result
```

The central idea is to divide work into smaller tasks and execute them across available computing resources.

---

## 3. Apache Spark Is a Processing Engine

An important architectural distinction:

**Spark is not a database.**

Spark primarily provides computation and data processing capabilities.

It can read data from different storage systems, transform it, and write results elsewhere.

For example:

```text
PostgreSQL
    |
    v
Apache Spark
    |
    v
Data Transformation
    |
    v
Parquet / Data Lake
    |
    v
BI / Analytics
```

Spark does not necessarily own the persistent storage of the datasets it processes.

It can work with:

- Relational databases.
- Object storage.
- Data Lakes.
- Parquet files.
- CSV files.
- JSON files.
- Distributed storage systems.

This distinction is important when designing a data architecture.

---

## 4. Main Components of Spark Architecture

A Spark application commonly involves three central components:

1. Driver.
2. Executors.
3. Cluster Manager.

```text
                  SPARK APPLICATION
                         |
                         v
                  +-------------+
                  |   DRIVER    |
                  |             |
                  | Coordinates |
                  | Execution   |
                  +------+------+
                         |
                         v
                  CLUSTER MANAGER
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
        +---------+ +---------+ +---------+
        |Executor | |Executor | |Executor |
        |    1    | |    2    | |    3    |
        +---------+ +---------+ +---------+
             |           |           |
             v           v           v
          Tasks       Tasks       Tasks
```

Each component has a different responsibility.

---

## 5. The Driver

The **Driver** is the process responsible for coordinating a Spark application.

Its responsibilities include:

- Maintaining application information.
- Creating the execution plan.
- Coordinating jobs.
- Scheduling tasks.
- Tracking execution progress.
- Collecting results when requested.

The Driver is the central coordinator of the Spark application.

Conceptually:

```text
User Code
    |
    v
Driver
    |
    +--> Analyze operations
    +--> Build execution plan
    +--> Schedule tasks
    +--> Coordinate executors
    +--> Track results
```

A Driver is not the same as a worker.

It coordinates the work performed by executors.

---

## 6. Executors

An **Executor** is a process that runs on a worker node and executes tasks assigned by the Driver.

Executors can:

- Process data partitions.
- Execute transformations.
- Store intermediate results in memory or disk.
- Return results and execution information.

Conceptually:

```text
             DRIVER
                |
       +--------+--------+
       |        |        |
       v        v        v
    Executor Executor Executor
       1        2        3
       |        |        |
       v        v        v
    Tasks    Tasks    Tasks
```

Executors are responsible for much of the actual distributed computation.

An application can use multiple executors depending on its deployment configuration and available resources.

---

## 7. Cluster Manager

The **Cluster Manager** is responsible for allocating resources to Spark applications.

It helps determine where the application's executors can run.

Examples of supported resource-management environments include:

- Standalone Cluster Manager.
- Apache Hadoop YARN.
- Kubernetes.

Conceptually:

```text
Spark Application
       |
       v
Cluster Manager
       |
       +--> Allocate CPU
       +--> Allocate Memory
       +--> Launch Executors
```

The Cluster Manager manages infrastructure resources.

The Driver coordinates the execution of the application.

These are related but different responsibilities.

---

## 8. Driver vs. Executor vs. Cluster Manager

| Component | Main Responsibility |
|---|---|
| Driver | Coordinates application execution |
| Executor | Executes tasks and processes data |
| Cluster Manager | Allocates computing resources |

A useful way to remember the distinction:

- The Driver organizes the work.
- The Cluster Manager provides resources.
- The Executors perform the work.

---

## 9. Spark Application

A **Spark Application** is a program that uses Spark to process data.

For example:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("FleetMaintenanceAnalysis")
    .getOrCreate()
)
```

This code creates or retrieves a SparkSession.

A Spark application may contain many operations, which Spark organizes into jobs and tasks.

---

## 10. SparkSession

The **SparkSession** is the main entry point for working with Spark's DataFrame and SQL APIs.

It provides access to:

- DataFrame operations.
- SQL queries.
- Data reading.
- Data writing.
- Runtime configuration.

Example:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("FleetAnalytics")
    .master("local[*]")
    .getOrCreate()
)

print(spark.version)
```

Important elements:

| Element | Meaning |
|---|---|
| `SparkSession` | Main interface for Spark SQL and DataFrames |
| `builder` | Configures the session |
| `appName()` | Defines the application name |
| `master()` | Specifies the execution environment |
| `getOrCreate()` | Gets an existing session or creates one |

The configuration `local[*]` means Spark runs locally using the available logical CPU cores.

It does not create a distributed cluster across multiple computers.

---

## 11. SparkContext

Historically, Spark applications interacted with Spark through **SparkContext**.

SparkContext represents the connection to the Spark execution environment and provides lower-level access to distributed processing functionality.

In modern Spark applications, SparkSession is generally the preferred entry point.

Conceptually:

```text
SparkSession
     |
     v
SparkContext
     |
     v
Spark Execution Environment
```

You may encounter both concepts when reading Spark documentation or existing code.

---

## 12. Jobs, Stages, and Tasks

Spark organizes execution into a hierarchy.

```text
SPARK APPLICATION
       |
       v
      JOB
       |
       v
     STAGES
       |
       v
     TASKS
       |
       v
   PARTITIONS
```

These concepts describe different levels of execution.

### Application

The complete Spark program.

### Job

A unit of execution triggered by an action.

### Stage

A group of tasks that can execute without crossing a shuffle boundary.

### Task

The smallest scheduled unit of work in a Spark execution stage, typically operating on one partition.

---

## 13. Understanding the Hierarchy

Imagine a Spark application that reads maintenance records, groups them by equipment type, and calculates total maintenance costs.

```text
Application
    |
    v
Read Data
    |
    v
Transform Data
    |
    v
Group by Equipment Type
    |
    v
Calculate Total Cost
    |
    v
Write Result
```

An action may trigger execution of the required transformations.

Spark then organizes the work into jobs, stages, and tasks.

The exact number of jobs and stages depends on the execution plan.

Do not assume that every Python statement creates a separate Spark job.

---

## 14. Transformations and Actions

One of the most important Spark concepts is the distinction between **transformations** and **actions**.

### Transformations

Transformations describe operations that produce a new DataFrame or RDD.

Examples:

- `select()`
- `filter()`
- `withColumn()`
- `groupBy()`
- `join()`

Example:

```python
maintenance_clean = maintenance.filter(
    maintenance["cost"] > 0
)
```

This describes a transformation.

It does not necessarily execute the complete computation immediately.

### Actions

Actions trigger computation and return results or produce output.

Examples:

- `show()`
- `count()`
- `collect()`
- `first()`
- `write()`

Example:

```python
maintenance_clean.show()
```

This requests results and triggers the required execution.

---

## 15. Lazy Evaluation

Spark uses **lazy evaluation** for many DataFrame and RDD transformations.

This means that Spark can delay executing transformations until an action requires a result.

Example:

```python
df = spark.read.csv(
    "maintenance.csv",
    header=True,
    inferSchema=True
)

step_1 = df.filter("cost > 0")

step_2 = step_1.select(
    "vehicle_id",
    "cost"
)

step_3 = step_2.groupBy(
    "vehicle_id"
).sum("cost")

step_3.show()
```

The transformations describe what should happen.

The action `show()` requests the result.

Spark can optimize the execution plan before performing the computation.

### Why Is Lazy Evaluation Useful?

It allows Spark to:

- Analyze multiple operations together.
- Optimize execution plans.
- Avoid unnecessary computation.
- Reduce data movement in suitable cases.

A useful principle:

> Spark can optimize a sequence of transformations before executing the required work.

---

## 16. Directed Acyclic Graph (DAG)

Spark represents execution dependencies using a **Directed Acyclic Graph**, commonly abbreviated as DAG.

A DAG describes operations and their dependencies.

- Directed: operations have a defined direction.
- Acyclic: dependencies do not form cycles.

Example:

```text
Read Data
    |
    v
Filter Invalid Records
    |
    v
Select Columns
    |
    v
Group by Vehicle
    |
    v
Calculate Total Cost
    |
    v
Write Result
```

A more complex DAG can contain parallel branches:

```text
              Read Data
                  |
                  v
             Clean Records
                  |
          +-------+-------+
          |               |
          v               v
     Calculate Cost   Calculate Downtime
          |               |
          v               v
          +-------+-------+
                  |
                  v
             Final Dataset
```

The DAG allows Spark to understand dependencies and organize execution.

---

## 17. Narrow and Wide Transformations

Transformations can be understood in terms of how they use partitions.

### Narrow Transformations

A narrow transformation can process data without requiring a broad redistribution of records between partitions.

Examples commonly include:

- `filter()`
- `select()`
- `withColumn()`

Conceptually:

```text
Partition 1 → Output Partition 1
Partition 2 → Output Partition 2
Partition 3 → Output Partition 3
```

Each output partition depends on a limited set of input partitions.

### Wide Transformations

A wide transformation requires data from multiple input partitions to be redistributed or combined.

Examples include:

- `groupBy()`
- `distinct()`
- Some joins.
- `repartition()`.

Conceptually:

```text
Input Partitions
   |      |      |
   v      v      v
  Data Redistribution
          |
          v
  Output Partitions
```

Wide transformations frequently involve a shuffle.

---

## 18. What Is a Shuffle?

A **shuffle** is the redistribution of data across partitions, often involving network communication between executors.

Example:

Suppose records are distributed by input partition:

```text
Partition 1:
Vehicle A
Vehicle B

Partition 2:
Vehicle A
Vehicle C

Partition 3:
Vehicle B
Vehicle C
```

We want to group all records by vehicle.

Spark may redistribute records so that records belonging to the same grouping key can be processed together.

```text
Before Shuffle:

P1       P2       P3
A, B     A, C     B, C

          |
          v

       SHUFFLE

          |
     +----+----+
     |    |    |
     v    v    v

     A    B    C
```

Shuffles can be expensive because they may involve:

- Network transfers.
- Disk operations.
- Serialization.
- Coordination between executors.

Understanding shuffles is essential for Spark performance optimization.

---

## 19. Partitions in Spark

A **partition** is a logical division of a dataset.

Spark uses partitions as units of parallel processing.

Example:

```text
Dataset
   |
   +---------+---------+---------+
   |         |         |         |
   v         v         v         v
Part 1    Part 2    Part 3    Part 4
```

Tasks operate on partitions.

The number and distribution of partitions affect execution parallelism.

### Important Distinction

A partition is not necessarily:

- A separate computer.
- A separate file.
- A separate database table.

It is a logical unit of distributed data processing.

---

## 20. Parallel Execution

Spark can execute tasks on multiple executors simultaneously.

Example:

```text
                 DRIVER
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Executor  Executor  Executor
           1         2         3
          |         |         |
          v         v         v
        Task 1    Task 2    Task 3
          |         |         |
          +---------+---------+
                    |
                    v
                 Results
```

The available parallelism depends on:

- Number of partitions.
- Available executor resources.
- CPU cores.
- Scheduling constraints.
- Data dependencies.

Having more executors does not guarantee better performance if the workload cannot use them efficiently.

---

## 21. Resilient Distributed Datasets (RDDs)

An **RDD** is a fundamental Spark abstraction representing an immutable distributed collection of elements.

RDD stands for:

**Resilient Distributed Dataset**

Its important characteristics include:

- Distributed data.
- Partition-based processing.
- Immutable transformations.
- Fault recovery through lineage.
- Functional operations.

Example:

```python
numbers = spark.sparkContext.parallelize(
    [1, 2, 3, 4, 5, 6]
)

squared = numbers.map(
    lambda number: number ** 2
)

print(squared.collect())
```

Expected output:

```text
[1, 4, 9, 16, 25, 36]
```

RDDs provide lower-level control over distributed collections.

However, for most structured data engineering tasks, DataFrames are generally preferred because Spark can optimize their execution.

---

## 22. DataFrames

A **Spark DataFrame** is a distributed dataset organized into named columns.

It resembles a relational table.

Example:

| vehicle_id | equipment_type | maintenance_cost |
|---|---|---:|
| 1001 | Excavator | 3500.00 |
| 1002 | Grader | 4200.00 |
| 1003 | Excavator | 1800.00 |

A DataFrame provides operations for:

- Selecting columns.
- Filtering records.
- Grouping data.
- Joining datasets.
- Aggregating values.
- Writing results.

Example:

```python
maintenance.select(
    "vehicle_id",
    "maintenance_cost"
)
```

DataFrames are one of the most important interfaces for modern Spark development.

We will study them in detail in a dedicated note.

---

## 23. Spark SQL

Spark SQL allows users to query structured data using SQL.

Example:

```python
maintenance.createOrReplaceTempView(
    "maintenance"
)

result = spark.sql("""
    SELECT
        equipment_type,
        SUM(maintenance_cost) AS total_cost
    FROM maintenance
    GROUP BY equipment_type
""")

result.show()
```

This is especially useful for people who already understand relational databases and SQL.

Spark SQL connects distributed processing with familiar analytical query concepts.

---

## 24. Data Sources and Formats

Spark can read and write many data formats.

Common examples:

| Format | Typical Use |
|---|---|
| CSV | Simple tabular exchange |
| JSON | Semi-structured data |
| Parquet | Analytical columnar storage |
| ORC | Columnar analytical storage |
| Avro | Serialization and data exchange |
| JDBC | Relational database connectivity |

Example reading Parquet:

```python
df = spark.read.parquet(
    "data/maintenance/"
)
```

Example writing Parquet:

```python
df.write.mode("overwrite").parquet(
    "output/maintenance/"
)
```

Parquet is particularly common in Data Engineering because it supports columnar storage and efficient analytical access.

---

## 25. Spark and Fault Tolerance

Spark can recover from certain failures using information about the transformations that produced distributed datasets.

This dependency information is known as **lineage**.

Conceptually:

```text
Original Data
     |
     v
Transformation 1
     |
     v
Transformation 2
     |
     v
Transformation 3
```

If a partition is lost, Spark may be able to recompute it from the original data and its transformations.

This is one reason RDDs are described as resilient.

Fault recovery still depends on the storage system, application design, checkpointing strategy, and type of failure.

---

## 26. Spark and Checkpointing

Checkpointing saves data or processing state to reliable storage to reduce dependence on long transformation histories.

Conceptually:

```text
Original Data
     |
     v
Transformation 1
     |
     v
Transformation 2
     |
     v
  CHECKPOINT
     |
     v
Transformation 3
```

Checkpointing can be useful when:

- Lineage becomes very long.
- Recovery would otherwise require expensive recomputation.
- Stateful streaming workloads need recovery support.

Checkpointing is different from simply caching data in memory.

---

## 27. Spark Ecosystem

Spark provides several components for different processing needs.

| Component | Purpose |
|---|---|
| Spark Core | Distributed execution foundation |
| Spark SQL | Structured data and SQL |
| Structured Streaming | Stream processing |
| MLlib | Machine learning |
| GraphX | Graph processing |

These components share the Spark execution foundation.

Not every application needs every component.

---

## 28. PySpark

**PySpark** is the Python API for Apache Spark.

It allows Python developers to use Spark's distributed processing capabilities.

Example:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("FleetDataProcessing")
    .master("local[*]")
    .getOrCreate()
)

data = [
    (1001, "Excavator", 3500.0),
    (1002, "Grader", 4200.0),
    (1003, "Excavator", 1800.0)
]

columns = [
    "vehicle_id",
    "equipment_type",
    "maintenance_cost"
]

df = spark.createDataFrame(
    data,
    columns
)

df.show()
```

Expected output:

```text
+----------+--------------+----------------+
|vehicle_id|equipment_type|maintenance_cost|
+----------+--------------+----------------+
|      1001|     Excavator|          3500.0|
|      1002|        Grader|          4200.0|
|      1003|     Excavator|          1800.0|
+----------+--------------+----------------+
```

This example uses local execution.

The same DataFrame API can be used in a properly configured Spark cluster.

---

## 29. Spark vs. Pandas

Both Pandas and Spark can process structured data, but they have different architectural characteristics.

| Characteristic | Pandas | Apache Spark |
|---|---|---|
| Main execution model | Single-process DataFrame operations | Distributed execution |
| Typical environment | One machine | One machine or cluster |
| Parallelism | Limited by the selected execution approach | Built around distributed task execution |
| Large-scale processing | Limited by machine resources | Designed for distributed workloads |
| SQL integration | Through additional tools or libraries | Spark SQL |
| Execution | Generally eager | Many transformations are lazy |

Pandas remains extremely useful.

Spark should not automatically replace it.

A sensible progression is:

1. Use Python and Pandas for local analysis.
2. Understand data volume and processing requirements.
3. Introduce Spark when distributed processing provides a real benefit.

---

## 30. When Should You Use Apache Spark?

Spark can be appropriate when:

- Datasets are too large for efficient single-machine processing.
- Distributed transformations are required.
- Large ETL pipelines must be executed.
- Data Lakes or Lakehouses need large-scale processing.
- Multiple partitions can be processed in parallel.

Spark may be unnecessary when:

- Data fits comfortably in memory.
- A SQL query can solve the problem efficiently.
- A small Python script is sufficient.
- Distributed coordination costs more than the processing itself.

The objective is not to use the most complex tool.

The objective is to solve the workload efficiently.

---

## 31. Practical Example: Fleet Maintenance Analytics

Consider an organization that stores maintenance events for its equipment.

The input dataset contains:

- Vehicle inventory number.
- Equipment type.
- Maintenance date.
- Maintenance cost.
- Downtime hours.

A simplified Spark workflow:

```text
Source Files / Database
          |
          v
      Spark Read
          |
          v
    Data Validation
          |
          v
   Data Transformation
          |
          v
     Aggregations
          |
          v
    Curated Dataset
          |
          v
    BI / Analytics
```

Example:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import (
    col,
    sum,
    avg,
    count
)

spark = (
    SparkSession.builder
    .appName("FleetMaintenanceAnalytics")
    .master("local[*]")
    .getOrCreate()
)

data = [
    (1001, "Excavator", 3500.0, 12.0),
    (1002, "Grader", 4200.0, 18.0),
    (1003, "Excavator", 1800.0, 6.0),
    (1004, "Backhoe", 2500.0, 9.0)
]

columns = [
    "vehicle_id",
    "equipment_type",
    "maintenance_cost",
    "downtime_hours"
]

maintenance = spark.createDataFrame(
    data,
    columns
)

valid_records = maintenance.filter(
    (col("maintenance_cost") >= 0) &
    (col("downtime_hours") >= 0)
)

summary = (
    valid_records
    .groupBy("equipment_type")
    .agg(
        count("*").alias("maintenance_events"),
        sum("maintenance_cost").alias("total_cost"),
        avg("downtime_hours").alias("average_downtime")
    )
)

summary.show()
```

The example demonstrates:

- Creating a SparkSession.
- Creating a DataFrame.
- Validating records.
- Grouping data.
- Calculating aggregations.

The sample data is illustrative, not an actual SIAPV database export.

---

## 32. Common Performance Mistakes

### Collecting Too Much Data

The `collect()` action transfers results to the Driver.

If the dataset is large, this can exhaust Driver memory.

Avoid:

```python
all_records = df.collect()
```

unless the result is known to be small.

### Excessive Shuffling

Operations such as grouping and joins can trigger expensive data redistribution.

### Poor Partitioning

Too few partitions can limit parallelism.

Too many tiny partitions can create scheduling and processing overhead.

### Unnecessary Caching

Caching data that is never reused can waste memory and resources.

### Using Spark for Small Workloads

A distributed engine introduces coordination overhead.

A small dataset may be processed more efficiently with Pandas or SQL.

---

## 33. Practical Exercises

### Exercise 1: Identify Spark Components

Explain the responsibilities of:

- Driver.
- Executor.
- Cluster Manager.
- SparkSession.
- SparkContext.

Draw their relationships.

### Exercise 2: Transformations vs. Actions

Classify the following operations:

- `select()`
- `filter()`
- `groupBy()`
- `show()`
- `count()`
- `collect()`
- `write()`

Explain which operations trigger execution.

### Exercise 3: Execution Hierarchy

Describe the relationship between:

- Application.
- Job.
- Stage.
- Task.
- Partition.

Create a diagram showing how they relate.

### Exercise 4: Identify a Shuffle

Consider this operation:

```python
df.groupBy("equipment_type").count()
```

Explain why Spark may need to redistribute records between partitions.

### Exercise 5: Choose the Right Tool

Select an appropriate tool for each situation:

1. Analyze 5,000 rows in a CSV file.
2. Process several billion historical records.
3. Query a relational database with SQL.
4. Build a distributed ETL pipeline.

Justify each choice.

### Exercise 6: Build a Small PySpark Application

Create a SparkSession and a DataFrame containing:

- `vehicle_id`
- `equipment_type`
- `maintenance_cost`
- `downtime_hours`

Then:

1. Filter invalid costs.
2. Group by equipment type.
3. Calculate total cost.
4. Calculate average downtime.
5. Display the results.

---

## 34. Knowledge Check

1. What is Apache Spark?
2. Why was Spark developed?
3. Is Spark a database?
4. What is the responsibility of the Driver?
5. What do Executors do?
6. What does a Cluster Manager provide?
7. What is a Spark Application?
8. What is SparkSession?
9. What is SparkContext?
10. What is the difference between a job and a stage?
11. What is a task?
12. What is a partition?
13. What is the difference between a transformation and an action?
14. What is lazy evaluation?
15. What is a DAG?
16. What is the difference between narrow and wide transformations?
17. What is a shuffle?
18. What is an RDD?
19. What is a DataFrame?
20. What is Spark SQL?
21. What is PySpark?
22. How does Spark differ from Pandas?
23. What is lineage?
24. What is checkpointing?
25. When should Spark not be used?

---

## 35. Glossary

| Term | Definition |
|---|---|
| Apache Spark | Distributed data processing framework |
| Driver | Process coordinating a Spark application |
| Executor | Process executing tasks on worker nodes |
| Cluster Manager | System allocating cluster resources |
| SparkSession | Main interface for Spark SQL and DataFrames |
| SparkContext | Lower-level connection to the Spark execution environment |
| Application | Complete Spark program |
| Job | Unit of execution triggered by an action |
| Stage | Group of tasks separated by shuffle dependencies |
| Task | Smallest scheduled unit of Spark execution |
| Partition | Logical division of a distributed dataset |
| Transformation | Operation describing a new dataset |
| Action | Operation that triggers execution or requests output |
| Lazy Evaluation | Delayed execution of transformations |
| DAG | Directed Acyclic Graph describing dependencies |
| Shuffle | Redistribution of data between partitions |
| RDD | Resilient Distributed Dataset |
| DataFrame | Distributed dataset organized into named columns |
| Spark SQL | Spark component for structured data and SQL |
| PySpark | Python API for Apache Spark |
| Lineage | Information describing how a dataset was derived |
| Checkpoint | Persisted recovery point for data or processing state |

---

## 36. Connections to Other Topics

### Previous Notes

- [[04 - Data Pipelines]]
- [[07 - Lakehouse Architecture]]
- [[08 - Cloud Data Architecture]]
- [[09 - Distributed Data Processing]]

### Next Notes

- [[11 - Apache Spark DataFrames]]
- [[12 - Spark SQL]]
- [[13 - Spark Transformations and Actions]]
- [[Spark Execution Model]]
- [[Spark Performance Optimization]]

### Related Concepts

- [[Distributed Systems]]
- [[Data Partitioning]]
- [[Parallel Processing]]
- [[03 - ETL and ELT]]
- [[Data Quality]]
- [[06 - Data Lakes]]

### Practical Applications

- [[Fleet Data Analytics]]
- [[Large-Scale ETL Pipelines]]
- [[SIAPV Data Architecture]]

---

## 37. Key Takeaways

1. Apache Spark is a distributed data processing framework, not a database.
2. Spark can process data using one machine or a distributed cluster.
3. The Driver coordinates application execution.
4. Executors perform distributed processing tasks.
5. The Cluster Manager allocates computing resources.
6. SparkSession is the main entry point for DataFrame and SQL operations.
7. Spark organizes execution into applications, jobs, stages, and tasks.
8. Transformations describe operations on data.
9. Actions trigger computation or request results.
10. Lazy evaluation allows Spark to optimize execution plans.
11. DAGs describe dependencies between operations.
12. Narrow transformations generally avoid broad data redistribution.
13. Wide transformations can require shuffling.
14. Partitions provide units of parallel processing.
15. RDDs represent immutable distributed collections.
16. DataFrames provide a structured interface for distributed data.
17. Spark SQL enables SQL-based analytical processing.
18. PySpark makes Spark accessible through Python.
19. Lineage supports recovery from certain failures.
20. Spark is most valuable when workload requirements justify distributed processing.

> [!summary]
> **Apache Spark** is a distributed processing framework that organizes computation across available resources. Its architecture is built around the **Driver, Executors, and Cluster Manager**. Its execution model uses **transformations, actions, lazy evaluation, DAGs, stages, tasks, and partitions**. These concepts provide the foundation for developing scalable Data Engineering pipelines with Python and SQL.
