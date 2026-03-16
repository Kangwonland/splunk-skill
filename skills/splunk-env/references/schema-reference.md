# splunk-env.md Schema Reference

This document defines the output format for `memory/splunk-env.md` — the file
where Splunk environment metadata is stored for Claude to reference.

---

## File Structure

```markdown
# Splunk Environment Profile

## Metadata
- **Deployment**: [standalone | distributed | Splunk Cloud]
- **Indexers**: [count]
- **Search Heads**: [count]
- **Splunk Version**: [version]
- **Last Updated**: [YYYY-MM-DD]
- **Tiers Configured**: [1 | 1-2 | 1-3]

## Indexes
[index tables grouped by domain]

## Sourcetypes
[sourcetype table]

## SPL Generation Notes
[scenario-to-index+sourcetype mappings]

## Lookups
[lookup table — Tier 2]

## Macros
[macro table — Tier 2]

## Data Models
[data model table — Tier 2]

## Hosts & Host Groups
[hosts and naming patterns — Tier 2]

## Eventtypes & Tags
[eventtype table — Tier 3]

## Key Field Names
[field names table — Tier 3]

## Installed TAs
[TA table — Tier 3]

## KV Store Collections
[KV store table — Tier 3]
```

---

## Section Schemas

### Metadata (Required)

Always present. Contains deployment overview.

```markdown
## Metadata
- **Deployment**: distributed
- **Indexers**: 3
- **Search Heads**: 2
- **Splunk Version**: 9.4.0
- **Last Updated**: 2026-03-17
- **Tiers Configured**: 1-2
```

### Indexes (Required — Tier 1)

Group by domain when 10+ indexes exist. Each row includes a **Use When**
column to guide SPL generation.

```markdown
## Indexes

### Security
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| sec_logs | Firewall/VPN traffic | pan:traffic, cisco:asa | Network attacks, policy violations, VPN issues |
| sec_events | Endpoint security | crowdstrike:events | Malware, process anomalies, endpoint compromise |

### Web
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| web_prod | Production web servers | apache:access, apache:error | Web errors, performance, access patterns |

### Infrastructure
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| infra_syslog | Linux/network syslog | syslog | System health, auth failures, device status |
| winevent | Windows event logs | WinEventLog:Security, WinEventLog:System | Windows auth, service status, GPO |
```

**Fewer than 10 indexes**: No subheadings needed — single flat table.

### Sourcetypes (Required — Tier 1)

```markdown
## Sourcetypes
| Sourcetype | Indexes | Category | Vendor/Product |
|------------|---------|----------|---------------|
| pan:traffic | sec_logs | Network Security | Palo Alto Networks Firewall |
| cisco:asa | sec_logs | Network Security | Cisco ASA |
| apache:access | web_prod | Web | Apache HTTP Server |
| syslog | infra_syslog | Infrastructure | Generic Syslog |
```

### SPL Generation Notes (Required — Tier 1)

Maps common use-case scenarios to the correct index+sourcetype combinations.

```markdown
## SPL Generation Notes
- Security correlation (all security data): `(index=sec_logs OR index=sec_events)`
- VPN/firewall attacks: `index=sec_logs sourcetype=pan:traffic`
- Endpoint threats: `index=sec_events sourcetype=crowdstrike:events`
- Web 500 errors: `index=web_prod sourcetype=apache:access status>=500`
- Windows logon failures: `index=winevent sourcetype=WinEventLog:Security EventCode=4625`
- Cross-source IP investigation: `(index=sec_logs OR index=web_prod OR index=infra_syslog) src_ip=<ip>`
```

### Lookups (Tier 2)

```markdown
## Lookups
| Name | Type | Key Fields | Output Fields | Purpose |
|------|------|-----------|---------------|---------|
| asset_db.csv | CSV | ip | hostname, owner, department | Asset enrichment |
| user_identity | KV Store | username | department, role, manager | User context |
```

### Macros (Tier 2)

```markdown
## Macros
| Name | Definition | Args | Purpose |
|------|-----------|------|---------|
| sec_index | `index=sec_logs` | 0 | Security index shorthand |
| get_severity(1) | `case($level$>7,"critical",$level$>4,"high",$level$>2,"medium",true(),"low")` | 1 | Severity classification |
```

### Data Models (Tier 2)

