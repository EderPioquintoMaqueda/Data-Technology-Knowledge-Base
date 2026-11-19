---
title: "PY-I05 Testing, Type Hints and Code Quality"
level: intermediate
tags: [python, intermediate, pytest, typing, ruff, logging]
prev: "[[PY-I04 Working with Data Files and APIs]]"
next: "[[PY-A01 Decorators, Closures and Context Managers]]"
---

# PY-I05 — Testing, Type Hints and Code Quality

## 🎯 Learning objectives
- Write unit tests with `pytest` (fixtures, parametrize, exceptions).
- Add type hints and check them with `mypy`/`pyright`.
- Lint and format automatically with `ruff`.
- Replace `print` with `logging`.

---

## 1. Testing with pytest
```python
# src/shop/pricing.py
def apply_discount(price: float, percent: float) -> float:
    if not 0 <= percent <= 100:
        raise ValueError("percent must be between 0 and 100")
    return round(price * (1 - percent / 100), 2)
```
```python
# tests/test_pricing.py
import pytest
from shop.pricing import apply_discount

def test_basic_discount():
    assert apply_discount(100, 10) == 90.0

@pytest.mark.parametrize("price, percent, expected", [
    (100, 0, 100.0),
    (100, 100, 0.0),
    (19.99, 15, 16.99),
])
def test_many_cases(price, percent, expected):
    assert apply_discount(price, percent) == expected

def test_invalid_percent():
    with pytest.raises(ValueError, match="between 0 and 100"):
        apply_discount(100, 150)
```
Run: `pytest -v` · with coverage: `pytest --cov=src`.

### Fixtures — reusable setup
```python
import pandas as pd, pytest

@pytest.fixture
def sample_df():
    return pd.DataFrame({"region": ["N", "S", "N"], "amount": [10, 5, 7]})

def test_total_by_region(sample_df):
    result = sample_df.groupby("region")["amount"].sum()
    assert result["N"] == 17
```
💡 Built-in fixture `tmp_path` gives you a temporary folder for file tests.

### What to test in data code
- Pure transformation functions (input df → expected output df) with `pd.testing.assert_frame_equal`.
- Edge cases: empty input, nulls, duplicates, wrong types.
- Data contracts: column names, dtypes, no negative prices — see also Pandera / Great Expectations.

## 2. Type hints
```python
from typing import Iterable, Optional, Literal, TypedDict

def mean(values: Iterable[float]) -> float: ...
def find_user(user_id: int) -> Optional[dict]: ...        # or: dict | None
Mode = Literal["append", "replace"]
def save(df: "pd.DataFrame", mode: Mode = "append") -> None: ...

class Order(TypedDict):
    id: int
    amount: float
```
Type hints do **not** change runtime behaviour; tools check them: `mypy src/` or `pyright`.

## 3. Linting and formatting with ruff
```bash
ruff check .          # find problems (unused imports, bugs, style)
ruff check . --fix    # fix automatically
ruff format .         # format code (Black-compatible)
```
### pre-commit — run checks before every commit
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```
`pre-commit install`

## 4. Logging instead of print
```python
import logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
)
log = logging.getLogger(__name__)

log.debug("Detailed info for debugging")
log.info("Loaded %d rows", 1500)
log.warning("Missing values in column %s", "price")
log.error("Could not connect to database")
log.exception("Unexpected error")   # inside except: includes traceback
```
| Level | Use |
|---|---|
| DEBUG | internal details |
| INFO | normal progress |
| WARNING | something odd, still running |
| ERROR | an operation failed |
| CRITICAL | the program cannot continue |

## 5. Clean code checklist
- [ ] Descriptive names (`customer_count`, not `cc`).
- [ ] Functions do one thing.
- [ ] No magic numbers → named constants.
- [ ] No duplicated code (DRY: *Don't Repeat Yourself*).
- [ ] Early returns instead of deep nesting.
- [ ] Docstrings on public functions.
- [ ] Tests pass and linter is clean before pushing.

---

## 🧪 Practice
1. Write 5 parametrized tests for a `clean_phone_number()` function.
2. Add type hints to an old script and run `mypy` until it is clean.
3. Replace all `print` calls in a pipeline with `logging`.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| test | /test/ | test | prueba |
| unit test | /ˈjuː.nɪt test/ | **YU**-nit test | prueba unitaria |
| assert | /əˈsɝːt/ | a-**SERT** | afirmar / verificar |
| fixture | /ˈfɪks.tʃɚ/ | **FIKS**-cher | dato/recurso de prueba preparado |
| edge case | /edʒ keɪs/ | ech keis | caso límite |
| coverage | /ˈkʌv.ɚ.ɪdʒ/ | **KA**-ve-rich | cobertura |
| type hint | /taɪp hɪnt/ | taip jint | anotación de tipo |
| linter | /ˈlɪn.t̬ɚ/ | **LIN**-ter | analizador de estilo |
| warning | /ˈwɔːr.nɪŋ/ | **UOR**-ning | advertencia |
| maintainable | /meɪnˈteɪ.nə.bəl/ | mein-**TÉI**-na-bol | mantenible |
| refactor | /ˌriːˈfæk.tɚ/ | ri-**FAK**-ter | refactorizar |
| expected | /ɪkˈspek.tɪd/ | ik-**SPEK**-tid | esperado |
| pass / fail | /pæs/ /feɪl/ | pas / feil | pasar / fallar |

### 💬 Useful sentences
- *"All tests **pass** on my machine, but one **fails** in CI."*
- *"Let's **add a test** for this **edge case** before we **refactor**."*

---
⬅️ [[PY-I04 Working with Data Files and APIs]] · ➡️ [[PY-A01 Decorators, Closures and Context Managers]] · 🏠 [[00 - Python Roadmap]]

#python #intermediate
