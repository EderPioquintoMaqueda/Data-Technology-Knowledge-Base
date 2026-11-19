---
title: "PY-A01 Decorators, Closures and Context Managers"
level: advanced
tags: [python, advanced, decorators, closures, context-managers, functools]
prev: "[[PY-I05 Testing, Type Hints and Code Quality]]"
next: "[[PY-A02 Dataclasses, Pydantic and Data Validation]]"
---

# PY-A01 — Decorators, Closures and Context Managers

## 🎯 Learning objectives
- Understand closures and first-class functions.
- Write decorators (with and without arguments) using `functools.wraps`.
- Use `functools`: `lru_cache`, `partial`, `cache`, `singledispatch`.
- Write context managers with classes and with `@contextmanager`.

---

## 1. Closures
A **closure** is an inner function that "remembers" variables from the enclosing scope.
```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor          # factor is captured
    return multiply

double = make_multiplier(2)
double(21)    # 42
```
Use `nonlocal` to modify a captured variable:
```python
def counter():
    count = 0
    def inc():
        nonlocal count
        count += 1
        return count
    return inc
```

## 2. Decorators
A **decorator** takes a function and returns a new function that adds behaviour (timing, logging, retries, caching, auth).

```python
import functools, time, logging
log = logging.getLogger(__name__)

def timer(func):
    @functools.wraps(func)              # keeps name & docstring of func
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        log.info("%s took %.3fs", func.__name__, time.perf_counter() - start)
        return result
    return wrapper

@timer                                   # same as: load_data = timer(load_data)
def load_data(path): ...
```

### Decorator with arguments (three levels)
```python
def retry(times: int = 3, delay: float = 1.0, exceptions=(Exception,)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as exc:
                    if attempt == times:
                        raise
                    log.warning("Attempt %d failed: %s. Retrying...", attempt, exc)
                    time.sleep(delay * 2 ** (attempt - 1))   # exponential backoff
        return wrapper
    return decorator

@retry(times=5, exceptions=(ConnectionError,))
def fetch_prices(): ...
```

### Stacking decorators
```python
@timer
@retry(times=3)
def job(): ...      # applied bottom-up: timer(retry(job))
```

## 3. `functools` toolbox
```python
from functools import lru_cache, cache, partial, reduce, singledispatch

@lru_cache(maxsize=1024)            # memoisation: same input → cached output
def exchange_rate(currency: str, day: str) -> float: ...

int_base2 = partial(int, base=2)    # pre-fill arguments
int_base2("1010")                   # 10

reduce(lambda a, b: a * b, [1, 2, 3, 4])   # 24

@singledispatch                      # function overloading by type
def to_sql_literal(value): raise TypeError(type(value))
@to_sql_literal.register
def _(value: str): return "'" + value.replace("'", "''") + "'"
@to_sql_literal.register
def _(value: int): return str(value)
```
⚠️ `lru_cache` needs hashable arguments (no lists/dicts) and should only cache **pure** functions.

## 4. Context managers
They guarantee **setup and teardown** (open/close, begin/commit, lock/unlock) even when exceptions occur.

### Class-based
```python
class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self
    def __exit__(self, exc_type, exc, tb):
        self.elapsed = time.perf_counter() - self.start
        return False        # False = do not swallow exceptions

with Timer() as t:
    heavy_work()
print(t.elapsed)
```

### Generator-based with `contextlib`
```python
from contextlib import contextmanager

@contextmanager
def db_transaction(conn):
    cur = conn.cursor()
    try:
        yield cur
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        cur.close()

with db_transaction(conn) as cur:
    cur.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
    cur.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
```
Other helpers: `contextlib.suppress(FileNotFoundError)`, `ExitStack` for a dynamic number of resources.

## 5. Real-world patterns
| Pattern | Tool |
|---|---|
| Measure execution time | `@timer` |
| Retry flaky network/DB calls | `@retry` (or the `tenacity` library) |
| Cache expensive lookups | `@lru_cache` |
| Register plugins / routes | decorators that add funcs to a dict (FastAPI `@app.get`) |
| Transactions, locks, temp dirs | context managers |

---

## 🧪 Practice
1. Write `@log_calls` that logs function name, args and return value.
2. Write `@validate_df(columns=[...])` that raises if a returned DataFrame misses columns.
3. Write a context manager `cd(path)` that temporarily changes the working directory.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| decorator | /ˈdek.ə.reɪ.t̬ɚ/ | **DE**-ko-rei-ter | decorador |
| wrapper | /ˈræp.ɚ/ | **RA**-per | envoltorio / envoltura |
| closure | /ˈkloʊ.ʒɚ/ | **KLOU**-yer | clausura (función cerrada) |
| enclosing | /ɪnˈkloʊ.zɪŋ/ | in-**KLOU**-zing | envolvente |
| capture | /ˈkæp.tʃɚ/ | **KAP**-cher | capturar |
| cache | /kæʃ/ | kash | caché |
| memoisation | /ˌmem.oʊ.aɪˈzeɪ.ʃən/ | me-mo-ai-**ZÉI**-shon | memoización |
| hashable | /ˈhæʃ.ə.bəl/ | **JA**-sha-bol | hasheable |
| setup / teardown | /ˈset.ʌp/ /ˈter.daʊn/ | **SET**-ap / **TER**-daun | preparación / limpieza |
| swallow (an error) | /ˈswɑː.loʊ/ | **SUA**-lou | tragarse (ocultar) un error |
| flaky | /ˈfleɪ.ki/ | **FLÉI**-ki | inestable / intermitente |
| overload | /ˌoʊ.vɚˈloʊd/ | ou-ver-**LOUD** | sobrecarga |

### 💬 Useful sentences
- *"The decorator **wraps** the function and **adds** retry logic **without changing** its code."*
- *"A context manager **guarantees** the connection **is closed** even if an error **occurs**."*

---
⬅️ [[PY-I05 Testing, Type Hints and Code Quality]] · ➡️ [[PY-A02 Dataclasses, Pydantic and Data Validation]] · 🏠 [[00 - Python Roadmap]]

#python #advanced
