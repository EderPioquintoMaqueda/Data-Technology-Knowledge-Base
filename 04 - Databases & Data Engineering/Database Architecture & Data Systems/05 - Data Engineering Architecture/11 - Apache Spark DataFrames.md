
---
title: "Apache Spark DataFrames"
aliases:
  - "PySpark DataFrames"
  - "Working with Spark DataFrames"
  - "Spark DataFrame Operations"
type: learning-note
domain: technology
category: data-engineering-architecture
subject: spark-dataframes
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12
tags:
  - apache-spark
  - pyspark
  - dataframes
  - data-engineering
  - distributed-data-processing
  - schema
  - data-transformation
  - data-cleaning
concepts:
  - spark-dataframe
  - spark-session
  - schema
  - structtype
  - structfield
  - spark-sql-data-types
  - transformations
  - actions
  - null-handling
  - data-cleaning
  - filtering
  - aggregation
  - joins
  - window-functions
  - parquet
  - lazy-evaluation
prerequisites:
  - "[[10 - Apache Spark Fundamentals]]"
  - "[[09 - Distributed Data Processing]]"
  - "[[04 - Data Pipelines]]"
related:
  - "[[12 - Spark SQL]]"
  - "[[13 - Spark Transformations and Actions]]"
  - "[[14 - Spark Joins and Aggregations]]"
  - "[[15 - Spark Performance Optimization]]"
  - "[[Data Quality]]"
  - "[[03 - ETL and ELT]]"
tools:
  - Apache Spark
  - PySpark
  - Python
  - SQL
  - Parquet
learning_objectives:
  - "Define Spark DataFrames and explain their distributed structure."
  - "Create DataFrames with explicit schemas."
  - "Inspect schemas and understand Spark SQL data types."
  - "Read and write common data formats."
  - "Select, filter, and derive columns."
  - "Handle null values, NaN values, and duplicates."
  - "Perform aggregations and joins."
  - "Use window functions for analytical tasks."
  - "Understand transformations, actions, and lazy execution."
  - "Apply practical performance best practices."
review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# Apache Spark DataFrames

## 1. Introduction

A **Spark DataFrame** is a distributed collection of data organized into named columns, similar to a relational database table.

It is one of the main abstractions used in Apache Spark for structured and semi-structured data processing.

A DataFrame combines three important characteristics:

1. **Distributed processing:** Data can be divided into partitions and processed across multiple executors.
2. **Structured representation:** Data is organized into columns with defined names and data types.
3. **Optimized execution:** Spark can analyze operations and optimize their execution through its query engine.

A Spark DataFrame is not simply a large Python list or a Pandas DataFrame.

It is a logical representation of distributed data that Spark can process across a cluster.

### 1.1 DataFrame mental model

Imagine a maintenance dataset:

| inventory_number | maintenance_date | maintenance_type | cost |
|---|---|---|---:|
| EQ-001 | 2026-01-10 | Preventive | 2500.00 |
| EQ-002 | 2026-01-15 | Corrective | 4800.00 |
| EQ-003 | 2026-01-20 | Preventive | 1800.00 |

In Spark, this information can be represented as a DataFrame.

```python
maintenance_df.show()
```

Conceptually, Spark treats this as a structured dataset with:

- Rows representing individual records.
- Columns representing attributes.
- A schema defining the structure.
- Partitions distributing the data.
- Transformations describing the operations to perform.

### 1.2 DataFrame versus Pandas DataFrame

Both abstractions organize data into rows and columns, but their execution models differ.

| Characteristic | Spark DataFrame | Pandas DataFrame |
|---|---|---|
| Processing model | Distributed | Usually single-machine |
| Execution | Lazy | Mostly eager |
| Data scale | Large distributed datasets | Data fitting available memory |
| Execution engine | Spark SQL engine | Python and optimized native libraries |
| Data distribution | Partitioned | Primarily local |
| Optimization | Catalyst and physical execution engine | Library-specific optimizations |
| Typical use | Distributed data engineering | Local analysis and data manipulation |

**Important:** A Spark DataFrame is not necessarily stored entirely in memory. It can represent data read from files, databases, or other sources. Spark processes it according to its execution plan and storage configuration.

---

## 2. The SparkSession

The `SparkSession` is the primary entry point for working with Spark DataFrames.

It allows applications to:

- Create DataFrames.
- Read external datasets.
- Execute SQL queries.
- Configure Spark.
- Access catalog information.
- Write data to storage systems.

### 2.1 Creating a SparkSession

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("FleetDataEngineering")
    .master("local[*]")
    .getOrCreate()
)
```

Explanation:

| Method | Purpose |
|---|---|
| `builder` | Starts the session configuration process. |
| `appName()` | Assigns an application name. |
| `master()` | Specifies the execution environment. |
| `getOrCreate()` | Returns an existing session or creates one. |

The following configuration:

```python
.master("local[*]")
```

instructs Spark to run locally using the available processor cores.

In a cluster environment, the application is submitted through the appropriate cluster manager instead.

### 2.2 Inspecting the session

```python
print(spark.version)
```

```python
print(spark.sparkContext.appName)
```

```python
print(spark.conf.get("spark.sql.shuffle.partitions"))
```

The final example retrieves a Spark SQL configuration value.

### 2.3 SparkSession and SparkContext

These objects have different responsibilities.

- `SparkSession` provides the modern interface for structured data processing.
- `SparkContext` represents the underlying connection to Spark execution resources.

Most DataFrame applications should interact with `SparkSession` rather than manually managing the lower-level context.

---

## 3. Creating DataFrames

Spark provides several ways to create DataFrames.

The most common approaches include:

- Python collections.
- Explicit schemas.
- Row objects.
- Existing RDDs.
- External data sources.

### 3.1 Creating a DataFrame from Python data

```python
data = [
    ("EQ-001", "Excavator", 2020, 2500.00),
    ("EQ-002", "Motor Grader", 2021, 4800.00),
    ("EQ-003", "Pickup", 2023, 1800.00)
]

columns = [
    "inventory_number",
    "equipment_type",
    "manufacturing_year",
    "maintenance_cost"
]

df = spark.createDataFrame(data, columns)

df.show()
```

Spark can infer the schema from the supplied data.

However, inferred schemas are not always appropriate for production workloads.

For example, an identifier might be interpreted differently depending on the values available during inference.

### 3.2 Creating a DataFrame with an explicit schema

Explicit schemas provide better control over data types and nullability.

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    StringType,
    IntegerType,
    DecimalType
)

schema = StructType([
    StructField("inventory_number", StringType(), False),
    StructField("equipment_type", StringType(), False),
    StructField("manufacturing_year", IntegerType(), True),
    StructField("maintenance_cost", DecimalType(12, 2), True)
])

df = spark.createDataFrame(data, schema=schema)

df.printSchema()
```

An explicit schema is particularly useful when:

- Building repeatable data pipelines.
- Validating incoming data.
- Processing financial information.
- Maintaining consistent data contracts.
- Avoiding unexpected type inference.

**Production principle:** Define schemas deliberately whenever the expected structure is known.

### 3.3 Creating DataFrames with Row

The `Row` class allows records to be represented as named objects.

```python
from pyspark.sql import Row

records = [
    Row(
        inventory_number="EQ-001",
        equipment_type="Excavator",
        maintenance_cost=2500.00
    ),
    Row(
        inventory_number="EQ-002",
        equipment_type="Motor Grader",
        maintenance_cost=4800.00
    )
]

df = spark.createDataFrame(records)

df.show()
```

