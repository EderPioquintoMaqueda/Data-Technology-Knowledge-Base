
---
title: "Spark SQL"
aliases:
  - "Apache Spark SQL"
  - "SQL Queries in PySpark"
  - "Spark SQL Engine"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: spark-sql
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - apache-spark
  - spark-sql
  - pyspark
  - sql
  - data-engineering
  - distributed-query-processing
  - data-transformation
concepts:
  - spark-sql
  - spark-session
  - temporary-views
  - global-temporary-views
  - sql-queries
  - common-table-expressions
  - subqueries
  - joins
  - aggregations
  - window-functions
  - catalyst-optimizer
  - logical-plan
  - physical-plan
  - query-execution
prerequisites:
  - "[[10 - Apache Spark Fundamentals]]"
  - "[[11 - Apache Spark DataFrames]]"
  - "[[04 - Data Pipelines]]"
related:
  - "[[13 - Spark Transformations and Actions]]"
  - "[[14 - Spark Joins and Aggregations]]"
  - "[[15 - Spark Performance Optimization]]"
  - "[[05 - Data Warehouses]]"
  - "[[03 - ETL and ELT]]"
tools:
  - Apache Spark
  - Spark SQL
  - PySpark
  - Python
  - SQL
learning_objectives:
  - "Explain the role of Spark SQL in the Apache Spark ecosystem."
  - "Understand how Spark SQL processes structured queries."
  - "Create and manage temporary and global temporary views."
  - "Write SQL queries against Spark DataFrames."
  - "Use filtering, projection, aggregation, and sorting."
  - "Apply joins, subqueries, and common table expressions."
  - "Use SQL window functions for analytical processing."
  - "Combine SQL queries with PySpark DataFrame operations."
  - "Understand Catalyst optimization and physical execution plans."
  - "Apply security and performance practices to Spark SQL workloads."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Spark SQL

## 1. Introduction

**Spark SQL** is the Apache Spark module for processing structured and semi-structured data through SQL queries and DataFrame operations.

It allows developers and data engineers to work with structured information using familiar SQL syntax while benefiting from Spark's distributed execution capabilities.

Spark SQL supports:

- SQL queries.
- DataFrame operations.
- Relational data processing.
- Structured data sources.
- Query optimization.
- Distributed query execution.
- Integration with external catalogs and storage systems.

It is particularly useful when working with analytical datasets, data pipelines, data lakes, and data warehouse architectures.

### 1.1 Why Spark SQL exists

Data engineers frequently need to:

- Retrieve specific columns.
- Filter records.
- Combine datasets.
- Calculate aggregates.
- Analyze historical information.
- Transform raw data into curated datasets.

These operations can be expressed through SQL.

For example:

```sql
SELECT
    equipment_type,
    COUNT(*) AS maintenance_events,
    SUM(maintenance_cost) AS total_cost
FROM maintenance_records
GROUP BY equipment_type;
```

Spark SQL can execute this query over distributed data.

The SQL statement describes the desired result. Spark determines how to execute the operations.

### 1.2 Spark SQL and traditional relational databases

Spark SQL shares many concepts with relational database systems, but the underlying execution architecture is different.

| Characteristic | Traditional RDBMS | Spark SQL |
|---|---|---|
| Main purpose | Transactional and analytical database operations | Distributed structured data processing |
| Execution | Database engine | Distributed Spark engine |
| Data organization | Tables and indexes | DataFrames, tables, and external data sources |
| Processing | Usually database-managed | Distributed across executors when configured |
| Query language | SQL dialect | Spark SQL dialect |
| Optimization | Database-specific optimizer | Catalyst optimizer |
| Typical workloads | OLTP and OLAP, depending on system | Large-scale analytical and data engineering workloads |

Spark SQL is not a replacement for every relational database.

A relational database may remain the appropriate system for transactional operations, while Spark SQL processes large analytical datasets.

### 1.3 Spark SQL and PySpark DataFrames

Spark SQL and the DataFrame API are two interfaces to structured data processing.

Consider the following DataFrame operation:

```python
maintenance_df.filter(
    col("maintenance_cost") > 3000
).select(
    "inventory_number",
    "maintenance_cost"
)
```

The equivalent SQL query is:

```sql
SELECT
    inventory_number,
    maintenance_cost
FROM maintenance_records
WHERE maintenance_cost > 3000;
```

Both interfaces can use Spark SQL's underlying optimization and execution infrastructure.

The choice depends on the task, team conventions, readability, and application requirements.

---

## 2. Spark SQL Architecture

Spark SQL is integrated into the Spark architecture.

A simplified processing flow is:

```text
SQL Query
    |
    v
SQL Parser
    |
    v
Parsed Logical Plan
    |
    v
Analyzer
    |
    v
Analyzed Logical Plan
    |
    v
Catalyst Optimizer
    |
    v
Optimized Logical Plan
    |
    v
Physical Planning
    |
    v
Physical Execution Plan
    |
    v
Distributed Execution
    |
    v
Result
```

### 2.1 SQL Parser

The parser receives the SQL statement and converts it into an initial internal representation.

For example:

```sql
SELECT inventory_number
FROM maintenance_records
WHERE maintenance_cost > 3000;
```

The parser identifies:

- Selected columns.
- Source relation.
- Filter condition.
- SQL operators.
- Expression structure.

At this stage, Spark has not necessarily executed the query.

### 2.2 Analyzer

The analyzer resolves references in the query.

It checks information such as:

- Whether the referenced table exists.
- Whether columns exist.
- Whether expressions are valid.
- Whether data types are compatible.
- Whether functions are recognized.

For example, if the query references a nonexistent column, analysis can fail before the actual distributed computation begins.

### 2.3 Catalyst Optimizer

Catalyst is Spark SQL's query optimization framework.

It transforms logical query plans into equivalent plans that can potentially execute more efficiently.

Examples of optimization techniques include:

- Predicate pushdown.
- Column pruning.
- Constant folding.
- Expression simplification.
- Boolean expression simplification.
- Join strategy selection.
- Reordering certain operations when valid.

The optimizer considers the query structure, available metadata, data source capabilities, and configuration.

### 2.4 Physical planning

After logical optimization, Spark selects physical execution strategies.

For example, a join may use:

- Broadcast hash join.
- Sort-merge join.
- Shuffled hash join.
- Nested-loop join in applicable cases.

The selected strategy depends on the query, available statistics, configuration, and data characteristics.

### 2.5 Distributed execution

The physical plan is executed through Spark's distributed processing architecture.

The driver coordinates the application, and executors perform tasks on data partitions.

The execution may involve:

- Reading files.
- Applying filters.
- Projecting columns.
- Redistributing records.
- Joining datasets.
- Calculating aggregates.
- Producing output.

---

## 3. Executing SQL Queries in PySpark

The `SparkSession` provides the `sql()` method.

It accepts a SQL statement and returns a Spark DataFrame.

### 3.1 Basic example

```python
result_df = spark.sql("""
    SELECT
        inventory_number,
        equipment_type
    FROM equipment
""")
```

The result is a DataFrame.

It can be inspected using the DataFrame API:

```python
result_df.show()
```

Its schema can also be examined:

```python
result_df.printSchema()
```

This demonstrates an important principle:

