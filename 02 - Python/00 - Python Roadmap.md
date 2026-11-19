---
title: Python Roadmap
type: moc
tags: [python, roadmap, moc]
---

# 🐍 Python Roadmap — From Beginner to Advanced

> **How to use this roadmap:** Study the notes in order. Each note has four parts: **Concept → Code → Practice → Vocabulary**. The vocabulary table at the end of every note teaches you the English words you need (with pronunciation and Spanish translation). Read the code out loud — it is also English practice.

## 🟢 Level 1 — Beginner (Weeks 1–4)
*Goal: write small scripts that read data, make decisions and produce results.*

| # | Note | You will learn |
|---|------|----------------|
| 1 | [[PY-B01 Setup, Variables and Data Types]] | Install Python, run code, `int`, `float`, `str`, `bool` |
| 2 | [[PY-B02 Strings and Input-Output]] | f-strings, string methods, `print`, `input` |
| 3 | [[PY-B03 Control Flow - Conditionals and Loops]] | `if/elif/else`, `for`, `while`, `break`, `continue` |
| 4 | [[PY-B04 Data Structures - Lists, Tuples, Dicts and Sets]] | The four core collections and when to use each |
| 5 | [[PY-B05 Functions]] | Parameters, return values, scope, `*args`, `**kwargs` |
| 6 | [[PY-B06 Files, Modules and Error Handling]] | `open`, `with`, `import`, `try/except` |

✅ **Checkpoint project:** a script that reads a CSV of expenses, groups them by category and prints a report.

## 🟡 Level 2 — Intermediate (Weeks 5–10)
*Goal: write clean, reusable, tested code organised as a project.*

| # | Note | You will learn |
|---|------|----------------|
| 7 | [[PY-I01 Comprehensions, Iterators and Generators]] | Pythonic loops, lazy evaluation, `yield` |
| 8 | [[PY-I02 Object-Oriented Programming]] | Classes, inheritance, dunder methods |
| 9 | [[PY-I03 Environments, Packages and Project Structure]] | `venv`, `pip`, `uv`, `pyproject.toml` |
| 10 | [[PY-I04 Working with Data Files and APIs]] | CSV, JSON, `pathlib`, `requests` |
| 11 | [[PY-I05 Testing, Type Hints and Code Quality]] | `pytest`, `ruff`, `mypy`, logging |

✅ **Checkpoint project:** a CLI tool that downloads data from a public API, validates it and saves it to CSV — with tests.

## 🔴 Level 3 — Advanced (Weeks 11–18)
*Goal: write production-grade data code that is fast, safe and connected to databases.*

| # | Note | You will learn |
|---|------|----------------|
| 12 | [[PY-A01 Decorators, Closures and Context Managers]] | Higher-order functions, `functools`, `contextlib` |
| 13 | [[PY-A02 Dataclasses, Pydantic and Data Validation]] | Typed models, validation at the boundaries |
| 14 | [[PY-A03 Concurrency - Threads, Processes and Asyncio]] | GIL, `concurrent.futures`, `async/await` |
| 15 | [[PY-A04 Performance, Profiling and Memory]] | `cProfile`, vectorisation, generators, `__slots__` |
| 16 | [[PY-A05 Python and Databases]] | `sqlite3`, `psycopg`, SQLAlchemy, DuckDB, pandas ↔ SQL |

✅ **Checkpoint project:** an ETL pipeline: API → validation (Pydantic) → PostgreSQL (SQLAlchemy) → report (pandas), running in Docker.

## 📚 Level 4 — Libraries (in parallel with Levels 2–3)
Start here: [[Python Libraries Map]]

| Library | Beginner → Advanced path |
|---------|--------------------------|
| **NumPy** | [[numpy_array_creation]] → [[numpy_indexing_slicing_views]] → [[numpy_boolean_masking]] → [[numpy_broadcasting_engine]] → [[numpy_linear_algebra_einsum]] → [[numpy_memory_architecture]] |
| **pandas** | [[pandas_series_dataframe_internals]] → [[pandas_io_data_loading]] → [[pandas_selection_filtering]] → [[pandas_data_cleaning_missing]] → [[pandas_transformations_apply]] → [[pandas_groupby_aggregation]] → [[pandas_merging_joining]] → [[pandas_timeseries]] → [[Pandas_High_Performance]] |
| **Polars** | [[Polars - Beginner to Advanced]] → [[Polars_High_Performance]] |
| **DuckDB** | [[DuckDB - SQL Inside Python]] |
| **Visualization** | [[matplotlib_architecture]] → [[matplotlib_customization]] → [[seaborn_statistical_distribution]] → [[seaborn_categorical_relationships]] → [[seaborn_multivariate_grids]] → [[Visualizacion_Interactiva]] |
| **SciPy / statsmodels** | [[SciPy_Statsmodels_Professional]] |
| **scikit-learn** | [[sklearn_api_design]] → [[sklearn_preprocessing]] → [[sklearn_linear_logistic_regression]] → [[sklearn_tree_ensemble_models]] → [[sklearn_model_evaluation]] → [[sklearn_feature_selection]] → [[sklearn_clustering_unsupervised]] → [[sklearn_pipelines_deployment]] |
| **TensorFlow** | [[tensorflow_tensors_autograd]] → [[tensorflow_sequential_api]] → [[tensorflow_functional_api]] → [[tensorflow_training_callbacks]] → [[tensorflow_convolutional_neural_networks]] → [[tensorflow_recurrent_neural_networks]] → [[tensorflow_transformers_attention]] → [[tensorflow_saving_deployment]] |
| **Web scraping** | [[Web_Scraping_Profesional]] |

## 🔗 Connections
- SQL track: [[00 - SQL Roadmap]]
- Databases: [[00 - Database Architecture Roadmap]]
- Tools: [[00 - Tools Roadmap]]
- English for programmers: [[Data and Programming]]

## 🗣️ How to read the pronunciation column
See [[How to Read Pronunciation]]. Short version: **CAPITAL letters = stressed syllable**. Example: *variable* → **VE**-ri-a-bol.

#python #roadmap #moc
