---
title: tstats prestats Mode
impact: MEDIUM
tags: tstats, prestats, append, multi-timerange, pre-aggregation
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/tstats"
---

## tstats prestats Mode

`prestats=true` emits raw pre-aggregation rows instead of final
aggregated results, enabling `| append` to combine multiple `tstats`
queries before final aggregation.

**Multi-timerange union pattern:**
```spl
| tstats prestats=true count WHERE index=web_logs earliest=-7d latest=-1d BY host
| append [
  | tstats prestats=true count WHERE index=web_logs earliest=-1d latest=now BY host
]
| stats count BY host
```
Combines two time windows before final `stats`.

**Notes:**
- `prestats=true` output is not human-readable — rows have internal
  format for consumption by `stats`.
- **No `AS` renaming** in prestats mode — field renaming in `BY` clause
  is not supported when `prestats=true`.
- `append=true` (different from `| append`) — appends rows to existing
  result set without clearing it (used with iterative tstats calls).
- Final `| stats` (not `| tstats`) must aggregate the prestats output.
- Useful for merging: different indexes, different time windows, different
  data models into one aggregation.
