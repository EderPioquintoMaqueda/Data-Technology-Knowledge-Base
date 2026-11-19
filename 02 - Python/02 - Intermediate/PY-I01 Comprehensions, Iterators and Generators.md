---
title: "PY-I01 Comprehensions, Iterators and Generators"
level: intermediate
tags: [python, intermediate, comprehensions, generators]
prev: "[[PY-B06 Files, Modules and Error Handling]]"
next: "[[PY-I02 Object-Oriented Programming]]"
---

# PY-I01 — Comprehensions, Iterators and Generators

## 🎯 Learning objectives
- Replace simple loops with list/dict/set comprehensions.
- Understand the iterator protocol (`iter`, `next`).
- Write generators with `yield` to process big files without loading them into memory.
- Use `itertools` for common patterns.

---

## 1. Comprehensions
```python
# Loop version
squares = []
for n in range(10):
    if n % 2 == 0:
        squares.append(n ** 2)

# Comprehension: [expression for item in iterable if condition]
squares = [n ** 2 for n in range(10) if n % 2 == 0]

# Conditional expression inside
labels = ["high" if s > 80 else "low" for s in scores]

# Dict and set comprehensions
lengths = {w: len(w) for w in ["sql", "python"]}
domains = {e.split("@")[1] for e in emails}

# Nested (flatten a matrix)
flat = [x for row in matrix for x in row]
```
💡 Rule: if a comprehension needs more than one `if` or two `for`, write a normal loop — readability first.

## 2. Iterables and iterators
- An **iterable** is anything you can loop over (`list`, `str`, `dict`, file).
- An **iterator** produces items one at a time with `next()` and remembers its position.

```python
it = iter([10, 20, 30])
next(it)   # 10
next(it)   # 20
next(it)   # 30
next(it)   # StopIteration
```
A `for` loop simply calls `iter()` and then `next()` until `StopIteration`.

## 3. Generators — lazy sequences
A function with `yield` returns a **generator**: values are produced *on demand*, so memory use stays constant.

```python
def read_large_csv(path):
    with open(path, encoding="utf-8") as f:
        next(f)                       # skip header
        for line in f:
            yield line.rstrip().split(",")

for row in read_large_csv("transactions_10GB.csv"):
    ...   # only ONE row in memory at a time
```

### Generator expressions
```python
total = sum(x ** 2 for x in range(10_000_000))   # no list created
```
| | List comprehension `[...]` | Generator expression `(...)` |
|---|---|---|
| Memory | stores everything | one item at a time |
| Reusable | yes | **no** (consumed once) |
| Use when | you need indexing / several passes | you stream / aggregate once |

### Pipelines of generators
```python
lines   = read_large_csv("sales.csv")
valid   = (r for r in lines if r[2] != "")
amounts = (float(r[2]) for r in valid)
print(sum(amounts))
```

## 4. `itertools` essentials
```python
from itertools import islice, chain, groupby, product, combinations, batched  # batched: 3.12+

list(islice(read_large_csv("big.csv"), 5))   # first 5 rows (like head)
list(chain([1, 2], [3, 4]))                   # [1, 2, 3, 4]
list(combinations("ABC", 2))                  # [('A','B'), ('A','C'), ('B','C')]
list(product([0, 1], repeat=2))               # all combinations of 2 bits
for batch in batched(range(10), 3):           # (0,1,2) (3,4,5) ...
    insert_into_db(batch)

data = sorted(sales, key=lambda r: r["region"])
for region, rows in groupby(data, key=lambda r: r["region"]):
    print(region, sum(r["amount"] for r in rows))
```

## 5. Useful built-ins
`any()`, `all()`, `zip()`, `enumerate()`, `sorted()`, `reversed()`, `map()`, `filter()`, `min/max(key=...)`.

```python
all(v > 0 for v in prices)          # are all prices positive?
any(r["status"] == "error" for r in logs)
max(employees, key=lambda e: e["salary"])
```

---

## 🧪 Practice
1. Convert `{"a": 1, "b": 2}` into `{1: "a", 2: "b"}` with a comprehension.
2. Write a generator `chunks(lst, n)` that yields lists of size `n`.
3. Write a generator pipeline that reads a log file and counts lines containing `"ERROR"`.
4. Measure memory: `sys.getsizeof([x for x in range(10**6)])` vs `sys.getsizeof(x for x in range(10**6))`.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| comprehension | /ˌkɑːm.prɪˈhen.ʃən/ | kam-pri-**JEN**-shon | comprensión (de listas) |
| iterable | /ˈɪt.ɚ.ə.bəl/ | **I**-te-ra-bol | iterable |
| iterator | /ˈɪt.ə.reɪ.t̬ɚ/ | **I**-te-rei-ter | iterador |
| generator | /ˈdʒen.ə.reɪ.t̬ɚ/ | **YE**-ne-rei-ter | generador |
| yield | /jiːld/ | yild | producir / ceder |
| lazy | /ˈleɪ.zi/ | **LÉI**-zi | perezoso (evaluación diferida) |
| on demand | /ɑːn dɪˈmænd/ | an di-**MAND** | bajo demanda |
| stream | /striːm/ | strim | flujo / transmitir |
| flatten | /ˈflæt.ən/ | **FLA**-ten | aplanar |
| readability | /ˌriː.dəˈbɪl.ə.t̬i/ | ri-da-**BI**-li-ti | legibilidad |
| consume | /kənˈsuːm/ | kon-**SUM** | consumir |
| batch | /bætʃ/ | batch | lote |

### 💬 Useful sentences
- *"Generators **evaluate lazily**, so they **scale to** files that don't fit in memory."*
- *"This comprehension **is equivalent to** the loop above."*

---
⬅️ [[PY-B06 Files, Modules and Error Handling]] · ➡️ [[PY-I02 Object-Oriented Programming]] · 🏠 [[00 - Python Roadmap]]

#python #intermediate
