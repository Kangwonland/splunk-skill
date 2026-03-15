---
title: Real-Time Search Modifiers
impact: CRITICAL
tags: time, realtime, rt, rtsearch, rtorder, windowed-rt
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/specify-time-ranges/specify-time-ranges-for-real-time-searches"
---

## Real-Time Search Modifiers

Real-time searches use `rt` prefix time modifiers. UI-based RT and
programmatic RT behave differently — confusing them causes missed events.

**Incorrect:**
```spl
index=web_logs earliest=rt latest=rt
```
Continuous real-time: captures events as they arrive but shows no
historical data. Cannot be used with transforming commands reliably
(events arrive out of order).

**Correct (windowed real-time):**
```spl
index=web_logs earliest=rt-5m latest=rt
```
Windowed RT: uses a 5-minute sliding window, buffers for ordering,
works correctly with `stats` and `timechart`.

**Notes:**
- **`rt` prefix:** `earliest=rt` means "real-time now". Works in the Splunk
  UI and via the REST API (when creating real-time search jobs). Not
  applicable to scheduled searches, which run on a defined schedule and
  cannot operate in continuous real-time mode.
- **Windowed RT:** `rt-30s` / `rt+0s` — maintains a time window, handles
  out-of-order events. Suitable for dashboards.
- **`rtorder`:** Use `| rtorder` command after a continuous RT search to
  buffer and reorder events: `| rtorder buffer_span=5s`
- **CLI:** Use `./splunk rtsearch` for real-time searches from CLI;
  `earliest_time` and `latest_time` are **required** parameters.
- Real-time searches consume continuous resources — always use windowed RT
  in production dashboards.
