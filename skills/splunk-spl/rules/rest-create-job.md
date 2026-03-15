---
title: Creating Search Jobs via REST API
impact: HIGH
tags: rest, search-jobs, sid, dispatchState, async, poll
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api"
---

## Creating Search Jobs via REST API

`POST /services/search/jobs` creates an async search job. Poll
`dispatchState` until `DONE` before fetching results.

**Create a job:**
```bash
SID=$(curl -sk -u admin:changeme \
  https://localhost:8089/services/search/jobs \
  -d search="search index=web_logs status=500 | stats count by host" \
  -d earliest_time="-24h" \
  -d latest_time="now" \
  -d status_buckets=300 \
  | grep -o '<sid>[^<]*' | sed 's/<sid>//')
echo "SID: $SID"
```

**Poll until DONE:**
```bash
while true; do
  STATE=$(curl -sk -u admin:changeme \
    "https://localhost:8089/services/search/jobs/$SID" \
    | grep -o 'dispatchState[^<]*' | head -1)
  echo "State: $STATE"
  [[ "$STATE" == *"DONE"* ]] && break
  sleep 2
done
```

**Notes:**
- **Dispatch states:** `QUEUED` → `PARSING` → `RUNNING` → `FINALIZING` → `DONE`.
  Also: `FAILED`, `PAUSED`.
- **ALWAYS set `earliest_time`/`latest_time`** — default is All Time (scans everything).
- **`status_buckets=300`** — required for timeline and event distribution access.
  Without it, `/timeline` and `/summary` endpoints return empty.
- `output_mode=json` on the job status endpoint for JSON responses.
- `exec_mode=oneshot` — synchronous execution, returns results directly without polling.
  Use only for fast searches.
- See: `rest-retrieve-results.md` for fetching results after DONE.
