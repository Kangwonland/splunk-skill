# SPL2 Compatibility and Platform Requirements

## Platform Availability (v9.4)
| Platform | SPL2 Status |
|----------|-------------|
| Splunk Cloud Platform ≥ v9.4.2510 | GA |
| Splunk Enterprise v9.4 (Linux) | GA |
| Splunk Enterprise v9.4 (Windows) | Not available |
| Splunk Enterprise < v9.4 | Not available |
| Splunk Enterprise Security | Via app context |

## Feature Availability by Version
| Feature | Min Version |
|---------|-------------|
| `from` / `into` | 9.4 |
| `branch` / `route` | 9.4 |
| `spl1` wrapper | 9.4 |
| SPL2 modules (`.spl2` files) | 9.4 |
| Lambda expressions | 9.4 |
| Explicit data types | 9.4 |
| `expand` / `flatten` | 9.4 |
| `ocsf` | 9.4 (requires add-on) |

## Compatibility Profile Levels
| Level | Description |
|-------|-------------|
| SPL1 only | Classic mode, no SPL2 |
| SPL2 with SPL1 wrapper | SPL2 default with `\| spl1` for legacy commands |
| SPL2 strict | SPL2 only, no SPL1 wrapper allowed |

## Key Behavioral Differences
| Behavior | SPL1 | SPL2 |
|----------|------|------|
| Type coercion | Implicit | Explicit cast required |
| Wildcard search | `field=val*` | `like(field, "val%")` |
| Case sensitivity | Case-insensitive by default | Case-sensitive (use `ilike`) |
| Comments | Not supported | `/* ... */` |
| Stats renaming | Optional AS | Required AS |
| Sort direction | `+`/`-` prefix | `ASC`/`DESC` suffix |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl2-search-reference/9.4/spl2-overview/about-spl2
