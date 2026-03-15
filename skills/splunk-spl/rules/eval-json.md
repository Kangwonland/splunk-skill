---
title: JSON Eval Functions
impact: HIGH
tags: eval, json, json_extract, json_object, json_set, spath
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/evaluation-functions/about-evaluation-functions"
---

## JSON Eval Functions

Use eval JSON functions to parse, build, and modify JSON inline without
a separate `spath` command.

**Incorrect:**
```spl
index=app_logs
| spath input=payload path=user.id output=user_id
| spath input=payload path=user.name output=user_name
```
Multiple `spath` commands are verbose and each makes a separate pass.

**Correct:**
```spl
index=app_logs
| eval user_id=json_extract(payload, "user.id"),
       user_name=json_extract(payload, "user.name")
```
Single `eval` with multiple `json_extract()` calls in one pass.

**Notes:**
- `json_extract(field, "path.to.key")` — extract a value from JSON string.
- `json_object("key1", val1, "key2", val2)` — build a JSON object string.
- `json_set(field, "path", value)` — set/update a key in a JSON string.
- `json_del(field, "path")` — remove a key from a JSON string.
- `json_keys(field)` — return top-level keys as a multivalue field.
- `json_array_to_mv(field)` — convert a JSON array to a multivalue field.
- `spath()` as eval function: `eval val=spath(field, "path")` — equivalent
  to `json_extract` but also handles XML.
