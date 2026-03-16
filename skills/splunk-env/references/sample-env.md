# Sample: splunk-env.md

This is a complete example of a filled-out Splunk environment profile with
Tier 1 and Tier 2 data.

---

```markdown
# Splunk Environment Profile

## Metadata
- **Deployment**: distributed
- **Indexers**: 3
- **Search Heads**: 2 (search head cluster)
- **Splunk Version**: 9.4.0
- **Last Updated**: 2026-03-17
- **Tiers Configured**: 1-2

## Indexes

### Security
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| sec_logs | Firewall and VPN traffic logs | pan:traffic, cisco:asa | Network attacks, policy violations, VPN brute force |
| sec_events | Endpoint security events | crowdstrike:events, sysmon | Malware detection, process anomalies, endpoint compromise |
| sec_audit | Security audit trails | WinEventLog:Security | Account changes, privilege escalation, logon events |

### Web
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| web_prod | Production web server logs | apache:access, apache:error | HTTP errors, performance, access patterns, bot detection |
| web_staging | Staging environment logs | apache:access | Pre-production testing, deployment validation |

### Infrastructure
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| infra_syslog | Linux and network syslog | syslog | System health, auth failures, device status |
| winevent | Windows event logs | WinEventLog:System, WinEventLog:Application | Service failures, system errors, app crashes |
| infra_metrics | Infrastructure metrics | collectd_http | CPU, memory, disk monitoring |

## Sourcetypes
| Sourcetype | Indexes | Category | Vendor/Product |
|------------|---------|----------|---------------|
| pan:traffic | sec_logs | Network Security | Palo Alto Networks Firewall |
| cisco:asa | sec_logs | Network Security | Cisco ASA Firewall |
| crowdstrike:events | sec_events | Endpoint Security | CrowdStrike Falcon |
| sysmon | sec_events | Endpoint Security | Microsoft Sysmon |
| WinEventLog:Security | sec_audit | Windows Security | Microsoft Windows |
| WinEventLog:System | winevent | Windows System | Microsoft Windows |
| WinEventLog:Application | winevent | Windows Application | Microsoft Windows |
| apache:access | web_prod, web_staging | Web | Apache HTTP Server |
| apache:error | web_prod | Web | Apache HTTP Server |
| syslog | infra_syslog | Infrastructure | Generic Syslog (RFC 3164/5424) |
| collectd_http | infra_metrics | Metrics | collectd |

## SPL Generation Notes
- Security correlation (all): `(index=sec_logs OR index=sec_events OR index=sec_audit)`
- VPN/firewall attacks: `index=sec_logs sourcetype=pan:traffic`
- Endpoint threat hunting: `index=sec_events sourcetype=crowdstrike:events`
- Windows logon analysis: `index=sec_audit sourcetype=WinEventLog:Security EventCode IN (4624, 4625, 4648)`
- Privilege escalation: `index=sec_audit sourcetype=WinEventLog:Security EventCode IN (4672, 4673, 4674)`
- Web 500 errors: `index=web_prod sourcetype=apache:access status>=500`
- Web bot detection: `index=web_prod sourcetype=apache:access | where like(useragent, "%bot%")`
- Linux auth failures: `index=infra_syslog sourcetype=syslog "authentication failure"`
- Cross-source IP investigation: `(index=sec_logs OR index=sec_events OR index=web_prod) src_ip=<ip>`
- Process anomaly (endpoint): `index=sec_events sourcetype=sysmon EventCode=1`

## Lookups
| Name | Type | Key Fields | Output Fields | Purpose |
|------|------|-----------|---------------|---------|
| asset_db.csv | CSV | ip | hostname, owner, department, criticality | Asset enrichment for IP-based events |
| user_identity | KV Store | username | full_name, department, role, manager | User identity correlation |
| geo_private_ranges.csv | CSV | cidr | location, zone | Internal network zone classification |
| threat_intel_ip.csv | CSV | ip | threat_category, confidence, source | Known malicious IP matching |

## Macros
| Name | Definition | Args | Purpose |
|------|-----------|------|---------|
| sec_index | `index=sec_logs` | 0 | Security index shorthand |
| all_sec | `(index=sec_logs OR index=sec_events OR index=sec_audit)` | 0 | All security indexes |
| get_severity(1) | `case($level$>7,"critical",$level$>4,"high",$level$>2,"medium",true(),"low")` | 1 | Severity classification |
| private_ip | `(src_ip="10.*" OR src_ip="172.16.*" OR src_ip="192.168.*")` | 0 | Private IP filter |

## Data Models
| Name | Accelerated | Retention | Key Datasets |
|------|------------|-----------|-------------|
| Network_Traffic | Yes | 7 days | All_Traffic, Blocked_Traffic, Allowed_Traffic |
| Authentication | Yes | 30 days | Authentication, Failed_Authentication |
| Endpoint | No | — | Processes, Services, Filesystem_Changes |

## Hosts & Host Groups

### Key Hosts
| Host | Role | Index | Sourcetypes |
|------|------|-------|-------------|
| fw-hq-01 | Primary HQ firewall | sec_logs | pan:traffic |
| fw-hq-02 | Secondary HQ firewall | sec_logs | pan:traffic |
| dc-hq-01 | HQ domain controller | sec_audit, winevent | WinEventLog:Security, WinEventLog:System |
| web-prod-lb | Production load balancer | web_prod | apache:access |

### Host Groups (Naming Patterns)
| Pattern | Role | Count | Location |
|---------|------|-------|----------|
| fw-* | Firewalls | 12 | All sites |
| dc-* | Domain controllers | 4 | Internal (HQ, DR) |
| web-prod-* | Production web servers | 8 | DMZ |
| web-stg-* | Staging web servers | 3 | Internal |
| linux-* | Linux servers | 25 | Mixed |
```

---

## Corresponding MEMORY.md Pointer

```markdown
## Splunk Environment
- [splunk-env.md](./splunk-env.md) — Security: sec_logs, sec_events, sec_audit | Web: web_prod, web_staging | Infra: infra_syslog, winevent, infra_metrics | topology: distributed (3 idx, 2 sh) | tiers: 1-2
```
