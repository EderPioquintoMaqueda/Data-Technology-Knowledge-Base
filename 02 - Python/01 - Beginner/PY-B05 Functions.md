---
title: "PY-B05 Functions"
level: beginner
tags: [python, beginner, functions]
prev: "[[PY-B04 Data Structures - Lists, Tuples, Dicts and Sets]]"
next: "[[PY-B06 Files, Modules and Error Handling]]"
---

# PY-B05 — Functions

## 🎯 Learning objectives
- Define functions with parameters, defaults and return values.
- Understand positional vs keyword arguments, `*args` and `**kwargs`.
- Understand **scope** (local vs global).
- Write small, pure, documented functions.

---

## 1. Defining and calling
```python
def net_price(price: float, tax_rate: float = 0.16) -> float:
    """Return the price including tax."""
    return round(price * (1 + tax_rate), 2)

net_price(100)                # 116.0  (uses the default)
net_price(100, 0.08)          # positional
net_price(price=100, tax_rate=0.08)   # keyword arguments (clearer)
```
- **Parameters** are the names in the definition; **arguments** are the values you pass.
- A function without `return` returns `None`.

## 2. Returning several values
```python
def stats(values: list[float]) -> tuple[float, float, float]:
    return min(values), max(values), sum(values) / len(values)

low, high, mean = stats([4, 8, 15, 16, 23, 42])
```

## 3. `*args` and `**kwargs`
```python
def total(*amounts):              # any number of positional args → tuple
    return sum(amounts)
total(1, 2, 3)                    # 6

def build_query(table, **filters):   # any keyword args → dict
    where = " AND ".join(f"{k} = '{v}'" for k, v in filters.items())
    return f"SELECT * FROM {table} WHERE {where};"

build_query("orders", status="paid", country="MX")
```
⚠️ The example above is for learning only — never build real SQL with f-strings (SQL injection). See [[PY-A05 Python and Databases]].

### Keyword-only and positional-only arguments
```python
def connect(host, port, /, *, timeout=10):   # host, port positional-only; timeout keyword-only
    ...
connect("localhost", 5432, timeout=5)
```

## 4. ⚠️ The mutable default trap
```python
def add_item(item, bucket=[]):    # ❌ the same list is reused on every call
    bucket.append(item); return bucket

def add_item(item, bucket=None):  # ✅
    if bucket is None:
        bucket = []
    bucket.append(item); return bucket
```

## 5. Scope — LEGB rule
Python looks for a name in: **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in.
```python
rate = 0.16            # global
def f():
    rate = 0.08        # local — does NOT change the global
    return rate
```
Avoid `global`. Pass values as parameters and return results (**pure functions**).

## 6. Lambda functions
Small anonymous functions, mainly used as arguments:
```python
square = lambda x: x ** 2
sorted(words, key=lambda w: len(w))
```

## 7. Functions are objects
```python
def shout(text): return text.upper()
def apply(func, value): return func(value)
apply(shout, "hello")     # 'HELLO'
list(map(str.strip, [" a ", " b"]))      # ['a', 'b']
list(filter(lambda x: x > 0, [-1, 2, 3]))   # [2, 3]
```

## 8. Docstrings and good design
```python
def normalize_email(email: str) -> str:
    """Lower-case and strip an email address.

    Args:
        email: raw email typed by a user.
    Returns:
        The cleaned email.
    Example:
        >>> normalize_email("  Ana@Mail.COM ")
        'ana@mail.com'
    """
    return email.strip().lower()
```
**Rules of thumb:** one function = one job · short (≤ 20–30 lines) · descriptive verb names (`load_data`, `clean_names`, `save_report`).

---

## 🧪 Practice
1. Write `celsius_to_fahrenheit(c)` and the inverse.
2. Write `describe(values)` that returns a dict with count, mean, min, max.
3. Write `safe_divide(a, b, default=0)`.
4. Write `apply_discount(prices, *, percent)` that forces `percent` to be passed by keyword.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| function | /ˈfʌŋk.ʃən/ | **FANK**-shon | función |
| define | /dɪˈfaɪn/ | di-**FÁIN** | definir |
| call | /kɑːl/ | kol | llamar / invocar |
| parameter | /pəˈræm.ə.t̬ɚ/ | pa-**RA**-me-ter | parámetro |
| argument | /ˈɑːrɡ.jə.mənt/ | **AR**-guiu-ment | argumento |
| default | /dɪˈfɑːlt/ | di-**FOLT** | valor por defecto |
| return | /rɪˈtɝːn/ | ri-**TERN** | devolver / retornar |
| scope | /skoʊp/ | skoup | alcance / ámbito |
| side effect | /saɪd ɪˈfekt/ | said i-**FEKT** | efecto secundario |
| keyword | /ˈkiː.wɝːd/ | **KI**-werd | palabra clave |
| reuse | /riːˈjuːz/ | ri-**YUZ** | reutilizar |
| anonymous | /əˈnɑː.nə.məs/ | a-**NA**-ni-mos | anónimo |
| docstring | /ˈdɑːk.strɪŋ/ | **DAK**-string | cadena de documentación |
| behaviour | /bɪˈheɪ.vjɚ/ | bi-**JÉI**-vier | comportamiento |

### 💬 Useful sentences
- *"This function **takes** two arguments and **returns** a float."* — Esta función recibe dos argumentos y devuelve un float.
- *"Avoid **mutable defaults**; use `None` instead."*

---
⬅️ [[PY-B04 Data Structures - Lists, Tuples, Dicts and Sets]] · ➡️ [[PY-B06 Files, Modules and Error Handling]] · 🏠 [[00 - Python Roadmap]]

#python #beginner
