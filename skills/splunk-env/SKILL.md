---
name: splunk-env
description: >
  TRIGGER when: user wants to configure Splunk environment metadata,
  mentions /splunk-env, "my indexes", "our sourcetypes", "configure Splunk
  environment", imports Splunk conf files (indexes.conf, props.conf,
  transforms.conf, macros.conf), or asks to save/register their Splunk
  deployment's index names, sourcetype names, lookups, or macros.
  DO NOT TRIGGER when: user asks about SPL syntax, search commands,
  query optimization, or best practices (handled by splunk-spl).
  DO NOT TRIGGER when: user writes in Korean (use splunk-env-ko).
  Captures Splunk environment metadata and stores it in Claude memory
  so the splunk-spl skill generates environment-specific SPL.
license: MIT
metadata:
  author: splunk-skill
  version: "1.0.0"
---

# Splunk Environment Metadata Configuration

This skill captures your Splunk environment metadata (indexes, sourcetypes,
lookups, macros, etc.) and stores it in Claude memory. When the **splunk-spl**
skill later generates SPL queries, it automatically references your environment
data to produce **environment-specific SPL** instead of generic queries.

## How It Works

**Without splunk-env** (generic SPL):
```spl
index=main sourcetype=firewall action=blocked
| stats count by src_ip, dest_ip
```

**With splunk-env** (environment-specific SPL):
```spl
index=sec_logs sourcetype=pan:traffic action=blocked
| stats count by src_ip, dest_ip
```

The difference: Claude knows your actual index names, sourcetypes, and field
names — no guessing, no placeholders.

---

## Quick Start

Three methods to register your environment. Choose what fits:

| Method | Best For | How |
|--------|----------|-----|
| **A. Conf File Import** | Splunk admins with server access | Point to `$SPLUNK_HOME`, auto-parse conf files |
| **B. SPL Discovery** | Users with search access | Run discovery SPL queries, paste results |
| **C. Manual Input** | Anyone | Type or paste your index/sourcetype lists |

**Example prompts:**
- *"Scan my Splunk conf files at /opt/splunk and extract environment metadata"*
- *"Here are my indexes: sec_logs, web_prod, infra_syslog. Save them."*
- *"I ran `| eventcount summarize=false index=*` — here are the results: ..."*

---

## Method A: Conf File Import

Parse Splunk configuration files to automatically extract environment metadata.

### Steps

1. **Ask the user** for their `$SPLUNK_HOME` path (e.g., `/opt/splunk`).
2. **Scan** conf files using safe extraction commands.
3. **Present** extracted data for user confirmation.
4. **Save** confirmed data to `memory/splunk-env.md`.

### Scan Scope

Scan these directories in order:
- `$SPLUNK_HOME/etc/system/local/` — system-level overrides
- `$SPLUNK_HOME/etc/apps/*/local/` — per-app customizations
- `$SPLUNK_HOME/etc/apps/*/default/` — app defaults

When 30+ indexes are found, group by prefix and let the user select relevant groups.

### Parsing Rules

See **references/conf-parsing-guide.md** for detailed per-file extraction commands.

### Security Rules

- **NEVER** read `external_cmd`, `external_type`, `password`, `secret`, `token`, or `key` fields from any conf file
- **NEVER** store `$SPLUNK_HOME` paths in `splunk-env.md` (infrastructure exposure risk)
- Extract **only** metadata: names, definitions, types — nothing executable or sensitive

---

## Method B: SPL Discovery

Run SPL queries to discover environment metadata from a running Splunk instance.

### Steps

1. **Provide** the appropriate discovery query for each category.
2. **User runs** the query in Splunk and pastes results.
3. **Parse** results — accept CSV, tab-separated, or one-per-line formats.
4. **Confirm** extracted data with the user before saving.

### Discovery Queries

See **references/discovery-queries.md** for the complete set of Tier 1/2/3 queries.

---

## Method C: Manual Input

Accept environment metadata in any format the user provides.

### Accepted Formats

- Comma-separated lists: `sec_logs, web_prod, infra_syslog`
- One per line
- Free-form text: *"We have sec_logs for security, web_prod for web traffic"*
- Table/CSV paste from Splunk

Parse the input, confirm with the user, then save.

---

## Progressive Disclosure

Collect metadata in tiers. Complete Tier 1 first, then offer higher tiers.

### Tier 1 (Required)

| Category | What to Collect |
|----------|----------------|
| **Indexes** | Name, purpose, associated sourcetypes, use-when scenarios |
| **Sourcetypes** | Name, associated indexes, category, vendor/product |
| **Topology** | Deployment type (standalone/distributed/cloud), indexer count, search head count |

### Tier 2 (Recommended)

| Category | What to Collect |
|----------|----------------|
| **Lookups** | Filename/name, type (CSV/KV Store), key fields, output fields |
| **Macros** | Name, definition, argument count |
| **Data Models** | Name, acceleration status, retention, key datasets |
| **Hosts & Host Groups** | Key hosts (name, role, index, sourcetypes), naming patterns (pattern, role, count, location) |

