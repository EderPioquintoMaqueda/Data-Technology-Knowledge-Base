---

title: "Data Warehouses"  
aliases:

- "Data Warehouse Fundamentals"
    
- "Enterprise Data Warehouse"
    
- "Analytical Data Warehouse"  
    type: learning-note  
    domain: technology  
    category: data-engineering-architecture  
    subject: data-warehouses  
    level: intermediate  
    difficulty: intermediate  
    status: in-progress  
    created: 2026-11-12  
    updated: 2026-11-12  
    tags:
    
- data-warehouse
    
- data-engineering
    
- data-architecture
    
- business-intelligence
    
- analytics
    
- olap
    
- dimensional-modeling
    
- fact-table
    
- dimension-table  
    concepts:
    
- data-warehouse
    
- olap
    
- analytical-processing
    
- dimensional-modeling
    
- fact-table
    
- dimension-table
    
- star-schema
    
- snowflake-schema
    
- grain
    
- surrogate-key
    
- slowly-changing-dimension
    
- data-mart
    
- historical-data
    
- aggregation
    
- business-intelligence  
    prerequisites:
    
- "[[01 - Data Architecture Overview]]"
    
- "[[04 - OLTP vs OLAP]]"
    
- "[[03 - ETL and ELT]]"
    
- "[[04 - Data Pipelines]]"  
    related:
    
- "[[06 - Data Lakes]]"
    
- "[[07 - Lakehouse Architecture]]"
    
- "[[03 - Data Quality]]"
    
- "[[04 - Data Governance]]"
    
- "[[Data Modeling]]"  
    tools:
    
- PostgreSQL
    
- SQL
    
- Python
    
- Pandas
    
- dbt
    
- Apache Airflow
    
- Power BI  
    learning_objectives:
    
- "Define a data warehouse and explain its purpose."
    
- "Distinguish operational databases from analytical data warehouses."
    
- "Understand dimensional modeling."
    
- "Differentiate fact tables from dimension tables."
    
- "Understand grain and its importance."
    
- "Compare star and snowflake schemas."
    
- "Understand surrogate keys."
    
- "Explain slowly changing dimensions."
    
- "Understand data marts and their relationship with data warehouses."
    
- "Design a conceptual warehouse using fleet information."  
    review:  
    frequency: weekly  
    last_reviewed: null  
    next_review: null
    

---

# Data Warehouses

## 1. Introduction

A **data warehouse** is a centralized system designed to store, organize, and analyze historical data from multiple sources.

Its primary purpose is not to support individual operational transactions. Instead, it supports:

- Business analysis.
    
- Reporting.
    
- Business Intelligence.
    
- Historical analysis.
    
- Performance measurement.
    
- Strategic decision-making.
    

A data warehouse integrates information from different operational systems and organizes it for analytical queries.

```
Operational Systems
      |
      v
Data Integration
      |
      v
Transformation
      |
      v
+-------------------+
|   DATA WAREHOUSE  |
+-------------------+
      |
      +--------+--------+
      |        |        |
      v        v        v
   Reports   Dashboards Analysis
```

The warehouse becomes a trusted analytical layer between operational systems and business users.

---

## 2. Why Do Organizations Need Data Warehouses?

Operational systems are designed to execute daily business processes.

For example, an operational database may manage:

- Customer registrations.
    
- Financial transactions.
    
- Vehicle records.
    
- Maintenance records.
    
- Product inventory.
    

These systems must process transactions efficiently and preserve operational consistency.

Analytical questions are different.

Examples:

- How has maintenance spending changed over five years?
    
- Which equipment category has the highest operational cost?
    
- How many incidents occurred by region?
    
- What trends can be observed over time?
    

Running complex analytical queries directly against an operational system can create problems:

- Increased workload.
    
- Slower transactional performance.
    
- Complex queries.
    
- Inconsistent historical information.
    
- Difficult integration between multiple systems.
    

A data warehouse separates **operational processing** from **analytical processing**.

---

## 3. Main Characteristics of a Data Warehouse

A traditional data warehouse is commonly described using four characteristics.

### 3.1 Subject-oriented

Data is organized around important business subjects.

Examples:

