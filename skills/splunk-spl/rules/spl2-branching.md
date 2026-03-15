---
title: SPL2 branch Command — Parallel Pipelines
impact: MEDIUM
tags: spl2, branch, parallel, pipeline, investigation
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-search-commands/branch"
---

## SPL2 branch Command — Parallel Pipelines

`branch` splits the pipeline into parallel sub-pipelines, each operating
on the same input data. Results are merged.

**Parallel investigation:**
```spl2
from main
| where sourcetype="access_combined"
| branch
  [stats count() AS total_requests BY host]
  [where status=500 | stats count() AS errors BY host]
```
Both branches receive the same filtered events. Results are union-merged.

**Multi-path analysis:**
```spl2
from main
| where index="web_logs"
| branch
  [stats avg(response_time) AS avg_rt BY host | eval metric="avg_rt"]
  [stats max(response_time) AS max_rt BY host | eval metric="max_rt"]
  [stats count() AS requests BY host | eval metric="requests"]
| stats values(avg_rt) AS avg_rt, values(max_rt) AS max_rt, values(requests) AS requests BY host
```

**Notes:**
- Each branch in `[...]` receives an identical copy of the input dataset.
- Branches execute in parallel — faster than sequential SPL1 subsearches.
- Result merging: branches are appended (union), not joined.
- **No SPL1 equivalent** — closest approximation: multiple `| append` subsearches
  (sequential, not parallel).
- `route` command (SPL2): like `branch` but routes events to different
  branches based on a condition (events are NOT duplicated).