**Spark SQL queries return DataFrames, allowing SQL and PySpark operations to be combined.**

### 3.2 Executing a query directly

```python
spark.sql("""
    SELECT
        COUNT(*) AS total_equipment
    FROM equipment
""").show()
```

### 3.3 Storing query results

```python
equipment_summary = spark.sql("""
    SELECT
        equipment_type,
        COUNT(*) AS equipment_count
    FROM equipment
    GROUP BY equipment_type
""")
```

The query result can be reused in subsequent transformations.

```python
equipment_summary.orderBy(
    "equipment_count",
    ascending=False
).show()
```

---

## 4. Temporary Views

A Spark DataFrame is not automatically available as a SQL table.

To query a DataFrame using SQL, it can be registered as a temporary view.

A temporary view provides a SQL-accessible name for the DataFrame.

### 4.1 Creating a temporary view

```python
maintenance_df.createOrReplaceTempView(
    "maintenance_records"
)
```

The view can now be referenced through SQL:

```python
result_df = spark.sql("""
    SELECT *
    FROM maintenance_records
""")
```

### 4.2 Temporary view characteristics

A local temporary view:

- Is associated with the SparkSession.
- Can be queried using SQL.
- Does not automatically persist the underlying data to storage.
- Is available while its session remains active.
- Can be replaced using `createOrReplaceTempView()`.

A temporary view is a logical SQL interface, not necessarily a physical copy of the DataFrame.

### 4.3 Replacing a temporary view

```python
maintenance_df.createOrReplaceTempView(
    "maintenance_records"
)
```

If the view already exists, the operation replaces its definition.

### 4.4 Dropping a temporary view

```python
spark.catalog.dropTempView(
    "maintenance_records"
)
```

This removes the temporary view from the session catalog.

### 4.5 Checking whether a view exists

```python
spark.catalog.tableExists(
    "maintenance_records"
)
```

This checks whether the named relation exists in the accessible catalog.

---

## 5. Global Temporary Views

A global temporary view has a broader scope than a local temporary view.

It is associated with the Spark application rather than only one SparkSession.

Global temporary views are stored in the reserved `global_temp` database.

### 5.1 Creating a global temporary view

```python
maintenance_df.createOrReplaceGlobalTempView(
    "maintenance_records_global"
)
```

### 5.2 Querying a global temporary view

```python
result_df = spark.sql("""
    SELECT *
    FROM global_temp.maintenance_records_global
""")
```

### 5.3 Dropping a global temporary view

```python
spark.catalog.dropGlobalTempView(
    "maintenance_records_global"
)
```

### 5.4 Temporary versus global temporary views

| Characteristic | Temporary view | Global temporary view |
|---|---|---|
| Scope | SparkSession | Spark application |
| SQL reference | View name | `global_temp.view_name` |
| Shared across sessions | No | Yes, within the application |
| Persistent storage | No | No |
| Lifetime | Session lifetime | Application lifetime |

Global temporary views are useful when multiple sessions within the same Spark application need access to a shared logical view.

They should not be confused with permanent database tables.

---

## 6. Basic SQL Operations

Spark SQL supports familiar relational SQL operations.

The following examples assume that a temporary view named `maintenance_records` has already been created.

### 6.1 SELECT

Retrieve specific columns:

```sql
SELECT
    inventory_number,
    maintenance_date,
    maintenance_cost
FROM maintenance_records;
```

Retrieve all columns:

```sql
SELECT *
FROM maintenance_records;
```

In production queries, explicitly selecting the required columns is usually preferable to using `SELECT *`.

### 6.2 WHERE

Filter records:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_cost > 3000;
```

Multiple conditions:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_cost > 2000
  AND maintenance_type = 'Preventive';
```

Using OR:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_type = 'Preventive'
   OR maintenance_type = 'Corrective';
```

### 6.3 DISTINCT

Retrieve unique values:

```sql
SELECT DISTINCT
    equipment_type
FROM equipment;
```

Distinct can also be applied to multiple columns:

```sql
SELECT DISTINCT
    inventory_number,
    maintenance_type
FROM maintenance_records;
```

Removing duplicates may require data redistribution, depending on the execution plan.

### 6.4 ORDER BY

Sort query results:

```sql
SELECT
    inventory_number,
    maintenance_cost
FROM maintenance_records
ORDER BY maintenance_cost DESC;
```

Multiple ordering columns:

```sql
SELECT *
FROM maintenance_records
ORDER BY
    equipment_type ASC,
    maintenance_date DESC;
```

### 6.5 LIMIT

Restrict the number of returned records:

```sql
SELECT *
FROM maintenance_records
LIMIT 10;
```

For consistent top-N results, combine `LIMIT` with a meaningful `ORDER BY`.

---

## 7. SQL Expressions and Calculated Columns

SQL allows new values to be calculated directly in a query.

### 7.1 Arithmetic expressions

```sql
SELECT
    inventory_number,
    maintenance_cost,
    maintenance_cost * 1.16 AS cost_with_tax
FROM maintenance_records;
```

### 7.2 Conditional expressions with CASE

```sql
SELECT
    inventory_number,
    maintenance_cost,
    CASE
        WHEN maintenance_cost IS NULL THEN 'UNKNOWN'
        WHEN maintenance_cost >= 5000 THEN 'HIGH'
        WHEN maintenance_cost >= 2000 THEN 'MEDIUM'
        ELSE 'LOW'
    END AS cost_category
FROM maintenance_records;
```

`CASE` provides conditional logic similar to `when()` and `otherwise()` in PySpark.

### 7.3 Casting values

```sql
SELECT
    CAST(manufacturing_year AS INT) AS manufacturing_year
FROM equipment;
```

Spark also supports the `CAST` shorthand:

```sql
SELECT
    maintenance_cost::DECIMAL(12, 2)
FROM maintenance_records;
```

The standard `CAST()` form is generally more portable across SQL systems.

### 7.4 String functions

```sql
SELECT
    UPPER(TRIM(equipment_type)) AS normalized_type
FROM equipment;
```

Other useful functions include:

| Function | Purpose |
|---|---|
| `UPPER()` | Converts text to uppercase |
| `LOWER()` | Converts text to lowercase |
| `TRIM()` | Removes surrounding whitespace |
| `CONCAT()` | Combines strings |
| `LENGTH()` | Returns string length |
| `SUBSTRING()` | Extracts part of a string |
| `REPLACE()` | Replaces matching text |

Example:

```sql
SELECT
    CONCAT(
        inventory_number,
        ' - ',
        equipment_type
    ) AS equipment_label
FROM equipment;
```

### 7.5 Numeric functions

```sql
SELECT
    ROUND(maintenance_cost, 2) AS rounded_cost,
    ABS(maintenance_cost) AS absolute_cost
FROM maintenance_records;
```

Other useful functions include:

- `CEIL()`
- `FLOOR()`
- `ROUND()`
- `ABS()`
- `POWER()`
- `SQRT()`

### 7.6 Date functions

Extract the year:

```sql
SELECT
    YEAR(maintenance_date) AS maintenance_year
FROM maintenance_records;
```

Extract the month:

```sql
SELECT
    MONTH(maintenance_date) AS maintenance_month
