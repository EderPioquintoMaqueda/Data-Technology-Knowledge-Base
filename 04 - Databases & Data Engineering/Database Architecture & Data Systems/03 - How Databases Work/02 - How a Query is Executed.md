
---
title: "How a Query is Executed"
aliases:
  - "SQL Query Execution"
  - "Database Query Processing"
  - "Query Execution Lifecycle"
type: learning-note
domain: technology
category: database-systems
subject: query-processing
level: intermediate
difficulty: intermediate
status: in-progress
created: 2026-11-12
updated: 2026-11-12

tags:
  - database-architecture
  - query-processing
  - sql
  - query-execution
  - query-optimization
  - postgresql
  - database-performance
  - execution-plans

concepts:
  - sql-query
  - query-lifecycle
  - parsing
  - syntax-analysis
  - semantic-analysis
  - query-rewriting
  - query-optimization
  - execution-planning
  - query-executor
  - sequential-scan
  - index-scan
  - join-algorithms
  - execution-plan
  - explain
  - explain-analyze

prerequisites:
  - "[[00 - Database Architecture Roadmap]]"
  - "[[01 - What is a Database System]]"
  - "[[01 - Database Management Systems]]"
  - "[[06 - Schema Design Principles]]"

related:
  - "[[03 - Storage Pages and Tables]]"
  - "[[04 - Indexes]]"
  - "[[05 - Transactions and ACID]]"
  - "[[06 - Concurrency and Locks]]"
  - "[[04 - PostgreSQL Query Planning]]"
  - "[[05 - PostgreSQL Performance Fundamentals]]"

tools:
  - PostgreSQL
  - SQL
  - pgAdmin
  - psql
  - EXPLAIN
  - EXPLAIN ANALYZE

learning_objectives:
  - Explain the lifecycle of an SQL query.
  - Distinguish parsing, semantic analysis, rewriting, planning, and execution.
  - Understand the role of the query optimizer.
  - Interpret the basic structure of an execution plan.
  - Distinguish sequential scans, index scans, and bitmap scans.
  - Understand how PostgreSQL executes joins.
  - Use EXPLAIN and EXPLAIN ANALYZE.
  - Identify factors that influence query performance.

review:
  frequency: weekly
  last_reviewed: null
  next_review: null
---

# 02 — How a Query is Executed

## 1. Introduction

When a user executes an SQL statement, the database does not simply search for the requested information and return it immediately.

A DBMS performs several internal operations before producing a result.

These operations form the query processing lifecycle.

Consider this query:

```sql
SELECT
    inventory_number,
    brand,
    model
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

From the developer's perspective, this is a simple instruction:

"Find the vehicle whose inventory number is 13990."

However, PostgreSQL must perform several tasks:

1. Receive the SQL statement.
2. Parse its syntax.
3. Analyze its meaning.
4. Verify referenced objects and permissions.
5. Transform or rewrite the query when necessary.
6. Generate possible execution strategies.
7. Estimate the cost of those strategies.
8. Select an execution plan.
9. Execute the selected plan.
10. Return the result to the client.

Understanding this process helps explain why two SQL statements that return the same information can have different performance characteristics.

**SQL describes what information is required. The DBMS determines how to retrieve it.**

## 2. The Query Execution Lifecycle

The general query processing lifecycle can be represented as:

```text
              SQL STATEMENT
                    |
                    v
             +-------------+
             |   PARSING   |
             +-------------+
                    |
                    v
          +-------------------+
          | SEMANTIC ANALYSIS |
          +-------------------+
                    |
                    v
           +-----------------+
           | QUERY REWRITING |
           +-----------------+
                    |
                    v
            +---------------+
            | QUERY PLANNING|
            +---------------+
                    |
                    v
           +------------------+
           | QUERY OPTIMIZER  |
           +------------------+
                    |
                    v
           +------------------+
           | EXECUTION PLAN   |
           +------------------+
                    |
                    v
             +-------------+
             |  EXECUTION  |
             +-------------+
                    |
                    v
             +-------------+
             |   RESULTS   |
             +-------------+
