# nse-calendar

[![PyPI version](https://badge.fury.io/py/nse-calendar.svg)](https://badge.fury.io/py/nse-calendar)
[![Python versions](https://img.shields.io/pypi/pyversions/nse-calendar.svg)](https://pypi.org/project/nse-calendar/)
[![Data Sync](https://github.com/kartikkasat/nse-calendar-data/actions/workflows/sync-sheet.yml/badge.svg)](https://github.com/kartikkasat/nse-calendar-data/actions/workflows/sync-sheet.yml)

**NSE (National Stock Exchange of India) trading calendar for Python.**

Holiday detection, trading day arithmetic, flexible date parsing and formatting, and optional Pandas integration. Zero mandatory dependencies. Works offline after first use.

---
## 📅 Data Coverage

| Year Range | Status | Source |
|------------|--------|--------|
| 1990 – 2009 | ✅ Available | NSE historical archives |
| 2010 – 2025 | ✅ Available | NSE official annual circulars |
| 2026 | ✅ Available | NSE official annual circular |
| 2027 and beyond | 🔄 Added each December | NSE official annual circular |

NSE publishes the following year's holiday calendar every December. The dataset is updated automatically within 24 hours of publication.
---

## 📦 Installation

```bash
pip install nse-calendar
```

---

## ⚡ Quick Start

```python
import nse_holidays as nse

# Is this a trading day?
nse.is_nse_holiday("2026-01-26")           # True  — Republic Day
nse.is_nse_holiday("2026-05-25")           # False — normal Monday

# Move forward by trading days
nse.add_trading_days("2026-01-23", 1)      # date(2026, 1, 27)  — skips weekend + Republic Day

# Count trading days in a range
nse.trading_days_between("2026-01-01", "2026-01-31")   # 20

# Parse any format, output any format
d = nse.parse_date("25-May-2026")
nse.format_date(d, "dd/mm/yyyy")           # '25/05/2026'
nse.format_date(d, "dd Month yyyy")        # '25 May 2026'
```

---

## 📚 API Reference

### Holiday Checks

#### `is_nse_holiday(date) -> bool`

Returns `True` if the date is an NSE trading holiday **or** a weekend. Returns `False` for normal trading days.

```python
nse.is_nse_holiday("2026-01-26")    # True  — Republic Day
nse.is_nse_holiday("2026-05-23")    # True  — Saturday
nse.is_nse_holiday("2026-05-25")    # False — Monday, normal session
nse.is_nse_holiday(datetime.date(2026, 12, 25))  # True — Christmas
```

---

#### `get_holidays(year) -> list[dict]`

Returns all NSE holidays for a given year as a list of `{"date": "YYYY-MM-DD", "description": "..."}` dicts. Returns an empty list if the year is not in the dataset.

```python
nse.get_holidays(2026)
# [
#   {"date": "2026-01-26", "description": "Republic Day"},
#   {"date": "2026-03-03", "description": "Maha Shivaratri"},
#   ...
# ]

len(nse.get_holidays(2026))     # 16
nse.get_holidays(1995)          # []  — not in dataset
```

---

### Trading Day Arithmetic

#### `add_trading_days(date, n) -> datetime.date`

Advances a date by exactly `n` trading days, skipping weekends and NSE holidays. Pass a negative `n` to go backwards (equivalent to `subtract_trading_days`).

```python
# Friday 23-Jan + 1 → skips Saturday, Sunday, Republic Day (Mon 26-Jan) → Tuesday 27-Jan
nse.add_trading_days("2026-01-23", 1)     # date(2026, 1, 27)

# Normal week, no holidays in the way
nse.add_trading_days("2026-05-25", 5)     # date(2026, 6, 1)

# Negative n goes backward
nse.add_trading_days("2026-05-25", -3)    # date(2026, 5, 20)
```

---

#### `subtract_trading_days(date, n) -> datetime.date`

Moves a date back by exactly `n` trading days, skipping weekends and NSE holidays.

```python
nse.subtract_trading_days("2026-01-27", 1)    # date(2026, 1, 23) — skips Republic Day + weekend
nse.subtract_trading_days("2026-06-01", 5)    # date(2026, 5, 25)
```

---

#### `trading_days_between(start, end, inclusive="both") -> int`

Counts trading days between two dates. Order of `start` / `end` does not matter — reversed inputs return the same count.

`inclusive` controls whether boundary dates are counted:

| `inclusive` | Counts `start`? | Counts `end`? |
|-------------|-----------------|---------------|
| `"both"` (default) | ✅ | ✅ |
| `"left"` | ✅ | ❌ |
| `"right"` | ❌ | ✅ |
| `"neither"` | ❌ | ❌ |

```python
# Monday to Friday — 5 trading days inclusive
nse.trading_days_between("2026-06-01", "2026-06-05")                    # 5
nse.trading_days_between("2026-06-01", "2026-06-05", inclusive="neither")  # 3

# Reversed inputs — same result
nse.trading_days_between("2026-06-05", "2026-06-01")                    # 5

# Spanning a holiday (Republic Day, 26-Jan)
nse.trading_days_between("2026-01-22", "2026-01-30")                    # 5  (not 7)
```

---

#### `next_trading_day(date) -> datetime.date`

Returns the **next** trading day strictly after the given date. Always moves forward at least one calendar day.

```python
nse.next_trading_day("2026-01-26")    # date(2026, 1, 27) — skips Republic Day
nse.next_trading_day("2026-05-22")    # date(2026, 5, 25) — skips Sat + Sun
nse.next_trading_day("2026-05-25")    # date(2026, 5, 26) — already a trading day, still moves forward
```

---

#### `prev_trading_day(date) -> datetime.date`

Returns the **previous** trading day strictly before the given date. Always moves back at least one calendar day.

```python
nse.prev_trading_day("2026-01-27")    # date(2026, 1, 23) — skips Republic Day + weekend
nse.prev_trading_day("2026-05-25")    # date(2026, 5, 22) — skips Sat + Sun
```

---

#### `nearest_trading_day(date, direction="forward") -> datetime.date`

If the date is already a trading day, returns it unchanged. Otherwise snaps to the nearest trading day.

`direction` controls which way to snap when the date is not a trading day:
- `"forward"` (default) — snap to the next trading day
- `"backward"` — snap to the previous trading day

```python
# Already a trading day — returned as-is
nse.nearest_trading_day("2026-05-25")                          # date(2026, 5, 25)

# Republic Day — snap forward
nse.nearest_trading_day("2026-01-26")                          # date(2026, 1, 27)

# Republic Day — snap backward
nse.nearest_trading_day("2026-01-26", direction="backward")    # date(2026, 1, 23)

# Saturday — snap forward
nse.nearest_trading_day("2026-05-23")                          # date(2026, 5, 25)
```

---

### Date Parsing

#### `parse_date(value) -> datetime.date`

Converts virtually any date representation to a `datetime.date`. Accepts strings in all common formats, `datetime.date`, `datetime.datetime`, Unix timestamps, and `pandas.Timestamp`.

**Supported string formats:**

| Input | Parsed as |
|-------|-----------|
| `"2026-05-25"` | 25 May 2026 |
| `"25-05-2026"` | 25 May 2026 |
| `"25/05/2026"` | 25 May 2026 |
| `"25.05.2026"` | 25 May 2026 |
| `"25-May-2026"` | 25 May 2026 |
| `"25 May 2026"` | 25 May 2026 |
| `"25 may 2026"` | 25 May 2026 |
| `"25-MAY-2026"` | 25 May 2026 |
| `"25-January-2026"` | 25 Jan 2026 |
| `"25-May-26"` | 25 May 2026 |
| `"20260525"` | 25 May 2026 |
| `"May 25, 2026"` | 25 May 2026 |

```python
nse.parse_date("25-May-2026")              # date(2026, 5, 25)
nse.parse_date("25/05/2026")              # date(2026, 5, 25)
nse.parse_date(datetime.date(2026, 5, 25)) # date(2026, 5, 25)  — passthrough
nse.parse_date(1748131200)                # date from Unix timestamp
```

Raises `ValueError` if the input cannot be parsed.

---

### Date Formatting

#### `format_date(date, fmt) -> str`

Formats a date using a named style token or a standard `strftime` pattern (any format starting with `%`).

**Named format tokens:**

| Format string | Output |
|---------------|--------|
| `"dd-mm-yyyy"` | `25-05-2026` |
| `"dd/mm/yyyy"` | `25/05/2026` |
| `"dd-month-yyyy"` | `25-May-2026` |
| `"dd mon yyyy"` | `25 May 2026` |
| `"dd Month yyyy"` | `25 May 2026` |
| `"dd-Mon-yyyy"` | `25-May-2026` |
| `"dd Mon yy"` | `25 May 26` |
| `"dd-MM-yy"` | `25-05-26` |

Any `strftime` pattern also works:

```python
nse.format_date("2026-05-25", "dd-mm-yyyy")          # '25-05-2026'
nse.format_date("2026-05-25", "dd Month yyyy")        # '25 May 2026'
nse.format_date("2026-05-25", "%A, %d %B %Y")         # 'Monday, 25 May 2026'
nse.format_date("2026-03-20", "dd Month yyyy")        # '20 March 2026'
```

---

### Pandas Integration

#### `resolve_dataframe(df, col, output_col=None, fmt="dd-mm-yyyy") -> pd.DataFrame`

Parses a mixed-format date column in a DataFrame, formats each value uniformly, and appends two boolean flag columns. Returns a copy — the original DataFrame is not modified.

| Added column | Type | Description |
|---|---|---|
| `{col}` (or `output_col`) | str | Dates reformatted to `fmt` |
| `{col}_is_holiday` | bool | `True` if NSE holiday or weekend |
| `{col}_is_weekend` | bool | `True` if Saturday or Sunday |

```python
import pandas as pd
import nse_holidays as nse

df = pd.DataFrame({
    "trade_date": ["2026-01-26", "25-05-2026", "2026-05-23", "01-Jan-2026"]
})

out = nse.resolve_dataframe(df, "trade_date", fmt="dd-mm-yyyy")

#   trade_date  trade_date_is_holiday  trade_date_is_weekend
#   26-01-2026               True                  False     ← Republic Day
#   25-05-2026              False                  False     ← Normal Monday
#   23-05-2026               True                   True     ← Saturday
#   01-01-2026              False                  False     ← Normal Thursday
```

Use `output_col` to write to a different column instead of overwriting:

```python
out = nse.resolve_dataframe(df, "trade_date", output_col="formatted_date", fmt="dd/mm/yyyy")
```

---

### Data Control

#### `refresh_holidays(force=False) -> None`

Forces a fresh fetch from GitHub, bypassing the local cache. Use this when you know new holiday data has been published and don't want to wait for the 30-day automatic refresh.

```python
nse.refresh_holidays()           # fetch only if cache is stale
nse.refresh_holidays(force=True) # always fetch from remote
```

---

#### `get_data_source_info() -> dict`

Returns metadata about the current data state — useful for debugging or logging in production systems.

```python
nse.get_data_source_info()
# {
#   "source":       "cache",          # "cache" | "remote" | "none"
#   "last_fetched": "2026-06-05T09:14:22.431",
#   "cache_path":   "/home/user/.cache/nse_holidays/holidays.json"
# }
```

`source` values:

| Value | Meaning |
|-------|---------|
| `"cache"` | Loaded from local `~/.cache/nse_holidays/holidays.json` |
| `"remote"` | Fetched fresh from GitHub |
| `"none"` | No data available — all calls return safe defaults |

---

## 🌐 How Data Works

1. **On first use**, the library downloads `holidays.json` from the public GitHub data repository.
2. **Cache** lives at `~/.cache/nse_holidays/holidays.json` and is considered fresh for 30 days.
3. **In December**, the library automatically checks if next year's data is present. If not, it refreshes regardless of cache age — so January 1 always has correct data.
4. **If GitHub is unreachable** and no cache exists, all holiday checks return `False` and a `RuntimeWarning` is emitted. The library never raises an exception due to a data fetch failure.

Raw JSON URL (also usable directly):
```
https://raw.githubusercontent.com/kartikkasat/nse-calendar-data/main/holidays.json
```

---

## 🤝 Contributing

To report a missing or incorrect holiday, open an issue and include the relevant NSE circular reference. A maintainer will update the source data and it will propagate automatically.

---

## 📜 License

MIT — see [LICENSE](LICENSE).

---

Maintained by [Kartik Kasat](https://github.com/kartikkasat).
