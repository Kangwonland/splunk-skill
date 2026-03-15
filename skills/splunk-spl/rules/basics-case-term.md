---
title: CASE() and TERM() for Exact Matching
impact: CRITICAL
tags: basics, CASE, TERM, phrase-matching, segmentation
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## CASE() and TERM() for Exact Matching

Use `TERM()` to match an exact term (bypassing segmentation) and `CASE()`
for case-sensitive matching.

**Incorrect:**
```spl
index=web_logs uri=/login/admin
```
The forward slash segments the value; Splunk searches for `/`, `login`,
and `admin` separately, returning unexpected results.

**Correct:**
```spl
index=web_logs TERM(/login/admin)
```
`TERM()` treats the entire string as a single search term, bypassing
major and minor breakers.

**Notes:**
- `CASE(ExactValue)` enforces case-sensitive matching: `CASE(ERROR)` will
  not match `error` or `Error`.
- `TERM()` only works on indexed terms — it cannot be applied to
  search-time extracted fields.
- Both `CASE()` and `TERM()` are **not** search directives
  (unlike `REQUIRED_TAGS()`).
- See: `basics-directives.md` for actual search directives.