```

This is a conceptual representation.

In PostgreSQL, planning includes optimization and plan generation. These are related stages rather than completely independent systems.

The exact internal implementation differs between DBMS platforms.

## 3. Stage 1 — Query Reception

The process begins when a client sends an SQL statement to the database server.

For example, a Python application may send:

```sql
SELECT *
FROM fleet.vehicles
WHERE vehicle_id = 1001;
```

The SQL statement travels through the database connection.

The DBMS receives the request and associates it with the relevant session.

Before execution, the system must operate within the context of that connection.

This includes information such as:

- Authenticated database user.
- Current database.
- Session configuration.
- Transaction state.
- Search path.
- Relevant permissions.

### 3.1 SQL is a declarative language

SQL is primarily declarative.

This means the developer specifies the desired result rather than explicitly describing every low-level operation needed to retrieve it.

For example:

```sql
SELECT *
FROM fleet.vehicles
WHERE department_id = 10;
```

The developer does not specify:

- Which disk blocks to read.
- Which index to use.
- How to access the table pages.
- Which physical join algorithm to use.
- How to manage memory buffers.

The DBMS determines these details.

This separation between logical intent and physical execution is one of the fundamental characteristics of relational databases.

## 4. Stage 2 — Parsing

Parsing is the process of analyzing the SQL statement's syntax.

The parser checks whether the statement follows the grammatical rules of SQL.

Consider:

```sql
SELECT *
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

This is syntactically valid.

Now consider:

```sql
SELEC *
FROM fleet.vehicles;
```

The keyword `SELEC` is invalid.

The parser identifies the syntax error.

### 4.1 What does the parser produce?

A parser typically converts the SQL statement into an internal representation, such as a parse tree.

A parse tree represents the structure of the statement.

For example:

```sql
SELECT brand
FROM fleet.vehicles
WHERE vehicle_id = 1001;
```

A simplified logical representation is:

```text
SELECT
  |
  +-- Column: brand
  |
  +-- FROM: fleet.vehicles
  |
  +-- WHERE
        |
        +-- vehicle_id = 1001
```

This is not the exact PostgreSQL internal parse tree. It is a conceptual illustration of the statement's structure.

### 4.2 Syntax errors

Examples of syntax errors include:

```sql
SELECT FROM fleet.vehicles;
```

```sql
SELECT *
fleet.vehicles;
```

```sql
SELECT *
FROM fleet.vehicles
WHERE = 1001;
```

The DBMS rejects statements that cannot be interpreted according to SQL grammar.

### 4.3 Important distinction

A statement can be syntactically correct but still be invalid.

For example:

```sql
SELECT nonexistent_column
FROM fleet.vehicles;
```

The SQL grammar is valid.

However, the referenced column may not exist.

This type of problem is identified during semantic analysis.

## 5. Stage 3 — Semantic Analysis

Semantic analysis determines whether the parsed statement makes sense within the database environment.

The DBMS verifies information such as:

- Whether tables exist.
- Whether columns exist.
- Whether referenced schemas are accessible.
- Whether data types are compatible.
- Whether functions and operators are valid.
- Whether the user has the required permissions.

### 5.1 Object validation

Consider:

```sql
SELECT inventory_number
FROM fleet.vehicles;
```

PostgreSQL must determine whether:

- The `fleet` schema exists.
- The `vehicles` table exists.
- The `inventory_number` column exists.

If one of these objects does not exist or cannot be resolved, the statement cannot proceed normally.

### 5.2 Data type validation

Consider:

```sql
SELECT *
FROM fleet.vehicles
WHERE inventory_number = 13990;
```

Suppose `inventory_number` is defined as `VARCHAR`.

The comparison involves a text column and an integer literal.

PostgreSQL may apply supported type conversions in some situations, but the expression must ultimately have valid operator and type semantics.

A clearer query is:

