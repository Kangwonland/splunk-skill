---
title: Subsearch Basics
impact: HIGH
tags: subsearch, brackets, execution-order, generating-command
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-subsearches"
---

## Subsearch Basics

A subsearch runs first, produces a result set, and passes it to the outer
search as a filter. The subsearch must start with a generating command.

**Incorrect:**
```spl
index=web_logs [ where status=500 ]
```
`where` is not a generating command — subsearch must start with `search`,
`inputlookup`, `tstats`, or another generating command.

**Correct:**
```spl
index=web_logs [
  search index=error_log level=CRITICAL
  | return 100 host
]
```
`search` is the generating command. `return` formats results as a filter.

**Notes:**
- Subsearch executes **before** the outer search — results are materialized
  into a filter string inserted at the `[` position.
- Default result format: `(field=val1 OR field=val2 OR ...)` joined by the
  outer search field name.
- `| return N field` — limit to N results, output as `field=val` pairs.
- `| return $field` — prepend `$` to output as bare `val1 val2 val3` (for
  use with `IN` syntax: `status IN (500, 503, 504)` is more efficient).
- Subsearch results are cached for `ttl=300` seconds (default) in `limits.conf`.
- See: `subsearch-limits.md` for hard limits.
