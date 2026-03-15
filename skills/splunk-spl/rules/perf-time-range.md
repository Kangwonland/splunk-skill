---
title: Narrow Time Ranges
impact: CRITICAL
tags: performance, time-range, all-time, optimization
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches"
---

## Narrow Time Ranges

"All Time" searches scan every bucket in an index. Always specify a time
range — even a generous one is far better than All Time.

**Incorrect:**
```spl
index=web_logs status=500
| stats count by host
```
Without `earliest`/`latest`, this is an All Time search — scans all
historical buckets. On production indexes this can run for minutes or hours.

**Correct:**
```spl
index=web_logs status=500 earliest=-24h latest=now
| stats count by host
```

**Notes:**
- Splunk organizes data in time-based buckets. Time range filters allow
  Splunk to skip entire buckets outside the window — the most impactful
  single optimization available.
- **Snap-to alignment** improves bucket skipping: `earliest=-1d@d` aligns
  to bucket boundaries better than `earliest=-24h`.
- For scheduled reports, always use relative time with snap-to:
  `earliest=-1d@d latest=@d` (yesterday).
- Ad-hoc searches: set time range in the UI time picker or pass
  `earliest_time`/`latest_time` in the REST API.
- Avoid `All Time` in scheduled searches — it re-scans the entire index
  on every run and grows slower as data accumulates.
