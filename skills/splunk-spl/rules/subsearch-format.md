---
title: Subsearch format Command
impact: HIGH
tags: subsearch, format, AND, OR, emptystr, row-prefix
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-f/format"
---

## Subsearch format Command

The `format` command controls how subsearch results are assembled into a
search filter string. The default behavior (implicit `format`) suffices
for most cases.

**Incorrect:**
```spl
index=web_logs [
  index=blocklist
  | fields ip
  | format
]
```
Without specifying the row/column separators, the format defaults to
`(ip=val1 OR ip=val2 ...)` — correct, but explicit format allows AND logic.

**Correct (AND logic across multiple fields):**
```spl
index=web_logs [
  index=threat_intel
  | fields src_ip, country
  | format "(" "(" "OR" ")" "AND" ")"
]
```
Produces: `((src_ip=x AND country=y) OR (src_ip=a AND country=b))`

**Notes:**
- **Implicit `format`** (no explicit command): Splunk uses OR between rows,
  field=value for each column. Sufficient for single-field subsearches.
- **`format mvlist=true`** — output results as a multivalue field instead
  of a search string. Useful for `| where field IN (mvlist)`.
- **`emptystr=""`** — controls the string output when subsearch returns no results.
  Default empty string causes outer search to return no events (intended behavior).
  Set `emptystr="NOT x=*"` to return all events when subsearch is empty.
- Row/column prefix/suffix syntax: `format "rowPrefix" "colPrefix" "colSep" "colSuffix" "rowSep" "rowSuffix"`
