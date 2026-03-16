---
name: splunk-env-ko
description: >
  TRIGGER when: 사용자가 Splunk 환경 메타데이터 설정을 요청할 때.
  /splunk-env-ko, "내 인덱스", "우리 소스타입", "Splunk 환경 설정",
  "인덱스 등록", "소스타입 저장", conf 파일 임포트 언급 시 트리거.
  DO NOT TRIGGER when: SPL 문법, 검색 명령어, 쿼리 최적화,
  베스트 프랙티스 질문 (splunk-spl이 처리).
  DO NOT TRIGGER when: 영어로 입력 (use splunk-env for English).
  Splunk 환경 메타데이터를 Claude 메모리에 저장하여
  splunk-spl 스킬이 환경 맞춤 SPL을 생성하도록 지원.
license: MIT
metadata:
  author: splunk-skill
  version: "1.0.0"
---

# Splunk 환경 메타데이터 설정

이 스킬은 사용자의 Splunk 환경 메타데이터(인덱스, 소스타입, 룩업, 매크로 등)를
Claude 메모리에 저장합니다. 이후 **splunk-spl** 스킬이 SPL 쿼리를 생성할 때
저장된 환경 데이터를 자동으로 참조하여 **환경 맞춤 SPL**을 생성합니다.

## 동작 원리

**splunk-env 없이** (제네릭 SPL):
```spl
index=main sourcetype=firewall action=blocked
| stats count by src_ip, dest_ip
```

**splunk-env 있을 때** (환경 맞춤 SPL):
```spl
index=sec_logs sourcetype=pan:traffic action=blocked
| stats count by src_ip, dest_ip
```

차이점: Claude가 사용자의 실제 인덱스명, 소스타입, 필드명을 알고 있어
추측이나 플레이스홀더 없이 정확한 SPL을 생성합니다.

---

## 빠른 시작

환경 등록 방법 3가지 중 선택하세요:

| 방법 | 적합한 사용자 | 설명 |
|------|------------|------|
| **A. Conf 파일 임포트** | Splunk 서버 접근 가능한 관리자 | `$SPLUNK_HOME` 경로 지정 후 conf 파일 자동 파싱 |
| **B. SPL 디스커버리** | 검색 권한이 있는 사용자 | 디스커버리 SPL 쿼리 실행 후 결과 붙여넣기 |
| **C. 수동 입력** | 모든 사용자 | 인덱스/소스타입 목록 직접 입력 |

**예시 프롬프트:**
- *"/opt/splunk에 있는 conf 파일 스캔해서 환경 정보 추출해줘"*
- *"우리 인덱스: sec_logs, web_prod, infra_syslog. 저장해줘."*
- *"`| eventcount summarize=false index=*` 결과야: ..."*

---

## 방법 A: Conf 파일 임포트

Splunk 설정 파일을 파싱하여 환경 메타데이터를 자동 추출합니다.

### 절차

1. **사용자에게** `$SPLUNK_HOME` 경로를 요청합니다 (예: `/opt/splunk`).
2. **Conf 파일을 스캔**하여 안전한 추출 명령어로 메타데이터를 파싱합니다.
3. **추출된 데이터를 사용자에게 보여주고** 확인을 받습니다.
4. **확인된 데이터를** `memory/splunk-env.md`에 저장합니다.

### 스캔 범위

다음 디렉토리를 순서대로 스캔합니다:
- `$SPLUNK_HOME/etc/system/local/` — 시스템 레벨 설정
- `$SPLUNK_HOME/etc/apps/*/local/` — 앱별 커스터마이제이션
- `$SPLUNK_HOME/etc/apps/*/default/` — 앱 기본값

인덱스가 30개 이상 발견되면 접두사별로 그룹화하여 사용자가 관련 그룹을 선택하게 합니다.

### 파싱 규칙

자세한 파일별 추출 명령어는 **references/conf-parsing-guide.md**를 참조하세요.

### 보안 규칙

- **절대** `external_cmd`, `external_type`, `password`, `secret`, `token`, `key` 필드를 읽지 마세요
- **절대** `$SPLUNK_HOME` 경로를 `splunk-env.md`에 저장하지 마세요 (인프라 노출 위험)
- **오직** 메타데이터만 추출합니다: 이름, 정의, 타입 — 실행 가능한 코드나 민감 정보는 제외

