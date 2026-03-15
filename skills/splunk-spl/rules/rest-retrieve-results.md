---
title: Retrieving Search Results via REST API
impact: HIGH
tags: rest, results, events, output_mode, pagination, max_count
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/use-the-splunk-rest-api/access-splunk-search-results-using-the-rest-api"
---

## Retrieving Search Results via REST API

Use `/results` for post-completion results. Use `/events` during the search
for preview. Paginate large result sets with `offset` and `count`.

**Fetch results (after DONE):**
```bash
curl -sk -u admin:changeme \
  "https://localhost:8089/services/search/jobs/$SID/results" \
  -d output_mode=json \
  -d count=100 \
  -d offset=0
```

**Paginate through all results:**
```bash
OFFSET=0; COUNT=1000
while true; do
  RESULTS=$(curl -sk -u admin:changeme \
    "https://localhost:8089/services/search/jobs/$SID/results?output_mode=json&count=$COUNT&offset=$OFFSET")
  NRESULTS=$(echo "$RESULTS" | python3 -c "import sys,json; d=json.load(sys.stdin); print(len(d['results']))")
  echo "$RESULTS"
  [ "$NRESULTS" -lt "$COUNT" ] && break
  OFFSET=$((OFFSET + COUNT))
done
```

**Notes:**
- `/results` — available only after `dispatchState=DONE`. Returns transformed results.
- `/events` — available during and after search. Returns raw matched events.
- `output_mode` options: `json` (default: `xml`), `csv`, `xml`.
- `max_count=10000` — default result limit. Increase in request or via `limits.conf`.
- `count=0` — returns up to `max_count` results (not truly unlimited).
- **Field filtering:** `f=field1&f=field2` — return only specified fields.
- See: `rest-export.md` for streaming large result sets without a job SID.
