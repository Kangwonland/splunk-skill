# splunk-env.md 스키마 레퍼런스

이 문서는 `memory/splunk-env.md`의 출력 형식을 정의합니다 — Splunk 환경
메타데이터가 저장되는 파일의 스키마입니다.

---

## 파일 구조

```markdown
# Splunk Environment Profile

## Metadata
- **Deployment**: [standalone | distributed | Splunk Cloud]
- **Indexers**: [개수]
- **Search Heads**: [개수]
- **Splunk Version**: [버전]
- **Last Updated**: [YYYY-MM-DD]
- **Tiers Configured**: [1 | 1-2 | 1-3]

## Indexes
[도메인별로 그룹화된 인덱스 테이블]

## Sourcetypes
[소스타입 테이블]

## SPL Generation Notes
[시나리오별 인덱스+소스타입 매핑]

## Lookups
[룩업 테이블 — Tier 2]

## Macros
[매크로 테이블 — Tier 2]

## Data Models
[데이터 모델 테이블 — Tier 2]

## Hosts & Host Groups
[호스트 및 명명 패턴 — Tier 2]

## Eventtypes & Tags
[이벤트타입 테이블 — Tier 3]

## Key Field Names
[필드명 테이블 — Tier 3]

## Installed TAs
[TA 테이블 — Tier 3]

## KV Store Collections
[KV Store 테이블 — Tier 3]
```

---

## 섹션별 스키마

### Metadata (필수)

항상 존재합니다. 배포 개요를 포함합니다.

```markdown
## Metadata
- **Deployment**: distributed
- **Indexers**: 3
- **Search Heads**: 2
- **Splunk Version**: 9.4.0
- **Last Updated**: 2026-03-17
- **Tiers Configured**: 1-2
```

### Indexes (필수 — Tier 1)

인덱스 10개 이상이면 도메인별로 그룹화합니다. 각 행에 **Use When** 컬럼을
포함하여 SPL 생성을 안내합니다.

```markdown
## Indexes

### Security
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| sec_logs | 방화벽/VPN 트래픽 | pan:traffic, cisco:asa | 네트워크 공격, 정책 위반, VPN 이슈 |
| sec_events | 엔드포인트 보안 | crowdstrike:events | 악성코드, 프로세스 이상, 엔드포인트 침해 |

### Web
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| web_prod | 프로덕션 웹 서버 | apache:access, apache:error | 웹 에러, 성능, 접근 패턴 |

### Infrastructure
| Index | Purpose | Sourcetypes | Use When |
|-------|---------|-------------|----------|
| infra_syslog | Linux/네트워크 syslog | syslog | 시스템 상태, 인증 실패, 장치 상태 |
| winevent | Windows 이벤트 로그 | WinEventLog:Security, WinEventLog:System | Windows 인증, 서비스 상태, GPO |
```

**인덱스 10개 미만**: 소제목 없이 단일 테이블 사용.

### Sourcetypes (필수 — Tier 1)

```markdown
## Sourcetypes
| Sourcetype | Indexes | Category | Vendor/Product |
|------------|---------|----------|---------------|
| pan:traffic | sec_logs | Network Security | Palo Alto Networks Firewall |
| cisco:asa | sec_logs | Network Security | Cisco ASA |
| apache:access | web_prod | Web | Apache HTTP Server |
| syslog | infra_syslog | Infrastructure | Generic Syslog |
```

### SPL Generation Notes (필수 — Tier 1)

일반적 사용 시나리오를 올바른 인덱스+소스타입 조합에 매핑합니다.

```markdown
## SPL Generation Notes
- 보안 상관분석 (전체): `(index=sec_logs OR index=sec_events)`
- VPN/방화벽 공격: `index=sec_logs sourcetype=pan:traffic`
- 엔드포인트 위협: `index=sec_events sourcetype=crowdstrike:events`
- 웹 500 에러: `index=web_prod sourcetype=apache:access status>=500`
- Windows 로그온 실패: `index=winevent sourcetype=WinEventLog:Security EventCode=4625`
- IP 크로스소스 조사: `(index=sec_logs OR index=web_prod OR index=infra_syslog) src_ip=<ip>`
```

