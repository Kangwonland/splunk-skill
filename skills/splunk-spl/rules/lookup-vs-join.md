---
title: lookup vs join Trade-offs
impact: HIGH
tags: lookup, join, performance, trade-off, selection-guide
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/lookups/about-lookups"
---

## lookup vs join Trade-offs

`lookup` is almost always faster than `join`. Use `join` only when
`lookup` cannot express the required logic.

**Incorrect (join for static enrichment):**
```spl
index=web_logs
| join type=left status_code [
  | inputlookup http_codes.csv
  | rename code as status_code
]
```

**Correct:**
```spl
index=web_logs
| lookup http_codes.csv status_code OUTPUT description, severity
```

**Notes:**
- **`lookup` (O(1) hash):** Lookup key is hashed — constant time regardless
  of lookup table size. Supports automatic lookup (no SPL needed).
- **`join` (O(n*m)):** Cross-product matching — performance degrades with
  both dataset sizes. Limited to `subsearch_maxout=50000` rows.
- **When `join` is unavoidable:**
  - Multiple join keys with complex conditions.
  - You need `max=0` to get all matching rows (one-to-many).
  - The second dataset changes frequently (too dynamic for a lookup file).
- **`lookup` limitations:**
  - Lookup file must fit in memory (configurable via `max_matches` in `transforms.conf`).
  - No support for range conditions (use `BETWEEN` stanzas or `eval` + `lookup`).
  - For range lookups, configure `RANGE` in `transforms.conf`.
