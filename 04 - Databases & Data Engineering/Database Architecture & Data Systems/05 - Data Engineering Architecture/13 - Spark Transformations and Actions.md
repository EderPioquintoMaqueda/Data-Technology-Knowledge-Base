
---
title: "Spark Transformations and Actions"
aliases:
  - "Spark Execution Model"
  - "Transformations vs Actions in Apache Spark"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: spark-transformations-actions
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - apache-spark
  - pyspark
  - transformations
  - actions
  - data-engineering
  - distributed-processing
  - lazy-evaluation
  - dag
concepts:
  - transformation
  - action
  - lazy-evaluation
  - DAG
  - job
  - stage
  - task
  - narrow-transformation
  - wide-transformation
  - shuffle
  - partition
  - caching
  - checkpointing
  - lineage
prerequisites:
  - "[[10 - Apache Spark Fundamentals]]"
  - "[[11 - Apache Spark DataFrames]]"
  - "[[12 - Spark SQL]]"
related:
  - "[[09 - Distributed Data Processing]]"
  - "[[04 - Data Pipelines]]"
  - "[[14 - Spark Joins and Aggregations]]"
  - "[[15 - Spark Performance Optimization]]"
tools:
  - Apache Spark
  - PySpark
  - Python
  - SQL
  - Parquet
learning_objectives:
  - "Distinguish transformations from actions."
  - "Explain lazy evaluation and query plan construction."
  - "Understand narrow and wide dependencies."
  - "Explain how Spark organizes jobs, stages, and tasks."
  - "Recognize operations that cause shuffles."
  - "Manage partitions and parallelism."
  - "Use cache, persist, and checkpoint appropriately."
  - "Understand lineage-based fault tolerance."
  - "Apply safe practices for actions and external side effects."
  - "Build and inspect a complete Spark processing workflow."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Spark Transformations and Actions

## 1. Introduction

Apache Spark processes data through a sequence of operations. Some operations describe how data should be transformed, while others request a result or produce an external output.

These two categories are:

- **Transformations:** Create a new DataFrame or RDD from an existing distributed dataset.
- **Actions:** Trigger computation or request a result from a distributed dataset.

Understanding this distinction is essential because Spark uses lazy evaluation and distributed execution planning.

For example:

```python
filtered_df = df.filter(df["status"] == "ACTIVE")
```

This statement defines a transformation. It does not necessarily read and process every record immediately.

By contrast:

```python
filtered_df.show()
```

This is an action. It requests results and causes Spark to execute the necessary computation.

The difference affects performance, memory consumption, parallelism, fault tolerance, and application design.

## 2. Transformations

### 2.1 Definition

A transformation produces a new distributed dataset from an existing one.

Conceptually:

```text
Input Dataset
      |
      v
Transformation
      |
      v
Output Dataset
```

Transformations do not generally modify the original DataFrame or RDD. They describe a new logical result.

Example:

```python
active_equipment = equipment_df.filter(
    equipment_df["status"] == "ACTIVE"
)
```

The original `equipment_df` remains unchanged. The variable `active_equipment` references a new DataFrame representing the filtered result.

### 2.2 Transformation characteristics

- They describe data-processing operations.
- They return a new DataFrame or RDD.
- They are generally evaluated lazily.
- They can be combined into a larger execution plan.
- They may operate within partitions or require communication between partitions.
- They can be optimized before execution.

### 2.3 Common DataFrame transformations

| Transformation | Purpose |
|---|---|
| `select()` | Select columns or expressions |
| `filter()` / `where()` | Keep rows that satisfy a condition |
| `withColumn()` | Create or replace a column |
| `drop()` | Remove columns |
| `distinct()` | Remove duplicate rows |
| `dropDuplicates()` | Remove duplicates based on selected columns |
| `union()` | Combine compatible DataFrames by column position |
| `unionByName()` | Combine DataFrames by column name |
| `join()` | Combine rows using a join condition |
| `groupBy()` | Define grouping columns |
| `agg()` | Define aggregate calculations |
| `orderBy()` | Sort rows |
| `limit()` | Restrict the number of returned rows |
| `repartition()` | Redistribute data into partitions |
| `coalesce()` | Reduce the number of partitions, usually without a full shuffle |
| `cache()` | Mark a DataFrame for caching |
| `persist()` | Mark a DataFrame for persistence using a selected storage level |

Example:

```python
from pyspark.sql import functions as F

processed_df = (
    equipment_df
    .select(
        "inventory_number",
        "equipment_type",
        "status",
        "operating_hours"
    )
    .filter(F.col("status") == "ACTIVE")
    .withColumn(
        "operating_hours",
        F.col("operating_hours").cast("double")
    )
)
```

