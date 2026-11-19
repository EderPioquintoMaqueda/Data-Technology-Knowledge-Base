---
title: "PY-A04 Performance, Profiling and Memory"
level: advanced
tags: [python, advanced, performance, profiling, memory, vectorization]
prev: "[[PY-A03 Concurrency - Threads, Processes and Asyncio]]"
next: "[[PY-A05 Python and Databases]]"
---

# PY-A04 — Performance, Profiling and Memory

## 🎯 Learning objectives
- **Measure before optimising** (`timeit`, `cProfile`, `line_profiler`, `memray`).
- Apply the optimisation ladder: algorithm → data structure → vectorisation → compiled code → parallelism.
- Reduce memory in pandas (dtypes, categoricals, chunking, Parquet).

> "Premature optimization is the root of all evil." — Donald Knuth. First make it **correct**, then **measure**, then make it **fast**.

---

## 1. Measure
```python
import timeit
timeit.timeit("sum(range(1000))", number=10_000)
```
In Jupyter: `%timeit df["a"] * 2` · `%%time` for a whole cell.

### Profile a whole program
```bash
python -m cProfile -o profile.out main.py
python -m pstats profile.out          # then: sort cumtime → stats 15
pip install snakeviz && snakeviz profile.out   # visual flame chart
```
- **`line_profiler`** (`@profile`, `kernprof -l -v script.py`) → time per line.
- **`memray`** / **`tracemalloc`** → where memory is allocated.
- **`py-spy top --pid 1234`** → profile a running process without changing code.

## 2. The optimisation ladder
| Step | Example | Typical speed-up |
|---|---|---|
| 1. Better **algorithm** | O(n²) nested loop → dict lookup O(n) | 10–1000× |
| 2. Right **data structure** | `x in list` → `x in set` | 100× |
| 3. **Built-ins** & comprehensions | `sum()`, `"".join()` instead of `+=` | 2–5× |
| 4. **Vectorisation** | loop over rows → NumPy/pandas/Polars columns | 10–100× |
| 5. **Compiled code** | Numba `@njit`, Cython, Rust (PyO3) | 10–100× |
| 6. **Parallelism** | processes, Polars, DuckDB, Spark | ≈ number of cores |

### Example — the same task, four ways
```python
import numpy as np, pandas as pd
df = pd.DataFrame({"price": np.random.rand(1_000_000) * 100,
                   "qty": np.random.randint(1, 10, 1_000_000)})

# ❌ 1. iterrows — very slow (~30 s)
total = [row.price * row.qty for _, row in df.iterrows()]
# 😐 2. apply row-wise — slow (~5 s)
total = df.apply(lambda r: r.price * r.qty, axis=1)
# ✅ 3. vectorised — fast (~5 ms)
total = df["price"] * df["qty"]
# ✅ 4. conditional logic vectorised
df["band"] = np.select([df.price < 20, df.price < 60], ["low", "mid"], default="high")
```

### Numba for loops that cannot be vectorised
```python
from numba import njit
@njit
def moving_max(a, window):
    out = np.empty(a.size - window + 1)
    for i in range(out.size):
        out[i] = a[i:i + window].max()
    return out
```

## 3. Memory optimisation in pandas
```python
df.info(memory_usage="deep")

# Downcast numbers
df["qty"] = pd.to_numeric(df["qty"], downcast="integer")     # int64 → int8
df["price"] = df["price"].astype("float32")
# Low-cardinality strings → category (often 90% less memory)
df["country"] = df["country"].astype("category")
# Arrow-backed strings (pandas 2.x)
df = pd.read_csv("big.csv", dtype_backend="pyarrow")
# Read only what you need
pd.read_csv("big.csv", usecols=["id", "amount"], dtype={"id": "int32"})
# Process in chunks
for chunk in pd.read_csv("huge.csv", chunksize=500_000):
    aggregate(chunk)
```
💡 For files larger than RAM: **Polars lazy** (`pl.scan_parquet`) or **DuckDB** — see [[Polars - Beginner to Advanced]] and [[DuckDB - SQL Inside Python]].

## 4. Python-level memory tricks
- Generators instead of lists for streams → [[PY-I01 Comprehensions, Iterators and Generators]].
- `__slots__` / `@dataclass(slots=True)` for millions of small objects.
- `sys.getsizeof(obj)` (shallow) vs `pympler.asizeof` (deep).
- `del big_df; gc.collect()` inside long notebooks.

## 5. String building
```python
# ❌ O(n²)
s = ""
for part in parts: s += part
# ✅ O(n)
s = "".join(parts)
```

## 6. I/O is often the real bottleneck
| Format | Read 1 GB of data (approx.) |
|---|---|
| CSV | slow, all strings parsed |
| Parquet (columnar, compressed) | 5–20× faster, reads only needed columns |
| Feather / Arrow IPC | fastest for temporary files |

---

## 🧪 Practice
1. Profile one of your scripts with `cProfile` and find the top 3 functions by cumulative time.
2. Reduce the memory of a 1M-row DataFrame by at least 50% with dtypes.
3. Rewrite an `apply(axis=1)` with `np.where`/`np.select` and measure the speed-up.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| performance | /pɚˈfɔːr.məns/ | per-**FOR**-mans | rendimiento |
| profiling | /ˈproʊ.faɪ.lɪŋ/ | **PROU**-fai-ling | perfilado (medición) |
| measure | /ˈmeʒ.ɚ/ | **ME**-yer | medir |
| speed-up | /ˈspiːd.ʌp/ | **SPID**-ap | aceleración |
| overhead | /ˈoʊ.vɚ.hed/ | **OU**-ver-jed | sobrecarga |
| allocate | /ˈæl.ə.keɪt/ | **A**-lo-keit | asignar (memoria) |
| downcast | /ˈdaʊn.kæst/ | **DÁUN**-kast | reducir el tipo |
| chunk | /tʃʌŋk/ | chank | fragmento / bloque |
| premature | /ˌpriː.məˈtʃʊr/ | pri-ma-**CHUR** | prematuro |
| bottleneck | /ˈbɑː.t̬əl.nek/ | **BA**-tol-nek | cuello de botella |
| approximately | /əˈprɑːk.sə.mət.li/ | a-**PRAK**-si-mit-li | aproximadamente |
| row-wise | /ˈroʊ.waɪz/ | **ROU**-uaiz | fila por fila |

### 💬 Useful sentences
- *"**Let's profile it first** — we don't know where the bottleneck is yet."*
- *"Vectorising the loop **cut** the runtime **from** 30 seconds **to** 5 milliseconds."*

---
⬅️ [[PY-A03 Concurrency - Threads, Processes and Asyncio]] · ➡️ [[PY-A05 Python and Databases]] · 🏠 [[00 - Python Roadmap]]

#python #advanced
