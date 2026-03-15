---
title: Index Time vs Event Time
impact: HIGH
tags: time, index-time, event-time, _indextime, earliest, index_earliest
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/specify-time-ranges/about-searching-with-time"
---

## Index Time vs Event Time

`earliest`/`latest` filter by event time (`_time`). Use
`index_earliest`/`index_latest` when you need to filter by when events
were *indexed*, not when they occurred.

**Incorrect:**
```spl
index=web_logs earliest=-1h latest=now
```
If events arrive late (network delays, batch ingestion), events that
happened within the last hour but arrived later are missed.

**Correct (catch late-arriving events):**
```spl
index=web_logs index_earliest=-2h index_latest=now
| where _time >= relative_time(now(), "-1h")
```
Searches buckets indexed in the last 2 hours, then filters by event time.

**Notes:**
- **`_time`:** The event timestamp (when the event occurred). Set by
  `TIME_PREFIX`/`TIME_FORMAT` in props.conf, or defaults to index time.
- **`_indextime`:** When the event was written to the index. Always set
  by Splunk, cannot be spoofed.
- **`earliest`/`latest`:** Filter on `_time`. Most searches use these.
- **`index_earliest`/`index_latest`:** Filter on `_indextime`. Use for:
  - Compliance searches that need "what was indexed when"
  - Finding late-arriving events
  - Re-indexing investigations
- Mixing both: `index_earliest` opens the bucket scope; `earliest`/`latest`
  then filter within those buckets.