```sql
SELECT *
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

Using the correct type makes the intended comparison explicit.

### 5.3 Permission validation

A user may be able to connect to a database but lack permission to read a specific table.

For example:

```sql
SELECT *
FROM fleet.vehicles;
```

If the user does not have the required privileges, PostgreSQL rejects the operation.

Authentication and authorization are different concepts:

- Authentication determines who the user is.
- Authorization determines what the user is allowed to do.

## 6. Stage 4 — Query Rewriting

Query rewriting transforms a query into another logically equivalent representation when this is appropriate.

The purpose is to simplify or transform the query before physical execution planning.

In PostgreSQL, the rewrite system also handles rules and expands views.

### 6.1 View expansion

Suppose the database contains this view:

```sql
CREATE VIEW fleet.active_vehicles AS
SELECT
    vehicle_id,
    inventory_number,
    brand
FROM fleet.vehicles
WHERE is_active = TRUE;
```

The application executes:

```sql
SELECT *
FROM fleet.active_vehicles;
```

PostgreSQL expands the view definition into the query representation used for subsequent planning.

Conceptually, the request becomes similar to:

```sql
SELECT
    vehicle_id,
    inventory_number,
    brand
FROM fleet.vehicles
WHERE is_active = TRUE;
```

The database can then plan access to the underlying table.

### 6.2 Why rewriting matters

Rewriting allows the DBMS to work with a transformed representation of the original SQL statement.

Possible transformations include:

- View expansion.
- Rule application.
- Predicate transformations.
- Other logically equivalent query transformations.

Not every optimization occurs in the rewrite stage.

Many important decisions are performed later by the planner and optimizer.

## 7. Stage 5 — Query Planning and Optimization

This is one of the most important stages of query processing.

The query optimizer evaluates possible execution strategies and selects a plan.

Consider:

```sql
SELECT *
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

The database may have several ways to retrieve the requested row.

For example:

**Option A — Sequential scan**

Read the table's pages and examine rows until the matching record is found.

**Option B — Index scan**

Use an index to locate the relevant entry and retrieve the corresponding table row.

Both approaches can return the same result.

However, their costs may differ.

### 7.1 What is an execution plan?

An execution plan is the structured description of how the DBMS intends to execute a query.

It identifies operations such as:

- Sequential scans.
- Index scans.
- Joins.
- Sorting.
- Aggregation.
- Filtering.
- Data retrieval.

A plan is not the SQL statement itself.

It is the execution strategy selected by the database.

### 7.2 Cost-based optimization

PostgreSQL uses a cost-based optimizer.

It estimates the cost of alternative execution strategies using information such as:

- Table statistics.
- Estimated row counts.
- Data distribution.
- Available indexes.
- Selectivity.
- Expected disk access.
- CPU processing.
- Memory requirements.
- Configuration parameters.

The optimizer compares estimated costs and selects a plan.

### 7.3 What does cost mean?

Cost is an internal estimate used by the optimizer.

It is not directly equivalent to milliseconds or money.

PostgreSQL represents plan costs using estimated units based on configurable cost parameters.

A lower estimated cost generally indicates a more attractive plan under the optimizer's model.

However, the estimated cost is not a guarantee of actual execution time.

### 7.4 Statistics

The optimizer needs information about the data to estimate how expensive a query may be.

PostgreSQL maintains statistics about tables and columns.

These statistics help estimate:

- Number of rows.
- Number of distinct values.
- Data distribution.
- Frequency of certain values.
- Selectivity of conditions.

The `ANALYZE` command updates planner statistics.

Example:

```sql
ANALYZE fleet.vehicles;
```

PostgreSQL also runs automatic statistics collection through autovacuum mechanisms.

If statistics are outdated or insufficient, the optimizer may select a suboptimal plan.

### 7.5 Selectivity

Selectivity describes how much data a condition is expected to return.

Suppose a table contains 100,000 vehicles.

Query A:

```sql
SELECT *
FROM fleet.vehicles
WHERE vehicle_id = 1001;
```

This condition may return only one row.

Query B:

```sql
SELECT *
FROM fleet.vehicles
WHERE is_active = TRUE;
```

If 95% of the vehicles are active, the condition may return approximately 95,000 rows.

The first condition is highly selective.

The second is less selective.

Selectivity can influence whether an index is beneficial.

## 8. Stage 6 — Physical Execution Plan

