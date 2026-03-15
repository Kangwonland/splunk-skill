---
title: Search Macro Best Practices
impact: HIGH
tags: macro, best-practices, DRY, index-abstraction, validation
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/knowledge-objects/macros/about-search-macros"
---

## Search Macro Best Practices

Use macros to abstract index names, common filters, and repeated SPL
patterns. This prevents hard-coded values from spreading across dashboards.

**Incorrect (hard-coded index in every search):**
```spl
index=prod_web_logs_v2 sourcetype=access_combined status>=400
```
When the index is renamed or split, every search must be updated.

**Correct (index abstracted into a macro):**
Macro `web_index`:
```
index=prod_web_logs_v2 sourcetype=access_combined
```
Usage:
```spl
`web_index` status>=400
```

**Notes:**
- **DRY principle:** Define once in a macro; reference everywhere.
  Index names, sourcetype filters, common eval expressions.
- **Argument validation:** Use the `validation` field in `macros.conf`
  to reject invalid arguments: `isint($threshold$)` rejects non-integer values.
- **Shared macros:** Place macros in a shared app (e.g., `search`) to make
  them available across all apps. App-specific macros stay in the app.
- **Documenting macros:** Add description in `macros.conf` — visible in
  Splunk Web's macro editor and to other developers.
- **Testing macros:** Expand inline to verify: Settings > Advanced Search > Search Macros.
- Avoid macros for one-off searches — the abstraction cost outweighs the benefit.