---

## 방법 B: SPL 디스커버리

Splunk에서 디스커버리 쿼리를 실행하여 환경 메타데이터를 수집합니다.

### 절차

1. **카테고리별 디스커버리 쿼리를 제공**합니다.
2. **사용자가 Splunk에서 쿼리를 실행**하고 결과를 붙여넣습니다.
3. **결과를 파싱**합니다 — CSV, 탭 구분, 행별 형식 모두 수용합니다.
4. **추출된 데이터를 사용자에게 확인** 후 저장합니다.

### 디스커버리 쿼리

Tier 1/2/3 전체 쿼리는 **references/discovery-queries.md**를 참조하세요.

---

## 방법 C: 수동 입력

사용자가 제공하는 모든 형식의 환경 메타데이터를 수용합니다.

### 수용 형식

- 쉼표 구분: `sec_logs, web_prod, infra_syslog`
- 한 줄에 하나씩
- 자유 형식: *"sec_logs는 보안용이고 web_prod는 웹 트래픽이야"*
- Splunk에서 복사한 테이블/CSV

입력을 파싱하고 사용자 확인 후 저장합니다.

---

## 단계적 수집 (Progressive Disclosure)

메타데이터를 티어별로 수집합니다. Tier 1을 먼저 완료한 후 상위 티어를 안내합니다.

### Tier 1 (필수)

| 카테고리 | 수집 항목 |
|----------|----------|
| **인덱스** | 이름, 용도, 연결된 소스타입, 사용 시나리오 |
| **소스타입** | 이름, 연결된 인덱스, 카테고리, 벤더/제품 |
| **토폴로지** | 배포 유형 (standalone/distributed/cloud), 인덱서 수, 서치헤드 수 |

### Tier 2 (권장)

| 카테고리 | 수집 항목 |
|----------|----------|
| **룩업** | 파일명/이름, 타입 (CSV/KV Store), 키 필드, 출력 필드 |
| **매크로** | 이름, 정의, 인수 개수 |
| **데이터 모델** | 이름, 가속 상태, 보존 기간, 주요 데이터셋 |
| **호스트 & 호스트 그룹** | 주요 호스트 (이름, 역할, 인덱스, 소스타입), 명명 패턴 (패턴, 역할, 수, 위치) |

### Tier 3 (선택)

| 카테고리 | 수집 항목 |
|----------|----------|
| **이벤트타입 & 태그** | 이름, 검색 정의, 연결된 태그 |
| **주요 필드명** | 필드명, 설명, 사용되는 소스타입 |
| **설치된 TA** | 앱 이름, 버전, 제공 소스타입 |
| **KV Store 컬렉션** | 컬렉션명, 앱, 키 필드 |

### Tier 1 완료 후

안내: *"인덱스와 소스타입을 저장했습니다. 룩업, 매크로, 데이터 모델, 호스트 정보도 추가하시겠습니까?"*

---

## 저장 형식

모든 환경 메타데이터는 사용자의 프로젝트 메모리 디렉토리 내
`memory/splunk-env.md`에 저장됩니다.

### 규칙

