# 예시: splunk-env.md

Tier 1과 Tier 2가 채워진 완성 예시 Splunk 환경 프로필입니다.

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
| sec_logs | 방화벽/VPN 트래픽 로그 | pan:traffic, cisco:asa | 네트워크 공격, 정책 위반, VPN 무차별 대입 |
| sec_events | 엔드포인트 보안 이벤트 | crowdstrike:events, sysmon | 악성코드 탐지, 프로세스 이상, 엔드포인트 침해 |
| sec_audit | 보안 감사 기록 | WinEventLog:Security | 계정 변경, 권한 상승, 로그온 이벤트 |

### Web
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| web_prod | 프로덕션 웹 서버 로그 | apache:access, apache:error | HTTP 에러, 성능, 접근 패턴, 봇 탐지 |
| web_staging | 스테이징 환경 로그 | apache:access | 배포 전 테스트, 배포 검증 |

### Infrastructure
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| infra_syslog | Linux/네트워크 syslog | syslog | 시스템 상태, 인증 실패, 장치 상태 |
| winevent | Windows 이벤트 로그 | WinEventLog:System, WinEventLog:Application | 서비스 실패, 시스템 에러, 앱 크래시 |
| infra_metrics | 인프라 메트릭 | collectd_http | CPU, 메모리, 디스크 모니터링 |

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
- 보안 상관분석 (전체): `(index=sec_logs OR index=sec_events OR index=sec_audit)`
- VPN/방화벽 공격: `index=sec_logs sourcetype=pan:traffic`
- 엔드포인트 위협 헌팅: `index=sec_events sourcetype=crowdstrike:events`
- Windows 로그온 분석: `index=sec_audit sourcetype=WinEventLog:Security EventCode IN (4624, 4625, 4648)`
- 권한 상승: `index=sec_audit sourcetype=WinEventLog:Security EventCode IN (4672, 4673, 4674)`
- 웹 500 에러: `index=web_prod sourcetype=apache:access status>=500`
- 웹 봇 탐지: `index=web_prod sourcetype=apache:access | where like(useragent, "%bot%")`
- Linux 인증 실패: `index=infra_syslog sourcetype=syslog "authentication failure"`
- IP 크로스소스 조사: `(index=sec_logs OR index=sec_events OR index=web_prod) src_ip=<ip>`
- 프로세스 이상 (엔드포인트): `index=sec_events sourcetype=sysmon EventCode=1`

## Lookups
| Name | Type | Key Fields | Output Fields | Purpose |
|------|------|-----------|---------------|---------|
| asset_db.csv | CSV | ip | hostname, owner, department, criticality | IP 기반 이벤트 자산 인리치먼트 |
| user_identity | KV Store | username | full_name, department, role, manager | 사용자 ID 상관분석 |
| geo_private_ranges.csv | CSV | cidr | location, zone | 내부 네트워크 존 분류 |
| threat_intel_ip.csv | CSV | ip | threat_category, confidence, source | 악성 IP 매칭 |

## Macros
| Name | Definition | Args | Purpose |
|------|-----------|------|---------|
| sec_index | `index=sec_logs` | 0 | 보안 인덱스 단축 |
| all_sec | `(index=sec_logs OR index=sec_events OR index=sec_audit)` | 0 | 전체 보안 인덱스 |
| get_severity(1) | `case($level$>7,"critical",$level$>4,"high",$level$>2,"medium",true(),"low")` | 1 | 심각도 분류 |
| private_ip | `(src_ip="10.*" OR src_ip="172.16.*" OR src_ip="192.168.*")` | 0 | 사설 IP 필터 |

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
| fw-hq-01 | 본사 1차 방화벽 | sec_logs | pan:traffic |
| fw-hq-02 | 본사 2차 방화벽 | sec_logs | pan:traffic |
| dc-hq-01 | 본사 도메인 컨트롤러 | sec_audit, winevent | WinEventLog:Security, WinEventLog:System |
| web-prod-lb | 프로덕션 로드밸런서 | web_prod | apache:access |

### Host Groups (Naming Patterns)
| Pattern | Role | Count | Location |
|---------|------|-------|----------|
| fw-* | 방화벽 | 12 | All sites |
| dc-* | 도메인 컨트롤러 | 4 | Internal (HQ, DR) |
| web-prod-* | 프로덕션 웹 서버 | 8 | DMZ |
| web-stg-* | 스테이징 웹 서버 | 3 | Internal |
| linux-* | Linux 서버 | 25 | Mixed |
```

---

## 대응 MEMORY.md 포인터

```markdown
## Splunk Environment
- [splunk-env.md](./splunk-env.md) — Security: sec_logs, sec_events, sec_audit | Web: web_prod, web_staging | Infra: infra_syslog, winevent, infra_metrics | topology: distributed (3 idx, 2 sh) | tiers: 1-2
```
