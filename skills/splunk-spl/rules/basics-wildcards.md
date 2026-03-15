---
title: Wildcard Usage and Pitfalls
impact: CRITICAL
tags: basics, wildcard, performance, leading-wildcard
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## Wildcard Usage and Pitfalls

Never use a leading wildcard (`*value`) in search terms — it forces a
full index scan.

**Incorrect:**
```spl
index=web_logs uri=*admin*
```
Leading `*` cannot use the Bloom filter or index optimizations. Splunk
scans every event.

**Correct:**
```spl
index=web_logs uri=admin*
```
Trailing wildcards can use index lookup structures. Or use `rex` for
mid-string matching:

```spl
index=web_logs | rex field=uri "admin" | where isnotnull(uri)
```

**Notes:**
- `min_prefix_len=1` in `limits.conf [search]` sets the minimum prefix
  length before a wildcard for index queries.
- For exact phrase matching across segmentation boundaries, prefer
  `TERM()` (see `basics-case-term.md`).
- Wildcards in field **names** (not values) are supported by many commands
  (e.g., `fields host*`, `foreach www*`).
