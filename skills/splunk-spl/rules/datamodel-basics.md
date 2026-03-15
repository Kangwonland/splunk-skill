---
title: Data Model Basics
impact: HIGH
tags: datamodel, pivot, from, dataset, hierarchy, cim
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/knowledge-objects/data-models/about-data-models"
---

## Data Model Basics

Data models define hierarchical event datasets for pivot-based searches
and accelerated `tstats` queries.

**Search using datamodel command:**
```spl
| datamodel Authentication Authentication search
| fields Authentication.action, Authentication.user, Authentication.src
```
`datamodel <ModelName> <DatasetName> search` returns raw events matching
the dataset constraints.

**Pivot against a data model:**
```spl
| pivot Authentication Authentication count(Authentication) AS count SPLITROW Authentication.action PERIOD latest
```

**From syntax (SPL2-style):**
```spl
| from datamodel:Authentication.Authentication
| search Authentication.action=failure
```

**Notes:**
- Data model hierarchy: root dataset → child datasets (each child adds constraints).
- `| datamodel` returns fields prefixed with `ModelName.` — use `rename` or CIM add-on field aliases.
- `strict_fields=true` — restricts results to fields defined in the data model.
- Data model permissions: private → shared → global via Settings > Data Models.
- CIM (Common Information Model): standardizes field names across sourcetypes.
  See: `datamodel-cim.md`.
