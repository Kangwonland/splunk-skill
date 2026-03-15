---
title: Field Extraction with rex
impact: HIGH
tags: extraction, rex, regex, named-group, field-extraction
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-r/rex"
---

## Field Extraction with rex

Use named capture groups to extract fields from raw events. Unnamed groups
are ignored and unnamed matches are not stored as fields.

**Incorrect:**
```spl
index=web_logs | rex "(\d+\.\d+\.\d+\.\d+)"
```
No named group — the matched IP address is not stored in any field.

**Correct:**
```spl
index=web_logs | rex "(?<client_ip>\d+\.\d+\.\d+\.\d+)"
```
The captured value is stored in `client_ip` field for downstream use.

**Notes:**
- `rex field=_raw` is the default — omit `field=` when extracting from raw events.
- `rex field=uri mode=sed "s/\?.*$//"` — use `mode=sed` for in-place field replacement.
- `rex` vs `regex`: `rex` extracts fields; `regex` filters events (like `where` with regex).
- `max_match=1` (default) — only the **first** match is extracted. To extract
  **all** occurrences into a multivalue field, set `max_match=0`:
  `rex field=_raw "key=(?<vals>[^,]+)" max_match=0`
- Prefer `EXTRACT-` transforms in `props.conf` for recurring extractions
  to avoid per-search overhead.