### Tier 3 (Optional)

| Category | What to Collect |
|----------|----------------|
| **Eventtypes & Tags** | Name, search definition, associated tags |
| **Key Field Names** | Field name, description, sourcetypes where found |
| **Installed TAs** | App name, version, sourcetypes provided |
| **KV Store Collections** | Collection name, app, key fields |

### After Tier 1

Prompt: *"Indexes and sourcetypes saved. Would you like to add lookups, macros, data models, or host information as well?"*

---

## Storage Format

All environment metadata is stored in `memory/splunk-env.md` within the
user's project memory directory.

### Rules

- **Only include sections that have data** — never create empty tables.
- If a category was checked but has no data: `## Lookups\nNone configured.`
- Group indexes by domain when 10+ indexes exist (### Security, ### Web, etc.)
- Each index row includes a **Use When** column for SPL generation guidance.
- Include **SPL Generation Notes** mapping common scenarios to index+sourcetype combinations.
- Support multiple instances with `## Instance: Production` / `## Instance: Development` headers.

### Schema

See **references/schema-reference.md** for the full output schema definition.
See **references/sample-env.md** for a complete filled-out example.

### Index Table Example

```markdown
## Indexes

### Security
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| sec_logs | Firewall/VPN traffic | pan:traffic, cisco:asa | Network attacks, policy violations |
| sec_events | Endpoint security | crowdstrike:events | Malware, process anomalies |

### Web
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| web_prod | Production web servers | apache:access | Web errors, performance, access analysis |

## SPL Generation Notes
- Security correlation: `(index=sec_logs OR index=sec_events)`
- VPN attacks: `index=sec_logs sourcetype=pan:traffic`
- Web 500 errors: `index=web_prod sourcetype=apache:access status>=500`
```

---

## MEMORY.md Update

After saving `splunk-env.md`, add a pointer with a category summary to `MEMORY.md`:

```markdown
## Splunk Environment
- [splunk-env.md](./splunk-env.md) — Security: sec_logs, sec_events | Web: web_prod | Infra: infra_syslog, winevent | topology: distributed | tiers: 1-2
```

### Why Include a Summary

Claude auto-loads only `MEMORY.md`. With key values in the pointer:
1. Simple SPL generation uses the summary directly — no extra file read needed.
2. Complex requests trigger Claude to read `splunk-env.md` for full details.

### Rules

- For 30+ indexes: use category names + representative indexes only (details in splunk-env.md).
- If a pointer already exists: **update** it — never add a duplicate.
- Include: key index names, topology type, and which tiers are filled.

---

## Update & Extend

This skill also handles modifications to an existing environment profile.

### Operations

| User Request | Action |
|-------------|--------|
| *"Add macros to my env"* | Append the new section to `splunk-env.md` |
| *"Change index web_prod to web_production"* | Update that specific entry |
| *"Rescan my conf files"* | Re-parse, merge with existing data, show changes |
| *"Show my current environment"* | Read and display `splunk-env.md` |

### Existing File Handling

When `splunk-env.md` already exists:
1. Read and display the current profile to the user.
2. Ask: *"Would you like to update the existing profile or start fresh?"*
3. On update: merge new data, preserve unchanged sections.
4. On fresh: replace entirely after confirmation.

---

## Integration with splunk-spl

When `splunk-env.md` exists, the splunk-spl skill automatically generates
environment-aware SPL:

**Query**: *"Find brute force attacks on our VPN"*

Without env:
```spl
index=main sourcetype=vpn action=failure
| stats count by src_ip
| where count > 10
```

With env:
```spl
index=sec_logs sourcetype=pan:traffic action=blocked app=ssl-vpn
| stats count by src_ip
| where count > 10
| lookup asset_db.csv ip AS src_ip OUTPUT hostname, owner
```

The environment data provides: correct index, correct sourcetype, relevant
lookup enrichment, and field names that match the actual data.

---

## Reference Files

| File | Contents |
|------|----------|
| `references/conf-parsing-guide.md` | Per-conf-file extraction commands and security rules |
| `references/discovery-queries.md` | Tier 1/2/3 SPL discovery queries |
| `references/schema-reference.md` | Full `splunk-env.md` output schema |
| `references/sample-env.md` | Complete example environment file |

---

## Limitations

- **Not a live connection**: This skill stores static metadata — it does not query Splunk in real-time.
- **Conf inheritance**: Complex conf inheritance chains (system → app default → app local) may result in missing overrides. Always confirm extracted data with the user.
- **Project scope**: Environment data is stored per-project in Claude memory. Different projects need separate setup.
- **Staleness**: If the Splunk environment changes (new indexes, retired sourcetypes), the user must update the profile manually or re-run discovery.
