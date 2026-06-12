## 🗓️ nse-calendar

📦 **NSE (National Stock Exchange of India) trading calendar for Python.**

[![PyPI version](https://img.shields.io/pypi/v/nse-calendar?color=blue)](https://pypi.org/project/nse-calendar/)
[![Python versions](https://img.shields.io/pypi/pyversions/nse-calendar)](https://pypi.org/project/nse-calendar/)
[![GitHub Stars](https://img.shields.io/github/stars/kartikkasat/nse-calendar-data?style=flat&color=lightblue)](https://github.com/kartikkasat/nse-calendar-data/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/kartikkasat/nse-calendar-data?style=flat&color=yellow)](https://github.com/kartikkasat/nse-calendar-data/network/members)
[![Pepy Total Downloads](https://img.shields.io/pepy/dt/nse-calendar?color=blue)](https://pypi.org/project/nse-calendar/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![PyPI - Implementation](https://img.shields.io/pypi/implementation/nse-calendar?color=blue)](https://pypi.org/project/nse-calendar/)

 **Features:** Holiday detection, trading day arithmetic, flexible date parsing/formatting, and seamless Pandas integration. 
 **Zero mandatory dependencies.**

---

## 🔍 Data Coverage

The package includes one of the most comprehensive publicly available NSE holiday datasets, covering trading holidays from 1996 onwards and updated annually using official NSE circulars and archives.

| Year Range | Status | Source |
| :--- | :--- | :--- |
| **1996–2009** | ✅ Available | NSE historical archives |
| **2010–2025** | ✅ Available | NSE official annual circulars |
| **2026** | ✅ Available | NSE official annual circular |
| **2027 and beyond** | ⏳ Added each December | NSE official annual circular |

---

## ⚡ Installation

```bash
pip install nse-calendar

```

> **Note:** `pandas` and `requests` are optional dependencies. The library works flawlessly with Python's standard library alone.

---

## 🚀 Quick Start

```python
import nse_calendar as nse

# Is this a trading day?
nse.is_nse_holiday("2026-01-26")       # True (Republic Day)
nse.is_nse_holiday("2026-05-25")       # False

# Move forward by trading days
nse.add_trading_days("2026-01-23", 1)   # 2026-01-27

# Count trading days in a range
nse.trading_days_between("2026-01-01", "2026-01-31")

# Parse and format dates easily
d = nse.parse_date("25-May-2026")
nse.format_date(d, "dd/mm/yyyy")       # '25/05/2026'
nse.format_date(d, "dd Month yyyy")   # '25 May 2026'

```

---

## 🛠️ API Reference

### 1. Holiday Checks

#### `is_nse_holiday(date) -> bool`

Returns `True` if the date is an NSE trading holiday or a weekend. Returns `False` for normal trading days.

```python
nse.is_nse_holiday("2026-01-26")                  # True  - Republic Day
nse.is_nse_holiday("2026-05-23")                  # True  - Saturday
nse.is_nse_holiday("2026-05-25")                  # False - Monday, normal session
nse.is_nse_holiday(datetime.date(2026, 12, 25))    # True  - Christmas

```

#### `get_holidays(year) -> list[dict]`

Returns all NSE holidays for a given year as a list of dictionaries. Returns an empty list if the year is not present in the dataset.

```python
nse.get_holidays(2026)
# Output:
# [
#     {"date": "2026-01-26", "description": "Republic Day"},
#     {"date": "2026-03-03", "description": "Maha Shivaratri"},
#     ...
# ]

len(nse.get_holidays(2026))   # 16
nse.get_holidays(1995)        # []

```

---

### 2. Special Sessions

Special sessions, such as **Muhurat Trading on Diwali**, override normal weekday and weekend rules. They are considered trading days even if they fall on a weekend.

#### `is_special_session(date) -> bool`

Returns `True` if the date is a special trading session.

```python
nse.is_special_session("2026-11-08")   # True (Diwali Lakshmi Puja)
nse.is_special_session("2026-01-26")   # False

```

#### `get_special_sessions(year) -> list[dict]`

Returns all special trading sessions for a given year.

```python
nse.get_special_sessions(2026)
# Output:
# [
#     {"date": "2026-11-08", "description": "Diwali Lakshmi Puja"}
# ]

```

---

### 3. Trading Day Arithmetic

#### `add_trading_days(date, n) -> datetime.date`

Advances a date by exactly `n` trading days, skipping weekends and NSE holidays. A negative value of `n` moves backward.

```python
nse.add_trading_days("2026-01-23", 1)   # date(2026, 1, 27)
nse.add_trading_days("2026-05-25", 5)   # date(2026, 6, 1)
nse.add_trading_days("2026-05-25", -3)  # date(2026, 5, 20)

```

#### `subtract_trading_days(date, n) -> datetime.date`

Moves a date backward by exactly `n` trading days.

```python
nse.subtract_trading_days("2026-01-27", 1) # date(2026, 1, 23)
nse.subtract_trading_days("2026-06-01", 5) # date(2026, 5, 25)

```

#### `trading_days_between(start, end, inclusive="both") -> int`

Counts trading days between two dates. The order of start and end does not matter.

| `inclusive` option | Count Start? | Count End? |
| --- | --- | --- |
| `"both"` | Yes | Yes |
| `"left"` | Yes | No |
| `"right"` | No | Yes |
| `"neither"` | No | No |

```python
nse.trading_days_between("2026-06-01", "2026-06-05")                       # 5
nse.trading_days_between("2026-06-01", "2026-06-05", inclusive="neither")  # 3
nse.trading_days_between("2026-01-22", "2026-01-30")                       # 5

```

#### `next_trading_day(date) -> datetime.date` | `prev_trading_day(date)`

Returns the next or previous strict trading day.

```python
nse.next_trading_day("2026-01-26")  # date(2026, 1, 27)
nse.prev_trading_day("2026-01-27")  # date(2026, 1, 23)

```

#### `nearest_trading_day(date, direction="forward") -> datetime.date`

If the supplied date is already a trading day, it returns it unchanged. Otherwise, it finds the nearest one based on direction (`"forward"` or `"backward"`).

```python
nse.nearest_trading_day("2026-05-25")                        # date(2026, 5, 25)
nse.nearest_trading_day("2026-01-26")                        # date(2026, 1, 27)
nse.nearest_trading_day("2026-01-26", direction="backward")  # date(2026, 1, 23)

```

---

### 4. Date Parsing & Formatting

#### `parse_date(value) -> datetime.date`

Converts strings (`"2026-05-25"`, `"25-May-26"`, `"May 25, 2026"`), Unix timestamps, or Pandas/Python datetimes smoothly into a `datetime.date` object.

```python
nse.parse_date("25-May-2026")  # date(2026, 5, 25)
nse.parse_date(1748131200)     # date from Unix timestamp

```

#### `format_date(date, fmt) -> str`

Formats a date using human-readable tokens (`dd-mm-yyyy`, `dd Month yyyy`, etc.) or any standard `%` `strftime` pattern.

```python
nse.format_date("2026-05-25", "dd Month yyyy")  # '25 May 2026'
nse.format_date("2026-05-25", "%A, %d %B %Y")    # 'Monday, 25 May 2026'

```

---

### 5. Pandas Integration

#### `resolve_dataframe(df, col, output_col=None, fmt="dd-mm-yyyy") -> pd.DataFrame`

Parses mixed-format data columns, formats them uniformly, and returns helper flags without modifying your original DataFrame.

| Added Column | Type | Description |
| --- | --- | --- |
| `{col}` or `output_col` | `str` | Reformatted dates using `fmt` |
| `{col}_is_holiday` | `bool` | `True` if an NSE holiday or weekend |
| `{col}_is_weekend` | `bool` | `True` if a Saturday or Sunday |

```python
import pandas as pd
import nse_calendar as nse

df = pd.DataFrame({"trade_date": ["2026-01-26", "25-05-2026", "2026-05-23"]})
out = nse.resolve_dataframe(df, "trade_date", fmt="dd-mm-yyyy")

```

---

## 🔄 Data Control & Management

The library uses a smart **two-layer data loading strategy**:

1. **Local Cache:** Stored at `~/.cache/nse_calendar/holidays.json` (Valid for 30 days).
2. **Remote Source:** Fetched from GitHub if the cache expires or is missing.

```python
nse.refresh_holidays(force=True)  # Manually force a fresh fetch from remote
nse.get_data_source_info()        # View active cache metadata status
```

## 🤝 Contributing

Spot a missing holiday or a wrong date? Please open an issue and include the official **NSE circular reference** so the dataset can be updated for everyone!

## 📄 License

MIT License. See [LICENSE](https://www.google.com/search?q=LICENSE) for details.

---

Maintained with ❤️ by [Kartik Kasat*](https://github.com/kartikkasat)
