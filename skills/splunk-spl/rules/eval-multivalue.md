---
title: Multivalue Eval Functions
impact: HIGH
tags: eval, multivalue, mvcount, mvexpand, split, mvindex
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/evaluation-functions/about-evaluation-functions"
---

## Multivalue Eval Functions

Splunk fields can hold multiple values. Use multivalue functions to
manipulate, filter, and iterate over them without exploding events.

**Incorrect:**
```spl
index=web_logs
| stats values(uri) as uris by host
| where like(uris, "%admin%")
```
`like()` does not iterate over individual values in a multivalue field —
it operates on the field's string representation, making its behavior
unreliable for multivalue data. Use `mvfilter` for explicit per-value matching.

**Correct:**
```spl
index=web_logs
| stats values(uri) as uris by host
| where mvcount(mvfilter(match(uris, "admin"))) > 0
```

**Notes:**
- `mvcount(field)` — number of values in a multivalue field.
- `mvindex(field, 0)` — first value; `mvindex(field, -1)` — last value.
- `mvappend(field, "new_val")` — append a value to a multivalue field.
- `mvfilter(match(field, "pattern"))` — filter values matching a regex.
- `mvsort(field)` — sort values alphabetically.
- `mvjoin(field, ",")` — join multivalue into a delimited string.
- `split(string, ",")` — split a delimited string into multivalue field.
- `| mvexpand field` — explode a multivalue field into separate events
  (one event per value). Warning: multiplies event count.
