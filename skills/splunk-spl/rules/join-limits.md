---
title: join Command Limits
impact: MEDIUM
tags: join, limits, subsearch_maxout, memory, limits.conf
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-j/join"
---

## join Command Limits

`join` uses subsearch internally and is subject to both subsearch and
join-specific limits. Exceeding them silently truncates results.

**Incorrect (assuming all rows are matched):**
```spl
index=web_logs
| join type=left session_id [
  search index=sessions
  | table session_id, user_id
]
```
If the session index has more than 50,000 unique session IDs, the subsearch
result is truncated to `subsearch_maxout=50000`.

**Correct (for large lookups — use lookup file):**
```spl
index=web_logs
| lookup sessions.csv session_id OUTPUT user_id
```

**Notes:**
- **`limits.conf [join]`:**
  - `subsearch_maxout = 50000` — max rows from join subsearch (default).
  - `subsearch_maxtime = 60` — max seconds for join subsearch (default).
- **`limits.conf [subsearch]`:**
  - `maxout = 10000` — applies to `[ ]` subsearches, not `| join` subsearches.
- **Memory impact:** Both datasets must fit in search head memory simultaneously.
  Large joins on high-cardinality fields can exhaust memory.
- Monitor join memory with: `index=_audit action=search | search info_search_et=* | table info_*`
