---
title: SPL2 Lambda Expressions
impact: MEDIUM
tags: spl2, lambda, higher-order, eval, function
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2"
---

## SPL2 Lambda Expressions

SPL2 supports lambda expressions for inline function definitions in
higher-order functions.

**Lambda syntax:**
```spl2
lambda <param>: <expression>
lambda <param1>, <param2>: <expression>
```

**Using lambda with map function:**
```spl2
from main
| eval doubled_values = map(split(raw_numbers, ","), lambda x: tonumber(x) * 2)
```

**Lambda in custom function definition:**
```spl2
$define function transform(lst, fn) = map(lst, fn);

from main
| eval result = transform(my_list, lambda x: x + 1)
```

**Field template syntax `{fieldname}`:**
```spl2
from main
| eval msg = "Host {host} returned status {status}"
```
`{fieldname}` interpolates the field value into the string.

**Notes:**
- Lambda expressions are anonymous functions — no `$define` needed.
- Parameters are positional — no named parameters in lambdas.
- Lambdas can reference fields from the current event via closure.
- **SPL1 equivalent:** No direct equivalent. SPL1 uses `mvmap()` for
  multivalue field transformation: `| eval result = mvmap(mv_field, value * 2)`.
- Field templates `{field}` are SPL2-only — use `eval` + `"..."` concatenation in SPL1.
