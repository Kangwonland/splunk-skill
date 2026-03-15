---
title: foreach Command — Iteration Modes
impact: HIGH
tags: foreach, multifield, multivalue, json, iteration, template
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-f/foreach"
---

## foreach Command — Iteration Modes

`foreach` iterates over a set of fields or values, applying a template
subsearch to each. Four modes exist; each uses different token substitution.

**Mode 1 — multifield (default):**
```spl
index=web_logs
| foreach req_* [eval total_req_<<FIELD>> = '<<FIELD>>' * 1.1]
```
`<<FIELD>>` is the matched field name; `<<MATCHSTR>>` is the glob match portion.

**Mode 2 — multivalue:**
```spl
index=web_logs
| eval methods = split("GET POST PUT DELETE", " ")
| foreach mode=multivalue methods [
  | eval method_count_<<ITEM>> = coalesce('method_count_<<ITEM>>', 0) + 1
]
```
`<<ITEM>>` is each value in the multivalue field. Only one `eval` allowed per iteration in this mode.

**Mode 3 — json_array:**
```spl
index=api_logs
| eval tags_json = json_array("web", "api", "mobile")
| foreach mode=json_array tags_json [
  | eval tag_<<ITER>> = "<<ITEM>>"
]
```
`<<ITEM>>` is each JSON array element; `<<ITER>>` is the 0-based index.

**Mode 4 — auto_collections:**
```spl
index=structured_logs
| foreach mode=auto_collections devices [eval device_<<ITER>> = '<<ITEM>>']
```
Handles both multivalue fields and JSON arrays automatically.

**Notes:**
- `<<MATCHSEG1>>`, `<<MATCHSEG2>>`, `<<MATCHSEG3>>` — capture groups from wildcard glob pattern.
- Single `eval` restriction in MV/JSON modes: no piped commands inside the template.
- Multifield mode supports piped sub-pipeline: `foreach field* [| eval x=<<FIELD>> | stats sum(x)]`.
- `maxvals=N` — max iterations (default 50, `limits.conf [foreach]`).
