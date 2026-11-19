---
title: "PY-I02 Object-Oriented Programming"
level: intermediate
tags: [python, intermediate, oop, classes]
prev: "[[PY-I01 Comprehensions, Iterators and Generators]]"
next: "[[PY-I03 Environments, Packages and Project Structure]]"
---

# PY-I02 — Object-Oriented Programming (OOP)

## 🎯 Learning objectives
- Define classes with attributes and methods.
- Use inheritance, composition and abstract base classes.
- Implement "dunder" (double underscore) methods.
- Use properties, class methods and static methods.

---

## 1. Classes and objects
A **class** is a blueprint; an **object** (instance) is something built from it.

```python
class BankAccount:
    bank_name = "DataBank"                 # class attribute (shared)

    def __init__(self, owner: str, balance: float = 0.0):
        self.owner = owner                 # instance attributes
        self.balance = balance

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("amount must be positive")
        self.balance += amount

    def withdraw(self, amount: float) -> None:
        if amount > self.balance:
            raise ValueError("insufficient funds")
        self.balance -= amount

acc = BankAccount("Ana", 100)
acc.deposit(50)
acc.balance        # 150
```
`self` is the instance itself — Python passes it automatically.

## 2. The four pillars
| Pillar | Meaning | Python tool |
|---|---|---|
| **Encapsulation** | hide internal state, expose a clean interface | `_private` convention, `@property` |
| **Abstraction** | expose *what*, hide *how* | `abc.ABC`, `@abstractmethod` |
| **Inheritance** | reuse and specialise a parent class | `class Child(Parent)` |
| **Polymorphism** | same method name, different behaviour | method overriding, duck typing |

## 3. Properties (encapsulation)
```python
class Temperature:
    def __init__(self, celsius: float):
        self.celsius = celsius            # goes through the setter

    @property
    def celsius(self) -> float:
        return self._celsius

    @celsius.setter
    def celsius(self, value: float) -> None:
        if value < -273.15:
            raise ValueError("below absolute zero")
        self._celsius = value

    @property
    def fahrenheit(self) -> float:        # computed, read-only
        return self._celsius * 9 / 5 + 32
```

## 4. Inheritance and abstract classes
```python
from abc import ABC, abstractmethod

class DataSource(ABC):
    @abstractmethod
    def read(self) -> list[dict]: ...

class CSVSource(DataSource):
    def __init__(self, path): self.path = path
    def read(self):
        import csv
        with open(self.path, encoding="utf-8") as f:
            return list(csv.DictReader(f))

class APISource(DataSource):
    def __init__(self, url): self.url = url
    def read(self):
        import requests
        return requests.get(self.url, timeout=10).json()

def load(source: DataSource):        # polymorphism: works with ANY source
    return source.read()
```
`super().__init__(...)` calls the parent's constructor.

💡 **Composition over inheritance:** prefer "has a" (an `ETLJob` *has a* `DataSource`) to deep "is a" hierarchies.

## 5. Dunder (magic) methods
```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount, self.currency = amount, currency
    def __repr__(self):               # for developers / debugging
        return f"Money({self.amount!r}, {self.currency!r})"
    def __str__(self):                # for users / print()
        return f"{self.amount:,.2f} {self.currency}"
    def __add__(self, other):
        if self.currency != other.currency:
            raise ValueError("currency mismatch")
        return Money(self.amount + other.amount, self.currency)
    def __eq__(self, other):
        return (self.amount, self.currency) == (other.amount, other.currency)
    def __lt__(self, other):
        return self.amount < other.amount

print(Money(10) + Money(5.5))   # 15.50 USD
```
Other useful dunders: `__len__`, `__getitem__`, `__iter__`, `__contains__`, `__enter__/__exit__`, `__call__`.

## 6. Class methods and static methods
```python
class Employee:
    def __init__(self, name, salary): self.name, self.salary = name, salary

    @classmethod
    def from_csv_row(cls, row: str) -> "Employee":   # alternative constructor
        name, salary = row.split(",")
        return cls(name, float(salary))

    @staticmethod
    def is_valid_salary(value: float) -> bool:       # utility, no self/cls
        return value > 0

e = Employee.from_csv_row("Ana,3200")
```

## 7. When should a data person use OOP?
- ✅ Connectors (database, API), pipeline steps, ML model wrappers, configuration objects.
- ✅ scikit-learn custom transformers (`fit` / `transform`) — see [[sklearn_api_design]].
- ❌ Simple transformations → plain functions are enough.
- For "data containers", prefer `@dataclass` → [[PY-A02 Dataclasses, Pydantic and Data Validation]].

---

## 🧪 Practice
1. Create `Product(name, price, stock)` with a `total_value` property and a `__repr__`.
2. Create `Inventory` that *contains* a list of products, supports `len(inv)` and `for p in inv`.
3. Create an abstract `Exporter` with `CSVExporter` and `JSONExporter` subclasses.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| class | /klæs/ | klas | clase |
| object | /ˈɑːb.dʒekt/ | **AB**-yekt | objeto |
| instance | /ˈɪn.stəns/ | **INS**-tans | instancia |
| attribute | /ˈæt.rɪ.bjuːt/ | **A**-tri-biut | atributo |
| blueprint | /ˈbluː.prɪnt/ | **BLU**-print | plano / plantilla |
| inheritance | /ɪnˈher.ɪ.t̬əns/ | in-**JE**-ri-tans | herencia |
| inherit | /ɪnˈher.ɪt/ | in-**JE**-rit | heredar |
| override | /ˌoʊ.vɚˈraɪd/ | ou-ver-**RÁID** | sobrescribir (método) |
| encapsulation | /ɪnˌkæp.sjəˈleɪ.ʃən/ | in-kap-siu-**LÉI**-shon | encapsulamiento |
| polymorphism | /ˌpɑː.liˈmɔːr.fɪ.zəm/ | pa-li-**MOR**-fi-zem | polimorfismo |
| abstract | /ˈæb.strækt/ | **AB**-strakt | abstracto |
| constructor | /kənˈstrʌk.tɚ/ | kon-**STRAK**-ter | constructor |
| funds | /fʌndz/ | fandz | fondos |
| insufficient | /ˌɪn.səˈfɪʃ.ənt/ | in-so-**FI**-shent | insuficiente |

### 💬 Useful sentences
- *"`CSVSource` **inherits from** `DataSource` and **overrides** `read()`."*
- *"We **favour composition over inheritance**."* — Preferimos composición sobre herencia.

---
⬅️ [[PY-I01 Comprehensions, Iterators and Generators]] · ➡️ [[PY-I03 Environments, Packages and Project Structure]] · 🏠 [[00 - Python Roadmap]]

#python #intermediate
