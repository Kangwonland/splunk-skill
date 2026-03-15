# splunk-spl — Claude Code Plugin

Splunk SPL and SPL2 best practices skill for Claude Code.

## What It Does

Provides authoritative SPL/SPL2 patterns, performance optimization rules,
and reference material for Splunk Enterprise 9.4. Claude automatically
applies these guidelines when you write, review, or optimize Splunk searches.

## Coverage

- 86 best practice rules across 25 categories
- 21 reference tables (command indexes, function catalogs, limits.conf values)
- SPL → SPL2 migration guide
- Splunk CLI and REST API patterns
- All rules reference `help.splunk.com` (Splunk Enterprise 9.4)

## Categories

| Priority | Category | Rules |
|----------|----------|-------|
| CRITICAL | Core Basics, Time Modifiers, Performance | 18 |
| HIGH | Extraction, Eval, Stats, Subsearch, Join, Lookup, Macros, SPL2, Migration | 38 |
| MEDIUM | Transaction, Iteration, Data Models, tstats, Summary, Output, CLI, REST | 22 |
| LOW-MEDIUM | Geo, Anomaly, Prediction, Metrics, Set Operations | 8 |

## Installation

```bash
# Latest version
claude plugin install https://github.com/Kangwonland/splunk-skill

# Specific version
claude plugin install https://github.com/Kangwonland/splunk-skill@v1.0.0-splunk9.4
```

## Usage

Claude automatically uses this skill when you ask about Splunk searches.

## Versioning

Tags follow the format `v{skill-version}-splunk{splunk-version}`:
- `v1.0.0-splunk9.4` — Skill v1.0.0 for Splunk Enterprise 9.4

Branches (e.g., `v9.4`) are maintained for hotfixes.

## Reference

All documentation sourced from:
`https://help.splunk.com/en/splunk-enterprise/search` (v9.4)

> Note: `docs.splunk.com` is deprecated. All URLs use `help.splunk.com`.

## License

MIT