```markdown
## Data Models
| Name | Accelerated | Retention | Key Datasets |
|------|------------|-----------|-------------|
| Network_Traffic | Yes | 7 days | All_Traffic, Blocked_Traffic |
| Authentication | Yes | 30 days | Failed_Authentication |
```

### Hosts & Host Groups (Tier 2)

```markdown
## Hosts & Host Groups

### Key Hosts
| Host | Role | Index | Sourcetypes |
|------|------|-------|-------------|
| fw-hq-01 | Primary HQ firewall | sec_logs | pan:traffic |
| dc-hq-01 | HQ domain controller | winevent | WinEventLog:Security |

### Host Groups (Naming Patterns)
| Pattern | Role | Count | Location |
|---------|------|-------|----------|
| fw-* | Firewalls | 12 | All sites |
| web-prod-* | Production web servers | 8 | DMZ |
| dc-* | Domain controllers | 4 | Internal |
```

- **Key Hosts**: Critical individual devices (firewalls, DCs, etc.)
- **Host Groups**: Naming patterns for wildcard SPL generation (e.g., `host=fw-*`)
- **Conf source**: `inputs.conf` `[monitor://]` stanzas can reveal host info
- **SPL discovery**: `| metadata type=hosts index=* | table host | sort host`

### Eventtypes & Tags (Tier 3)

```markdown
## Eventtypes & Tags
| Eventtype | Search | Tags |
|-----------|--------|------|
| firewall_blocked | `sourcetype=pan:traffic action=blocked` | network, firewall, attack |
| failed_login | `sourcetype=WinEventLog:Security EventCode=4625` | authentication, failure |
```

### Key Field Names (Tier 3)

```markdown
## Key Field Names
| Field | Description | Sourcetypes |
|-------|------------|-------------|
| src_ip | Source IP address | pan:traffic, cisco:asa, syslog |
| dest_ip | Destination IP address | pan:traffic, cisco:asa |
| action | Firewall action (allowed/blocked) | pan:traffic |
| EventCode | Windows event ID | WinEventLog:Security, WinEventLog:System |
```

### Installed TAs (Tier 3)

```markdown
## Installed TAs
| App | Version | Sourcetypes Provided |
|-----|---------|---------------------|
| Splunk_TA_paloalto | 8.0.1 | pan:traffic, pan:system, pan:config |
| Splunk_TA_windows | 8.6.0 | WinEventLog:Security, WinEventLog:System |
```

### KV Store Collections (Tier 3)

```markdown
## KV Store Collections
| Collection | App | Key Fields | Purpose |
|-----------|-----|-----------|---------|
| user_identity | SA-IdentityManagement | username | User identity correlation |
| asset_lookup | SA-NetworkProtection | ip, mac | Asset inventory |
```

---

## Multiple Values in Columns

| Column | Format | Example |
|--------|--------|---------|
| Sourcetypes (in Index table) | Comma-separated | `pan:traffic, cisco:asa` |
| Indexes (in Sourcetype table) | Comma-separated | `infra_syslog, sec_logs` |
| Key Fields / Output Fields | Comma-separated | `ip, hostname` |
| Macro Args | Count only | `1` |
| Data Model Datasets | Comma-separated | `All_Traffic, Blocked_Traffic` |
| Eventtype Tags | Comma-separated | `network, firewall, attack` |
| TA Sourcetypes | Comma-separated | `pan:traffic, pan:system, pan:config` |

---

## Empty Category Handling

If a category was queried/checked but has no data:

```markdown
## KV Store Collections
None configured.
```

If a category was never checked, omit the section entirely.

---

## Multiple Instances

For organizations with separate Splunk deployments:

```markdown
# Splunk Environment Profile

## Instance: Production
### Metadata
...
### Indexes
...

## Instance: Development
### Metadata
...
### Indexes
...
```

Each instance has its own complete set of sections.

---

## MEMORY.md Pointer Format

The pointer in MEMORY.md includes a summary of key values:

```markdown
## Splunk Environment
- [splunk-env.md](./splunk-env.md) — Security: sec_logs, sec_events | Web: web_prod | Infra: infra_syslog, winevent | topology: distributed | tiers: 1-2
```

**For 30+ indexes**: Use category names + representative indexes only:
```markdown
- [splunk-env.md](./splunk-env.md) — Security (12 indexes): sec_logs, sec_events, ... | Web (8): web_prod, web_staging, ... | topology: cloud | tiers: 1-3
```
