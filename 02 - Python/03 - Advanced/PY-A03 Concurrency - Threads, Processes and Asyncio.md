---
title: "PY-A03 Concurrency - Threads, Processes and Asyncio"
level: advanced
tags: [python, advanced, concurrency, asyncio, multiprocessing, gil]
prev: "[[PY-A02 Dataclasses, Pydantic and Data Validation]]"
next: "[[PY-A04 Performance, Profiling and Memory]]"
---

# PY-A03 — Concurrency: Threads, Processes and Asyncio

## 🎯 Learning objectives
- Tell **I/O-bound** from **CPU-bound** work and pick the right tool.
- Understand the GIL (and free-threaded Python 3.13+).
- Use `concurrent.futures` (threads and processes).
- Write `async/await` code with `asyncio` and `httpx`/`aiohttp`.

---

## 1. Concurrency vs parallelism
- **Concurrency:** dealing with many tasks at once (switching while waiting). 🧑‍🍳 one cook, many pots.
- **Parallelism:** doing many tasks at the *same instant* on several CPU cores. 🧑‍🍳🧑‍🍳🧑‍🍳 many cooks.

| Workload | Bottleneck | Example | Best tool |
|---|---|---|---|
| **I/O-bound** | waiting for network/disk/DB | 500 API calls, downloading files | `threading` / `asyncio` |
| **CPU-bound** | calculations | parsing, feature engineering in pure Python, image processing | `multiprocessing` / vectorisation (NumPy, Polars) |

## 2. The GIL (Global Interpreter Lock)
In CPython, only **one thread executes Python bytecode at a time**. Therefore:
- Threads speed up I/O-bound work (the GIL is released while waiting).
- Threads do **not** speed up pure-Python CPU work → use processes.
- NumPy, pandas, Polars, DuckDB release the GIL in their C/Rust code, so they parallelise internally.
- Python 3.13+ offers an experimental **free-threaded** build (`python3.13t`) without the GIL; 3.14 makes it officially supported but still optional.

## 3. `concurrent.futures` — the simplest API
### Threads for I/O
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

urls = [f"https://api.example.com/items/{i}" for i in range(200)]

def fetch(url: str) -> dict:
    r = requests.get(url, timeout=10)
    r.raise_for_status()
    return r.json()

results, errors = [], []
with ThreadPoolExecutor(max_workers=20) as pool:
    futures = {pool.submit(fetch, u): u for u in urls}
    for fut in as_completed(futures):
        try:
            results.append(fut.result())
        except Exception as exc:
            errors.append((futures[fut], exc))
```

### Processes for CPU
```python
from concurrent.futures import ProcessPoolExecutor

def heavy(n: int) -> int:
    return sum(i * i for i in range(n))

if __name__ == "__main__":                     # REQUIRED on Windows
    with ProcessPoolExecutor() as pool:        # default: one worker per core
        totals = list(pool.map(heavy, [10_000_000] * 8, chunksize=1))
```
⚠️ Processes don't share memory: arguments and results are **pickled** (serialised). Send small inputs (e.g., file paths), not giant DataFrames.

## 4. asyncio — thousands of concurrent I/O tasks in one thread
```python
import asyncio, httpx

async def fetch(client: httpx.AsyncClient, url: str, sem: asyncio.Semaphore) -> dict:
    async with sem:                            # limit concurrency (be polite to APIs)
        r = await client.get(url, timeout=10)
        r.raise_for_status()
        return r.json()

async def main(urls: list[str]) -> list:
    sem = asyncio.Semaphore(50)
    async with httpx.AsyncClient() as client:
        tasks = [fetch(client, u, sem) for u in urls]
        return await asyncio.gather(*tasks, return_exceptions=True)

results = asyncio.run(main(urls))
```
Key ideas:
- `async def` defines a **coroutine**; `await` pauses it until the result is ready, letting others run.
- Never call blocking code (`requests`, `time.sleep`) inside async code → use `httpx`, `asyncio.sleep`, `asyncpg`, or `await asyncio.to_thread(blocking_func)`.
- `asyncio.TaskGroup` (3.11+) is the modern structured alternative to `gather`.

```python
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(fetch(client, u1, sem))
    t2 = tg.create_task(fetch(client, u2, sem))
```

## 5. Shared state and race conditions
```python
import threading
lock = threading.Lock()
counter = 0
def safe_increment():
    global counter
    with lock:            # only one thread at a time in this block
        counter += 1
```
Prefer designs **without** shared state: each worker returns a result; the main thread combines them. Use `queue.Queue` for producer/consumer pipelines.

## 6. Decision tree
```
Is the code waiting (network, disk, DB)?
├── Yes → few tasks (<100)? ThreadPoolExecutor : asyncio
└── No (CPU) → can it be vectorised? NumPy/Polars/DuckDB : ProcessPoolExecutor
                   └── still too slow / too big? → Dask, Ray, Spark → [[10 - Apache Spark Fundamentals]]
```

---

## 🧪 Practice
1. Download 100 URLs sequentially and with 20 threads; compare the time.
2. Rewrite the same with `asyncio` + `httpx` and a semaphore of 10.
3. Compute prime counts for 8 large ranges with `ProcessPoolExecutor`; compare with a single process.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| concurrency | /kənˈkɝː.ən.si/ | kon-**KE**-ren-si | concurrencia |
| parallelism | /ˈper.ə.lel.ɪ.zəm/ | **PE**-ra-le-li-zem | paralelismo |
| thread | /θred/ | zred (z = th inglesa) | hilo |
| process | /ˈprɑː.ses/ | **PRA**-ses | proceso |
| bound (I/O-bound) | /baʊnd/ | baund | limitado por (E/S) |
| bottleneck | /ˈbɑː.t̬əl.nek/ | **BA**-tol-nek | cuello de botella |
| lock | /lɑːk/ | lak | candado / bloqueo |
| race condition | /reɪs kənˈdɪʃ.ən/ | reis kon-**DI**-shon | condición de carrera |
| coroutine | /ˌkoʊ.ruːˈtiːn/ | kou-ru-**TIN** | corrutina |
| await | /əˈweɪt/ | a-**UÉIT** | esperar (a) |
| blocking | /ˈblɑː.kɪŋ/ | **BLA**-king | bloqueante |
| worker | /ˈwɝː.kɚ/ | **UER**-ker | trabajador (proceso/hilo) |
| throughput | /ˈθruː.pʊt/ | **ZRU**-put | rendimiento (volumen procesado) |
| pickle / serialise | /ˈpɪk.əl/ | **PI**-kol | serializar |

### 💬 Useful sentences
- *"This job is **I/O-bound**, so threads **will help**; for CPU-bound work we **need** processes."*
- *"We **throttled** the requests to 50 at a time **to avoid** hitting the rate limit."*

---
⬅️ [[PY-A02 Dataclasses, Pydantic and Data Validation]] · ➡️ [[PY-A04 Performance, Profiling and Memory]] · 🏠 [[00 - Python Roadmap]]

#python #advanced