Once the optimizer selects a plan, PostgreSQL executes the operations described by that plan.

Several access methods and algorithms may be used.

### 8.1 Sequential Scan

A sequential scan reads table pages and examines their rows.

Example:

```sql
SELECT *
FROM fleet.vehicles;
```

When the query needs most or all of the table, a sequential scan may be efficient.

A sequential scan is not automatically a bad plan.

It can be the best strategy for a particular workload.

### 8.2 Index Scan

An index scan uses an index to locate matching entries.

Example:

```sql
SELECT *
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

If a suitable index exists, PostgreSQL may use it to find the matching record.

The process is conceptually:

1. Search the index.
2. Identify the matching entry.
3. Locate the corresponding table row.
4. Retrieve the requested columns.

An index scan can be especially useful when a query returns a small portion of a large table.

### 8.3 Index Only Scan

An index-only scan retrieves the required information from the index without necessarily accessing the table for each matching row.

Example:

```sql
SELECT inventory_number
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

If the index contains all required columns and PostgreSQL can establish tuple visibility using its visibility map, an index-only scan may be possible.

It is not guaranteed merely because an index contains the selected column.

### 8.4 Bitmap Scan

A bitmap scan is another access strategy.

It can be useful when multiple matching index entries must be collected before accessing the corresponding table pages.

Conceptually:

1. Use one or more indexes to identify matching row locations.
2. Build a bitmap representing those locations.
3. Visit the relevant table pages.

This can reduce repeated page access when many rows match a condition.

### 8.5 Comparison

| Access method | General behavior |
|---|---|
| Sequential Scan | Reads table pages and examines rows. |
| Index Scan | Uses an index to locate matching rows. |
| Index Only Scan | Retrieves required information from an index when visibility conditions permit. |
| Bitmap Scan | Collects matching row locations before accessing table pages. |

The optimizer chooses an access strategy based on estimated cost.

## 9. Join Algorithms

When a query combines multiple tables, the DBMS must determine how to join their records.

Consider:

```sql
SELECT
    v.inventory_number,
    d.department_name
FROM fleet.vehicles AS v
INNER JOIN fleet.departments AS d
    ON v.department_id = d.department_id;
```

PostgreSQL may use different join algorithms.

### 9.1 Nested Loop Join

A nested loop join processes one input and searches for matching rows in the other input.

Conceptually:

```text
For each row in Table A:
    Find matching rows in Table B
```

It can be effective when:

- One input is small.
- An appropriate index exists on the other input.
- The join produces relatively few matches.

### 9.2 Hash Join

A hash join builds a hash table from one input and uses it to find matching rows from the other input.

Conceptually:

```text
Build hash table from Table A
             |
             v
Read Table B
             |
             v
Find matching hash entries
```

It can be efficient for equality joins involving larger inputs.

Memory availability influences its behavior.

### 9.3 Merge Join

A merge join processes two inputs ordered by their join keys.

Conceptually:

```text
Sorted Table A + Sorted Table B
                 |
                 v
          Merge matching keys
```

It can be efficient when inputs are already ordered or when sorting costs are justified.

### 9.4 Join algorithm comparison

| Algorithm | Main principle | Typical use |
|---|---|---|
| Nested Loop | Repeated matching | Small input or indexed lookup |
| Hash Join | Hash-based matching | Equality joins with larger inputs |
| Merge Join | Ordered matching | Inputs available in compatible order |

The optimizer decides which algorithm is appropriate.

Developers should understand these algorithms rather than attempting to force one without evidence.

## 10. Stage 7 — Returning Results

After execution, the DBMS returns the requested rows to the client.

For example:

```sql
SELECT
    inventory_number,
    brand,
    model
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

Possible result:

| inventory_number | brand | model |
|---|---|---|
| 13990 | Shantui | SG16 |

The application receives the result through the database connection.

It can then:

- Display it.
- Convert it into application objects.
- Use it in calculations.
- Generate a report.
- Send it to another service.

For queries that return large datasets, results may be transmitted in batches or through cursor-based mechanisms.

The application should avoid retrieving more information than it actually needs.

## 11. Understanding EXPLAIN

PostgreSQL provides `EXPLAIN` to display the execution plan selected for a query.

Example:

```sql
EXPLAIN
SELECT *
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

