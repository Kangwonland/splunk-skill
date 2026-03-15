# limits.conf Command-Specific Stanzas Reference

## [stats] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxvalues` | `100000` | Max values stored per cell by `values()` / `list()`. Excess values are silently truncated. |
| `maxresultrows` | `50000` | Max result rows returned by the `stats` command. |
| `maxmem_mb` | `500` | Max memory per `stats` command (MB). |

## [searchresults] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxresultrows` | `50000` | Max rows for transforming commands |
| `compression_level` | `1` | Result compression (0=none, 9=max) |

## [email] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `result_limit` | `10000` | Max results in sendemail |
| `from` | `splunk@localhost` | Default from address |
| `mailserver` | `localhost` | SMTP server |
| `use_ssl` | `false` | Use SSL for SMTP |
| `use_tls` | `false` | Use STARTTLS for SMTP |

## [collect] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `collect_ignore_minor_breakers` | `false` | Ignore minor breakers in tstats collect |
| `maxheap` | `200` | Max heap size in MB |

## [metrics] Stanza (for mcollect)
| Parameter | Default | Description |
|-----------|---------|-------------|
| `batch_size` | `1000` | Events per mcollect batch |

## [inputproc] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `max_rows_per_table` | `50000` | Max rows in lookup tables |

## [thruput] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxKBps` | `0` (unlimited) | Max KB/second indexing throughput |

Source: https://help.splunk.com/en/splunk-enterprise/administer/limits-conf/9.4/limits-conf-reference
