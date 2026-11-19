---
title: "PY-I04 Working with Data Files and APIs"
level: intermediate
tags: [python, intermediate, csv, json, api, requests, datetime]
prev: "[[PY-I03 Environments, Packages and Project Structure]]"
next: "[[PY-I05 Testing, Type Hints and Code Quality]]"
---

# PY-I04 — Working with Data Files and APIs

## 🎯 Learning objectives
- Read/write CSV, JSON, Excel and Parquet.
- Handle dates and times correctly (time zones!).
- Consume REST APIs with `requests`: parameters, headers, pagination, errors, retries.

---

## 1. CSV with the standard library
```python
import csv

with open("orders.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)          # each row → dict
    orders = [row for row in reader]

with open("summary.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["region", "total"])
    writer.writeheader()
    writer.writerows([{"region": "North", "total": 170}])
```
For analysis, jump to pandas: `pd.read_csv("orders.csv", sep=";", decimal=",", parse_dates=["date"])` — see [[pandas_io_data_loading]].

## 2. JSON
```python
import json
data = json.loads('{"id": 1, "tags": ["a", "b"]}')   # str → Python
text = json.dumps(data, indent=2, ensure_ascii=False) # Python → str (keeps ñ, á)

with open("config.json", encoding="utf-8") as f:
    config = json.load(f)
with open("out.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
```
| JSON | Python |
|---|---|
| object `{}` | dict |
| array `[]` | list |
| string | str |
| number | int / float |
| true / false | True / False |
| null | None |

### Flattening nested JSON
```python
import pandas as pd
pd.json_normalize(api_response["results"], sep="_")
```

## 3. Excel and Parquet
```python
df = pd.read_excel("budget.xlsx", sheet_name="2026")      # needs openpyxl
df.to_parquet("budget.parquet", index=False)              # columnar, compressed, typed
```
💡 **Parquet** is the standard format for analytics: 5–10× smaller than CSV and keeps data types.

## 4. Dates and times
```python
from datetime import datetime, date, timedelta, timezone
from zoneinfo import ZoneInfo

now_utc = datetime.now(timezone.utc)
now_mx  = now_utc.astimezone(ZoneInfo("America/Mexico_City"))
d = datetime.strptime("18/11/2026 20:47", "%d/%m/%Y %H:%M")   # parse
d.strftime("%Y-%m-%d")                                         # format → '2026-11-18'
d.isoformat()                                                  # '2026-11-18T20:47:00'
d + timedelta(days=30)
(date(2026, 12, 31) - date(2026, 11, 18)).days                 # 43
```
✅ Store timestamps in **UTC**; convert to local time only for display.

| Code | Meaning | Example |
|---|---|---|
| `%Y` | 4-digit year | 2026 |
| `%m` | month | 11 |
| `%d` | day | 18 |
| `%H:%M:%S` | time | 20:47:00 |
| `%A` / `%B` | weekday / month name | Wednesday / November |

## 5. REST APIs with `requests`
An **API** (Application Programming Interface) lets programs talk to each other over HTTP.

| HTTP method | Action | SQL analogy |
|---|---|---|
| GET | read | SELECT |
| POST | create | INSERT |
| PUT / PATCH | update | UPDATE |
| DELETE | delete | DELETE |

| Status code | Meaning |
|---|---|
| 200 OK / 201 Created | success |
| 400 Bad Request | your request is wrong |
| 401 / 403 | not authenticated / not allowed |
| 404 Not Found | resource does not exist |
| 429 Too Many Requests | rate limit — slow down |
| 500–503 | server error — retry later |

```python
import os, requests

session = requests.Session()
session.headers.update({"Authorization": f"Bearer {os.environ['API_TOKEN']}"})

resp = session.get(
    "https://api.example.com/v1/orders",
    params={"status": "paid", "page": 1, "per_page": 100},
    timeout=10,                         # ALWAYS set a timeout
)
resp.raise_for_status()                 # raises HTTPError for 4xx/5xx
orders = resp.json()
```

### Pagination
```python
def fetch_all(url: str) -> list[dict]:
    results, page = [], 1
    while True:
        r = session.get(url, params={"page": page}, timeout=10)
        r.raise_for_status()
        batch = r.json()["results"]
        if not batch:
            break
        results.extend(batch)
        page += 1
    return results
```

### Retries with exponential backoff
```python
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

retry = Retry(total=5, backoff_factor=1, status_forcelist=[429, 500, 502, 503, 504])
session.mount("https://", HTTPAdapter(max_retries=retry))
```

### Posting JSON
```python
r = session.post("https://api.example.com/v1/tickets", json={"title": "Bug", "priority": "high"}, timeout=10)
r.status_code   # 201
```

---

## 🧪 Practice
1. Call `https://api.github.com/repos/pandas-dev/pandas` and print stars, forks and open issues.
2. Download all pages from a paginated public API and save the result as Parquet.
3. Convert a list of ISO timestamps in UTC to Mexico City time and group them by weekday.

---

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| header | /ˈhed.ɚ/ | **JE**-der | encabezado |
| field | /fiːld/ | fild | campo |
| parse | /pɑːrs/ | pars | analizar / interpretar |
| request | /rɪˈkwest/ | ri-**KUEST** | petición / solicitud |
| response | /rɪˈspɑːns/ | ri-**SPANS** | respuesta |
| endpoint | /ˈend.pɔɪnt/ | **END**-point | punto de acceso (URL) |
| token | /ˈtoʊ.kən/ | **TOU**-ken | token / credencial |
| rate limit | /reɪt ˈlɪm.ɪt/ | reit **LI**-mit | límite de peticiones |
| pagination | /ˌpædʒ.əˈneɪ.ʃən/ | pa-ye-**NÉI**-shon | paginación |
| retry | /ˌriːˈtraɪ/ | ri-**TRÁI** | reintentar |
| timeout | /ˈtaɪm.aʊt/ | **TÁIM**-aut | tiempo de espera |
| time zone | /ˈtaɪm zoʊn/ | taim zoun | zona horaria |
| timestamp | /ˈtaɪm.stæmp/ | **TÁIM**-stamp | marca de tiempo |
| backoff | /ˈbæk.ɑːf/ | **BAK**-of | espera progresiva |

### 💬 Useful sentences
- *"The API **returned** a 429, so we **backed off** and **retried**."*
- *"Always **set a timeout** on network calls."*

---
⬅️ [[PY-I03 Environments, Packages and Project Structure]] · ➡️ [[PY-I05 Testing, Type Hints and Code Quality]] · 🏠 [[00 - Python Roadmap]]

#python #intermediate