- Customers.
    
- Products.
    
- Sales.
    
- Finance.
    
- Vehicles.
    
- Maintenance.
    

This differs from operational systems, which are often organized around individual applications or processes.

### 3.2 Integrated

A warehouse combines information from multiple sources.

Different systems may represent the same concept differently.

For example:

```
System A: ACTIVE
System B: Active
System C: 1
```

During integration, the warehouse can standardize these values.

```
Operational Status:
IN_SERVICE
```

### 3.3 Time-variant

A warehouse preserves historical information.

Operational systems often focus on the current state.

A warehouse allows questions such as:

- What was the value last year?
    
- How has the metric changed?
    
- What was the trend over time?
    

Time is therefore a central dimension in analytical systems.

### 3.4 Non-volatile

Analytical data is generally more stable than operational data.

Data is primarily:

- Loaded.
    
- Updated through controlled processes.
    
- Preserved for historical analysis.
    

Users typically do not perform individual transactional updates directly against warehouse tables.

---

## 4. Operational Database vs. Data Warehouse

|Characteristic|Operational Database|Data Warehouse|
|---|---|---|
|Primary purpose|Execute business operations|Support analysis|
|Processing type|OLTP|OLAP|
|Data focus|Current operations|Historical analysis|
|Typical operations|INSERT, UPDATE, DELETE|SELECT, aggregation|
|Schema design|Often normalized|Often dimensional|
|Users|Applications and operational staff|Analysts and decision-makers|
|Query pattern|Short transactions|Complex analytical queries|
|Data sources|Usually one application|Multiple systems|
|Historical data|Limited or operationally focused|Central component|

### Example

An operational fleet system may answer:

> What is the current status of vehicle 13990?

A data warehouse may answer:

> How has the operational availability of motor graders changed during the last three years?

Both questions are valuable, but they require different data structures.

---

# 5. Data Warehouse Architecture

A warehouse architecture commonly includes several layers.

```
+---------------------------+
|       Source Systems      |
|                           |
| PostgreSQL | APIs | Files |
+-------------+-------------+
              |
              v
+---------------------------+
|      Data Integration     |
|                           |
| ETL / ELT / Pipelines     |
+-------------+-------------+
              |
              v
+---------------------------+
|       Staging Layer       |
|                           |
| Temporary Raw Data        |
+-------------+-------------+
              |
              v
+---------------------------+
|       Data Warehouse      |
|                           |
| Integrated Historical     |
| Analytical Data           |
+-------------+-------------+
              |
              v
+---------------------------+
|      Consumption Layer    |
|                           |
| BI | Reports | Analytics  |
+---------------------------+
```

Each layer has a different responsibility.

---

## 6. Source Systems

Source systems contain the original operational data.

Examples include:

- PostgreSQL databases.
    
- ERP systems.
    
- CRM platforms.
    
- APIs.
    
- Excel files.
    
- CSV files.
    
- Financial applications.
    

The warehouse does not necessarily replace these systems.

Instead, it receives copies of relevant information for analytical purposes.

---

## 7. The Staging Layer

The **staging layer** is an intermediate area where data can be stored before final transformation and loading.

Its purpose may include:

- Temporary storage.
    
- Initial validation.
    
- Data cleaning.
    
- Source reconciliation.
    
- Transformation preparation.
    

Example:

```
Operational Database
        |
        v
   Raw Extraction
        |
        v
   Staging Tables
        |
        v
Transformation
        |
        v
Data Warehouse
```

The staging layer helps separate raw source information from curated analytical data.

---

## 8. Dimensional Modeling

One of the most important approaches for designing data warehouses is **dimensional modeling**.

Dimensional modeling organizes information into two primary categories:

- Facts.
    
- Dimensions.
    

```
              DIMENSIONS
           /      |      \
          v       v       v
       Time    Vehicle   Location
          \       |       /
           \      |      /
             FACT TABLE
```

This structure is designed to make analytical queries easier to understand and execute.

---

# 9. Fact Tables

A **fact table** stores measurable business events.

Examples of facts include:

- Sales transactions.
    
- Maintenance costs.
    
- Vehicle mileage.
    
- Equipment operating hours.
    
