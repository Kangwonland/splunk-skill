---
title: Splunk CLI Search Basics
impact: MEDIUM
tags: cli, splunk-search, command-line, output, maxout
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api"
---

## Splunk CLI Search Basics

The `./splunk search` command runs SPL queries from the command line.

**Basic search:**
```bash
./splunk search 'index=web_logs status=500 | stats count by host' \
  -earliest_time -24h -latest_time now
```

**Real-time search:**
```bash
./splunk rtsearch 'index=web_logs | stats count by status' \
  -earliest_time rt-30s -latest_time rt
```

**Output to file:**
```bash
./splunk search 'index=web_logs | stats count by host' \
  -earliest_time -24h -output csv > results.csv
```

**Notes:**
- **No default time range:** Without `-earliest_time`/`-latest_time`, the
  search runs as All Time — scans everything.
- **Default `maxout=100`** — only 100 results returned by default.
  Override with `-maxout 10000` (or 0 for unlimited within `limits.conf`).
- **Quote handling:** Linux/macOS: single quotes around SPL. Windows: double
  quotes. Nested quotes in SPL require escaping.
- `./splunk` is at `$SPLUNK_HOME/bin/splunk`.
- Must be run as the Splunk OS user or with `sudo -u splunk`.
- See: `cli-parameters.md` for full parameter reference.