This approach can make small examples and structured record creation easier to read.

### 3.4 Creating DataFrames from RDDs

An existing RDD can be converted into a DataFrame.

```python
records_rdd = spark.sparkContext.parallelize([
    ("EQ-001", "Excavator"),
    ("EQ-002", "Motor Grader")
])

df = spark.createDataFrame(
    records_rdd,
    ["inventory_number", "equipment_type"]
)
```

Although this is supported, DataFrames and higher-level APIs are generally preferred for structured workloads because Spark can optimize them more effectively.

---

## 4. Understanding DataFrame Schemas

A **schema** describes the structure of a DataFrame.

It defines:

- Column names.
- Column data types.
- Nullability.
- Nested structures.

Schemas are fundamental to reliable data processing.

### 4.1 StructType

`StructType` represents the complete schema of a DataFrame or a nested record.

```python
from pyspark.sql.types import StructType

schema = StructType([
    StructField("inventory_number", StringType(), False),
    StructField("equipment_type", StringType(), False),
    StructField("manufacturing_year", IntegerType(), True)
])
```

### 4.2 StructField

`StructField` defines an individual column.

Its main properties are:

| Property | Meaning |
|---|---|
| `name` | Column name |
| `dataType` | Expected data type |
| `nullable` | Whether null values are permitted |
| `metadata` | Optional additional metadata |

Example:

```python
StructField(
    "maintenance_cost",
    DecimalType(12, 2),
    True
)
```

This describes a column named `maintenance_cost` that accepts decimal values and permits nulls.

### 4.3 Inspecting a schema

```python
df.printSchema()
```

Example output:

```text
root
 |-- inventory_number: string (nullable = false)
 |-- equipment_type: string (nullable = false)
 |-- manufacturing_year: integer (nullable = true)
 |-- maintenance_cost: decimal(12,2) (nullable = true)
```

Other inspection methods:

```python
df.schema
```

```python
df.dtypes
```

```python
df.columns
```

```python
len(df.columns)
```

These methods help verify the structure before applying transformations.

---

## 5. Spark SQL Data Types

Spark uses its own SQL data type system.

The selected type affects:

- Storage representation.
- Comparisons.
- Arithmetic operations.
- Aggregations.
- Data validation.
- Compatibility with external systems.

### 5.1 Common primitive data types

| Spark data type | Description | Example |
|---|---|---|
| `StringType` | Text | `"Excavator"` |
| `IntegerType` | 32-bit integer | `2026` |
| `LongType` | 64-bit integer | `9876543210` |
| `ShortType` | 16-bit integer | `120` |
| `ByteType` | 8-bit integer | `12` |
| `FloatType` | 32-bit floating point | `12.5` |
| `DoubleType` | 64-bit floating point | `1250.75` |
| `DecimalType` | Fixed-precision decimal | `1250.75` |
| `BooleanType` | True or false | `True` |
| `DateType` | Calendar date | `2026-11-12` |
| `TimestampType` | Date and time | `2026-11-12 15:30:00` |

### 5.2 DecimalType and financial information

Financial values should generally use fixed-precision decimal types rather than binary floating-point types.

```python
DecimalType(12, 2)
```

This means:

- Precision: 12 total digits.
- Scale: 2 digits after the decimal point.

For example:

```text
1250000.75
```

A decimal type is generally more appropriate for monetary values because it provides controlled decimal precision.

### 5.3 Complex data types

Spark also supports nested and collection-based types.

| Data type | Description |
|---|---|
| `ArrayType` | Collection of values |
| `MapType` | Key-value collection |
| `StructType` | Nested structured record |

Example:

```python
from pyspark.sql.types import (
    ArrayType,
    MapType,
    StructType,
    StructField,
    StringType,
    IntegerType
)

schema = StructType([
    StructField("inventory_number", StringType(), False),
    StructField("maintenance_codes", ArrayType(StringType()), True),
    StructField(
        "equipment_details",
        StructType([
            StructField("brand", StringType(), True),
            StructField("year", IntegerType(), True)
        ]),
        True
    )
])
```

Nested structures are especially common in JSON data.

---

## 6. Reading Data from External Sources

Spark DataFrames can be created from multiple storage formats and systems.

Common sources include:

- CSV files.
- JSON files.
- Parquet files.
- Relational databases.
- Cloud object storage.
- Data lake platforms.

### 6.1 Reading CSV files

```python
df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("data/maintenance.csv")
)
```

Important CSV options:

| Option | Purpose |
|---|---|
| `header` | Indicates whether the first row contains column names |
| `inferSchema` | Attempts to infer column types |
| `delimiter` | Specifies the field separator |
| `encoding` | Defines the character encoding |
| `quote` | Defines the quotation character |
| `escape` | Defines the escape character |
| `mode` | Controls malformed-record handling |

For a production pipeline, an explicit schema is preferable.

```python
df = (
    spark.read
    .schema(schema)
    .option("header", True)
    .option("delimiter", ",")
    .csv("data/maintenance.csv")
)
```

### 6.2 Reading JSON files

```python
df = spark.read.json("data/maintenance.json")
```

JSON can contain nested structures, arrays, and objects.

Inspect the resulting schema:

```python
df.printSchema()
```

For known structures, provide an explicit schema:

```python
df = (
    spark.read
    .schema(schema)
    .json("data/maintenance.json")
)
```

### 6.3 Reading Parquet files

```python
df = spark.read.parquet("data/maintenance.parquet")
```

Parquet is a columnar storage format commonly used in analytical data systems.

Its characteristics include:

- Column-oriented storage.
- Efficient compression.
- Schema information.
- Column pruning.
- Predicate pushdown support.

Spark can often read only the columns required by a query instead of loading every column from a file.

### 6.4 Reading from a relational database using JDBC

Spark can connect to relational databases through JDBC.

Example:

```python
jdbc_url = "jdbc:postgresql://localhost:5432/fleet_database"

connection_properties = {
    "user": "database_user",
    "password": "database_password",
    "driver": "org.postgresql.Driver"
}

df = (
    spark.read
    .jdbc(
        url=jdbc_url,
        table="maintenance_records",
        properties=connection_properties
    )
)
```

The PostgreSQL JDBC driver must be available to the Spark application.

Credentials should not be hardcoded in production code. Use an appropriate secrets-management mechanism.

For large tables, JDBC reads can be parallelized using partitioning options, but the partition column and database workload must be considered carefully.

### 6.5 Handling malformed records

Spark supports different parsing modes for supported file readers.

| Mode | Behavior |
|---|---|
| `PERMISSIVE` | Attempts to process records and handles malformed data according to the reader's rules |
| `FAILFAST` | Stops execution when malformed records are encountered |
| `DROPMALFORMED` | Drops malformed records where supported |

Example:

```python
df = (
    spark.read
    .option("mode", "FAILFAST")
    .option("header", True)
    .csv("data/maintenance.csv")
)
```

Do not automatically discard malformed records without understanding their cause.

A robust pipeline should make data-quality problems visible and preserve information needed for investigation.

---

## 7. Selecting and Renaming Columns

Column selection is one of the most frequent DataFrame operations.

### 7.1 select()

```python
df.select(
    "inventory_number",
    "equipment_type",
    "maintenance_cost"
)
```

The result is another DataFrame containing only the selected columns.

### 7.2 Using col()

