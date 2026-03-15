---
title: Example-Based Field Extraction with erex
impact: HIGH
tags: extraction, erex, auto-extraction, examples
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-e/erex"
---

## Example-Based Field Extraction with erex

`erex` generates a regex automatically from example values. Use when you
know what the extracted value looks like but not how to write the regex.

**Incorrect:**
```spl
index=web_logs | rex "user=([A-Za-z0-9_]+)"
```
Manually writing regex is error-prone and may miss edge cases.

**Correct:**
```spl
index=web_logs | erex username examples="jsmith, alice_99, bob.jones"
```
`erex` infers a regex from the examples and extracts `username` from events.

**Notes:**
- `erex` outputs the generated regex in the `erex_regex` field — inspect
  it to verify accuracy: `| erex username examples="..." | table erex_regex | dedup erex_regex`
- `counterexamples="admin"` — provide negative examples to refine the regex.
- `erex` is slower than `rex` — use it for exploration, then replace with
  an explicit `rex` pattern once validated.
- The generated regex is applied to `_raw` by default; use `field=fieldname`
  to extract from a specific field.
