
---
title: "Spark Joins and Aggregations"
aliases:
  - "Joining and Aggregating Data in Apache Spark"
  - "Spark DataFrame Joins"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: spark-joins-aggregations
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - apache-spark
  - pyspark
  - joins
  - aggregations
  - data-engineering
  - distributed-processing
  - data-analysis
concepts:
  - inner-join
  - left-join
  - right-join
  - full-outer-join
  - semi-join
  - anti-join
  - broadcast-join
  - shuffle-join
  - group-by
  - aggregate-functions
  - data-skew
  - join-keys
prerequisites:
  - "[[11 - Apache Spark DataFrames]]"
  - "[[12 - Spark SQL]]"
  - "[[13 - Spark Transformations and Actions]]"
related:
  - "[[04 - Data Pipelines]]"
  - "[[09 - Distributed Data Processing]]"
  - "[[15 - Spark Performance Optimization]]"
tools:
  - Apache Spark
  - PySpark
  - Python
  - SQL
learning_objectives:
  - "Understand how Spark combines datasets using joins."
  - "Distinguish common relational join types."
  - "Choose appropriate join keys."
  - "Understand broadcast and shuffle join strategies."
  - "Apply aggregation functions to distributed datasets."
  - "Recognize data skew and duplicate-key problems."
  - "Build an analytical dataset using multiple DataFrames."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Spark Joins and Aggregations

## 1. Introduction

Data engineering frequently requires combining information from different sources.

For example, an organization may maintain separate datasets for:

- Equipment inventory.
- Maintenance records.
- Operating hours.
- Administrative assignments.
- Operational costs.

These datasets can be combined using **joins** and summarized using **aggregations**.

Apache Spark provides DataFrame and SQL operations for these tasks.

Two fundamental operations are:

- **Join:** Combines records from two datasets using a relationship or matching condition.
- **Aggregation:** Calculates summarized values from a collection of records.

Together, these operations allow engineers to transform raw data into analytical datasets.

## 2. Understanding Joins

### 2.1 What is a join?

A join combines rows from two datasets based on a matching condition.

Consider two tables.

**Equipment**

| inventory_number | equipment_type |
|---|---|
| INV-001 | Excavator |
| INV-002 | Grader |
| INV-003 | Loader |

**Maintenance**

| maintenance_id | inventory_number | cost |
|---|---|---:|
| 101 | INV-001 | 2500 |
| 102 | INV-001 | 1800 |
| 103 | INV-002 | 3200 |

Both datasets share the column `inventory_number`.

This column can be used to associate maintenance records with their corresponding equipment.

Conceptually:

```text
Equipment
    |
    | inventory_number
    |
    v
  JOIN
    ^
    | inventory_number
    |
Maintenance
```

The result contains information from both datasets.

### 2.2 Join keys

A join key is the column or combination of columns used to match records.

Example:

```python
equipment_df.join(
    maintenance_df,
    on="inventory_number"
)
```

A valid join key should represent the intended relationship between the datasets.

Before joining, verify:

- Both columns represent the same business concept.
- Their data types are compatible.
- Their values use consistent formats.
- Null values are understood.
- Duplicate keys are intentional or handled appropriately.

A technically valid join can still produce incorrect business results if the key does not represent the correct relationship.

## 3. Types of Joins

### 3.1 Inner Join

An inner join returns only records with matching keys in both datasets.

```python
inner_df = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="inner"
)
```

Conceptually:

```text
Equipment       Maintenance
    A                A
    B                C
    C                D

Inner Join:
    A
```

Use cases:

- Finding equipment with maintenance records.
- Matching transactions with valid customers.
- Combining records that exist in both systems.

### 3.2 Left Join

A left join returns every record from the left dataset and matching records from the right dataset.

If no match exists, right-side columns contain NULL.

```python
left_df = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="left"
)
```

Example:

| inventory_number | equipment_type | maintenance_id |
|---|---|---:|
| INV-001 | Excavator | 101 |
| INV-001 | Excavator | 102 |
| INV-002 | Grader | 103 |
| INV-003 | Loader | NULL |

Use cases:

- Listing all equipment, including equipment without maintenance history.
- Keeping all customers while attaching available transaction information.
- Identifying missing related records.

### 3.3 Right Join

A right join preserves all records from the right dataset and includes matching records from the left dataset.

```python
right_df = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="right"
)
```

Use cases:

- Preserving every maintenance record.
- Identifying maintenance records without a corresponding equipment record.

A right join can often be rewritten as a left join by reversing the order of the datasets.

### 3.4 Full Outer Join

A full outer join returns matching and non-matching records from both datasets.

