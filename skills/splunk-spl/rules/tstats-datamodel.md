---
title: tstats with Data Models
impact: HIGH
tags: tstats, datamodel, summariesonly, nodename, WHERE-clause
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/tstats"
---

## tstats with Data Models

`tstats` on accelerated data models is the fastest way to query structured
event data at scale.

**Basic data model query:**
```spl
| tstats summariesonly=true count FROM datamodel=Authentication.Authentication
  WHERE Authentication.action=failure
  BY Authentication.user, Authentication.src span=1h
```

**Target a child dataset with nodename:**
```spl
| tstats summariesonly=true count FROM datamodel=Authentication
  WHERE nodename=Authentication.Failed_Authentication
  BY Authentication.user span=1d
```

**Notes:**
- `FROM datamodel=ModelName.DatasetName` — dot notation targets the dataset.
  `FROM datamodel=ModelName` with `WHERE nodename=` is equivalent.
- `summariesonly=true` — only accelerated data (fast, may miss recent gaps).
- `summariesonly=false` — accelerated + live data (complete, slightly slower).
- `allow_old_summaries=true` — use summaries after schema changes.
- **WHERE clause:** Only indexed fields and data model attributes can filter.
  Search-time extractions are not available in tstats WHERE.
- `span=` — time grouping granularity for `BY _time span=Xm`.
- Results use prefixed field names: `Authentication.user` (not `user`).
  Rename with `| rename Authentication.user AS user`.