- Number of incidents.
    

A fact table usually contains:

- Measurements.
    
- Foreign keys to dimensions.
    

### Example: Maintenance Fact Table

|maintenance_key|date_key|vehicle_key|location_key|cost|
|---|---|---|---|---|
|1|20261101|101|20|15,000|
|2|20261102|102|21|8,500|
|3|20261102|101|20|4,200|

The table records measurable events.

---

## 10. Dimension Tables

A **dimension table** provides descriptive context for facts.

Examples:

- Date.
    
- Vehicle.
    
- Location.
    
- Supplier.
    
- Employee.
    

### Example: Vehicle Dimension

|vehicle_key|inventory_number|manufacturer|model|category|
|---|---|---|---|---|
|101|13990|Shantui|Motor Grader|Heavy Equipment|
|102|13994|Caterpillar|Backhoe Loader|Heavy Equipment|

The dimension helps users interpret the facts.

For example:

```
Maintenance Cost
      |
      +-- Date
      |
      +-- Vehicle
      |
      +-- Location
```

---

# 11. Facts and Dimensions

A useful question when designing a warehouse is:

> Is this information something we measure, or something that describes the measurement?

### Facts

Facts usually answer:

- How much?
    
- How many?
    
- How long?
    

Examples:

- Cost.
    
- Quantity.
    
- Mileage.
    
- Operating hours.
    

### Dimensions

Dimensions provide context:

- Who?
    
- What?
    
- Where?
    
- When?
    
- Which category?
    

Examples:

- Vehicle.
    
- Date.
    
- Location.
    
- Manufacturer.
    

### Example

```
Fact:
Maintenance Cost = 15,000 MXN

Dimensions:
Date = 2026-11-01
Vehicle = 13990
Category = Heavy Equipment
Location = Central Workshop
```

---

# 12. Grain

**Grain** defines exactly what one row in a fact table represents.

This is one of the most important decisions in dimensional modeling.

### Example

Suppose we create a maintenance fact table.

Possible grains include:

**Option A:**

> One row represents one maintenance event.

**Option B:**

> One row represents one vehicle per day.

**Option C:**

> One row represents one vehicle per month.

These designs answer different questions.

### Example: Event-level grain

|date|vehicle|maintenance_event|cost|
|---|---|---|---|
|2026-11-01|13990|Oil Change|2,500|
|2026-11-01|13990|Tire Repair|1,800|

### Example: Monthly grain

|month|vehicle|total_cost|
|---|---|---|
|2026-11|13990|4,300|

The event-level table provides more detail.

The monthly table is already aggregated.

### Grain principle

Define the grain **before** defining columns.

A clear grain prevents inconsistent rows and incorrect aggregations.

---

# 13. Measures

Measures are numerical values stored in fact tables.

Examples:

- Cost.
    
- Revenue.
    
- Quantity.
    
- Mileage.
    
- Operating hours.
    

Measures can behave differently during aggregation.

## 13.1 Additive measures

Can be summed across all dimensions.

Example:

```
Maintenance Cost
```

Daily costs can generally be added to calculate monthly or annual costs.

## 13.2 Semi-additive measures

Can be added across some dimensions but not all.

Example:

```
Daily Account Balance
```

Balances can be added across accounts but not usually across time.

## 13.3 Non-additive measures

Cannot be meaningfully summed.

Examples:

- Percentage.
    
- Ratio.
    
- Average.
    

For example:

```
Average Maintenance Cost
```

Summing averages can produce misleading results.

The correct calculation may require the original values.

---

# 14. Star Schema

The **star schema** is one of the most common dimensional models.

A central fact table connects directly to multiple dimension tables.

```
              +-------------+
              | Dim Date    |
              +-------------+
                     |
                     |
+-------------+     |     +-------------+
| Dim Vehicle |-----+-----| Dim Location|
+-------------+     |     +-------------+
                     |
              +-------------+
              | Fact        |
              | Maintenance |
              +-------------+
                     |
                     |
              +-------------+
              | Dim Type    |
              +-------------+
```

The structure resembles a star.

### Advantages

- Easy to understand.
    
- Simple analytical queries.
    
