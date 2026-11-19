---
title: Knowledge Map
type: moc
tags: [knowledge-map, connections]
---

# 🕸️ Knowledge Map

Nothing exists in isolation. This map shows how the areas depend on each other.

```
                    ┌──────────────┐
                    │ Mathematics  │
                    └──────┬───────┘
                           ▼
┌─────────┐   ┌────────────────────┐   ┌──────────────────┐
│ English │──►│ Python & libraries │◄─►│       SQL        │
└────┬────┘   └─────────┬──────────┘   └────────┬─────────┘
     │                  ▼                       ▼
     │        ┌──────────────────┐   ┌──────────────────────────┐
     │        │  Data analysis   │◄──│ Databases & data eng.    │
     │        └────────┬─────────┘   └────────────┬─────────────┘
     │                 ▼                          ▼
     │        ┌──────────────────┐   ┌──────────────────────────┐
     └───────►│ Visualization/BI │   │ Machine learning → AI    │
              └──────────────────┘   └──────────────────────────┘
        Tools (Git, Linux, Docker, dbt, Airflow) support everything
```

## One concept, many areas — examples
### Missing data (NULL)
- Statistics: imputation, MCAR/MAR/MNAR → [[pandas_data_cleaning_missing]]
- Python: `None`, `np.nan`, `pd.NA` → [[numpy_boolean_masking]]
- SQL: `IS NULL`, `COALESCE`, three-valued logic → [[SQL-I02 CASE, NULLs, Strings and Dates]]
- Modeling: `NOT NULL` constraints → [[03 - Keys and Constraints]]
- BI: "(Blank)" in Power BI → [[Power BI]]

### Joining data
- SQL joins → [[SQL-B03 JOINs]] · pandas → [[pandas_merging_joining]] · Polars → [[Polars - Beginner to Advanced]] · Spark → [[14 - Spark Joins and Aggregations]] · theory → [[02 - Relationships and Cardinality]]

### Window functions
- SQL → [[SQL-I03 Window Functions]] · pandas `transform` → [[pandas_groupby_aggregation]] · Polars `.over()` · DAX `CALCULATE` context → [[DAX_Maestro]]

### Performance
- Python → [[PY-A04 Performance, Profiling and Memory]] · NumPy memory → [[numpy_memory_architecture]] · SQL → [[SQL-A01 Query Optimization and Indexes]] · internals → [[04 - Indexes]] · scale → [[02 - Database Scalability]]

### Transactions & reliability
- SQL → [[SQL-A02 Transactions, Isolation and Locking in Practice]] · Python → [[PY-A05 Python and Databases]] · theory → [[05 - Transactions and ACID]] · pipelines → [[Apache Airflow Fundamentals]]

🏠 [[Home]]

#knowledge-map