```python
from pyspark.sql.functions import col

df.select(
    col("inventory_number"),
    col("maintenance_cost")
)
```

Using `col()` makes it easier to construct expressions.

### 7.3 Using alias()

```python
df.select(
    col("maintenance_cost").alias("total_cost")
)
```

`alias()` assigns a different name to the resulting expression.

### 7.4 Using selectExpr()

```python
df.selectExpr(
    "inventory_number",
    "maintenance_cost * 1.16 AS cost_with_tax"
)
```

`selectExpr()` accepts SQL expressions as strings.

### 7.5 Removing columns

```python
df_without_notes = df.drop("internal_notes")
```

### 7.6 Adding or replacing columns

```python
df = df.withColumn(
    "maintenance_cost",
    col("maintenance_cost").cast("double")
)
```

`withColumn()` creates a new DataFrame with the specified column added or replaced.

It does not modify the original DataFrame in place.

### 7.7 Renaming columns

```python
df = df.withColumnRenamed(
    "maintenance_cost",
    "service_cost"
)
```

---

## 8. Filtering Rows

Filtering selects records that satisfy specified conditions.

### 8.1 filter() and where()

Both methods can be used to filter rows.

```python
df.filter(col("maintenance_cost") > 3000)
```

```python
df.where(col("maintenance_cost") > 3000)
```

They are equivalent for ordinary DataFrame filtering.

### 8.2 Multiple conditions

Use:

- `&` for AND.
- `|` for OR.
- `~` for NOT.

Example:

```python
df.filter(
    (col("maintenance_cost") > 2000) &
    (col("equipment_type") == "Excavator")
)
```

Another example:

```python
df.filter(
    (col("equipment_type") == "Excavator") |
    (col("equipment_type") == "Motor Grader")
)
```

**Important:** Parentheses are required around individual conditions when combining them with `&` or `|`.

Do not use Python's `and` or `or` operators to combine Spark column expressions.

### 8.3 Filtering by a range

```python
df.filter(
    col("manufacturing_year").between(2018, 2024)
)
```

### 8.4 Filtering null values

```python
df.filter(col("maintenance_cost").isNull())
```

```python
df.filter(col("maintenance_cost").isNotNull())
```

### 8.5 Filtering with isin()

```python
df.filter(
    col("equipment_type").isin(
        "Excavator",
        "Motor Grader",
        "Pickup"
    )
)
```

### 8.6 SQL null semantics

SQL uses three-valued logic:

- TRUE
- FALSE
- UNKNOWN

A comparison involving `NULL` generally evaluates to UNKNOWN rather than TRUE or FALSE.

For example:

```python
df.filter(col("maintenance_cost") > 3000)
```

Rows where `maintenance_cost` is null do not satisfy the filter.

Use `isNull()` and `isNotNull()` when testing for missing values.

---

## 9. Creating Derived Columns

Derived columns are calculated from existing information.

They are useful for:

- Calculating costs.
- Categorizing records.
- Standardizing text.
- Creating time-based attributes.
- Preparing analytical variables.

### 9.1 Arithmetic expressions

```python
df = df.withColumn(
    "cost_with_tax",
    col("maintenance_cost") * 1.16
)
```

### 9.2 Conditional expressions with when()

```python
from pyspark.sql.functions import when

df = df.withColumn(
    "cost_category",
    when(col("maintenance_cost") >= 5000, "High")
    .when(col("maintenance_cost") >= 2000, "Medium")
    .otherwise("Low")
)
```

This is similar to a SQL `CASE WHEN` expression.

### 9.3 Casting data types

```python
df = df.withColumn(
    "manufacturing_year",
    col("manufacturing_year").cast("integer")
)
```

Casting attempts to convert a value into another type.

Invalid conversions can result in null values or errors depending on the type, Spark version, and SQL configuration.

Validate conversions rather than assuming every value is valid.

### 9.4 Literal values with lit()

```python
from pyspark.sql.functions import lit

df = df.withColumn(
    "data_source",
    lit("maintenance_system")
)
```

`lit()` creates a literal column expression.

### 9.5 Handling multiple possible values with coalesce()

```python
from pyspark.sql.functions import coalesce

df = df.withColumn(
    "effective_cost",
    coalesce(
        col("approved_cost"),
        col("estimated_cost"),
        lit(0)
    )
)
```

`coalesce()` returns the first non-null expression.

Use it carefully: replacing missing financial values with zero is only appropriate when zero has a valid business meaning.

### 9.6 Text functions

```python
from pyspark.sql.functions import (
    lower,
    upper,
    trim,
    concat,
    lit
)

df = df.withColumn(
    "normalized_equipment_type",
    upper(trim(col("equipment_type")))
)
```

Concatenation:

```python
df = df.withColumn(
    "equipment_label",
    concat(
        col("inventory_number"),
        lit(" - "),
        col("equipment_type")
    )
)
```

### 9.7 Date functions

```python
from pyspark.sql.functions import (
    to_date,
    year,
    month,
    dayofmonth,
    datediff,
    current_date
)

df = df.withColumn(
    "maintenance_date",
    to_date(col("maintenance_date"), "yyyy-MM-dd")
)
```

Extract date components:

```python
df = df.withColumn(
    "maintenance_year",
    year(col("maintenance_date"))
)
```

```python
df = df.withColumn(
    "maintenance_month",
    month(col("maintenance_date"))
)
```

Calculate elapsed days:

```python
df = df.withColumn(
    "days_since_maintenance",
    datediff(
        current_date(),
        col("maintenance_date")
    )
)
```

Date calculations depend on correctly parsed date and timestamp values.

---

## 10. Handling Missing Values

Missing values are common in real-world datasets.

They may represent:

- Information that was never recorded.
- Information that is unavailable.
- An incomplete data-entry process.
- An inapplicable attribute.
- A failed extraction or transformation.

A missing value should not automatically be interpreted as zero.

### 10.1 dropna()

Remove rows containing null values.

```python
df_clean = df.dropna()
```

This can remove more information than intended.

A more specific example:

```python
df_clean = df.dropna(
    subset=[
        "inventory_number",
        "maintenance_date"
    ]
)
```

This removes records where either required field is null.

### 10.2 fillna()

Replace null values.

```python
df_clean = df.fillna({
    "equipment_type": "Unknown",
    "maintenance_status": "Not specified"
})
```

Numeric example:

```python
df_clean = df.fillna({
    "maintenance_count": 0
})
```

Only use zero when it accurately represents the business meaning of the missing value.

### 10.3 replace()

Replace specific values.

```python
df_clean = df.replace(
    "N/A",
    None,
    subset=["equipment_type"]
)
```

This can help standardize inconsistent representations of missing information.

### 10.4 Null versus NaN

`NULL` and `NaN` are not identical.

- `NULL` represents a missing or unknown value.
- `NaN` is a special floating-point value representing an undefined or unrepresentable numeric result.

For floating-point columns, use `isnan()` when checking for NaN.

```python
from pyspark.sql.functions import isnan

df.filter(
    isnan(col("measurement_value"))
)
```

A more comprehensive check may consider both:

```python
df.filter(
    col("measurement_value").isNull() |
    isnan(col("measurement_value"))
)
```

The `isnan()` function is intended for floating-point expressions.

---

## 11. Removing Duplicate Records

Duplicate records can distort:

- Counts.
- Financial totals.
- Equipment inventories.
- Operational indicators.
- Analytical results.