- Good for BI tools.
    
- Fewer joins than highly normalized structures.
    

### Limitation

Dimension tables may contain some repeated descriptive information.

In analytical systems, this trade-off is often acceptable.

---

# 15. Snowflake Schema

A **snowflake schema** normalizes some dimension tables into additional related tables.

Example:

```
          +-------------+
          | Dim Vehicle |
          +-------------+
                 |
                 v
          +-------------+
          | Dim Model   |
          +-------------+
                 |
                 v
          +-------------+
          | Dim Brand   |
          +-------------+

                 |
                 v

          +-------------+
          | Fact        |
          | Maintenance |
          +-------------+
```

### Advantages

- Reduced data duplication.
    
- More normalized dimension structures.
    
- Centralized attribute management.
    

### Limitations

- More joins.
    
- More complex queries.
    
- Potentially less intuitive for business users.
    

---

# 16. Star Schema vs. Snowflake Schema

|Characteristic|Star Schema|Snowflake Schema|
|---|---|---|
|Dimension structure|Denormalized|More normalized|
|Number of joins|Lower|Higher|
|Query simplicity|Higher|Lower|
|Storage redundancy|Higher|Lower|
|BI usability|Usually simpler|Can be more complex|

Neither design is automatically correct.

The decision depends on:

- Data complexity.
    
- Performance requirements.
    
- Governance.
    
- Maintenance.
    
- User needs.
    

---

# 17. Keys in Data Warehouses

Data warehouses use different types of keys.

## 17.1 Natural key

A natural key originates from the business domain.

Examples:

- Vehicle inventory number.
    
- Customer number.
    
- Product code.
    

Example:

```
Inventory Number: 13990
```

Natural keys may change or may have different meanings across source systems.

---

## 17.2 Surrogate key

A surrogate key is an artificial identifier created by the warehouse.

Example:

```
vehicle_key = 101
```

The warehouse may use:

```
vehicle_key = 101
inventory_number = 13990
```

The surrogate key becomes the internal warehouse identifier.

### Benefits

- Independent from source systems.
    
- Supports historical tracking.
    
- Avoids dependence on changing business identifiers.
    
- Simplifies relationships.
    

---

# 18. Slowly Changing Dimensions

A **Slowly Changing Dimension (SCD)** is a technique for handling changes to descriptive information over time.

For example, a vehicle may change:

- Assigned location.
    
- Operational category.
    
- Responsible department.
    

The warehouse must decide how to handle the change.

---

## 18.1 SCD Type 1: Overwrite

The old value is replaced.

```
Before:
Vehicle 13990 -> Location A

After:
Vehicle 13990 -> Location B
```

Historical information about Location A is lost.

### Use case

Use Type 1 when historical values are not important.

---

## 18.2 SCD Type 2: Historical rows

A new dimension record is created when the value changes.

```
vehicle_key | inventory_number | location | valid_from | valid_to
------------+------------------+----------+------------+----------
101         | 13990            | Location A | 2025-01-01 | 2026-06-30
205         | 13990            | Location B | 2026-07-01 | NULL
```

The warehouse preserves history.

### Use case

Use Type 2 when historical analysis requires the correct value for each period.

---

## 18.3 SCD Type 3: Limited history

The dimension stores the current and previous value.

```
vehicle_key | current_location | previous_location
------------+------------------+------------------
101         | Location B       | Location A
```

This approach preserves limited historical information.

---

# 19. Date Dimensions

Time is fundamental to analytical systems.

A common practice is to create a dedicated date dimension.

Example:

|date_key|full_date|day|month|quarter|year|
|---|---|---|---|---|---|
|20261101|2026-11-01|1|November|Q4|2026|

Instead of repeatedly calculating date attributes in every query, analytical models can join to the date dimension.

A date dimension can also contain:

- Day of week.
    
- Week number.
    
- Fiscal period.
    
- Business day indicator.
    
- Holiday indicator.
    

---

# 20. Data Marts

A **data mart** is a specialized analytical dataset focused on a particular business area.

Examples:

- Finance data mart.
    
- Sales data mart.
    
- Human resources data mart.
    
- Fleet data mart.
    

