---
title: SPL1 to SPL2 Syntax Differences
impact: HIGH
tags: migration, spl2, spl1, syntax, differences
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL1 to SPL2 Syntax Differences

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
