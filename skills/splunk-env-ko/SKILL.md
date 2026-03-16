---
name: splunk-env-ko
description: >
  TRIGGER when: 사용자가 Splunk 환경 메타데이터 설정을 요청할 때.
  /splunk-env-ko, "내 인덱스", "우리 소스타입", "Splunk 환경 설정",
  "인덱스 등록", "소스타입 저장", env-config.md 편집 언급 시 트리거.
  DO NOT TRIGGER when: SPL 문법, 검색 명령어, 쿼리 최적화,
  베스트 프랙티스 질문 (splunk-spl이 처리).
  DO NOT TRIGGER when: 영어로 입력 (use splunk-env for English).
  env-config.md에 저장된 Splunk 환경 메타데이터를 관리하여
  splunk-spl 스킬이 환경 맞춤 SPL을 생성하도록 지원.
license: MIT
metadata:
  author: splunk-skill
  version: "1.0.0"
---

# Splunk 환경 메타데이터 설정

이 스킬은 **`env-config.md`** 파일을 관리합니다 — 사용자가 자신의 Splunk 환경
(인덱스, 소스타입, 호스트, 룩업, 매크로)을 미리 등록하는 설정 파일입니다.
**splunk-spl** 스킬이 이 파일을 읽어 **환경 맞춤 SPL**을 생성합니다.

## 동작 원리

1. 사용자가 이 스킬 디렉토리의 **`env-config.md`**에 환경 정보를 입력
2. SPL 생성 시 splunk-spl 스킬이 env-config.md를 자동으로 참조
3. 정확한 인덱스명, 소스타입, 필드명으로 SPL 생성

**env-config.md 없이** (제네릭):
```spl
index=main sourcetype=firewall action=blocked
| stats count by src_ip
```

**env-config.md 있을 때** (환경 맞춤):
```spl
index=sec_logs sourcetype=pan:traffic action=blocked
| stats count by src_ip
| lookup asset_db.csv ip AS src_ip OUTPUT hostname, owner
```

---

## env-config.md

설정 파일은 이 스킬 디렉토리의 **`env-config.md`**에 있습니다.
예시 데이터가 미리 채워져 있으며, 사용자가 자신의 환경으로 교체합니다.

### 이 스킬이 트리거되면:

1. **env-config.md를 읽어** 현재 설정을 파악합니다.
2. **사용자 요청에 따라 편집**: 항목 추가, 수정, 삭제.
3. **수정된 파일을 저장**합니다.

### 파일 구조

마크다운 테이블로 카테고리별 구성:

| 섹션 | 필수 | 내용 |
|------|------|------|
| **Metadata** | 예 | 배포 유형, 인덱서/SH 수, 버전 |
| **Indexes** | 예 | 이름, 용도, 소스타입, 사용 시나리오 |
| **Sourcetypes** | 예 | 이름, 인덱스, 카테고리, 벤더/제품 |
| **SPL Generation Notes** | 예 | 시나리오 → 인덱스+소스타입 매핑 |
| **Hosts & Host Groups** | 권장 | 주요 호스트, 명명 패턴 |
| **Lookups** | 선택 | 이름, 타입, 키/출력 필드 |
| **Macros** | 선택 | 이름, 정의, 인수 |
| **Data Models** | 선택 | 이름, 가속, 데이터셋 |

### 편집 규칙

- **데이터가 있는 섹션만 포함** — 불필요한 섹션은 삭제.
- 인덱스 10개 이상이면 도메인별 그룹화 (### Security, ### Web 등).
- 각 인덱스에 **Use When** 컬럼 필수 — SPL 생성의 핵심 가이드.
- **SPL Generation Notes**가 가장 중요 — 시나리오별 정확한 인덱스+소스타입 매핑.
- 마크다운 테이블 형식 유지.

---

## 사용자 상호작용

### 환경 정보 등록

사용자가 아무 형식으로 환경 정보를 알려주면:

> "우리 인덱스: sec_logs (방화벽), web_prod (아파치), infra (syslog).
> 방화벽 호스트는 fw-*. 인덱서 3대, distributed."

현재 env-config.md를 읽고, 새 정보를 반영하고, 결과를 보여준 뒤 저장.

### 기존 항목 수정

> "web_prod를 web_production으로 바꿔" 또는 "매크로 all_sec 추가해줘"

env-config.md를 읽고, 변경 적용, 저장.

### 현재 설정 확인

> "내 Splunk 환경 보여줘" 또는 "env 설정에 뭐가 있어?"

env-config.md를 읽어서 표시.

### Conf 파일에서 임포트 (관리자 전용)

> "/opt/splunk conf 파일 스캔해줘"

conf 파일 파싱 (자세한 내용: **references/conf-parsing-guide.md**) 후
env-config.md에 반영. 보안: 비밀번호/시크릿 절대 읽지 않음, 경로 저장 안 함.

### SPL 디스커버리 결과 임포트

> "디스커버리 쿼리 결과야: ..."

결과 파싱 (자세한 내용: **references/discovery-queries.md**) 후 env-config.md에 반영.

---

## splunk-spl과의 연동

splunk-spl 스킬은 SPL 생성 시 `env-config.md`를 읽습니다.
사용하는 주요 섹션:

- **Indexes** → 정확한 `index=` 값
- **Sourcetypes** → 정확한 `sourcetype=` 값
- **SPL Generation Notes** → 시나리오별 쿼리 매핑
- **Hosts & Host Groups** → `host=` 패턴
- **Lookups** → `| lookup` 인리치먼트
- **Macros** → `` `macro_name` `` 사용

---

## 참조 파일

| 파일 | 내용 |
|------|------|
| `env-config.md` | **설정 파일** — 사용자가 직접 편집 |
| `references/conf-parsing-guide.md` | conf 파일 추출 명령어 (관리자 전용) |
| `references/discovery-queries.md` | SPL 디스커버리 쿼리 |
| `references/schema-reference.md` | env-config.md 전체 스키마 |
| `references/sample-env.md` | 완성된 예시 |

---

## 제한 사항

- **라이브 연결 아님**: 정적 메타데이터, 실시간 Splunk 쿼리가 아닙니다.
- **수동 업데이트**: 환경 변경 시 사용자가 env-config.md를 직접 업데이트해야 합니다.
- **설치별**: 각 플러그인 설치에 자체 env-config.md가 있습니다.
