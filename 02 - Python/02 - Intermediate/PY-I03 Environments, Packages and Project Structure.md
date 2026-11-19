---
title: "PY-I03 Environments, Packages and Project Structure"
level: intermediate
tags: [python, intermediate, venv, pip, uv, packaging]
prev: "[[PY-I02 Object-Oriented Programming]]"
next: "[[PY-I04 Working with Data Files and APIs]]"
---

# PY-I03 — Virtual Environments, Packages and Project Structure

## 🎯 Learning objectives
- Isolate dependencies per project with virtual environments.
- Install, pin and reproduce packages (`pip`, `requirements.txt`, `uv`).
- Organise a professional project folder.
- Manage configuration and secrets with environment variables.

---

## 1. Why virtual environments?
Project A needs `pandas 1.5`, project B needs `pandas 2.2`. Installing globally breaks one of them. A **virtual environment** is a private folder with its own Python and packages.

### Option A — `venv` + `pip` (built in)
```bash
python -m venv .venv
# Activate
.venv\Scripts\activate         # Windows (PowerShell / CMD)
source .venv/bin/activate      # macOS / Linux
pip install pandas sqlalchemy psycopg[binary]
pip freeze > requirements.txt  # pin exact versions
pip install -r requirements.txt   # reproduce on another machine
deactivate
```

### Option B — `uv` (modern, very fast, recommended in 2026)
```bash
pip install uv              # or the official installer
uv init my_project && cd my_project
uv add pandas polars duckdb      # adds to pyproject.toml + uv.lock
uv add --dev pytest ruff
uv run python main.py           # runs inside the env automatically
uv sync                         # reproduce env from the lock file
```

### Option C — `conda` / `mamba`
Useful when you need non-Python binaries (CUDA, GDAL). `conda create -n ds python=3.12`.

## 2. Semantic versioning
`pandas==2.2.3` → **MAJOR.MINOR.PATCH**
- MAJOR: breaking changes · MINOR: new features · PATCH: bug fixes.
- `>=2.2,<3` allows minor/patch updates but not a breaking major version.

## 3. Professional project layout
```
sales-pipeline/
├── pyproject.toml          # metadata + dependencies + tool config
├── uv.lock                 # exact versions (commit it!)
├── README.md
├── .env.example            # template of required env vars (no secrets)
├── .gitignore              # .venv/, .env, __pycache__/, data/raw/
├── src/
│   └── sales_pipeline/
│       ├── __init__.py
│       ├── config.py
│       ├── extract.py
│       ├── transform.py
│       └── load.py
├── tests/
│   └── test_transform.py
├── notebooks/              # exploration only
└── data/
    ├── raw/                # never edit by hand
    └── processed/
```

### Minimal `pyproject.toml`
```toml
[project]
name = "sales-pipeline"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = ["pandas>=2.2", "sqlalchemy>=2.0", "python-dotenv>=1.0"]

[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

## 4. Configuration and secrets
❌ Never hard-code passwords in code or commit them to Git.

```bash
# .env  (listed in .gitignore)
DB_URL=postgresql+psycopg://user:secret@localhost:5432/sales
```
```python
# config.py
import os
from dotenv import load_dotenv
load_dotenv()
DB_URL = os.environ["DB_URL"]          # fails loudly if missing
LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO")
```

## 5. Command-line entry points
```python
# src/sales_pipeline/cli.py
import argparse
def main():
    parser = argparse.ArgumentParser(description="Run the sales pipeline")
    parser.add_argument("--date", required=True, help="YYYY-MM-DD")
    parser.add_argument("--dry-run", action="store_true")
    args = parser.parse_args()
    print(args.date, args.dry_run)
if __name__ == "__main__":
    main()
```
`python -m sales_pipeline.cli --date 2026-11-18 --dry-run`

---

## 🧪 Practice
1. Create a project with `uv init`, add `pandas` and `pytest`, and run a script with `uv run`.
2. Write a `.gitignore` for a data project.
3. Move a database password from your code to a `.env` file.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| environment | /ɪnˈvaɪ.rən.mənt/ | in-**VÁI**-ron-ment | entorno |
| dependency | /dɪˈpen.dən.si/ | di-**PEN**-den-si | dependencia |
| requirement | /rɪˈkwaɪr.mənt/ | ri-**KUÁIR**-ment | requisito |
| pin (a version) | /pɪn/ | pin | fijar (una versión) |
| lock file | /lɑːk faɪl/ | lak fail | archivo de bloqueo |
| reproduce | /ˌriː.prəˈduːs/ | ri-pro-**DUS** | reproducir |
| layout | /ˈleɪ.aʊt/ | **LÉI**-aut | estructura / disposición |
| hard-code | /ˌhɑːrdˈkoʊd/ | jard-**KOUD** | escribir fijo en el código |
| secret | /ˈsiː.krət/ | **SI**-krit | secreto / credencial |
| breaking change | /ˈbreɪ.kɪŋ tʃeɪndʒ/ | **BRÉI**-king cheinch | cambio incompatible |
| upgrade | /ˈʌp.ɡreɪd/ | **AP**-greid | actualizar |
| entry point | /ˈen.tri pɔɪnt/ | **EN**-tri point | punto de entrada |

### 💬 Useful sentences
- *"**Activate** the virtual environment **before** installing packages."*
- *"Never **commit** secrets to the repository."* — Nunca subas credenciales al repositorio.

---
⬅️ [[PY-I02 Object-Oriented Programming]] · ➡️ [[PY-I04 Working with Data Files and APIs]] · 🏠 [[00 - Python Roadmap]]

#python #intermediate
