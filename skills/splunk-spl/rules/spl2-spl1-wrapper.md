---
title: SPL2 spl1 Wrapper Command
impact: HIGH
tags: spl2, spl1, wrapper, interop, legacy, migration
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-search-commands/spl1"
---

## SPL2 spl1 Wrapper Command

`| spl1 "..."` embeds a legacy SPL1 pipeline inside an SPL2 search.
Use for SPL1 commands not yet available in SPL2.

**Embedding SPL1 commands in SPL2:**
```spl2
from main
| where sourcetype="access_combined"
| spl1 "transaction session_id maxspan=30m"
| stats count() AS sessions BY host
```
`transaction` is not yet in SPL2 — `spl1` wrapper executes it.

**Complex SPL1 subsearch:**
```spl2
from main
| spl1 "| inputlookup blocklist.csv | return 1000 ip"
```

**Notes:**
- `spl1 "..."` — the argument is a complete SPL1 pipeline string.
- The wrapper receives the SPL2 pipeline's current result set as input.
- SPL1 commands not yet in SPL2: `transaction`, `inputlookup`, `geostats`,
  `iplocation`, `anomalydetection`, `predict`, and others.
- **Check SPL2 command availability** before using the wrapper — the SPL2
  command set grows with each Splunk release.
- The wrapper adds overhead — avoid for high-frequency or performance-critical paths.
- `spl1` wrapper works in both Splunk Cloud and Enterprise (v9.4+).
