# Conf 파일 파싱 가이드

Splunk 설정 파일에서 환경 메타데이터를 안전하게 추출하는 명령어입니다.
사용자가 `$SPLUNK_HOME` 디렉토리 접근을 제공할 때 사용합니다.

## 스캔 순서

1. `$SPLUNK_HOME/etc/system/local/` — 시스템 레벨 설정
2. `$SPLUNK_HOME/etc/apps/*/local/` — 앱별 커스터마이제이션 (높은 우선순위)
3. `$SPLUNK_HOME/etc/apps/*/default/` — 앱 기본값 (낮은 우선순위)

동일 stanza가 여러 위치에 있으면 `local/`이 `default/`를 오버라이드합니다.

---

## indexes.conf

**추출 대상**: 인덱스 이름 (stanza 이름)

```bash
grep '^\[' indexes.conf | sed 's/[][]//g' | grep -v '^default$' | grep -v '^volume:' | sort -u
```

**제외**: `[default]`, `[volume:*]` stanza — 내부용입니다.

**추가 데이터** (stanza 내 존재 시):
- `homePath` → 인덱스 위치 유추 (경로 자체는 저장하지 않음)
- `frozenTimePeriodInSecs` → 보존 기간 유추

---

## props.conf

**추출 대상**: 소스타입 이름 (stanza 이름)

```bash
grep '^\[' props.conf | sed 's/[][]//g' | grep -v '^source::' | grep -v '^host::' | grep -v '^default$' | sort -u
```

**제외**: `[source::*]`, `[host::*]`, `[default]` stanza — 소스/호스트별
오버라이드이며 소스타입 정의가 아닙니다.

**추가 데이터**:
- `TRANSFORMS-*` 라인 → transforms.conf 룩업 연결
- `TIME_FORMAT` / `TIME_PREFIX` → 시간 파싱 설정 참고

---

## transforms.conf

**추출 대상**: 룩업 정의 (`filename =`이 있는 stanza만)

```bash
grep -B1 'filename\s*=' transforms.conf | grep '^\[' | sed 's/[][]//g' | sort -u
```

**룩업 파일명 추출**:
```bash
grep 'filename\s*=' transforms.conf | sed 's/.*=\s*//' | sort -u
```

### 보안: 매우 중요

**transforms.conf에서 절대 읽거나 추출하지 말아야 할 필드:**
- `external_cmd`
- `external_type`
- `password`, `secret`, `token`, `key`를 포함하는 모든 필드

오직 추출: stanza 이름과 `filename` 값만.

---

## macros.conf

**추출 대상**: 매크로 이름과 정의

```bash
# Stanza 이름 (매크로 이름)
grep '^\[' macros.conf | sed 's/[][]//g' | sort -u

# 정의
grep -A1 '^\[' macros.conf | grep 'definition\s*='
```

**인수 개수 추출**: `args =` 필드 확인, 또는 정의 내 `$arg$` 플레이스홀더 개수 카운트.

---

## eventtypes.conf

**추출 대상**: 이벤트타입 이름과 검색 정의

```bash
# Stanza 이름
grep '^\[' eventtypes.conf | sed 's/[][]//g' | sort -u

# 검색 정의
grep 'search\s*=' eventtypes.conf
```

---

## tags.conf

**추출 대상**: 태그 할당

```bash
grep '^\[' tags.conf | sed 's/[][]//g' | sort -u
```

Stanza 형식은 `[eventtype=name]` — stanza 헤더에서 이벤트타입 이름을 추출합니다.

---

## datamodels.conf

**추출 대상**: 데이터 모델 이름과 가속 상태

```bash
# Stanza 이름
grep '^\[' datamodels.conf | sed 's/[][]//g' | sort -u

# 가속 상태
grep 'acceleration\s*=' datamodels.conf
```

---

## inputs.conf

**추출 대상**: 모니터 stanza에서 호스트 정보

```bash
# 호스트 오버라이드가 있는 모니터 stanza
grep -A5 '^\[monitor://' inputs.conf | grep -E '^\[monitor://|^host\s*='
```

주요 호스트와 관련 로그 소스를 파악하는 데 사용합니다.

---

## 대규모 환경 처리

인덱스가 30개 이상 발견되면:

1. **접두사별 그룹화**: 공통 접두사 추출 (예: `sec_*`, `web_*`, `app_*`)
2. **그룹을 사용자에게 제시**: *"45개 인덱스를 6개 그룹으로 분류했습니다: sec (12), web (8), app (15), infra (5), test (3), misc (2). 어떤 그룹이 필요하세요?"*
3. **선택된 그룹만** `splunk-env.md`에 저장

---

## 일반 보안 규칙

1. **절대** 파일 경로(`$SPLUNK_HOME`, `/opt/splunk/...`)를 splunk-env.md에 저장하지 마세요
2. **절대** 인증 정보, 토큰, 비밀번호, 키를 읽거나 저장하지 마세요
3. **절대** conf 파일 내용을 실행하지 마세요 — 텍스트만 파싱
4. **오직** 메타데이터만 추출: 이름, 정의, 타입
5. **항상** 추출된 데이터를 저장 전에 사용자에게 확인
