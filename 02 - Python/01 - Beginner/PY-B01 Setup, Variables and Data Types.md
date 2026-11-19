---
title: "PY-B01 Setup, Variables and Data Types"
level: beginner
tags: [python, beginner, variables, data-types]
next: "[[PY-B02 Strings and Input-Output]]"
---

# PY-B01 — Setup, Variables and Data Types

## 🎯 Learning objectives
By the end of this note you will be able to:
- Install Python and run code in three ways (REPL, script, notebook).
- Create variables and follow naming conventions.
- Use the basic data types: `int`, `float`, `str`, `bool`, `None`.
- Convert between types and check a value's type.

---

## 1. Installing and running Python

1. Download Python 3.12+ from python.org (on Windows, tick **"Add python.exe to PATH"**).
2. Install **VS Code** and the **Python** extension.
3. Check the installation in a terminal:

```bash
python --version      # Python 3.12.x
python                # opens the REPL (interactive shell) — exit with exit()
python my_script.py   # runs a file
```

| Way to run code | When to use it |
|---|---|
| **REPL** (`>>>`) | Quick experiments, one line at a time |
| **Script** (`.py`) | Programs you want to keep and re-run |
| **Jupyter Notebook** (`.ipynb`) | Data exploration with tables and charts |

## 2. Variables

A **variable** is a name that points to a value stored in memory. In Python you do not declare the type — Python infers it (**dynamic typing**).

```python
project_name = "Sales Dashboard"   # str
rows = 1_250                        # int (underscores make big numbers readable)
price = 19.99                       # float
is_active = True                    # bool
manager = None                      # "no value"

print(type(rows))   # <class 'int'>
```

### Naming rules and conventions
- Use **snake_case**: `total_sales`, not `TotalSales` or `totalsales`.
- Names cannot start with a number: `2nd_value` ❌ → `second_value` ✅
- Do not use reserved words: `class`, `def`, `if`, `for`, `list` (shadowing built-ins is a classic bug).
- CONSTANTS in UPPER_CASE: `MAX_RETRIES = 3`.

### Multiple assignment and swapping
```python
x, y, z = 1, 2, 3
a = b = 0
x, y = y, x          # swap without a temporary variable
```

## 3. Numeric types

```python
10 + 3    # 13   addition
10 - 3    # 7    subtraction
10 * 3    # 30   multiplication
10 / 3    # 3.333... true division (always float)
10 // 3   # 3    floor division
10 % 3    # 1    modulo (remainder)
10 ** 3   # 1000 exponent
```

⚠️ **Floating-point precision:** `0.1 + 0.2 == 0.30000000000000004`. For money use `decimal.Decimal` or store cents as `int`.

```python
from decimal import Decimal
Decimal("0.1") + Decimal("0.2")   # Decimal('0.3')
round(3.14159, 2)                  # 3.14
```

## 4. Booleans and comparison

```python
5 > 3          # True
5 == 5.0       # True  (equal value)
5 != 4         # True
"a" < "b"      # True  (alphabetical order)

age = 30
18 <= age < 65         # True — chained comparison
(age > 18) and (age < 25)   # False
not True                    # False
```

**Truthiness:** these values are "falsy" → `0`, `0.0`, `""`, `[]`, `{}`, `None`, `False`. Everything else is "truthy".

## 5. Type conversion (casting)

```python
int("42")        # 42
float("3.5")     # 3.5
str(100)         # "100"
bool("")         # False
int("3.5")       # ❌ ValueError → use int(float("3.5"))
isinstance(3.5, float)   # True — preferred way to check types
```

## 6. Comments and style
```python
# A single-line comment explains WHY, not WHAT.
"""A docstring documents a module, function or class."""
```
Follow **PEP 8** (the official style guide): 4 spaces per indentation level, lines ≤ 88–100 characters.

---

## 🧪 Practice
1. Create variables for an employee: name, age, salary, is_remote. Print each value with its type.
2. Convert the string `"1,250.75"` into a float (hint: `.replace(",", "")`).
3. Calculate how many full weeks and remaining days there are in 100 days (`//` and `%`).
4. Why does `0.1 + 0.2 == 0.3` return `False`? Fix it with `math.isclose`.

<details><summary>Solution 3</summary>

```python
days = 100
weeks, remaining = days // 7, days % 7   # 14 weeks, 2 days
# or: weeks, remaining = divmod(days, 7)
```
</details>

---

## 🗣️ Vocabulary | Vocabulario

| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| variable | /ˈver.i.ə.bəl/ | **VE**-ri-a-bol | variable |
| value | /ˈvæl.juː/ | **VAL**-yu | valor |
| integer | /ˈɪn.tɪ.dʒər/ | **IN**-ti-yer | entero |
| float | /floʊt/ | flout | número decimal (coma flotante) |
| string | /strɪŋ/ | string | cadena de texto |
| boolean | /ˈbuː.li.ən/ | **BU**-li-an | booleano |
| assign | /əˈsaɪn/ | a-**SÁIN** | asignar |
| assignment | /əˈsaɪn.mənt/ | a-**SÁIN**-ment | asignación |
| type | /taɪp/ | taip | tipo |
| cast / casting | /kæst/ | kast | convertir tipo |
| remainder | /rɪˈmeɪn.dər/ | ri-**MÉIN**-der | residuo |
| division | /dɪˈvɪʒ.ən/ | di-**VI**-shon | división |
| reserved word | /rɪˈzɝːvd wɝːd/ | ri-**SERVD** werd | palabra reservada |
| indentation | /ˌɪn.denˈteɪ.ʃən/ | in-den-**TÉI**-shon | sangría |
| install | /ɪnˈstɑːl/ | in-**STOL** | instalar |
| run | /rʌn/ | ran | ejecutar |
| shell | /ʃel/ | shel | intérprete de comandos |

### 💬 Useful sentences
- *"The variable `rows` **stores** an integer."* — La variable `rows` guarda un entero.
- *"Python **infers** the type at runtime."* — Python infiere el tipo en tiempo de ejecución.
- *"This line **raises** a ValueError because the string is not a number."* — Esta línea lanza un ValueError porque la cadena no es un número.

---
➡️ Next: [[PY-B02 Strings and Input-Output]] · 🏠 [[00 - Python Roadmap]]

#python #beginner