```
              DATA WAREHOUSE
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
 Finance Mart    Sales Mart    Fleet Mart
```

Data marts can make information easier to organize and consume for specific teams.

---

# 21. Enterprise Data Warehouse

An **Enterprise Data Warehouse (EDW)** integrates information from multiple areas of an organization.

It provides a centralized analytical foundation.

Example:

```
Finance System -----+
                    |
Fleet System -------+
                    |
HR System ----------+----> Enterprise Data Warehouse
                    |
Operations ---------+
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Finance      Operations     BI
```

An EDW requires strong:

- Data governance.
    
- Integration standards.
    
- Data quality controls.
    
- Security policies.
    
- Ownership definitions.
    

---

# 22. Data Warehouse Design Process

A structured design process can follow these steps.

## Step 1: Define the business process

Identify what is being analyzed.

Examples:

- Sales.
    
- Maintenance.
    
- Inventory.
    
- Financial expenses.
    

## Step 2: Define the grain

Determine what one fact table row represents.

Example:

> One row represents one maintenance event.

## Step 3: Identify dimensions

Determine the descriptive context.

Examples:

- Date.
    
- Vehicle.
    
- Location.
    
- Maintenance type.
    

## Step 4: Identify facts

Determine measurable values.

Examples:

- Cost.
    
- Duration.
    
- Mileage.
    
- Operating hours.
    

## Step 5: Define keys

Determine how facts connect to dimensions.

## Step 6: Define historical behavior

Determine whether changes require SCD Type 1, 2, or another strategy.

## Step 7: Define quality controls

Determine:

- Required fields.
    
- Valid ranges.
    
- Duplicate rules.
    
- Reconciliation procedures.
    

---

# 23. Example: Fleet Data Warehouse

A conceptual fleet warehouse can analyze operational and maintenance information.

## 23.1 Business process

Analyze maintenance activity.

## 23.2 Grain

One row represents:

> One maintenance event performed on one vehicle.

## 23.3 Fact table

```
fact_maintenance
```

Possible columns:

|Column|Description|
|---|---|
|maintenance_key|Fact identifier|
|date_key|Reference to date|
|vehicle_key|Reference to vehicle|
|location_key|Reference to location|
|maintenance_type_key|Reference to maintenance type|
|cost|Maintenance cost|
|labor_hours|Hours of work|

## 23.4 Dimensions

```
dim_date
dim_vehicle
dim_location
dim_maintenance_type
```

## 23.5 Star schema

```
                 dim_date
                    |
                    |
dim_vehicle ---- fact_maintenance ---- dim_location
                    |
                    |
          dim_maintenance_type
```

## 23.6 Example analytical questions

This model could answer:

- What was the total maintenance cost by month?
    
- Which equipment category has the highest maintenance cost?
    
- How many maintenance events occurred by location?
    
- What type of maintenance requires the most labor hours?
    
- How has maintenance spending changed over time?
    

---

# 24. SQL Example

Suppose the warehouse contains:

```
fact_maintenance
dim_date
dim_vehicle
```

A query could calculate maintenance cost by year and vehicle category.

```
SELECT
    d.year,
    v.category,
    SUM(f.cost) AS total_maintenance_cost
FROM fact_maintenance AS f
INNER JOIN dim_date AS d
    ON f.date_key = d.date_key
INNER JOIN dim_vehicle AS v
    ON f.vehicle_key = v.vehicle_key
GROUP BY
    d.year,
    v.category
ORDER BY
    d.year,
    total_maintenance_cost DESC;
```

The query combines:

- A fact: maintenance cost.
    
- A time dimension.
    
- A vehicle dimension.
    

This is the fundamental pattern of dimensional analysis.

---

# 25. Data Warehouse Loading

Warehouse data is loaded through controlled pipelines.

A simplified process may be:

```
Source Systems
      |
      v
Extract Data
      |
      v
Staging
      |
      v
Validate
      |
      v
Load Dimensions
      |
      v
Load Facts
      |
      v
Reconcile
      |
      v
Publish
```

Dimensions are often loaded before facts because fact records depend on dimension keys.

---

# 26. Incremental Loading

Reloading an entire warehouse can be expensive.