```python
full_df = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="full_outer"
)
```

Conceptually:

```text
Left Dataset       Right Dataset
     A                  A
     B                  C
     D                  E

Full Outer Join:
     A
     B
     D
     C
     E
```

Non-matching columns are represented with NULL values.

Use cases:

- Reconciling two data sources.
- Comparing inventory systems.
- Identifying missing records on either side.

### 3.5 Left Semi Join

A left semi join returns rows from the left dataset when a matching key exists in the right dataset.

It does not return columns from the right dataset.

```python
semi_df = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="left_semi"
)
```

Use cases:

- Finding equipment with at least one maintenance record.
- Filtering customers that have transactions.
- Checking whether a relationship exists.

### 3.6 Left Anti Join

A left anti join returns rows from the left dataset that have no matching key in the right dataset.

```python
anti_df = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="left_anti"
)
```

Use cases:

- Finding equipment without maintenance records.
- Identifying customers without transactions.
- Detecting missing relationships between systems.

### 3.7 Join comparison

| Join type | Returned records |
|---|---|
| Inner | Matching records from both datasets |
| Left | All left records and matching right records |
| Right | All right records and matching left records |
| Full Outer | All records from both datasets |
| Left Semi | Left records with a match |
| Left Anti | Left records without a match |

## 4. Joining on Multiple Columns

Some relationships require more than one matching column.

For example, two datasets may identify a transaction using both:

- `equipment_id`
- `transaction_date`

Example:

```python
condition = (
    (equipment_df["equipment_id"] == events_df["equipment_id"]) &
    (equipment_df["location_id"] == events_df["location_id"])
)

joined_df = equipment_df.join(
    events_df,
    on=condition,
    how="inner"
)
```

Multiple-column conditions should be used when the combination represents the actual relationship.

Do not add unnecessary conditions simply to make a join appear more restrictive.

## 5. Handling Duplicate Column Names

When two DataFrames contain columns with the same name, a join can produce ambiguous references.

Example:

```python
joined_df = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="inner"
)
```

If both datasets contain a column called `status`, referring to `status` may become ambiguous.

A safer approach is to rename or alias columns before joining.

```python
from pyspark.sql import functions as F

equipment = equipment_df.alias("equipment")
maintenance = maintenance_df.alias("maintenance")

joined_df = (
    equipment
    .join(
        maintenance,
        F.col("equipment.inventory_number") ==
        F.col("maintenance.inventory_number"),
        "inner"
    )
    .select(
        F.col("equipment.inventory_number"),
        F.col("equipment.equipment_type"),
        F.col("maintenance.maintenance_id"),
        F.col("maintenance.cost")
    )
)
```

This makes the origin of each selected column explicit.

## 6. Understanding Join Cardinality

Join cardinality describes how many records can match between datasets.

Common relationships include:

### One-to-One

Each record in the first dataset matches at most one record in the second dataset.

```text
Equipment A ---- Record 1
Equipment B ---- Record 2
Equipment C ---- Record 3
```

### One-to-Many

One record in the first dataset can match multiple records in the second dataset.

```text
Equipment A ---- Maintenance 1
             ---- Maintenance 2
             ---- Maintenance 3
```

This is common when equipment has multiple maintenance events.

### Many-to-Many

Multiple records in each dataset can match multiple records in the other dataset.

This relationship may require a bridge table or an intermediate mapping dataset.

### Why cardinality matters

Suppose one equipment record matches three maintenance records.

The resulting joined dataset contains three rows for that equipment.

This is expected for a one-to-many relationship.

However, if the equipment dataset also contains duplicate inventory numbers, the number of resulting rows may increase unexpectedly.

Always inspect key uniqueness and expected cardinality before interpreting join results.

## 7. Join Strategies in Spark

Spark can use different physical strategies to execute joins.

The selected strategy depends on factors such as:

- Dataset sizes.
- Available statistics.
- Join conditions.
- Partitioning.
- Configuration settings.
- Resource availability.

Two important strategies are broadcast joins and shuffle-based joins.

### 7.1 Broadcast Join

A broadcast join sends a relatively small dataset to the worker nodes so that it can be used locally while processing a larger dataset.

Example:

```python
from pyspark.sql.functions import broadcast

joined_df = large_df.join(
    broadcast(small_df),
    on="equipment_type",
    how="inner"
)
```

Conceptually:

```text
             Small Dataset
                   |
                   v
             Broadcast Data
              /    |    \
             v     v     v
          Worker Worker Worker
             |     |     |
             v     v     v
          Large Dataset Partitions
```

Advantages:

