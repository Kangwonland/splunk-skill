---
title: stats vs eventstats vs streamstats
impact: HIGH
tags: stats, eventstats, streamstats, aggregation, selection-guide
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/statistical-and-charting-functions/about-statistical-and-charting-functions"
---

## stats vs eventstats vs streamstats

Choose the right aggregation command based on whether you need to retain
raw events and whether order matters.

**Incorrect (using stats when original events are needed):**
```spl
index=web_logs
| stats avg(response_time) as avg_rt by host
| where response_time > avg_rt * 2
```
After `stats`, individual `response_time` values no longer exist.

**Correct:**
```spl
index=web_logs
| eventstats avg(response_time) as avg_rt by host
| where response_time > avg_rt * 2
| table _time, host, response_time, avg_rt
```
`eventstats` adds the aggregate as a new field while retaining all original events.

**Notes:**
- **`stats`:** Aggregates events into summary rows. Original events are lost.
  Fastest; runs in parallel reduce mode. Use for final reporting.
- **`eventstats`:** Adds aggregate fields to each original event. Events retained.
  Centralized (search head only) — expensive on large datasets.
- **`streamstats`:** Running/cumulative aggregation in event order.
  `| streamstats sum(bytes) as cumulative_bytes by session_id`
  Use for session analysis, running totals, window functions.
- **`streamstats window=N`:** Sliding window over last N events:
  `| streamstats window=5 avg(response_time) as rolling_avg`
- Decision: `stats` for summaries → `eventstats` for per-event enrichment
  → `streamstats` for ordered/windowed calculations.