An incremental process loads only new or changed information.

Example:

```
Last successful load:
2026-11-11 02:00

Current extraction:
Records changed after 2026-11-11 02:00
```

Incremental loading requires a reliable way to identify changes.

Possible methods include:

- Updated timestamps.
    
- Change Data Capture.
    
- Version numbers.
    
- Source transaction logs.
    

---

# 27. Data Quality in Warehouses

A warehouse is only useful if its information can be trusted.

Common quality controls include:

### Completeness

Are all expected records present?

### Validity

Do values follow expected rules?

### Uniqueness

Are duplicate records controlled?

### Consistency

Do related systems represent the information consistently?

### Timeliness

Is the data sufficiently current?

### Accuracy

Does the data correctly represent the source or business reality?

---

# 28. Data Lineage

**Data lineage** describes the path that information follows from its origin to its analytical destination.

Example:

```
Operational System
        |
        v
Raw Extraction
        |
        v
Staging Table
        |
        v
Transformation
        |
        v
Fact Table
        |
        v
Power BI Dashboard
```

Lineage helps answer:

- Where did this value come from?
    
- Which transformation created it?
    
- Which source systems contributed to it?
    
- What reports are affected by a data change?
    

---

# 29. Common Data Warehouse Mistakes

## 29.1 Starting with technology instead of business questions

The warehouse should support real analytical requirements.

Start by understanding:

- What decisions need support?
    
- What metrics are required?
    
- What historical analysis is valuable?
    

## 29.2 Defining facts without grain

Without a clear grain, aggregations can produce incorrect results.

## 29.3 Mixing grains in one fact table

For example:

```
One row = individual maintenance event
```

should not be mixed with:

```
One row = monthly vehicle total
```

These represent different levels of detail.

## 29.4 Treating all dimensions as static

Descriptive information can change.

Historical behavior must be intentionally designed.

## 29.5 Ignoring source data quality

A warehouse does not automatically fix incorrect source information.

Validation and reconciliation remain necessary.

## 29.6 Excessive normalization

Highly normalized analytical models can become difficult for users and BI tools to query.

## 29.7 Excessive denormalization

Duplicating information without control can create inconsistency and maintenance problems.

---

# 30. Practical Exercises

## Exercise 1: Identify facts and dimensions

For a vehicle maintenance system, classify the following as facts or dimensions:

- Maintenance cost.
    
- Vehicle.
    
- Date.
    
- Labor hours.
    
- Location.
    
- Maintenance type.
    
- Inventory number.
    

Explain your reasoning.

---

## Exercise 2: Define the grain

Design a fact table for fuel consumption.

Choose one grain:

- One fuel transaction.
    
- One vehicle per day.
    
- One vehicle per month.
    

Explain what analytical questions your selected grain can answer.

---

## Exercise 3: Design a star schema

Create a star schema for maintenance analysis.

Include:

- One fact table.
    
- At least four dimensions.
    
- Keys.
    
- Measures.
    

Draw the relationships.

---

## Exercise 4: SCD design

A vehicle changes its assigned department.

Decide whether to use:

- Type 1.
    
- Type 2.
    
- Type 3.
    

Explain the consequences of each option.

---

## Exercise 5: Write an analytical query

Write a SQL query that calculates:

> Total maintenance cost by month and equipment category.

Identify:

- The fact table.
    
- The dimensions.
    
- The measure.
    
- The grouping attributes.
    

---

## Exercise 6: Design a SIAPV analytical warehouse

Using SIAPV as a conceptual example, design a warehouse that supports:

- Fleet inventory analysis.
    
- Mileage analysis.
    
- Horometer analysis.
    
- Maintenance analysis.
    
- Incident analysis.
    

Define:

1. Business processes.
    
2. Fact tables.
    
3. Dimensions.
    
4. Grain for each fact table.
    
5. Measures.
    
6. Historical requirements.
    
7. Data quality controls.
    

---

# 31. Knowledge Check

1. What is a data warehouse?
    
2. How does a warehouse differ from an operational database?
    
3. What does subject-oriented mean?
    
4. Why is historical information important?
    
5. What is dimensional modeling?
    
