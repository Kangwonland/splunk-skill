---
title: join Command Patterns
impact: HIGH
tags: join, inner-join, left-join, outer-join, dataset-combination
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-j/join"
---

## join Command Patterns

`join` combines the main search results with a subsearch result on a common
field. Understand join types to avoid missing events.

**Incorrect (assuming all events are returned):**
```spl
index=web_logs
| join host [
  search index=inventory
  | table host, datacenter
]
```
Default `join` is an inner join — events without a matching host in
inventory are dropped silently.

**Correct (left join to preserve all events):**
```spl
index=web_logs
| join type=left host [
  search index=inventory
  | table host, datacenter
]
```
Left join retains all outer search events; `datacenter` is null for
hosts not found in inventory.

**Notes:**
- `type=inner` (default) — only events with a match in both datasets.
- `type=left` — all outer events; inner events matched where possible.
- `type=outer` — alias for left join in Splunk (full outer join not supported).
- **`overwrite=true`** (default) — subsearch fields overwrite outer fields
  with the same name. Set `overwrite=false` to keep outer values.
- `max=N` — max results from the subsearch per outer event (default: 1).
  `max=0` returns all matches (may multiply row count).
- Prefer `| lookup` for static datasets; see `join-avoid.md`.
