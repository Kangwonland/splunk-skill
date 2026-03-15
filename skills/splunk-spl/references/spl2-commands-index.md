# SPL2 Commands Index

| Command | Description | SPL1 Equivalent |
|---------|-------------|-----------------|
| `from` | Retrieve data from a named dataset | `search index=` |
| `into` | Write results to a dataset | `\| collect` |
| `where` | Filter events with an expression | `\| where` |
| `eval` | Evaluate expressions and create fields | `\| eval` |
| `stats` | Calculate aggregate statistics | `\| stats` |
| `sort` | Sort results by field (ASC/DESC) | `\| sort` |
| `fields` | Keep or remove fields | `\| fields` |
| `head` | Return first N results | `\| head` |
| `tail` | Return last N results | `\| tail` |
| `rename` | Rename fields | `\| rename` |
| `dedup` | Remove duplicate events | `\| dedup` |
| `table` | Return tabular results | `\| table` |
| `lookup` | Add fields from a lookup table | `\| lookup` |
| `join` | Combine with subsearch results | `\| join` |
| `append` | Append subsearch results | `\| append` |
| `rex` | Extract fields with regex | `\| rex` |
| `spath` | Extract JSON/XML fields | `\| spath` |
| `timechart` | Per-time-bucket chart aggregation | `\| timechart` |
| `chart` | Statistical aggregation for charts | `\| chart` |
| `top` | Most common values | `\| top` |
| `rare` | Least common values | `\| rare` |
| `bin` | Bucket continuous values | `\| bin` |
| `fillnull` | Replace null values | `\| fillnull` |
| `makemv` | Create multivalue field | `\| makemv` |
| `mvexpand` | Expand multivalue to rows | `\| mvexpand` |
| `transaction` | Group events into transactions | `\| transaction` |
| `streamstats` | Running statistics per event | `\| streamstats` |
| `eventstats` | Add stats to all events | `\| eventstats` |
| `tstats` | Query tsidx indexed fields | `\| tstats` |
| `mstats` | Query metrics indexes | `\| mstats` |
| `predict` | ML-based forecasting | `\| predict` |
| `outlier` | Remove/mark outlier values | `\| outlier` |
| `cluster` | Group similar events | `\| cluster` |
| `kmeans` | K-means clustering | `\| kmeans` |
| `branch` | Parallel pipeline execution | No equivalent |
| `route` | Conditional event routing | No equivalent |
| `thru` | Pass-through sub-pipeline | No equivalent |
| `expand` | Expand array/object to rows | `\| mvexpand` (partial) |
| `flatten` | Flatten nested JSON | `\| spath` (partial) |
| `ocsf` | Normalize to OCSF schema | Manual eval |
| `spl1` | Embed SPL1 pipeline in SPL2 | N/A |
| `typer` | Detect field data types | No equivalent |
| `decrypt` | Decrypt field values | No equivalent |
| `inputlookup` | Load from lookup (SPL1 compat) | `\| inputlookup` |
| `outputlookup` | Write to lookup (SPL1 compat) | `\| outputlookup` |
| `makeresults` | Create empty result set | `\| makeresults` |
| `geostats` | Geographic aggregation | `\| geostats` |
| `iplocation` | IP geolocation enrichment | `\| iplocation` |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4
