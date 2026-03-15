---
name: splunk-spl
description: >
  TRIGGER when: user mentions Splunk, SPL, SPL2, or any Splunk search
  command (stats, eval, rex, where, transaction, foreach, join, lookup,
  tstats, datamodel, collect, map, subsearch, eventstats, streamstats).
  Also trigger for: Splunk search optimization, Splunk field extraction,
  Splunk REST API, Splunk CLI, SPL-to-SPL2 migration, or troubleshooting
  Splunk search behavior.
  DO NOT TRIGGER when: user asks about other log/search platforms
  (Elasticsearch, OpenSearch, Datadog, Sumo Logic, Sentinel/KQL,
  Logstash, Grafana) even if the concepts overlap.
  Provides Splunk Enterprise 9.4 best practices — 86 rules covering
  all SPL/SPL2 commands, performance patterns, and migration guides.
license: MIT
metadata:
  author: splunk-skill
  version: "1.0.0"
  splunk-version: "9.4"
  skill-updated: "2026-03-10"
  reference: "https://help.splunk.com/en/splunk-enterprise/search"
---

# Splunk SPL / SPL2 Best Practices

86 rules across 25 categories with incorrect/correct examples and official
limits.conf values.

> All documentation references use **help.splunk.com** only.
> docs.splunk.com is deprecated and no longer maintained.

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Files |
|----------|----------|--------|--------|-------|
| 1 | Core Search Basics | CRITICAL | `basics-` | 9 |
| 1 | Time Modifiers | CRITICAL | `time-` | 3 |
| 1 | Performance Optimization | CRITICAL | `perf-` | 6 |
| 2 | Field Extraction | HIGH | `extraction-` | 5 |
| 2 | Eval & Functions | HIGH | `eval-` | 5 |
| 2 | Statistics & Aggregation | HIGH | `stats-` | 4 |
| 2 | Subsearch | HIGH | `subsearch-` | 4 |
| 2 | Join | HIGH | `join-` | 3 |
| 2 | Lookup | HIGH | `lookup-` | 3 |
| 2 | Macros | HIGH | `macro-` | 3 |
| 3 | Transaction | MEDIUM | `transaction-` | 2 |
| 3 | Iteration (foreach/map) | MEDIUM | `iteration-` | 2 |
| 3 | Data Models | MEDIUM | `datamodel-` | 3 |
| 3 | tstats | MEDIUM | `tstats-` | 4 |
| 3 | Summary Indexing | MEDIUM | `summary-` | 2 |
| 3 | Output & Reshaping | MEDIUM | `output-` | 3 |
| 4 | Geographic Commands | LOW-MEDIUM | `geo-` | 2 |
| 4 | Anomaly Detection | LOW-MEDIUM | `anomaly-` | 2 |
| 4 | Prediction & Trending | LOW-MEDIUM | `prediction-` | 1 |
| 4 | Metrics | LOW-MEDIUM | `metrics-` | 1 |
| 4 | Set Operations | LOW-MEDIUM | `set-` | 1 |
| 5 | CLI Search | MEDIUM | `cli-` | 3 |
| 5 | REST API | MEDIUM | `rest-` | 4 |
| 6 | SPL2 | HIGH | `spl2-` | 7 |
| 6 | SPL→SPL2 Migration | HIGH | `migration-` | 4 |

## Quick Reference

### 1. Core Search Basics (CRITICAL)
- `basics-search-pipeline` — Pipeline structure, command ordering
- `basics-field-expressions` — Boolean logic, `NOT` vs `!=`, predicates
- `basics-case-term` — `CASE()` / `TERM()` phrase matching
- `basics-wildcards` — Wildcard patterns, never use leading `*`
- `basics-default-fields` — `_raw`,`_time`,`host`,`source`,`sourcetype`
- `basics-search-types` — Raw event vs Transforming; Sparse vs Dense
- `basics-sql-mapping` — SQL → SPL equivalents
- `basics-directives` — `REQUIRED_TAGS()`, `REQUIRED_EVENTTYPES()`, `READ_SUMMARY()`
- `basics-search-modes` — Fast / Smart / Verbose mode

### 2. Time Modifiers (CRITICAL)
- `time-relative-syntax` — Unit aliases, snap-to (`@h`,`@d`,`@w0`,`@q`), chained offsets
- `time-realtime-modifiers` — `rt` prefix, windowed RT, `rtorder`
- `time-index-vs-event` — `earliest`/`latest` vs `index_earliest`/`index_latest`

### 3. Performance Optimization (CRITICAL)
- `perf-filter-early` — Filter before first pipe
- `perf-fields-command` — Drop unused fields early with `| fields`
- `perf-index-sourcetype` — Always specify `index=` and `sourcetype=`
- `perf-time-range` — Narrow time ranges; avoid All Time
- `perf-avoid-wildcard-prefix` — Never use leading `*` in index queries
- `perf-search-normalization` — Built-in optimization, `localop`, `redistribute`

### 4. Field Extraction (HIGH)
- `extraction-rex` — Named groups `(?<field>pattern)`
- `extraction-erex` — Example-based auto-extraction
- `extraction-spath` — JSON/XML extraction
- `extraction-multikv` — Table-formatted event extraction
- `extraction-kvform` — Form-template extraction