FROM maintenance_records;
```

Calculate elapsed days:

```sql
SELECT
    inventory_number,
    DATEDIFF(
        CURRENT_DATE(),
        maintenance_date
    ) AS days_since_maintenance
FROM maintenance_records;
```

Other useful date functions include:

| Function | Purpose |
|---|---|
| `CURRENT_DATE()` | Current date |
| `CURRENT_TIMESTAMP()` | Current timestamp |
| `YEAR()` | Extracts year |
| `MONTH()` | Extracts month |
| `DAY()` | Extracts day of month |
| `DATE_ADD()` | Adds days to a date |
| `DATE_SUB()` | Subtracts days from a date |
| `DATEDIFF()` | Calculates the difference in days |
| `TO_DATE()` | Converts a value into a date |

Date and timestamp interpretation should account for formats and session time-zone configuration.

---

## 8. Working with NULL Values

SQL uses `NULL` to represent missing or unknown information.

It is important to understand how SQL evaluates null values.

### 8.1 IS NULL

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_cost IS NULL;
```

### 8.2 IS NOT NULL

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_cost IS NOT NULL;
```

### 8.3 Why equality does not work with NULL

Incorrect:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_cost = NULL;
```

Correct:

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_cost IS NULL;
```

A comparison with `NULL` normally produces UNKNOWN rather than TRUE or FALSE.

### 8.4 COALESCE

Returns the first non-null expression.

```sql
SELECT
    inventory_number,
    COALESCE(
        maintenance_cost,
        0
    ) AS effective_cost
FROM maintenance_records;
```

The replacement value must have a valid business interpretation.

### 8.5 NULLIF

Returns NULL when two expressions are equal.

```sql
SELECT
    total_cost / NULLIF(maintenance_count, 0)
        AS average_cost
FROM equipment_summary;
```

This prevents division by zero when the count is zero.

### 8.6 COUNT and NULL

Different aggregate expressions treat null values differently.

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(maintenance_cost) AS rows_with_cost
FROM maintenance_records;
```

- `COUNT(*)` counts rows.
- `COUNT(column)` counts non-null values in that column.

This distinction is important when calculating data completeness.

---

## 9. Aggregations and GROUP BY

Aggregations summarize records.

### 9.1 COUNT

```sql
SELECT
    COUNT(*) AS total_maintenance_events
FROM maintenance_records;
```

### 9.2 SUM

```sql
SELECT
    SUM(maintenance_cost) AS total_maintenance_cost
FROM maintenance_records;
```

### 9.3 AVG

```sql
SELECT
    AVG(maintenance_cost) AS average_maintenance_cost
FROM maintenance_records;
```

### 9.4 MIN and MAX

```sql
SELECT
    MIN(maintenance_cost) AS minimum_cost,
    MAX(maintenance_cost) AS maximum_cost
FROM maintenance_records;
```

### 9.5 GROUP BY

Calculate maintenance totals by equipment type:

```sql
SELECT
    equipment_type,
    COUNT(*) AS maintenance_events,
    SUM(maintenance_cost) AS total_cost,
    AVG(maintenance_cost) AS average_cost
FROM equipment
INNER JOIN maintenance_records
    ON equipment.inventory_number =
       maintenance_records.inventory_number
GROUP BY equipment_type;
```

### 9.6 HAVING

`HAVING` filters groups after aggregation.

```sql
SELECT
    equipment_type,
    SUM(maintenance_cost) AS total_cost
FROM equipment
INNER JOIN maintenance_records
    ON equipment.inventory_number =
       maintenance_records.inventory_number
GROUP BY equipment_type
HAVING SUM(maintenance_cost) > 5000;
```

### 9.7 WHERE versus HAVING

| Clause | Purpose |
|---|---|
| `WHERE` | Filters individual rows before grouping |
| `HAVING` | Filters aggregated groups |

Example:

```sql
SELECT
    equipment_type,
    SUM(maintenance_cost) AS total_cost
FROM equipment
INNER JOIN maintenance_records
    ON equipment.inventory_number =
       maintenance_records.inventory_number
WHERE maintenance_date >= DATE '2026-01-01'
GROUP BY equipment_type
HAVING SUM(maintenance_cost) > 5000;
```

The `WHERE` clause restricts the maintenance records considered.

The `HAVING` clause filters the resulting equipment-type groups.

---

## 10. SQL Joins

Joins combine information from multiple relations.

Assume two views:

- `equipment`
- `maintenance_records`

Both contain `inventory_number`.

### 10.1 INNER JOIN

Returns matching records.

```sql
SELECT
    e.inventory_number,
    e.equipment_type,
    m.maintenance_date,
    m.maintenance_cost
FROM equipment AS e
INNER JOIN maintenance_records AS m
    ON e.inventory_number = m.inventory_number;
```

### 10.2 LEFT JOIN

Returns all records from the left relation and matching records from the right relation.

```sql
SELECT
    e.inventory_number,
    e.equipment_type,
    m.maintenance_date,
    m.maintenance_cost
FROM equipment AS e
LEFT JOIN maintenance_records AS m
    ON e.inventory_number = m.inventory_number;
```

This is useful when every equipment unit must appear even if it has no maintenance history.

### 10.3 RIGHT JOIN

```sql
SELECT
    e.inventory_number,
    m.maintenance_date,
    m.maintenance_cost
FROM equipment AS e
RIGHT JOIN maintenance_records AS m
    ON e.inventory_number = m.inventory_number;
```

### 10.4 FULL OUTER JOIN

```sql
SELECT
    e.inventory_number AS equipment_inventory_number,
    m.inventory_number AS maintenance_inventory_number,
    m.maintenance_cost
FROM equipment AS e
FULL OUTER JOIN maintenance_records AS m
    ON e.inventory_number = m.inventory_number;
```

### 10.5 LEFT SEMI JOIN

Returns records from the left relation that have a match in the right relation.

```sql
SELECT
    e.*
FROM equipment AS e
LEFT SEMI JOIN maintenance_records AS m
    ON e.inventory_number = m.inventory_number;
```

### 10.6 LEFT ANTI JOIN

Returns records from the left relation without a match in the right relation.

```sql
SELECT
    e.*
FROM equipment AS e
LEFT ANTI JOIN maintenance_records AS m
    ON e.inventory_number = m.inventory_number;
```

This is useful for identifying equipment units without maintenance records.

### 10.7 Join cardinality

A join can produce more rows than either input relation.

Suppose:

- Equipment contains one row for `EQ-001`.
- Maintenance contains three records for `EQ-001`.

Joining these relations produces three rows for that equipment unit.

This is not necessarily duplication.

It reflects the one-to-many relationship between equipment and maintenance events.

Always understand the expected relationship before interpreting join results.

---

## 11. Subqueries

A subquery is a query nested inside another query.

It can be used in:

- The `FROM` clause.
- The `WHERE` clause.
- The `SELECT` clause, where supported.
- Other SQL expressions.

### 11.1 Subquery in FROM

```sql
SELECT
    equipment_type,
    total_cost
FROM (
    SELECT
        e.equipment_type,
        SUM(m.maintenance_cost) AS total_cost
    FROM equipment AS e
    INNER JOIN maintenance_records AS m
        ON e.inventory_number = m.inventory_number
    GROUP BY e.equipment_type
) AS equipment_costs
WHERE total_cost > 5000;
```

