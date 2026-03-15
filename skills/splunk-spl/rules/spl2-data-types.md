---
title: SPL2 Data Types
impact: MEDIUM
tags: spl2, data-types, string, number, bool, ip, timestamp, typer
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL2 Data Types

SPL2 has explicit built-in data types. Type errors are caught at parse
time rather than producing silent null values.

**Built-in types:**
| Type | Description | Example |
|------|-------------|---------|
| `string` | Text value | `"hello"` |
| `number` | Numeric (int or float) | `42`, `3.14` |
| `bool` | Boolean | `true`, `false` |
| `ip` | IP address | `192.168.1.1` |
| `timestamp` | Date/time value | `2024-01-01T00:00:00Z` |

**Type detection with typer:**
```spl2
from main
| typer
| where _type_src_ip="ip"
```
`typer` adds `_type_<fieldname>` fields identifying the detected type.

**Type casting:**
```spl2
from main
| eval port_num = tonumber(port)
| eval ts = totime(timestamp_str, "%Y-%m-%dT%H:%M:%S")
```

**Notes:**
- SPL2 type system prevents silent coercion errors common in SPL1.
- `tonumber()`, `tostring()`, `tobool()`, `toip()`, `totime()` — explicit casts.
- **SPL1 comparison:** SPL1 uses implicit type coercion — `"100" > "99"` evaluates
  as string comparison (false), not numeric (true). SPL2 raises a type error instead.
- Custom type schemas can be defined in modules for structured data validation.
- `typer` command: `_type_<field>` values: `"string"`, `"number"`, `"bool"`, `"ip"`, `"null"`.
