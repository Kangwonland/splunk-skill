---
title: Use fields Command to Drop Unused Fields
impact: CRITICAL
tags: performance, fields, field-list, optimization
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches"
---

## Use fields Command to Drop Unused Fields

Each field extracted and carried through the pipeline consumes memory and
network bandwidth. Drop unused fields as early as possible.

**Incorrect:**
```spl
index=web_logs
| eval duration_ms=response_time*1000
| stats avg(duration_ms) by status
```
All extracted fields (uri, user_agent, referer, etc.) are carried through
the pipeline even though only `response_time` and `status` are needed.

**Correct:**
```spl
index=web_logs
| fields response_time status
| eval duration_ms=response_time*1000
| stats avg(duration_ms) by status
```
`| fields` is a distributable streaming command — it runs on indexers,
reducing data transferred to the search head.

**Notes:**
- `| fields field1 field2` — keep only listed fields (allowlist)
- `| fields - field1 field2` — remove listed fields (denylist)
- Place `| fields` immediately after the search string for maximum effect.
- `_raw` and `_time` are always retained unless explicitly removed.
- Removing `_raw` with `| fields - _raw` significantly reduces memory
  for large result sets: `index=web_logs | fields - _raw | stats count by status`
