---
title: tscollect and Summary tsidx Files
impact: MEDIUM
tags: tscollect, tstats, tsidx, summary, re-query
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/tscollect"
---

## tscollect and Summary tsidx Files

`| tscollect` writes indexed field summaries (tsidx files) for later
`| tstats` re-query — faster than `| collect` for aggregation workloads.

**Write a summary:**
```spl
index=web_logs earliest=-1h latest=now
| stats count by host, status, _time span=5m
| tscollect index=summary_tsidx
```

**Re-query the summary with tstats:**
```spl
| tstats sum(count) WHERE index=summary_tsidx BY host, status span=1h
```

**Notes:**
- `tscollect` stores data as tsidx (time-series index) files — queryable
  only via `tstats`, not `search`.
- Fields written must be numeric or string — complex types not supported.
- `namespace=N` — namespace ID for managing summary tsidx files (default 0).
- **Use case:** Pre-aggregate high-volume data at 5-minute granularity,
  then query the summary at 1-hour granularity — orders of magnitude faster.
- Contrast with `collect` (writes to event index, queryable via `search`).
- Summary tsidx files are stored in `$SPLUNK_HOME/var/lib/splunk/<index>/db/`.