### Lookups (Tier 2)

```markdown
## Lookups
| Name | Type | Key Fields | Output Fields | Purpose |
|------|------|-----------|---------------|---------|
| asset_db.csv | CSV | ip | hostname, owner, department | 자산 인리치먼트 |
| user_identity | KV Store | username | department, role, manager | 사용자 컨텍스트 |
```

### Macros (Tier 2)

```markdown
## Macros
| Name | Definition | Args | Purpose |
|------|-----------|------|---------|
| sec_index | `index=sec_logs` | 0 | 보안 인덱스 단축 |
| get_severity(1) | `case($level$>7,"critical",$level$>4,"high",$level$>2,"medium",true(),"low")` | 1 | 심각도 분류 |
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
| fw-hq-01 | 본사 1차 방화벽 | sec_logs | pan:traffic |
| dc-hq-01 | 본사 도메인 컨트롤러 | winevent | WinEventLog:Security |

### Host Groups (Naming Patterns)
| Pattern | Role | Count | Location |
|---------|------|-------|----------|
| fw-* | 방화벽 | 12 | All sites |
| web-prod-* | 프로덕션 웹 서버 | 8 | DMZ |
| dc-* | 도메인 컨트롤러 | 4 | Internal |
```

- **Key Hosts**: 핵심 개별 장비 (방화벽, DC 등 인프라 핵심 자산)
- **Host Groups**: 네이밍 패턴 기반 그룹 (와일드카드 SPL 생성용, 예: `host=fw-*`)
- **conf 소스**: `inputs.conf`의 `[monitor://]` stanza에서 호스트 정보 추출 가능
- **SPL 디스커버리**: `| metadata type=hosts index=* | table host | sort host`

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
| src_ip | 소스 IP 주소 | pan:traffic, cisco:asa, syslog |
| dest_ip | 목적지 IP 주소 | pan:traffic, cisco:asa |
| action | 방화벽 동작 (allowed/blocked) | pan:traffic |
| EventCode | Windows 이벤트 ID | WinEventLog:Security, WinEventLog:System |
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
| user_identity | SA-IdentityManagement | username | 사용자 ID 상관분석 |
| asset_lookup | SA-NetworkProtection | ip, mac | 자산 인벤토리 |
```

---

## 컬럼 내 복수 값 처리

| 컬럼 | 형식 | 예시 |
|------|------|------|
| Sourcetypes (인덱스 테이블) | 쉼표 구분 | `pan:traffic, cisco:asa` |
| Indexes (소스타입 테이블) | 쉼표 구분 | `infra_syslog, sec_logs` |
| Key Fields / Output Fields | 쉼표 구분 | `ip, hostname` |
| Macro Args | 개수만 | `1` |
| Data Model Datasets | 쉼표 구분 | `All_Traffic, Blocked_Traffic` |
| Eventtype Tags | 쉼표 구분 | `network, firewall, attack` |
| TA Sourcetypes | 쉼표 구분 | `pan:traffic, pan:system, pan:config` |

---

## 빈 카테고리 처리

카테고리를 확인/조회했으나 데이터가 없는 경우:

```markdown
## KV Store Collections
None configured.
```

카테고리를 확인하지 않았으면 해당 섹션을 아예 생략합니다.

---

## 복수 인스턴스

별도의 Splunk 배포를 가진 조직의 경우:

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

각 인스턴스는 자체적인 전체 섹션 세트를 가집니다.

---

## MEMORY.md 포인터 형식

MEMORY.md의 포인터에는 핵심 값 요약을 포함합니다:

```markdown
## Splunk Environment
- [splunk-env.md](./splunk-env.md) — Security: sec_logs, sec_events | Web: web_prod | Infra: infra_syslog, winevent | topology: distributed | tiers: 1-2
```

**인덱스 30개 이상 시**: 카테고리명 + 대표 인덱스만:
```markdown
- [splunk-env.md](./splunk-env.md) — Security (12 indexes): sec_logs, sec_events, ... | Web (8): web_prod, web_staging, ... | topology: cloud | tiers: 1-3
```
