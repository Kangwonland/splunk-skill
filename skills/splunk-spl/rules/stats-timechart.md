---
title: timechart Patterns and Configuration
impact: HIGH
tags: stats, timechart, span, limit, time-series, visualization
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/statistical-and-charting-functions/about-statistical-and-charting-functions"
---

## timechart Patterns and Configuration

`timechart` creates time-series data. Misconfiguring `span=` and `limit=`
causes misleading charts and truncated data.

**Incorrect:**
```spl
index=web_logs
| timechart count by status
```
Without `span=`, Splunk auto-selects a span that may be too coarse. With
many `status` values, extra values are collapsed into "OTHER".

**Correct:**
```spl
index=web_logs
| timechart span=1h count by status limit=0
```
`limit=0` disables the "OTHER" bucket — all values are shown.

**Notes:**
- `span=` sets the time bucket size: `span=1m`, `span=1h`, `span=1d`, `span=1w`.
- `limit=N` — max number of `by` field values shown (default: 10). Values
  beyond the limit are grouped into "OTHER". Set `limit=0` to show all.
- `useother=false` — hide the "OTHER" bucket entirely when `limit` is set.
- `usenull=false` — hide null values in the series.
- `timechart` always outputs `_time` as the first column — cannot be removed.
- `per_second(field)` / `per_minute(field)` — rate functions:
  `| timechart per_minute(bytes) as bytes_per_min`
- `cont=false` — skip time buckets with no events (gaps instead of zeros).
