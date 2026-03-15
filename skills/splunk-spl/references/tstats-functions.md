# tstats Supported Functions Reference

## Supported Aggregation Functions
| Function | Description | Notes |
|----------|-------------|-------|
| `avg(field)` | Average | Numeric fields only |
| `count` / `count(field)` | Event count | `count` counts events; `count(field)` counts non-null |
| `dc(field)` | Distinct count | Approximate for large sets |
| `estdc(field)` | Estimated distinct count | Faster than `dc` |
| `first(field)` | First value in time order | |
| `last(field)` | Last value in time order | |
| `latest(field)` | Same as `last` | |
| `earliest(field)` | Same as `first` | |
| `max(field)` | Maximum value | |
| `median(field)` | Median (50th percentile) | |
| `min(field)` | Minimum value | |
| `mode(field)` | Most frequent value | |
| `perc<N>(field)` | Nth percentile | e.g., `perc95` |
| `range(field)` | Max minus min | |
| `stdev(field)` | Standard deviation | |
| `sum(field)` | Sum | Numeric only |
| `sumsq(field)` | Sum of squares | |
| `values(field)` | All distinct values (multivalue) | |
| `var(field)` | Variance | |
| `rate(field)` | Rate per second | |

## NOT Supported in tstats
| Feature | Workaround |
|---------|------------|
| `count(eval(...))` | Use `eval` in `| stats` instead |
| Wildcard field names in `BY` | Enumerate fields explicitly |
| `list(field)` | Use `values(field)` |
| Search-time extracted fields | Index with `INDEXED=true` in fields.conf |
| `mvexpand` in BY clause | Post-process with `| mvexpand` |

## Special Parameters
| Parameter | Description |
|-----------|-------------|
| `fillnull_value=string` | Replace null field values |
| `local=true` | Debug: run on search head only |
| `prestats=true` | Emit pre-aggregation rows for `append` |
| `summariesonly=true` | Data model queries: use only accelerated data |
| `allow_old_summaries=true` | Data model: use summaries after schema change |
| `nodename=Root.Child` | Data model: target specific dataset |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-t/tstats
