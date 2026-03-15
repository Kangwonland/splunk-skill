---
title: Search Modes (Fast / Smart / Verbose)
impact: MEDIUM
tags: basics, search-mode, fast-mode, verbose-mode, performance
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## Search Modes (Fast / Smart / Verbose)

Search mode affects which fields are extracted and what data is returned.

**Incorrect:**
```spl
| rest /services/search/jobs splunk_server=local
| search mode=verbose
```
Setting mode in SPL has no effect — search mode is set at job dispatch time.

**Correct:**
Set mode in the UI (Search > Mode dropdown) or via REST API at job creation:
```bash
curl -u admin:password https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs" \
  -d search_mode=fast
```

**Notes:**
- **Fast mode:** Only extracts fields required by the search. No field
  discovery. Fastest execution. Best for known-field searches.
- **Smart mode (default):** Applies field extraction based on the search type.
  Transforming searches behave like Fast mode; event searches like Verbose.
- **Verbose mode:** Extracts all fields, builds field summary, runs event
  type tagging. Slowest but shows all available fields. Use for exploration.
- Mode cannot be changed mid-search via SPL — it is a job-level setting.
