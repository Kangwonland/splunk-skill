---
title: Table-Formatted Event Extraction with multikv
impact: MEDIUM
tags: extraction, multikv, table, key-value
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-m/multikv"
---

## Table-Formatted Event Extraction with multikv

`multikv` extracts fields from events formatted as ASCII tables (header row
+ data rows). Common for `netstat`, `ps`, `top`, and similar CLI outputs.

**Incorrect:**
```spl
index=os_logs sourcetype=ps_output
| rex field=_raw "(?<pid>\d+)\s+(?<user>\S+)\s+(?<cmd>\S+)"
```
Manually parsing tabular data with regex is fragile when column widths vary.

**Correct:**
```spl
index=os_logs sourcetype=ps_output
| multikv forceheader=1
| table pid, user, cmd, %cpu
```
`multikv` reads the first row as headers and extracts subsequent rows as events.

**Notes:**
- `forceheader=1` — treat the first line as the header row.
- `multikv fields pid user cmd` — extract only specified columns.
- `multikv conf=my_multikv_stanza` — use a custom stanza from `transforms.conf`
  for complex table formats.
- Each data row becomes a separate event — the original event is discarded.
- Configure `REPORT-multikv` in `props.conf` for automatic extraction at index time.
