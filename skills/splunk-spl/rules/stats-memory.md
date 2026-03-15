---
title: stats Memory Optimization
impact: HIGH
tags: stats, memory, dc, estdc, values, list, performance
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/statistical-and-charting-functions/about-statistical-and-charting-functions"
---

## stats Memory Optimization

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
- `perc95(field)` — approximate (T-digest algorithm, ~2% error, lower memory);
  `exactperc95(field)` — exact (higher memory). Use `perc95` for large datasets.