This pipeline defines several transformations. Spark can analyze and optimize the combined plan before executing it.

## 3. Actions

### 3.1 Definition

An action requests a result from a distributed dataset or sends its contents to an external destination.

Conceptually:

```text
Dataset
   |
   v
Action
   |
   v
Execution
   |
   v
Result or External Output
```

Actions are important because they generally trigger the execution of the transformations required to produce their results.

### 3.2 Common DataFrame actions

| Action | Purpose |
|---|---|
| `show()` | Display a limited number of rows |
| `count()` | Count the number of rows |
| `collect()` | Return all rows to the driver |
| `take(n)` | Return up to n rows to the driver |
| `first()` | Return the first row |
| `head()` | Return the first row or a limited number of rows |
| `foreach()` | Apply a function to each row |
| `write()` | Write the DataFrame to an external destination |

Examples:

```python
# Display a sample
processed_df.show(10)

# Count rows
total_rows = processed_df.count()

# Retrieve a limited sample
sample_rows = processed_df.take(5)

# Retrieve the first row
first_row = processed_df.first()

# Write results
processed_df.write.mode("overwrite").parquet(
    "output/processed_equipment"
)
```

### 3.3 The danger of collect()

The `collect()` method transfers all rows from the distributed workers to the driver.

```python
all_rows = processed_df.collect()
```

This can be dangerous when the dataset is large.

Suppose a DataFrame contains several million records. The distributed workers may have enough combined memory to process the data, but the driver may not have enough memory to hold the complete result.

Potential consequences include:

- Driver memory exhaustion.
- Application failure.
- Excessive network transfer.
- Long execution times.

A safer approach for inspecting a dataset is:

```python
processed_df.show(20)
```

Or:

```python
sample_rows = processed_df.take(20)
```

**Important:** `take(n)` limits the number of returned rows, but Spark may still need to examine multiple partitions to find them. It is not equivalent to collecting the entire dataset.

### 3.4 Actions and external side effects

Operations such as `foreach()` can execute code with external effects.

```python
def process_record(row):
    print(row["inventory_number"])

processed_df.foreach(process_record)
```

In a distributed environment, this function executes on worker processes, not necessarily on the driver.

There are important considerations:

- Tasks may be retried after failures.
- A side effect may happen more than once.
- Execution order is not guaranteed globally.
- Worker processes do not share ordinary local memory.
- External systems may receive duplicate requests.

For example, directly sending a payment, creating an external record, or triggering an irreversible operation inside a task requires careful design.

Use idempotent operations, deduplication, transactional sinks, or other mechanisms that make retries safe.

## 4. Lazy Evaluation

### 4.1 Definition

Lazy evaluation means Spark can defer executing transformations until an action requires their results.

Instead of immediately processing each operation, Spark builds an execution plan.

Example:

```python
df = spark.read.parquet("data/equipment")

active_df = df.filter(
    df["status"] == "ACTIVE"
)

selected_df = active_df.select(
    "inventory_number",
    "equipment_type",
    "operating_hours"
)

result_df = selected_df.filter(
    selected_df["operating_hours"] > 1000
)
```

These statements define a sequence of operations.

An action requests the result:

```python
result_df.show()
```

Spark then executes the operations needed to satisfy that request.

### 4.2 Why lazy evaluation matters

Lazy evaluation allows Spark to:

1. Understand the complete sequence of transformations.
2. Optimize the logical plan.
3. Choose physical execution strategies.
4. Reduce unnecessary data processing.
5. Combine compatible operations.
6. Execute work across multiple partitions.

Consider:

```python
df.filter("status = 'ACTIVE'").select("inventory_number")
```

Spark may be able to apply filtering and column selection efficiently without materializing every intermediate result as a separate physical dataset.

### 4.3 Lazy evaluation does not mean nothing happens immediately

Some work can happen before an action:

- Python code is executed to construct the DataFrame transformations.
- Spark may resolve column references.
- Schema information may be obtained.
- Logical analysis and validation can occur during certain operations.
- Some metadata or planning operations may execute independently.

For example:

```python
df.select("column_that_does_not_exist")
```

Depending on the operation and context, Spark may report an analysis error before an action is called.

The important distinction is that lazy evaluation primarily concerns the execution of the distributed data-processing plan, not the complete absence of work before an action.

## 5. The Spark Execution Model

Spark organizes distributed computation using a Directed Acyclic Graph (DAG).

A DAG represents a sequence of dependencies between operations.

### 5.1 Directed Acyclic Graph

- **Directed:** Dependencies have a direction.
- **Acyclic:** The dependency graph does not contain a cycle.
- **Graph:** Operations and their dependencies are represented as connected elements.

