---
title: transaction vs stats — When to Use Each
impact: HIGH
tags: transaction, stats, performance, grouping, sessions
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/transaction"
---

## transaction vs stats — When to Use Each

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
