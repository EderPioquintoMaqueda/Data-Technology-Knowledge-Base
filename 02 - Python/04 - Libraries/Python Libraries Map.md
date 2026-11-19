---
title: Python Libraries Map
type: moc
tags: [python, libraries, ecosystem, moc]
---

# 🗺️ Python Libraries Map — The Data & AI Ecosystem

> Python is the **glue**; the heavy work runs in C, C++, Rust, Fortran and CUDA. Choosing the right library is about **data size, speed, memory and purpose** — not taste.

## 1. The layers
```
┌──────────────────────────────────────────────────────────────┐
│ L5  Apps & delivery   FastAPI · Streamlit · Dash · Airflow    │
│ L4  Deep learning     PyTorch · TensorFlow/Keras · JAX · HF   │
│ L3  Machine learning  scikit-learn · XGBoost · LightGBM       │
│ L2  Science & stats   SciPy · statsmodels                      │
│ L1  Tabular data      pandas · Polars · DuckDB · PyArrow       │
│ L0  Numeric engine    NumPy (ndarray)                          │
└──────────────────────────────────────────────────────────────┘
      Visualisation across all layers: matplotlib · seaborn · plotly
      Databases: sqlite3 · psycopg · SQLAlchemy · DuckDB
```

## 2. Decision matrix
| If you need to… | Use | Why |
|---|---|---|
| Fast maths on arrays/matrices | **NumPy** | contiguous memory + vectorised C loops |
| Explore/clean a dataset that fits in RAM (< ~2 GB) | **pandas** | richest API, huge community |
| Process 5–100 GB on a laptop, or need speed | **Polars** | Rust, multi-threaded, lazy query optimiser |
| Run SQL on CSV/Parquet/DataFrames, no server | **DuckDB** | in-process OLAP database |
| Talk to PostgreSQL/MySQL/SQL Server | **SQLAlchemy + psycopg** | pooling, ORM, safe parameters |
| Statistical tests, p-values, regressions with summaries | **statsmodels / SciPy** | inference, not only prediction |
| Classic ML (trees, regression, clustering) | **scikit-learn** | consistent `fit/predict` API |
| Gradient boosting on tabular data | **XGBoost / LightGBM / CatBoost** | best accuracy for tables |
| Neural networks, LLMs | **PyTorch** (+ Hugging Face) / TensorFlow | GPU, autograd |
| Static charts for reports | **matplotlib + seaborn** | full control |
| Interactive charts / dashboards | **plotly / Streamlit** | zoom, hover, web |
| Validate data | **Pydantic / Pandera** | contracts at boundaries |
| Scrape websites | **requests + BeautifulSoup / Playwright** | static vs JS pages |

## 3. Learning paths (beginner → advanced)
### NumPy
[[numpy_array_creation]] → [[numpy_indexing_slicing_views]] → [[numpy_boolean_masking]] → [[numpy_broadcasting_engine]] → [[numpy_linear_algebra_einsum]] → [[numpy_memory_architecture]]

### pandas
[[pandas_series_dataframe_internals]] → [[pandas_io_data_loading]] → [[pandas_selection_filtering]] → [[pandas_data_cleaning_missing]] → [[pandas_transformations_apply]] → [[pandas_groupby_aggregation]] → [[pandas_merging_joining]] → [[pandas_timeseries]] → [[Pandas_High_Performance]]

### Polars & DuckDB
[[Polars - Beginner to Advanced]] → [[Polars_High_Performance]] · [[DuckDB - SQL Inside Python]]

### Visualisation
[[matplotlib_architecture]] → [[matplotlib_customization]] → [[seaborn_statistical_distribution]] → [[seaborn_categorical_relationships]] → [[seaborn_multivariate_grids]] → [[Visualizacion_Interactiva]]

### Statistics
[[SciPy_Statsmodels_Professional]]

### Machine learning
[[sklearn_api_design]] → [[sklearn_preprocessing]] → [[sklearn_linear_logistic_regression]] → [[sklearn_tree_ensemble_models]] → [[sklearn_model_evaluation]] → [[sklearn_feature_selection]] → [[sklearn_clustering_unsupervised]] → [[sklearn_pipelines_deployment]]

### Deep learning
[[tensorflow_tensors_autograd]] → [[tensorflow_sequential_api]] → [[tensorflow_functional_api]] → [[tensorflow_training_callbacks]] → [[tensorflow_convolutional_neural_networks]] → [[tensorflow_recurrent_neural_networks]] → [[tensorflow_transformers_attention]] → [[tensorflow_saving_deployment]]

## 4. A professional pipeline uses several libraries
1. **Ingest** → Polars `scan_parquet` / DuckDB / `requests`
2. **Validate** → Pydantic (records) / Pandera (frames)
3. **Transform** → Polars or pandas (or SQL in DuckDB/PostgreSQL)
4. **Store** → PostgreSQL via SQLAlchemy, or Parquet files
5. **Analyse** → statsmodels / SciPy
6. **Model** → scikit-learn / XGBoost / PyTorch
7. **Serve** → FastAPI → [[FastAPI_ML_Deployment]]

## 5. Install a complete data stack
```bash
uv add numpy pandas polars pyarrow duckdb sqlalchemy "psycopg[binary]" \
       scipy statsmodels scikit-learn matplotlib seaborn plotly \
       pydantic pandera requests jupyterlab
```

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| library | /ˈlaɪ.brer.i/ | **LÁI**-bre-ri | librería / biblioteca |
| framework | /ˈfreɪm.wɝːk/ | **FRÉIM**-uerk | marco de trabajo |
| ecosystem | /ˈiː.koʊˌsɪs.təm/ | **I**-kou-sis-tem | ecosistema |
| array | /əˈreɪ/ | a-**RÉI** | arreglo |
| tabular | /ˈtæb.jə.lɚ/ | **TA**-biu-ler | tabular |
| glue | /ɡluː/ | glu | pegamento |
| heavy lifting | /ˈhev.i ˈlɪf.tɪŋ/ | **JE**-vi **LIF**-ting | trabajo pesado |
| suitable | /ˈsuː.t̬ə.bəl/ | **SU**-ta-bol | adecuado |

🏠 [[00 - Python Roadmap]]

#python #libraries #moc
