# splunk-skill — Claude Code 플러그인

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Splunk](https://img.shields.io/badge/Splunk_Enterprise-9.4-green.svg)
![Rules](https://img.shields.io/badge/SPL_Rules-86-blue.svg)
![Benchmark](https://img.shields.io/badge/Benchmark-95.83%25-brightgreen.svg)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)
![Security Scan](https://img.shields.io/badge/Skill_Scanner-Passed-brightgreen.svg)

Claude Code용 Splunk 스킬 모음 — 공식 **SPL/SPL2 베스트 프랙티스**와
사용자 환경 메타데이터를 활용한 **환경 맞춤 SPL 생성**.

> 🇺🇸 **English**: [README.md](README.md)

---

## 하나의 플러그인, 두 개의 스킬

| 스킬 | 역할 | 트리거 |
|------|------|--------|
| **`splunk-spl`** | SPL/SPL2 베스트 프랙티스, 최적화, 마이그레이션 | SPL·Splunk 검색 질문 전반 |
| **`splunk-env`** | 인덱스·소스타입·호스트 등록 → splunk-spl이 참조 | 영문 환경 설정 요청 |
| **`splunk-env-ko`** | 한글 환경 설정 (스키마 동일) | 한국어 환경 설정 요청 |

**연동 방식**: `splunk-env`가 `env-config.md`에 환경 정보를 저장하면,
`splunk-spl`이 SPL 작성 시 이 파일을 자동으로 읽어 **사용자 환경에 맞는 `index=`,
`sourcetype=`, `host=` 값으로 쿼리를 생성**합니다 — 제네릭 `index=main` 대신.

---

## 1. splunk-spl — SPL/SPL2 베스트 프랙티스

Splunk Enterprise 9.4 기준 공식 SPL/SPL2 패턴, 성능 최적화 규칙, 레퍼런스 자료를
제공합니다. 사용자가 Splunk 검색을 작성/리뷰/최적화할 때 Claude가 자동으로 이
가이드를 적용합니다.

### 포함 범위

- **86개 베스트 프랙티스 규칙** (25개 카테고리)
- **21개 레퍼런스 테이블** (명령어 인덱스, 함수 카탈로그, limits.conf 값)
- SPL → SPL2 마이그레이션 가이드
- Splunk CLI 및 REST API 패턴
- 모든 규칙은 `help.splunk.com` (Splunk Enterprise 9.4) 기반

### 카테고리

| 우선순위 | 카테고리 | 규칙 수 |
|----------|----------|---------|
| CRITICAL | Core Basics, Time Modifiers, Performance | 18 |
| HIGH | Extraction, Eval, Stats, Subsearch, Join, Lookup, Macros, SPL2, Migration | 42 |
| MEDIUM | Transaction, Iteration, Data Models, tstats, Summary, Output, CLI, REST | 19 |
| LOW-MEDIUM | Geo, Anomaly, Prediction, Metrics, Set Operations | 7 |

### 벤치마크

`splunk-spl` 스킬 적용 시: **95.83%** vs 미적용: 83.33% (+12.50pp, 36개 평가)

---

## 2. splunk-env — 환경 메타데이터

`env-config.md`를 관리합니다 — 사용자가 자신의 Splunk 환경(인덱스, 소스타입,
호스트, 룩업, 매크로)을 등록하는 미리 채워진 마크다운 파일입니다.
`splunk-spl`이 이 파일을 자동으로 읽어 **환경 맞춤 SPL**을 생성합니다.

### env-config.md 없을 때 (제네릭)

```spl
index=main sourcetype=firewall action=blocked
| stats count by src_ip
```

### env-config.md 있을 때 (환경 맞춤)

```spl
index=sec_logs sourcetype=pan:traffic action=blocked
| stats count by src_ip
| lookup asset_db.csv ip AS src_ip OUTPUT hostname, owner
```

### 입력 항목 (Tier 기반 스키마)

| Tier | 섹션 | 필수 여부 | 내용 |
|------|------|-----------|------|
| **1** | `Metadata` | ✅ 필수 | 배포 유형, 인덱서·SH 개수, Splunk 버전 |
| **1** | `Indexes` | ✅ 필수 | 이름, 용도, 소스타입, **Use When** (SPL 생성의 핵심) |
| **1** | `Sourcetypes` | ✅ 필수 | 소스타입, 인덱스, 카테고리, 벤더·제품 |
| **1** | `SPL Generation Notes` | ✅ 필수 | 시나리오 → 정확한 `index=... sourcetype=...` 매핑 |
| **2** | `Hosts & Host Groups` | 권장 | 핵심 호스트 + 명명 패턴 (예: `fw-*`) |
| **2** | `Lookups` | 선택 | 이름, 타입(CSV·KV Store), 키·출력 필드 |
| **2** | `Macros` | 선택 | 이름, 정의, 인수 |
| **2** | `Data Models` | 선택 | 이름, 가속 여부, 보관 기간, 핵심 데이터셋 |
| **3** | `Eventtypes & Tags` | 선택 | 이벤트타입, 검색식, 태그 |
| **3** | `Key Field Names` | 선택 | 필드명, 설명, 소스타입 |
| **3** | `Installed TAs` | 선택 | 앱, 버전, 제공 소스타입 |
| **3** | `KV Store Collections` | 선택 | 컬렉션, 앱, 키 필드 |

전체 스키마: [`skills/splunk-env-ko/references/schema-reference.md`](skills/splunk-env-ko/references/schema-reference.md)

### 편집 방법

- **직접 편집**: `skills/splunk-env-ko/env-config.md` 파일을 열어 환경 정보 입력
- **대화로 요청**: 평문으로 환경을 알려주면 스킬이 대신 편집
  - 예: *"우리 인덱스는 sec_logs(방화벽), web_prod(아파치)야. 방화벽 호스트는 fw-*."*
- **자동 임포트**: `.conf` 파일 스캔 또는 디스커버리 쿼리 실행
  ([`references/conf-parsing-guide.md`](skills/splunk-env-ko/references/conf-parsing-guide.md),
  [`references/discovery-queries.md`](skills/splunk-env-ko/references/discovery-queries.md) 참조)

---

## 설치

### 방법 A — 온라인 설치 (권장)

```bash
# splunk-spl (SPL 베스트 프랙티스)
claude plugin install https://github.com/Kangwonland/splunk-skill@v1.0.0-splunk9.4

# splunk-env (환경 메타데이터)
claude plugin install https://github.com/Kangwonland/splunk-skill@env-v1.0.2
```

### 방법 B — Airgap / 수동 설치

GitHub 접근이 불가능한 폐쇄망 환경에서 사용:

1. **`.skill` 번들 다운로드** (인터넷 가능한 PC에서):
   - splunk-spl: [`splunk-spl.skill`](https://github.com/Kangwonland/splunk-skill/releases/tag/v1.0.0-splunk9.4)
   - splunk-env (영문): [`splunk-env.skill`](https://github.com/Kangwonland/splunk-skill/releases/tag/env-v1.0.2)
   - splunk-env (한글): `splunk-env-ko.skill` (동일 릴리즈)

2. **파일 전송**: `.skill` 파일을 폐쇄망 호스트로 옮깁니다.

3. **압축 해제** (`.skill`은 ZIP 아카이브입니다):

   ```bash
   mkdir -p ~/.claude/skills
   unzip splunk-spl.skill -d ~/.claude/skills/
   unzip splunk-env.skill -d ~/.claude/skills/
   # 한글판:
   unzip splunk-env-ko.skill -d ~/.claude/skills/
   ```

   결과 디렉토리 구조:
   ```
   ~/.claude/skills/
   ├── splunk-spl/
   │   ├── SKILL.md
   │   └── references/
   ├── splunk-env/
   │   ├── SKILL.md
   │   ├── env-config.md       ← 이 파일 편집
   │   └── references/
   └── splunk-env-ko/
       └── ...
   ```

4. **Claude Code 재시작**. `~/.claude/skills/`에서 스킬이 자동 로딩됩니다.

---

## 사용법

질문 내용에 따라 Claude가 자동으로 스킬을 선택합니다:

| 질문 예시 | 트리거되는 스킬 |
|-----------|----------------|
| "로그인 실패 찾는 SPL 써줘" | `splunk-spl` |
| "이 tstats 쿼리 최적화해줘" | `splunk-spl` |
| "Register my indexes: sec_logs, web_prod" | `splunk-env` |
| "내 방화벽 호스트는 fw-*야" | `splunk-env-ko` |

슬래시 커맨드로 직접 호출도 가능:

```
/splunk-spl      # SPL 생성
/splunk-env      # 환경 설정 (영문)
/splunk-env-ko   # 환경 설정 (한글)
```

---

## 버전 관리

각 스킬이 독립적인 릴리즈 태그를 갖습니다:

- **splunk-spl**: `v{skill-version}-splunk{splunk-version}` — 예: `v1.0.0-splunk9.4`
- **splunk-env**: `env-v{version}` — 예: `env-v1.0.2`

핫픽스용 브랜치 (예: `v9.4`)도 별도 관리됩니다.

---

## 레퍼런스

- Splunk 공식 문서: `https://help.splunk.com/en/splunk-enterprise/search` (v9.4)
- Claude Code 스킬 가이드: `https://docs.claude.com/en/docs/claude-code/skills`

> 참고: `docs.splunk.com`은 deprecated 상태입니다. 모든 SPL 규칙은 `help.splunk.com`을 참조합니다.

## 라이선스

MIT
