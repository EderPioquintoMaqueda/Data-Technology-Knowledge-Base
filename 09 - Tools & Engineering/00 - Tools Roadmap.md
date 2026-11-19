---
title: Tools Roadmap
type: moc
tags: [tools, engineering, roadmap, moc]
---

# 🛠️ Tools & Engineering Roadmap

> Tools don't replace knowledge, but they multiply it. Learn them **in this order**, each one when you first need it.

## 🟢 Level 1 — Daily tools
| Tool | What for | Notes |
|---|---|---|
| **VS Code + Jupyter** | writing code & exploring data | [[Developer Environment - VS Code, Jupyter and Terminal]] |
| **Terminal (Bash / PowerShell / WSL)** | running everything | [[LIN_01_Command_Line_Navigation_and_File_Management]] |
| **Git & GitHub** | version control, collaboration | [[GIT_01_Beginner_Basics_Commit_and_Branch]] |
| **DBeaver / pgAdmin** | querying databases visually | [[PostgreSQL Practical Guide]] |
| **Excel / Power BI** | business users' world | [[Power BI]] |

## 🟡 Level 2 — Professional workflow
| Tool | What for | Notes |
|---|---|---|
| Git branching, PRs, conflicts | team work | [[GIT_02_Intermediate_Merging_Conflicts_and_Remote_Repositories]] |
| Linux permissions, processes, SSH | servers | [[LIN_02_Permissions_Processes_and_System_Monitoring]] |
| **Docker & Compose** | reproducible environments, local databases | [[DOCKER_01_Beginner_Containers_Images_and_Dockerfile]] → [[DOCKER_02_Intermediate_Volumes_Networks_and_Compose]] |
| uv / venv / pyproject | Python dependencies | [[PY-I03 Environments, Packages and Project Structure]] |
| pytest, ruff, pre-commit | quality | [[PY-I05 Testing, Type Hints and Code Quality]] |
| REST APIs & FastAPI | serving data and models | [[APIs y Cloud]] · [[FastAPI_ML_Deployment]] |

## 🔴 Level 3 — Data engineering stack
| Tool | What for | Notes |
|---|---|---|
| Bash scripting + cron | simple automation | [[LIN_03_Bash_Scripting_and_Automation]] |
| **dbt** | SQL transformations as code, tests, docs | [[dbt - Data Build Tool]] |
| **Apache Airflow** | orchestrating pipelines with dependencies | [[Apache Airflow Fundamentals]] |
| n8n / webhooks | low-code automation | [[ORCH_01_Architecture_Webhooks_and_n8n_Basics]] → [[ORCH_02_Automated_Data_Pipelines_and_Database_Sync]] → [[ORCH_03_Pro_Error_Handling_and_Self_Hosted_Scaling]] |
| Docker in production | lean, secure images | [[DOCKER_03_Pro_MultiStage_Builds_and_Production_Best_Practices]] |
| Git advanced, CI/CD | automation of quality | [[GIT_03_Pro_Rebase_Stashing_and_Collaborative_Workflows]] |
| Spark | big data processing | [[10 - Apache Spark Fundamentals]] |
| Cloud (AWS/Azure/GCP/IBM Cloud) | managed databases, storage, compute | [[08 - Cloud Data Architecture]] |

## 🧭 The modern data stack (how the tools fit)
```
Sources (APIs, apps, files)
   │  Python / Airbyte / Fivetran  (Extract & Load)
   ▼
Warehouse / Lakehouse  (PostgreSQL, Snowflake, BigQuery, Databricks, DuckDB)
   │  dbt  (Transform with SQL, test, document)
   ▼
Marts (star schemas)  ──►  BI (Power BI, Tableau, Looker)  /  ML (scikit-learn, PyTorch)
         ▲
   Airflow / Dagster / Prefect orchestrate everything · Git + CI/CD version everything · Docker packages everything
```

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| tool | /tuːl/ | tul | herramienta |
| workflow | /ˈwɝːk.floʊ/ | **UERK**-flou | flujo de trabajo |
| pipeline | /ˈpaɪp.laɪn/ | **PÁIP**-lain | canalización / flujo de datos |
| orchestration | /ˌɔːr.kəˈstreɪ.ʃən/ | or-kes-**TRÉI**-shon | orquestación |
| deployment | /dɪˈplɔɪ.mənt/ | di-**PLOI**-ment | despliegue |
| stack | /stæk/ | stak | conjunto de tecnologías |

#tools #roadmap #moc