Example:

```text
Read Data
    |
    v
Filter Active Records
    |
    v
Select Columns
    |
    v
Group by Equipment Type
    |
    v
Calculate Average Hours
    |
    v
Write Results
```

Spark uses the dependencies between operations to organize execution.

### 5.2 Jobs

A job is a unit of execution triggered by an action that requires Spark to compute data.

Example:

```python
processed_df.count()
```

This action typically causes Spark to submit a job.

A single application can execute many jobs because it can contain many actions.

```python
processed_df.count()
processed_df.show()
processed_df.write.parquet("output/equipment")
```

These operations can result in separate jobs, although Spark may reuse cached data or other intermediate results where appropriate.

### 5.3 Stages

A job is divided into stages.

A stage is a group of tasks that can execute without crossing a shuffle boundary.

Shuffle operations often create boundaries between stages because data must be redistributed across partitions.

Example:

```text
Stage 1
  Read
    |
  Filter
    |
  Partial Aggregation
    |
    v
  Shuffle
    |
    v
Stage 2
  Final Aggregation
    |
  Write
```

This is a simplified illustration. Actual stage boundaries depend on the execution plan and Spark's implementation.

### 5.4 Tasks

A task is an individual unit of work executed by a Spark task executor.

Tasks generally process one partition for a particular stage.

For example, if a stage has eight input partitions, Spark may create approximately eight tasks for that stage.

The number of tasks can differ between stages because operations such as repartitioning and shuffling can change partition counts.

### 5.5 Job, stage, and task relationship

```text
Spark Application
      |
      v
    Job
      |
      +------------------+
      |                  |
      v                  v
   Stage 1            Stage 2
      |                  |
      v                  v
   Tasks               Tasks
   T1 T2 T3            T1 T2 T3 T4
```

The diagram illustrates the hierarchy, not a fixed number of stages or tasks.

| Concept | Meaning | Main responsibility |
|---|---|---|
| Application | A Spark program | Coordinates the overall workload |
| Job | Work triggered by an action | Computes a requested result |
| Stage | Group of tasks separated by shuffle boundaries | Executes a portion of the job |
| Task | Unit of distributed execution | Processes a partition |

## 6. Narrow and Wide Transformations

Not all transformations have the same impact on distributed execution.

Spark distinguishes dependencies according to how output partitions depend on input partitions.

### 6.1 Narrow transformations

A narrow transformation occurs when each output partition depends on a limited number of input partitions, commonly one.

This allows Spark to process related operations without redistributing the entire dataset between workers.

Examples:

- `filter()`
- `select()`
- `withColumn()`
- Many simple column expressions.

Conceptual example:

```text
Input Partitions       Output Partitions

Partition 1  ------->  Partition 1
Partition 2  ------->  Partition 2
Partition 3  ------->  Partition 3
```

Benefits:

- Usually lower communication overhead.
- Often fewer execution boundaries.
- Operations can frequently be pipelined.
- Efficient for filtering and projection.

### 6.2 Wide transformations

A wide transformation occurs when output partitions depend on data from multiple input partitions.

This commonly requires a shuffle.

Examples include:

- `groupBy()`
- `distinct()`
- Many joins.
- `orderBy()`.
- `repartition()`.

Conceptual example:

```text
Input Partitions

Partition 1 ----+
Partition 2 ----+----> Shuffle ----> Output Partition 1
Partition 3 ----+                 -> Output Partition 2
Partition 4 ----+                 -> Output Partition 3
```

Data is redistributed according to the requirements of the operation.

### 6.3 Comparison

| Characteristic | Narrow | Wide |
|---|---|---|
| Dependencies | Limited | Multiple input partitions may contribute |
| Shuffle | Usually not required | Commonly required |
| Network transfer | Usually lower | Potentially significant |
| Stage boundary | Usually does not introduce one | Often introduces one |
| Example | `filter()` | `groupBy()` |

**Important:** Do not classify an operation only by its name without considering the physical plan. Optimizations, partitioning, join strategies, and execution details can affect the actual behavior.

## 7. Shuffle Operations

### 7.1 Definition

A shuffle is the redistribution of data between partitions, usually involving network communication and intermediate storage.

It occurs when Spark must reorganize data to perform an operation.

For example, consider grouping maintenance records by equipment type.

```python
maintenance_summary = (
    maintenance_df
    .groupBy("equipment_type")
    .count()
)
```

Records with the same grouping key must be brought together for aggregation.

If those records are distributed across different partitions, Spark needs to exchange data between partitions.

### 7.2 Why shuffles are expensive

A shuffle can involve:

