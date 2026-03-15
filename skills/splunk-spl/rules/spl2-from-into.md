---
title: SPL2 from and into — Entry and Exit Points
impact: HIGH
tags: spl2, from, into, dataset, entry-point
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-search-commands/from"
---

## SPL2 from and into — Entry and Exit Points

`from` replaces the leading `search` command in SPL2. `into` writes
results to a dataset.

**SPL1 equivalent:**
```spl
search index=web_logs sourcetype=access_combined status=500
| stats count by host
```

**SPL2 from syntax:**
```spl2
from main
| where sourcetype="access_combined" AND status=500
| stats count() AS count BY host
```

**from datamodel:**
```spl2
from datamodel:Authentication.Authentication
| where action="failure"
| stats count() AS failures BY user
```

**into (write results):**
```spl2
from main
| stats count() AS count BY host
| into summary
```
Writes to the `summary` dataset (index).

**from lookup with module namespace:**
```spl2
from lookup:myapp.geo_lookup
| where country="Korea"
```
Lookups defined in a module use `module.name` syntax (see `spl2-modules` rule).
The `$default` namespace is used for lookups not in an explicit module.

**Notes:**
- `from <index>` — search an event index by name (no `index=` prefix).
- `from datamodel:<Name>.<Dataset>` — query a data model dataset.
- `from lookup:<name>` — read from a lookup file (in `$default` namespace).
- `from lookup:<module>.<name>` — read from a lookup in a specific module namespace.
- `into <index>` — write results to an index (equivalent to `| collect`).
- `from` requires explicit time range via `| where _time > relative_time(now(), "-24h")`.
- Default output of `from`: all events in the dataset (equivalent to `search *`).
