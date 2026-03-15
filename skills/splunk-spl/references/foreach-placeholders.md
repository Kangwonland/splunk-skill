# foreach Command Placeholders Reference

## Multifield Mode (default)
Iterates over field names matching a glob pattern.

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `<<FIELD>>` | Full matched field name | `req_count_web` → `req_count_web` |
| `<<MATCHSTR>>` | Portion matching `*` in pattern | `req_count_*` on `req_count_web` → `web` |
| `<<MATCHSEG1>>` | First `*` capture in multi-wildcard pattern | `val_*_*` on `val_a_b` → `a` |
| `<<MATCHSEG2>>` | Second `*` capture | `val_*_*` on `val_a_b` → `b` |
| `<<MATCHSEG3>>` | Third `*` capture | `val_*_*_*` on `val_a_b_c` → `c` |

**Example:**
```spl
| foreach req_* [eval total_<<MATCHSTR>> = '<<FIELD>>' * 100]
```
On `req_web`, `req_api`: creates `total_web = 'req_web' * 100`, `total_api = 'req_api' * 100`.

## Multivalue Mode (`mode=multivalue`)
Iterates over values in a multivalue field.

| Placeholder | Description |
|-------------|-------------|
| `<<ITEM>>` | Current value from the multivalue field |
| `<<ITER>>` | 0-based iteration index |

**Example:**
```spl
| eval methods = split("GET POST PUT", " ")
| foreach mode=multivalue methods [eval count_<<ITEM>> = 0]
```

## JSON Array Mode (`mode=json_array`)
Iterates over elements in a JSON array field.

| Placeholder | Description |
|-------------|-------------|
| `<<ITEM>>` | Current JSON array element value |
| `<<ITER>>` | 0-based array index |

## Auto Collections Mode (`mode=auto_collections`)
Handles both multivalue and JSON arrays automatically.

| Placeholder | Description |
|-------------|-------------|
| `<<ITEM>>` | Current element (MV value or JSON element) |
| `<<ITER>>` | 0-based iteration index |

## Notes
- `maxvals=50` in `limits.conf [foreach]` limits iterations.
- In multivalue/JSON/auto modes: only a single `eval` is allowed per template.
- In multifield mode: full pipeline allowed inside `[...]`.
- Custom placeholder rename: not supported — use `<<FIELD>>`, `<<ITEM>>`, etc. as-is.

Source: https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-f/foreach