### 11.1 dropDuplicates()

Remove completely duplicated rows:

```python
df_distinct = df.dropDuplicates()
```

### 11.2 Duplicates based on selected columns

```python
df_unique = df.dropDuplicates([
    "inventory_number",
    "maintenance_date",
    "maintenance_type"
])
```

This considers the selected columns when identifying duplicate records.

### 11.3 Important determinism consideration

`dropDuplicates()` does not provide a business rule for choosing which record should survive when non-key columns differ.

For example, suppose two records have the same inventory number and maintenance date but different costs.

Simply removing duplicates does not guarantee that the record with the most recent update or the highest-quality source will be retained.

If a specific record must be selected, define an ordering rule using a window function.

This distinction is important in production data pipelines.

---

## 12. Sorting and Limiting Results

### 12.1 orderBy()

Sort records by one or more columns.

```python
df.orderBy("maintenance_date")
```

Descending order:

```python
df.orderBy(
    col("maintenance_cost").desc()
)
```

Multiple columns:

```python
df.orderBy(
    col("equipment_type").asc(),
    col("maintenance_cost").desc()
)
```

### 12.2 sort()

`sort()` is an alias for `orderBy()` in the DataFrame API.

```python
df.sort(
    col("maintenance_date").desc()
)
```

### 12.3 limit()

Restrict the number of returned rows.

```python
df.limit(10)
```

A limited DataFrame is useful for inspection, testing, and sampling.

However, without a defined ordering, the selected rows are not guaranteed to represent a consistent business-defined top ten.

---

## 13. Aggregations and Grouping

Aggregations summarize multiple records into calculated values.

Common operations include:

- Count.
- Sum.
- Average.
- Minimum.
- Maximum.
- Distinct count.

### 13.1 Basic aggregation

```python
from pyspark.sql.functions import (
    count,
    sum,
    avg,
    min,
    max
)

df.agg(
    count("*").alias("total_records"),
    sum("maintenance_cost").alias("total_cost"),
    avg("maintenance_cost").alias("average_cost"),
    min("maintenance_cost").alias("minimum_cost"),
    max("maintenance_cost").alias("maximum_cost")
).show()
```

### 13.2 Grouping records

```python
df.groupBy("equipment_type").count()
```

This calculates the number of records for each equipment type.

### 13.3 Multiple aggregation functions

```python
df.groupBy("equipment_type").agg(
    count("*").alias("maintenance_events"),
    sum("maintenance_cost").alias("total_cost"),
    avg("maintenance_cost").alias("average_cost"),
    max("maintenance_cost").alias("highest_cost")
)
```

### 13.4 Counting distinct values

```python
from pyspark.sql.functions import countDistinct

df.agg(
    countDistinct("inventory_number").alias("unique_equipment")
)
```

### 13.5 Why aggregations can be expensive

A grouped aggregation may require Spark to redistribute records across partitions.

This redistribution is called a **shuffle**.

For example, if records are distributed across several executors, Spark may need to bring records with the same equipment type together before calculating the group-level results.

Shuffles can involve:

- Network communication.
- Disk operations.
- Serialization.
- Memory consumption.

Their cost depends on data volume, partitioning, execution plans, and available resources.

---

## 14. Joining DataFrames

A join combines information from two or more DataFrames using a relationship between their columns.

Consider two datasets.

**Equipment DataFrame**

| inventory_number | equipment_type |
|---|---|
| EQ-001 | Excavator |
| EQ-002 | Motor Grader |
| EQ-003 | Pickup |

**Maintenance DataFrame**

| inventory_number | maintenance_cost |
|---|---:|
| EQ-001 | 2500.00 |
| EQ-001 | 1800.00 |
| EQ-002 | 4800.00 |

The inventory number connects the two datasets.

### 14.1 Inner join

Returns records with matching keys in both DataFrames.

```python
result = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="inner"
)
```

### 14.2 Left join

Returns all records from the left DataFrame and matching records from the right DataFrame.

```python
result = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="left"
)
```

Equipment without maintenance records remains in the result with null values for maintenance columns.

### 14.3 Right join

Returns all records from the right DataFrame and matching records from the left DataFrame.

```python
result = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="right"
)
```

### 14.4 Full outer join

Returns matching and nonmatching records from both DataFrames.

```python
result = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="full"
)
```

### 14.5 Left semi join

Returns rows from the left DataFrame that have a match in the right DataFrame.

```python
result = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="left_semi"
)
```

Only columns from the left DataFrame are returned.

### 14.6 Left anti join

Returns rows from the left DataFrame that do not have a match in the right DataFrame.

```python
result = equipment_df.join(
    maintenance_df,
    on="inventory_number",
    how="left_anti"
)
```

This is useful for identifying equipment without maintenance records.

### 14.7 Cross join

Returns the Cartesian product of both DataFrames.

```python
result = equipment_df.crossJoin(
    maintenance_type_df
)
```

If the first DataFrame contains 100 rows and the second contains 5 rows, the result contains 500 combinations.

Cross joins should be used intentionally because their output can grow rapidly.

### 14.8 Join types summary

| Join type | Result |
|---|---|
| Inner | Matching rows from both sides |
| Left | All left rows and matching right rows |
| Right | All right rows and matching left rows |
| Full outer | All rows from both sides |
| Left semi | Left rows with a match |
| Left anti | Left rows without a match |
| Cross | Every possible combination |

### 14.9 Handling ambiguous columns

If both DataFrames contain columns with the same name, Spark may report ambiguous references when those columns are used later.

Use aliases and explicit column references.

```python
equipment = equipment_df.alias("equipment")
maintenance = maintenance_df.alias("maintenance")

result = (
    equipment.join(
        maintenance,
        col("equipment.inventory_number") ==
        col("maintenance.inventory_number"),
        "inner"
    )
    .select(
        col("equipment.inventory_number"),
        col("equipment.equipment_type"),
        col("maintenance.maintenance_cost")
    )
)
```

### 14.10 Null values in join keys

Ordinary equality joins do not treat two null keys as equal.

If null-safe equality is required, Spark provides the null-safe equality operator through `eqNullSafe()`.

```python
result = left_df.join(
    right_df,
    col("left_df_key").eqNullSafe(
        col("right_df_key")
    ),
    "inner"
)
```

Use null-safe matching only when it reflects the intended business relationship. Matching two unknown identifiers can produce misleading results.

### 14.11 Broadcast joins

When one DataFrame is sufficiently small, Spark may distribute it to executors to reduce the cost of a large shuffle.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="inventory_number",
    how="inner"
)
```

Broadcast joins can improve performance, but they are not automatically appropriate for every small-looking dataset.

Consider:

- The actual size after filtering.
- Available executor memory.
- Broadcast thresholds.
- Network transfer cost.
- The physical execution plan.

---

## 15. Union and UnionByName

A union combines rows from multiple DataFrames.

It is different from a join.

- A join combines columns based on a relationship.
- A union appends rows from compatible datasets.

### 15.1 union()

```python
combined_df = january_df.union(february_df)
```

The DataFrames must have compatible schemas and corresponding columns in the correct order.

### 15.2 unionByName()

```python
combined_df = january_df.unionByName(
    february_df
)
```

This matches columns by name rather than position.

It is generally safer when datasets contain the same fields in different column orders.

### 15.3 Allowing missing columns

```python
combined_df = january_df.unionByName(
    february_df,
    allowMissingColumns=True
)
```

Missing columns are filled with null values.

### 15.4 Union does not automatically remove duplicates

Spark's `union()` and `unionByName()` preserve duplicate rows.

If duplicate elimination is required:

```python
combined_df = combined_df.distinct()
```

Duplicate removal can be expensive because it may require a shuffle.

---

## 16. Window Functions

Window functions calculate values across related rows without collapsing the dataset into one row per group.

They are especially useful for:

- Ranking records.
- Finding the latest event.
- Comparing consecutive measurements.
- Calculating running totals.
- Identifying duplicate records according to business rules.

### 16.1 Defining a window

```python
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("inventory_number")
    .orderBy(col("maintenance_date").desc())
)
```

This defines a separate ordered window for each inventory number.

### 16.2 row_number()

Assigns a sequential number to rows within each window.

```python
from pyspark.sql.functions import row_number