1. Reading records from existing partitions.
2. Organizing records by keys or partitioning rules.
3. Serializing intermediate data.
4. Transferring data across the network.
5. Writing intermediate shuffle files.
6. Reading shuffled data in subsequent stages.

Potential performance costs include:

- Network traffic.
- Disk I/O.
- Serialization overhead.
- Memory pressure.
- Additional execution stages.
- Increased sensitivity to data skew.

### 7.3 Common shuffle-producing operations

| Operation | Why redistribution may be needed |
|---|---|
| `groupBy()` | Records must be grouped by key |
| `distinct()` | Duplicate values must be identified |
| `orderBy()` | Data must be globally ordered |
| `join()` | Matching keys may need to be colocated |
| `repartition()` | Records are redistributed into new partitions |

The exact execution strategy can vary. For example, some joins can avoid a full shuffle through broadcast strategies or existing partitioning.

### 7.4 Inspecting shuffle behavior

Use:

```python
maintenance_summary.explain()
```

For more detailed output:

```python
maintenance_summary.explain("formatted")
```

Look for physical-plan operators such as:

```text
Exchange
```

An `Exchange` commonly indicates a data redistribution or shuffle boundary.

## 8. Partitions and Parallelism

### 8.1 What is a partition?

A partition is a logical subdivision of a distributed dataset.

Spark processes data through partitions so that multiple tasks can work on different portions of the dataset.

Example:

```text
Dataset
   |
   +---- Partition 1
   |
   +---- Partition 2
   |
   +---- Partition 3
   |
   +---- Partition 4
```

Each partition can be processed independently when the operation's dependencies permit it.

### 8.2 Why partitions matter

Partitioning affects:

- Number of tasks.
- Available parallelism.
- Data distribution.
- Memory requirements.
- Shuffle overhead.
- Task execution time.

Too few partitions may limit parallelism.

Too many small partitions may create scheduling and task-management overhead.

The ideal number depends on dataset size, cluster resources, workload characteristics, and operation types.

### 8.3 Inspecting partitions

For a DataFrame:

```python
partition_count = (
    processed_df
    .rdd
    .getNumPartitions()
)

print(partition_count)
```

The conversion to an RDD here is used only to inspect partition count. It is not necessary to convert DataFrames to RDDs for ordinary DataFrame processing.

### 8.4 Parallelism

Parallelism refers to the ability to execute multiple tasks simultaneously.

If Spark has sufficient available resources and a stage contains many independent tasks, multiple tasks can run concurrently.

However:

```text
More partitions != Always faster execution
```

Excessive partition counts can increase scheduling overhead and produce many small files when writing data.

## 9. repartition() and coalesce()

These methods control the number or distribution of partitions.

### 9.1 repartition()

`repartition()` redistributes records and can increase or decrease the number of partitions.

```python
repartitioned_df = processed_df.repartition(12)
```

It can also repartition using one or more columns:

```python
repartitioned_df = processed_df.repartition(
    12,
    "equipment_type"
)
```

Characteristics:

- Usually involves a shuffle.
- Can increase or decrease partition count.
- Can redistribute data more evenly according to the chosen strategy.
- Can be useful before parallel processing or partitioned writes.

### 9.2 coalesce()

`coalesce()` is commonly used to reduce the number of partitions with less data movement than a full repartition.

```python
smaller_df = processed_df.coalesce(4)
```

Characteristics:

- Primarily intended to reduce partitions.
- Usually avoids a full shuffle when reducing partitions.
- May produce uneven partition sizes.
- Is not a general replacement for repartitioning.

### 9.3 Comparison

| Feature | `repartition()` | `coalesce()` |
|---|---|---|
| Increase partitions | Yes | No, not in its ordinary use |
| Reduce partitions | Yes | Yes |
| Full shuffle | Usually | Usually avoided when reducing |
| Redistribution | More complete | Less complete |
| Typical use | Change distribution | Reduce partitions efficiently |

A practical example:

```python
# Increase parallelism and redistribute records
distributed_df = processed_df.repartition(12)

# Reduce the number of output partitions
compact_df = distributed_df.coalesce(4)
```

Do not choose partition counts arbitrarily. Measure the workload and inspect the resulting execution.

## 10. cache() and persist()

### 10.1 Why caching exists

Suppose the same DataFrame is used several times:

```python
active_df = (
    equipment_df
    .filter("status = 'ACTIVE'")
)

active_df.count()
active_df.groupBy("equipment_type").count().show()
active_df.write.parquet("output/active_equipment")
```

Without persistence, Spark may recompute parts of the lineage for separate actions.

Caching can allow Spark to reuse previously computed partitions.

### 10.2 cache()

