---
id: pandas-timeseries
title: Pandas Module 8 - Time Series Analysis, Datetime Indexing, and Resampling
tags: [python, pandas, data-science, time-series, datetime, resampling]
aliases: [Pandas Time Series, Datetime Indexing, Resample and Roll]
---

# Pandas Module 8: Time Series Analysis, Datetime Indexing & Resampling

Time series data—such as financial ticks, sensor telemetry, and infrastructure monitoring logs—requires specialized temporal indexing. Pandas provides robust native support for datetime objects, time-zone conversions, and frequency resampling.

---

## 1. Beginner Level: Converting Strings to Datetime & Date Indexing

Standard text representations of dates (e.g., `'2026-08-19'`) are treated as generic strings by default. Converting them to `datetime64` enables powerful calendar-based operations.

```python
import pandas as pd

# Raw data with date strings
data = {
    'Date_Str': ['2026-01-15', '2026-02-20', '2026-03-10'],
    'Metric_Value': [120, 145, 132]
}
df = pd.DataFrame(data)

# 1. Convert string column to datetime
df['Date'] = pd.to_datetime(df['Date_Str'])

# 2. Set datetime as the DataFrame index for time-series operations
df.set_index('Date', inplace=True)
df.drop(columns=['Date_Str'], inplace=True)

print("Datetime Indexed DataFrame:\n", df)
```

---

## 2. Intermediate Level: Resampling, Date Parts and Time Zones

```python
df.loc["2026-02"]                      # partial string indexing: all of February
df.loc["2026-01-01":"2026-02-28"]

daily.resample("W").sum()              # weekly totals   (D, W, MS, ME, QE, YE, h, min)
daily.resample("MS").agg({"sales": "sum", "visits": "mean"})

df["year"] = df.index.year
df["month_name"] = df.index.month_name()
df["weekday"] = df.index.day_name()
df["is_weekend"] = df.index.dayofweek >= 5

ts = df.tz_localize("UTC").tz_convert("America/Mexico_City")
pd.date_range("2026-01-01", periods=12, freq="MS")       # first day of each month
pd.bdate_range("2026-11-01", "2026-11-30")                # business days
```

---

## 3. Advanced Level: Rolling Windows, Lags and Growth

```python
s = daily["sales"]
s.rolling(7).mean()                 # 7-day moving average
s.rolling("30D").sum()              # time-based window
s.ewm(span=14).mean()               # exponentially weighted
s.shift(1)                          # lag 1 (yesterday)
s.diff()                            # change vs previous
s.pct_change(periods=7)             # week-over-week growth
s.pct_change(periods=365)           # year-over-year (daily data)
s.expanding().max()                 # running maximum

# Fill missing dates explicitly
full = s.asfreq("D", fill_value=0)

# Features for forecasting
feat = pd.DataFrame({"y": s, "lag_1": s.shift(1), "lag_7": s.shift(7), "ma_7": s.shift(1).rolling(7).mean()})
```
⚠️ Avoid **data leakage**: features at time *t* must use only data available before *t* (`shift` first, then `rolling`).

## 🧪 Practice
1. Resample hourly sensor data to daily min/mean/max.
2. Compute month-over-month growth of revenue.
3. Build lag and moving-average features for a forecasting model.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| time series | /taɪm ˈsɪr.iːz/ | taim **SI**-riz | serie temporal |
| resample | /riːˈsæm.pəl/ | ri-**SAM**-pol | remuestrear |
| moving average | /ˈmuː.vɪŋ ˈæv.ɚ.ɪdʒ/ | **MU**-ving **A**-ve-rich | media móvil |
| lag | /læɡ/ | lag | rezago |
| growth | /ɡroʊθ/ | grouz | crecimiento |
| weekday | /ˈwiːk.deɪ/ | **UIK**-dei | día de la semana |
| leakage | /ˈliː.kɪdʒ/ | **LI**-kich | fuga (de información) |
| year-over-year | /jɪr ˈoʊ.vɚ jɪr/ | yir **OU**-ver yir | interanual |

⬅️ [[pandas_merging_joining]] · ➡️ [[Pandas_High_Performance]] · 🏠 [[Python Libraries Map]]