ranked_df = df.withColumn(
    "row_number",
    row_number().over(window_spec)
)
```

### 16.3 Finding the latest maintenance event

```python
latest_maintenance = (
    ranked_df
    .filter(col("row_number") == 1)
    .drop("row_number")
)
```

This selects one latest maintenance record per equipment unit.

For reliable results, the ordering should include a unique tie-breaker if multiple records can have the same maintenance date.

For example:

```python
window_spec = (
    Window
    .partitionBy("inventory_number")
    .orderBy(
        col("maintenance_date").desc(),
        col("record_updated_at").desc(),
        col("maintenance_id").desc()
    )
)
```

### 16.4 rank() and dense_rank()

These functions assign rankings based on ordering.

- `rank()` leaves gaps after tied rankings.
- `dense_rank()` does not leave gaps.

Example ranking:

| Value | rank() | dense_rank() |
|---:|---:|---:|
| 100 | 1 | 1 |
| 100 | 1 | 1 |
| 90 | 3 | 2 |
| 80 | 4 | 3 |

### 16.5 lag() and lead()

`lag()` retrieves a value from a previous row in the window.

`lead()` retrieves a value from a subsequent row.

```python
from pyspark.sql.functions import lag, lead

comparison_window = (
    Window
    .partitionBy("inventory_number")
    .orderBy("maintenance_date")
)

df = df.withColumn(
    "previous_cost",
    lag("maintenance_cost").over(comparison_window)
)

df = df.withColumn(
    "next_cost",
    lead("maintenance_cost").over(comparison_window)
)
```

These functions are useful for analyzing changes over time.

---

## 17. User-Defined Functions

A **User-Defined Function (UDF)** allows custom Python logic to be applied to DataFrame values.

Example:

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

def classify_cost(cost):
    if cost is None:
        return "Unknown"

    if cost >= 5000:
        return "High"

    if cost >= 2000:
        return "Medium"

    return "Low"

classify_cost_udf = udf(
    classify_cost,
    StringType()
)

df = df.withColumn(
    "cost_category",
    classify_cost_udf(col("maintenance_cost"))
)
```

UDFs can be useful when built-in Spark functions cannot express the required logic.

However, Python UDFs may introduce serialization and execution overhead and can limit some query optimizations.

**Preferred approach:** Use Spark's built-in functions whenever possible.

For example, the previous classification can be implemented without a Python UDF:

```python
df = df.withColumn(
    "cost_category",
    when(col("maintenance_cost").isNull(), "Unknown")
    .when(col("maintenance_cost") >= 5000, "High")
    .when(col("maintenance_cost") >= 2000, "Medium")
    .otherwise("Low")
)
```

Built-in expressions are generally easier for Spark to optimize.

---

## 18. Transformations and Actions

Spark DataFrame operations can be divided into two broad categories.

### 18.1 Transformations

Transformations define a new DataFrame from an existing one.

Examples:

```python
df.select("inventory_number")
```

```python
df.filter(col("maintenance_cost") > 2000)
```

```python
df.groupBy("equipment_type").count()
```

```python
df.withColumn(
    "cost_with_tax",
    col("maintenance_cost") * 1.16
)
```

Transformations are generally lazy.

Spark records the operations rather than immediately executing the complete computation.

### 18.2 Actions

Actions trigger execution and return results or produce output.

Examples:

```python
df.show()
```

```python
df.count()
```

```python
df.collect()
```

```python
df.first()
```

```python
df.take(5)
```

Writing data is also an action.

### 18.3 Understanding lazy evaluation

Consider:

```python
filtered_df = df.filter(
    col("maintenance_cost") > 3000
)

selected_df = filtered_df.select(
    "inventory_number",
    "maintenance_cost"
)
```

At this point, Spark has constructed a logical plan.

The actual computation is triggered when an action is executed:

```python
selected_df.show()
```

This allows Spark to optimize the overall operation before performing the work.

### 18.4 Avoiding collect()

The following operation retrieves all rows into the driver's memory:

```python
records = df.collect()
```

For a large dataset, this can exhaust driver memory.

Prefer distributed operations and retrieve only a small amount of data when necessary.

```python
df.show(10)
```

```python
df.take(10)
```

`show()` is intended for inspection, while `take()` returns a limited number of rows to the driver.

Neither should be used as a substitute for distributed processing of large datasets.

---

## 19. Understanding Execution Plans

Spark can display the plan associated with a DataFrame.

```python
df.explain()
```

For more detail:

```python
df.explain("extended")
```

An execution plan helps identify:

- Data sources.
- Filters.
- Projections.
- Joins.
- Aggregations.
- Shuffles.
- Broadcast operations.
- Physical execution strategies.

### 19.1 Logical and physical plans

A simplified distinction:

| Plan | Purpose |
|---|---|
| Parsed logical plan | Represents the interpreted operation structure |
| Analyzed logical plan | Resolves references and data types |
| Optimized logical plan | Applies logical optimizations |
| Physical plan | Describes how Spark intends to execute the operations |

Spark's Catalyst optimizer is responsible for important SQL and DataFrame optimizations.

The physical execution engine then executes the selected plan.

### 19.2 Example

```python
result = (
    df.filter(col("maintenance_cost") > 2000)
    .select("inventory_number", "maintenance_cost")
)

result.explain("formatted")
```

Inspecting the plan can help determine whether filters are applied early and whether an expensive operation such as a shuffle is required.

---

## 20. Writing DataFrames

Spark can write DataFrames to several formats.

### 20.1 Writing Parquet

```python
df.write \
    .mode("overwrite") \
    .parquet("output/maintenance_parquet")
```

Parquet is commonly used for analytical datasets and data lake storage.

### 20.2 Writing CSV

```python
df.write \
    .mode("overwrite") \
    .option("header", True) \
    .csv("output/maintenance_csv")
```

### 20.3 Writing JSON

```python
df.write \
    .mode("overwrite") \
    .json("output/maintenance_json")
```

### 20.4 Save modes

| Mode | Behavior |
|---|---|
| `append` | Adds output to existing data |
| `overwrite` | Replaces existing output |
| `error` | Fails if output already exists |
| `ignore` | Does nothing if output already exists |

Example:

```python
df.write.mode("append").parquet(
    "output/maintenance_history"
)
```

Choose the mode according to the pipeline's intended behavior.

An accidental overwrite can destroy previously stored data.

### 20.5 Partitioning output data

Spark can organize output files into directory partitions.

```python
df.write \
    .mode("overwrite") \
    .partitionBy("maintenance_year") \
    .parquet("output/maintenance_by_year")
```