```python
active_df = active_df.cache()
```

This marks the DataFrame for caching.

The data is generally materialized when an action computes it.

Example:

```python
active_df.cache()

# Materialize the cached DataFrame
active_df.count()

# Reuse the cached result
active_df.groupBy("equipment_type").count().show()
```

### 10.3 persist()

`persist()` provides control over the storage level.

```python
active_df.persist()
```

A storage level can also be specified:

```python
from pyspark import StorageLevel

active_df.persist(
    StorageLevel.MEMORY_AND_DISK
)
```

The exact available levels and defaults depend on the Spark API, version, and language.

Do not assume that every DataFrame or RDD uses the same default storage behavior.

### 10.4 When to cache

Caching can be useful when:

- A DataFrame is reused by multiple actions.
- An expensive transformation is repeatedly required.
- Iterative processing reuses the same dataset.
- Recomputing the lineage is more expensive than storing the result.

Avoid caching when:

- The dataset is used only once.
- The dataset is too large for available resources.
- The cache competes with more important workloads.
- The storage overhead exceeds the cost of recomputation.

### 10.5 Release cached data

Use:

```python
active_df.unpersist()
```

This requests removal of the persisted data from cache or storage.

Good resource management includes releasing datasets that are no longer needed.

## 11. Checkpointing and Lineage

### 11.1 What is lineage?

Lineage is the record of transformations and dependencies used to produce a dataset.

Example:

```text
Source Data
    |
    v
Filter
    |
    v
Select
    |
    v
Join
    |
    v
Aggregate
    |
    v
Final Dataset
```

If a partition is lost, Spark can often recompute it using the original data and the transformations that produced it.

This is an important part of Spark's fault-tolerance model.

### 11.2 What is checkpointing?

Checkpointing saves a dataset to reliable storage and truncates its dependency lineage.

It can be useful when:

- A computation has a very long lineage.
- Repeated recomputation becomes expensive.
- A reliable recovery point is required.
- Long-running iterative workloads need to control lineage growth.

### 11.3 Reliable checkpoint example

```python
spark.sparkContext.setCheckpointDir(
    "hdfs:///spark-checkpoints"
)

checkpointed_df = processed_df.checkpoint(
    eager=True
)
```

A reliable checkpoint directory should use storage suitable for the application's durability requirements.

The example uses a distributed filesystem path. The actual path must exist and be accessible to the Spark application.

### 11.4 localCheckpoint()

```python
local_checkpointed_df = processed_df.localCheckpoint(
    eager=True
)
```

Local checkpointing can truncate lineage without providing the same reliability guarantees as a reliable checkpoint.

It should not be treated as a durable recovery mechanism.

### 11.5 Cache versus checkpoint

| Characteristic | Cache / Persist | Checkpoint |
|---|---|---|
| Main purpose | Reuse computed data | Truncate lineage |
| Storage | Memory and/or disk according to storage level | Checkpoint storage |
| Lineage | Preserved | Truncated after checkpointing |
| Recovery guarantees | Depends on available data and lineage | Depends on checkpoint storage |
| Typical use | Repeated reuse | Long lineage and recovery control |

Caching and checkpointing solve different problems. One should not automatically replace the other.

## 12. RDD Transformations and Actions

Spark originally exposed the Resilient Distributed Dataset (RDD) API as a central abstraction for distributed processing.

DataFrames provide a higher-level structured API.

### 12.1 RDD transformations

Examples:

```python
rdd = spark.sparkContext.parallelize(
    [1, 2, 3, 4, 5, 6]
)

filtered_rdd = rdd.filter(
    lambda value: value > 3
)

mapped_rdd = filtered_rdd.map(
    lambda value: value * 10
)
```

The operations `filter()` and `map()` are transformations.

### 12.2 RDD actions

```python
result = mapped_rdd.collect()

total = mapped_rdd.sum()

number_of_records = mapped_rdd.count()
```

These operations request results and trigger execution.

### 12.3 DataFrames versus RDDs

| Feature | DataFrame | RDD |
|---|---|---|
| Data representation | Structured rows and columns | Distributed objects |
| Schema | Explicit or inferred | No relational schema by default |
| Optimization | Catalyst and SQL optimization | Lower-level execution model |
| API level | Higher-level | Lower-level |
| Typical use | Analytics, ETL, structured processing | Specialized low-level processing |

For most structured data engineering tasks, prefer DataFrames and Spark SQL.

Converting DataFrames to RDDs unnecessarily can reduce the optimizer's ability to reason about the computation.

## 13. Catalyst and Physical Execution

Spark SQL uses Catalyst to analyze and optimize structured queries.

A simplified execution sequence is:

