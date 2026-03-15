---
title: Time Eval Functions
impact: HIGH
tags: eval, time, strftime, strptime, relative_time, now, toepoch
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/evaluation-functions/about-evaluation-functions"
---

## Time Eval Functions

Use time eval functions to format, parse, and calculate with timestamps.
Mixing epoch and formatted times without conversion causes type errors.

**Incorrect:**
```spl
index=web_logs
| eval age=now() - timestamp
```
If `timestamp` is a formatted string (not epoch), subtraction fails.

**Correct:**
```spl
index=web_logs
| eval epoch_ts=strptime(timestamp, "%Y-%m-%d %H:%M:%S")
| eval age_seconds=now() - epoch_ts
| eval age_human=tostring(age_seconds/3600, "duration")
```

**Notes:**
- `now()` — current time as Unix epoch (integer seconds).
- `_time` — event timestamp, already in epoch format.
- `strftime(_time, "%Y-%m-%d %H:%M:%S")` — epoch → formatted string.
- `strptime("2024-01-15 10:30:00", "%Y-%m-%d %H:%M:%S")` — string → epoch.
- `relative_time(now(), "-1d@d")` — apply a relative time modifier to an epoch.
- `toepoch("01/15/2024:10:30:00")` — parse common date formats to epoch.
- `fromepoch(epoch)` — epoch → human-readable string (uses local timezone).
- See: `references/time-format-variables.md` for `strftime`/`strptime` format codes.