The resulting directory structure may look like:

```text
maintenance_by_year/
    maintenance_year=2024/
    maintenance_year=2025/
    maintenance_year=2026/
```

Partitioning can improve queries that filter on the partition column.

However, creating too many small partitions can produce excessive files and metadata overhead.

Partition columns should be selected according to expected query patterns and data volume.

### 20.6 Writing to PostgreSQL using JDBC

```python
df.write \
    .format("jdbc") \
    .option("url", jdbc_url) \
    .option("dbtable", "curated_maintenance") \
    .option("user", "database_user") \
    .option("password", "database_password") \
    .option("driver", "org.postgresql.Driver") \
    .mode("append") \
    .save()
```

Production considerations include:

- Credential security.
- Target table schema.
- Transaction behavior.
- Batch size.
- Number of concurrent connections.
- Database write capacity.
- Retry and failure handling.

Increasing Spark write parallelism can overload the relational database.

A distributed processing engine does not remove the resource limits of its destination system.

---

## 21. DataFrame Performance Best Practices

A DataFrame application can be correct and still perform poorly.

Performance depends on the execution plan, data distribution, storage format, cluster resources, and operation design.

### 21.1 Select only required columns

Avoid reading or carrying unnecessary columns through the entire pipeline.

```python
df = df.select(
    "inventory_number",
    "maintenance_date",
    "maintenance_cost"
)
```

This can reduce data movement and enable column pruning.

### 21.2 Filter early

```python
df = df.filter(
    col("maintenance_year") == 2026
)
```

Filtering early can reduce the volume of data processed by later operations.

Spark's optimizer may also push filters toward the data source when supported.

### 21.3 Prefer Parquet for analytical workloads

Parquet supports columnar reads and compression.

It is often a better choice than CSV for repeatedly queried analytical datasets.

### 21.4 Avoid unnecessary collect()

Keep large datasets distributed.

Use `collect()` only when the result is known to be small enough for driver memory.

### 21.5 Avoid row-by-row Python processing

Instead of collecting records and iterating over them in Python, use DataFrame expressions.

Less suitable for large datasets:

```python
records = df.collect()

for record in records:
    print(record)
```

Preferred:

```python
df.filter(
    col("maintenance_cost") > 3000
).show()
```

### 21.6 Understand repartition() and coalesce()

`repartition()` changes the number of partitions and redistributes data.

```python
df_repartitioned = df.repartition(8)
```

It commonly involves a shuffle.

`coalesce()` reduces the number of partitions, generally avoiding a full shuffle.

```python
df_reduced = df.coalesce(2)
```

Use `coalesce()` when reducing partitions is appropriate and a full redistribution is unnecessary.

Neither method should be applied automatically.

### 21.7 Cache and persist

Caching can be useful when a DataFrame is reused multiple times.

```python
df_cached = df.cache()
```

Or:

```python
df_persisted = df.persist()
```

Caching is lazy: the data is generally materialized when an action executes.

Release cached data when it is no longer needed:

```python
df_cached.unpersist()
```

Caching every intermediate DataFrame can waste memory and reduce overall performance.

### 21.8 Understand data skew

Data skew occurs when some partitions contain significantly more data than others.

For example, if one equipment type accounts for most maintenance records, a grouping or join involving that key may produce uneven workloads.

Possible techniques include:

- Reviewing key distributions.
- Filtering unnecessary records.
- Selecting appropriate join strategies.
- Repartitioning by suitable keys.
- Using skew-aware techniques when justified.

### 21.9 Broadcast joins

Broadcasting a genuinely small DataFrame can avoid shuffling a large DataFrame during certain joins.

However, broadcast size and executor memory must be considered.

### 21.10 Avoid excessive small files

Writing many tiny files can increase:

- File-system metadata operations.
- Query planning overhead.
- Listing time.
- Read initialization cost.

Control output partitioning and file sizes according to the storage system and workload.

### 21.11 Performance checklist

Before optimizing, ask:

- Are unnecessary columns being processed?
- Can filters be pushed closer to the data source?
- Are there expensive shuffles?
- Are join keys appropriately distributed?
- Is data skew present?
- Are there too many small output files?
- Is caching actually beneficial?
- Is the driver collecting excessive data?
- Does the physical execution plan match expectations?

Optimization should be evidence-driven rather than based on arbitrary configuration changes.

---

# 22. Complete Practical Example: Fleet Maintenance Data Pipeline

This example demonstrates a small structured-data workflow using PySpark.

The scenario involves equipment records and maintenance events.

The dataset is illustrative and does not represent an actual production database schema.

The objective is to:

1. Create structured DataFrames.
2. Define explicit schemas.
3. Inspect incoming records.
4. Clean and standardize values.
5. Calculate derived costs.
6. Aggregate maintenance expenses.
7. Join maintenance information with equipment information.
8. Identify the latest maintenance event.
9. Write curated analytical datasets.

## 22.1 Initialize Spark

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("FleetMaintenancePipeline")
    .master("local[*]")
    .getOrCreate()
)
```

## 22.2 Import required functions and types

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

from pyspark.sql.types import (
    StructType,
    StructField,
    StringType,
    IntegerType,
    DecimalType,
    DateType
)
```

## 22.3 Define the equipment schema

```python
equipment_schema = StructType([
    StructField("inventory_number", StringType(), False),
    StructField("equipment_type", StringType(), False),
    StructField("manufacturing_year", IntegerType(), True),
    StructField("operational_status", StringType(), True)
])
```

## 22.4 Create equipment records

```python
equipment_data = [
    ("EQ-001", "Excavator", 2020, "Active"),
    ("EQ-002", "Motor Grader", 2021, "Active"),
    ("EQ-003", "Pickup", 2023, "Active"),
    ("EQ-004", "Excavator", 2019, "Inactive")
]

equipment_df = spark.createDataFrame(
    equipment_data,
    schema=equipment_schema
)
```

## 22.5 Define the maintenance schema

```python
maintenance_schema = StructType([
    StructField("maintenance_id", StringType(), False),
    StructField("inventory_number", StringType(), False),
    StructField("maintenance_date", DateType(), False),
    StructField("maintenance_type", StringType(), True),
    StructField("maintenance_cost", DecimalType(12, 2), True),
    StructField("record_updated_at", StringType(), True)
])
```

## 22.6 Create maintenance records

```python
from datetime import date

maintenance_data = [
    (
        "M-001",
        "EQ-001",
        date(2026, 1, 10),
        "Preventive",
        2500.00,
        "2026-01-11 10:00:00"
    ),
    (
        "M-002",
        "EQ-001",
        date(2026, 3, 15),
        "Corrective",
        4800.00,
        "2026-03-16 09:00:00"
    ),
    (
        "M-003",
        "EQ-002",
        date(2026, 2, 20),
        "Preventive",
        1800.00,
        "2026-02-21 12:00:00"
    ),
    (
        "M-004",
        "EQ-003",
        date(2026, 4, 5),
        "Corrective",
        3200.00,
        "2026-04-06 08:00:00"
    ),
    (
        "M-005",
        "EQ-001",
        date(2026, 5, 10),
        "Preventive",
        2100.00,
        "2026-05-11 11:00:00"
    )
]

maintenance_df = spark.createDataFrame(
    maintenance_data,
    schema=maintenance_schema
)
```

## 22.7 Inspect the incoming datasets

```python
equipment_df.printSchema()

maintenance_df.printSchema()
```