```text
DataFrame Operations
        |
        v
Logical Plan
        |
        v
Analysis
        |
        v
Logical Optimization
        |
        v
Physical Planning
        |
        v
Physical Execution
```

### 13.1 Logical plan

The logical plan describes what the query should do.

It focuses on the requested operations rather than the exact execution strategy.

### 13.2 Optimized logical plan

Catalyst applies transformations to improve the logical plan.

Examples include:

- Predicate pushdown when supported.
- Projection pruning.
- Constant folding.
- Simplification of expressions.
- Other rule-based optimizations.

### 13.3 Physical plan

The physical plan describes how Spark intends to execute the computation.

It can include:

- Scans.
- Filters.
- Projections.
- Exchanges.
- Joins.
- Aggregations.
- Sort operations.

### 13.4 Whole-stage code generation

Spark can combine compatible physical operators into generated code to reduce execution overhead.

This is commonly associated with whole-stage code generation.

It is not applied to every operation. Its availability depends on the physical operators, execution settings, and Spark version.

### 13.5 Inspecting plans

```python
result_df.explain()
```

For extended information:

```python
result_df.explain("extended")
```

For a formatted representation:

```python
result_df.explain("formatted")
```

Learning to read execution plans is essential for understanding performance.

## 14. Understanding an Example Job

Consider this operation:

```python
summary_df = (
    maintenance_df
    .filter("status = 'COMPLETED'")
    .groupBy("equipment_type")
    .count()
)

summary_df.show()
```

The action `show()` requests the result.

A simplified execution could look like this:

```text
Job
 |
 +-- Stage 1
 |     |
 |     +-- Read input partitions
 |     +-- Filter completed records
 |     +-- Perform partial aggregation
 |
 +-- Shuffle
       |
       +-- Stage 2
             |
             +-- Read shuffled data
             +-- Complete aggregation
             +-- Return rows for display
```

### 14.1 What happens in Stage 1?

Spark reads input partitions.

Tasks process the records assigned to their partitions and filter out records that do not satisfy the condition.

Spark may perform partial aggregation before sending intermediate results through the shuffle.

### 14.2 What happens at the shuffle boundary?

Intermediate records are redistributed so that records belonging to the same grouping key can be processed together.

This involves data exchange and creates a stage boundary.

### 14.3 What happens in Stage 2?

Tasks process the shuffled partitions and complete the aggregation.

The final result is made available to the action.

The exact number of stages and tasks depends on the physical plan, input partitions, shuffle configuration, and Spark version.

## 15. Practical PySpark Workflow

This example illustrates a structured data-processing workflow using equipment and maintenance records.

It is an educational example, not a representation of an actual SIAPV database schema.

### 15.1 Create sample data

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = (
    SparkSession.builder
    .appName("SparkTransformationsAndActions")
    .getOrCreate()
)

equipment_data = [
    ("INV-1001", "Excavator", "ACTIVE", 1250),
    ("INV-1002", "Grader", "ACTIVE", 2100),
    ("INV-1003", "Excavator", "INACTIVE", 800),
    ("INV-1004", "Loader", "ACTIVE", 1600),
    ("INV-1005", "Grader", "ACTIVE", 950),
]

equipment_columns = [
    "inventory_number",
    "equipment_type",
    "status",
    "operating_hours"
]

equipment_df = spark.createDataFrame(
    equipment_data,
    equipment_columns
)

equipment_df.show()
```

### 15.2 Apply transformations

```python
active_equipment_df = (
    equipment_df
    .filter(F.col("status") == "ACTIVE")
    .select(
        "inventory_number",
        "equipment_type",
        "operating_hours"
    )
    .withColumn(
        "operating_hours",
        F.col("operating_hours").cast("double")
    )
)
```

No final result has been requested yet.

### 15.3 Create a derived column

```python
classified_equipment_df = (
    active_equipment_df
    .withColumn(
        "usage_category",
        F.when(
            F.col("operating_hours") >= 1500,
            "HIGH_USAGE"
        ).otherwise("STANDARD_USAGE")
    )
)
```

This transformation adds a classification column.

### 15.4 Aggregate records

```python
equipment_summary_df = (
    classified_equipment_df
    .groupBy("equipment_type")
    .agg(
        F.count("*").alias("equipment_count"),
        F.avg("operating_hours").alias(
            "average_operating_hours"
        ),
        F.max("operating_hours").alias(
            "maximum_operating_hours"
        )
    )
)
```

The grouping operation may require a shuffle.

### 15.5 Inspect the physical plan

```python
equipment_summary_df.explain("formatted")
```

Review the plan for:

- Scans.
- Filters.
- Projections.
- Aggregation operators.
- Exchange operators.
- Shuffle boundaries.

### 15.6 Trigger execution

```python
equipment_summary_df.show()
```

This action requests the final summary.

### 15.7 Persist a reused dataset

If the classified dataset will be reused several times:

```python
classified_equipment_df.cache()

