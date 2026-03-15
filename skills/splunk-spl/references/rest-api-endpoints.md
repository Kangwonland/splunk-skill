# Splunk REST API Endpoints Reference

Base URL: `https://splunk-host:8089/services/`

## Search Job Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/services/search/jobs` | Create a new search job |
| `GET` | `/services/search/jobs/{sid}` | Get job status and metadata |
| `GET` | `/services/search/jobs/{sid}/results` | Get final results (after DONE) |
| `GET` | `/services/search/jobs/{sid}/events` | Get matched events (during/after) |
| `GET` | `/services/search/jobs/{sid}/summary` | Field summary statistics |
| `GET` | `/services/search/jobs/{sid}/timeline` | Event timeline buckets |
| `POST` | `/services/search/jobs/{sid}/control` | Pause/resume/cancel/finalize job |
| `DELETE` | `/services/search/jobs/{sid}` | Delete job |
| `GET` | `/services/search/jobs/export` | Stream results without persistent job |

## Authentication Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/services/auth/login` | Get session token (Basic Auth) |
| `DELETE` | `/services/auth/login/{sessionKey}` | Logout / invalidate token |
| `GET` | `/services/authentication/users` | List users |
| `GET` | `/services/authentication/current-context` | Current user info |

## Key POST Parameters for /services/search/jobs
| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `search` | Yes | — | SPL search string |
| `earliest_time` | Recommended | All Time | Start of time range |
| `latest_time` | Recommended | All Time | End of time range |
| `status_buckets` | For timeline | `0` | Number of timeline buckets |
| `exec_mode` | No | `normal` | `normal`, `blocking`, `oneshot` |
| `output_mode` | No | `xml` | `xml`, `json`, `csv` |
| `max_count` | No | `10000` | Max results to return |
| `rf` | No | — | Required fields to extract |
| `index_earliest` | No | — | Index time range start |
| `index_latest` | No | — | Index time range end |

## dispatchState Values
| State | Description |
|-------|-------------|
| `QUEUED` | Job waiting to start |
| `PARSING` | SPL being parsed |
| `RUNNING` | Search executing |
| `FINALIZING` | Results being finalized |
| `DONE` | Complete, results available |
| `FAILED` | Search failed |
| `PAUSED` | Job paused by user |
| `ZOMBIED` | Job killed by system |

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api