The inner query calculates the cost by equipment type.

The outer query filters the calculated results.

### 11.2 Subquery with IN

```sql
SELECT *
FROM equipment
WHERE inventory_number IN (
    SELECT inventory_number
    FROM maintenance_records
);
```

This returns equipment units that appear in the maintenance records.

### 11.3 Subquery with EXISTS

```sql
SELECT *
FROM equipment AS e
WHERE EXISTS (
    SELECT 1
    FROM maintenance_records AS m
    WHERE m.inventory_number = e.inventory_number
);
```

`EXISTS` checks whether the subquery produces at least one matching row.

### 11.4 NOT EXISTS

```sql
SELECT *
FROM equipment AS e
WHERE NOT EXISTS (
    SELECT 1
    FROM maintenance_records AS m
    WHERE m.inventory_number = e.inventory_number
);
```

This identifies equipment units without maintenance records.

`NOT EXISTS` is often preferable to `NOT IN` when null values may appear in the subquery result, because null semantics can make `NOT IN` behave unexpectedly.

---

## 12. Common Table Expressions (CTEs)

A **Common Table Expression (CTE)** is a named temporary result defined within a SQL statement.

CTEs improve query organization and readability.

They are introduced using `WITH`.

### 12.1 Basic CTE

```sql
WITH equipment_costs AS (
    SELECT
        inventory_number,
        SUM(maintenance_cost) AS total_cost
    FROM maintenance_records
    GROUP BY inventory_number
)

SELECT *
FROM equipment_costs;
```

The CTE is available to the statement that defines it.

It does not automatically create a persistent table or a session-level view.

### 12.2 Multiple CTEs

```sql
WITH maintenance_totals AS (
    SELECT
        inventory_number,
        COUNT(*) AS event_count,
        SUM(maintenance_cost) AS total_cost
    FROM maintenance_records
    GROUP BY inventory_number
),

equipment_information AS (
    SELECT
        inventory_number,
        equipment_type,
        manufacturing_year
    FROM equipment
)

SELECT
    e.inventory_number,
    e.equipment_type,
    e.manufacturing_year,
    COALESCE(m.event_count, 0) AS event_count,
    m.total_cost
FROM equipment_information AS e
LEFT JOIN maintenance_totals AS m
    ON e.inventory_number = m.inventory_number;
```

This query separates the aggregation from the equipment information.

### 12.3 Why CTEs are useful

CTEs help with:

- Organizing complex SQL.
- Reusing named query results within a statement.
- Separating logical processing stages.
- Improving readability.
- Simplifying debugging.

A CTE is not necessarily materialized as a separate physical dataset.

Spark may optimize the entire query as one logical plan.

---

## 13. Window Functions in Spark SQL

Window functions calculate values across related rows without collapsing the original records into one row per group.

They are important for analytical queries.

### 13.1 ROW_NUMBER()

Assigns a sequential number within a window.

```sql
SELECT
    inventory_number,
    maintenance_date,
    maintenance_cost,
    ROW_NUMBER() OVER (
        PARTITION BY inventory_number
        ORDER BY maintenance_date DESC
    ) AS row_number
FROM maintenance_records;
```

### 13.2 Finding the latest maintenance event

```sql
WITH ranked_maintenance AS (
    SELECT
        inventory_number,
        maintenance_date,
        maintenance_type,
        maintenance_cost,
        ROW_NUMBER() OVER (
            PARTITION BY inventory_number
            ORDER BY
                maintenance_date DESC,
                maintenance_id DESC
        ) AS row_number
    FROM maintenance_records
)

SELECT
    inventory_number,
    maintenance_date,
    maintenance_type,
    maintenance_cost
FROM ranked_maintenance
WHERE row_number = 1;
```

The query:

1. Groups records by inventory number.
2. Orders each group by maintenance date.
3. Assigns row numbers.
4. Selects the first record.

For reliable results, the ordering should contain a suitable tie-breaker when multiple records share the same date.

### 13.3 RANK()

```sql
SELECT
    inventory_number,
    maintenance_cost,
    RANK() OVER (
        ORDER BY maintenance_cost DESC
    ) AS cost_rank
FROM maintenance_records;
```

Tied values receive the same rank, and subsequent ranks may contain gaps.

### 13.4 DENSE_RANK()

```sql
SELECT
    inventory_number,
    maintenance_cost,
    DENSE_RANK() OVER (
        ORDER BY maintenance_cost DESC
    ) AS cost_rank
FROM maintenance_records;
```

Tied values receive the same rank without gaps in subsequent ranking positions.

### 13.5 LAG()

Retrieves a value from a preceding row in the window.

```sql
SELECT
    inventory_number,
    maintenance_date,
    maintenance_cost,
    LAG(maintenance_cost) OVER (
        PARTITION BY inventory_number
        ORDER BY maintenance_date
    ) AS previous_cost
FROM maintenance_records;
```

### 13.6 LEAD()

Retrieves a value from a subsequent row.

```sql
SELECT
    inventory_number,
    maintenance_date,
    maintenance_cost,
    LEAD(maintenance_cost) OVER (
        PARTITION BY inventory_number
        ORDER BY maintenance_date
    ) AS next_cost
FROM maintenance_records;
```

### 13.7 Running totals

```sql
SELECT
    inventory_number,
    maintenance_date,
    maintenance_cost,
    SUM(maintenance_cost) OVER (
        PARTITION BY inventory_number
        ORDER BY maintenance_date
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    ) AS cumulative_cost
FROM maintenance_records;
```

The result calculates cumulative maintenance costs for each equipment unit.

---

## 14. Combining Spark SQL and PySpark

One of the most useful characteristics of Spark is the ability to combine SQL statements with DataFrame operations.

Both interfaces work with the same structured data processing engine.

### 14.1 DataFrame to SQL

Create a DataFrame:

```python
maintenance_df = spark.read.parquet(
    "data/maintenance"
)
```

Register it as a view:

```python
maintenance_df.createOrReplaceTempView(
    "maintenance_records"
)
```

Query it using SQL:

```python
summary_df = spark.sql("""
    SELECT
        inventory_number,
        SUM(maintenance_cost) AS total_cost
    FROM maintenance_records
    GROUP BY inventory_number
""")
```

### 14.2 SQL to DataFrame transformations

Continue using the DataFrame API:

```python
from pyspark.sql.functions import col

high_cost_equipment = (
    summary_df
    .filter(col("total_cost") > 5000)
    .orderBy(col("total_cost").desc())
)
```

### 14.3 DataFrame to SQL again

Register the result:

```python
high_cost_equipment.createOrReplaceTempView(
    "high_cost_equipment"
)
```

Then query it:

```python
spark.sql("""
    SELECT *
    FROM high_cost_equipment
    ORDER BY total_cost DESC
""").show()
```

This approach allows teams to use SQL for relational transformations and PySpark for application logic and workflow orchestration.

### 14.4 When to use each interface

| Situation | Recommended interface |
|---|---|
| Complex relational query | Spark SQL |
| Familiar SQL-based transformations | Spark SQL |
| Python application logic | PySpark |
| Dynamic DataFrame composition | DataFrame API |
| Reusable Python functions | PySpark |
| Analytical window queries | Either SQL or DataFrame API |
| Data pipeline orchestration | PySpark with SQL as needed |

