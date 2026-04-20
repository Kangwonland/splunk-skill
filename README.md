# splunk-skill — Claude Code Plugin

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Splunk](https://img.shields.io/badge/Splunk_Enterprise-9.4-green.svg)
![Rules](https://img.shields.io/badge/SPL_Rules-86-blue.svg)
![Benchmark](https://img.shields.io/badge/Benchmark-95.83%25-brightgreen.svg)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)
![Security Scan](https://img.shields.io/badge/Skill_Scanner-Passed-brightgreen.svg)

Splunk skills for Claude Code — authoritative **SPL/SPL2 best practices** plus
**environment-aware SPL generation** via your own Splunk metadata.

> 🇰🇷 **한국어 문서**: [README.ko.md](README.ko.md)

---

## Two Skills, One Plugin

| Skill | Purpose | Trigger |
|-------|---------|---------|
| **`splunk-spl`** | SPL/SPL2 best practices, optimization, migration | Any SPL/Splunk search question |
| **`splunk-env`** | Register your indexes, sourcetypes, hosts → splunk-spl reads this | English environment setup |
| **`splunk-env-ko`** | 한글 환경 설정 (identical schema) | 한국어 환경 설정 요청 |

**How they work together**: `splunk-env` stores your environment in `env-config.md`.
When you ask `splunk-spl` to write a query, it reads that file and produces SPL with
the **correct `index=`, `sourcetype=`, `host=` values for your environment** — instead
of generic `index=main`.

---

## 1. splunk-spl — SPL/SPL2 Best Practices

Provides authoritative SPL/SPL2 patterns, performance optimization rules, and
reference material for Splunk Enterprise 9.4. Claude automatically applies these
guidelines when you write, review, or optimize Splunk searches.

### Coverage

- **86 best practice rules** across 25 categories
- **21 reference tables** (command indexes, function catalogs, limits.conf values)
- SPL → SPL2 migration guide
- Splunk CLI and REST API patterns
- All rules reference `help.splunk.com` (Splunk Enterprise 9.4)

### Categories

| Priority | Category | Rules |
|----------|----------|-------|
| CRITICAL | Core Basics, Time Modifiers, Performance | 18 |
| HIGH | Extraction, Eval, Stats, Subsearch, Join, Lookup, Macros, SPL2, Migration | 42 |
| MEDIUM | Transaction, Iteration, Data Models, tstats, Summary, Output, CLI, REST | 19 |
| LOW-MEDIUM | Geo, Anomaly, Prediction, Metrics, Set Operations | 7 |

### Benchmark

With `splunk-spl` skill: **95.83%** vs without skill: 83.33% (+12.50pp, 36 evals)

---

## 2. splunk-env — Environment Metadata

Manages `env-config.md` — a pre-filled markdown file where you register your Splunk
environment (indexes, sourcetypes, hosts, lookups, macros). `splunk-spl` reads it
automatically to generate **environment-tailored SPL**.

### Without env-config.md (generic)

```spl
index=main sourcetype=firewall action=blocked
| stats count by src_ip
```

### With env-config.md (tailored to your environment)

```spl
index=sec_logs sourcetype=pan:traffic action=blocked
| stats count by src_ip
| lookup asset_db.csv ip AS src_ip OUTPUT hostname, owner
```

### Input Items (Tiered Schema)

| Tier | Section | Required | Content |
|------|---------|----------|---------|
| **1** | `Metadata` | ✅ | Deployment type, Indexer/SH count, Splunk version |
| **1** | `Indexes` | ✅ | Name, Purpose, Sourcetypes, **Use When** (critical for SPL generation) |
| **1** | `Sourcetypes` | ✅ | Sourcetype, Indexes, Category, Vendor/Product |
| **1** | `SPL Generation Notes` | ✅ | Scenario → correct `index=... sourcetype=...` mapping |
| **2** | `Hosts & Host Groups` | Recommended | Key hosts + naming patterns (e.g., `fw-*`) |
| **2** | `Lookups` | Optional | Name, Type (CSV/KV Store), Key/Output fields |
| **2** | `Macros` | Optional | Name, Definition, Args |
| **2** | `Data Models` | Optional | Name, Accelerated, Retention, Key Datasets |
| **3** | `Eventtypes & Tags` | Optional | Eventtype, Search, Tags |
| **3** | `Key Field Names` | Optional | Field, Description, Sourcetypes |
| **3** | `Installed TAs` | Optional | App, Version, Sourcetypes provided |
| **3** | `KV Store Collections` | Optional | Collection, App, Key fields |

Full schema: [`skills/splunk-env/references/schema-reference.md`](skills/splunk-env/references/schema-reference.md)

### Editing

- **Direct edit**: open `skills/splunk-env/env-config.md` and fill in your environment
- **Conversational**: tell Claude your environment in plain text and the skill edits the file
- **Auto-import**: scan your `.conf` files or run discovery queries
  (see [`references/conf-parsing-guide.md`](skills/splunk-env/references/conf-parsing-guide.md)
  and [`references/discovery-queries.md`](skills/splunk-env/references/discovery-queries.md))

---

## Installation

### Option A — Online (Recommended)

```bash
# splunk-spl (SPL best practices)
claude plugin install https://github.com/Kangwonland/splunk-skill@v1.0.0-splunk9.4

# splunk-env (environment metadata)
claude plugin install https://github.com/Kangwonland/splunk-skill@env-v1.0.2
```

### Option B — Airgap / Manual Install

For environments without GitHub access:

1. **Download the `.skill` bundle** from the Releases page on a machine with internet:
   - splunk-spl: [`splunk-spl.skill`](https://github.com/Kangwonland/splunk-skill/releases/tag/v1.0.0-splunk9.4)
   - splunk-env (English): [`splunk-env.skill`](https://github.com/Kangwonland/splunk-skill/releases/tag/env-v1.0.2)
   - splunk-env (Korean): `splunk-env-ko.skill` (same release)

2. **Transfer** the `.skill` file to the airgap host.

3. **Extract** into Claude Code's skills directory (`.skill` files are ZIP archives):

   ```bash
   mkdir -p ~/.claude/skills
   unzip splunk-spl.skill -d ~/.claude/skills/
   unzip splunk-env.skill -d ~/.claude/skills/
   # Or Korean:
   unzip splunk-env-ko.skill -d ~/.claude/skills/
   ```

   Resulting layout:
   ```
   ~/.claude/skills/
   ├── splunk-spl/
   │   ├── SKILL.md
   │   └── references/
   ├── splunk-env/
   │   ├── SKILL.md
   │   ├── env-config.md       ← edit this
   │   └── references/
   └── splunk-env-ko/
       └── ...
   ```

4. **Restart Claude Code**. The skills auto-load from `~/.claude/skills/`.

---

## Usage

Claude automatically selects the right skill based on your question:

| You ask | Skill triggered |
|---------|-----------------|
| "Write an SPL to find failed logins" | `splunk-spl` |
| "How do I optimize this tstats query?" | `splunk-spl` |
| "Register my indexes: sec_logs, web_prod" | `splunk-env` |
| "내 방화벽 호스트는 fw-*야" | `splunk-env-ko` |

Or invoke explicitly:

```
/splunk-spl      # SPL generation
/splunk-env      # environment setup (English)
/splunk-env-ko   # 환경 설정 (한국어)
```

---

## Versioning

Independent release tags for each skill:

- **splunk-spl**: `v{skill-version}-splunk{splunk-version}` — e.g., `v1.0.0-splunk9.4`
- **splunk-env**: `env-v{version}` — e.g., `env-v1.0.2`

Branches (e.g., `v9.4`) are maintained for hotfixes.

---

## Reference

- Splunk docs: `https://help.splunk.com/en/splunk-enterprise/search` (v9.4)
- Claude Code skills: `https://docs.claude.com/en/docs/claude-code/skills`

> Note: `docs.splunk.com` is deprecated. All SPL rules link to `help.splunk.com`.

## License

MIT
