---
title: REST API Export Endpoint
impact: MEDIUM
tags: rest, export, streaming, large-results, no-sid
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api"
---

## REST API Export Endpoint

`GET /services/search/jobs/export` streams search results without creating
a persistent job — ideal for large result sets.

**Stream results directly to file:**
```bash
curl -sk -u admin:changeme \
  "https://localhost:8089/services/search/jobs/export" \
  -d search="search index=web_logs status=500 earliest=-24h latest=now | fields host, status, uri" \
  -d output_mode=csv \
  > results.csv
```

**Stream JSON:**
```bash
curl -sk -u admin:changeme \
  "https://localhost:8089/services/search/jobs/export" \
  -d "search=search index=web_logs | stats count by host" \
  -d output_mode=json \
  -d earliest_time="-1h" \
  -d latest_time="now"
```

**Notes:**
- **No SID created** — results stream directly without a persistent job.
  Cannot resume or poll; must consume the full stream.
- **`output_mode`:** `csv`, `json`, `xml` (default xml), `raw`.
- **ALWAYS set `earliest_time`/`latest_time`** — no default time range.
- **No `max_count` limit** — returns all matching results (unlike `/results`
  which defaults to 10,000).
- **Timeout:** Set `max_time=300` to cap execution time.
- **`search` parameter** must include the leading `search` keyword or SPL command.
- Use for ETL pipelines, data exports, and large batch retrievals.
- Vs `/results`: export streams (no persistent job); `/results` requires completed job.
