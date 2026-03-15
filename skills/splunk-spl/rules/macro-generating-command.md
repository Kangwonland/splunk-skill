---
title: Macros with Generating Commands
impact: HIGH
tags: macro, generating-command, pipe, from, search, inputlookup
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/knowledge-objects/macros/about-search-macros"
---

## Macros with Generating Commands

When a macro starts with a generating command (`search`, `inputlookup`,
`tstats`, `from`), it must be preceded by `|` (pipe) at the call site.

**Incorrect:**
```spl
`get_blocked_ips`
| stats count by host
```
If `get_blocked_ips` expands to `inputlookup blocklist.csv`, the search
string becomes `inputlookup blocklist.csv | stats count by host` — valid
only as a standalone search, not when appended to another search.

**Correct (macro used as a standalone search):**
```spl
`get_blocked_ips`
| lookup server_metadata.csv ip OUTPUT country
```

**Correct (macro used as a subsearch generating command):**
```spl
index=web_logs [
  `get_blocked_ips`
  | return 1000 ip
]
```

**Correct (macro preceded by pipe when generating in a pipeline):**
```spl
index=web_logs
| append [`get_blocked_ips`]
```

**Notes:**
- `max_macro_depth = 100` in `limits.conf [search]` — limits recursive macro expansion.
- `max_subsearch_depth = 8` applies when macros are used inside subsearches.
- A generating command macro that starts with `search` can appear at the
  beginning of a search string without a pipe.
- Macros that start with `|` (transforming commands) must always be
  preceded by a pipe at the call site:
  ```
  Macro `add_geo`: | lookup geo.csv ip OUTPUT country
  Usage: index=web_logs `add_geo` | stats count by country
  ```