The two interfaces are complementary rather than competing technologies.

---

## 15. SQL Functions and Built-In Capabilities

Spark SQL provides a large set of built-in functions.

These functions can be used directly in SQL statements.

### 15.1 String functions

```sql
SELECT
    UPPER(equipment_type) AS equipment_type,
    LOWER(operational_status) AS operational_status,
    TRIM(inventory_number) AS inventory_number
FROM equipment;
```

### 15.2 Numeric functions

```sql
SELECT
    ROUND(maintenance_cost, 2) AS rounded_cost,
    ABS(maintenance_cost) AS absolute_cost
FROM maintenance_records;
```

### 15.3 Date functions

```sql
SELECT
    maintenance_date,
    YEAR(maintenance_date) AS year,
    MONTH(maintenance_date) AS month,
    DAY(maintenance_date) AS day
FROM maintenance_records;
```

### 15.4 Conditional functions

```sql
SELECT
    inventory_number,
    IF(
        maintenance_cost > 3000,
        'HIGH',
        'NORMAL'
    ) AS cost_status
FROM maintenance_records;
```

`CASE WHEN` is often preferable when several conditions are required.

### 15.5 Array and collection functions

Spark SQL also supports complex data operations.

Examples include:

- `SIZE()`
- `ARRAY_CONTAINS()`
- `EXPLODE()`
- `POSEXPLODE()`
- `ELEMENT_AT()`
- `MAP_KEYS()`
- `MAP_VALUES()`

For example, consider a JSON-derived column containing an array of maintenance codes.

```sql
SELECT
    inventory_number,
    EXPLODE(maintenance_codes) AS maintenance_code
FROM maintenance_json;
```

`EXPLODE()` generates a separate output row for each array element.

This is useful when processing nested data.

---

## 16. Query Execution and Lazy Evaluation

Spark SQL follows the same general lazy execution principles as the DataFrame API.

Creating a query result does not necessarily mean that all underlying data has already been processed.

### 16.1 Defining a query

```python
result_df = spark.sql("""
    SELECT
        equipment_type,
        SUM(maintenance_cost) AS total_cost
    FROM equipment
    INNER JOIN maintenance_records
        ON equipment.inventory_number =
           maintenance_records.inventory_number
    GROUP BY equipment_type
""")
```

Spark constructs a query plan.

### 16.2 Triggering execution

```python
result_df.show()
```

The action triggers the required computation.

Other actions include:

```python
result_df.count()
```

```python
result_df.collect()
```

```python
result_df.write.mode("overwrite").parquet(
    "output/equipment_cost_summary"
)
```

### 16.3 Why lazy execution matters

Lazy evaluation gives Spark an opportunity to optimize multiple operations together.

For example, Spark may be able to:

- Remove unnecessary columns.
- Push filters toward a data source.
- Simplify expressions.
- Choose an efficient join strategy.
- Reduce intermediate data movement.

This is one of the reasons Spark SQL can optimize structured queries.

---

## 17. Understanding Spark SQL Execution Plans

Execution plans help explain how Spark intends to process a query.

### 17.1 explain()

```python
result_df.explain()
```

### 17.2 Extended plan

```python
result_df.explain("extended")
```

This can display the parsed, analyzed, optimized, and physical plans.

### 17.3 Formatted plan

```python
result_df.explain("formatted")
```

This provides a more structured view of the physical plan.

### 17.4 Example query

```python
query_df = spark.sql("""
    SELECT
        inventory_number,
        SUM(maintenance_cost) AS total_cost
    FROM maintenance_records
    WHERE maintenance_cost > 1000
    GROUP BY inventory_number
""")
```

Inspect the plan:

```python
query_df.explain("formatted")
```

### 17.5 Operations to recognize

| Operation | Meaning |
|---|---|
| Scan | Reads data from a source |
| Filter | Applies a condition |
| Project | Selects or calculates columns |
| HashAggregate | Performs aggregation using a hash-based strategy |
| SortAggregate | Performs aggregation involving sorting |
| Exchange | Redistributes data, commonly associated with a shuffle |
| BroadcastExchange | Prepares data for broadcast distribution |
| BroadcastHashJoin | Performs a broadcast hash join |
| SortMergeJoin | Performs a sort-merge join |
| Window | Executes window calculations |

The exact operations depend on Spark version, query structure, data source, and configuration.

### 17.6 Exchange and shuffle

An `Exchange` operation often indicates redistribution between partitions.

For example, a grouped aggregation may require records with the same grouping key to be processed together.

A shuffle can be expensive because it may involve:

- Network transfer.
- Serialization.
- Disk operations.
- Memory pressure.

The presence of a shuffle is not automatically a problem. Many correct distributed operations require data redistribution.

The goal is to understand whether the operation is necessary and whether its cost is reasonable.

---

## 18. Spark SQL and External Data Sources

Spark SQL can query data from different storage systems.

### 18.1 Creating a view from Parquet

```python
parquet_df = spark.read.parquet(
    "data/maintenance_parquet"
)

parquet_df.createOrReplaceTempView(
    "maintenance_records"
)
```

### 18.2 Creating a view from CSV

```python
csv_df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("data/maintenance.csv")
)

csv_df.createOrReplaceTempView(
    "maintenance_csv"
)
```

### 18.3 Creating a view from JSON

```python
json_df = spark.read.json(
    "data/maintenance.json"
)

json_df.createOrReplaceTempView(
    "maintenance_json"
)
```

### 18.4 Querying different sources through SQL

Once registered, the views can be queried using SQL.

```sql
SELECT
    inventory_number,
    maintenance_cost
FROM maintenance_records;
```

Spark SQL separates the relational query interface from the underlying storage format.

The physical data may come from different sources.

---

## 19. Catalogs, Databases, and Tables

Spark provides catalog functionality for discovering and managing metadata about databases, tables, and views.

A catalog is not the same thing as the underlying storage system.

It provides metadata and namespace organization.

### 19.1 Show databases

```sql
SHOW DATABASES;
```

### 19.2 Create a database

```sql
CREATE DATABASE IF NOT EXISTS fleet_analytics;
```

### 19.3 Select a database

```sql
USE fleet_analytics;
```

### 19.4 Show tables

```sql
SHOW TABLES;
```

### 19.5 Describe a table

```sql
DESCRIBE maintenance_records;
```

### 19.6 Create a managed table from a query

```sql
CREATE TABLE maintenance_summary
USING PARQUET
AS
SELECT
    inventory_number,
    SUM(maintenance_cost) AS total_cost
FROM maintenance_records
GROUP BY inventory_number;
```

The table's data and metadata management depend on the configured catalog and table type.

A managed table is generally managed by the catalog's storage conventions.

### 19.7 Create an external table

```sql
CREATE TABLE external_maintenance
USING PARQUET
LOCATION '/data/external/maintenance';
```

An external table references data at a specified storage location.

The exact lifecycle and deletion behavior depend on the catalog and table implementation.

### 19.8 Important distinction