classified_equipment_df.count()
```

The count action materializes the cached dataset.

After the remaining operations:

```python
classified_equipment_df.unpersist()
```

### 15.8 Write the output

```python
(
    equipment_summary_df
    .write
    .mode("overwrite")
    .parquet("output/equipment_summary")
)
```

The write operation triggers execution and sends the result to the configured destination.

## 16. Structured Streaming Considerations

The concepts of transformations and execution plans also apply to Structured Streaming, but the execution model includes additional streaming-specific behavior.

A streaming DataFrame represents an unbounded or continuously arriving dataset.

Example:

```python
stream_df = (
    spark.readStream
    .format("json")
    .schema(event_schema)
    .load("input/events")
)

filtered_stream_df = stream_df.filter(
    F.col("event_type") == "MAINTENANCE"
)
```

The transformation defines part of a streaming query.

Execution is started through a streaming write:

```python
query = (
    filtered_stream_df
    .writeStream
    .format("parquet")
    .option("checkpointLocation", "checkpoints/events")
    .start("output/events")
)
```

Unlike a typical batch action such as `collect()`, Structured Streaming uses a long-running streaming query with incremental processing and checkpoint management.

Streaming execution has additional concepts such as:

- Triggers.
- Output modes.
- Event time.
- Watermarks.
- Streaming checkpoints.
- Stateful processing.

These topics will be developed separately.

## 17. Common Mistakes

### Mistake 1: Assuming transformations execute immediately

Incorrect assumption:

```python
filtered_df = df.filter("status = 'ACTIVE'")
```

This generally defines a plan rather than immediately processing all records.

Better understanding: an action requests the result.

### Mistake 2: Calling collect() on large datasets

```python
records = df.collect()
```

This can exhaust driver memory.

Better approach:

```python
df.show(20)
```

### Mistake 3: Using too many actions

```python
df.count()
df.show()
df.collect()
```

Each action may trigger additional computation.

Better approach: plan the required outputs and consider persistence when repeated computation is expensive.

### Mistake 4: Ignoring shuffle operations

```python
df.groupBy("equipment_type").count()
```

Grouping may require data redistribution.

Better approach: inspect the physical plan and understand the cost of wide transformations.

### Mistake 5: Caching everything

Caching every intermediate DataFrame can consume memory and reduce overall performance.

Better approach: cache datasets that are reused and expensive to recompute.

### Mistake 6: Assuming coalesce() always balances data

Reducing partition count does not guarantee equally sized partitions.

Better approach: inspect partition sizes and use repartitioning when a more complete redistribution is required.

### Mistake 7: Performing unsafe external operations inside foreach()

Retries can cause repeated side effects.

Better approach: design idempotent writes and use appropriate transactional or deduplication mechanisms.

## 18. Practical Design Principles

When designing a Spark workflow:

1. Prefer DataFrame and Spark SQL operations for structured data.
2. Keep transformations logically organized.
3. Avoid unnecessary actions.
4. Filter early when doing so reduces the amount of data processed.
5. Select only the columns required by subsequent operations.
6. Identify wide transformations and their shuffle costs.
7. Choose partition counts based on workload measurements.
8. Cache only datasets that are reused.
9. Release cached datasets when they are no longer needed.
10. Use reliable checkpointing when lineage management requires it.
11. Avoid collecting large datasets into the driver.
12. Inspect physical plans before attempting complex performance changes.
13. Make external side effects safe against retries.
14. Measure actual performance instead of assuming that a transformation is efficient.

### 18.1 When is Spark appropriate?

Spark is designed for distributed processing and becomes valuable when workload size, complexity, or throughput justifies distributed execution.

For a relatively small operational dataset, a conventional PostgreSQL database may be simpler and more efficient.

For example, a fleet of approximately 200 units does not inherently require Spark.

Spark becomes more relevant when combining larger volumes of operational history, telemetry, maintenance events, financial records, or multiple data sources for analytical processing.

The architectural decision should be based on workload requirements, not on the popularity of a technology.

## 19. Exercises

### Exercise 1: Identify transformations and actions

Classify each operation:

```python
df.filter("status = 'ACTIVE'")
df.select("inventory_number")
df.count()
df.show()
df.groupBy("equipment_type")
df.collect()
df.repartition(8)
df.write.parquet("output/data")
```

For each one, explain whether it defines a transformation or triggers an action.

### Exercise 2: Understand lazy evaluation

Consider:

```python
df = spark.read.parquet("data/equipment")

