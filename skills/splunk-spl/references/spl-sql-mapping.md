# SQL to SPL Mapping

Equivalence patterns for SQL users learning SPL.

| SQL | SPL | Notes |
|-----|-----|-------|
| `SELECT * FROM table` | `index=main \| table *` | SPL always starts with data source |
| `SELECT col1, col2 FROM table` | `index=main \| table col1 col2` | |
| `SELECT col AS alias FROM table` | `index=main \| rename col AS alias` | |
| `WHERE condition` | `index=main condition` or `\| where condition` | Search-time: before pipe; Eval-time: after pipe |
| `WHERE col LIKE 'val%'` | `index=main col=val*` | Trailing wildcard |
| `WHERE col LIKE '%val%'` | `index=main \| where like(col, "%val%")` | Leading wildcard: avoid |
| `WHERE col IS NULL` | `\| where isnull(col)` | |
| `WHERE col IS NOT NULL` | `\| where isnotnull(col)` | |
| `WHERE col IN (a, b, c)` | `col IN (a, b, c)` | SPL supports IN operator |
| `GROUP BY col` | `\| stats count by col` | |
| `GROUP BY col1, col2` | `\| stats count by col1, col2` | |
| `HAVING count > 10` | `\| stats count by col \| where count > 10` | |
| `ORDER BY col ASC` | `\| sort +col` | |
| `ORDER BY col DESC` | `\| sort -col` | |
| `LIMIT 10` | `\| head 10` | |
| `OFFSET 10 LIMIT 5` | `\| tail +11 \| head 5` | Approximate |
| `COUNT(*)` | `\| stats count` | |
| `COUNT(DISTINCT col)` | `\| stats dc(col)` | |
| `SUM(col)` | `\| stats sum(col)` | |
| `AVG(col)` | `\| stats avg(col)` | |
| `MAX(col)` | `\| stats max(col)` | |
| `MIN(col)` | `\| stats min(col)` | |
| `JOIN table2 ON t1.id = t2.id` | `\| lookup table2.csv id OUTPUT field1` | Use lookup when possible |
| `LEFT JOIN` | `\| join type=left id [search index=b]` | |
| `UNION` | `\| append [search index=b]` or `\| set union` | |
| `INSERT INTO` | `\| outputlookup table.csv` | |
| `UPDATE` | `\| outputlookup table.csv` (overwrites) | |
| `CREATE TABLE` | `\| outputlookup new_table.csv createempty=true` | |

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches
