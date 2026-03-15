---
title: Types of Searches
impact: CRITICAL
tags: basics, raw-event, transforming, sparse, dense, search-type
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/types-of-searches"
---

## Types of Searches

Understanding search types helps predict result behavior and performance.

**Incorrect:**
```spl
index=web_logs
| head 10
| stats count by status
```
`head 10` before `stats` gives you statistics over only 10 events —
almost never the intended result.

**Correct (pre-filter with time range to reduce data):**
```spl
index=web_logs earliest=-1h
| stats count by status
```

**Correct (limit report display rows — `head` belongs after `stats`):**
```spl
index=web_logs
| stats count by status
| head 10
```

**Notes:**
- **Raw event search:** Returns individual events from `_raw`. Commands:
  `search`, `where`, `eval`, `rex`, `fields`.
- **Transforming search:** Aggregates events into statistical results.
  Commands: `stats`, `chart`, `timechart`, `top`, `rare`, `geostats`.
  Once a transforming command runs, you cannot return to raw events.
- **Sparse vs Dense:** Sparse fields exist in few events (use `| search field=*`
  to filter); Dense fields exist in most events.
- Transforming commands mark the end of the "streaming" phase — data
  must be fully transferred to the search head before they execute.
