---
title: "PY-B04 Data Structures - Lists, Tuples, Dicts and Sets"
level: beginner
tags: [python, beginner, data-structures]
prev: "[[PY-B03 Control Flow - Conditionals and Loops]]"
next: "[[PY-B05 Functions]]"
---

# PY-B04 — Data Structures: Lists, Tuples, Dictionaries and Sets

## 🎯 Learning objectives
- Choose the right collection for each problem.
- Add, remove, search and sort items.
- Understand mutability and the "shared reference" trap.
- Know the time complexity (Big-O) of the most common operations.

---

## 0. Decision table

| Structure | Syntax | Ordered | Mutable | Duplicates | Typical use |
|---|---|---|---|---|---|
| **list** | `[1, 2, 3]` | ✅ | ✅ | ✅ | A sequence of rows/values |
| **tuple** | `(1, 2, 3)` | ✅ | ❌ | ✅ | Fixed records, coordinates, dict keys |
| **dict** | `{"id": 1}` | ✅ (insertion) | ✅ | keys ❌ | Lookup by key, JSON-like records |
| **set** | `{1, 2, 3}` | ❌ | ✅ | ❌ | Unique values, membership tests |

## 1. Lists
```python
scores = [88, 92, 75]
scores.append(99)          # add at the end      → [88, 92, 75, 99]
scores.insert(0, 60)       # add at position 0
scores.extend([70, 80])    # add many
scores.remove(75)          # remove first occurrence of a value
last = scores.pop()        # remove and return last item
scores.sort()              # sort in place
ranked = sorted(scores, reverse=True)   # new sorted list
scores.index(92)           # position of 92
92 in scores               # membership → True
scores[1:3]                # slicing works like strings
```

### Sorting by a key
```python
employees = [("Ana", 3200), ("Luis", 2800), ("Eva", 4100)]
employees.sort(key=lambda e: e[1], reverse=True)   # by salary, descending
```

## 2. Tuples
```python
point = (19.43, -99.13)       # latitude, longitude
lat, lon = point               # unpacking
single = (5,)                  # a 1-item tuple needs a comma
```
Use tuples for data that **must not change** and as dictionary keys: `{(2026, 11): 1500}`.

## 3. Dictionaries
```python
customer = {"id": 101, "name": "Ana", "country": "MX"}
customer["name"]                  # 'Ana'
customer.get("email", "n/a")      # safe access with default
customer["email"] = "ana@x.com"   # add / update
del customer["country"]           # delete
customer.keys(), customer.values(), customer.items()

for key, value in customer.items():
    print(f"{key} → {value}")
```

### Counting and grouping (very common in data work)
```python
from collections import Counter, defaultdict

colors = ["red", "blue", "red", "green", "red"]
Counter(colors)            # Counter({'red': 3, 'blue': 1, 'green': 1})
Counter(colors).most_common(1)   # [('red', 3)]

sales = [("North", 100), ("South", 50), ("North", 70)]
by_region = defaultdict(int)
for region, amount in sales:
    by_region[region] += amount   # {'North': 170, 'South': 50}
```

### Nested structures (JSON-like)
```python
order = {
    "id": 5001,
    "customer": {"name": "Luis", "tier": "gold"},
    "items": [{"sku": "A1", "qty": 2}, {"sku": "B7", "qty": 1}],
}
order["customer"]["tier"]          # 'gold'
sum(i["qty"] for i in order["items"])   # 3
```

## 4. Sets
```python
db_customers  = {"ana", "luis", "eva"}
crm_customers = {"eva", "mario"}

db_customers & crm_customers   # intersection → {'eva'}
db_customers | crm_customers   # union
db_customers - crm_customers   # difference → in DB but not CRM
db_customers ^ crm_customers   # symmetric difference
unique_ids = set([1, 1, 2, 3])  # remove duplicates → {1, 2, 3}
```

## 5. ⚠️ Mutability trap: references vs copies
```python
a = [1, 2, 3]
b = a          # b points to the SAME list
b.append(4)
print(a)       # [1, 2, 3, 4]  😱

c = a.copy()   # shallow copy  (or list(a), a[:])
import copy
d = copy.deepcopy(order)   # deep copy for nested structures
```

## 6. Performance (Big-O) cheat sheet
| Operation | list | dict / set |
|---|---|---|
| Access by index / key | O(1) | O(1) |
| `x in collection` | **O(n)** | **O(1)** |
| Append / add | O(1) | O(1) |
| Insert at start | O(n) | — |

💡 If you check membership many times, convert the list to a **set** first.

---

## 🧪 Practice
1. From `[3, 7, 3, 9, 7, 1]`, get the unique values sorted descending.
2. Build a dict `{word: length}` for `["sql", "python", "pandas"]`.
3. Count how many orders each customer made: `["ana","luis","ana","eva","ana"]`.
4. Find customers present in list A but missing in list B (two lists of 10,000 IDs). Which structure makes this fast?

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| list | /lɪst/ | list | lista |
| tuple | /ˈtuː.pəl/ | **TU**-pol | tupla |
| dictionary | /ˈdɪk.ʃə.ner.i/ | **DIK**-sho-ne-ri | diccionario |
| set | /set/ | set | conjunto |
| key | /kiː/ | ki | clave / llave |
| item | /ˈaɪ.t̬əm/ | **ÁI**-tem | elemento |
| append | /əˈpend/ | a-**PEND** | añadir al final |
| remove | /rɪˈmuːv/ | ri-**MUV** | eliminar |
| sort | /sɔːrt/ | sort | ordenar |
| unpacking | /ʌnˈpæk.ɪŋ/ | an-**PA**-king | desempaquetado |
| membership | /ˈmem.bɚ.ʃɪp/ | **MEM**-ber-ship | pertenencia |
| duplicate | /ˈduː.plɪ.kət/ | **DU**-pli-kit | duplicado |
| shallow copy | /ˈʃæl.oʊ ˈkɑː.pi/ | **SHA**-lou **KA**-pi | copia superficial |
| deep copy | /diːp ˈkɑː.pi/ | dip **KA**-pi | copia profunda |
| nested | /ˈnes.tɪd/ | **NES**-tid | anidado |

### 💬 Useful sentences
- *"Use a set **when you need** fast membership tests."*
- *"Dictionaries **map** keys **to** values."* — Los diccionarios asocian claves con valores.

---
⬅️ [[PY-B03 Control Flow - Conditionals and Loops]] · ➡️ [[PY-B05 Functions]] · 🏠 [[00 - Python Roadmap]]

#python #beginner
