---
title: transaction Command Parameter Patterns
impact: MEDIUM
tags: transaction, startswith, endswith, maxspan, maxpause, maxevents
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/transaction"
---

## transaction Command Parameter Patterns

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
