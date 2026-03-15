---
title: SPL2-Only Commands
impact: MEDIUM
tags: migration, spl2, new-commands, branch, route, expand, flatten
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL2-Only Commands

Commands available in SPL2 with no direct SPL1 equivalent.

| Command | Description | SPL1 Workaround |
|---------|-------------|-----------------|
| `branch` | Parallel pipeline execution | Sequential `\| append` subsearches |
| `route` | Conditional event routing to branches | `\| eval` + multiple `\| where` passes |
| `thru` | Pass events through a sub-pipeline without consuming | No equivalent |
| `expand` | Expand array/object fields into rows | `\| mvexpand` (multivalue only) |
| `flatten` | Flatten nested JSON into dot-notation fields | `\| spath` (partial) |
| `decrypt` | Decrypt encrypted fields | No equivalent |
| `ocsf` | Normalize events to OCSF schema | Manual `\| eval` field mapping |
| `into` | Write results to a dataset | `\| collect index=...` |

**branch example:**
```spl2
from main
| branch
  [stats count() AS requests BY host]
  [where status=500 | stats count() AS errors BY host]
```

**expand example:**
```spl2
from main
| expand tags
```
Creates one row per element in the `tags` array field.

**Notes:**
- SPL2 command set grows with each Splunk release — check current docs.
- Use `| spl1 "..."` wrapper for SPL1 commands not yet in SPL2.
- `ocsf` requires the OCSF add-on installed.