| Object | Main purpose |
|---|---|
| DataFrame | Structured processing abstraction |
| Temporary view | Session-scoped SQL name |
| Global temporary view | Application-scoped SQL name |
| Permanent view | Stored query definition in a catalog |
| Table | Catalog-managed relation associated with data |
| Database | Namespace for organizing relations |
| Catalog | Metadata management interface |

---

## 20. SQL Security and Safe Query Construction

SQL queries can become security risks when untrusted input is inserted directly into query strings.

### 20.1 Unsafe string interpolation

```python
equipment_type = user_input

query = f"""
    SELECT *
    FROM equipment
    WHERE equipment_type = '{equipment_type}'
"""

result_df = spark.sql(query)
```

If `user_input` contains malicious SQL syntax, the resulting statement may behave differently from what the application intended.

This is a form of SQL injection risk.

### 20.2 Prefer parameterized SQL where supported

Spark SQL provides parameter markers for supported versions and APIs.

Example:

```python
result_df = spark.sql(
    """
    SELECT *
    FROM equipment
    WHERE equipment_type = :equipment_type
    """,
    args={"equipment_type": "Excavator"}
)
```

Parameter support depends on the Spark version and API being used.

For environments without this capability, use a safe construction strategy appropriate to the application.

### 20.3 DataFrame filtering as an alternative

```python
result_df = equipment_df.filter(
    col("equipment_type") == equipment_type
)
```

This avoids constructing a SQL statement from untrusted input.

### 20.4 Security principles

- Do not concatenate untrusted input into SQL statements.
- Validate input types and allowed values.
- Protect credentials.
- Apply least-privilege access.
- Restrict access to sensitive datasets.
- Avoid exposing confidential data in logs.
- Review permissions for catalogs and storage locations.

---

## 21. Performance Best Practices for Spark SQL

Spark SQL optimization begins with understanding the workload.

### 21.1 Avoid SELECT *

Instead of:

```sql
SELECT *
FROM maintenance_records;
```

Prefer:

```sql
SELECT
    inventory_number,
    maintenance_date,
    maintenance_cost
FROM maintenance_records;
```

Selecting only required columns can reduce data scanning and processing.

### 21.2 Filter early

```sql
SELECT
    inventory_number,
    maintenance_cost
FROM maintenance_records
WHERE maintenance_date >= DATE '2026-01-01';
```

Early filtering can reduce the amount of data processed by subsequent operations.

Spark may push supported filters toward the data source.

### 21.3 Use suitable storage formats

For repeated analytical workloads, Parquet is often preferable to CSV because it supports columnar access and compression.

### 21.4 Avoid unnecessary DISTINCT

```sql
SELECT DISTINCT
    equipment_type
FROM equipment;
```

This is valid when unique values are required.

However, `DISTINCT` may require a shuffle.

Do not use it as a general method for hiding unexpected duplicates.

### 21.5 Understand join strategies

A join between a large dataset and a genuinely small dataset may benefit from broadcasting.

Inspect the physical plan to understand which strategy Spark selected.

### 21.6 Use partition pruning

If data is stored in partitions such as:

```text
maintenance/
    maintenance_year=2024/
    maintenance_year=2025/
    maintenance_year=2026/
```

A query filtering on the partition column may allow Spark to avoid reading irrelevant partitions.

```sql
SELECT *
FROM maintenance_records
WHERE maintenance_year = 2026;
```

Partition pruning depends on the data source and how the partition information is exposed.

### 21.7 Use statistics when available

Spark can use table and column statistics to improve planning decisions.

The availability and usefulness of statistics depend on the catalog and data source.

### 21.8 Avoid unnecessary nested queries

CTEs and subqueries improve readability, but excessive query complexity can make debugging and plan interpretation harder.

Use them to express logical stages clearly.

### 21.9 Inspect execution plans

```python
result_df.explain("formatted")
```

Use the plan to investigate:

- Large scans.
- Unnecessary exchanges.
- Join strategy.
- Repeated calculations.
- Unexpected data movement.
- Missing filter pushdown.

### 21.10 Performance checklist

Before deploying a Spark SQL pipeline, ask:

- Does the query select only required columns?
- Are filters applied appropriately?
- Is the join relationship understood?
- Are null values handled correctly?
- Are aggregations necessary?
- Does the query produce excessive duplicate rows?
- Is the physical plan reasonable?
- Are storage partitions useful for the expected queries?
- Are there unnecessary shuffles?
- Is the output format appropriate for downstream consumers?

---

# 22. Complete Practical Example: Fleet Maintenance Analytics with Spark SQL

This example builds an analytical workflow using SQL over Spark DataFrames.

The data is illustrative. It does not represent the actual production schema of any operational system.

## 22.1 Create a SparkSession

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("FleetSparkSQLAnalytics")
    .master("local[*]")
    .getOrCreate()
)
```

## 22.2 Create the equipment DataFrame

```python
equipment_data = [
    ("EQ-001", "Excavator", 2020, "Active"),
    ("EQ-002", "Motor Grader", 2021, "Active"),
    ("EQ-003", "Pickup", 2023, "Active"),
    ("EQ-004", "Excavator", 2019, "Inactive"),
    ("EQ-005", "Dump Truck", 2022, "Active")
]

equipment_columns = [
    "inventory_number",
    "equipment_type",
    "manufacturing_year",
    "operational_status"
]

equipment_df = spark.createDataFrame(
    equipment_data,
    equipment_columns
)
```

## 22.3 Create the maintenance DataFrame

```python
from datetime import date

maintenance_data = [
    ("M-001", "EQ-001", date(2026, 1, 10), "Preventive", 2500.00),
    ("M-002", "EQ-001", date(2026, 3, 15), "Corrective", 4800.00),
    ("M-003", "EQ-002", date(2026, 2, 20), "Preventive", 1800.00),
    ("M-004", "EQ-003", date(2026, 4, 5), "Corrective", 3200.00),
    ("M-005", "EQ-001", date(2026, 5, 10), "Preventive", 2100.00),
    ("M-006", "EQ-005", date(2026, 6, 12), "Corrective", 5500.00)
]

maintenance_columns = [
    "maintenance_id",
    "inventory_number",
    "maintenance_date",
    "maintenance_type",
    "maintenance_cost"
]

maintenance_df = spark.createDataFrame(
    maintenance_data,
    maintenance_columns
)
```

## 22.4 Register temporary views

```python
equipment_df.createOrReplaceTempView(
    "equipment"
)

