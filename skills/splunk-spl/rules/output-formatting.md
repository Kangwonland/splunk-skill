---
title: Output Formatting Commands
impact: LOW
tags: output, fieldformat, reltime, rangemap, gauge
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-f/fieldformat"
---

## Output Formatting Commands

`fieldformat`, `reltime`, `rangemap`, and `gauge` format values for
display without altering underlying data.

**fieldformat (display-only formatting):**
```spl
index=web_logs
| stats sum(bytes) as total_bytes by host
| fieldformat total_bytes = tostring(total_bytes, "commas")
```
`total_bytes` displays as `1,234,567` in the UI but retains numeric
value for further calculations.

**reltime (human-readable time):**
```spl
index=web_logs
| head 100
| reltime
| table _time, reltime, host
```
Adds `reltime` field: "5 minutes ago", "2 hours ago".

**rangemap (color-coded ranges):**
```spl
index=metrics
| stats avg(response_time) as avg_rt by host
| rangemap field=avg_rt low=0-200 elevated=201-500 high=501-1000 default=severe
```
Adds `range` field with values: `low`, `elevated`, `high`, `severe`.

**gauge:**
```spl
index=metrics
| stats avg(cpu_pct) as cpu by host
| gauge cpu 0 50 80 100
```

**Notes:**
- `fieldformat` only affects display — does NOT change the value for `where` or `eval`.
- `reltime` uses `_time` field; ensure `_time` is present before calling.
- `rangemap default=` — the range applied when value falls outside all defined ranges.
