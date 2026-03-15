# SPL1 to SPL2 Mapping

Side-by-side command equivalents for migration.

## Entry and Exit Points
| SPL1 | SPL2 |
|------|------|
| `search index=main` | `from main` |
| `search index=web_logs sourcetype=access_combined` | `from main \| where sourcetype="access_combined"` |
| `\| collect index=summary sourcetype=stash` | `\| into summary` |
| `\| inputlookup geo.csv` | `from lookup:geo` |

## Aggregation
| SPL1 | SPL2 |
|------|------|
| `\| stats count by host` | `\| stats count() AS count BY host` |
| `\| stats avg(rt) as avg_rt by host` | `\| stats avg(rt) AS avg_rt BY host` |
| `\| chart count by host, status` | `\| chart count() AS count BY host, status` |
| `\| timechart span=1h count by status` | `\| timechart span=1h count() AS count BY status` |

## Filtering
| SPL1 | SPL2 |
|------|------|
| `status=5*` | `\| where like(status, "5%")` |
| `status=Error*` (case-insensitive) | `\| where ilike(status, "error%")` |
| `NOT host=web01` | `\| where host != "web01"` |
| `\| where isnull(user)` | `\| where isnull(user)` |

## Sorting and Limiting
| SPL1 | SPL2 |
|------|------|
| `\| sort -count` | `\| sort count DESC` |
| `\| sort +host` | `\| sort host ASC` |
| `\| head 10` | `\| head 10` |

## Parallel Execution
| SPL1 | SPL2 |
|------|------|
| `\| append [search index=A]` + `\| append [search index=B]` | `\| branch [from A] [from B]` |
| No equivalent | `\| route cond1 [...] cond2 [...]` |

## Macros and Reuse
| SPL1 | SPL2 |
|------|------|
| `` `mymacro` `` | `import mymodule; myfunction()` |
| `` `mymacro(arg1,arg2)` `` | `mymodule.myfunction(arg1, arg2)` |

## SPL1 in SPL2 (Wrapper)
| SPL1 | SPL2 |
|------|------|
| `\| transaction session_id` | `\| spl1 "transaction session_id"` |
| `\| iplocation src_ip` | `\| spl1 "iplocation src_ip"` |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2
