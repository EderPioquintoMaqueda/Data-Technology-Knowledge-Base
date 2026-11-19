---
title: "PY-A02 Dataclasses, Pydantic and Data Validation"
level: advanced
tags: [python, advanced, dataclasses, pydantic, validation, pandera]
prev: "[[PY-A01 Decorators, Closures and Context Managers]]"
next: "[[PY-A03 Concurrency - Threads, Processes and Asyncio]]"
---

# PY-A02 — Dataclasses, Pydantic and Data Validation

## 🎯 Learning objectives
- Model records with `@dataclass` (frozen, slots, defaults).
- Validate external data with **Pydantic v2** (types, constraints, custom validators).
- Validate whole DataFrames with **Pandera**.
- Know *where* to validate: at the system boundaries.

---

## 1. Dataclasses
```python
from dataclasses import dataclass, field, asdict
from datetime import date

@dataclass(frozen=True, slots=True)      # immutable + memory-efficient
class Transaction:
    id: int
    amount: float
    currency: str = "USD"
    tags: list[str] = field(default_factory=list)   # never use [] as default
    created: date = field(default_factory=date.today)

    def __post_init__(self):
        if self.amount < 0:
            raise ValueError("amount must be >= 0")

t = Transaction(1, 99.5)
asdict(t)        # → dict, ready for JSON or a DataFrame
```
`@dataclass` generates `__init__`, `__repr__`, `__eq__` (and `__hash__` if frozen) for you.

| Option | Effect |
|---|---|
| `frozen=True` | attributes cannot change (safe as dict keys) |
| `slots=True` | lower memory, faster attribute access |
| `order=True` | enables `<`, `>` sorting |
| `kw_only=True` | all fields must be passed by keyword |

## 2. Pydantic v2 — validation and parsing
Dataclasses do **not** check types at runtime. Pydantic does — and it converts ("coerces") compatible input.

```python
from pydantic import BaseModel, Field, EmailStr, field_validator, model_validator, ValidationError
from datetime import datetime
from typing import Literal

class Customer(BaseModel):
    id: int
    name: str = Field(min_length=1, max_length=100)
    email: EmailStr                                # pip install "pydantic[email]"
    country: Literal["MX", "CO", "PE", "US"]
    credit_limit: float = Field(ge=0, le=1_000_000)
    signup: datetime

    @field_validator("name")
    @classmethod
    def clean_name(cls, v: str) -> str:
        return v.strip().title()

class Order(BaseModel):
    order_id: int
    customer: Customer
    quantity: int = Field(gt=0)
    unit_price: float = Field(gt=0)
    discount: float = 0.0

    @model_validator(mode="after")
    def discount_not_bigger_than_total(self):
        if self.discount > self.quantity * self.unit_price:
            raise ValueError("discount exceeds total")
        return self

raw = {"order_id": "17", "quantity": 2, "unit_price": 10, "customer": {
       "id": 1, "name": "  ana lopez ", "email": "ana@x.com", "country": "MX",
       "credit_limit": 500, "signup": "2026-11-18T10:00:00"}}
order = Order.model_validate(raw)      # "17" → 17, string → datetime
order.customer.name                    # 'Ana Lopez'
order.model_dump()                     # → dict
order.model_dump_json()                # → JSON string

try:
    Order.model_validate({**raw, "quantity": 0})
except ValidationError as e:
    print(e.errors())                  # precise, machine-readable errors
```

### Settings from environment variables
```python
from pydantic_settings import BaseSettings
class Settings(BaseSettings):
    db_url: str
    batch_size: int = 1000
    model_config = {"env_file": ".env"}
settings = Settings()                   # reads DB_URL, BATCH_SIZE
```

### Validating many records — keep the good, quarantine the bad
```python
valid, rejected = [], []
for rec in api_records:
    try:
        valid.append(Order.model_validate(rec))
    except ValidationError as e:
        rejected.append({"record": rec, "errors": e.errors()})
```

## 3. DataFrame validation with Pandera
```python
import pandera.pandas as pa      # pandera >= 0.24 (older: import pandera as pa)

schema = pa.DataFrameSchema({
    "order_id":   pa.Column(int, unique=True),
    "amount":     pa.Column(float, pa.Check.ge(0)),
    "status":     pa.Column(str, pa.Check.isin(["paid", "pending", "refunded"])),
    "created_at": pa.Column("datetime64[ns]"),
}, strict=True)                   # no unexpected columns

clean_df = schema.validate(df, lazy=True)   # lazy → report ALL errors at once
```

## 4. Where to validate?
```
External world  ──►  [VALIDATE]  ──►  your clean core logic  ──►  [VALIDATE]  ──►  database / API
 (APIs, CSVs,         Pydantic /                                    Pandera /
  user input)         Pandera                                       DB constraints
```
"**Parse, don't validate**": convert raw input into typed objects once, at the edge; inside, trust the types.

## 5. Comparison
| Tool | Runtime validation | Speed | Best for |
|---|---|---|---|
| `dataclass` | ❌ (manual) | fastest | internal records |
| `attrs` | optional | fast | libraries |
| **Pydantic** | ✅ | fast (Rust core) | APIs, configs, JSON |
| **Pandera** | ✅ (DataFrame) | vectorised | tabular pipelines |
| DB constraints | ✅ | — | the final line of defence → [[SQL-B04 Creating Tables - DDL, DML and Constraints]] |

---

## 🧪 Practice
1. Model a `Product` with Pydantic: SKU pattern `^[A-Z]{3}-\d{4}$`, price > 0, stock ≥ 0.
2. Validate 1,000 fake records (some broken) and write the rejected ones to `rejected.json`.
3. Write a Pandera schema for a sales DataFrame and make it fail on purpose.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| validate | /ˈvæl.ə.deɪt/ | **VA**-li-deit | validar |
| constraint | /kənˈstreɪnt/ | kon-**STRÉINT** | restricción |
| coerce | /koʊˈɝːs/ | kou-**ERS** | forzar conversión |
| boundary | /ˈbaʊn.dɚ.i/ | **BÁUN**-de-ri | frontera / límite |
| schema | /ˈskiː.mə/ | **SKI**-ma | esquema |
| field | /fiːld/ | fild | campo |
| frozen | /ˈfroʊ.zən/ | **FROU**-zen | congelado (inmutable) |
| quarantine | /ˈkwɔːr.ən.tiːn/ | **KUÓ**-ran-tin | cuarentena |
| reject | /rɪˈdʒekt/ | ri-**YEKT** | rechazar |
| trust | /trʌst/ | trast | confiar |
| exceed | /ɪkˈsiːd/ | ik-**SID** | exceder |
| settings | /ˈset̬.ɪŋz/ | **SE**-tings | configuración |

### 💬 Useful sentences
- *"Pydantic **raises a ValidationError** when the payload **doesn't match** the schema."*
- *"We validate data **at the edges** of the system."* — Validamos los datos en los bordes del sistema.

---
⬅️ [[PY-A01 Decorators, Closures and Context Managers]] · ➡️ [[PY-A03 Concurrency - Threads, Processes and Asyncio]] · 🏠 [[00 - Python Roadmap]]

#python #advanced
