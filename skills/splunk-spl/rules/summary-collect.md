---
title: Summary Indexing with collect
impact: HIGH
tags: summary, collect, sourcetype=stash, license, addinfo, tscollect
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/reports-and-alerts/create-and-manage-reports/use-summary-indexing-to-improve-report-performance"
---

## Summary Indexing with collect

`| collect` writes search results back into a Splunk index for fast
re-query. Use `sourcetype=stash` to avoid license consumption.

**Correct (scheduled search writing to summary index):**
```spl
index=web_logs earliest=-15m latest=now
| stats count avg(response_time) as avg_rt by host status
| collect index=summary sourcetype=stash addinfo=true
```
Runs every 15 minutes. `sourcetype=stash` is exempt from license metering.

**Re-query the summary:**
```spl
index=summary sourcetype=stash earliest=-24h
| stats sum(count) as total_count avg(avg_rt) as overall_avg by host status
```

**Notes:**
- `addinfo=true` — stamps `info_min_time` and `info_max_time` fields
  on each written event (the time range of the originating search).
- `testmode=true` — dry run: shows what would be written without writing.
- `output_format=hec` — write in HEC JSON format for HEC-compatible indexes.
- `marker=key=value` — add custom metadata field to all collected events.
- `run_in_preview=false` — prevent collect from running in preview mode.
- `sistats → collect` pattern: `| sistats count by host | collect index=summary`
  uses sistats for partial aggregation then collect for storage.
- See: `summary-tscollect.md` for tsidx-based summary pattern.
