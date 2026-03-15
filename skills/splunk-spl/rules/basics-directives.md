---
title: Search Execution Directives
impact: CRITICAL
tags: basics, directives, REQUIRED_TAGS, REQUIRED_EVENTTYPES, READ_SUMMARY, performance
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## Search Execution Directives

Search directives modify how Splunk executes a search. They are NOT the
same as `TERM()` or `CASE()` which modify term matching.

**Incorrect:**
```spl
index=web_logs REQUIRED_TAGS(web)
```
`REQUIRED_TAGS()` used as a standalone filter — it does not filter events,
it constrains which knowledge objects are applied.

**Correct (using READ_SUMMARY for acceleration):**
```spl
index=web_logs
| tstats summariesonly=true count FROM datamodel=Web.Web
```
Or, to force summary index usage:
```spl
READ_SUMMARY index=web_logs sourcetype=access_combined
| stats count by status
```

**Notes:**
- `REQUIRED_TAGS(tag1, tag2)` — limits event type application to objects
  tagged with all specified tags. Reduces knowledge object overhead.
- `REQUIRED_EVENTTYPES(et1, et2)` — restricts which event types are applied.
- `READ_SUMMARY` — instructs Splunk to read from summary indexes when available.
- These directives appear at the **start** of the search string, before any pipe.
- Directives are distinct from search commands — they cannot be piped.
