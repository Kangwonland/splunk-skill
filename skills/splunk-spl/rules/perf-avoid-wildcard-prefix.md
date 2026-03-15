---
title: Avoid Leading Wildcards in Search Terms
impact: CRITICAL
tags: performance, wildcard, leading-wildcard, bloom-filter, index-scan
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches"
---

## Avoid Leading Wildcards in Search Terms

A leading wildcard (`*value`) bypasses Bloom filter lookups and forces a
full lexicon scan across every bucket in the time range.

**Incorrect:**
```spl
index=web_logs *login*
```
Both leading and embedded wildcards force full event scanning. Cannot use
any index optimizations.

**Correct:**
```spl
index=web_logs login*
```
Trailing wildcard allows Bloom filter lookup for the `login` prefix.

For mid-string matching, use `rex` after initial filtering:
```spl
index=web_logs uri=login*
| rex field=uri "login(?<login_path>\S+)"
```

**Notes:**
- **Why leading `*` is expensive:** Splunk uses a per-bucket lexicon (sorted
  term list). A trailing wildcard does a prefix lookup (O(log n)). A leading
  wildcard must scan the entire lexicon (O(n)).
- **`min_prefix_len`** in `limits.conf [search]`: Minimum characters before
  a wildcard for index optimization. Default: `1`. Setting to `3` prevents
  very short prefix queries.
- `TERM(/login/admin)` — for terms containing special characters that would
  otherwise be segmented; see `basics-case-term.md`.
