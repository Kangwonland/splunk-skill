---
title: Splunk CLI Search Parameters Reference
impact: LOW
tags: cli, parameters, maxout, timeout, output, wrap
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api"
---

## Splunk CLI Search Parameters Reference

Full parameter reference for `./splunk search` and `./splunk rtsearch`.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `-app` | `search` | App context for search |
| `-auth` | _(required if not saved)_ | `username:password` |
| `-detach` | `false` | Return job SID immediately, don't wait |
| `-earliest_time` | _(none = All Time)_ | Start of time range |
| `-latest_time` | _(none = now)_ | End of time range |
| `-max_time` | `0` (unlimited) | Max seconds to run |
| `-maxout` | `100` | Max events returned |
| `-output` | `auto` | Output format: `auto`, `csv`, `json`, `xml`, `raw` |
| `-preview` | `true` | Stream preview results during search |
| `-timeout` | `0` (unlimited) | Job expiration timeout in seconds |
| `-uri` | `localhost:8089` | Management port URI |
| `-wrap` | `true` | Wrap long lines in terminal output |

**Example with multiple parameters:**
```bash
./splunk search 'index=web_logs | stats count by host' \
  -earliest_time -7d \
  -latest_time now \
  -maxout 0 \
  -output csv \
  -max_time 300 \
  -auth admin:changeme
```

**Notes:**
- `-maxout 0` means use system limit from `limits.conf [search] max_count`.
- `-detach true` returns a SID; use `./splunk search -sid <SID>` to retrieve results.
- `-output raw` outputs `_raw` only (no CSV headers or JSON wrapping).
