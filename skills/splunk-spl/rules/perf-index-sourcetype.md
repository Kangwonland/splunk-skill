---
title: Always Specify index= and sourcetype=
impact: CRITICAL
tags: performance, index, sourcetype, optimization, bloom-filter
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches"
---

## Always Specify index= and sourcetype=

Omitting `index=` causes Splunk to search all indexes the user has access
to. Omitting `sourcetype=` prevents bucket-level pre-filtering.

**Incorrect:**
```spl
host=webserver1 status=500
```
Searches every accessible index. On large deployments this can scan
terabytes of unrelated data.

**Correct:**
```spl
index=web_logs sourcetype=access_combined host=webserver1 status=500
```
`index=` limits the search to specific index buckets. `sourcetype=` enables
Bloom filter optimization and props.conf field extraction targeting.

**Notes:**
- **Bloom filter:** Splunk maintains per-bucket Bloom filters for `index`,
  `host`, `source`, `sourcetype`. Specifying these allows Splunk to skip
  entire buckets that cannot contain matching events.
- Multiple indexes: `index=web_logs OR index=app_logs`
- Wildcard index: `index=web_*` (use sparingly — still scans multiple indexes)
- Default index: If `index=` is omitted, Splunk uses the user's default
  index list from `authorize.conf` — unpredictable in multi-tenant environments.
- Always add both to scheduled searches and alerts to avoid scope creep.