### 5. Eval & Functions (HIGH)
- `eval-conditional` — `if()`, `case()`, `coalesce()`, `nullif()`
- `eval-multivalue` — `mvcount`, `mvexpand`, `split`, `mvindex`
- `eval-time` — `strftime`, `strptime`, `relative_time`, `now()`
- `eval-json` — `spath`, `json_extract`, `json_object`, `json_set`
- `eval-text` — `lower`, `replace`, `match`, `substr`, `printf`

### 6. Statistics & Aggregation (HIGH)
- `stats-vs-eventstats-streamstats` — Three-way selection guide
- `stats-timechart` — `timechart` patterns, `span=`, `limit=`
- `stats-by-clause` — Grouping patterns, high-cardinality pitfalls
- `stats-memory` — `dc()` vs `estdc()`, `values()` memory cost

### 7. Subsearch (HIGH)
- `subsearch-basics` — `[ ]` syntax, execution order
- `subsearch-format` — Implicit `format`, AND/OR structure, `emptystr=`
- `subsearch-limits` — `maxout=10000`, `maxtime=60s`, `max_subsearch_depth=8`
- `subsearch-vs-join-lookup` — Decision: lookup → stats → subsearch → join

### 8. Join (HIGH)
- `join-patterns` — `type=inner/left/outer`
- `join-avoid` — Prefer `lookup`/`append`/`stats`; `subsearch_maxout=50000`
- `join-limits` — Memory impact, `limits.conf` stanza

### 9. Lookup (HIGH)
- `lookup-add-fields` — `OUTPUT` vs `OUTPUTNEW`, wildcard matching
- `lookup-outputlookup` — Writing results to lookup tables
- `lookup-vs-join` — O(1) hash vs sequential scan

### 10. Macros (HIGH)
- `macro-definition` — Backtick syntax, `$arg$` tokens, naming `macro(n)`
- `macro-best-practices` — DRY, index/sourcetype abstraction
- `macro-generating-command` — Pipe prefix rule; `max_macro_depth=100`

### 11. Transaction (MEDIUM)
- `transaction-vs-stats` — Prefer `stats`; use `transaction` for multi-event grouping
- `transaction-patterns` — `startswith`/`endswith`, `maxspan`, `maxpause`, `maxevents`

### 12. Iteration (MEDIUM)
- `iteration-foreach` — 4 modes, placeholders `<<FIELD>>`/`<<ITEM>>`/`<<ITER>>`
- `iteration-map` — `$field$` tokens, `maxsearches=10`, risky command

### 13. Data Models (MEDIUM)
- `datamodel-basics` — `datamodel` command, `pivot`, dataset hierarchy
- `datamodel-acceleration` — `summariesonly`, `nodename`
- `datamodel-cim` — CIM normalization, cross-sourcetype patterns

### 14. tstats (MEDIUM)
- `tstats-basics` — Indexed fields only; `tstats` vs `stats` selection guide
- `tstats-datamodel` — `FROM datamodel=`, `nodename`, `summariesonly`
- `tstats-prefix` — `PREFIX()` directive, lowercase, `walklex` discovery
- `tstats-prestats` — `prestats=true`, multi-timerange `append=true` pattern

### 15. Summary Indexing (MEDIUM)
- `summary-collect` — `sourcetype=stash`, `sistats→collect`, `addinfo=true`
- `summary-tscollect` — `tscollect` → tsidx → `tstats` fast re-query

### 16. Output & Reshaping (MEDIUM)
- `output-reshape` — `transpose`, `untable`, `xyseries`
- `output-formatting` — `fieldformat`, `reltime`, `rangemap`, `gauge`
- `output-export` — `outputcsv`, `outputlookup`, `sendemail`, `collect`

### 17–21. Geographic / Anomaly / Prediction / Metrics / Set (LOW-MEDIUM)
- `geo-iplocation`, `geo-geostats`
- `anomaly-detection`, `anomaly-clustering`
- `prediction-trendline`
- `metrics-mstats`
- `set-operations`

### 22–23. CLI & REST API (MEDIUM)
- `cli-basics`, `cli-parameters`, `cli-rtsearch`
- `rest-auth`, `rest-create-job`, `rest-retrieve-results`, `rest-export`

### 24–25. SPL2 & Migration (HIGH)
- `spl2-from-into`, `spl2-modules`, `spl2-branching`, `spl2-lambda`, `spl2-data-types`, `spl2-custom-functions`, `spl2-spl1-wrapper`
- `migration-syntax-differences`, `migration-from-vs-search`, `migration-new-commands`, `migration-compatibility-profiles`

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/basics-search-pipeline.md
rules/perf-filter-early.md
rules/subsearch-limits.md
```

For large lookup tables (command lists, function catalogs, limits.conf values):

```
references/spl-commands-index.md
references/limits-conf-subsearch.md
references/time-modifiers.md
```

## Full Compiled Document

For comprehensive review tasks requiring all 86 rules at once: `AGENTS.md`
For targeted guidance, load individual files from `rules/` and `references/` as needed.

## Maintenance

For version update procedure, see: `references/update-workflow.md`

```bash
make update VERSION=<new-splunk-version>
```