- Can avoid shuffling the large dataset.
- Can reduce network communication.
- Often efficient when one side is genuinely small.

Risks:

- Broadcasting a dataset that is too large can consume substantial executor memory.
- The size of the dataset in memory may differ from its compressed storage size.
- Broadcasting is not automatically beneficial for every join.

### 7.2 Shuffle-Based Join

A shuffle-based join redistributes records so that matching keys can be processed together.

Conceptually:

```text
Dataset A Partitions
        |
        v
      Shuffle
        |
        +--------+
        |        |
        v        v
   Join Partition 1
   Join Partition 2
        ^
        |
      Shuffle
        |
Dataset B Partitions
```

This strategy can be appropriate when both datasets are large and broadcasting is not suitable.

It may involve:

- Network transfer.
- Intermediate storage.
- Serialization.
- Additional stages.

### 7.3 Comparing strategies

| Strategy | Typical situation | Main consideration |
|---|---|---|
| Broadcast Join | One dataset is small | Memory required on workers |
| Shuffle-Based Join | Both datasets are large | Network and shuffle cost |

The physical plan can be inspected using:

```python
joined_df.explain("formatted")
```

Look for operators such as:

- `BroadcastHashJoin`
- `SortMergeJoin`
- `ShuffledHashJoin`
- `Exchange`

The exact strategy depends on Spark's optimizer and execution configuration.

## 8. Aggregations

### 8.1 What is an aggregation?

An aggregation summarizes multiple records into calculated values.

For example, a maintenance dataset may contain hundreds of records per equipment type.

Instead of examining every record individually, an aggregation can calculate:

- Total maintenance cost.
- Average operating hours.
- Number of maintenance events.
- Maximum cost.
- Minimum operating hours.

### 8.2 Basic aggregation functions

PySpark provides aggregate functions through `pyspark.sql.functions`.

```python
from pyspark.sql import functions as F
```

| Function | Purpose |
|---|---|
| `count()` | Count rows or non-NULL values |
| `sum()` | Calculate a total |
| `avg()` | Calculate an average |
| `min()` | Find the minimum |
| `max()` | Find the maximum |
| `countDistinct()` | Count distinct values |
| `first()` | Return a first value according to the operation's semantics |
| `last()` | Return a last value according to the operation's semantics |

Example:

```python
summary_df = maintenance_df.agg(
    F.count("*").alias("total_records"),
    F.sum("cost").alias("total_cost"),
    F.avg("cost").alias("average_cost"),
    F.min("cost").alias("minimum_cost"),
    F.max("cost").alias("maximum_cost")
)
```

This produces a summary across the entire dataset.

### 8.3 Understanding NULL values

Aggregate functions do not all treat NULL values in the same way.

For example:

```python
maintenance_df.agg(
    F.count("*").alias("row_count"),
    F.count("cost").alias("cost_count"),
    F.sum("cost").alias("total_cost")
)
```

- `count("*")` counts rows.
- `count("cost")` counts non-NULL values in the `cost` column.
- `sum("cost")` calculates the sum of non-NULL values.

This distinction matters when working with incomplete records.

## 9. GROUP BY

A grouped aggregation calculates summaries for each distinct combination of grouping columns.

Example:

```python
equipment_summary = (
    maintenance_df
    .groupBy("equipment_type")
    .agg(
        F.count("*").alias("maintenance_count"),
        F.sum("cost").alias("total_cost"),
        F.avg("cost").alias("average_cost")
    )
)
```

Possible result:

| equipment_type | maintenance_count | total_cost | average_cost |
|---|---:|---:|---:|
| Excavator | 12 | 45000 | 3750 |
| Grader | 8 | 32000 | 4000 |
| Loader | 5 | 18000 | 3600 |

The values are illustrative.

### 9.1 Grouping by multiple columns

```python
summary_df = (
    maintenance_df
    .groupBy(
        "equipment_type",
        "maintenance_status"
    )
    .agg(
        F.count("*").alias("record_count"),
        F.sum("cost").alias("total_cost")
    )
)
```

This produces a separate summary for each equipment type and maintenance status combination.

### 9.2 Filtering aggregated results

Use `filter()` after defining the aggregation.

```python
high_cost_groups = (
    maintenance_df
    .groupBy("equipment_type")
    .agg(
        F.sum("cost").alias("total_cost")
    )
    .filter(F.col("total_cost") > 10000)
)
```

In SQL, the equivalent concept is `HAVING`.

```sql
SELECT
    equipment_type,
    SUM(cost) AS total_cost
FROM maintenance
GROUP BY equipment_type
HAVING SUM(cost) > 10000;
```

