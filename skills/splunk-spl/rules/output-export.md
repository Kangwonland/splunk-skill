---
title: Exporting and Sending Results
impact: MEDIUM
tags: output, outputcsv, outputlookup, sendemail, collect, export
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-o/outputcsv"
---

## Exporting and Sending Results

`outputcsv`, `outputlookup`, `sendemail`, and `collect` write results
to files, lookups, email, or indexes.

**outputcsv (write to CSV):**
```spl
index=web_logs
| stats count by status
| outputcsv status_counts.csv
```
Writes to `$SPLUNK_HOME/var/run/splunk/csv/status_counts.csv`.

**outputlookup (write to lookup file):**
```spl
index=web_logs
| stats count by host
| outputlookup host_counts.csv
```
Persists lookup for use in subsequent `| lookup` calls.

**sendemail (alert email):**
```spl
index=web_logs status=500
| stats count by host
| sendemail to="ops@example.com" subject="Error summary" inline=true
```

**collect (summary indexing):**
```spl
index=web_logs
| stats count by host
| collect index=summary sourcetype=stash
```

**Notes:**
- `outputcsv append=true` — append to existing CSV instead of overwriting.
- `outputlookup createempty=false` — don't overwrite if result set is empty.
- `sendemail result_limit=10000` — max results included in email
  (configurable in `limits.conf [email]`).
- `sendemail format=csv` / `format=table` — attachment format.
- `outputlookup` vs `outputcsv`: `outputlookup` integrates with lookup
  configurations in `transforms.conf`; `outputcsv` writes directly.
