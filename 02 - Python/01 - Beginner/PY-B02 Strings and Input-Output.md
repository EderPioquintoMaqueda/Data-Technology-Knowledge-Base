---
title: "PY-B02 Strings and Input-Output"
level: beginner
tags: [python, beginner, strings, io]
prev: "[[PY-B01 Setup, Variables and Data Types]]"
next: "[[PY-B03 Control Flow - Conditionals and Loops]]"
---

# PY-B02 — Strings and Input/Output

## 🎯 Learning objectives
- Create, index and slice strings.
- Use the most common string methods to clean text data.
- Format output with **f-strings**.
- Read user input safely.

---

## 1. Creating strings
```python
single = 'data'
double = "data"
multi  = """Line 1
Line 2"""
path   = r"C:\Users\ana\data.csv"   # raw string: backslashes are not escapes
```

## 2. Indexing and slicing
Strings are **sequences** — every character has a position (index), starting at **0**.

```python
word = "DATABASE"
word[0]      # 'D'
word[-1]     # 'E'   (negative index counts from the end)
word[0:4]    # 'DATA' (start included, end excluded)
word[4:]     # 'BASE'
word[::-1]   # 'ESABATAD' (reversed)
len(word)    # 8
```
Strings are **immutable**: `word[0] = "d"` raises `TypeError`. Create a new string instead.

## 3. Essential string methods (data cleaning)
```python
raw = "   Mexico City ;  "
raw.strip()            # 'Mexico City ;'
raw.strip(" ;")        # 'Mexico City'
"sales".upper()        # 'SALES'
"SALES".lower()        # 'sales'
"john smith".title()   # 'John Smith'
"a,b,c".split(",")     # ['a', 'b', 'c']
"-".join(["2026", "11", "18"])   # '2026-11-18'
"total_sales".replace("_", " ")  # 'total sales'
"report.csv".endswith(".csv")    # True
"data" in "big data"             # True
"banana".count("a")              # 3
"banana".find("n")               # 2 (-1 if not found)
"42".isdigit()                   # True
"  ".isspace()                   # True
```

## 4. f-strings (formatted string literals)
```python
name, revenue, growth = "North", 1250300.5, 0.0734
print(f"Region {name}: ${revenue:,.2f} ({growth:.1%})")
# Region North: $1,250,300.50 (7.3%)

f"{42:05d}"       # '00042'  zero padding
f"{'left':<10}|"  # 'left      |'
f"{'right':>10}|" # '     right|'
f"{3.14159:.3f}"  # '3.142'
f"{revenue=}"     # 'revenue=1250300.5'  (great for debugging)
```

## 5. Output: `print`
```python
print("a", "b", "c", sep=" | ")   # a | b | c
print("Loading", end="...")       # no newline
print()                            # empty line
```

## 6. Input: `input`
`input()` **always returns a string**. Convert and validate it:
```python
raw = input("How many rows? ")
if raw.isdigit():
    rows = int(raw)
    print(f"You asked for {rows} rows.")
else:
    print("Please type a whole number.")
```

## 7. Escape characters
| Sequence | Meaning |
|---|---|
| `\n` | new line |
| `\t` | tab |
| `\\` | backslash |
| `\"` | double quote |

---

## 🧪 Practice
1. Clean this list of cities: `["  bogotá", "LIMA ", " mexico city"]` → `["Bogotá", "Lima", "Mexico City"]`.
2. Given `email = "ana.lopez@company.com"`, extract the user (`ana.lopez`) and the domain (`company.com`).
3. Print a table of 3 products with names aligned left (15 chars) and prices aligned right with 2 decimals.
4. Ask the user for their name and year of birth; print their age in 2026.

<details><summary>Solutions</summary>

```python
cities = [c.strip().title() for c in ["  bogotá", "LIMA ", " mexico city"]]
user, domain = "ana.lopez@company.com".split("@")
for name, price in [("Laptop", 999.9), ("Mouse", 15), ("Monitor", 189.5)]:
    print(f"{name:<15}{price:>10.2f}")
```
</details>

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| character | /ˈker.ək.tər/ | **KE**-rak-ter | carácter |
| index (pl. indices/indexes) | /ˈɪn.deks/ | **IN**-deks | índice |
| slice | /slaɪs/ | slais | rebanada / segmento |
| immutable | /ɪˈmjuː.tə.bəl/ | i-**MIU**-ta-bol | inmutable |
| method | /ˈmeθ.əd/ | **ME**-zod | método |
| whitespace | /ˈwaɪt.speɪs/ | **UÁIT**-speis | espacio en blanco |
| split | /splɪt/ | split | dividir |
| join | /dʒɔɪn/ | yoin | unir |
| replace | /rɪˈpleɪs/ | ri-**PLÉIS** | reemplazar |
| padding | /ˈpæd.ɪŋ/ | **PA**-ding | relleno |
| prompt | /prɑːmpt/ | prampt | mensaje de solicitud |
| quote | /kwoʊt/ | kuout | comilla / cita |
| backslash | /ˈbæk.slæʃ/ | **BAK**-slash | barra invertida |
| left-aligned | /left əˈlaɪnd/ | left a-**LÁIND** | alineado a la izquierda |

### 💬 Useful sentences
- *"Strings are immutable, so `replace` **returns a new string**."* — Las cadenas son inmutables; `replace` devuelve una cadena nueva.
- *"The end index **is excluded** from the slice."* — El índice final se excluye del segmento.

---
⬅️ [[PY-B01 Setup, Variables and Data Types]] · ➡️ [[PY-B03 Control Flow - Conditionals and Loops]] · 🏠 [[00 - Python Roadmap]]

#python #beginner
