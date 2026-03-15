---
title: Filter Events as Early as Possible
impact: CRITICAL
tags: performance, filter, pipeline, optimization
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches"
---

## Filter Events as Early as Possible

Every event passed through a pipe costs CPU and memory. Filter aggressively
before the first pipe to reduce the dataset for all downstream commands.

**Incorrect:**
```spl
index=web_logs
| eval is_error=if(status>=500,1,0)
| where is_error=1
| stats count by host
```
`eval` and `where` run on all events — millions of rows computed unnecessarily.

**Correct:**
```spl
index=web_logs status>=500
| stats count by host
```
Push filters into the search string: Splunk applies them at the indexer
level using Bloom filters and index structures before data is transferred.

**Notes:**
- Distributable streaming commands (`eval`, `rex`, `fields`, `where`,
  `rename`) run on indexers when placed before centralized commands.
- Non-distributable commands (`streamstats`, `transaction`, `eventstats`)
  run on the search head — minimize data reaching them.
- Use `index=`, `sourcetype=`, `host=`, and time range filters first;
  they are applied at the bucket level before event scanning.
- See: `references/spl-commands-by-type.md` for distributable vs centralized.