filtered_df = df.filter(
    "operating_hours > 1000"
)

selected_df = filtered_df.select(
    "inventory_number",
    "operating_hours"
)
```

Answer:

1. Which operations have been defined?
2. Has the complete dataset necessarily been processed?
3. What happens when `selected_df.show()` is called?
4. What can Catalyst optimize?

### Exercise 3: Analyze dependencies

Classify these operations as commonly narrow or wide:

- `filter()`
- `select()`
- `withColumn()`
- `groupBy()`
- `distinct()`
- `repartition()`
- `orderBy()`

Explain why each operation has its typical dependency behavior.

### Exercise 4: Compare partition strategies

Suppose a DataFrame has 24 partitions.

Explain the likely effects of:

```python
df.repartition(8)
df.coalesce(8)
df.repartition(48)
```

Discuss shuffle behavior, parallelism, and possible performance consequences.

### Exercise 5: Build a processing pipeline

Create a PySpark workflow that:

1. Reads a Parquet dataset.
2. Filters active equipment.
3. Selects inventory number, equipment type, and operating hours.
4. Creates a usage classification.
5. Groups records by equipment type.
6. Calculates count and average operating hours.
7. Inspects the physical plan.
8. Displays the result.
9. Writes the output to Parquet.

Explain which operations are transformations and which are actions.

### Exercise 6: Evaluate caching

A DataFrame is used in five different analytical operations.

Answer:

1. Why might caching improve performance?
2. When could caching make performance worse?
3. How can the cached data be released?
4. How does checkpointing differ from caching?

## 20. Knowledge Check

1. What is the main difference between a transformation and an action?
2. What is lazy evaluation?
3. What does a DAG represent?
4. What triggers a Spark job?
5. What is a stage?
6. What is a task?
7. Why do wide transformations commonly require shuffles?
8. Why can `collect()` cause driver memory problems?
9. What is the difference between `repartition()` and `coalesce()`?
10. What is the purpose of caching?
11. What is the purpose of checkpointing?
12. How does lineage support fault tolerance?
13. Why should external side effects be handled carefully inside distributed tasks?
14. Why are DataFrames generally preferable to RDDs for structured analytics?
15. What information can `explain("formatted")` provide?

## 21. Glossary

| Term | Definition |
|---|---|
| Transformation | Operation that produces a new distributed dataset |
| Action | Operation that requests a result or external output |
| Lazy Evaluation | Deferred execution of data-processing operations |
| DAG | Directed Acyclic Graph representing computation dependencies |
| Job | Unit of Spark execution generally triggered by an action |
| Stage | Group of tasks separated by shuffle boundaries |
| Task | Unit of work executed for a partition |
| Partition | Logical subdivision of a distributed dataset |
| Narrow Transformation | Transformation with limited input-partition dependencies |
| Wide Transformation | Transformation requiring data from multiple partitions |
| Shuffle | Redistribution of data between partitions |
| Parallelism | Ability to execute multiple tasks concurrently |
| Cache | Mechanism for reusing computed data |
| Persist | Mechanism for storing computed data using a storage level |
| Checkpoint | Saved computation result used to truncate lineage |
| Lineage | Dependency history used to derive a dataset |
| Catalyst | Spark SQL optimizer |
| Physical Plan | Execution strategy selected for a query |
| Data Skew | Uneven distribution of data across partitions |
| Driver | Process coordinating the Spark application |
| Executor | Process responsible for executing tasks and storing data |

## 22. Related Notes

### Prerequisites

- [[10 - Apache Spark Fundamentals]]
- [[11 - Apache Spark DataFrames]]
- [[12 - Spark SQL]]

### Related concepts

- [[09 - Distributed Data Processing]]
- [[04 - Data Pipelines]]
- [[14 - Spark Joins and Aggregations]]
- [[15 - Spark Performance Optimization]]

## 23. Key Takeaways

- Transformations define how distributed data should be processed.
- Actions request results and generally trigger execution.
- Lazy evaluation allows Spark to optimize computation before executing it.
- Spark organizes distributed workloads into jobs, stages, and tasks.
- Narrow transformations generally avoid shuffle boundaries.
- Wide transformations commonly require data redistribution.
- Partitions influence parallelism and execution overhead.
- `repartition()` and `coalesce()` serve different partition-management purposes.
- Cache and persist support reuse; checkpointing manages lineage.
- Lineage helps Spark recover lost partitions.
- DataFrame and SQL operations are generally preferable for structured analytics.
- Understanding execution plans is fundamental to building efficient Spark applications.
