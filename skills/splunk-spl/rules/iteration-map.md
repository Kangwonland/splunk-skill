---
title: map Command — Iterative Search Execution
impact: HIGH
tags: map, iteration, subsearch, token-substitution, maxsearches
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-m/map"
---

## map Command — Iterative Search Execution

`map` executes a search for each result row, substituting field values
as tokens. Results are unioned into a single result set.

**Correct:**
```spl
index=web_logs status=500
| stats count by host
| map search="search index=web_logs host=$host$ | head 5"
```
Runs a separate search per `host` value, returning up to 5 events each.

**Notes:**
- **Token syntax:** `$fieldname$` substitutes the field value into the search string.
  `$_serial_id$` is the 0-based row number.
- **`maxsearches=N`** (default 10): Max iterations. `0` is **NOT** unlimited —
  it means 0 searches execute (bug-prone default). Set explicitly.
- **Cannot follow `append`/`appendpipe`** — `map` must be the first command
  after `|` in a pipeline that receives results.
- **Risky command:** `map` triggers Splunk's "risky command" safeguard in the UI.
  Users must confirm before execution. Disable per-user or role in `authorize.conf`:
  `srchFilter = NOT (map)`
- **Performance:** Each iteration is a full search. With `maxsearches=100` and
  slow subsearches, runtime multiplies by 100×.
- Use `| join` or `| lookup` for enrichment instead of `| map` when possible.
