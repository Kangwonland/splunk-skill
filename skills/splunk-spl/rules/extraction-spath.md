---
title: JSON and XML Extraction with spath
impact: HIGH
tags: extraction, spath, json, xml, structured-data
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-s/spath"
---

## JSON and XML Extraction with spath

Use `spath` to extract fields from JSON or XML structured data stored in
`_raw` or other fields. Auto-extraction via `KV_MODE=json` in props.conf
is preferred for recurring sources.

**Incorrect:**
```spl
index=app_logs | rex field=_raw "\"status\":\s*\"(?<status>[^\"]+)\""
```
Using regex to parse JSON is fragile — whitespace, key order, and encoding
variations cause mismatches.

**Correct:**
```spl
index=app_logs | spath input=_raw path=status
```
Or auto-extract all fields:
```spl
index=app_logs | spath
```

**Notes:**
- `spath path=response.body.errors{}.code` — dot notation for nested objects,
  `{}` for array iteration (creates multivalue field).
- `spath input=field_name` — extract from a specific field (not `_raw`).
- `eval status=spath(_raw, "response.status")` — use `spath()` as an eval
  function for inline extraction without a separate command.
- For high-volume JSON logs, configure `KV_MODE = json` in `props.conf`
  for index-time extraction — far more efficient than search-time `spath`.
- XML: use dot notation for elements, `@attr` for attributes:
  `spath path=root.item{}.@id`
