# Splunk 환경 설정
#
# 아래에 Splunk 환경 정보를 입력하세요.
# 필요한 섹션만 채우세요 — 불필요한 섹션은 삭제하거나 비워두세요.
# splunk-spl 스킬이 이 파일을 읽어 환경 맞춤 SPL을 생성합니다.

## Metadata

- **Deployment**: distributed
- **Indexers**: 3
- **Search Heads**: 2
- **Splunk Version**: 9.4.0

## Indexes

### Security
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| sec_logs | 방화벽/VPN 트래픽 | pan:traffic, cisco:asa | 네트워크 공격, 정책 위반 |
| sec_events | 엔드포인트 보안 | crowdstrike:events | 악성코드, 프로세스 이상 |

### Web
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| web_prod | 프로덕션 웹 서버 | apache:access | 웹 에러, 성능, 접근 분석 |

### Infrastructure
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| infra_syslog | Linux/네트워크 syslog | syslog | 시스템 상태, 인증 실패 |
| winevent | Windows 이벤트 로그 | WinEventLog:Security, WinEventLog:System | Windows 인증, 서비스 상태 |

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

- 보안 상관분석: `(index=sec_logs OR index=sec_events)`
- VPN/방화벽 공격: `index=sec_logs sourcetype=pan:traffic`
- 웹 500 에러: `index=web_prod sourcetype=apache:access status>=500`
- Windows 로그온 실패: `index=winevent sourcetype=WinEventLog:Security EventCode=4625`

## Hosts & Host Groups

### Key Hosts
| Host | Role | Index | Sourcetypes |
|------|------|-------|-------------|
| fw-hq-01 | 본사 1차 방화벽 | sec_logs | pan:traffic |
| dc-hq-01 | 본사 도메인 컨트롤러 | winevent | WinEventLog:Security |

### Host Groups (Naming Patterns)
| Pattern | Role | Count | Location |
|---------|------|-------|----------|
| fw-* | 방화벽 | 12 | All sites |
| web-prod-* | 프로덕션 웹 서버 | 8 | DMZ |
| dc-* | 도메인 컨트롤러 | 4 | Internal |

## Lookups

| Name | Type | Key Fields | Output Fields | Purpose |
|------|------|-----------|---------------|---------|
| asset_db.csv | CSV | ip | hostname, owner, department | 자산 인리치먼트 |
| user_identity | KV Store | username | department, role, manager | 사용자 컨텍스트 |

## Macros

| Name | Definition | Args | Purpose |
|------|-----------|------|---------|
| sec_index | `index=sec_logs` | 0 | 보안 인덱스 단축 |
| get_severity(1) | `case($level$>7,"critical",$level$>4,"high")` | 1 | 심각도 분류 |

## Data Models

| Name | Accelerated | Retention | Key Datasets |
|------|------------|-----------|-------------|
| Network_Traffic | Yes | 7 days | All_Traffic, Blocked_Traffic |
| Authentication | Yes | 30 days | Failed_Authentication |