6. What is a fact table?
    
7. What is a dimension table?
    
8. What is grain?
    
9. Why should grain be defined before columns?
    
10. What is an additive measure?
    
11. What is the difference between a star schema and a snowflake schema?
    
12. What is a surrogate key?
    
13. Why are surrogate keys useful?
    
14. What is a Slowly Changing Dimension?
    
15. What is the difference between SCD Type 1 and Type 2?
    
16. What is a date dimension?
    
17. What is a data mart?
    
18. What is incremental loading?
    
19. What is data lineage?
    
20. Why is data quality essential in a warehouse?
    

---

# 32. Glossary

|Term|Definition|
|---|---|
|Data Warehouse|Centralized system for historical and analytical data|
|OLAP|Online Analytical Processing|
|Fact Table|Table containing measurable business events|
|Dimension Table|Table containing descriptive context|
|Dimensional Modeling|Analytical modeling using facts and dimensions|
|Grain|Level of detail represented by one row|
|Measure|Numerical value used for analysis|
|Additive Measure|Measure that can be summed across dimensions|
|Semi-additive Measure|Measure that can be summed across some dimensions|
|Non-additive Measure|Measure that should not be directly summed|
|Star Schema|Model with a central fact table and direct dimensions|
|Snowflake Schema|Model with normalized dimension structures|
|Natural Key|Business identifier originating from the source|
|Surrogate Key|Artificial identifier created by the warehouse|
|Slowly Changing Dimension|Strategy for handling changing dimension attributes|
|SCD Type 1|Overwrite previous value|
|SCD Type 2|Create a new historical record|
|SCD Type 3|Preserve limited previous values|
|Data Mart|Analytical dataset for a specific business area|
|EDW|Enterprise Data Warehouse|
|Staging Layer|Intermediate area for data preparation|
|Incremental Load|Loading only new or changed data|
|Data Lineage|Record of data origin and transformation path|

---

# 33. Connections to Other Topics

## Previous notes

- [[01 - Data Architecture Overview]]
    
- [[04 - OLTP vs OLAP]]
    
- [[03 - ETL and ELT]]
    
- [[04 - Data Pipelines]]
    

## Next notes

- [[06 - Data Lakes]]
    
- [[07 - Lakehouse Architecture]]
    

## Related concepts

- [[Data Modeling]]
    
- [[Data Quality]]
    
- [[Data Governance]]
    
- [[SQL Aggregations]]
    
- [[PostgreSQL Architecture Overview]]
    

## Practical applications

- [[SIAPV Data Architecture]]
    
- [[SIAPV Database Design Review]]
    
- [[Business Intelligence Fundamentals]]
    

---

# 34. Key Takeaways

1. A data warehouse is designed for historical analysis and decision support.
    
2. It separates analytical workloads from operational transaction processing.
    
3. Warehouses integrate information from multiple sources.
    
4. Dimensional modeling organizes analytical data into facts and dimensions.
    
5. Fact tables store measurable events.
    
6. Dimension tables provide descriptive context.
    
7. Grain defines exactly what one row represents.
    
8. Star schemas simplify analytical queries.
    
9. Snowflake schemas normalize dimension structures.
    
10. Surrogate keys provide stable internal identifiers.
    
11. Slowly Changing Dimensions manage historical changes.
    
12. Data marts provide specialized analytical views.
    
13. Incremental loading improves processing efficiency.
    
14. Data quality and lineage are essential for trustworthy analytics.
    
15. A warehouse transforms operational data into a foundation for Business Intelligence.
    

---

> [!summary]  
> A **Data Warehouse** is not simply a large database. It is an analytical architecture designed to integrate, preserve, organize, and make historical information available for analysis. Its most important design decisions include the **business process, grain, facts, dimensions, keys, and historical behavior**.

```

**Next topic:** [[06 - Data Lakes]]

En el siguiente archivo veremos un enfoque diferente: los **Data Lakes**, diseñados para almacenar grandes volúmenes de información estructurada, semiestructurada y no estructurada. Después podremos comparar claramente **Data Warehouse vs. Data Lake vs. Lakehouse**, que es una distinción muy importante en arquitecturas modernas de datos.
```