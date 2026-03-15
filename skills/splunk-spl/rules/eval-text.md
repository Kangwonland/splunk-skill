---
title: Text Eval Functions
impact: MEDIUM
tags: eval, text, lower, upper, replace, substr, match, printf
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/evaluation-functions/about-evaluation-functions"
---

## Text Eval Functions

Use text functions in `eval` for string manipulation. Prefer built-in
functions over `rex` for simple transformations.

**Incorrect:**
```spl
index=web_logs
| rex field=uri "^(?<path>[^?]+)"
| rex field=path "^/(?<first_segment>[^/]+)"
```
Two separate `rex` commands for simple string operations.

**Correct:**
```spl
index=web_logs
| eval path=if(match(uri, "\?"), substr(uri, 1, index(uri, "?")-1), uri)
| eval first_segment=mvindex(split(ltrim(path, "/"), "/"), 0)
```

**Notes:**
- `lower(field)` / `upper(field)` — case conversion.
- `len(field)` — string length.
- `substr(field, start, length)` — substring extraction (1-indexed).
- `index(field, "substr")` — position of first occurrence (1-indexed), 0 if not found.
- `replace(field, "regex", "replacement")` — regex-based replacement.
- `match(field, "regex")` — returns 1 if field matches regex, 0 otherwise.
- `like(field, "pattern%")` — SQL-style pattern matching (`%` = wildcard).
- `printf("%05.2f", value)` — C-style formatting: `printf("%-10s %d", name, count)`
- `trim(field)` / `ltrim(field)` / `rtrim(field)` — whitespace trimming.
