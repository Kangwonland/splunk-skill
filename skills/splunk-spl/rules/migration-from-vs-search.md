---
title: SPL2 from vs SPL1 search — Equivalence Guide
impact: HIGH
tags: migration, from, search, equivalence, index, datamodel
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL2 from vs SPL1 search — Equivalence Guide

Mapping between SPL1 `search index=` patterns and SPL2 `from` equivalents.

| SPL1 | SPL2 |
|------|------|
| `search index=main` | `from main` |
| `search index=web_logs sourcetype=access_combined` | `from main \| where sourcetype="access_combined"` |
| `search index=web_logs host=web01` | `from main \| where host="web01"` |
| `search index=_internal` | `from _internal` |
| `\| datamodel Authentication Authentication search` | `from datamodel:Authentication.Authentication` |
| `\| inputlookup geo_data.csv` | `from lookup:geo_data` |

**Time range in SPL2:**
```spl2
from main
| where _time > relative_time(now(), "-24h") AND _time <= now()
| stats count() AS count BY host
```

**Multiple indexes (SPL1):**
```spl
index=web_logs OR index=app_logs status=500
```

**Multiple datasets (SPL2):**
```spl2
from main, app_logs
| where status=500
```

**Notes:**
- SPL2 `from <name>` uses the index name directly (no `index=` prefix).
- `sourcetype`, `host`, `source` filtering moves to `| where` clause.
- SPL2 does not support leading wildcard index names: `from web_*` is not valid.
- Time range: SPL2 searches use the UI time picker OR explicit `| where _time` filters.