## 10. Data Skew

### 10.1 Definition

Data skew occurs when records are distributed unevenly across partitions or join keys.

For example:

| equipment_type | record_count |
|---|---:|
| Excavator | 900000 |
| Grader | 50000 |
| Loader | 30000 |
| Other | 20000 |

If one key represents a disproportionately large amount of data, the corresponding partition may require much more processing than others.

### 10.2 Why skew matters

Data skew can cause:

- Uneven task execution times.
- Long-running tasks.
- Memory pressure.
- Large shuffle partitions.
- Reduced cluster efficiency.

A job may appear to be almost finished while one heavily skewed task continues running.

### 10.3 Identifying skew

A simple way to inspect key distribution is:

```python
key_distribution = (
    maintenance_df
    .groupBy("equipment_type")
    .count()
    .orderBy(F.desc("count"))
)

key_distribution.show()
```

For more detailed analysis, inspect partition sizes and execution metrics in the Spark UI.

### 10.4 Common mitigation strategies

Possible approaches include:

- Repartitioning when it improves distribution.
- Broadcasting a genuinely small dataset.
- Filtering unnecessary records before joining.
- Pre-aggregating data when logically valid.
- Using salting for heavily skewed join keys.
- Enabling Adaptive Query Execution when appropriate.

No single solution works for every skew problem.

## 11. Practical PySpark Example

This example combines equipment inventory and maintenance history.

It is an illustrative model, not the actual SIAPV schema.

### 11.1 Create the datasets

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = (
    SparkSession.builder
    .appName("SparkJoinsAndAggregations")
    .getOrCreate()
)

equipment_data = [
    ("INV-001", "Excavator", "ACTIVE"),
    ("INV-002", "Grader", "ACTIVE"),
    ("INV-003", "Loader", "INACTIVE"),
    ("INV-004", "Excavator", "ACTIVE")
]

equipment_columns = [
    "inventory_number",
    "equipment_type",
    "status"
]

equipment_df = spark.createDataFrame(
    equipment_data,
    equipment_columns
)

maintenance_data = [
    (101, "INV-001", 2500.0, "COMPLETED"),
    (102, "INV-001", 1800.0, "COMPLETED"),
    (103, "INV-002", 3200.0, "COMPLETED"),
    (104, "INV-003", 1200.0, "PENDING"),
    (105, "INV-004", 2100.0, "COMPLETED")
]

maintenance_columns = [
    "maintenance_id",
    "inventory_number",
    "cost",
    "maintenance_status"
]

