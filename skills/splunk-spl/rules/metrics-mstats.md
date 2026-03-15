---
title: Metrics Index and mstats Command
impact: HIGH
tags: metrics, mstats, mcollect, mpreview, metric-index
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-m/mstats"
---

## Metrics Index and mstats Command

Splunk metrics indexes store numeric time series data more efficiently
than event indexes. Query them with `mstats`.

**Query metrics with mstats:**
```spl
| mstats avg(cpu.percent) WHERE index=metrics BY host span=1m
```
Directly queries metric data points. `cpu.percent` is the metric name.

**Multiple metrics:**
```spl
| mstats avg(cpu.percent) AS cpu, avg(mem.percent) AS mem
  WHERE index=metrics BY host span=5m
  latest=-1h earliest=-2h
```

**Write to metrics index:**
```spl
index=web_logs
| stats avg(response_time) AS response_time BY host _time
| eval metric_name="web.response_time"
| mcollect index=mymetrics prefix_field=metric_name
```

**Inspect raw metric data:**
```spl
| mpreview index=metrics | head 10
```

**Notes:**
- Metric index vs event index: metrics store only `_time`, dimensions (string
  fields), and metric values (numbers). No `_raw` field.
- `mstats` aggregation functions: `avg`, `min`, `max`, `sum`, `count`, `stdev`, `perc`.
- `WHERE` clause: filter by metric dimensions (indexed fields captured at
  collection time via `mcollect`/`collectd`), not aggregated metric values.
  Use `| where` after `mstats` to filter on computed metric values.
- `fillnull_value=0` — fill missing time buckets with 0 instead of null.
- `mcollect` batch size: `batch_size=1000` (default, `limits.conf [metrics]`).
