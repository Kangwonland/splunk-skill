---
title: Relative Time Syntax and Snap-To
impact: CRITICAL
tags: time, relative-time, snap-to, earliest, latest
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/specify-time-ranges/about-searching-with-time"
---

## Relative Time Syntax and Snap-To

Use relative time modifiers with snap-to operators for accurate, repeatable
time windows. Missing snap-to (`@`) causes inconsistent results across runs.

**Incorrect:**
```spl
index=web_logs earliest=-7d latest=now
```
`-7d` is relative to *now* at search time — the window shifts with each run,
making scheduled reports inconsistent.

**Correct:**
```spl
index=web_logs earliest=-7d@d latest=@d
```
Snaps to midnight, giving a consistent full-day window regardless of when
the search runs.

**Notes:**
- **Unit aliases:** `s`=seconds, `m`=minutes, `h`=hours, `d`=days,
  `w`=weeks (w0=Sunday), `mon`=months, `q`=quarters, `y`=years
- **Snap-to syntax:** `@unit` — truncates to the start of that unit
  - `-1h@h` = start of the previous hour
  - `@w0` = start of Sunday (beginning of week)
  - `@q` = start of current quarter
- **Chained offsets:** `earliest=-mon@mon+7d` = 7 days into last month
- **Named times:** `earliest=@d` = midnight today, `earliest=-1d@d` = yesterday midnight
- See: `references/time-modifiers.md` for the full modifier reference table.
