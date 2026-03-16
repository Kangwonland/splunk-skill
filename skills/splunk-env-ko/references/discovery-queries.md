# SPL 디스커버리 쿼리

Splunk에서 환경 메타데이터를 수집하기 위한 쿼리입니다. 쿼리를 Splunk에서
실행하고 결과를 붙여넣으세요 — CSV, 탭 구분, 한 줄씩 어떤 형식이든 수용합니다.

---

## Tier 1 — 필수

### 인덱스 (활성 데이터가 있는 것만)

```spl
| eventcount summarize=false index=* | where count > 0 | dedup index | sort -count | table index count
```

반환: 이벤트가 1개 이상 있는 모든 인덱스 (이벤트 수 기준 정렬)

### 소스타입

```spl
| metadata type=sourcetypes index=* | table sourcetype totalCount firstTime recentTime
```

반환: 모든 소스타입, 이벤트 수, 시간 범위

### 토폴로지

```spl
| rest /services/server/info | table server_roles version numberOfVirtualCores physicalMemoryMB
```

반환: 서버 역할 (indexer, search head 등), 버전, 리소스 정보

### 인덱스-소스타입 매핑

```spl
| tstats count where index=* by index sourcetype | sort -count | table index sourcetype count
```

반환: 어떤 소스타입이 어떤 인덱스에 있는지 (정확한 매핑에 필수)

---

## Tier 2 — 권장

### 룩업

```spl
| rest /services/data/transforms/lookups | table title filename type fields_list
```

반환: 모든 룩업 정의 (타입: CSV, KV Store, external)

### 매크로

```spl
| rest /services/admin/macros | table title definition args
```

반환: 모든 검색 매크로, 정의, 인수 개수

### 데이터 모델

```spl
| rest /services/datamodel/model | table title acceleration acceleration.earliest_time eai:digest
```

반환: 데이터 모델, 가속 상태, 보존 기간

### 호스트 (메타데이터)

```spl
| metadata type=hosts index=* | table host totalCount firstTime recentTime | sort -totalCount
```

반환: 데이터를 보고하는 모든 호스트 (이벤트 수 기준 정렬)

### 호스트-인덱스-소스타입 매핑 (상위)

```spl
| tstats count where index=* by host index sourcetype | sort -count | head 100 | table host index sourcetype count
```

반환: 상위 100개 호스트-인덱스-소스타입 조합

---

## Tier 3 — 선택

### 이벤트타입

```spl
| rest /services/saved/eventtypes | table title search tags
```

반환: 정의된 이벤트타입, 검색 정의, 태그

### 설치된 TA 및 앱

```spl
| rest /services/apps/local | search title=*TA* OR title=*CIM* OR title=Splunk_SA* | table title version disabled
```

반환: Technology Add-on, CIM, Security Essentials 앱

### 모든 앱 (넓은 범위)

```spl
| rest /services/apps/local | search disabled=0 | table title version
```

반환: 활성화된 모든 앱

### KV Store 컬렉션

```spl
| rest /services/kvstore/collectionconfig | table title app
```

반환: KV Store 컬렉션 이름과 소속 앱

### 저장된 검색 (참고용)

```spl
| rest /services/saved/searches | search disabled=0 is_scheduled=1 | table title cron_schedule search | head 20
```

반환: 상위 20개 활성 예약 검색 (워크로드 파악에 유용)

---

## 사용자 안내

- **관리자 권한 없음?** Tier 1 쿼리는 일반 검색 권한으로 실행 가능합니다.
- **Tier 2-3** REST 쿼리는 admin 또는 `list_settings` 기능(capability)이 필요할 수 있습니다.
- **대규모 환경**: 결과가 너무 많으면 `| head 50`을 추가하여 제한하세요.
- **붙여넣기 형식**: 어떤 형식이든 가능합니다 — Splunk 출력을 그대로 붙여넣으세요.