Possible output:

```text
Index Scan using vehicles_inventory_number_key
on vehicles
  (cost=0.15..8.17 rows=1 width=...)
  Index Cond: (inventory_number = '13990')
```

The exact output depends on the table, indexes, statistics, PostgreSQL version, and configuration.

### 11.1 Understanding plan fields

| Field | Meaning |
|---|---|
| `Index Scan` | Selected access method. |
| `using ...` | Index used by the operation. |
| `cost` | Estimated startup and total costs. |
| `rows` | Estimated number of output rows. |
| `width` | Estimated average row width in bytes. |
| `Index Cond` | Condition applied through the index. |

### 11.2 Startup cost

The first cost value represents the estimated cost before the node can return its first row.

This matters for operations that must perform initial work before producing output.

### 11.3 Total cost

The second value represents the estimated total cost of producing all rows from that plan node.

These values are estimates, not execution times.

### 11.4 Estimated rows

The `rows` value represents the optimizer's estimated number of rows produced by the node.

If the estimate is substantially different from the actual number of rows, the optimizer may have difficulty selecting an appropriate plan.

### 11.5 Plan hierarchy

Execution plans contain nodes arranged in a tree-like structure.

A parent node receives information from child nodes.

For example:

```text
Nested Loop
  -> Index Scan on vehicles
  -> Index Scan on departments
```

The two index scans provide input to the nested loop operation.

Understanding the plan hierarchy is essential for performance analysis.

## 12. EXPLAIN ANALYZE

`EXPLAIN ANALYZE` executes the query and reports actual execution information.

Example:

```sql
EXPLAIN ANALYZE
SELECT *
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

The output includes information such as:

- Actual execution time.
- Actual rows returned.
- Number of loops.
- Planning time.
- Execution time.

### 12.1 Estimated versus actual values

Consider:

```text
Index Scan
  (cost=0.15..8.17 rows=1 width=...)
  (actual time=0.020..0.025 rows=1 loops=1)
