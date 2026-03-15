---
title: Clustering Commands — cluster and kmeans
impact: MEDIUM
tags: anomaly, cluster, kmeans, grouping, similarity
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-c/cluster"
---

## Clustering Commands — cluster and kmeans

`cluster` groups similar raw events; `kmeans` applies k-means clustering
to numeric fields.

**cluster (text similarity grouping):**
```spl
index=app_logs level=ERROR
| cluster field=_raw t=0.8
| stats count by cluster_label
```
Groups error messages with ≥80% similarity. `cluster_label` names each group.

**kmeans (numeric field clustering):**
```spl
index=metrics
| stats avg(cpu_pct) as cpu, avg(mem_pct) as mem by host
| kmeans k=5 reps=10 dt_thresh=0.001
| table host, cluster_id
```
Groups hosts into 5 clusters by CPU/memory profile.

**Notes:**
- `cluster t=N` — similarity threshold (0.0–1.0). Higher = more distinct clusters.
- `cluster showcount=true` — add `cluster_count` field (number of events in cluster).
- `kmeans k=N` — number of clusters (required). Choose based on domain knowledge.
- `kmeans reps=N` — number of random initializations (default 1). Higher = better
  but slower. Use `reps=10` for more stable results.
- `kmeans dt_thresh=N` — convergence threshold (default 0.0001).
- Both commands are non-distributable — run on search head with full result set.
- Reduce data volume with `sample` or aggregate with `stats` before clustering.