```python
equipment_df.show()

maintenance_df.show()
```

Inspect record counts:

```python
print("Equipment records:", equipment_df.count())

print("Maintenance records:", maintenance_df.count())
```

## 22.8 Standardize text values

```python
equipment_clean = equipment_df.withColumn(
    "equipment_type",
    F.upper(F.trim(F.col("equipment_type")))
)
```

```python
maintenance_clean = maintenance_df.withColumn(
    "maintenance_type",
    F.upper(F.trim(F.col("maintenance_type")))
)
```

Standardization reduces inconsistencies caused by capitalization and whitespace.

## 22.9 Validate required fields

```python
maintenance_valid = maintenance_clean.filter(
    F.col("maintenance_id").isNotNull() &
    F.col("inventory_number").isNotNull() &
    F.col("maintenance_date").isNotNull()
)
```

This removes records that do not satisfy the minimum requirements for this illustrative workflow.

A production pipeline should also record rejected rows and the reasons for rejection.

## 22.10 Calculate derived costs

```python
maintenance_enriched = maintenance_valid.withColumn(
    "cost_with_tax",
    F.col("maintenance_cost") * F.lit(1.16)
)
```

Add a cost category:

```python
maintenance_enriched = maintenance_enriched.withColumn(
    "cost_category",
    F.when(F.col("maintenance_cost").isNull(), "UNKNOWN")
    .when(F.col("maintenance_cost") >= 4000, "HIGH")
    .when(F.col("maintenance_cost") >= 2000, "MEDIUM")
    .otherwise("LOW")
)
```

## 22.11 Aggregate costs by equipment

```python
equipment_cost_summary = (
    maintenance_enriched
    .groupBy("inventory_number")
    .agg(
        F.count("*").alias("maintenance_events"),
        F.sum("maintenance_cost").alias("total_maintenance_cost"),
        F.avg("maintenance_cost").alias("average_maintenance_cost"),
        F.max("maintenance_cost").alias("highest_maintenance_cost")
    )
)
```

Inspect the result:

```python
equipment_cost_summary.show()
```

## 22.12 Aggregate costs by equipment type

Join the equipment dimension with the maintenance summary.

```python
equipment_type_summary = (
    equipment_df
    .join(
        maintenance_enriched,
        on="inventory_number",
        how="inner"
    )
    .groupBy("equipment_type")
    .agg(
        F.count("*").alias("maintenance_events"),
        F.sum("maintenance_cost").alias("total_cost"),
        F.avg("maintenance_cost").alias("average_cost")
    )
)
```

```python
equipment_type_summary.show()
```

## 22.13 Identify the latest maintenance event

```python
latest_window = (
    Window
    .partitionBy("inventory_number")
    .orderBy(
        F.col("maintenance_date").desc(),
        F.col("record_updated_at").desc(),
        F.col("maintenance_id").desc()
    )
)
```

```python
latest_maintenance = (
    maintenance_enriched
    .withColumn(
        "row_number",
        F.row_number().over(latest_window)
    )
    .filter(F.col("row_number") == 1)
    .drop("row_number")
)
```

This produces one latest maintenance record per equipment unit according to the defined ordering.

## 22.14 Combine equipment and latest maintenance information

```python
fleet_maintenance_status = (
    equipment_df.alias("equipment")
    .join(
        latest_maintenance.alias("maintenance"),
        on="inventory_number",
        how="left"
    )
    .select(
        F.col("inventory_number"),
        F.col("equipment_type"),
        F.col("manufacturing_year"),
        F.col("operational_status"),
        F.col("maintenance_date").alias("latest_maintenance_date"),
        F.col("maintenance_type").alias("latest_maintenance_type"),
        F.col("maintenance_cost").alias("latest_maintenance_cost")
    )
)
```

Inspect the final dataset:

```python
fleet_maintenance_status.show()
```

Because this is a left join, equipment without maintenance records remains in the output.

## 22.15 Write curated datasets

```python
equipment_cost_summary.write \
    .mode("overwrite") \
    .parquet("output/equipment_cost_summary")
```

```python
equipment_type_summary.write \
    .mode("overwrite") \
    .parquet("output/equipment_type_summary")
```

```python
fleet_maintenance_status.write \
    .mode("overwrite") \
    .parquet("output/fleet_maintenance_status")
```

The resulting datasets can be used by:

- Business intelligence tools.
- Analytical applications.
- Reporting pipelines.
- Data science workflows.
- Downstream data products.

## 22.16 Inspect the execution plan

```python
fleet_maintenance_status.explain("formatted")
```

Review the plan to understand how Spark executes the joins and window operations.

---

# 23. Practical Architecture Considerations

Knowing how to manipulate DataFrames is only one part of data engineering.

A production architecture must also address the context in which those DataFrames operate.

### 23.1 DataFrame does not mean database

A DataFrame is a processing abstraction.

It is not automatically:

- A persistent database table.
- A transactionally managed relational table.
- A complete data warehouse.
- A data catalog.
- A storage engine.

Persistence depends on the underlying storage system and write operations.

### 23.2 DataFrame does not guarantee data quality

A DataFrame can contain:

- Missing values.
- Duplicate records.
- Invalid identifiers.
- Inconsistent categories.
- Incorrect dates.
- Unexpected numeric values.
- Broken business relationships.

Data quality requires explicit validation rules.

### 23.3 DataFrame does not guarantee a unique record

Unless the pipeline explicitly enforces or validates uniqueness, multiple rows may contain the same identifier.

For example, an inventory number can legitimately appear in multiple maintenance events.

Uniqueness must be defined according to the business entity and the intended dataset.

### 23.4 DataFrame does not automatically guarantee transactional consistency

Spark DataFrame transformations do not, by themselves, provide all the transaction guarantees associated with a database system.

Transactional behavior depends on the storage layer and its supported capabilities.

### 23.5 Choosing the appropriate technology

Not every dataset needs Spark.

For a small operational fleet dataset, PostgreSQL may be sufficient for transactional operations, while Python and Pandas may be suitable for local analysis.

Spark becomes more attractive when data volume, distributed processing requirements, or integration complexity justify its operational cost.

Technology selection should follow workload requirements rather than tool popularity.

---

# 24. Common Mistakes

| Mistake | Why it is problematic | Better practice |
|---|---|---|
| Relying on inferred schemas everywhere | Data types may be inconsistent | Define explicit schemas |
| Using `collect()` on large datasets | Can exhaust driver memory | Keep processing distributed |
| Treating null as zero | Can distort business calculations | Define missing-value rules |
| Using Python `and` with Spark columns | Produces invalid expressions | Use `&` and parentheses |
| Using `dropDuplicates()` without a selection rule | Surviving records may not match business requirements | Use deterministic window logic |
| Assuming union removes duplicates | Duplicate rows remain | Use `distinct()` when appropriate |
| Using UDFs for simple operations | Can reduce optimization opportunities | Prefer built-in functions |
| Broadcasting a large dataset | Can exhaust executor memory | Verify actual dataset size |
| Repartitioning repeatedly | Can introduce expensive shuffles | Repartition only when justified |
| Caching everything | Wastes memory | Cache reused datasets selectively |
| Writing excessive small files | Creates metadata and read overhead | Control output partitioning |
| Ignoring execution plans | Makes performance problems harder to diagnose | Use `explain()` |

---

# 25. Exercises

## Exercise 1: DataFrame creation

