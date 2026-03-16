# Splunk Environment Configuration
#
# Fill in your Splunk environment information below.
# Only fill sections relevant to you — delete or leave empty sections you don't need.
# The splunk-spl skill reads this file to generate environment-specific SPL.

## Metadata

- **Deployment**: distributed
- **Indexers**: 3
- **Search Heads**: 2
- **Splunk Version**: 9.4.0

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

### Infrastructure
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| infra_syslog | Linux/network syslog | syslog | System health, auth failures |
| winevent | Windows event logs | WinEventLog:Security, WinEventLog:System | Windows auth, service status |

## Sourcetypes

| Sourcetype | Indexes | Category | Vendor/Product |
|------------|---------|----------|---------------|
| pan:traffic | sec_logs | Network Security | Palo Alto Networks Firewall |
| cisco:asa | sec_logs | Network Security | Cisco ASA |
| crowdstrike:events | sec_events | Endpoint Security | CrowdStrike Falcon |
| apache:access | web_prod | Web | Apache HTTP Server |
| syslog | infra_syslog | Infrastructure | Generic Syslog |
| WinEventLog:Security | winevent | Windows | Microsoft Windows |

## SPL Generation Notes

- Security correlation: `(index=sec_logs OR index=sec_events)`
- VPN/firewall attacks: `index=sec_logs sourcetype=pan:traffic`
- Web 500 errors: `index=web_prod sourcetype=apache:access status>=500`
- Windows logon failures: `index=winevent sourcetype=WinEventLog:Security EventCode=4625`

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

## Lookups

| Name | Type | Key Fields | Output Fields | Purpose |
|------|------|-----------|---------------|---------|
| asset_db.csv | CSV | ip | hostname, owner, department | Asset enrichment |
| user_identity | KV Store | username | department, role, manager | User context |

## Macros

| Name | Definition | Args | Purpose |
|------|-----------|------|---------|
| sec_index | `index=sec_logs` | 0 | Security index shorthand |
| get_severity(1) | `case($level$>7,"critical",$level$>4,"high")` | 1 | Severity classification |

## Data Models

| Name | Accelerated | Retention | Key Datasets |
|------|------------|-----------|-------------|
| Network_Traffic | Yes | 7 days | All_Traffic, Blocked_Traffic |
| Authentication | Yes | 30 days | Failed_Authentication |