```

The first line contains estimated values.

The second line contains actual execution information.

A simplified comparison:

| Metric | Estimated | Actual |
|---|---|---|
| Rows | `rows=1` | `rows=1` |
| Cost | `0.15..8.17` | Not applicable |
| Time | Not applicable | `0.020..0.025 ms` |
| Loops | Not applicable | `1` |

Actual values vary with hardware, cache state, database size, and concurrent activity.

### 12.2 Important safety consideration

`EXPLAIN ANALYZE` actually executes the statement.

For a SELECT query, this generally means it reads data.

For statements such as UPDATE, DELETE, or INSERT, the statement performs the corresponding operation.

Therefore, do not casually execute:

```sql
EXPLAIN ANALYZE
DELETE FROM fleet.vehicles;
```

on a production database.

When evaluating data-modifying statements, use an appropriate test environment or a carefully controlled transaction that is rolled back when suitable.

### 12.3 Additional options

PostgreSQL supports additional EXPLAIN options.

Example:

```sql
EXPLAIN (
    ANALYZE,
    BUFFERS,
    VERBOSE,
    FORMAT TEXT
)
SELECT *
FROM fleet.vehicles
WHERE inventory_number = '13990';
```

Important options:

| Option | Purpose |
|---|---|
| `ANALYZE` | Executes the query and reports actual statistics. |
| `BUFFERS` | Reports buffer access information. |
| `VERBOSE` | Provides more detailed plan information. |
| `FORMAT` | Selects the output format. |
| `COSTS` | Controls whether estimated costs are displayed. |

The `BUFFERS` option is particularly useful when investigating whether a query accesses pages from memory or requires physical reads.

## 13. Factors That Influence Query Performance

Query performance depends on more than SQL syntax.

Several factors influence execution.

### 13.1 Table size

A query over 200 records behaves differently from one over 200 million records.

The amount of data influences:

- Number of pages.
- Memory consumption.
- Storage access.
- Index size.
- Execution time.

### 13.2 Index availability

An appropriate index can reduce the amount of data that must be examined.

However, indexes also introduce overhead.

### 13.3 Data distribution

The distribution of values affects selectivity estimates.

A column containing mostly unique values behaves differently from one containing only a few repeated categories.

### 13.4 Statistics

Outdated statistics can lead to inaccurate estimates.

PostgreSQL uses `ANALYZE` to collect planner statistics.

### 13.5 Query structure

The following can influence query execution:

- Joins.
- Subqueries.
- Aggregations.
- Sorting.
- Filtering.
- Window functions.
- Data type conversions.

### 13.6 Memory and hardware

Execution can be affected by:

- Available RAM.
- CPU capacity.
- Storage latency.
- Disk throughput.
- Network latency.
- Concurrent workloads.

### 13.7 Network and application overhead

The database may execute a query quickly while the overall application request remains slow.

For example:

- The application creates too many connections.
- A network request is slow.
- The application retrieves unnecessary columns.
- The application processes thousands of rows inefficiently.
- Multiple queries are executed unnecessarily.

Therefore, database query time and end-to-end application response time are different measurements.

## 14. Common Query Performance Mistakes

| Mistake | Possible consequence | Better approach |
|---|---|---|
| Using `SELECT *` unnecessarily | Retrieves unused columns | Select only required columns |
| Missing indexes on frequent access paths | Excessive row or page scanning | Evaluate query patterns |
| Creating too many indexes | Increased write and storage overhead | Index selectively |
| Joining tables without suitable conditions | Excessive intermediate results | Review join logic |
| Ignoring statistics | Poor cost estimates | Maintain statistics |
| Applying functions to indexed columns without considering index usage | May prevent ordinary index use | Evaluate query and index design |
| Retrieving too many rows | More memory and network traffic | Filter and paginate |
| Ignoring actual execution plans | Incorrect performance assumptions | Use EXPLAIN ANALYZE safely |
| Assuming an index is always faster | Suboptimal access method | Let the optimizer compare costs |

An important principle:

**Performance optimization should be based on evidence from actual workload behavior, not assumptions about SQL statements.**

## 15. Practical PostgreSQL Laboratory

The following exercises use a small example dataset.

### 15.1 Create test tables

```sql
CREATE SCHEMA IF NOT EXISTS query_lab;

