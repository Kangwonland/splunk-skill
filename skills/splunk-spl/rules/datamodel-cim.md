---
title: CIM (Common Information Model) Field Conventions
impact: MEDIUM
tags: cim, datamodel, normalization, field-naming, sourcetype
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/knowledge-objects/data-models/about-data-models"
---

## CIM (Common Information Model) Field Conventions

CIM standardizes field names across sourcetypes so that searches and
dashboards work without per-sourcetype customization.

**Without CIM (sourcetype-specific):**
```spl
index=firewall sourcetype=palo_alto_networks
| stats count by src_ip, dst_port
```
Different field names per sourcetype break cross-sourcetype searches.

**With CIM (normalized):**
```spl
| from datamodel:Network_Traffic.Network_Traffic
| stats count by src, dest_port
```
CIM-compliant fields: `src`, `dest`, `src_port`, `dest_port`, `user`, `action`.

**Key CIM field names by category:**
- **Authentication:** `action` (success/failure), `user`, `src`, `dest`, `app`
- **Network Traffic:** `src`, `dest`, `src_port`, `dest_port`, `transport`, `bytes_in`, `bytes_out`
- **Web:** `uri_path`, `uri_query`, `http_method`, `status`, `bytes`, `src`, `dest`
- **Endpoint:** `dest`, `user`, `process`, `process_id`, `file_path`, `registry_path`

**Notes:**
- CIM add-ons provide field aliases and transforms that map sourcetype fields to CIM names.
- Install the Splunk Common Information Model Add-on from Splunkbase.
- `tag` field is CIM's primary classification mechanism: `tag=authentication`, `tag=network`.
- Cross-sourcetype correlation: normalize to CIM fields first, then correlate.
