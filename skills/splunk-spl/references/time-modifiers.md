# Time Modifier Reference

## Time Unit Abbreviations
| Unit | Abbreviations |
|------|---------------|
| Seconds | `s`, `sec`, `secs`, `second`, `seconds` |
| Minutes | `m`, `min`, `minute`, `minutes` |
| Hours | `h`, `hr`, `hrs`, `hour`, `hours` |
| Days | `d`, `day`, `days` |
| Weeks | `w`, `week`, `weeks` |
| Months | `mon`, `month`, `months` |
| Quarters | `q`, `qtr`, `qtrs`, `quarter`, `quarters` |
| Years | `y`, `yr`, `yrs`, `year`, `years` |

## Snap-to Alignment (`@`)
| Modifier | Description |
|----------|-------------|
| `@s` | Snap to second boundary |
| `@m` | Snap to minute boundary |
| `@h` | Snap to hour boundary |
| `@d` | Snap to day boundary (midnight) |
| `@w` / `@w0` | Snap to week (Sunday) |
| `@w1` | Snap to Monday |
| `@w2` | Snap to Tuesday |
| `@w3` | Snap to Wednesday |
| `@w4` | Snap to Thursday |
| `@w5` | Snap to Friday |
| `@w6` | Snap to Saturday |
| `@w7` | Snap to Sunday (same as `@w0`) |
| `@mon` | Snap to month (1st of month) |
| `@q` | Snap to quarter (Jan/Apr/Jul/Oct 1st) |
| `@y` | Snap to year (Jan 1st) |

## Common Examples
| Expression | Meaning |
|------------|---------|
| `earliest=-24h latest=now` | Last 24 hours (rolling) |
| `earliest=-1d@d latest=@d` | Yesterday (full day) |
| `earliest=@d latest=now` | Today so far |
| `earliest=-7d@d latest=@d` | Last 7 complete days |
| `earliest=-1mon@mon latest=@mon` | Last month |
| `earliest=-1w@w latest=@w` | Last week (Sun–Sat) |
| `earliest=-1w@w1 latest=@w1` | Last week (Mon–Sun) |
| `earliest=-1q@q latest=@q` | Last quarter |
| `earliest=-1y@y latest=@y` | Last year |

## Special Values
| Value | Meaning |
|-------|---------|
| `now` | Current time |
| `earliest=1` | Unix epoch (Jan 1, 1970) |
| `earliest=0` | Equivalent to epoch start |
| `latest=now` | Current time |
| `rt` | Real-time (current moment) |
| `rt-30s` | 30 seconds before now (for RT searches) |

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-time-range/specify-time-modifiers
