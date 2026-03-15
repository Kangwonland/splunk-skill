# limits.conf Subsearch and Join Stanzas Reference

## [subsearch] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxout` | `10000` | Max results returned from a `[ ]` subsearch |
| `maxtime` | `60` | Max seconds a subsearch can run |
| `ttl` | `300` | Seconds subsearch results are cached |
| `subsearch_artifacts_delete_policy` | `ttl` | When to delete: `ttl` or `access` |

## [join] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `subsearch_maxout` | `50000` | Max rows from join's internal subsearch |
| `subsearch_maxtime` | `60` | Max seconds for join's internal subsearch |

## [foreach] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxvals` | `50` | Max iterations for foreach command |

## [transaction] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxevents` | `1000` | Max events per transaction |
| `maxopentxn` | `100000` | Max simultaneously open transactions |
| `maxopenevents` | `100000` | Max open events across all transactions |

## [realtime] Stanza
| Parameter | Default | Description |
|-----------|---------|-------------|
| `queue_size` | `10000` | Max events in real-time queue |
| `query_cache_timeout` | `5` | Seconds between real-time re-queries |

Source: https://help.splunk.com/en/splunk-enterprise/administer/limits-conf/9.4/limits-conf-reference
