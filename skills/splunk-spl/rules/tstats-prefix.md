---
title: tstats PREFIX() for Raw Indexed Terms
impact: MEDIUM
tags: tstats, PREFIX, indexed-segments, walklex, raw-terms
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/tstats"
---

## tstats PREFIX() for Raw Indexed Terms

`PREFIX(key=)` in `tstats` accesses raw indexed segments (terms from
`_raw`) without defining explicit indexed fields.

**Correct:**
```spl
| tstats count WHERE index=web_logs PREFIX(uri_path=/api/) BY host
```
Counts events where `_raw` contains the term `uri_path=/api/` (as
a continuous indexed segment — no major breakers within the term).

**Discover available terms:**
```spl
| walklex index=web_logs prefix="status=" type=term
| table term, count
```
`walklex` enumerates the tsidx lexicon to find indexed terms.

**Notes:**
- **Case sensitivity:** PREFIX matching is case-sensitive against indexed terms.
  Many string fields are lowercased during indexing by Splunk's field processing
  (controlled by `props.conf` `TRANSFORMS` and `SEDCMD`). `PREFIX(Status=200)`
  will NOT match if the term is indexed as `status=200`. Verify actual indexed
  case with `walklex` before using PREFIX with mixed-case values.
- **No major breakers:** The term must appear as a continuous segment.
  Splunk's major breakers (space, `[`, `]`, `(`, `)`, `{`, `}`, `!`, etc.)
  split terms. `uri=/api/v1/users` is split at `/`.
- **collect compatibility:** When using `| collect` after `tstats` with PREFIX,
  set `collect_ignore_minor_breakers=true` in `limits.conf [collect]`.
- **Performance:** PREFIX scans are faster than full `search` but slower
  than explicit indexed field queries.
