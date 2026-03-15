---
title: SPL2 Modules
impact: MEDIUM
tags: spl2, modules, define, namespace, reuse, spl2-files
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL2 Modules

SPL2 modules (`.spl2` files) define reusable search components —
datasets, functions, and views — with explicit namespace management.

**Module file (my_searches.spl2):**
```spl2
$define namespace="myapp";

$define dataset web_errors = {
  from main
  | where sourcetype="access_combined" AND status>=500
};

$define function error_rate(total, errors) = (errors / total) * 100;
```

**Using a module in a search:**
```spl2
import myapp;

from web_errors
| stats count() AS total BY host
| eval rate = error_rate(total, count())
```

**Notes:**
- Module files use `.spl2` extension and are managed in the SPL2 Module Editor
  (Splunk Web: Settings > SPL2 Modules).
- `$define namespace="name"` — declares the module's namespace for imports.
- `$define dataset` — creates a reusable named dataset (view/CTE).
- `$define function` — creates a reusable scalar function.
- **Permissions:** Modules have private/app/global sharing like other knowledge objects.
- Modules replace saved searches and macros for SPL2-native workflows.
- SPL1 macros (`\`macroname\``) are not available in SPL2 — use module functions.
