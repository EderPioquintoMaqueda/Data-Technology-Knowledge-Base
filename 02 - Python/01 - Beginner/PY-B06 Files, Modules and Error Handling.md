---
title: "PY-B06 Files, Modules and Error Handling"
level: beginner
tags: [python, beginner, files, modules, exceptions]
prev: "[[PY-B05 Functions]]"
next: "[[PY-I01 Comprehensions, Iterators and Generators]]"
---

# PY-B06 — Files, Modules and Error Handling

## 🎯 Learning objectives
- Read and write text files safely with `with`.
- Organise code into modules and import them.
- Handle errors with `try / except / else / finally` and raise your own.

---

## 1. Reading and writing files
```python
# Write
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("First line\n")
    f.writelines(["Second\n", "Third\n"])

# Append
with open("notes.txt", "a", encoding="utf-8") as f:
    f.write("Appended line\n")

# Read everything / line by line (memory-friendly)
with open("notes.txt", encoding="utf-8") as f:
    for line in f:
        print(line.rstrip())
```
| Mode | Meaning |
|---|---|
| `"r"` | read (default) |
| `"w"` | write — **overwrites** the file |
| `"a"` | append |
| `"x"` | create, fail if exists |
| `"b"` | binary (`"rb"`, `"wb"`) |

✅ `with` closes the file automatically, even if an error happens (it is a **context manager** — see [[PY-A01 Decorators, Closures and Context Managers]]).
✅ Always pass `encoding="utf-8"` (Windows defaults can break accents: á, ñ).

### Paths with `pathlib`
```python
from pathlib import Path
data_dir = Path("data")
data_dir.mkdir(exist_ok=True)
file = data_dir / "sales_2026.csv"
file.exists(), file.suffix, file.stem      # False, '.csv', 'sales_2026'
for csv in data_dir.glob("*.csv"):
    print(csv.name)
```

## 2. Modules and imports
A **module** is a `.py` file; a **package** is a folder of modules (with `__init__.py`).

```python
import math                       # whole module
from statistics import mean, median
import pandas as pd               # alias convention
from my_project.cleaning import normalize_email   # your own code
```

### The `if __name__ == "__main__":` guard
```python
# cleaning.py
def normalize_email(e): return e.strip().lower()

if __name__ == "__main__":        # runs only when executed directly, not when imported
    print(normalize_email("  A@B.COM "))
```

### Useful standard library modules
| Module | Use |
|---|---|
| `math`, `statistics`, `random` | numbers |
| `datetime` | dates and times |
| `pathlib`, `os`, `shutil` | files and folders |
| `csv`, `json` | data formats |
| `collections`, `itertools` | advanced collections |
| `logging` | professional messages instead of `print` |

## 3. Errors and exceptions
```python
try:
    value = int(input("Number: "))
    result = 100 / value
except ValueError:
    print("That is not a number.")
except ZeroDivisionError:
    print("Cannot divide by zero.")
else:
    print(f"Result: {result}")      # runs only if no exception
finally:
    print("Done.")                   # always runs
```

### Common exceptions
| Exception | Typical cause |
|---|---|
| `SyntaxError` | invalid code (missing `:` or `)`) |
| `NameError` | variable not defined |
| `TypeError` | wrong type (`"3" + 3`) |
| `ValueError` | right type, wrong value (`int("abc")`) |
| `KeyError` | missing dict key |
| `IndexError` | list index out of range |
| `FileNotFoundError` | wrong path |
| `ZeroDivisionError` | division by zero |

### Raising your own errors
```python
def set_discount(percent: float) -> float:
    if not 0 <= percent <= 100:
        raise ValueError(f"percent must be 0–100, got {percent}")
    return percent / 100
```

### Reading a traceback (read it from the bottom up!)
```
Traceback (most recent call last):
  File "main.py", line 3, in <module>
    total = price + tax
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```
The **last line** tells you *what* happened; the lines above tell you *where*.

🚫 Never write a bare `except:` — it hides bugs. Catch specific exceptions.

---

## 🧪 Practice
1. Write a script that reads `numbers.txt` (one number per line), skips invalid lines and prints the sum.
2. Create a module `utils.py` with two functions and import it from `main.py`.
3. List every `.csv` file in a folder with its size in KB (`file.stat().st_size`).

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| file | /faɪl/ | fail | archivo |
| folder / directory | /ˈfoʊl.dɚ/ /dəˈrek.tɚ.i/ | **FOUL**-der / di-**REK**-to-ri | carpeta / directorio |
| path | /pæθ/ | paz | ruta |
| overwrite | /ˌoʊ.vɚˈraɪt/ | ou-ver-**RÁIT** | sobrescribir |
| encoding | /ɪnˈkoʊ.dɪŋ/ | in-**KOU**-ding | codificación |
| module | /ˈmɑː.dʒuːl/ | **MA**-yul | módulo |
| package | /ˈpæk.ɪdʒ/ | **PA**-kich | paquete |
| import | /ɪmˈpɔːrt/ | im-**PORT** | importar |
| exception | /ɪkˈsep.ʃən/ | ik-**SEP**-shon | excepción |
| raise | /reɪz/ | reiz | lanzar (una excepción) |
| catch | /kætʃ/ | katch | capturar |
| traceback | /ˈtreɪs.bæk/ | **TRÉIS**-bak | rastreo del error |
| bug | /bʌɡ/ | bag | error / fallo |
| out of range | /aʊt əv reɪndʒ/ | aut ov reinch | fuera de rango |

### 💬 Useful sentences
- *"The script **throws** a `FileNotFoundError` because the path is wrong."*
- *"**Read the traceback from the bottom up.**"* — Lee el rastreo de abajo hacia arriba.

---
⬅️ [[PY-B05 Functions]] · ➡️ [[PY-I01 Comprehensions, Iterators and Generators]] · 🏠 [[00 - Python Roadmap]]

#python #beginner
