---
title: When to Avoid join
impact: HIGH
tags: join, performance, anti-pattern, lookup, append
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-j/join"
---

## When to Avoid join

`join` is memory-intensive, not parallelizable, and limited by
`subsearch_maxout`. Use alternatives whenever possible.

**Incorrect (join for enrichment):**
```spl
index=web_logs
| join type=left status [
  | inputlookup http_status_codes.csv
]
```

**Correct (lookup is O(1) vs join's O(n*m)):**
```spl
index=web_logs
| lookup http_status_codes.csv status OUTPUT description, category
```

**Incorrect (join for self-aggregation):**
```spl
index=web_logs
| join host [
  search index=web_logs
  | stats avg(response_time) as avg_rt by host
]
```

**Correct (stats replaces the self-join):**
```spl
index=web_logs
| eventstats avg(response_time) as avg_rt by host
```

**Notes:**
- **Why `join` is slow:**
  - The subsearch runs separately and is fully materialized in memory.
  - Limited to `subsearch_maxout=50000` rows (from `limits.conf [join]`).
  - Cannot use parallel reduce — all processing is on the search head.
- **`| append` + `| stats`** for union-type joins:
  ```spl
  index=source1 | append [search index=source2] | dedup id
  ```
- **`| appendcols`** for column-binding (ZIP join):
  ```spl
  index=A | appendcols [search index=B | fields field_x]
  ```
- See: `subsearch-vs-join-lookup.md` for the decision guide.
