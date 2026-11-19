---
title: "Apache Airflow Fundamentals"
tags: [tools, airflow, orchestration, pipelines]
---

# 🌬️ Apache Airflow Fundamentals

Airflow schedules and monitors **workflows as code**. A workflow is a **DAG** (Directed Acyclic Graph) of **tasks** with dependencies, retries, alerts and a web UI.

| cron | Airflow |
|---|---|
| runs commands at times | runs **dependent** tasks |
| no retries, no UI | retries, backfills, UI, logs, alerts |
| one machine | scalable workers |

---

## 🟢 Beginner — run it locally
```bash
uv add "apache-airflow==3.*"
airflow standalone              # dev mode: DB + scheduler + UI at http://localhost:8080
```
(or the official `docker compose` file from the Airflow docs).

### First DAG (TaskFlow API)
```python
# dags/daily_sales.py
from datetime import datetime, timedelta
from airflow.sdk import dag, task        # Airflow 3 (Airflow 2: from airflow.decorators import dag, task)

@dag(
    schedule="30 2 * * *",               # every day 02:30
    start_date=datetime(2026, 1, 1),
    catchup=False,
    default_args={"retries": 3, "retry_delay": timedelta(minutes=5)},
    tags=["sales"],
)
def daily_sales():
    @task
    def extract() -> str:
        # download from API, save to data lake; return the file path
        return "/data/raw/orders_2026-11-18.parquet"

    @task
    def transform(path: str) -> str:
        import polars as pl
        out = path.replace("raw", "clean")
        pl.read_parquet(path).filter(pl.col("status") != "cancelled").write_parquet(out)
        return out

    @task
    def load(path: str) -> None:
        ...  # COPY into PostgreSQL (see PY-A05)

    load(transform(extract()))           # dependencies: extract >> transform >> load

daily_sales()
```

## 🟡 Intermediate
| Concept | Meaning |
|---|---|
| **DAG run** | one execution for a logical date / data interval |
| **Task instance** | one task inside one DAG run (states: queued, running, success, failed, up_for_retry) |
| **Operator** | template for a task (`BashOperator`, `SQLExecuteQueryOperator`, `PythonOperator`) |
| **Sensor** | waits for a condition (file arrives, partition exists) |
| **Connection / Variable** | credentials and config stored in Airflow, not in code |
| **XCom** | small messages between tasks (pass paths, not DataFrames!) |
| **Backfill** | run past intervals |

```python
from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator
refresh = SQLExecuteQueryOperator(
    task_id="refresh_mv", conn_id="pg_datashop",
    sql="REFRESH MATERIALIZED VIEW mv_monthly_revenue;",
)
```
Use the **data interval** for idempotent loads: process exactly `[data_interval_start, data_interval_end)`, so re-runs and backfills give the same result.

## 🔴 Advanced
- Dynamic task mapping: `process.expand(file=list_files())` — one task per file.
- Assets/datasets: trigger a DAG when an upstream table is updated (data-aware scheduling).
- Run dbt from Airflow (Cosmos package) → [[dbt - Data Build Tool]].
- Executors: Local, Celery, Kubernetes. Managed: Astronomer, AWS MWAA, Google Cloud Composer.
- Best practices: tasks idempotent & atomic; no heavy work at DAG parse time; secrets in connections/secret backends; alerts on failure (Slack/email); SLAs.
- Alternatives: **Dagster** (asset-centric), **Prefect** (Pythonic), **n8n** (low-code) → [[ORCH_01_Architecture_Webhooks_and_n8n_Basics]].

## 🧪 Practice
Create a DAG that: downloads a public CSV → validates it with Pandera → loads it into PostgreSQL → refreshes a materialized view; with 3 retries and a failure alert.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| scheduler | /ˈskedʒ.uː.lɚ/ | **SKE**-yu-ler | planificador |
| task | /tæsk/ | task | tarea |
| directed acyclic graph | /dəˈrek.tɪd eɪˈsaɪ.klɪk ɡræf/ | di-**REK**-tid ei-**SÁI**-klik graf | grafo acíclico dirigido |
| upstream / downstream | /ˌʌpˈstriːm/ /ˌdaʊnˈstriːm/ | ap-**STRIM** / daun-**STRIM** | aguas arriba / abajo (dependencias) |
| backfill | /ˈbæk.fɪl/ | **BAK**-fil | rellenar ejecuciones pasadas |
| sensor | /ˈsen.sɚ/ | **SEN**-ser | sensor |
| alert | /əˈlɝːt/ | a-**LERT** | alerta |

Related: [[04 - Data Pipelines]] · [[LIN_03_Bash_Scripting_and_Automation]] · 🏠 [[00 - Tools Roadmap]]