maintenance_df.createOrReplaceTempView(
    "maintenance_records"
)
```

The two DataFrames are now accessible through SQL.

## 22.5 Inspect the available data

```python
spark.sql("""
    SELECT *
    FROM equipment
""").show()
```

```python
spark.sql("""
    SELECT *
    FROM maintenance_records
""").show()
```

## 22.6 Calculate total maintenance costs

```python
total_cost_df = spark.sql("""
    SELECT
        SUM(maintenance_cost) AS total_maintenance_cost,
        AVG(maintenance_cost) AS average_maintenance_cost,
        COUNT(*) AS maintenance_events
    FROM maintenance_records
""")
```

```python
total_cost_df.show()
```

## 22.7 Calculate costs by equipment type

```python
equipment_type_costs = spark.sql("""
    SELECT
        e.equipment_type,
        COUNT(m.maintenance_id) AS maintenance_events,
        SUM(m.maintenance_cost) AS total_cost,
        AVG(m.maintenance_cost) AS average_cost
    FROM equipment AS e
    INNER JOIN maintenance_records AS m
        ON e.inventory_number = m.inventory_number
    GROUP BY e.equipment_type
    ORDER BY total_cost DESC
""")
```

```python
equipment_type_costs.show()
```

## 22.8 Identify equipment without maintenance records

```python
equipment_without_maintenance = spark.sql("""
    SELECT
        e.inventory_number,
        e.equipment_type,
        e.operational_status
    FROM equipment AS e
    LEFT ANTI JOIN maintenance_records AS m
        ON e.inventory_number = m.inventory_number
""")
```

```python
equipment_without_maintenance.show()
```

## 22.9 Identify the latest maintenance event

```python
latest_maintenance = spark.sql("""
    WITH ranked_records AS (
        SELECT
            maintenance_id,
            inventory_number,
            maintenance_date,
            maintenance_type,
            maintenance_cost,
            ROW_NUMBER() OVER (
                PARTITION BY inventory_number
                ORDER BY
                    maintenance_date DESC,
                    maintenance_id DESC
            ) AS row_number
        FROM maintenance_records
    )

    SELECT
        maintenance_id,
        inventory_number,
        maintenance_date,
        maintenance_type,
        maintenance_cost
    FROM ranked_records
    WHERE row_number = 1
""")
```

```python
latest_maintenance.show()
```

## 22.10 Create a complete equipment status dataset

```python
latest_maintenance.createOrReplaceTempView(
    "latest_maintenance"
)

fleet_status = spark.sql("""
    SELECT
        e.inventory_number,
        e.equipment_type,
        e.manufacturing_year,
        e.operational_status,
        m.maintenance_date AS latest_maintenance_date,
        m.maintenance_type AS latest_maintenance_type,
        m.maintenance_cost AS latest_maintenance_cost
    FROM equipment AS e
    LEFT JOIN latest_maintenance AS m
        ON e.inventory_number = m.inventory_number
""")
```

```python
fleet_status.show()
```

## 22.11 Add an analytical classification

```python
fleet_status.createOrReplaceTempView(
    "fleet_status"
)

classified_fleet = spark.sql("""
    SELECT
        inventory_number,
        equipment_type,
        operational_status,
        latest_maintenance_date,
        latest_maintenance_cost,
        CASE
            WHEN latest_maintenance_date IS NULL
                THEN 'NO MAINTENANCE HISTORY'
            WHEN latest_maintenance_cost >= 5000
                THEN 'HIGH COST'
            WHEN latest_maintenance_cost >= 2000
                THEN 'MEDIUM COST'
            ELSE 'LOW COST'
        END AS maintenance_category
    FROM fleet_status
""")
```

```python
classified_fleet.show()
```

## 22.12 Write the final analytical dataset

```python
classified_fleet.write \
    .mode("overwrite") \
    .parquet("output/fleet_status")
```

## 22.13 Inspect the execution plan

```python
classified_fleet.explain("formatted")
```

Review the plan to identify the scans, joins, projections, and other operations required by the query.

---

# 23. Practical Architecture: Where Spark SQL Fits

Spark SQL is especially valuable in data engineering architectures where structured data must be transformed into analytical datasets.

A simplified architecture may look like this:

```text
Operational Systems
        |
        v
   Data Ingestion
        |
        v
    Raw Storage
        |
        v
  Spark Processing
        |
        v
     Spark SQL
        |
        v
  Data Transformation
        |
        v
   Curated Storage
        |
        +----------------+
        |                |
        v                v
  Data Warehouse     BI Platform
        |
        v
  Analytical Reports
