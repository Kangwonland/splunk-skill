---
title: Anomaly Detection Commands
impact: MEDIUM
tags: anomaly, anomalydetection, anomalousvalue, outlier, probability
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-a/anomalydetection"
---

## Anomaly Detection Commands

Splunk provides three anomaly commands: `anomalydetection` (probability-based),
`anomalies`/`anomalousvalue` (unexpectedness scoring), and `outlier` (axis cleanup).

**anomalydetection (probability-based):**
```spl
index=web_logs
| stats count by host, status
| anomalydetection action=annotate
```
Adds `anomaly_score` and `is_anomaly` fields. `action=annotate` marks events;
`action=filter` removes normal events.

**anomalousvalue (field-level unexpectedness):**
```spl
index=web_logs
| anomalousvalue pthresh=0.02 action=annotate
```
Flags individual field values with unexpectedness score above threshold.

**outlier (remove chart outliers):**
```spl
index=metrics
| timechart avg(cpu_pct) by host
| outlier action=remove
```
Removes data points that would skew chart axes (not anomaly detection).

**Notes:**
- `anomalydetection` builds a frequency model per field value — memory intensive
  on high-cardinality fields.
- `pthresh=0.02` — probability threshold; values below this are anomalous.
- `action=annotate` — keeps all events, adds score fields.
- `action=filter` — removes non-anomalous events (for alert use cases).
- `outlier` uses IQR (interquartile range) by default: `param.IQRMultiplier=1.5`.
