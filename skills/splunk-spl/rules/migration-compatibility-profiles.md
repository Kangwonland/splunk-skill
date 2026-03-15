---
title: SPL2 Availability and Compatibility
impact: HIGH
tags: migration, spl2, compatibility, platform, availability, gradual-migration
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL2 Availability and Compatibility

SPL2 is available on specific platforms and can coexist with SPL1 via
compatibility profiles and the `spl1` wrapper.

**Platform availability (as of v9.4):**
| Platform | Status |
|----------|--------|
| Splunk Cloud Platform (v9.4.2510+) | GA |
| Splunk Enterprise v9.4 (Linux) | GA |
| Splunk Enterprise v9.4 (Windows) | Not available |
| Splunk Enterprise < v9.4 | Not available |

**Enabling SPL2 in Splunk Web:**
- Settings > Search > SPL2 Compatibility Profile → Enable SPL2

**Gradual migration strategy:**
```spl2
/* Phase 1: Use spl1 wrapper for unsupported commands */
from main
| where sourcetype="access_combined"
| spl1 "transaction session_id maxspan=30m"
| stats count() AS sessions BY host

/* Phase 2: Replace spl1 wrapper as SPL2 commands become available */
from main
| where sourcetype="access_combined"
| transaction session_id maxspan=30m  /* when available in SPL2 */
| stats count() AS sessions BY host
```

**Notes:**
- **Compatibility profile:** Controls whether the search bar accepts SPL1 or SPL2.
  Per-user or system-wide setting.
- `spl1 "..."` wrapper: runs SPL1 pipeline segments inside SPL2 searches.
- Saved searches and alerts can mix SPL1 and SPL2 via the wrapper.
- SPL2 dashboards use `| from` as the source command (Dashboard Studio).
- Monitor SPL2 adoption: Splunk release notes list newly added SPL2 commands.
