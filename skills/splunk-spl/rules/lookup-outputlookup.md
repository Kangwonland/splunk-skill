---
title: Writing Results to Lookup Tables
impact: MEDIUM
tags: lookup, outputlookup, write, CSV, KV-Store
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/lookups/about-lookups"
---

## Writing Results to Lookup Tables

Use `outputlookup` to persist search results to a CSV lookup or KV store
for later use by other searches.

**Incorrect:**
```spl
index=web_logs
| stats count by client_ip
| outputlookup ip_counts.csv append=true
```
`append=true` appends new rows each run — the file grows unboundedly.

**Correct:**
```spl
index=web_logs earliest=-1d@d latest=@d
| stats count by client_ip
| sort -count
| outputlookup ip_counts.csv
```
Without `append=true`, the file is replaced on each run. Schedule daily.

**Notes:**
- `outputlookup` writes to `$SPLUNK_HOME/etc/apps/<app>/lookups/` by default.
- `outputlookup <lookup_name>` — uses the lookup stanza defined in `transforms.conf`.
- `outputlookup <filename>.csv` — writes directly to the app's lookups directory.
- **`createinapp=true`** — write to the current app's lookups directory.
- **`max=N`** — limit rows written (default: no limit).
- **KV Store:** `outputlookup my_kv_collection` writes to a KV store collection.
  KV stores support concurrent reads/writes and partial updates.
- Scheduled searches writing to lookups are a common pattern for
  "pre-computed" data: run expensive aggregation daily, query fast lookup at search time.
