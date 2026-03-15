# Splunk CLI Search Parameters Reference

Full parameter reference for `./splunk search` and `./splunk rtsearch`.

## Parameters Table
| Parameter | Default | Description |
|-----------|---------|-------------|
| `-app` | `search` | Splunk app context for search execution |
| `-auth` | _(saved cred)_ | `username:password` for authentication |
| `-batch` | `false` | Run as batch search (no preview) |
| `-detach` | `false` | Return SID immediately without waiting |
| `-earliest_time` | _(none = All Time)_ | Search start time (relative or absolute) |
| `-header` | `true` | Include field names header row in output |
| `-index_earliest` | _(none)_ | Index time range start |
| `-index_latest` | _(none)_ | Index time range end |
| `-latest_time` | _(none = now)_ | Search end time |
| `-max_time` | `0` (unlimited) | Max seconds to run search |
| `-maxout` | `100` | Max events returned (0 = system limit) |
| `-output` | `auto` | Output format: `auto`, `csv`, `json`, `xml`, `raw` |
| `-preview` | `true` | Stream preview results as search runs |
| `-rf` | _(none)_ | Required fields to include in results |
| `-socket_timeout` | `0` | Socket timeout in seconds |
| `-timeout` | `0` | Job expiration timeout in seconds |
| `-uri` | `https://localhost:8089` | Management URI |
| `-wrap` | `true` | Wrap long lines in terminal output |

## Output Format Options
| Format | Description |
|--------|-------------|
| `auto` | Table for interactive, raw otherwise |
| `csv` | Comma-separated with header |
| `json` | JSON array of objects |
| `xml` | Splunk XML format |
| `raw` | `_raw` field only (no headers) |

## rtsearch-Specific Parameters
| Parameter | Note |
|-----------|------|
| `-earliest_time` | **Required** for rtsearch. Format: `rt`, `rt-30s`, `rt+30s` |
| `-latest_time` | **Required** for rtsearch. Usually `rt` |

## Usage Examples
```bash
# Historical search with CSV output
./splunk search 'index=web_logs | stats count by host' \
  -earliest_time -24h -latest_time now \
  -output csv -maxout 0

# Real-time windowed search
./splunk rtsearch 'index=web_logs status=500 | stats count by host' \
  -earliest_time rt-60s -latest_time rt

# Detached background search
./splunk search 'index=web_logs | stats count by status' \
  -earliest_time -7d -latest_time now \
  -detach true
```

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api
