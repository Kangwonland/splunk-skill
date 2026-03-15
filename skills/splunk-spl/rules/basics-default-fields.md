---
title: Default Fields and Indexed vs Search-Time Fields
impact: CRITICAL
tags: basics, default-fields, indexed-fields, _time, _raw, sourcetype
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## Default Fields and Indexed vs Search-Time Fields

Splunk adds several default fields at index time. Understanding which fields
are indexed vs search-time extracted affects search performance.

**Incorrect:**
```spl
index=web_logs | search host::webserver1
```
`host::` (double colon) is for indexed fields only. If `host` has been
overridden by a search-time extraction, this may not match.

**Correct:**
```spl
index=web_logs host=webserver1
```
Use `field=value` for all standard field lookups. Use `field::value` only
when you explicitly need index-time field matching.

**Notes:**
- **Default indexed fields:** `_time`, `_indextime`, `host`, `source`,
  `sourcetype`, `punct`, `linecount`
- **`_raw`:** The raw event text — always available, never indexed (search-time only)
- **`field=v` vs `field::v`:** Use `=` universally; use `::` only to bypass
  search-time extractions for performance on high-volume indexed fields.
- CIDR matching: `src_ip="192.168.1.0/24"` works in field expressions.
- See: `references/spl-default-fields.md` for the full default fields table.
