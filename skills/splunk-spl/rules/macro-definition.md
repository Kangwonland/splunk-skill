---
title: Defining and Using Search Macros
impact: HIGH
tags: macro, backtick, arguments, naming, knowledge-objects
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/knowledge-objects/macros/about-search-macros"
---

## Defining and Using Search Macros

Search macros expand inline at search time. Name macros with arity suffix
`macro_name(N)` to distinguish zero-arg from parameterized versions.

**Incorrect:**
```spl
`web_filter(index=web_logs)`
```
Passing an entire key=value expression as an argument is fragile — the macro
must handle arbitrary string injection.

**Correct macro definition (`web_filter(1)`):**
```
search index=web_logs sourcetype=access_combined $filter_expr$
```
Usage:
```spl
`web_filter(status=500)`
`web_filter(host=webserver1 status>=400)`
```

**Notes:**
- **Syntax:** `` `macro_name` `` (zero args) or `` `macro_name(arg1, arg2)` ``
- **Arguments:** Referenced as `$arg_name$` in the macro body. Argument
  names are defined in the macro configuration in Splunk Web or `macros.conf`.
- **Naming convention:** `my_macro` (0 args), `my_macro(1)` (1 arg),
  `my_macro(2)` (2 args) — each is a distinct macro.
- **Validation expressions:** Add regex validation in the macro definition
  to catch bad arguments at search time.
- Macros expand recursively — `max_macro_depth=100` in `limits.conf [search]`
  limits recursion depth.
- See: `macro-generating-command.md` for macros that begin with a pipe.
