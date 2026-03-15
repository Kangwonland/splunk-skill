# Search Directives Reference

Search directives modify Splunk search optimizer behavior. They appear
in the search string before `|`.

## Available Directives

| Directive | Syntax | Description |
|-----------|--------|-------------|
| `REQUIRED_TAGS` | `REQUIRED_TAGS(tag1, tag2)` | Only search events tagged with all specified tags |
| `REQUIRED_EVENTTYPES` | `REQUIRED_EVENTTYPES(et1, et2)` | Only search events matching specified eventtypes |
| `READ_SUMMARY` | `READ_SUMMARY` | Force use of summary index data only |

## Examples

**REQUIRED_TAGS:**
```spl
REQUIRED_TAGS(authentication, network)
| stats count by action
```
Splunk optimizer skips sourcetypes that don't have both tags.

**REQUIRED_EVENTTYPES:**
```spl
REQUIRED_EVENTTYPES(successful_login, failed_login)
| stats count by user
```
Restricts search to events matching specified eventtypes.

**READ_SUMMARY:**
```spl
READ_SUMMARY
index=summary sourcetype=stash
| stats sum(count) by host
```
Forces reading from summary index, preventing live data mixing.

## Notes
- Directives improve performance by reducing scope before search begins.
- Multiple directives can be combined in one search string.
- `REQUIRED_TAGS` and `REQUIRED_EVENTTYPES` work through `tags.conf` and `eventtypes.conf`.
- Directive names are case-insensitive.
- Directives appear BEFORE the first `|` pipe.

Source: https://help.splunk.com/en/splunk-enterprise/search/search-manual/9.4/optimize-searches/about-optimizing-searches
