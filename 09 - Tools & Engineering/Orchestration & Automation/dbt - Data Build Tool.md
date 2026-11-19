---
title: "dbt - Data Build Tool"
tags: [tools, dbt, sql, analytics-engineering, elt]
---

# 🧱 dbt — Data Build Tool

**dbt** turns SQL `SELECT` statements into tested, documented, version-controlled tables and views inside your warehouse. It is the "T" of **ELT** and the core tool of the *analytics engineer*.

```
raw tables ──► staging models (clean) ──► intermediate ──► marts (facts & dimensions) ──► BI
   sources         stg_*.sql                 int_*.sql        fct_*.sql / dim_*.sql
```

---

## 🟢 Beginner
```bash
uv add dbt-core dbt-postgres          # or dbt-duckdb, dbt-snowflake, dbt-bigquery...
dbt init datashop_dbt
dbt debug                             # test the connection (profiles.yml)
```
```yaml
# ~/.dbt/profiles.yml
datashop_dbt:
  target: dev
  outputs:
    dev:
      type: postgres
      host: localhost
      user: shop
      password: "{{ env_var('DB_PASSWORD') }}"
      port: 5432
      dbname: datashop
      schema: analytics
      threads: 4
```
### A model is just a SELECT
```sql
-- models/staging/stg_orders.sql
SELECT
    id            AS order_id,
    customer_id,
    employee_id,
    order_date,
    LOWER(status) AS status
FROM {{ source('shop', 'orders') }}
```
```sql
-- models/marts/fct_sales.sql
SELECT o.order_id, o.customer_id, o.order_date, i.product_id,
       i.quantity * i.unit_price * (1 - i.discount) AS revenue
FROM {{ ref('stg_orders') }} o
JOIN {{ ref('stg_order_items') }} i USING (order_id)
WHERE o.status IN ('paid', 'shipped')
```
`ref()` builds the **dependency graph (DAG)** automatically; `dbt run` builds everything in the right order.

## 🟡 Intermediate
### Sources and tests
```yaml
# models/staging/_sources.yml
version: 2
sources:
  - name: shop
    schema: public
    tables:
      - name: orders
      - name: order_items
models:
  - name: stg_orders
    columns:
      - name: order_id
        tests: [unique, not_null]
      - name: status
        tests:
          - accepted_values:
              values: ['pending', 'paid', 'shipped', 'cancelled', 'refunded']
      - name: customer_id
        tests:
          - relationships: {to: ref('stg_customers'), field: customer_id}
```
```bash
dbt build          # run + test in DAG order
dbt test --select stg_orders
dbt docs generate && dbt docs serve     # documentation website with lineage graph
```
### Materializations
| Type | Result | Use |
|---|---|---|
| `view` (default) | a view | light staging models |
| `table` | rebuilt table | marts queried often |
| `incremental` | only new rows appended/merged | big fact tables |
| `ephemeral` | inlined CTE | helper logic |

```sql
{{ config(materialized='incremental', unique_key='order_id') }}
SELECT * FROM {{ ref('stg_orders') }}
{% if is_incremental() %}
  WHERE order_date > (SELECT MAX(order_date) FROM {{ this }})
{% endif %}
```

## 🔴 Advanced
- **Jinja macros** for reusable SQL (`{% macro cents_to_dollars(col) %}...{% endmacro %}`).
- **Packages:** `dbt_utils`, `dbt_expectations` (`packages.yml` → `dbt deps`).
- **Snapshots** = SCD Type 2 history of changing dimensions.
- **Selectors:** `dbt build --select state:modified+` in CI (only changed models and their children).
- **Seeds:** small CSV lookup tables (`dbt seed`).
- **Semantic layer / metrics**, exposures, model contracts (enforced column types).

## 🧪 Practice
Build a dbt project on top of [[SQL Practice Database]]: `stg_*` for each table, `dim_customers`, `dim_products`, `fct_sales`, with tests on keys and accepted values.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| model | /ˈmɑː.dəl/ | **MA**-del | modelo |
| lineage | /ˈlɪn.i.ɪdʒ/ | **LI**-ni-ich | linaje (trazabilidad) |
| materialization | /məˌtɪr.i.ə.ləˈzeɪ.ʃən/ | ma-ti-ria-li-**ZÉI**-shon | materialización |
| incremental | /ˌɪŋ.krəˈmen.t̬əl/ | in-kre-**MEN**-tal | incremental |
| seed | /siːd/ | sid | semilla (datos fijos) |
| snapshot | /ˈsnæp.ʃɑːt/ | **SNAP**-shat | instantánea |

Related: [[03 - ETL and ELT]] · [[05 - Data Warehouses]] · [[SQL-I04 Set Operations, Views and Data Modeling in Practice]] · 🏠 [[00 - Tools Roadmap]]
