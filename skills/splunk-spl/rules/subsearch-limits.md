---
title: Subsearch Limits and limits.conf
impact: HIGH
tags: subsearch, limits, maxout, maxtime, limits.conf, performance
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-subsearches"
---

## Subsearch Limits and limits.conf

Subsearches have hard resource limits. Exceeding them silently truncates
results — the outer search runs on an incomplete filter.

**Incorrect:**
```spl
index=web_logs [
  index=blocklist
  | table ip
]
```
If blocklist has more than 10,000 IPs, results are silently truncated to
`maxout=10000`. The outer search misses blocked IPs beyond this limit.

**Correct (for large filter sets — use lookup instead):**
```spl
index=web_logs
| lookup blocklist.csv ip OUTPUT is_blocked
| where is_blocked=true
```
Lookups have no result-count limit and are significantly faster.

**Notes:**
- **`limits.conf [subsearch]`:**
  - `maxout = 10000` — max results returned by a subsearch (default).
  - `maxtime = 60` — max seconds a subsearch can run (default).
  - `ttl = 300` — seconds subsearch results are cached (default).
- **`limits.conf [search]`:**
  - `max_subsearch_depth = 8` — max nesting depth of subsearches.
- When subsearch hits `maxout`, no warning is shown in the UI — results
  are quietly truncated.
- **Alternatives for large datasets:**
  - `| lookup` — no size limit, O(1) hash lookup.
  - `| tstats` with `WHERE` — indexed fields, no subsearch needed.
  - `| inputlookup` as outer search — avoids subsearch entirely.
- See: `subsearch-vs-join-lookup.md` for decision guide.
