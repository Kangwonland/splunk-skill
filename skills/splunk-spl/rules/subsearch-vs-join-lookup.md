---
title: Subsearch vs join vs lookup Decision Guide
impact: HIGH
tags: subsearch, join, lookup, selection-guide, performance
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-subsearches"
---

## Subsearch vs join vs lookup Decision Guide

Use the right combination method for the task — lookup is almost always
fastest, join is usually the worst choice.

**Decision order (fastest to slowest):**

1. **`| lookup`** — static or KV-store data, O(1) hash lookup, no size limit.
2. **`| stats`** — self-join equivalent, aggregates inline without a separate dataset.
3. **Subsearch `[ ]`** — dynamic filter, up to 10,000 results, 60s timeout.
4. **`| join`** — last resort, memory-intensive, limited parallelism.

**Incorrect (using join for lookup-type enrichment):**
```spl
index=web_logs
| join type=left host [
  | inputlookup server_metadata.csv
]
```

**Correct:**
```spl
index=web_logs
| lookup server_metadata.csv host OUTPUT datacenter, owner
```

**Notes:**
- **Use subsearch when:**
  - Filter values are dynamic (computed from another index at search time).
  - Result set is < 10,000 rows.
  - You need OR-joined filter values.
- **Use lookup when:**
  - Enriching events with data from a CSV or KV store.
  - Dataset is static or updated periodically.
  - Scale matters (millions of events).
- **Use stats for self-join:**
  `| stats values(field_a) as a, first(field_b) as b by key` replaces
  many `join` use cases.
- **Use join only when:**
  - You need exact row-by-row matching with preserved duplicates.
  - No other method works.
- See: `join-avoid.md` for join performance details.
