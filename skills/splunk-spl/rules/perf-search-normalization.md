---
title: Search Normalization and Parallel Reduce
impact: HIGH
tags: performance, normalization, localop, redistribute, parallel-reduce
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches"
---

## Search Normalization and Parallel Reduce

Splunk's search optimizer automatically rewrites searches for distributed
execution. Understanding `localop` and `redistribute` helps override this
when needed.

**Incorrect (forcing unnecessary centralization):**
```spl
index=web_logs
| eventstats count by host
| where count > 1000
```
`eventstats` is centralized — all data moves to the search head before
any filtering.

**Correct (using stats instead for distributable aggregation):**
```spl
index=web_logs
| stats count by host
| where count > 1000
```
`stats` with `by` clause is distributable in Splunk's parallel reduce mode:
partial aggregations run on indexers, then are merged on the search head.

**Notes:**
- **Search normalization:** Splunk automatically rewrites `search` commands
  and moves distributable commands to indexers. This is transparent.
- **`| localop`:** Forces all subsequent commands to run on the search head
  only. Use when indexer-local data (lookups, REST endpoints) is needed:
  `index=_internal | localop | rest /services/search/jobs`
- **`| redistribute`:** Re-distributes events across indexers by a field
  for parallel processing of centralized commands:
  `index=web_logs | redistribute by host | transaction host maxspan=5m`
- **Parallel reduce:** `stats`, `chart`, `timechart`, `top`, `rare` all
  support partial aggregation on indexers. Commands like `transaction`,
  `streamstats`, `eventstats` do not.
- See: `references/spl-commands-by-type.md` for command distribution categories.