- **데이터가 있는 섹션만 포함** — 빈 테이블을 만들지 않습니다.
- 카테고리를 확인했으나 데이터가 없으면: `## Lookups\nNone configured.`
- 인덱스 10개 이상이면 도메인별 그룹화 (### Security, ### Web 등)
- 각 인덱스 행에 **Use When** 컬럼을 포함하여 SPL 생성 가이드를 제공합니다.
- **SPL Generation Notes**로 일반적 시나리오를 인덱스+소스타입에 매핑합니다.
- 복수 인스턴스: `## Instance: Production` / `## Instance: Development` 헤더 사용

### 스키마

전체 출력 스키마 정의: **references/schema-reference.md**
완성된 예시: **references/sample-env.md**

### 인덱스 테이블 예시

```markdown
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

## SPL Generation Notes
- 보안 상관분석: `(index=sec_logs OR index=sec_events)`
- VPN 공격: `index=sec_logs sourcetype=pan:traffic`
- 웹 500 에러: `index=web_prod sourcetype=apache:access status>=500`
```

---

## MEMORY.md 업데이트

`splunk-env.md` 저장 후, `MEMORY.md`에 카테고리 요약이 포함된 포인터를 추가합니다:

```markdown
## Splunk Environment
- [splunk-env.md](./splunk-env.md) — Security: sec_logs, sec_events | Web: web_prod | Infra: infra_syslog, winevent | topology: distributed | tiers: 1-2
```

### 요약을 포함하는 이유

Claude는 `MEMORY.md`만 자동 로드합니다. 포인터에 핵심 값 요약이 있으면:
1. 간단한 SPL 생성 시 별도 파일 Read 없이 즉시 활용 가능
2. 복잡한 요청 시 Claude가 "더 자세한 환경 정보 필요" → splunk-env.md Read

### 규칙

- 인덱스 30개 이상: 카테고리명 + 대표 인덱스만 요약 (상세는 splunk-env.md에서 Read)
- 포인터가 이미 존재하면: **업데이트** — 중복 추가하지 않음
- 포함 항목: 주요 인덱스명, 토폴로지 유형, 설정된 티어

---

## 업데이트 & 확장

기존 환경 프로필의 수정도 이 스킬이 처리합니다.

### 작업 유형

| 사용자 요청 | 동작 |
|------------|------|
| *"매크로 추가해줘"* | `splunk-env.md`에 해당 섹션 추가 |
| *"인덱스 web_prod를 web_production으로 바꿔"* | 해당 항목만 수정 |
| *"conf 파일 다시 읽어"* | 재파싱 후 기존 데이터와 머지, 변경사항 표시 |
| *"현재 환경 보여줘"* | `splunk-env.md` 읽어서 표시 |

### 기존 파일 처리

`splunk-env.md`가 이미 존재할 때:
1. 기존 프로필을 읽어 사용자에게 보여줍니다.
2. 확인: *"기존 프로필을 업데이트하시겠습니까, 새로 시작하시겠습니까?"*
3. 업데이트 선택 시: 새 데이터 머지, 변경 없는 섹션 유지
4. 새로 시작 선택 시: 확인 후 전체 교체

---

## splunk-spl과의 연동

`splunk-env.md`가 존재하면 splunk-spl 스킬이 자동으로 환경 인식 SPL을 생성합니다:

**질문**: *"VPN 방화벽에서 무차별 대입 공격 찾아줘"*

환경 없이:
```spl
index=main sourcetype=vpn action=failure
| stats count by src_ip
| where count > 10
```

환경 있을 때:
```spl
index=sec_logs sourcetype=pan:traffic action=blocked app=ssl-vpn
| stats count by src_ip
| where count > 10
| lookup asset_db.csv ip AS src_ip OUTPUT hostname, owner
```

환경 데이터가 제공하는 것: 정확한 인덱스, 정확한 소스타입, 관련 룩업 인리치먼트,
실제 데이터에 맞는 필드명.

---

## 참조 파일

| 파일 | 내용 |
|------|------|
| `references/conf-parsing-guide.md` | conf 파일별 추출 명령어 + 보안 규칙 |
| `references/discovery-queries.md` | Tier 1/2/3 SPL 디스커버리 쿼리 |
| `references/schema-reference.md` | `splunk-env.md` 전체 출력 스키마 |
| `references/sample-env.md` | 완성된 예시 환경 파일 |

---

## 제한 사항

- **라이브 연결 아님**: 정적 메타데이터를 저장합니다 — Splunk에 실시간 쿼리하지 않습니다.
- **Conf 상속**: 복잡한 설정 상속 체인(system → app default → app local)에서 일부 오버라이드가 누락될 수 있습니다. 추출된 데이터는 항상 사용자에게 확인합니다.
- **프로젝트 범위**: 환경 데이터는 Claude 메모리에 프로젝트별로 저장됩니다. 다른 프로젝트에는 별도 설정이 필요합니다.
- **최신성**: Splunk 환경이 변경되면(새 인덱스, 폐기된 소스타입) 사용자가 프로필을 수동으로 업데이트하거나 디스커버리를 다시 실행해야 합니다.