maintenance_df = spark.createDataFrame(
    maintenance_data,
    maintenance_columns
)
```

### 11.2 Join the datasets

```python
joined_df = (
    equipment_df
    .join(
        maintenance_df,
        on="inventory_number",
        how="left"
    )
)
```

This preserves all equipment records and attaches matching maintenance records.

### 11.3 Filter completed maintenance

```python
completed_df = joined_df.filter(
    F.col("maintenance_status") == "COMPLETED"
)
```

Notice that this filter removes rows where the maintenance status is NULL.

If the objective is to preserve equipment without maintenance history, filtering should be designed carefully.

For example, the following condition retains unmatched equipment:

```python
completed_or_missing_df = joined_df.filter(
    (F.col("maintenance_status") == "COMPLETED") |
    (F.col("maintenance_status").isNull())
)
```

### 11.4 Aggregate maintenance costs

```python
equipment_cost_summary = (
    completed_df
    .groupBy("equipment_type")
    .agg(
        F.count("maintenance_id").alias(
            "completed_events"
        ),
        F.sum("cost").alias(
            "total_maintenance_cost"
        ),
        F.avg("cost").alias(
            "average_maintenance_cost"
        )
    )
)
```

### 11.5 Inspect the result

```python
equipment_cost_summary.show()
```

### 11.6 Inspect the execution plan

```python
equipment_cost_summary.explain("formatted")
```

Review the join strategy, exchanges, and aggregation operators.

## 12. Common Mistakes

### Mistake 1: Joining with the wrong key

A join can execute successfully while producing incorrect relationships.

Always verify that the key represents the intended relationship.

### Mistake 2: Ignoring duplicate keys

Duplicate keys can multiply records during a join.

Check uniqueness when the business relationship is expected to be one-to-one.

### Mistake 3: Using an inner join when unmatched records matter

An inner join removes records without matches.

Use a left or full outer join when preserving unmatched records is required.

### Mistake 4: Filtering after a left join without considering NULL values

A filter on a right-side column can remove unmatched rows and effectively change the intended result.

Decide whether unmatched records should remain.

### Mistake 5: Broadcasting a large dataset

Broadcasting can cause memory problems if the dataset is too large.

Inspect sizes and execution plans before forcing a broadcast.

### Mistake 6: Misinterpreting aggregate results

A sum or count may be misleading when records contain duplicates or incomplete values.

Validate the underlying data before interpreting the summary.

### Mistake 7: Ignoring data skew

Uneven key distributions can produce slow tasks even when most partitions finish quickly.

Inspect key frequencies and execution metrics.

## 13. Practical Design Principles

1. Define the business relationship before writing a join.
2. Validate key data types and formats.
3. Understand the expected cardinality.
4. Check duplicate keys before joining.
5. Select only the columns required from each dataset.
6. Use the appropriate join type for the analytical requirement.
7. Preserve unmatched records when necessary.
8. Use broadcast joins only when appropriate.
9. Inspect physical plans for expensive exchanges.
10. Validate aggregate results against the source data.
11. Investigate data skew when task execution is uneven.
12. Document the meaning of each analytical measure.

## 14. Exercises

### Exercise 1: Identify join types

For each requirement, choose an appropriate join:

1. Return only equipment with maintenance records.
2. Return all equipment, including equipment without maintenance records.
3. Find equipment without maintenance records.
4. Reconcile two inventory systems and preserve all records.
5. Find equipment that has at least one maintenance event without returning maintenance columns.

### Exercise 2: Analyze cardinality

Suppose one equipment record matches five maintenance records.

Answer:

1. How many rows can the join produce for that equipment?
2. What happens if the equipment dataset contains two records with the same key?
3. Why can duplicate keys affect aggregation results?

### Exercise 3: Design an aggregation

Create a summary grouped by equipment type containing:

- Number of maintenance events.
- Total maintenance cost.
- Average maintenance cost.
- Maximum maintenance cost.

Explain the meaning of each measure.

### Exercise 4: Understand join strategies

Explain:

1. What is a broadcast join?
2. Why can it avoid a large shuffle?
3. What happens if the broadcast dataset is too large?
4. When might a shuffle-based join be more appropriate?

### Exercise 5: Detect data skew

Create a grouped count of records by join key.

Identify the most frequent keys and explain how an uneven distribution could affect execution.

## 15. Knowledge Check

1. What is a join?
2. What is a join key?
3. What is the difference between inner and left joins?
4. What does a left anti join return?
5. What is join cardinality?
6. Why can duplicate keys multiply rows?
7. What is a broadcast join?
8. What is a shuffle-based join?
9. What is an aggregation?
10. What is the purpose of GROUP BY?
11. How does COUNT(*) differ from COUNT(column)?
12. What is data skew?
13. Why can a left join followed by a filter remove unmatched rows?
14. How can the physical plan help identify join strategies?
15. Why should aggregation results be validated?

## 16. Glossary

| Term | Definition |
|---|---|
| Join | Operation that combines datasets using a matching condition |
| Join Key | Column or columns used to match records |
| Inner Join | Returns matching records |
| Left Join | Preserves all records from the left dataset |
| Full Outer Join | Preserves records from both datasets |
| Semi Join | Returns left-side records with a match |
| Anti Join | Returns left-side records without a match |
| Cardinality | Number of records or matches in a relationship |
| Broadcast Join | Join strategy that distributes a small dataset to workers |
| Shuffle Join | Join strategy involving redistribution of records |
| Aggregation | Calculation that summarizes multiple records |
| GROUP BY | Groups records by one or more columns |
| Data Skew | Uneven distribution of records across keys or partitions |
| NULL | Representation of missing or unknown data |
| Join Condition | Expression defining which records match |

## 17. Related Notes

### Prerequisites

- [[11 - Apache Spark DataFrames]]
- [[12 - Spark SQL]]
- [[13 - Spark Transformations and Actions]]

### Related concepts

- [[04 - Data Pipelines]]
- [[09 - Distributed Data Processing]]
- [[15 - Spark Performance Optimization]]

## 18. Key Takeaways

- Joins combine datasets through matching conditions.
- Join types determine which matching and unmatched records are preserved.
- Join keys and cardinality must be validated.
- Duplicate keys can multiply rows.
- Broadcast joins can be efficient when one dataset is small.
- Shuffle-based joins redistribute data and may involve significant overhead.
- Aggregations summarize records using functions such as COUNT, SUM, AVG, MIN, and MAX.
- GROUP BY produces summaries for distinct key combinations.
- NULL values affect aggregate calculations.
- Data skew can create uneven workloads and slow execution.
- Physical execution plans help explain how Spark performs joins and aggregations.