```

### 23.1 Operational systems

These systems generate business and operational records.

Examples:

- Fleet management applications.
- ERP platforms.
- Financial systems.
- Maintenance applications.
- Customer management systems.

### 23.2 Raw storage

Incoming information is stored in its original or minimally transformed form.

Common storage formats include:

- CSV.
- JSON.
- Parquet.
- Database extracts.

### 23.3 Spark processing

Spark reads the data and performs distributed processing.

### 23.4 Spark SQL transformations

SQL is used to:

- Standardize fields.
- Filter invalid records.
- Combine datasets.
- Calculate business metrics.
- Produce analytical views.
- Create curated datasets.

### 23.5 Curated storage

The processed datasets are stored in formats suitable for downstream use.

For example:

- Parquet files.
- Managed tables.
- Lakehouse tables.
- Data warehouse tables.

### 23.6 Consumption

The resulting datasets can support:

- Business intelligence.
- Reporting.
- Machine learning.
- Financial analysis.
- Operational analytics.

---

# 24. Common Mistakes

| Mistake | Why it is problematic | Better practice |
|---|---|---|
| Assuming every DataFrame is automatically a SQL table | SQL needs a registered relation or catalog object | Create a view or use a table |
| Confusing temporary views with persistent tables | Temporary views do not automatically persist data | Understand object scope and storage |
| Using `SELECT *` in every query | Processes unnecessary columns | Select required fields |
| Using `= NULL` | Does not correctly test missing values | Use `IS NULL` |
| Using WHERE for aggregate filtering | Aggregate conditions belong after grouping | Use HAVING |
| Ignoring join cardinality | Can multiply records unexpectedly | Understand key relationships |
| Assuming CTEs are materialized | A CTE is not necessarily stored separately | Inspect the execution plan |
| Using `NOT IN` with nullable subqueries | Null semantics can produce unexpected results | Consider `NOT EXISTS` |
| Ignoring tie-breaking in ROW_NUMBER | Latest-record selection may be nondeterministic | Define a complete ordering |
| Concatenating untrusted input into SQL | Creates SQL injection risks | Use parameterization or safe expressions |
| Assuming SQL execution is always immediate | Spark uses lazy evaluation | Trigger actions deliberately |
| Ignoring the physical plan | Makes performance issues difficult to identify | Use `explain()` |

---

# 25. Exercises

## Exercise 1: Temporary views

Create a DataFrame containing equipment information.

Requirements:

1. Register it as a temporary view.
2. Query the view using `spark.sql()`.
3. Display all records.
4. Select only inventory number and equipment type.
5. Drop the temporary view.

## Exercise 2: Basic SQL operations

Using a maintenance view:

1. Retrieve all records.
2. Select three columns.
3. Filter costs greater than 3000.
4. Sort costs in descending order.
5. Retrieve the first five records.
6. Select distinct maintenance types.

## Exercise 3: Conditional logic

Create a query that categorizes maintenance costs:

- HIGH: at least 5000.
- MEDIUM: at least 2000.
- LOW: below 2000.
- UNKNOWN: missing cost.

Use `CASE WHEN`.

## Exercise 4: Aggregations

Calculate:

1. Total maintenance events.
2. Total maintenance cost.
3. Average maintenance cost.
4. Highest maintenance cost.
5. Number of records with a non-null cost.
6. Total cost by equipment type.

Use `GROUP BY` and `HAVING`.

## Exercise 5: Joins

Create equipment and maintenance views.

Perform:

1. INNER JOIN.
2. LEFT JOIN.
3. LEFT SEMI JOIN.
4. LEFT ANTI JOIN.
5. FULL OUTER JOIN.

Explain which records appear in each result.

## Exercise 6: Common Table Expressions

Create a CTE that:

1. Calculates total maintenance cost per inventory number.
2. Joins the result with equipment information.
3. Identifies equipment with total maintenance cost above 5000.
4. Sorts the results by total cost.

## Exercise 7: Window functions

Write a SQL query that:

1. Partitions maintenance records by inventory number.
2. Orders events by maintenance date.
3. Calculates `ROW_NUMBER()`.
4. Identifies the latest event.
5. Calculates the previous maintenance cost using `LAG()`.

## Exercise 8: SQL and PySpark integration

Create a workflow that:

1. Reads a Parquet dataset.
2. Registers it as a temporary view.
3. Executes a SQL aggregation.
4. Applies a PySpark filter.
5. Registers the resulting DataFrame as another view.
6. Executes a second SQL query.
7. Writes the final dataset to Parquet.

## Exercise 9: Query optimization

Create a query involving:

- A filter.
- A join.
- A grouping operation.
- A calculated column.

Inspect the physical plan using:

```python
result_df.explain("formatted")
```

Identify:

1. The scan operation.
2. The filter operation.
3. The join strategy.
4. Any exchange operations.
5. The aggregation strategy.

---

# 26. Knowledge Check

**1. What is Spark SQL primarily designed for?**

A. Managing operating system processes.  
B. Processing structured data through SQL and DataFrame APIs.  
C. Replacing Python.  
D. Managing network connections.

**2. What does `spark.sql()` return?**

A. A Python dictionary.  
B. A Spark DataFrame.  
C. A CSV file.  
D. A database connection.

**3. What is required to query an ordinary DataFrame by a SQL name?**

A. An index.  
B. A temporary view or another catalog relation.  
C. A Python list.  
D. A separate Spark cluster.

**4. What is the scope of a local temporary view?**

A. The SparkSession.  
B. Every Spark application.  
C. Every database server.  
D. The entire file system.

**5. Which SQL clause filters aggregated groups?**

A. WHERE.  
B. ORDER BY.  
C. HAVING.  
D. LIMIT.

**6. Which expression correctly tests for missing values?**

A. `column = NULL`  
B. `column IS NULL`  
C. `column == NULL`  
D. `column EQUALS NULL`

**7. What is a CTE?**

A. A named query result defined within a SQL statement.  
B. A Spark executor.  
C. A storage format.  
D. A database index.

**8. Which function assigns sequential numbers within a window?**

A. `SUM()`  
B. `ROW_NUMBER()`  
C. `COUNT()`  
D. `COALESCE()`

**9. What does the Catalyst optimizer do?**

A. Manages user passwords.  
B. Optimizes logical query plans.  
C. Stores all data in memory.  
D. Creates operating system processes.

**10. What does `EXPLAIN` help you understand?**

A. The structure of the query execution plan.  
B. The user's network speed.  
C. The number of Python packages installed.  
D. The contents of a password file.

## Answer Key

| Question | Answer |
|---|---|
| 1 | B |
| 2 | B |
| 3 | B |
| 4 | A |
| 5 | C |
| 6 | B |
| 7 | A |
| 8 | B |
| 9 | B |
| 10 | A |

---

# 27. Technical Glossary

| Term | Definition |
|---|---|
| Spark SQL | Spark module for structured data processing through SQL and DataFrame APIs |
| SQL Parser | Component that parses SQL statements |
| Analyzer | Component that resolves tables, columns, functions, and types |
| Catalyst | Spark SQL query optimization framework |
| Logical Plan | Representation of the requested relational operations |
| Physical Plan | Representation of the execution strategies selected by Spark |
| Temporary View | Session-scoped SQL name for a relation |
| Global Temporary View | Application-scoped temporary SQL view |
| Catalog | Interface for metadata and relation management |
| Database | Namespace for organizing tables and views |
| Table | Catalog relation associated with stored or managed data |
| CTE | Named query result defined within a SQL statement |
| Subquery | Query nested inside another query |
| Predicate | Expression that evaluates a condition |
| Projection | Selection or calculation of output columns |
| Aggregation | Calculation that summarizes multiple records |
| GROUP BY | SQL clause that groups records |
| HAVING | SQL clause that filters aggregated groups |
| Join | Operation combining relations |
| Window Function | Function that calculates values across related rows |
| ROW_NUMBER | Function assigning sequential row numbers |
| RANK | Ranking function that leaves gaps after ties |
| DENSE_RANK | Ranking function that does not leave gaps after ties |
| LAG | Function retrieving a previous row's value |
| LEAD | Function retrieving a subsequent row's value |
| Predicate Pushdown | Applying supported filters closer to the data source |
| Column Pruning | Avoiding unnecessary column reads |
| Shuffle | Redistribution of data between partitions |
| Exchange | Physical-plan operation commonly associated with redistribution |
| Broadcast Join | Join strategy that distributes a small relation to executors |
| SQL Injection | Security vulnerability caused by unsafe SQL construction |
| Lazy Evaluation | Delayed execution of transformations until an action is triggered |

---

# 28. Related Concepts and Learning Path

### Previous concepts

- [[10 - Apache Spark Fundamentals]]
- [[11 - Apache Spark DataFrames]]
- [[04 - Data Pipelines]]
- [[03 - ETL and ELT]]
- [[05 - Data Warehouses]]

### Next concepts

- [[13 - Spark Transformations and Actions]]
- [[14 - Spark Joins and Aggregations]]
- [[15 - Spark Performance Optimization]]

### Conceptual relationships

```text
Apache Spark Fundamentals
          |
          v
     SparkSession
          |
          v
     Spark DataFrames
          |
          v
       Spark SQL
          |
          +-------------------------+
          |            |            |
          v            v            v
       SQL Views    SQL Queries   Catalog
          |            |
          v            v
      Relational   Query Analysis
      Operations       |
          |            v
          |         Catalyst
          |         Optimizer
          |            |
          +------------+
                       |
                       v
                Physical Planning
                       |
                       v
               Distributed Execution
                       |
                       v
                Curated Datasets
```

---

# 29. Key Takeaways

1. Spark SQL is the structured query processing module of Apache Spark.
2. It allows SQL statements to operate on distributed datasets.
3. Spark SQL and the DataFrame API share the same underlying structured processing infrastructure.
4. `SparkSession.sql()` executes SQL statements and returns DataFrames.
5. Temporary views expose DataFrames through SQL names without automatically persisting their data.
6. Global temporary views have application-level scope.
7. Spark SQL supports filtering, projection, grouping, aggregation, sorting, joins, subqueries, and CTEs.
8. Window functions enable ranking and historical analysis without collapsing records.
9. Catalyst analyzes and optimizes logical query plans.
10. Physical plans describe the execution strategies selected by Spark.
11. SQL queries follow lazy execution principles.
12. Query performance depends on data volume, storage format, partitioning, join strategy, and execution planning.
13. SQL security requires careful handling of user input and credentials.
14. Spark SQL is particularly useful for analytical processing and data engineering pipelines.

**Final principle:**

Spark SQL separates the description of a data transformation from its distributed execution. You express what result is required through SQL, while Spark analyzes, optimizes, and executes the corresponding operations across the available computing resources.
