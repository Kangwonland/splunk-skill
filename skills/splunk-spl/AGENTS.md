# Splunk SPL / SPL2 Best Practices — Complete Reference

**Version:** 1.0.0  
**Splunk Version:** 9.4  
**Source:** help.splunk.com/en/splunk-enterprise/search  
**Updated:** 2026-02-21

> This document is for AI agents and LLMs. Contains all 86 rules compiled inline,
> organized by category. For progressive disclosure, load individual rule files
> from `rules/` directory instead. All documentation URLs reference
> help.splunk.com (docs.splunk.com is deprecated).

---

## Table of Contents

1. [Core Search Basics](#core-search-basics) — 9 rules
2. [Time and Date](#time-and-date) — 3 rules
3. [Performance Optimization](#performance-optimization) — 6 rules
4. [Field Extraction](#field-extraction) — 5 rules
5. [eval Functions](#eval-functions) — 5 rules
6. [Statistics and Aggregation](#statistics-and-aggregation) — 4 rules
7. [Subsearches](#subsearches) — 4 rules
8. [join Command](#join-command) — 3 rules
9. [Lookups](#lookups) — 3 rules
10. [Macros](#macros) — 3 rules
11. [Transaction and Iteration](#transaction-and-iteration) — 4 rules
12. [Data Models](#data-models) — 3 rules
13. [tstats Command](#tstats-command) — 4 rules
14. [Summary Indexing](#summary-indexing) — 2 rules
15. [Output and Formatting](#output-and-formatting) — 4 rules
16. [Geographic Commands](#geographic-commands) — 2 rules
17. [Anomaly and Prediction](#anomaly-and-prediction) — 3 rules
18. [Metrics](#metrics) — 1 rule
19. [CLI and REST API](#cli-and-rest-api) — 7 rules
20. [SPL2](#spl2) — 7 rules
21. [Migration SPL1 → SPL2](#migration-spl1--spl2) — 4 rules

---

## Core Search Basics

### Search Pipeline Structure [CRITICAL]

Place filtering commands as early as possible to minimize data flowing
through subsequent pipeline stages.

**Incorrect:**
```spl
index=web_logs
| stats count by status
| where status=500
| lookup error_codes.csv status OUTPUT description
| where description="Internal Server Error"
```
Lookups and `where` filters run on the full dataset.

**Correct:**
```spl
index=web_logs status=500
| lookup error_codes.csv status OUTPUT description
| where description="Internal Server Error"
| stats count by status, description
```
Filter at the `search` command level (before any pipe) to reduce events
reaching downstream commands.

**Notes:**
- Distributable streaming commands (`eval`, `rex`, `rename`, `fields`,
  `where`) run on indexers — place them before centralized commands
  (`streamstats`, `transaction`) to reduce data transferred to the search head.
- Move `lookup` filters immediately after the `lookup` command.
- See: `references/spl-commands-by-type.md` for command type classification.

### Types of Searches [CRITICAL]

Understanding search types helps predict result behavior and performance.

**Incorrect:**
```spl
index=web_logs
| stats count by status
| head 10
```
Using `head` after a transforming command (`stats`) only limits the table
rows — the full aggregation still ran on all events.

**Correct:**
```spl
index=web_logs
| head 10
| stats count by status
```
Or for large datasets, pre-filter before aggregating:
```spl
index=web_logs earliest=-1h
| stats count by status
```

**Notes:**
- **Raw event search:** Returns individual events from `_raw`. Commands:
  `search`, `where`, `eval`, `rex`, `fields`.
- **Transforming search:** Aggregates events into statistical results.
  Commands: `stats`, `chart`, `timechart`, `top`, `rare`, `geostats`.
  Once a transforming command runs, you cannot return to raw events.
- **Sparse vs Dense:** Sparse fields exist in few events (use `| search field=*`
  to filter); Dense fields exist in most events.
- Transforming commands mark the end of the "streaming" phase — data
  must be fully transferred to the search head before they execute.

### Default Fields and Indexed vs Search-Time Fields [CRITICAL]

Splunk adds several default fields at index time. Understanding which fields
are indexed vs search-time extracted affects search performance.

**Incorrect:**
```spl
index=web_logs | search host::webserver1
```
`host::` (double colon) is for indexed fields only. If `host` has been
overridden by a search-time extraction, this may not match.

**Correct:**
```spl
index=web_logs host=webserver1
```
Use `field=value` for all standard field lookups. Use `field::value` only
when you explicitly need index-time field matching.

**Notes:**
- **Default indexed fields:** `_time`, `_indextime`, `host`, `source`,
  `sourcetype`, `punct`, `linecount`
- **`_raw`:** The raw event text — always available, never indexed (search-time only)
- **`field=v` vs `field::v`:** Use `=` universally; use `::` only to bypass
  search-time extractions for performance on high-volume indexed fields.
- CIDR matching: `src_ip="192.168.1.0/24"` works in field expressions.
- See: `references/spl-default-fields.md` for the full default fields table.

### Field Expressions and Boolean Logic [CRITICAL]

`NOT field="value"` and `field!="value"` behave differently for null/missing fields.

**Incorrect:**
```spl
index=auth field!="admin"
```
Excludes events where `field` is "admin" but also **excludes events where
`field` does not exist** (null).

**Correct:**
```spl
index=auth NOT field="admin"
```
Returns events where `field` is not "admin" **including** events where
`field` is absent.

**Notes:**
- Use `field::value` (double colon) only for indexed fields — cannot be
  used on search-time extracted fields.
- CIDR notation: `ip="10.0.0.0/24"` matches all IPs in that subnet.
- AND is implicit between terms; OR must be explicit and uppercase.
- See: `references/spl-default-fields.md` for indexed vs search-time fields.

### Wildcard Usage and Pitfalls [CRITICAL]

Never use a leading wildcard (`*value`) in search terms — it forces a
full index scan.

**Incorrect:**
```spl
index=web_logs uri=*admin*
```
Leading `*` cannot use the Bloom filter or index optimizations. Splunk
scans every event.

**Correct:**
```spl
index=web_logs uri=admin*
```
Trailing wildcards can use index lookup structures. Or use `rex` for
mid-string matching:

```spl
index=web_logs | rex field=uri "admin" | where isnotnull(uri)
```

**Notes:**
- `min_prefix_len=1` in `limits.conf [search]` sets the minimum prefix
  length before a wildcard for index queries.
- For exact phrase matching across segmentation boundaries, prefer
  `TERM()` (see `basics-case-term.md`).
- Wildcards in field **names** (not values) are supported by many commands
  (e.g., `fields host*`, `foreach www*`).

### CASE() and TERM() for Exact Matching [CRITICAL]

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

### Search Execution Directives [CRITICAL]

Search directives modify how Splunk executes a search. They are NOT the
same as `TERM()` or `CASE()` which modify term matching.

**Incorrect:**
```spl
index=web_logs REQUIRED_TAGS(web)
```
`REQUIRED_TAGS()` used as a standalone filter — it does not filter events,
it constrains which knowledge objects are applied.

**Correct (using READ_SUMMARY for acceleration):**
```spl
index=web_logs
| tstats summariesonly=true count FROM datamodel=Web.Web
```
Or, to force summary index usage:
```spl
READ_SUMMARY index=web_logs sourcetype=access_combined
| stats count by status
```

**Notes:**
- `REQUIRED_TAGS(tag1, tag2)` — limits event type application to objects
  tagged with all specified tags. Reduces knowledge object overhead.
- `REQUIRED_EVENTTYPES(et1, et2)` — restricts which event types are applied.
- `READ_SUMMARY` — instructs Splunk to read from summary indexes when available.
- These directives appear at the **start** of the search string, before any pipe.
- Directives are distinct from search commands — they cannot be piped.

### Search Modes (Fast / Smart / Verbose) [MEDIUM]

Search mode affects which fields are extracted and what data is returned.

**Incorrect:**
```spl
| rest /services/search/jobs splunk_server=local
| search mode=verbose
```
Setting mode in SPL has no effect — search mode is set at job dispatch time.

**Correct:**
Set mode in the UI (Search > Mode dropdown) or via REST API at job creation:
```bash
curl -u admin:password https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs" \
  -d search_mode=fast
```

**Notes:**
- **Fast mode:** Only extracts fields required by the search. No field
  discovery. Fastest execution. Best for known-field searches.
- **Smart mode (default):** Applies field extraction based on the search type.
  Transforming searches behave like Fast mode; event searches like Verbose.
- **Verbose mode:** Extracts all fields, builds field summary, runs event
  type tagging. Slowest but shows all available fields. Use for exploration.
- Mode cannot be changed mid-search via SPL — it is a job-level setting.

### SQL to SPL Equivalents [HIGH]

Common SQL patterns and their SPL equivalents.

**Incorrect (thinking in SQL):**
```sql
SELECT host, COUNT(*) as cnt
FROM web_logs
WHERE status = 500
GROUP BY host
HAVING cnt > 10
ORDER BY cnt DESC
LIMIT 5
```

**Correct SPL equivalent:**
```spl
index=web_logs status=500
| stats count AS cnt by host
| where cnt > 10
| sort -cnt
| head 5
```

**Notes:**
- See: `references/spl-sql-mapping.md` for the complete SQL→SPL mapping table.
- `WHERE` → filter in search string (before first `|`) or `| where`
- `GROUP BY` → `| stats ... by field`
- `HAVING` → `| where` after `stats`
- `ORDER BY` → `| sort`
- `LIMIT` → `| head`
- `JOIN` → prefer `| lookup` for O(1) performance; see `join-avoid.md`
- `DISTINCT` → `| dedup field` or `| stats count by field`

---

## Time and Date

### Relative Time Syntax and Snap-To [CRITICAL]

Use relative time modifiers with snap-to operators for accurate, repeatable
time windows. Missing snap-to (`@`) causes inconsistent results across runs.

**Incorrect:**
```spl
index=web_logs earliest=-7d latest=now
```
`-7d` is relative to *now* at search time — the window shifts with each run,
making scheduled reports inconsistent.

**Correct:**
```spl
index=web_logs earliest=-7d@d latest=@d
```
Snaps to midnight, giving a consistent full-day window regardless of when
the search runs.

**Notes:**
- **Unit aliases:** `s`=seconds, `m`=minutes, `h`=hours, `d`=days,
  `w`=weeks (w0=Sunday), `mon`=months, `q`=quarters, `y`=years
- **Snap-to syntax:** `@unit` — truncates to the start of that unit
  - `-1h@h` = start of the previous hour
  - `@w0` = start of Sunday (beginning of week)
  - `@q` = start of current quarter
- **Chained offsets:** `earliest=-mon@mon+7d` = 7 days into last month
- **Named times:** `earliest=@d` = midnight today, `earliest=-1d@d` = yesterday midnight
- See: `references/time-modifiers.md` for the full modifier reference table.

### Index Time vs Event Time [HIGH]

`earliest`/`latest` filter by event time (`_time`). Use
`index_earliest`/`index_latest` when you need to filter by when events
were *indexed*, not when they occurred.

**Incorrect:**
```spl
index=web_logs earliest=-1h latest=now
```
If events arrive late (network delays, batch ingestion), events that
happened within the last hour but arrived later are missed.

**Correct (catch late-arriving events):**
```spl
index=web_logs index_earliest=-2h index_latest=now
| where _time >= relative_time(now(), "-1h")
```
Searches buckets indexed in the last 2 hours, then filters by event time.

**Notes:**
- **`_time`:** The event timestamp (when the event occurred). Set by
  `TIME_PREFIX`/`TIME_FORMAT` in props.conf, or defaults to index time.
- **`_indextime`:** When the event was written to the index. Always set
  by Splunk, cannot be spoofed.
- **`earliest`/`latest`:** Filter on `_time`. Most searches use these.
- **`index_earliest`/`index_latest`:** Filter on `_indextime`. Use for:
  - Compliance searches that need "what was indexed when"
  - Finding late-arriving events
  - Re-indexing investigations
- Mixing both: `index_earliest` opens the bucket scope; `earliest`/`latest`
  then filter within those buckets.

### Real-Time Search Modifiers [CRITICAL]

Real-time searches use `rt` prefix time modifiers. UI-based RT and
programmatic RT behave differently — confusing them causes missed events.

**Incorrect:**
```spl
index=web_logs earliest=rt latest=rt
```
Continuous real-time: captures events as they arrive but shows no
historical data. Cannot be used with transforming commands reliably
(events arrive out of order).

**Correct (windowed real-time):**
```spl
index=web_logs earliest=rt-5m latest=rt
```
Windowed RT: uses a 5-minute sliding window, buffers for ordering,
works correctly with `stats` and `timechart`.

**Notes:**
- **`rt` prefix (UI-only):** `earliest=rt` means "real-time now". Only
  meaningful in the Splunk UI search bar — not in scheduled searches or REST API.
- **Windowed RT:** `rt-30s` / `rt+0s` — maintains a time window, handles
  out-of-order events. Suitable for dashboards.
- **`rtorder`:** Use `| rtorder` command after a continuous RT search to
  buffer and reorder events: `| rtorder buffer_span=5s`
- **CLI:** Use `./splunk rtsearch` for real-time searches from CLI;
  `earliest_time` and `latest_time` are **required** parameters.
- Real-time searches consume continuous resources — always use windowed RT
  in production dashboards.

---

## Performance Optimization

### Always Specify index= and sourcetype= [CRITICAL]

Omitting `index=` causes Splunk to search all indexes the user has access
to. Omitting `sourcetype=` prevents bucket-level pre-filtering.

**Incorrect:**
```spl
host=webserver1 status=500
```
Searches every accessible index. On large deployments this can scan
terabytes of unrelated data.

**Correct:**
```spl
index=web_logs sourcetype=access_combined host=webserver1 status=500
```
`index=` limits the search to specific index buckets. `sourcetype=` enables
Bloom filter optimization and props.conf field extraction targeting.

**Notes:**
- **Bloom filter:** Splunk maintains per-bucket Bloom filters for `index`,
  `host`, `source`, `sourcetype`. Specifying these allows Splunk to skip
  entire buckets that cannot contain matching events.
- Multiple indexes: `index=web_logs OR index=app_logs`
- Wildcard index: `index=web_*` (use sparingly — still scans multiple indexes)
- Default index: If `index=` is omitted, Splunk uses the user's default
  index list from `authorize.conf` — unpredictable in multi-tenant environments.
- Always add both to scheduled searches and alerts to avoid scope creep.

### Narrow Time Ranges [CRITICAL]

"All Time" searches scan every bucket in an index. Always specify a time
range — even a generous one is far better than All Time.

**Incorrect:**
```spl
index=web_logs status=500
| stats count by host
```
Without `earliest`/`latest`, this is an All Time search — scans all
historical buckets. On production indexes this can run for minutes or hours.

**Correct:**
```spl
index=web_logs status=500 earliest=-24h latest=now
| stats count by host
```

**Notes:**
- Splunk organizes data in time-based buckets. Time range filters allow
  Splunk to skip entire buckets outside the window — the most impactful
  single optimization available.
- **Snap-to alignment** improves bucket skipping: `earliest=-1d@d` aligns
  to bucket boundaries better than `earliest=-24h`.
- For scheduled reports, always use relative time with snap-to:
  `earliest=-1d@d latest=@d` (yesterday).
- Ad-hoc searches: set time range in the UI time picker or pass
  `earliest_time`/`latest_time` in the REST API.
- Avoid `All Time` in scheduled searches — it re-scans the entire index
  on every run and grows slower as data accumulates.

### Filter Events as Early as Possible [CRITICAL]

Every event passed through a pipe costs CPU and memory. Filter aggressively
before the first pipe to reduce the dataset for all downstream commands.

**Incorrect:**
```spl
index=web_logs
| eval is_error=if(status>=500,1,0)
| where is_error=1
| stats count by host
```
`eval` and `where` run on all events — millions of rows computed unnecessarily.

**Correct:**
```spl
index=web_logs status>=500
| stats count by host
```
Push filters into the search string: Splunk applies them at the indexer
level using Bloom filters and index structures before data is transferred.

**Notes:**
- Distributable streaming commands (`eval`, `rex`, `fields`, `where`,
  `rename`) run on indexers when placed before centralized commands.
- Non-distributable commands (`streamstats`, `transaction`, `eventstats`)
  run on the search head — minimize data reaching them.
- Use `index=`, `sourcetype=`, `host=`, and time range filters first;
  they are applied at the bucket level before event scanning.
- See: `references/spl-commands-by-type.md` for distributable vs centralized.

### Use fields Command to Drop Unused Fields [CRITICAL]

Each field extracted and carried through the pipeline consumes memory and
network bandwidth. Drop unused fields as early as possible.

**Incorrect:**
```spl
index=web_logs
| eval duration_ms=response_time*1000
| stats avg(duration_ms) by status
```
All extracted fields (uri, user_agent, referer, etc.) are carried through
the pipeline even though only `response_time` and `status` are needed.

**Correct:**
```spl
index=web_logs
| fields response_time status
| eval duration_ms=response_time*1000
| stats avg(duration_ms) by status
```
`| fields` is a distributable streaming command — it runs on indexers,
reducing data transferred to the search head.

**Notes:**
- `| fields field1 field2` — keep only listed fields (allowlist)
- `| fields - field1 field2` — remove listed fields (denylist)
- Place `| fields` immediately after the search string for maximum effect.
- `_raw` and `_time` are always retained unless explicitly removed.
- Removing `_raw` with `| fields - _raw` significantly reduces memory
  for large result sets: `index=web_logs | fields - _raw | stats count by status`

### Avoid Leading Wildcards in Search Terms [CRITICAL]

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

### Search Normalization and Parallel Reduce [HIGH]

Splunk's search optimizer automatically rewrites searches for distributed
execution. Understanding `localop` and `redistribute` helps override this
when needed.

**Incorrect (forcing unnecessary centralization):**
```spl
index=web_logs
| eventstats count by host
| where count > 1000
```
`eventstats` is centralized — all data moves to the search head before
any filtering.

**Correct (using stats instead for distributable aggregation):**
```spl
index=web_logs
| stats count by host
| where count > 1000
```
`stats` with `by` clause is distributable in Splunk's parallel reduce mode:
partial aggregations run on indexers, then are merged on the search head.

**Notes:**
- **Search normalization:** Splunk automatically rewrites `search` commands
  and moves distributable commands to indexers. This is transparent.
- **`| localop`:** Forces all subsequent commands to run on the search head
  only. Use when indexer-local data (lookups, REST endpoints) is needed:
  `index=_internal | localop | rest /services/search/jobs`
- **`| redistribute`:** Re-distributes events across indexers by a field
  for parallel processing of centralized commands:
  `index=web_logs | redistribute by host | transaction host maxspan=5m`
- **Parallel reduce:** `stats`, `chart`, `timechart`, `top`, `rare` all
  support partial aggregation on indexers. Commands like `transaction`,
  `streamstats`, `eventstats` do not.
- See: `references/spl-commands-by-type.md` for command distribution categories.

---

## Field Extraction

### Field Extraction with rex [HIGH]

Use named capture groups to extract fields from raw events. Unnamed groups
are ignored and unnamed matches are not stored as fields.

**Incorrect:**
```spl
index=web_logs | rex "(\d+\.\d+\.\d+\.\d+)"
```
No named group — the matched IP address is not stored in any field.

**Correct:**
```spl
index=web_logs | rex "(?<client_ip>\d+\.\d+\.\d+\.\d+)"
```
The captured value is stored in `client_ip` field for downstream use.

**Notes:**
- `rex field=_raw` is the default — omit `field=` when extracting from raw events.
- `rex field=uri mode=sed "s/\?.*$//"` — use `mode=sed` for in-place field replacement.
- `rex` vs `regex`: `rex` extracts fields; `regex` filters events (like `where` with regex).
- `max_match=1` (default) — only the **first** match is extracted. To extract
  **all** occurrences into a multivalue field, set `max_match=0`:
  `rex field=_raw "key=(?<vals>[^,]+)" max_match=0`
- Prefer `EXTRACT-` transforms in `props.conf` for recurring extractions
  to avoid per-search overhead.

### JSON and XML Extraction with spath [HIGH]

Use `spath` to extract fields from JSON or XML structured data stored in
`_raw` or other fields. Auto-extraction via `KV_MODE=json` in props.conf
is preferred for recurring sources.

**Incorrect:**
```spl
index=app_logs | rex field=_raw "\"status\":\s*\"(?<status>[^\"]+)\""
```
Using regex to parse JSON is fragile — whitespace, key order, and encoding
variations cause mismatches.

**Correct:**
```spl
index=app_logs | spath input=_raw path=status
```
Or auto-extract all fields:
```spl
index=app_logs | spath
```

**Notes:**
- `spath path=response.body.errors{}.code` — dot notation for nested objects,
  `{}` for array iteration (creates multivalue field).
- `spath input=field_name` — extract from a specific field (not `_raw`).
- `eval status=spath(_raw, "response.status")` — use `spath()` as an eval
  function for inline extraction without a separate command.
- For high-volume JSON logs, configure `KV_MODE = json` in `props.conf`
  for index-time extraction — far more efficient than search-time `spath`.
- XML: use dot notation for elements, `@attr` for attributes:
  `spath path=root.item{}.@id`

### Example-Based Field Extraction with erex [HIGH]

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

### Table-Formatted Event Extraction with multikv [MEDIUM]

`multikv` extracts fields from events formatted as ASCII tables (header row
+ data rows). Common for `netstat`, `ps`, `top`, and similar CLI outputs.

**Incorrect:**
```spl
index=os_logs sourcetype=ps_output
| rex field=_raw "(?<pid>\d+)\s+(?<user>\S+)\s+(?<cmd>\S+)"
```
Manually parsing tabular data with regex is fragile when column widths vary.

**Correct:**
```spl
index=os_logs sourcetype=ps_output
| multikv forceheader=1
| table pid, user, cmd, %cpu
```
`multikv` reads the first row as headers and extracts subsequent rows as events.

**Notes:**
- `forceheader=1` — treat the first line as the header row.
- `multikv fields pid user cmd` — extract only specified columns.
- `multikv conf=my_multikv_stanza` — use a custom stanza from `transforms.conf`
  for complex table formats.
- Each data row becomes a separate event — the original event is discarded.
- Configure `REPORT-multikv` in `props.conf` for automatic extraction at index time.

### Form-Template Extraction with kvform [LOW]

`kvform` extracts field-value pairs using a template that matches the
structure of form-style log entries (label: value).

**Incorrect:**
```spl
index=form_logs | rex "Name:\s*(?<name>\S+)\s+Email:\s*(?<email>\S+)"
```
Fragile regex breaks when field order changes or spacing differs.

**Correct:**
```spl
index=form_logs | kvform form=contact_form
```
With a `contact_form` stanza in `transforms.conf` defining the template.

**Notes:**
- `kvform` requires a `kvform` stanza in `transforms.conf` or `$SPLUNK_HOME/etc/apps/search/lookups/`.
- Template format: `field_name: <value_pattern>`
- Less common than `rex`/`spath` — primarily used for structured text forms
  (support tickets, email templates, configuration snippets).
- Consider `rex` with named groups as a simpler alternative for most cases.

---

## eval Functions

### Conditional Eval Functions [HIGH]

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

### Multivalue Eval Functions [HIGH]

Splunk fields can hold multiple values. Use multivalue functions to
manipulate, filter, and iterate over them without exploding events.

**Incorrect:**
```spl
index=web_logs
| stats values(uri) as uris by host
| where like(uris, "%admin%")
```
`like()` on a multivalue field only matches if the first value contains the pattern.

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

### Time Eval Functions [HIGH]

Use time eval functions to format, parse, and calculate with timestamps.
Mixing epoch and formatted times without conversion causes type errors.

**Incorrect:**
```spl
index=web_logs
| eval age=now() - timestamp
```
If `timestamp` is a formatted string (not epoch), subtraction fails.

**Correct:**
```spl
index=web_logs
| eval epoch_ts=strptime(timestamp, "%Y-%m-%d %H:%M:%S")
| eval age_seconds=now() - epoch_ts
| eval age_human=tostring(age_seconds/3600, "duration")
```

**Notes:**
- `now()` — current time as Unix epoch (integer seconds).
- `_time` — event timestamp, already in epoch format.
- `strftime(_time, "%Y-%m-%d %H:%M:%S")` — epoch → formatted string.
- `strptime("2024-01-15 10:30:00", "%Y-%m-%d %H:%M:%S")` — string → epoch.
- `relative_time(now(), "-1d@d")` — apply a relative time modifier to an epoch.
- `toepoch("01/15/2024:10:30:00")` — parse common date formats to epoch.
- `fromepoch(epoch)` — epoch → human-readable string (uses local timezone).
- See: `references/time-format-variables.md` for `strftime`/`strptime` format codes.

### JSON Eval Functions [HIGH]

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

### Text Eval Functions [MEDIUM]

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

---

## Statistics and Aggregation

### stats vs eventstats vs streamstats [HIGH]

Choose the right aggregation command based on whether you need to retain
raw events and whether order matters.

**Incorrect (using stats when original events are needed):**
```spl
index=web_logs
| stats avg(response_time) as avg_rt by host
| where response_time > avg_rt * 2
```
After `stats`, individual `response_time` values no longer exist.

**Correct:**
```spl
index=web_logs
| eventstats avg(response_time) as avg_rt by host
| where response_time > avg_rt * 2
| table _time, host, response_time, avg_rt
```
`eventstats` adds the aggregate as a new field while retaining all original events.

**Notes:**
- **`stats`:** Aggregates events into summary rows. Original events are lost.
  Fastest; runs in parallel reduce mode. Use for final reporting.
- **`eventstats`:** Adds aggregate fields to each original event. Events retained.
  Centralized (search head only) — expensive on large datasets.
- **`streamstats`:** Running/cumulative aggregation in event order.
  `| streamstats sum(bytes) as cumulative_bytes by session_id`
  Use for session analysis, running totals, window functions.
- **`streamstats window=N`:** Sliding window over last N events:
  `| streamstats window=5 avg(response_time) as rolling_avg`
- Decision: `stats` for summaries → `eventstats` for per-event enrichment
  → `streamstats` for ordered/windowed calculations.

### stats BY Clause and High-Cardinality Pitfalls [HIGH]

High-cardinality `by` fields (user IDs, session IDs, IPs) cause `stats`
to produce millions of rows, overwhelming search head memory.

**Incorrect:**
```spl
index=web_logs
| stats count, values(uri) as uris by session_id
```
If there are 10M unique sessions, this produces 10M rows with potentially
large multivalue `uris` fields — likely to hit memory limits.

**Correct:**
```spl
index=web_logs
| stats count by session_id
| where count > 100
| sort -count
| head 1000
```
Filter immediately after `stats` to reduce the result set.

**Notes:**
- `maxresultrows` in `limits.conf [searchresults]` caps result rows (default: 50000).
  Exceeding this silently truncates results.
- **High-cardinality alternatives:**
  - `dc(field)` — distinct count without enumerating all values.
  - `estdc(field)` — estimated distinct count (faster, less memory).
  - `top N field` — top N values with counts (built-in limit).
- `values(field)` accumulates all distinct values — memory grows with cardinality.
  Use `list(field)` to preserve order but limit with `| head`.
- For session analysis, prefer `transaction` or `streamstats` over
  `stats values()` to avoid unbounded memory growth.

### stats Memory Optimization [HIGH]

Some `stats` functions accumulate all values in memory. Use approximate
or bounded alternatives when cardinality is high.

**Incorrect:**
```spl
index=web_logs
| stats dc(client_ip) as unique_ips, values(client_ip) as all_ips by country
```
`dc()` must store all unique IPs to count them exactly. `values()` stores
every IP — memory scales with unique IP count per country.

**Correct:**
```spl
index=web_logs
| stats estdc(client_ip) as approx_unique_ips, count by country
```
`estdc()` uses HyperLogLog approximation — ~2% error, constant memory.

**Notes:**
- **Memory-intensive functions:** `values()`, `list()`, `dc()` — store all values.
- **Memory-efficient alternatives:**
  - `estdc(field)` for `dc(field)` when exact count is not required.
  - `count` instead of `values()` when enumeration is not needed.
  - `first(field)` / `last(field)` instead of `values()` when one value suffices.
- `values(field)` returns distinct values sorted; `list(field)` returns all
  values in order (may include duplicates).
- `limits.conf [stats]` — `maxvalues=100000` limits values per cell (default).
  Exceeding this causes silent truncation.
- For approximate percentiles: `perc95(field)` is exact; `exactperc95(field)`
  forces exact calculation (more memory). Use `perc` for large datasets.

### timechart Patterns and Configuration [HIGH]

`timechart` creates time-series data. Misconfiguring `span=` and `limit=`
causes misleading charts and truncated data.

**Incorrect:**
```spl
index=web_logs
| timechart count by status
```
Without `span=`, Splunk auto-selects a span that may be too coarse. With
many `status` values, extra values are collapsed into "OTHER".

**Correct:**
```spl
index=web_logs
| timechart span=1h count by status limit=0
```
`limit=0` disables the "OTHER" bucket — all values are shown.

**Notes:**
- `span=` sets the time bucket size: `span=1m`, `span=1h`, `span=1d`, `span=1w`.
- `limit=N` — max number of `by` field values shown (default: 10). Values
  beyond the limit are grouped into "OTHER". Set `limit=0` to show all.
- `useother=false` — hide the "OTHER" bucket entirely when `limit` is set.
- `usenull=false` — hide null values in the series.
- `timechart` always outputs `_time` as the first column — cannot be removed.
- `per_second(field)` / `per_minute(field)` — rate functions:
  `| timechart per_minute(bytes) as bytes_per_min`
- `cont=false` — skip time buckets with no events (gaps instead of zeros).

---

## Subsearches

### Subsearch Basics [HIGH]

A subsearch runs first, produces a result set, and passes it to the outer
search as a filter. The subsearch must start with a generating command.

**Incorrect:**
```spl
index=web_logs [ where status=500 ]
```
`where` is not a generating command — subsearch must start with `search`,
`inputlookup`, `tstats`, or another generating command.

**Correct:**
```spl
index=web_logs [
  search index=error_log level=CRITICAL
  | return 100 host
]
```
`search` is the generating command. `return` formats results as a filter.

**Notes:**
- Subsearch executes **before** the outer search — results are materialized
  into a filter string inserted at the `[` position.
- Default result format: `(field=val1 OR field=val2 OR ...)` joined by the
  outer search field name.
- `| return N field` — limit to N results, output as `field=val` pairs.
- `| return $field` — prepend `$` to output as bare `val1 val2 val3` (for
  use with `IN` syntax: `status IN (500, 503, 504)` is more efficient).
- Subsearch results are cached for `ttl=300` seconds (default) in `limits.conf`.
- See: `subsearch-limits.md` for hard limits.

### Subsearch Limits and limits.conf [HIGH]

Subsearches have hard resource limits. Exceeding them silently truncates
results — the outer search runs on an incomplete filter.

**Incorrect:**
```spl
index=web_logs [
  index=blocklist
  | table ip
]
```
If blocklist has more than 10,000 IPs, results are silently truncated to
`maxout=10000`. The outer search misses blocked IPs beyond this limit.

**Correct (for large filter sets — use lookup instead):**
```spl
index=web_logs
| lookup blocklist.csv ip OUTPUT is_blocked
| where is_blocked=true
```
Lookups have no result-count limit and are significantly faster.

**Notes:**
- **`limits.conf [subsearch]`:**
  - `maxout = 10000` — max results returned by a subsearch (default).
  - `maxtime = 60` — max seconds a subsearch can run (default).
  - `ttl = 300` — seconds subsearch results are cached (default).
- **`limits.conf [search]`:**
  - `max_subsearch_depth = 8` — max nesting depth of subsearches.
- When subsearch hits `maxout`, no warning is shown in the UI — results
  are quietly truncated.
- **Alternatives for large datasets:**
  - `| lookup` — no size limit, O(1) hash lookup.
  - `| tstats` with `WHERE` — indexed fields, no subsearch needed.
  - `| inputlookup` as outer search — avoids subsearch entirely.
- See: `subsearch-vs-join-lookup.md` for decision guide.

### Subsearch format Command [HIGH]

The `format` command controls how subsearch results are assembled into a
search filter string. The default behavior (implicit `format`) suffices
for most cases.

**Incorrect:**
```spl
index=web_logs [
  index=blocklist
  | fields ip
  | format
]
```
Without specifying the row/column separators, the format defaults to
`(ip=val1 OR ip=val2 ...)` — correct, but explicit format allows AND logic.

**Correct (AND logic across multiple fields):**
```spl
index=web_logs [
  index=threat_intel
  | fields src_ip, country
  | format "(" "(" "OR" ")" "AND" ")"
]
```
Produces: `((src_ip=x AND country=y) OR (src_ip=a AND country=b))`

**Notes:**
- **Implicit `format`** (no explicit command): Splunk uses OR between rows,
  field=value for each column. Sufficient for single-field subsearches.
- **`format mvlist=true`** — output results as a multivalue field instead
  of a search string. Useful for `| where field IN (mvlist)`.
- **`emptystr=""`** — controls the string output when subsearch returns no results.
  Default empty string causes outer search to return no events (intended behavior).
  Set `emptystr="NOT x=*"` to return all events when subsearch is empty.
- Row/column prefix/suffix syntax: `format "rowPrefix" "colPrefix" "colSep" "colSuffix" "rowSep" "rowSuffix"`

### Subsearch vs join vs lookup Decision Guide [HIGH]

Use the right combination method for the task — lookup is almost always
fastest, join is usually the worst choice.

**Decision order (fastest to slowest):**

1. **`| lookup`** — static or KV-store data, O(1) hash lookup, no size limit.
2. **`| stats`** — self-join equivalent, aggregates inline without a separate dataset.
3. **Subsearch `[ ]`** — dynamic filter, up to 10,000 results, 60s timeout.
4. **`| join`** — last resort, memory-intensive, limited parallelism.

**Incorrect (using join for lookup-type enrichment):**
```spl
index=web_logs
| join type=left host [
  | inputlookup server_metadata.csv
]
```

**Correct:**
```spl
index=web_logs
| lookup server_metadata.csv host OUTPUT datacenter, owner
```

**Notes:**
- **Use subsearch when:**
  - Filter values are dynamic (computed from another index at search time).
  - Result set is < 10,000 rows.
  - You need OR-joined filter values.
- **Use lookup when:**
  - Enriching events with data from a CSV or KV store.
  - Dataset is static or updated periodically.
  - Scale matters (millions of events).
- **Use stats for self-join:**
  `| stats values(field_a) as a, first(field_b) as b by key` replaces
  many `join` use cases.
- **Use join only when:**
  - You need exact row-by-row matching with preserved duplicates.
  - No other method works.
- See: `join-avoid.md` for join performance details.

---

## join Command

### When to Avoid join [HIGH]

`join` is memory-intensive, not parallelizable, and limited by
`subsearch_maxout`. Use alternatives whenever possible.

**Incorrect (join for enrichment):**
```spl
index=web_logs
| join type=left status [
  | inputlookup http_status_codes.csv
]
```

**Correct (lookup is O(1) vs join's O(n*m)):**
```spl
index=web_logs
| lookup http_status_codes.csv status OUTPUT description, category
```

**Incorrect (join for self-aggregation):**
```spl
index=web_logs
| join host [
  search index=web_logs
  | stats avg(response_time) as avg_rt by host
]
```

**Correct (stats replaces the self-join):**
```spl
index=web_logs
| eventstats avg(response_time) as avg_rt by host
```

**Notes:**
- **Why `join` is slow:**
  - The subsearch runs separately and is fully materialized in memory.
  - Limited to `subsearch_maxout=50000` rows (from `limits.conf [join]`).
  - Cannot use parallel reduce — all processing is on the search head.
- **`| append` + `| stats`** for union-type joins:
  ```spl
  index=source1 | append [search index=source2] | dedup id
  ```
- **`| appendcols`** for column-binding (ZIP join):
  ```spl
  index=A | appendcols [search index=B | fields field_x]
  ```
- See: `subsearch-vs-join-lookup.md` for the decision guide.

### join Command Patterns [HIGH]

`join` combines the main search results with a subsearch result on a common
field. Understand join types to avoid missing events.

**Incorrect (assuming all events are returned):**
```spl
index=web_logs
| join host [
  search index=inventory
  | table host, datacenter
]
```
Default `join` is an inner join — events without a matching host in
inventory are dropped silently.

**Correct (left join to preserve all events):**
```spl
index=web_logs
| join type=left host [
  search index=inventory
  | table host, datacenter
]
```
Left join retains all outer search events; `datacenter` is null for
hosts not found in inventory.

**Notes:**
- `type=inner` (default) — only events with a match in both datasets.
- `type=left` — all outer events; inner events matched where possible.
- `type=outer` — alias for left join in Splunk (full outer join not supported).
- **`overwrite=true`** (default) — subsearch fields overwrite outer fields
  with the same name. Set `overwrite=false` to keep outer values.
- `max=N` — max results from the subsearch per outer event (default: 1).
  `max=0` returns all matches (may multiply row count).
- Prefer `| lookup` for static datasets; see `join-avoid.md`.

### join Command Limits [MEDIUM]

`join` uses subsearch internally and is subject to both subsearch and
join-specific limits. Exceeding them silently truncates results.

**Incorrect (assuming all rows are matched):**
```spl
index=web_logs
| join type=left session_id [
  search index=sessions
  | table session_id, user_id
]
```
If the session index has more than 50,000 unique session IDs, the subsearch
result is truncated to `subsearch_maxout=50000`.

**Correct (for large lookups — use lookup file):**
```spl
index=web_logs
| lookup sessions.csv session_id OUTPUT user_id
```

**Notes:**
- **`limits.conf [join]`:**
  - `subsearch_maxout = 50000` — max rows from join subsearch (default).
  - `subsearch_maxtime = 60` — max seconds for join subsearch (default).
- **`limits.conf [subsearch]`:**
  - `maxout = 10000` — applies to `[ ]` subsearches, not `| join` subsearches.
- **Memory impact:** Both datasets must fit in search head memory simultaneously.
  Large joins on high-cardinality fields can exhaust memory.
- Monitor join memory with: `index=_audit action=search | search info_search_et=* | table info_*`

---

## Lookups

### Adding Fields from Lookup Tables [HIGH]

Use `OUTPUT` to always overwrite, `OUTPUTNEW` to only add fields for events
where the field doesn't already exist.

**Incorrect:**
```spl
index=web_logs
| lookup geo_data.csv ip OUTPUT ip, country, city
```
Including the join key (`ip`) in `OUTPUT` is redundant — it already exists.

**Correct:**
```spl
index=web_logs
| lookup geo_data.csv ip OUTPUT country, city, latitude, longitude
```

**Notes:**
- `OUTPUT field1 AS alias1, field2` — rename lookup output fields inline.
- `OUTPUTNEW field1, field2` — only write output if field doesn't exist.
  Preserves existing enrichment from earlier lookups.
- **Wildcard lookups:** CSV lookup with `WILDCARD(field)` in `transforms.conf`
  allows `*` in lookup values: `192.168.*` matches any IP in that subnet.
- **Cidr lookups:** `CIDR(ip_field)` in `transforms.conf` for subnet matching.
- **Case-insensitive:** Add `case_sensitive_match = false` in `transforms.conf`.
- `| lookup ... WHERE condition` — not supported in SPL; use `| where` after lookup.
- Automatic lookup (configured in `transforms.conf`) runs at search time without
  explicit `| lookup` command.

### lookup vs join Trade-offs [HIGH]

`lookup` is almost always faster than `join`. Use `join` only when
`lookup` cannot express the required logic.

**Incorrect (join for static enrichment):**
```spl
index=web_logs
| join type=left status_code [
  | inputlookup http_codes.csv
  | rename code as status_code
]
```

**Correct:**
```spl
index=web_logs
| lookup http_codes.csv status_code OUTPUT description, severity
```

**Notes:**
- **`lookup` (O(1) hash):** Lookup key is hashed — constant time regardless
  of lookup table size. Supports automatic lookup (no SPL needed).
- **`join` (O(n*m)):** Cross-product matching — performance degrades with
  both dataset sizes. Limited to `subsearch_maxout=50000` rows.
- **When `join` is unavoidable:**
  - Multiple join keys with complex conditions.
  - You need `max=0` to get all matching rows (one-to-many).
  - The second dataset changes frequently (too dynamic for a lookup file).
- **`lookup` limitations:**
  - Lookup file must fit in memory (configurable via `max_matches` in `transforms.conf`).
  - No support for range conditions (use `BETWEEN` stanzas or `eval` + `lookup`).
  - For range lookups, configure `RANGE` in `transforms.conf`.

### Writing Results to Lookup Tables [MEDIUM]

Use `outputlookup` to persist search results to a CSV lookup or KV store
for later use by other searches.

**Incorrect:**
```spl
index=web_logs
| stats count by client_ip
| outputlookup ip_counts.csv append=true
```
`append=true` appends new rows each run — the file grows unboundedly.

**Correct:**
```spl
index=web_logs earliest=-1d@d latest=@d
| stats count by client_ip
| sort -count
| outputlookup ip_counts.csv
```
Without `append=true`, the file is replaced on each run. Schedule daily.

**Notes:**
- `outputlookup` writes to `$SPLUNK_HOME/etc/apps/<app>/lookups/` by default.
- `outputlookup <lookup_name>` — uses the lookup stanza defined in `transforms.conf`.
- `outputlookup <filename>.csv` — writes directly to the app's lookups directory.
- **`createinapp=true`** — write to the current app's lookups directory.
- **`max=N`** — limit rows written (default: no limit).
- **KV Store:** `outputlookup my_kv_collection` writes to a KV store collection.
  KV stores support concurrent reads/writes and partial updates.
- Scheduled searches writing to lookups are a common pattern for
  "pre-computed" data: run expensive aggregation daily, query fast lookup at search time.

---

## Macros

### Defining and Using Search Macros [HIGH]

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

### Search Macro Best Practices [HIGH]

Use macros to abstract index names, common filters, and repeated SPL
patterns. This prevents hard-coded values from spreading across dashboards.

**Incorrect (hard-coded index in every search):**
```spl
index=prod_web_logs_v2 sourcetype=access_combined status>=400
```
When the index is renamed or split, every search must be updated.

**Correct (index abstracted into a macro):**
Macro `web_index`:
```
index=prod_web_logs_v2 sourcetype=access_combined
```
Usage:
```spl
`web_index` status>=400
```

**Notes:**
- **DRY principle:** Define once in a macro; reference everywhere.
  Index names, sourcetype filters, common eval expressions.
- **Argument validation:** Use the `validation` field in `macros.conf`
  to reject invalid arguments: `isint($threshold$)` rejects non-integer values.
- **Shared macros:** Place macros in a shared app (e.g., `search`) to make
  them available across all apps. App-specific macros stay in the app.
- **Documenting macros:** Add description in `macros.conf` — visible in
  Splunk Web's macro editor and to other developers.
- **Testing macros:** Expand inline to verify: Settings > Advanced Search > Search Macros.
- Avoid macros for one-off searches — the abstraction cost outweighs the benefit.

### Macros with Generating Commands [HIGH]

When a macro starts with a generating command (`search`, `inputlookup`,
`tstats`, `from`), it must be preceded by `|` (pipe) at the call site.

**Incorrect:**
```spl
`get_blocked_ips`
| stats count by host
```
If `get_blocked_ips` expands to `inputlookup blocklist.csv`, the search
string becomes `inputlookup blocklist.csv | stats count by host` — valid
only as a standalone search, not when appended to another search.

**Correct (macro used as a standalone search):**
```spl
`get_blocked_ips`
| lookup server_metadata.csv ip OUTPUT country
```

**Correct (macro used as a subsearch generating command):**
```spl
index=web_logs [
  `get_blocked_ips`
  | return 1000 ip
]
```

**Correct (macro preceded by pipe when generating in a pipeline):**
```spl
index=web_logs
| append [`get_blocked_ips`]
```

**Notes:**
- `max_macro_depth = 100` in `limits.conf [search]` — limits recursive macro expansion.
- `max_subsearch_depth = 8` applies when macros are used inside subsearches.
- A generating command macro that starts with `search` can appear at the
  beginning of a search string without a pipe.
- Macros that start with `|` (transforming commands) must always be
  preceded by a pipe at the call site:
  ```
  Macro `add_geo`: | lookup geo.csv ip OUTPUT country
  Usage: index=web_logs `add_geo` | stats count by country
  ```

---

## Transaction and Iteration

### transaction vs stats — When to Use Each [HIGH]

Use `stats` by default. Use `transaction` only when multi-event grouping
with time/count constraints across events is required.

**Incorrect (using transaction for simple aggregation):**
```spl
index=web_logs
| transaction session_id
| stats count avg(duration) by host
```
`transaction` materializes full event text for every group — expensive.
`stats` does not need full event text.

**Correct (stats for aggregation):**
```spl
index=web_logs
| stats count avg(response_time) as avg_rt by session_id, host
```

**When transaction IS correct:**
```spl
index=web_logs
| transaction session_id startswith="GET /login" endswith="POST /logout" maxspan=30m
| eval session_duration = duration
| stats avg(session_duration) by host
```
`startswith`/`endswith` logic and time-bounded grouping cannot be
replicated with `stats`.

**Notes:**
- `transaction` keeps `_raw` for all grouped events — high memory usage.
- `stats` uses parallel reduce on indexers; `transaction` cannot.
- `transaction` adds `duration`, `eventcount`, `closed_txn` fields.
- Max events per transaction: `maxevents=1000` (default, `limits.conf [transaction]`).
- Max span: `maxspan` — no default limit (can span entire index).
- See: `transaction-patterns.md` for parameter reference.

### transaction Command Parameter Patterns [MEDIUM]

`transaction` groups events by field equality plus optional start/end
markers and time/count constraints.

**Basic grouping by field:**
```spl
index=web_logs
| transaction session_id maxspan=30m maxpause=5m
```
Groups events with the same `session_id` that occur within 30 minutes
and have no more than 5 minutes between consecutive events.

**Start/end marker grouping:**
```spl
index=app_logs
| transaction user startswith="login" endswith="logout" maxspan=8h
| where NOT open_ended
```
`open_ended=true` on incomplete transactions (no end event).

**Multi-field grouping:**
```spl
index=web_logs
| transaction session_id, src_ip maxspan=1h
```
All fields listed must match to group events together.

**Notes:**
- `startswith=eval(expr)` / `endswith=eval(expr)` — eval expression version:
  `startswith=eval(action="start")`.
- **Silent data loss:** When a transaction exceeds `maxspan`, `maxevents`, or
  `maxopentxn`, it is **evicted (completely dropped)** from results by default —
  not truncated. This is the most common cause of "missing transactions."
- `keepevicted=true` — retain evicted (partial) transactions in results.
  Evicted transactions have `closed_txn=0` so you can identify them.
- `keeporphans=true` — include events that never matched a start condition.
- `mvlist=true` — store grouped field values as multivalue list.
- `maxevents=N` — max events per transaction (default 1000, `limits.conf [transaction]`).
- `maxopentxn=N` — max open transactions tracked simultaneously (default 100k).
- `connected=false` — disable pairing by proximity; each `startswith` opens a new transaction.

### foreach Command — Iteration Modes [HIGH]

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

### map Command — Iterative Search Execution [HIGH]

`map` executes a search for each result row, substituting field values
as tokens. Results are unioned into a single result set.

**Correct:**
```spl
index=web_logs status=500
| stats count by host
| map search="search index=web_logs host=$host$ | head 5"
```
Runs a separate search per `host` value, returning up to 5 events each.

**Notes:**
- **Token syntax:** `$fieldname$` substitutes the field value into the search string.
  `$_serial_id$` is the 0-based row number.
- **`maxsearches=N`** (default 10): Max iterations. `0` is **NOT** unlimited —
  it means 0 searches execute (bug-prone default). Set explicitly.
- **Cannot follow `append`/`appendpipe`** — `map` must be the first command
  after `|` in a pipeline that receives results.
- **Risky command:** `map` triggers Splunk's "risky command" safeguard in the UI.
  Users must confirm before execution. Disable per-user or role in `authorize.conf`:
  `srchFilter = NOT (map)`
- **Performance:** Each iteration is a full search. With `maxsearches=100` and
  slow subsearches, runtime multiplies by 100×.
- Use `| join` or `| lookup` for enrichment instead of `| map` when possible.

---

## Data Models

### Data Model Basics [HIGH]

Data models define hierarchical event datasets for pivot-based searches
and accelerated `tstats` queries.

**Search using datamodel command:**
```spl
| datamodel Authentication Authentication search
| fields Authentication.action, Authentication.user, Authentication.src
```
`datamodel <ModelName> <DatasetName> search` returns raw events matching
the dataset constraints.

**Pivot against a data model:**
```spl
| pivot Authentication Authentication count(Authentication) AS count SPLITROW Authentication.action PERIOD latest
```

**From syntax (SPL2-style):**
```spl
| from datamodel:Authentication.Authentication
| search Authentication.action=failure
```

**Notes:**
- Data model hierarchy: root dataset → child datasets (each child adds constraints).
- `| datamodel` returns fields prefixed with `ModelName.` — use `rename` or CIM add-on field aliases.
- `strict_fields=true` — restricts results to fields defined in the data model.
- Data model permissions: private → shared → global via Settings > Data Models.
- CIM (Common Information Model): standardizes field names across sourcetypes.
  See: `datamodel-cim.md`.

### Data Model Acceleration [HIGH]

Accelerated data models pre-build tsidx summary files for `tstats` queries,
dramatically reducing search time for large datasets.

**Without acceleration (slow):**
```spl
| datamodel Authentication Authentication search
| stats count by Authentication.action
```

**With acceleration (fast via tstats):**
```spl
| tstats summariesonly=true count FROM datamodel=Authentication.Authentication BY Authentication.action
```

**Notes:**
- Enable acceleration: Settings > Data Models > Edit > Accelerate.
- `summariesonly=true` — only use pre-built summaries; returns no results
  if summaries are incomplete. Use for performance-critical dashboards.
- `summariesonly=false` — uses summaries + live data to fill gaps. Slower
  but more complete.
- `allow_old_summaries=true` — use summaries even after schema changes.
  Risk: fields renamed/removed in the model may be stale.
- `nodename=Root.Child.GrandChild` — target a specific dataset node.
- **Role-based search filters** are NOT applied to accelerated data models.
  Use object-level permissions to restrict access instead.
- Acceleration time range controlled by `datamodel.conf` `acceleration.earliest_time`.

### CIM (Common Information Model) Field Conventions [MEDIUM]

CIM standardizes field names across sourcetypes so that searches and
dashboards work without per-sourcetype customization.

**Without CIM (sourcetype-specific):**
```spl
index=firewall sourcetype=palo_alto_networks
| stats count by src_ip, dst_port
```
Different field names per sourcetype break cross-sourcetype searches.

**With CIM (normalized):**
```spl
| from datamodel:Network_Traffic.Network_Traffic
| stats count by src, dest_port
```
CIM-compliant fields: `src`, `dest`, `src_port`, `dest_port`, `user`, `action`.

**Key CIM field names by category:**
- **Authentication:** `action` (success/failure), `user`, `src`, `dest`, `app`
- **Network Traffic:** `src`, `dest`, `src_port`, `dest_port`, `transport`, `bytes_in`, `bytes_out`
- **Web:** `uri_path`, `uri_query`, `http_method`, `status`, `bytes`, `src`, `dest`
- **Endpoint:** `dest`, `user`, `process`, `process_id`, `file_path`, `registry_path`

**Notes:**
- CIM add-ons provide field aliases and transforms that map sourcetype fields to CIM names.
- Install the Splunk Common Information Model Add-on from Splunkbase.
- `tag` field is CIM's primary classification mechanism: `tag=authentication`, `tag=network`.
- Cross-sourcetype correlation: normalize to CIM fields first, then correlate.

---

## tstats Command

### tstats Command Basics [CRITICAL]

`tstats` queries indexed fields directly from tsidx files without loading
raw events — the fastest aggregation method in Splunk.

**Incorrect (using stats on large dataset when tstats would work):**
```spl
index=web_logs sourcetype=access_combined
| stats count by host, status
```

**Correct (tstats for indexed fields):**
```spl
| tstats count WHERE index=web_logs sourcetype=access_combined BY host, status
```
Runs directly on index metadata — no event loading, no field extraction.

**Notes:**
- **Indexed fields only:** `tstats` works on `index`, `host`, `source`,
  `sourcetype`, `_time`, and fields in `fields.conf` with `INDEXED=true`.
  Search-time extracted fields (from `props.conf`/`transforms.conf`) are NOT available.
- **tstats vs stats comparison:**
  | Feature | tstats | stats |
  |---------|--------|-------|
  | Field scope | Indexed only | All extracted |
  | Speed | Very fast | Depends on volume |
  | Parallelism | Full | Parallel reduce |
  | Data models | Yes (summariesonly) | No |
- `fillnull_value="NULL"` — replace null field values with a string.
- `local=true` — run locally on search head (debug mode, skips indexers).
- `prestats=true` — emit pre-aggregation rows for `append` patterns.

### tstats with Data Models [HIGH]

`tstats` on accelerated data models is the fastest way to query structured
event data at scale.

**Basic data model query:**
```spl
| tstats summariesonly=true count FROM datamodel=Authentication.Authentication
  WHERE Authentication.action=failure
  BY Authentication.user, Authentication.src span=1h
```

**Target a child dataset with nodename:**
```spl
| tstats summariesonly=true count FROM datamodel=Authentication
  WHERE nodename=Authentication.Failed_Authentication
  BY Authentication.user span=1d
```

**Notes:**
- `FROM datamodel=ModelName.DatasetName` — dot notation targets the dataset.
  `FROM datamodel=ModelName` with `WHERE nodename=` is equivalent.
- `summariesonly=true` — only accelerated data (fast, may miss recent gaps).
- `summariesonly=false` — accelerated + live data (complete, slightly slower).
- `allow_old_summaries=true` — use summaries after schema changes.
- **WHERE clause:** Only indexed fields and data model attributes can filter.
  Search-time extractions are not available in tstats WHERE.
- `span=` — time grouping granularity for `BY _time span=Xm`.
- Results use prefixed field names: `Authentication.user` (not `user`).
  Rename with `| rename Authentication.user AS user`.

### tstats PREFIX() for Raw Indexed Terms [MEDIUM]

`PREFIX(key=)` in `tstats` accesses raw indexed segments (terms from
`_raw`) without defining explicit indexed fields.

**Correct:**
```spl
| tstats count WHERE index=web_logs PREFIX(uri_path=/api/) BY host
```
Counts events where `_raw` contains the term `uri_path=/api/` (as
a continuous indexed segment — no major breakers within the term).

**Discover available terms:**
```spl
| walklex index=web_logs prefix="status=" type=term
| table term, count
```
`walklex` enumerates the tsidx lexicon to find indexed terms.

**Notes:**
- **Lowercase requirement:** `PREFIX(key=)` values are always lowercased
  for matching. `PREFIX(Status=200)` will NOT match `Status=200` in events
  if the term is indexed as `status=200`.
- **No major breakers:** The term must appear as a continuous segment.
  Splunk's major breakers (space, `[`, `]`, `(`, `)`, `{`, `}`, `!`, etc.)
  split terms. `uri=/api/v1/users` is split at `/`.
- **collect compatibility:** When using `| collect` after `tstats` with PREFIX,
  set `collect_ignore_minor_breakers=true` in `limits.conf [collect]`.
- **Performance:** PREFIX scans are faster than full `search` but slower
  than explicit indexed field queries.

### tstats prestats Mode [MEDIUM]

`prestats=true` emits raw pre-aggregation rows instead of final
aggregated results, enabling `| append` to combine multiple `tstats`
queries before final aggregation.

**Multi-timerange union pattern:**
```spl
| tstats prestats=true count WHERE index=web_logs earliest=-7d latest=-1d BY host
| append [
  | tstats prestats=true count WHERE index=web_logs earliest=-1d latest=now BY host
]
| stats count BY host
```
Combines two time windows before final `stats`.

**Notes:**
- `prestats=true` output is not human-readable — rows have internal
  format for consumption by `stats`.
- **No `AS` renaming** in prestats mode — field renaming in `BY` clause
  is not supported when `prestats=true`.
- `append=true` (different from `| append`) — appends rows to existing
  result set without clearing it (used with iterative tstats calls).
- Final `| stats` (not `| tstats`) must aggregate the prestats output.
- Useful for merging: different indexes, different time windows, different
  data models into one aggregation.

---

## Summary Indexing

### Summary Indexing with collect [HIGH]

`| collect` writes search results back into a Splunk index for fast
re-query. Use `sourcetype=stash` to avoid license consumption.

**Correct (scheduled search writing to summary index):**
```spl
index=web_logs earliest=-15m latest=now
| stats count avg(response_time) as avg_rt by host status
| collect index=summary sourcetype=stash addinfo=true
```
Runs every 15 minutes. `sourcetype=stash` is exempt from license metering.

**Re-query the summary:**
```spl
index=summary sourcetype=stash earliest=-24h
| stats sum(count) as total_count avg(avg_rt) as overall_avg by host status
```

**Notes:**
- `addinfo=true` — stamps `info_min_time` and `info_max_time` fields
  on each written event (the time range of the originating search).
- `testmode=true` — dry run: shows what would be written without writing.
- `output_format=hec` — write in HEC JSON format for HEC-compatible indexes.
- `marker=key=value` — add custom metadata field to all collected events.
- `run_in_preview=false` — prevent collect from running in preview mode.
- `sistats → collect` pattern: `| sistats count by host | collect index=summary`
  uses sistats for partial aggregation then collect for storage.
- See: `summary-tscollect.md` for tsidx-based summary pattern.

### tscollect and Summary tsidx Files [MEDIUM]

`| tscollect` writes indexed field summaries (tsidx files) for later
`| tstats` re-query — faster than `| collect` for aggregation workloads.

**Write a summary:**
```spl
index=web_logs earliest=-1h latest=now
| stats count by host, status, _time span=5m
| tscollect index=summary_tsidx
```

**Re-query the summary with tstats:**
```spl
| tstats sum(count) WHERE index=summary_tsidx BY host, status span=1h
```

**Notes:**
- `tscollect` stores data as tsidx (time-series index) files — queryable
  only via `tstats`, not `search`.
- Fields written must be numeric or string — complex types not supported.
- `namespace=N` — namespace ID for managing summary tsidx files (default 0).
- **Use case:** Pre-aggregate high-volume data at 5-minute granularity,
  then query the summary at 1-hour granularity — orders of magnitude faster.
- Contrast with `collect` (writes to event index, queryable via `search`).
- Summary tsidx files are stored in `$SPLUNK_HOME/var/lib/splunk/<index>/db/`.

---

## Output and Formatting

### Exporting and Sending Results [MEDIUM]

`outputcsv`, `outputlookup`, `sendemail`, and `collect` write results
to files, lookups, email, or indexes.

**outputcsv (write to CSV):**
```spl
index=web_logs
| stats count by status
| outputcsv status_counts.csv
```
Writes to `$SPLUNK_HOME/var/run/splunk/csv/status_counts.csv`.

**outputlookup (write to lookup file):**
```spl
index=web_logs
| stats count by host
| outputlookup host_counts.csv
```
Persists lookup for use in subsequent `| lookup` calls.

**sendemail (alert email):**
```spl
index=web_logs status=500
| stats count by host
| sendemail to="ops@example.com" subject="Error summary" inline=true
```

**collect (summary indexing):**
```spl
index=web_logs
| stats count by host
| collect index=summary sourcetype=stash
```

**Notes:**
- `outputcsv append=true` — append to existing CSV instead of overwriting.
- `outputlookup createempty=false` — don't overwrite if result set is empty.
- `sendemail result_limit=10000` — max results included in email
  (configurable in `limits.conf [email]`).
- `sendemail format=csv` / `format=table` — attachment format.
- `outputlookup` vs `outputcsv`: `outputlookup` integrates with lookup
  configurations in `transforms.conf`; `outputcsv` writes directly.

### Output Formatting Commands [LOW]

`fieldformat`, `reltime`, `rangemap`, and `gauge` format values for
display without altering underlying data.

**fieldformat (display-only formatting):**
```spl
index=web_logs
| stats sum(bytes) as total_bytes by host
| fieldformat total_bytes = tostring(total_bytes, "commas")
```
`total_bytes` displays as `1,234,567` in the UI but retains numeric
value for further calculations.

**reltime (human-readable time):**
```spl
index=web_logs
| head 100
| reltime
| table _time, reltime, host
```
Adds `reltime` field: "5 minutes ago", "2 hours ago".

**rangemap (color-coded ranges):**
```spl
index=metrics
| stats avg(response_time) as avg_rt by host
| rangemap field=avg_rt low=0-200 elevated=201-500 high=501-1000 default=severe
```
Adds `range` field with values: `low`, `elevated`, `high`, `severe`.

**gauge:**
```spl
index=metrics
| stats avg(cpu_pct) as cpu by host
| gauge cpu 0 50 80 100
```

**Notes:**
- `fieldformat` only affects display — does NOT change the value for `where` or `eval`.
- `reltime` uses `_time` field; ensure `_time` is present before calling.
- `rangemap default=` — the range applied when value falls outside all defined ranges.

### Output Reshaping Commands [MEDIUM]

`transpose`, `untable`, and `xyseries` reshape tabular results for
visualization and further processing.

**transpose (rows → columns):**
```spl
index=web_logs
| stats count by status, host
| transpose 5 column_name=host header_field=status
```
Pivots: status values become column headers, hosts become rows (up to 5 columns).

**xyseries (stat table → matrix):**
```spl
index=web_logs
| stats count by host, status
| xyseries host status count
```
Produces a matrix: hosts as rows, status codes as column headers.

**untable (matrix → rows — inverse of xyseries):**
```spl
index=web_logs
| stats count by host, status
| xyseries host status count
| untable host status count
```

**Notes:**
- `transpose N` — limits output to N columns (default unlimited).
- `transpose includeempty=true` — include null-valued fields.
- `xyseries` is the inverse of `timechart`: converts rows to columns.
- `untable field1 field2 value` — specify row key, column key, value field names.
- All three are non-distributable streaming commands — run on search head.

### Set Operations — union, diff, intersect [MEDIUM]

`| set` performs set operations on two result sets. Results from a
subsearch are combined with the current pipeline using set logic.

**union (all events from both):**
```spl
index=web_logs status=500
| set union [search index=app_logs level=ERROR]
```
Returns all events from both searches (removes duplicates).

**diff (events in main but not in subsearch):**
```spl
index=web_logs
| stats count by host
| set diff [search index=maintenance | table host]
```
Returns hosts in web_logs not in maintenance list.

**intersect (events in both):**
```spl
index=web_logs
| stats count by host
| set intersect [search index=monitored | table host]
```
Returns only hosts present in both datasets.

**Notes:**
- `| set` requires both datasets to have the same fields.
- **`| append` + `| dedup` as alternative to set union:**
  ```spl
  index=web_logs | append [search index=app_logs] | dedup _raw
  ```
- **`selfjoin` pattern:**
  `index=web_logs | selfjoin session_id` — joins each row with other rows
  sharing the same `session_id`. Rarely needed; use `stats` instead.
- Set operations are non-distributable — run on the search head.
- Field ordering must match between both datasets for correct operation.

---

## Geographic Commands

### iplocation Command [MEDIUM]

`| iplocation` enriches events with geographic data (country, city,
latitude, longitude) based on an IP address field.

**Basic usage:**
```spl
index=web_logs
| iplocation src_ip
| stats count by Country
```
Adds `Country`, `City`, `Region`, `lat`, `lon` fields for each `src_ip`.

**Get all available fields:**
```spl
index=web_logs
| iplocation allfields=true src_ip
| table src_ip, Country, Region, City, lat, lon, MetroCode, Timezone
```

**Incorrect (wrong field name):**
```spl
index=web_logs
| iplocation ip_address
```
Default field for iplocation is `clientip`. Specify field name explicitly.

**Notes:**
- Default input field: `clientip` (if no field argument given).
- Output fields: `Country`, `Region`, `City`, `lat`, `lon` (always added).
- `allfields=true` — also adds `MetroCode`, `Timezone`, `DmaCode`.
- `lang=en` — language for location names (default English).
- External DB: Configure `iplookups.conf` to point to a custom MaxMind DB.
- Events with RFC1918 (private) or IPv6 addresses return null geo fields.
- See: `geo-geostats.md` for map visualization.

### geostats and geom Commands [MEDIUM]

`geostats` and `geom` power Splunk's cluster map and choropleth map
visualizations.

**Cluster map (geostats):**
```spl
index=web_logs
| iplocation src_ip
| geostats latfield=lat longfield=lon count by status
```
Renders a cluster map: circles sized by count, colored by status breakdown.

**Choropleth map (geom):**
```spl
index=web_logs
| iplocation src_ip
| stats count by Country
| geom geo_countries featureIdField=Country
```
Colors countries on a world map by count.

**Filter by bounding box (geomfilter):**
```spl
index=web_logs
| iplocation src_ip
| geomfilter min_lat=30 max_lat=60 min_lon=-10 max_lon=40
| geostats latfield=lat longfield=lon count
```

**Notes:**
- `geostats globallimit=N` — max distinct values of `by` field rendered (default 100).
- `geostats maxzoomlevel=N` — controls aggregation granularity at zoom levels.
- `geom geo_countries` — built-in country geometry lookup.
  Other built-ins: `geo_us_states`, `geo_us_counties`.
- Custom KML/KMZ geometry files can be added via lookup configuration.
- `geomfilter` clips events outside a lat/lon bounding box.

---

## Anomaly and Prediction

### Anomaly Detection Commands [MEDIUM]

Splunk provides three anomaly commands: `anomalydetection` (probability-based),
`anomalies`/`anomalousvalue` (unexpectedness scoring), and `outlier` (axis cleanup).

**anomalydetection (probability-based):**
```spl
index=web_logs
| stats count by host, status
| anomalydetection action=annotate
```
Adds `anomaly_score` and `is_anomaly` fields. `action=annotate` marks events;
`action=filter` removes normal events.

**anomalousvalue (field-level unexpectedness):**
```spl
index=web_logs
| anomalousvalue pthresh=0.02 action=annotate
```
Flags individual field values with unexpectedness score above threshold.

**outlier (remove chart outliers):**
```spl
index=metrics
| timechart avg(cpu_pct) by host
| outlier action=remove
```
Removes data points that would skew chart axes (not anomaly detection).

**Notes:**
- `anomalydetection` builds a frequency model per field value — memory intensive
  on high-cardinality fields.
- `pthresh=0.02` — probability threshold; values below this are anomalous.
- `action=annotate` — keeps all events, adds score fields.
- `action=filter` — removes non-anomalous events (for alert use cases).
- `outlier` uses IQR (interquartile range) by default: `param.IQRMultiplier=1.5`.

### Clustering Commands — cluster and kmeans [MEDIUM]

`cluster` groups similar raw events; `kmeans` applies k-means clustering
to numeric fields.

**cluster (text similarity grouping):**
```spl
index=app_logs level=ERROR
| cluster field=_raw t=0.8
| stats count by cluster_label
```
Groups error messages with ≥80% similarity. `cluster_label` names each group.

**kmeans (numeric field clustering):**
```spl
index=metrics
| stats avg(cpu_pct) as cpu, avg(mem_pct) as mem by host
| kmeans k=5 reps=10 dt_thresh=0.001
| table host, cluster_id
```
Groups hosts into 5 clusters by CPU/memory profile.

**Notes:**
- `cluster t=N` — similarity threshold (0.0–1.0). Higher = more distinct clusters.
- `cluster showcount=true` — add `cluster_count` field (number of events in cluster).
- `kmeans k=N` — number of clusters (required). Choose based on domain knowledge.
- `kmeans reps=N` — number of random initializations (default 1). Higher = better
  but slower. Use `reps=10` for more stable results.
- `kmeans dt_thresh=N` — convergence threshold (default 0.0001).
- Both commands are non-distributable — run on search head with full result set.
- Reduce data volume with `sample` or aggregate with `stats` before clustering.

### predict and trendline Commands [MEDIUM]

`predict` uses ML to forecast time series; `trendline` calculates moving
averages; `x11` performs seasonal decomposition.

**predict (ML forecasting):**
```spl
index=metrics
| timechart span=1h avg(cpu_pct) as cpu
| predict cpu AS predicted algorithm=LLP5 future_timespan=24 upper95=upper lower95=lower
```
Forecasts 24 future time periods using LLP5 (Local Level Prediction with 5-period cycle).

**trendline (moving average):**
```spl
index=metrics
| timechart span=1h avg(cpu_pct) as cpu
| trendline sma5(cpu) AS trend_sma wma5(cpu) AS trend_wma ema5(cpu) AS trend_ema
```
`sma5` = 5-period simple moving average; `wma` = weighted; `ema` = exponential.

**x11 (seasonal decomposition):**
```spl
index=metrics
| timechart span=1h avg(cpu_pct) as cpu
| x11 cpu
```
Adds `XTrend`, `XSeasonal`, `XResidual` fields.

**Notes:**
- `predict algorithm=LLP` — Local Level Prediction (trend only).
  `LLP5` = LLP with 5-period seasonality. `LL` = no trend.
- `predict holdback=N` — withhold last N points for validation.
- `trendline` period range: 2–10000.
- `x11` requires a contiguous time series (no gaps).
- All three commands require output from `timechart` as input (sorted time series).

---

## Metrics

### Metrics Index and mstats Command [HIGH]

Splunk metrics indexes store numeric time series data more efficiently
than event indexes. Query them with `mstats`.

**Query metrics with mstats:**
```spl
| mstats avg(cpu.percent) WHERE index=metrics BY host span=1m
```
Directly queries metric data points. `cpu.percent` is the metric name.

**Multiple metrics:**
```spl
| mstats avg(cpu.percent) AS cpu, avg(mem.percent) AS mem
  WHERE index=metrics BY host span=5m
  latest=-1h earliest=-2h
```

**Write to metrics index:**
```spl
index=web_logs
| stats avg(response_time) AS response_time BY host _time
| eval metric_name="web.response_time"
| mcollect index=mymetrics prefix_field=metric_name
```

**Inspect raw metric data:**
```spl
| mpreview index=metrics | head 10
```

**Notes:**
- Metric index vs event index: metrics store only `_time`, dimensions (string
  fields), and metric values (numbers). No `_raw` field.
- `mstats` aggregation functions: `avg`, `min`, `max`, `sum`, `count`, `stdev`, `perc`.
- `WHERE` clause: filter by dimensions (string fields), not metric values.
  Use `| where` after `mstats` for value filtering.
- `fillnull_value=0` — fill missing time buckets with 0 instead of null.
- `mcollect` batch size: `batch_size=1000` (default, `limits.conf [metrics]`).

---

## CLI and REST API

### Splunk CLI Search Basics [MEDIUM]

The `./splunk search` command runs SPL queries from the command line.

**Basic search:**
```bash
./splunk search 'index=web_logs status=500 | stats count by host' \
  -earliest_time -24h -latest_time now
```

**Real-time search:**
```bash
./splunk rtsearch 'index=web_logs | stats count by status' \
  -earliest_time rt-30s -latest_time rt
```

**Output to file:**
```bash
./splunk search 'index=web_logs | stats count by host' \
  -earliest_time -24h -output csv > results.csv
```

**Notes:**
- **No default time range:** Without `-earliest_time`/`-latest_time`, the
  search runs as All Time — scans everything.
- **Default `maxout=100`** — only 100 results returned by default.
  Override with `-maxout 10000` (or 0 for unlimited within `limits.conf`).
- **Quote handling:** Linux/macOS: single quotes around SPL. Windows: double
  quotes. Nested quotes in SPL require escaping.
- `./splunk` is at `$SPLUNK_HOME/bin/splunk`.
- Must be run as the Splunk OS user or with `sudo -u splunk`.
- See: `cli-parameters.md` for full parameter reference.

### Splunk CLI Search Parameters Reference [LOW]

Full parameter reference for `./splunk search` and `./splunk rtsearch`.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `-app` | `search` | App context for search |
| `-auth` | _(required if not saved)_ | `username:password` |
| `-detach` | `false` | Return job SID immediately, don't wait |
| `-earliest_time` | _(none = All Time)_ | Start of time range |
| `-latest_time` | _(none = now)_ | End of time range |
| `-max_time` | `0` (unlimited) | Max seconds to run |
| `-maxout` | `100` | Max events returned |
| `-output` | `auto` | Output format: `auto`, `csv`, `json`, `xml`, `raw` |
| `-preview` | `true` | Stream preview results during search |
| `-timeout` | `0` (unlimited) | Job expiration timeout in seconds |
| `-uri` | `localhost:8089` | Management port URI |
| `-wrap` | `true` | Wrap long lines in terminal output |

**Example with multiple parameters:**
```bash
./splunk search 'index=web_logs | stats count by host' \
  -earliest_time -7d \
  -latest_time now \
  -maxout 0 \
  -output csv \
  -max_time 300 \
  -auth admin:changeme
```

**Notes:**
- `-maxout 0` means use system limit from `limits.conf [search] max_count`.
- `-detach true` returns a SID; use `./splunk search -sid <SID>` to retrieve results.
- `-output raw` outputs `_raw` only (no CSV headers or JSON wrapping).

### Splunk CLI Real-Time Search [MEDIUM]

`./splunk rtsearch` executes a real-time search that streams results
as events arrive.

**Windowed real-time (30-second window):**
```bash
./splunk rtsearch 'index=web_logs status=500 | stats count by host' \
  -earliest_time rt-30s \
  -latest_time rt
```
Returns results within a sliding 30-second window.

**Unbounded real-time:**
```bash
./splunk rtsearch 'index=web_logs status=500' \
  -earliest_time rt \
  -latest_time rt
```
Streams all incoming events matching the filter.

**Async detached search:**
```bash
./splunk rtsearch 'index=web_logs | stats count by host' \
  -earliest_time rt-5m -latest_time rt \
  -detach true
# Returns SID. Retrieve results with:
./splunk search -sid <SID>
```

**Notes:**
- **`earliest_time` and `latest_time` are REQUIRED** for `rtsearch` — no defaults.
- **RT time syntax:** `rt` = now (real-time); `rt-30s` = 30 seconds before now;
  `rt+30s` = 30 seconds in the future.
- **Windowed RT searches** maintain a rolling time window — good for dashboards.
- **Unbounded RT** (`rt` to `rt`) streams all incoming events — no historical data.
- Real-time searches consume more resources than historical — use sparingly.
- `detach=true` allows the search to continue after CLI disconnects.

### Splunk REST API Authentication [HIGH]

The Splunk REST API supports Basic Auth and Session Token authentication.

**Basic Auth:**
```bash
curl -k -u admin:changeme \
  https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs | head 10"
```

**Session Token (recommended for multi-request scripts):**
```bash
# Step 1: Get session token
TOKEN=$(curl -sk -u admin:changeme \
  https://localhost:8089/services/auth/login \
  -d username=admin -d password=changeme \
  | grep -o '<sessionKey>[^<]*' | sed 's/<sessionKey>//')

# Step 2: Use token
curl -k -H "Authorization: Splunk $TOKEN" \
  https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs | head 10"
```

**Notes:**
- **`-k` flag** — skip SSL certificate verification for self-signed certs.
  For production, use proper certificates and remove `-k`.
- **Session token TTL:** Default 1 hour (configurable in `web.conf` `sessionTimeout`).
- **Token auth also supported:** Splunk authentication tokens (Settings > Tokens)
  for long-lived API access without username/password.
  Header: `Authorization: Bearer <token>`.
- Management port: default `8089` (configured in `server.conf`).
- REST API base URL: `https://splunk-host:8089/services/`.

### Creating Search Jobs via REST API [HIGH]

`POST /services/search/jobs` creates an async search job. Poll
`dispatchState` until `DONE` before fetching results.

**Create a job:**
```bash
SID=$(curl -sk -u admin:changeme \
  https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs status=500 | stats count by host" \
  -d earliest_time="-24h" \
  -d latest_time="now" \
  -d status_buckets=300 \
  | grep -o '<sid>[^<]*' | sed 's/<sid>//')
echo "SID: $SID"
```

**Poll until DONE:**
```bash
while true; do
  STATE=$(curl -sk -u admin:changeme \
    "https://localhost:8089/services/search/jobs/$SID" \
    | grep -o 'dispatchState[^<]*' | head -1)
  echo "State: $STATE"
  [[ "$STATE" == *"DONE"* ]] && break
  sleep 2
done
```

**Notes:**
- **Dispatch states:** `QUEUED` → `PARSING` → `RUNNING` → `FINALIZING` → `DONE`.
  Also: `FAILED`, `PAUSED`.
- **ALWAYS set `earliest_time`/`latest_time`** — default is All Time (scans everything).
- **`status_buckets=300`** — required for timeline and event distribution access.
  Without it, `/timeline` and `/summary` endpoints return empty.
- `output_mode=json` on the job status endpoint for JSON responses.
- `exec_mode=oneshot` — synchronous execution, returns results directly without polling.
  Use only for fast searches.
- See: `rest-retrieve-results.md` for fetching results after DONE.

### Retrieving Search Results via REST API [HIGH]

Use `/results` for post-completion results. Use `/events` during the search
for preview. Paginate large result sets with `offset` and `count`.

**Fetch results (after DONE):**
```bash
curl -sk -u admin:changeme \
  "https://localhost:8089/services/search/jobs/$SID/results" \
  -d output_mode=json \
  -d count=100 \
  -d offset=0
```

**Paginate through all results:**
```bash
OFFSET=0; COUNT=1000
while true; do
  RESULTS=$(curl -sk -u admin:changeme \
    "https://localhost:8089/services/search/jobs/$SID/results?output_mode=json&count=$COUNT&offset=$OFFSET")
  NRESULTS=$(echo "$RESULTS" | python3 -c "import sys,json; d=json.load(sys.stdin); print(len(d['results']))")
  echo "$RESULTS"
  [ "$NRESULTS" -lt "$COUNT" ] && break
  OFFSET=$((OFFSET + COUNT))
done
```

**Notes:**
- `/results` — available only after `dispatchState=DONE`. Returns transformed results.
- `/events` — available during and after search. Returns raw matched events.
- `output_mode` options: `json` (default: `xml`), `csv`, `xml`.
- `max_count=10000` — default result limit. Increase in request or via `limits.conf`.
- `count=0` — returns up to `max_count` results (not truly unlimited).
- **Field filtering:** `f=field1&f=field2` — return only specified fields.
- See: `rest-export.md` for streaming large result sets without a job SID.

### REST API Export Endpoint [MEDIUM]

`GET /services/search/jobs/export` streams search results without creating
a persistent job — ideal for large result sets.

**Stream results directly to file:**
```bash
curl -sk -u admin:changeme \
  "https://localhost:8089/services/search/jobs/export" \
  -d search="search index=web_logs status=500 earliest=-24h latest=now | fields host, status, uri" \
  -d output_mode=csv \
  > results.csv
```

**Stream JSON:**
```bash
curl -sk -u admin:changeme \
  "https://localhost:8089/services/search/jobs/export" \
  -d "search=search index=web_logs | stats count by host" \
  -d output_mode=json \
  -d earliest_time="-1h" \
  -d latest_time="now"
```

**Notes:**
- **No SID created** — results stream directly without a persistent job.
  Cannot resume or poll; must consume the full stream.
- **`output_mode`:** `csv`, `json`, `xml` (default xml), `raw`.
- **ALWAYS set `earliest_time`/`latest_time`** — no default time range.
- **No `max_count` limit** — returns all matching results (unlike `/results`
  which defaults to 10,000).
- **Timeout:** Set `max_time=300` to cap execution time.
- **`search` parameter** must include the leading `search` keyword or SPL command.
- Use for ETL pipelines, data exports, and large batch retrievals.
- Vs `/results`: export streams (no persistent job); `/results` requires completed job.

---

## SPL2

### SPL2 from and into — Entry and Exit Points [HIGH]

`from` replaces the leading `search` command in SPL2. `into` writes
results to a dataset.

**SPL1 equivalent:**
```spl
search index=web_logs sourcetype=access_combined status=500
| stats count by host
```

**SPL2 from syntax:**
```spl2
from main
| where sourcetype="access_combined" AND status=500
| stats count() AS count BY host
```

**from datamodel:**
```spl2
from datamodel:Authentication.Authentication
| where action="failure"
| stats count() AS failures BY user
```

**into (write results):**
```spl2
from main
| stats count() AS count BY host
| into summary
```
Writes to the `summary` dataset (index).

**Notes:**
- `from <index>` — search an event index by name (no `index=` prefix).
- `from datamodel:<Name>.<Dataset>` — query a data model dataset.
- `from lookup:<name>` — read from a lookup file (in `$default` namespace).
- `from lookup:<module>.<name>` — read from a lookup in a specific module namespace.
- `into <index>` — write results to an index (equivalent to `| collect`).
- `from` requires explicit time range via `| where _time > relative_time(now(), "-24h")`.
- Default output of `from`: all events in the dataset (equivalent to `search *`).

### SPL2 Data Types [MEDIUM]

SPL2 has explicit built-in data types. Type errors are caught at parse
time rather than producing silent null values.

**Built-in types:**
| Type | Description | Example |
|------|-------------|---------|
| `string` | Text value | `"hello"` |
| `number` | Numeric (int or float) | `42`, `3.14` |
| `bool` | Boolean | `true`, `false` |
| `ip` | IP address | `192.168.1.1` |
| `timestamp` | Date/time value | `2024-01-01T00:00:00Z` |

**Type detection with typer:**
```spl2
from main
| typer
| where _type_src_ip="ip"
```
`typer` adds `_type_<fieldname>` fields identifying the detected type.

**Type casting:**
```spl2
from main
| eval port_num = tonumber(port)
| eval ts = totime(timestamp_str, "%Y-%m-%dT%H:%M:%S")
```

**Notes:**
- SPL2 type system prevents silent coercion errors common in SPL1.
- `tonumber()`, `tostring()`, `tobool()`, `toip()`, `totime()` — explicit casts.
- **SPL1 comparison:** SPL1 uses implicit type coercion — `"100" > "99"` evaluates
  as string comparison (false), not numeric (true). SPL2 raises a type error instead.
- Custom type schemas can be defined in modules for structured data validation.
- `typer` command: `_type_<field>` values: `"string"`, `"number"`, `"bool"`, `"ip"`, `"null"`.

### SPL2 branch Command — Parallel Pipelines [MEDIUM]

`branch` splits the pipeline into parallel sub-pipelines, each operating
on the same input data. Results are merged.

**Parallel investigation:**
```spl2
from main
| where sourcetype="access_combined"
| branch
  [stats count() AS total_requests BY host]
  [where status=500 | stats count() AS errors BY host]
```
Both branches receive the same filtered events. Results are union-merged.

**Multi-path analysis:**
```spl2
from main
| where index="web_logs"
| branch
  [stats avg(response_time) AS avg_rt BY host | eval metric="avg_rt"]
  [stats max(response_time) AS max_rt BY host | eval metric="max_rt"]
  [stats count() AS requests BY host | eval metric="requests"]
| stats values(avg_rt) AS avg_rt, values(max_rt) AS max_rt, values(requests) AS requests BY host
```

**Notes:**
- Each branch in `[...]` receives an identical copy of the input dataset.
- Branches execute in parallel — faster than sequential SPL1 subsearches.
- Result merging: branches are appended (union), not joined.
- **No SPL1 equivalent** — closest approximation: multiple `| append` subsearches
  (sequential, not parallel).
- `route` command (SPL2): like `branch` but routes events to different
  branches based on a condition (events are NOT duplicated).

### SPL2 Lambda Expressions [MEDIUM]

SPL2 supports lambda expressions for inline function definitions in
higher-order functions.

**Lambda syntax:**
```spl2
lambda <param>: <expression>
lambda <param1>, <param2>: <expression>
```

**Using lambda with map function:**
```spl2
from main
| eval doubled_values = map(values_list, lambda x: x * 2)
```

**Lambda in custom function definition:**
```spl2
$define function transform(lst, fn) = map(lst, fn);

from main
| eval result = transform(my_list, lambda x: x + 1)
```

**Field template syntax `{fieldname}`:**
```spl2
from main
| eval msg = "Host {host} returned status {status}"
```
`{fieldname}` interpolates the field value into the string.

**Notes:**
- Lambda expressions are anonymous functions — no `$define` needed.
- Parameters are positional — no named parameters in lambdas.
- Lambdas can reference fields from the current event via closure.
- **SPL1 equivalent:** No direct equivalent. SPL1 uses `mvmap()` for
  multivalue field transformation: `| eval result = mvmap(mv_field, value * 2)`.
- Field templates `{field}` are SPL2-only — use `eval` + `"..."` concatenation in SPL1.

### SPL2 Modules [MEDIUM]

SPL2 modules (`.spl2` files) define reusable search components —
datasets, functions, and views — with explicit namespace management.

**Module file (my_searches.spl2):**
```spl2
$define namespace="myapp";

$define dataset web_errors = {
  from main
  | where sourcetype="access_combined" AND status>=500
};

$define function error_rate(total, errors) = (errors / total) * 100;
```

**Using a module in a search:**
```spl2
import myapp;

from web_errors
| stats count() AS total BY host
| eval rate = error_rate(total, count())
```

**Notes:**
- Module files use `.spl2` extension and are managed in the SPL2 Module Editor
  (Splunk Web: Settings > SPL2 Modules).
- `$define namespace="name"` — declares the module's namespace for imports.
- `$define dataset` — creates a reusable named dataset (view/CTE).
- `$define function` — creates a reusable scalar function.
- **Permissions:** Modules have private/app/global sharing like other knowledge objects.
- Modules replace saved searches and macros for SPL2-native workflows.
- SPL1 macros (`` `macroname` ``) are not available in SPL2 — use module functions.

### SPL2 Custom Functions [MEDIUM]

SPL2 allows defining custom eval functions and command functions in modules.

**Custom eval function:**
```spl2
$define function bytes_to_mb(bytes) = bytes / 1048576;

from main
| eval size_mb = bytes_to_mb(file_size)
```

**Custom function with multiple parameters:**
```spl2
$define function rate_per_hour(count, duration_seconds) =
  (count / duration_seconds) * 3600;

from main
| stats count() AS req_count, max(_time) - min(_time) AS duration BY host
| eval hourly_rate = rate_per_hour(req_count, duration)
```

**Documentation comments:**
```spl2
/*
 * Converts bytes to human-readable size string.
 * @param bytes - File size in bytes
 * @returns String like "1.2 MB" or "3.4 GB"
 */
$define function human_size(bytes) =
  case(bytes < 1024, tostring(bytes) + " B",
       bytes < 1048576, tostring(round(bytes/1024, 1)) + " KB",
       bytes < 1073741824, tostring(round(bytes/1048576, 1)) + " MB",
       true, tostring(round(bytes/1073741824, 2)) + " GB");
```

**Notes:**
- Custom functions are defined in `.spl2` module files (not inline in searches).
- **SPL1 equivalent:** Search macros with arguments: `` `mymacro(arg1, arg2)` ``.
- Functions are pure (no side effects) — they transform field values.
- Recursive functions are not supported.
- Custom command functions (pipeline-level) use `$define command` syntax.

### SPL2 spl1 Wrapper Command [HIGH]

`| spl1 "..."` embeds a legacy SPL1 pipeline inside an SPL2 search.
Use for SPL1 commands not yet available in SPL2.

**Embedding SPL1 commands in SPL2:**
```spl2
from main
| where sourcetype="access_combined"
| spl1 "transaction session_id maxspan=30m"
| stats count() AS sessions BY host
```
`transaction` is not yet in SPL2 — `spl1` wrapper executes it.

**Complex SPL1 subsearch:**
```spl2
from main
| spl1 "| inputlookup blocklist.csv | return 1000 ip"
```

**Notes:**
- `spl1 "..."` — the argument is a complete SPL1 pipeline string.
- The wrapper receives the SPL2 pipeline's current result set as input.
- SPL1 commands not yet in SPL2: `transaction`, `inputlookup`, `geostats`,
  `iplocation`, `anomalydetection`, `predict`, and others.
- **Check SPL2 command availability** before using the wrapper — the SPL2
  command set grows with each Splunk release.
- The wrapper adds overhead — avoid for high-frequency or performance-critical paths.
- `spl1` wrapper works in both Splunk Cloud and Enterprise (v9.4+).

---

## Migration SPL1 → SPL2

### SPL1 to SPL2 Syntax Differences [HIGH]

Key syntax changes when migrating from SPL1 to SPL2.

| Feature | SPL1 | SPL2 |
|---------|------|------|
| Entry point | `search index=web_logs` | `from main` |
| Aggregation | `stats count by host` | `stats count() AS count BY host` |
| Field reference | `host` (implicit) | `host` (same) |
| String literal | `"value"` or `value` | `"value"` (explicit) |
| Wildcard search | `status=5*` | `like(status, "5%")` |
| Case insensitive | `status=Error*` | `ilike(status, "error%")` |
| Macro | `` `mymacro` `` | `import mymodule; myfunction()` |
| Subsearch | `[ search index=... ]` | `| branch [...]` or `from` |
| Comment | No inline comments | `/* comment */` |
| Null check | `NOT field=*` | `isnull(field)` |

**SPL1:**
```spl
index=web_logs status=5*
| stats count by host, status
| sort -count
```

**SPL2 equivalent:**
```spl2
from main
| where like(status, "5%")
| stats count() AS count BY host, status
| sort count DESC
```

**Notes:**
- `stats` aggregation functions require explicit `()`: `count()`, `avg()`, `sum()`.
- `AS` renaming is required in SPL2 `stats` (no implicit field naming).
- `sort` uses `ASC`/`DESC` not `+`/`-`.
- Boolean operators: `AND`, `OR`, `NOT` (same as SPL1).
- `eval` syntax is mostly compatible with SPL1.

### SPL2 from vs SPL1 search — Equivalence Guide [HIGH]

Mapping between SPL1 `search index=` patterns and SPL2 `from` equivalents.

| SPL1 | SPL2 |
|------|------|
| `search index=main` | `from main` |
| `search index=web_logs sourcetype=access_combined` | `from main \| where sourcetype="access_combined"` |
| `search index=web_logs host=web01` | `from main \| where host="web01"` |
| `search index=_internal` | `from _internal` |
| `\| datamodel Authentication Authentication search` | `from datamodel:Authentication.Authentication` |
| `\| inputlookup geo_data.csv` | `from lookup:geo_data` |

**Time range in SPL2:**
```spl2
from main
| where _time > relative_time(now(), "-24h") AND _time <= now()
| stats count() AS count BY host
```

**Multiple indexes (SPL1):**
```spl
index=web_logs OR index=app_logs status=500
```

**Multiple datasets (SPL2):**
```spl2
from main, app_logs
| where status=500
```

**Notes:**
- SPL2 `from <name>` uses the index name directly (no `index=` prefix).
- `sourcetype`, `host`, `source` filtering moves to `| where` clause.
- SPL2 does not support leading wildcard index names: `from web_*` is not valid.
- Time range: SPL2 searches use the UI time picker OR explicit `| where _time` filters.

### SPL2 Availability and Compatibility [HIGH]

SPL2 is available on specific platforms and can coexist with SPL1 via
compatibility profiles and the `spl1` wrapper.

**Platform availability (as of v9.4):**
| Platform | Status |
|----------|--------|
| Splunk Cloud Platform (v9.4.2510+) | GA |
| Splunk Enterprise v9.4 (Linux) | GA |
| Splunk Enterprise v9.4 (Windows) | Not available |
| Splunk Enterprise < v9.4 | Not available |

**Enabling SPL2 in Splunk Web:**
- Settings > Search > SPL2 Compatibility Profile → Enable SPL2

**Gradual migration strategy:**
```spl2
/* Phase 1: Use spl1 wrapper for unsupported commands */
from main
| where sourcetype="access_combined"
| spl1 "transaction session_id maxspan=30m"
| stats count() AS sessions BY host

/* Phase 2: Replace spl1 wrapper as SPL2 commands become available */
from main
| where sourcetype="access_combined"
| transaction session_id maxspan=30m  /* when available in SPL2 */
| stats count() AS sessions BY host
```

**Notes:**
- **Compatibility profile:** Controls whether the search bar accepts SPL1 or SPL2.
  Per-user or system-wide setting.
- `spl1 "..."` wrapper: runs SPL1 pipeline segments inside SPL2 searches.
- Saved searches and alerts can mix SPL1 and SPL2 via the wrapper.
- SPL2 dashboards use `| from` as the source command (Dashboard Studio).
- Monitor SPL2 adoption: Splunk release notes list newly added SPL2 commands.

### SPL2-Only Commands [MEDIUM]

Commands available in SPL2 with no direct SPL1 equivalent.

| Command | Description | SPL1 Workaround |
|---------|-------------|-----------------|
| `branch` | Parallel pipeline execution | Sequential `\| append` subsearches |
| `route` | Conditional event routing to branches | `\| eval` + multiple `\| where` passes |
| `thru` | Pass events through a sub-pipeline without consuming | No equivalent |
| `expand` | Expand array/object fields into rows | `\| mvexpand` (multivalue only) |
| `flatten` | Flatten nested JSON into dot-notation fields | `\| spath` (partial) |
| `decrypt` | Decrypt encrypted fields | No equivalent |
| `ocsf` | Normalize events to OCSF schema | Manual `\| eval` field mapping |
| `into` | Write results to a dataset | `\| collect index=...` |

**branch example:**
```spl2
from main
| branch
  [stats count() AS requests BY host]
  [where status=500 | stats count() AS errors BY host]
```

**expand example:**
```spl2
from main
| expand tags
```
Creates one row per element in the `tags` array field.

**Notes:**
- SPL2 command set grows with each Splunk release — check current docs.
- Use `| spl1 "..."` wrapper for SPL1 commands not yet in SPL2.
- `ocsf` requires the OCSF add-on installed.
