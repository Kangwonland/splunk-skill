---
title: Data Model Acceleration
impact: HIGH
tags: datamodel, acceleration, summariesonly, tstats, tsidx, allow_old_summaries
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/knowledge-objects/data-models/accelerate-data-models"
---

## Data Model Acceleration

Accelerated data models pre-build tsidx summary files for `tstats` queries,
dramatically reducing search time for large datasets.

**Without acceleration (slow):**
```spl
| datamodel Authentication Authentication search
| stats count by Authentication.action
```

**With acceleration (fast via tstats):**
```spl
| tstats summariesonly=true count FROM datamodel=Authentication.Authentication BY Authentication.action
```

**Notes:**
- Enable acceleration: Settings > Data Models > Edit > Accelerate.
- `summariesonly=true` — only use pre-built summaries; returns no results
  if summaries are incomplete. Use for performance-critical dashboards.
- `summariesonly=false` — uses summaries + live data to fill gaps. Slower
  but more complete.
- `allow_old_summaries=true` — use summaries even after schema changes.
  Risk: fields renamed/removed in the model may be stale.
- `nodename=Root.Child.GrandChild` — target a specific dataset node.
- **Role-based search filters** are NOT applied to accelerated data models.
  Use object-level permissions to restrict access instead.
- Acceleration time range controlled by `datamodel.conf` `acceleration.earliest_time`.