CREATE TABLE query_lab.departments (
    department_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE query_lab.vehicles (
    vehicle_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    inventory_number VARCHAR(40) NOT NULL UNIQUE,
    brand VARCHAR(80) NOT NULL,
    department_id INTEGER REFERENCES query_lab.departments(department_id),
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);
```

### 15.2 Insert sample data

```sql
INSERT INTO query_lab.departments (department_name)
VALUES
    ('Administration'),
    ('Maintenance'),
    ('Operations');

INSERT INTO query_lab.vehicles (
    inventory_number,
    brand,
    department_id,
    is_active
)
VALUES
    ('13990', 'Shantui', 1, TRUE),
    ('13994', 'Caterpillar', 2, TRUE),
    ('13995', 'John Deere', 2, TRUE),
    ('13996', 'Case', 3, FALSE),
    ('12141', 'Nissan', 1, TRUE);
```

### 15.3 Execute a basic query

```sql
SELECT *
FROM query_lab.vehicles;
```

Then:

```sql
EXPLAIN
SELECT *
FROM query_lab.vehicles;
```

Observe the selected access method.

For a very small table, a sequential scan is likely.

### 15.4 Filter records

```sql
EXPLAIN
SELECT *
FROM query_lab.vehicles
WHERE inventory_number = '13990';
```

Observe the plan.

The UNIQUE constraint creates an index that PostgreSQL may use.

### 15.5 Analyze actual execution

```sql
EXPLAIN ANALYZE
SELECT *
FROM query_lab.vehicles
WHERE inventory_number = '13990';
```

Compare estimated and actual rows.

### 15.6 Analyze a join

```sql
EXPLAIN ANALYZE
SELECT
    v.inventory_number,
    v.brand,
    d.department_name
FROM query_lab.vehicles AS v
INNER JOIN query_lab.departments AS d
    ON v.department_id = d.department_id;
```

Identify:

- The join operation.
- The table access methods.
- The estimated rows.
- The actual rows.
- The execution time.

### 15.7 Important experiment

A five-row table is too small to demonstrate meaningful performance differences.

The optimizer may prefer sequential scans even when indexes exist.

For more representative experiments, generate a larger test dataset and compare execution plans.

Do not interpret a sequential scan on a tiny table as evidence of a missing or ineffective index.

## 16. A More Advanced Concept: Prepared Statements

Applications frequently execute similar queries with different parameter values.

For example:

```sql
SELECT *
FROM fleet.vehicles
WHERE inventory_number = $1;
```

A prepared statement allows a query to be parsed and prepared for repeated execution.

In PostgreSQL:

```sql
PREPARE find_vehicle (VARCHAR) AS
SELECT *
FROM fleet.vehicles
WHERE inventory_number = $1;
```

Execute it:

```sql
EXECUTE find_vehicle('13990');
```

Execute it again with another value:

```sql
EXECUTE find_vehicle('13994');
```

Prepared statements can reduce repeated parsing and planning overhead in appropriate workloads.

PostgreSQL may use custom or generic plans depending on the situation and its planning decisions.

A generic plan is reused across executions, while a custom plan can be generated using the supplied parameter values.

This distinction matters when parameter values have very different selectivity.

## 17. Query Execution and Database Architecture

Query processing connects several internal DBMS components.

```text
Application
    |
    v
Database Connection
    |
    v
SQL Parser
    |
    v
Semantic Analysis
    |
    v
Query Rewriter
    |
    v
Query Planner and Optimizer
    |
    v
Execution Plan
    |
    v
Query Executor
    |
    v
Storage and Buffer Management
    |
    v
Result Set
    |
    v
Application
```

This process demonstrates that query execution is not simply a search operation.

It is a coordinated interaction between:

- SQL processing.
- Database metadata.
- Statistics.
- Optimization.
- Memory.
- Storage.
- Transaction visibility.
- Execution algorithms.

A query's behavior depends on the complete database environment.

## 18. Knowledge Check

Answer these questions without consulting the previous sections.

1. What is the query execution lifecycle?
2. What is the difference between parsing and semantic analysis?
3. Why can a syntactically valid query still fail?
4. What is the purpose of query rewriting?
5. What does the query optimizer evaluate?
6. What is an execution plan?
7. What is the difference between a sequential scan and an index scan?
8. When might PostgreSQL use a bitmap scan?
9. What are the three major join algorithms?
10. What is selectivity?
11. Why does the optimizer need statistics?
12. What is the difference between EXPLAIN and EXPLAIN ANALYZE?
13. Why must EXPLAIN ANALYZE be used carefully with data-modifying statements?
14. What does the estimated cost represent?
15. Why is a sequential scan not always a performance problem?
16. How can network latency affect application response time?
17. What is the difference between a custom plan and a generic plan?

## 19. Practical Exercises

### Exercise 1 — Query lifecycle explanation

Take this query:

```sql
SELECT
    v.inventory_number,
    d.department_name
FROM fleet.vehicles AS v
INNER JOIN fleet.departments AS d
    ON v.department_id = d.department_id
WHERE v.is_active = TRUE;
```

Describe what happens at each stage:

- Reception.
- Parsing.
- Semantic analysis.
- Rewriting.
- Planning.
- Optimization.
- Execution.
- Result delivery.

### Exercise 2 — Execution plan inspection

Choose an existing table in your PostgreSQL environment.

Execute:

```sql
EXPLAIN
SELECT *
FROM your_schema.your_table;
```

Then:

```sql
EXPLAIN ANALYZE
SELECT *
FROM your_schema.your_table;
```

Document:

- Access method.
- Estimated rows.
- Actual rows.
- Estimated cost.
- Actual execution time.

### Exercise 3 — Compare filtering strategies

Create a test table with a sufficiently large number of records.

Compare:

```sql
SELECT *
FROM query_lab.vehicles
WHERE inventory_number = '13990';
```

with:

```sql
SELECT *
FROM query_lab.vehicles
WHERE is_active = TRUE;
```

Investigate why the optimizer may select different access methods.

### Exercise 4 — Query optimization review

Take three queries from a database project.

For each one:

1. Identify its purpose.
2. Identify its filtering conditions.
3. Identify its joins.
4. Run EXPLAIN ANALYZE.
5. Identify the execution plan.
6. Compare estimated and actual rows.
7. Propose a possible improvement.
8. Test the improvement.
9. Document the result.

### Exercise 5 — SIAPV architecture analysis

Consider a query that retrieves:

- Vehicle inventory number.
- Brand and model.
- Current department.
- Latest maintenance date.
- Latest odometer reading.
- Latest engine-hour reading.

Think about:

- Which tables are required?
- Which relationships are involved?
- Which joins may be necessary?
- Which columns should be indexed?
- How would you retrieve the latest measurement?
- Would a regular view or materialized view be appropriate?

Do not assume that the illustrative table structure matches the actual SIAPV implementation.

## 20. Technical Vocabulary

| English term | Spanish meaning |
|---|---|
| Query execution | Ejecución de consultas |
| Query processing | Procesamiento de consultas |
| Query parser | Analizador sintáctico de consultas |
| Parsing | Análisis sintáctico |
| Parse tree | Árbol sintáctico |
| Semantic analysis | Análisis semántico |
| Query rewriting | Reescritura de consultas |
| Query planner | Planificador de consultas |
| Query optimizer | Optimizador de consultas |
| Execution plan | Plan de ejecución |
| Query executor | Ejecutor de consultas |
| Cost estimation | Estimación de costos |
| Selectivity | Selectividad |
| Table statistics | Estadísticas de tablas |
| Sequential scan | Escaneo secuencial |
| Index scan | Escaneo mediante índice |
| Index only scan | Escaneo exclusivo del índice |
| Bitmap scan | Escaneo mediante mapa de bits |
| Nested loop join | Unión mediante bucles anidados |
| Hash join | Unión mediante tabla hash |
| Merge join | Unión por combinación ordenada |
| Result set | Conjunto de resultados |
| Actual execution time | Tiempo real de ejecución |
| Estimated rows | Filas estimadas |
| Prepared statement | Sentencia preparada |
| Generic plan | Plan genérico |
| Custom plan | Plan personalizado |
| Query performance | Rendimiento de consultas |

## 21. Key Takeaways

- SQL describes the desired result, while the DBMS determines how to retrieve it.
- Query processing involves parsing, semantic analysis, rewriting, planning, optimization, and execution.
- The parser validates SQL syntax.
- Semantic analysis checks whether the statement is meaningful within the database.
- Query rewriting transforms the query representation when appropriate.
- The optimizer evaluates possible execution strategies.
- An execution plan describes how the DBMS intends to execute a query.
- Sequential scans, index scans, and bitmap scans are different access strategies.
- Nested loop, hash, and merge joins are important join algorithms.
- Statistics help the optimizer estimate query costs and row counts.
- EXPLAIN displays the selected plan.
- EXPLAIN ANALYZE executes the query and reports actual execution information.
- A sequential scan is not necessarily inefficient.
- Query performance depends on data distribution, indexes, memory, storage, statistics, and workload.
- Performance improvements should be validated using actual execution plans.

**The ability to read and interpret execution plans is one of the most important skills in database performance engineering.**

## 22. Related Notes

### Previous concepts

- [[01 - Database Management Systems]]
- [[06 - Schema Design Principles]]
- [[04 - OLTP vs OLAP]]

### Next concepts

- [[03 - Storage Pages and Tables]]
- [[04 - Indexes]]
- [[05 - Transactions and ACID]]
- [[06 - Concurrency and Locks]]

### PostgreSQL specialization

- [[01 - PostgreSQL Architecture Overview]]
- [[03 - PostgreSQL Storage]]
- [[04 - PostgreSQL Query Planning]]
- [[05 - PostgreSQL Performance Fundamentals]]
