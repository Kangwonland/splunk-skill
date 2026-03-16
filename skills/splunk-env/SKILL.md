---
name: splunk-env
description: >
  TRIGGER when: user wants to configure Splunk environment metadata,
  mentions /splunk-env, "my indexes", "our sourcetypes", "configure Splunk
  environment", or asks to save/register their Splunk deployment's index
  names, sourcetype names, hosts, lookups, or macros.
  Also trigger when: user asks to edit env-config.md or update their
  Splunk environment settings.
  DO NOT TRIGGER when: user asks about SPL syntax, search commands,
  query optimization, or best practices (handled by splunk-spl).
  DO NOT TRIGGER when: user writes in Korean (use splunk-env-ko).
  Manages Splunk environment metadata in env-config.md so the splunk-spl
  skill generates environment-specific SPL.
license: MIT
metadata:
  author: splunk-skill
  version: "1.0.0"
---

# Splunk Environment Metadata Configuration

This skill manages `env-config.md` — a pre-filled configuration file where users
register their Splunk environment (indexes, sourcetypes, hosts, lookups, macros).
The **splunk-spl** skill reads this file to generate **environment-specific SPL**.

## How It Works

1. User edits **`env-config.md`** in this skill directory with their environment info
2. When generating SPL, the splunk-spl skill reads env-config.md automatically
3. SPL is generated with the correct index names, sourcetypes, and field names

**Without env-config.md** (generic):
```spl
index=main sourcetype=firewall action=blocked
| stats count by src_ip
```

**With env-config.md** (environment-specific):
```spl
index=sec_logs sourcetype=pan:traffic action=blocked
| stats count by src_ip
| lookup asset_db.csv ip AS src_ip OUTPUT hostname, owner
```

---

## env-config.md

The configuration file is at **`env-config.md`** in this skill's directory.
It ships with example data — the user replaces it with their own environment.

### When this skill is triggered:

1. **Read** `env-config.md` to understand the current configuration.
2. **Help the user edit it**: add, modify, or remove entries as requested.
3. **Write** the updated file back.

### File Structure

The file uses markdown tables organized by category:

| Section | Required | What It Contains |
|---------|----------|-----------------|
| **Metadata** | Yes | Deployment type, indexer/SH count, version |
| **Indexes** | Yes | Name, purpose, sourcetypes, use-when scenarios |
| **Sourcetypes** | Yes | Name, indexes, category, vendor/product |
| **SPL Generation Notes** | Yes | Scenario → index+sourcetype mappings |
| **Hosts & Host Groups** | Recommended | Key hosts and naming patterns |
| **Lookups** | Optional | Name, type, key/output fields |
| **Macros** | Optional | Name, definition, args |
| **Data Models** | Optional | Name, acceleration, datasets |

### Rules for Editing

- **Only include sections that have data** — delete sections the user doesn't need.
- Group indexes by domain (### Security, ### Web, etc.) when 10+ indexes.
- Each index must have a **Use When** column — this is what guides SPL generation.
- **SPL Generation Notes** are critical — they map scenarios to exact index+sourcetype.
- Keep the markdown table format consistent.

---

## User Interaction

When a user triggers this skill, they typically want to:

### Register environment info

User provides index/host/sourcetype info in any format:

> "Our indexes: sec_logs (firewall), web_prod (apache), infra (syslog).
> Firewalls are named fw-*. We have 3 indexers, distributed."

Read the current env-config.md, merge the new info, show the result, write it back.

### Edit existing entries

> "Change web_prod to web_production" or "Add a new macro: `all_sec`"

Read env-config.md, apply the change, write back.

### View current config

> "Show my Splunk environment" or "What's in my env config?"

Read and display env-config.md.

### Import from conf files (admin only)

> "Scan /opt/splunk conf files"

Parse conf files (see **references/conf-parsing-guide.md**), merge results into
env-config.md. Security: never read passwords/secrets, never store file paths.

### Import from SPL discovery

> "Here are my discovery query results: ..."

Parse results (see **references/discovery-queries.md**), merge into env-config.md.

---

## Integration with splunk-spl

The splunk-spl skill should read `env-config.md` when generating SPL.
Key sections it uses:

- **Indexes** → correct `index=` values
- **Sourcetypes** → correct `sourcetype=` values
- **SPL Generation Notes** → scenario-to-query mappings
- **Hosts & Host Groups** → `host=` patterns
- **Lookups** → `| lookup` enrichment
- **Macros** → `\`macro_name\`` usage

---

## Reference Files

| File | Contents |
|------|----------|
| `env-config.md` | **The configuration file** — users edit this |
| `references/conf-parsing-guide.md` | Conf file extraction commands (admin only) |
| `references/discovery-queries.md` | SPL discovery queries |
| `references/schema-reference.md` | Full env-config.md schema |
| `references/sample-env.md` | Complete example |

---

## Limitations

- **Not a live connection**: Static metadata, not real-time Splunk queries.
- **Manual updates**: Users must update env-config.md when their environment changes.
- **Per-installation**: Each plugin installation has its own env-config.md.
