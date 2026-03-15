---
title: Splunk CLI Real-Time Search
impact: MEDIUM
tags: cli, rtsearch, real-time, rt, windowed, earliest, latest
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api"
---

## Splunk CLI Real-Time Search

`./splunk rtsearch` executes a real-time search that streams results
as events arrive.

**Windowed real-time (30-second window):**
```bash
./splunk rtsearch 'index=web_logs status=500 | stats count by host' \
  -earliest_time rt-30s \
  -latest_time rt
```
Returns results within a sliding 30-second window.

**Unbounded real-time:**
```bash
./splunk rtsearch 'index=web_logs status=500' \
  -earliest_time rt \
  -latest_time rt
```
Streams all incoming events matching the filter.

**Async detached search:**
```bash
./splunk rtsearch 'index=web_logs | stats count by host' \
  -earliest_time rt-5m -latest_time rt \
  -detach true
# Returns SID. Retrieve results with:
./splunk search -sid <SID>
```

**Notes:**
- **`earliest_time` and `latest_time` are REQUIRED** for `rtsearch` — no defaults.
- **RT time syntax:** `rt` = now (real-time); `rt-30s` = 30 seconds before now;
  `rt+30s` = 30 seconds in the future.
- **Windowed RT searches** maintain a rolling time window — good for dashboards.
- **Unbounded RT** (`rt` to `rt`) streams all incoming events — no historical data.
- Real-time searches consume more resources than historical — use sparingly.
- `detach=true` allows the search to continue after CLI disconnects.
