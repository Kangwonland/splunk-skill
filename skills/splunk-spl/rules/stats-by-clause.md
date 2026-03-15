---
title: stats BY Clause and High-Cardinality Pitfalls
impact: HIGH
tags: stats, by-clause, cardinality, grouping, performance
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/statistical-and-charting-functions/about-statistical-and-charting-functions"
---

## stats BY Clause and High-Cardinality Pitfalls

High-cardinality `by` fields (user IDs, session IDs, IPs) cause `stats`
to produce millions of rows, overwhelming search head memory.

**Incorrect:**
```spl
index=web_logs
| stats count, values(uri) as uris by session_id
```
If there are 10M unique sessions, this produces 10M rows with potentially
large multivalue `uris` fields — likely to hit memory limits.

**Correct:**
```spl
index=web_logs
| stats count by session_id
| where count > 100
| sort -count
| head 1000
```
Filter immediately after `stats` to reduce the result set.

**Notes:**
- `maxresultrows` in `limits.conf [searchresults]` caps result rows (default: 50000).
  Exceeding this silently truncates results.
- **High-cardinality alternatives:**
  - `dc(field)` — distinct count without enumerating all values.
  - `estdc(field)` — estimated distinct count (faster, less memory).
  - `top N field` — top N values with counts (built-in limit).
- `values(field)` accumulates all distinct values — memory grows with cardinality.
  Use `list(field)` to preserve order but limit with `| head`.
- For session analysis, prefer `transaction` or `streamstats` over
  `stats values()` to avoid unbounded memory growth.
