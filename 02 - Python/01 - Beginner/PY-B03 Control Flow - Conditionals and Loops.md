---
title: "PY-B03 Control Flow - Conditionals and Loops"
level: beginner
tags: [python, beginner, control-flow, loops]
prev: "[[PY-B02 Strings and Input-Output]]"
next: "[[PY-B04 Data Structures - Lists, Tuples, Dicts and Sets]]"
---

# PY-B03 — Control Flow: Conditionals and Loops

## 🎯 Learning objectives
- Make decisions with `if / elif / else` and `match`.
- Repeat work with `for` and `while`.
- Control loops with `break`, `continue`, `else`.
- Use `range`, `enumerate` and `zip` like a professional.

---

## 1. Conditionals
```python
temperature = 31

if temperature >= 30:
    status = "hot"
elif temperature >= 18:
    status = "pleasant"
else:
    status = "cold"
```
- Indentation (4 spaces) defines the **block**. There are no braces `{}`.
- Conditions are evaluated top to bottom; the **first** true branch runs.

### Conditional expression (ternary)
```python
label = "adult" if age >= 18 else "minor"
```

### `match` (Python 3.10+) — structural pattern matching
```python
def http_message(code: int) -> str:
    match code:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500 | 502 | 503:
            return "Server Error"
        case _:
            return "Unknown"
```

## 2. `for` loops — iterate over a collection
```python
for city in ["Lima", "Quito", "Bogotá"]:
    print(city)

for i in range(5):          # 0,1,2,3,4
    ...
for i in range(2, 10, 2):   # 2,4,6,8
    ...
```

### `enumerate` — index + value
```python
for position, name in enumerate(["Ana", "Luis"], start=1):
    print(position, name)   # 1 Ana / 2 Luis
```

### `zip` — iterate several sequences in parallel
```python
products = ["A", "B", "C"]
prices   = [10, 20, 30]
for product, price in zip(products, prices):
    print(product, price)
```

## 3. `while` loops — repeat while a condition is true
```python
attempts = 0
while attempts < 3:
    print("Trying to connect...")
    attempts += 1
```
⚠️ Always make sure the condition eventually becomes `False` — otherwise you create an **infinite loop** (stop it with `Ctrl + C`).

## 4. `break`, `continue`, and `else`
```python
for row in rows:
    if row is None:
        continue        # skip this item, go to the next
    if row == "STOP":
        break           # exit the loop completely
    process(row)
else:
    print("Finished without break")   # runs only if the loop was NOT broken
```

## 5. Accumulator pattern (the most common loop in data work)
```python
sales = [120, 340, 90, 410]
total = 0
for s in sales:
    total += s
average = total / len(sales)

# Pythonic version:
total = sum(sales); maximum = max(sales); minimum = min(sales)
```

## 6. Nested loops
```python
for row in range(1, 4):
    for col in range(1, 4):
        print(row * col, end="\t")
    print()
```
💡 Nested loops over big data are slow (O(n²)). In data work prefer vectorised tools (NumPy/pandas) — see [[PY-A04 Performance, Profiling and Memory]].

---

## 🧪 Practice
1. **FizzBuzz:** print numbers 1–30; "Fizz" for multiples of 3, "Buzz" for 5, "FizzBuzz" for both.
2. Given a list of temperatures, count how many days were above 30°C.
3. Ask for a password until the user types `"python123"` (max 3 attempts).
4. With `zip`, print `name: score` pairs and the name of the best student.

<details><summary>Solution 1</summary>

```python
for n in range(1, 31):
    out = ("Fizz" if n % 3 == 0 else "") + ("Buzz" if n % 5 == 0 else "")
    print(out or n)
```
</details>

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| condition | /kənˈdɪʃ.ən/ | kon-**DI**-shon | condición |
| statement | /ˈsteɪt.mənt/ | **STÉIT**-ment | sentencia / instrucción |
| branch | /bræntʃ/ | branch | rama |
| loop | /luːp/ | lup | bucle / ciclo |
| iterate | /ˈɪt.ə.reɪt/ | **I**-te-reit | iterar / recorrer |
| iteration | /ˌɪt.əˈreɪ.ʃən/ | i-te-**RÉI**-shon | iteración |
| nested | /ˈnes.tɪd/ | **NES**-tid | anidado |
| skip | /skɪp/ | skip | saltar / omitir |
| exit | /ˈek.sɪt/ | **EK**-sit | salir |
| infinite | /ˈɪn.fɪ.nət/ | **IN**-fi-nit | infinito |
| attempt | /əˈtempt/ | a-**TEMPT** | intento |
| accumulator | /əˈkjuː.mjə.leɪ.t̬ər/ | a-**KIU**-miu-lei-ter | acumulador |
| whether | /ˈweð.ər/ | **UE**-der | si (condicional) |
| otherwise | /ˈʌð.ɚ.waɪz/ | **Á**-der-uais | de lo contrario |

### 💬 Useful sentences
- *"**If** the file exists, read it; **otherwise**, create it."*
- *"The loop **stops as soon as** it finds a match."* — El bucle se detiene en cuanto encuentra una coincidencia.
- *"`continue` **skips** the rest of the current iteration."*

---
⬅️ [[PY-B02 Strings and Input-Output]] · ➡️ [[PY-B04 Data Structures - Lists, Tuples, Dicts and Sets]] · 🏠 [[00 - Python Roadmap]]

#python #beginner
