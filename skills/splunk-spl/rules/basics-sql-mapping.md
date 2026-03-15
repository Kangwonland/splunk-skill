---
title: SQL to SPL Equivalents
impact: HIGH
tags: basics, sql, mapping, translation
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-overview/about-the-search-language"
---

## SQL to SPL Equivalents

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
