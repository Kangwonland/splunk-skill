---
title: tstats Command Basics
impact: CRITICAL
tags: tstats, indexed-fields, performance, tsidx, stats-comparison
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/tstats"
---

## tstats Command Basics

`tstats` queries indexed fields directly from tsidx files without loading
raw events — the fastest aggregation method in Splunk.

**Incorrect (using stats on large dataset when tstats would work):**
```spl
index=web_logs sourcetype=access_combined
| stats count by host, status
```

**Correct (tstats for indexed fields):**
```spl
| tstats count WHERE index=web_logs sourcetype=access_combined BY host, status
```
Runs directly on index metadata — no event loading, no field extraction.

**Notes:**
- **Indexed fields only:** `tstats` works on `index`, `host`, `source`,
  `sourcetype`, `_time`, and fields in `fields.conf` with `INDEXED=true`.
  Search-time extracted fields (from `props.conf`/`transforms.conf`) are NOT available.
- **tstats vs stats comparison:**
  | Feature | tstats | stats |
  |---------|--------|-------|
  | Field scope | Indexed only | All extracted |
  | Speed | Very fast | Depends on volume |
  | Parallelism | Full | Parallel reduce |
  | Data models | Yes (summariesonly) | No |
- `fillnull_value="NULL"` — replace null field values with a string.
- `local=true` — run locally on search head (debug mode, skips indexers).
- `prestats=true` — emit pre-aggregation rows for `append` patterns.
