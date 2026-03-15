---
title: Search Pipeline Structure
impact: CRITICAL
tags: basics, pipeline, command-order, performance
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## Search Pipeline Structure

Place filtering commands as early as possible to minimize data flowing
through subsequent pipeline stages.

**Incorrect:**
```spl
index=web_logs
| stats count by status
| where status=500
| lookup error_codes.csv status OUTPUT description
| where description="Internal Server Error"
```
Lookups and `where` filters run on the full dataset.

**Correct:**
```spl
index=web_logs status=500
| lookup error_codes.csv status OUTPUT description
| where description="Internal Server Error"
| stats count by status, description
```
Filter at the `search` command level (before any pipe) to reduce events
reaching downstream commands.

**Notes:**
- Distributable streaming commands (`eval`, `rex`, `rename`, `fields`,
  `where`) run on indexers — place them before centralized commands
  (`streamstats`, `transaction`) to reduce data transferred to the search head.
- Move `lookup` filters immediately after the `lookup` command.
- See: `references/spl-commands-by-type.md` for command type classification.
