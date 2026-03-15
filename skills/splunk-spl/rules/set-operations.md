---
title: Set Operations — union, diff, intersect
impact: MEDIUM
tags: set, union, diff, intersect, append, dedup
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-s/set"
---

## Set Operations — union, diff, intersect

`| set` performs set operations on two result sets. Results from a
subsearch are combined with the current pipeline using set logic.

**union (all events from both):**
```spl
index=web_logs status=500
| set union [search index=app_logs level=ERROR]
```
Returns all events from both searches (removes duplicates).

**diff (events in main but not in subsearch):**
```spl
index=web_logs
| stats count by host
| set diff [search index=maintenance | table host]
```
Returns hosts in web_logs not in maintenance list.

**intersect (events in both):**
```spl
index=web_logs
| stats count by host
| set intersect [search index=monitored | table host]
```
Returns only hosts present in both datasets.

**Notes:**
- `| set` requires both datasets to have the same fields.
- **`| append` + `| dedup` as alternative to set union:**
  ```spl
  index=web_logs | append [search index=app_logs] | dedup _raw
  ```
- **`selfjoin` pattern:**
  `index=web_logs | selfjoin session_id` — joins each row with other rows
  sharing the same `session_id`. Rarely needed; use `stats` instead.
- Set operations are non-distributable — run on the search head.
- Field ordering must match between both datasets for correct operation.
