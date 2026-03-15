---
title: Adding Fields from Lookup Tables
impact: HIGH
tags: lookup, OUTPUT, OUTPUTNEW, wildcard, enrichment
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/lookups/about-lookups"
---

## Adding Fields from Lookup Tables

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
