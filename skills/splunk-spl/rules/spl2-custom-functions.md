---
title: SPL2 Custom Functions
impact: MEDIUM
tags: spl2, custom-functions, define, eval-function, command-function
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL2 Custom Functions

SPL2 allows defining custom eval functions and command functions in modules.

**Custom eval function:**
```spl2
$define function bytes_to_mb(bytes) = bytes / 1048576;

from main
| eval size_mb = bytes_to_mb(file_size)
```

**Custom function with multiple parameters:**
```spl2
$define function rate_per_hour(count, duration_seconds) =
  (count / duration_seconds) * 3600;

from main
| stats count() AS req_count, max(_time) - min(_time) AS duration BY host
| eval hourly_rate = rate_per_hour(req_count, duration)
```

**Documentation comments:**
```spl2
/*
 * Converts bytes to human-readable size string.
 * @param bytes - File size in bytes
 * @returns String like "1.2 MB" or "3.4 GB"
 */
$define function human_size(bytes) =
  case(bytes < 1024, tostring(bytes) + " B",
       bytes < 1048576, tostring(round(bytes/1024, 1)) + " KB",
       bytes < 1073741824, tostring(round(bytes/1048576, 1)) + " MB",
       true, tostring(round(bytes/1073741824, 2)) + " GB");
```

**Notes:**
- Custom functions are defined in `.spl2` module files (not inline in searches).
- **SPL1 equivalent:** Search macros with arguments: `` `mymacro(arg1, arg2)` ``.
- Functions are pure (no side effects) — they transform field values.
- Recursive functions are not supported.
- Custom command functions (pipeline-level) use `$define command` syntax.
