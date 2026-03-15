# SPL Default Fields Reference

Default fields automatically available in every event.

## Index-Time Fields
| Field | Description | Example |
|-------|-------------|---------|
| `_time` | Event timestamp (Unix epoch) | `1710000000` |
| `_raw` | Full raw event text | `2024-01-01 GET /api 200` |
| `index` | Index name | `main` |
| `host` | Host that generated the event | `webserver01` |
| `source` | Source file or input | `/var/log/access.log` |
| `sourcetype` | Sourcetype classification | `access_combined` |
| `linecount` | Number of lines in event | `1` |
| `punct` | Punctuation signature | `::/-: - - [//:+] "" ` |
| `_indextime` | Time event was indexed (Unix epoch) | `1710000100` |
| `_cd` | Bucket ID and event offset | `0:1234` |
| `_si` | Source index (distributed search) | `server1:main` |

## Search-Time Fields
| Field | Description | Example |
|-------|-------------|---------|
| `splunk_server` | Search head hostname | `sh01.internal` |
| `splunk_server_group` | Search head group | `shc_group1` |
| `_serial` | Result row number (0-based) | `0` |
| `eventtype` | Matching event type names | `web_error` |
| `tag` | Matching tag names | `authentication` |
| `tag::field` | Tags for specific field | `tag::status` |

## Subsearch Output Fields
| Field | Description |
|-------|-------------|
| `search` | Filter expression from subsearch (via format/return) |

## Notes
- `_raw` and `_time` are always retained unless explicitly removed with `fields - _raw`.
- `_indextime` vs `_time`: `_indextime` is when data arrived at Splunk; `_time` is the event's actual time.
- `punct` is useful for clustering events with similar structures.

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/search-time-range
