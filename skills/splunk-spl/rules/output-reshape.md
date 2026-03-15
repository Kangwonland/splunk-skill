---
title: Output Reshaping Commands
impact: MEDIUM
tags: output, transpose, untable, xyseries, reshape
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/transpose"
---

## Output Reshaping Commands

`transpose`, `untable`, and `xyseries` reshape tabular results for
visualization and further processing.

**transpose (rows → columns):**
```spl
index=web_logs
| stats count by status, host
| transpose 5 column_name=host header_field=status
```
Pivots: status values become column headers, hosts become rows (up to 5 columns).

**xyseries (stat table → matrix):**
```spl
index=web_logs
| stats count by host, status
| xyseries host status count
```
Produces a matrix: hosts as rows, status codes as column headers.

**untable (matrix → rows — inverse of xyseries):**
```spl
index=web_logs
| stats count by host, status
| xyseries host status count
| untable host status count
```

**Notes:**
- `transpose N` — limits output to N columns (default unlimited).
- `transpose includeempty=true` — include null-valued fields.
- `xyseries` is the inverse of `timechart`: converts rows to columns.
- `untable field1 field2 value` — specify row key, column key, value field names.
- All three are non-distributable streaming commands — run on search head.