Create a DataFrame containing:

- Inventory number.
- Equipment type.
- Manufacturing year.
- Operational status.

Requirements:

1. Create at least five records.
2. Define an explicit schema.
3. Inspect the schema.
4. Display the records.

## Exercise 2: Column operations

Using the previous DataFrame:

1. Select inventory number and equipment type.
2. Rename operational status.
3. Add a column named `equipment_age`.
4. Calculate the age using the current year.
5. Display the resulting DataFrame.

## Exercise 3: Filtering

Create the following filters:

1. Equipment manufactured after 2020.
2. Active equipment only.
3. Equipment manufactured between 2018 and 2024.
4. Equipment that is either an excavator or a pickup.
5. Equipment with a missing manufacturing year.

## Exercise 4: Data quality

Create a maintenance dataset containing:

- Null values.
- Duplicate records.
- Inconsistent capitalization.
- Invalid maintenance costs.

Develop a cleaning process that:

1. Standardizes text.
2. Identifies incomplete records.
3. Removes duplicates according to a defined rule.
4. Validates numeric values.
5. Produces a cleaned DataFrame.

Explain every decision.

## Exercise 5: Aggregations

Calculate:

1. Total maintenance events.
2. Total maintenance cost.
3. Average maintenance cost.
4. Highest maintenance cost.
5. Number of distinct equipment units.
6. Total cost by equipment type.

## Exercise 6: Joins

Create two DataFrames:

- Equipment.
- Maintenance records.

Perform:

1. Inner join.
2. Left join.
3. Left semi join.
4. Left anti join.

Explain which records appear in each result.

## Exercise 7: Window functions

For each equipment unit:

1. Order maintenance events by date.
2. Assign a row number.
3. Identify the latest event.
4. Compare the latest cost with the previous cost using `lag()`.

## Exercise 8: Data output

Write the final curated dataset in:

1. CSV.
2. JSON.
3. Parquet.

Compare the resulting directory structures and discuss which format is more appropriate for repeated analytical queries.

## Exercise 9: Execution plans

Create a DataFrame transformation involving:

- A filter.
- A column selection.
- A join.
- An aggregation.

Use `explain("formatted")`.

Identify the main operations and determine whether a shuffle is required.

---

# 26. Knowledge Check

**1. What is a Spark DataFrame?**

A. A local Python list.  
B. A distributed, structured data abstraction.  
C. A relational database server.  
D. A CSV file.

**2. What defines the columns and data types of a DataFrame?**

A. Executor.  
B. Schema.  
C. Partition manager.  
D. Cluster manager.

**3. Which method displays the DataFrame schema?**

A. `showSchema()`  
B. `describeSchema()`  
C. `printSchema()`  
D. `schemaShow()`

**4. Which operation filters records?**

A. `filter()`  
B. `collect()`  
C. `write()`  
D. `cache()`

**5. Which function creates a conditional expression?**

A. `when()`  
B. `groupBy()`  
C. `repartition()`  
D. `persist()`

**6. Which operation combines rows from compatible DataFrames?**

A. `join()`  
B. `union()`  
C. `filter()`  
D. `alias()`

**7. Which join identifies records without a match on the right side?**

A. Inner join.  
B. Left semi join.  
C. Left anti join.  
D. Cross join.

**8. Which function can identify the latest event per equipment unit when used with an appropriate window?**

A. `row_number()`  
B. `countDistinct()`  
C. `coalesce()`  
D. `concat()`

**9. What does `collect()` do?**

A. Writes data to Parquet.  
B. Retrieves rows to the driver.  
C. Removes duplicate rows.  
D. Creates a new schema.

**10. Why should built-in Spark functions generally be preferred over Python UDFs?**

A. They always use less storage.  
B. They generally offer better opportunities for Spark optimization.  
C. They eliminate null values.  
D. They automatically create indexes.

## Answer Key

| Question | Answer |
|---|---|
| 1 | B |
| 2 | B |
| 3 | C |
| 4 | A |
| 5 | A |
| 6 | B |
| 7 | C |
| 8 | A |
| 9 | B |
| 10 | B |

---

# 27. Technical Glossary

| Term | Definition |
|---|---|
| DataFrame | Distributed structured data abstraction in Spark |
| SparkSession | Main entry point for Spark DataFrame operations |
| Schema | Definition of column names, types, and nullability |
| StructType | Representation of a structured schema |
| StructField | Definition of an individual schema field |
| Data type | Specification of the kind of value a column contains |
| Transformation | Operation that produces a new DataFrame |
| Action | Operation that triggers execution or retrieves results |
| Lazy evaluation | Delayed execution of transformations |
| Column expression | Declarative expression representing a column operation |
| Filtering | Selecting records that satisfy conditions |
| Projection | Selecting or calculating columns |
| Aggregation | Summarizing multiple records |
| Grouping | Organizing records according to common values |
| Join | Combining datasets using a relationship |
| Union | Combining rows from compatible datasets |
| Window function | Calculation across related rows without collapsing them |
| UDF | User-Defined Function |
| Shuffle | Redistribution of data between partitions |
| Broadcast join | Join strategy that distributes a small dataset to executors |
| Data skew | Uneven distribution of records across partitions |
| Partition pruning | Avoiding unnecessary storage partitions during reads |
| Column pruning | Avoiding unnecessary column reads |
| Predicate pushdown | Applying supported filters closer to the data source |
| Parquet | Columnar file format for analytical data |
| Cache | Retaining computed data for potential reuse |
| Persist | Storing computed data using a selected storage level |
| Execution plan | Representation of how Spark will process a query |

---

# 28. Related Concepts and Learning Path

This note connects the fundamental concepts of Spark with practical data engineering operations.

### Previous concepts

- [[10 - Apache Spark Fundamentals]]
- [[09 - Distributed Data Processing]]
- [[04 - Data Pipelines]]
- [[03 - ETL and ELT]]
- [[Data Quality]]

### Next concepts

- [[12 - Spark SQL]]
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
          +----------------------+
          |          |           |
          v          v           v
        Schema   Transformations  Actions
          |          |           |
          v          v           v
      Data Types  Data Cleaning  Execution
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Filtering  Aggregation  Joins
                     |
                     v
               Window Functions
                     |
                     v
               Curated Datasets
                     |
                     v
                Data Storage
```

---

# 29. Key Takeaways

1. Spark DataFrames provide a structured abstraction for distributed data processing.
2. A DataFrame is not inherently a persistent database table or an in-memory dataset.
3. `SparkSession` is the main entry point for DataFrame operations.
4. Explicit schemas improve consistency and reliability.
5. Spark supports primitive, decimal, date, timestamp, and complex data types.
6. DataFrames can read and write CSV, JSON, Parquet, and relational database sources.
7. `select()`, `filter()`, and `withColumn()` are fundamental transformation methods.
8. Null handling and duplicate removal require explicit business rules.
9. Aggregations and joins may require expensive data redistribution.
10. Window functions support ranking and time-based record selection.
11. Transformations are generally lazy, while actions trigger execution.
12. `explain()` helps analyze logical and physical execution plans.
13. Parquet is commonly used for analytical storage.
14. Built-in Spark functions are generally preferable to Python UDFs.
15. Performance optimization should be guided by actual execution plans and workload characteristics.

**Final principle:**

A Spark DataFrame is more than a table-like object. It is a declarative interface for describing structured data transformations that Spark can optimize and execute across distributed computing resources.
