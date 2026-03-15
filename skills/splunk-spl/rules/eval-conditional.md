---
title: Conditional Eval Functions
impact: HIGH
tags: eval, if, case, coalesce, nullif, conditional
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/evaluation-functions/about-evaluation-functions"
---

## Conditional Eval Functions

Use `if()`, `case()`, `coalesce()`, and `nullif()` for conditional field
assignments. Choosing the right function avoids nested logic and null pitfalls.

**Incorrect:**
```spl
index=web_logs
| eval status_label=if(status=200, "OK", if(status=404, "Not Found", if(status=500, "Error", "Unknown")))
```
Nested `if()` chains are hard to read and maintain.

**Correct:**
```spl
index=web_logs
| eval status_label=case(
    status=200, "OK",
    status=404, "Not Found",
    status=500, "Error",
    true(), "Unknown"
  )
```
`case()` evaluates conditions in order — include `true()` as the final
catch-all.

**Notes:**
- `if(condition, true_val, false_val)` — simple binary conditional.
- `case(cond1, val1, cond2, val2, ..., true(), default)` — multi-branch.
- `coalesce(field1, field2, "default")` — returns the first non-null value.
  Use to merge fields: `eval user=coalesce(user, username, "anonymous")`
- `nullif(field, value)` — returns null if field equals value, else returns field.
  Use to clean sentinel values: `eval duration=nullif(duration, -1)`
- `validate(cond, "error_msg")` — for macros: returns error string if condition fails.
