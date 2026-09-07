# RouteMate — spec.md (기능 명세서)

> **문서 상태**: [Gate 2 승인 대기]  
> **문서 버전**: 1.3.1  
> **최종 수정일**: 2026-09-07  
> **승인 이력**: (v1.3.0 검토 피드백 반영, v1.3.1 Gate 2 검토 중)

---

## 1. 전체 화면 구조도 (Sitemap & Navigation)

```text
[홈 화면 (HomeScreen)]
   ├── 1. 상단: 빠른 길찾기 바 (출발지 ⇄ 목적지 인풋 & 맞바꾸기 버튼)
   │        ├── 출발지 선택 모달 (현재위치(동적) / 장소검색 / 저장장소)
   │        └── 목적지 선택 모달 (장소검색 / 저장장소 / 최근검색)
   │
   ├── 2. 중앙: 저장된 경로 카드 리스트 (SavedRouteCard)
   │        ├── [바로 검색] 버튼 → 즉시 결과 화면 이동 (1-Tap)
   │        └── 카드 본체 터치 → 조건 임시 변경 바텀시트
   │
   ├── 3. 주변 대중교통 간이 바: [출발지 주변] / [목적지 주변] 토글
   │        └── ODsay pointSearch 연동 (상위 10개 + [더보기])
   │
   └── 4. 하단 탭 바: [홈] | [저장 경로 관리] | [설정]

[경로 결과 화면 (RouteResultScreen)]
   ├── 상단: 현재 조건 요약 바 & [조건 변경] 버튼
   ├── 시간 안내 배너: "현재 조회 소요시간 기준 단순 가산값 안내"
   ├── 참조 상태 뱃지: "[저장 당시 위치 사용 (참조 장소 삭제됨)]" (해당 시)
   ├── 수단 탭: [전체(혼합)] | [지하철 전용] | [버스 전용]
   ├── 정렬 칩: [빠른 순(기본)] | [환승 적은 순] | [도보 적은 순]
   ├── 경로 목록: RouteSummaryCard 리스트
   └── 하단 액션: [이 검색조건 저장하기]

[경로 상세 화면 (RouteDetailScreen)]
   ├── 요약 헤더: 출발지 → 목적지, 총 소요시간, 예상 도착시각, 요금
   ├── 세부 타임라인 (스텝별 아코디언)
   └── 하단: [출발지 주변 역·정류장] / [목적지 주변 역·정류장] 보기 버튼

[저장 경로 관리 화면 (SavedRoutesScreen)]
   ├── 저장 경로 탭: 카드 순서 변경(Drag), 수정, 삭제
   └── 저장 장소 탭: [🏠 집], [🏢 회사], 즐겨찾기 장소 등록/수정/삭제

[설정 화면 (SettingsScreen)]
   ├── 위치 정보 권한 상태 확인 및 persist() 영속성 안내
   ├── 기본 교통수단 / 기본 정렬 기준 설정
   ├── 데이터 백업/복원/초기화 (JSON Export/Import, 전체 초기화)
   └── 개발 환경 전용 Mock 모드 토글 (DEV 환경 한정, 배포 빌드 시 제거)
```

---

## 2. 화면별 상세 스펙 및 비즈니스 규칙

### 2.1 홈 화면 (HomeScreen)
* **빠른 길찾기 바**:
  - `출발지 인풋`: 기본값 "📍 현재 위치" (터치 시 장소 선택 모달 오픈)
  - `맞바꾸기 버튼 (⇄)` 동작:
    - 출발지가 "동적 현재 위치(`isDynamicLocation: true`)"일 때 맞바꾸기 클릭 시, **목적지는 동적이 될 수 없으므로 현재 시점의 위경도 좌표 및 행정동 주소를 고정 스냅샷으로 변환하여 목적지로 설정**함.
  - `목적지 인풋`: 플레이스홀더 "어디로 갈까요?"
  - `[경로 검색]` 버튼: 출발지와 목적지가 모두 유효할 때 활성화 (최소 터치 높이 48px).
* **저장된 경로 카드 리스트**:
  - 카드 내 `[바로 검색]` 터치 시:
    - 저장 조건이 `SPECIFIC(08:30)`일 때, 현재 시각이 08:30 이전이면 "오늘 08:30"으로 검색.
    - 현재 시각이 08:30을 이미 지났다면 **자동으로 "내일 08:30"으로 간주**하여 검색을 수행하고 결과 화면에 안내 뱃지 노출.
  - 카드 본체 터치: [조건 임시 변경 바텀시트] 오픈.
* **주변 대중교통 퀵 바**:
  - 선택된 출발지(기본: 현재 위치) 또는 목적지를 기준으로 반경 500m 내 버스 정류장과 지하철역 목록 표시.
  - 기본 상위 10개 노출 후, 결과가 더 있으면 하단 **`[더보기 (N개 전체보기)]`** 버튼 제공.

### 2.2 장소 검색 및 위치 획득
* **위치 획득 실패 시 처리 규칙**:
  - 기기 GPS 신호 없음 또는 권한 거부 시, **과거 위치를 자동 사용하지 않음**.
  - 상단에 `"현재 위치를 획득할 수 없습니다. (마지막 확인: 08:10)"` 경고를 노출하고, 사용자가 **[마지막 위치 사용]** 또는 **[직접 장소 검색]** 중 명시적으로 선택하게 함.
* **최근 검색어**:
  - 장소 선택 시 최근 검색어에 자동 기록 (최대 10개 FIFO, 기기 고유의 로컬 캐시).

### 2.3 주변 역·정류장 시트 (NearbyTransitSheet)
* **데이터 제공사**: ODsay `pointSearch` (stationClass=1:2)
* **거리 및 표시 기준**:
  - 기준 좌표와의 **직선거리(하버사인 계산값)** 표시: **"직선거리 N m (보행속도 80m/분 환산 시 약 M분)"**
  - 고유 식별: `stationClass_stationID`로 식별하여 동일 이름의 반대편 정류장(상/하행)이나 ARS-ID 없는 지하철역 유지.

### 2.4 경로 결과 화면 (RouteResultScreen)
* **검색 날짜 및 시각 고정**:
  - 검색 버튼을 누르는 순간 `new Date()`를 호출하여 실제 검색 기준 일시(YYYY-MM-DD HH:mm)를 확정하여 쿼리 실행. 검색 도중 자정이 경과하더라도 해당 검색 세션의 기준 일시가 유지되도록 보장.
* **화면 필수 안내 배너**:  
  `💡 예상 도착 09:15 · 현재 조회된 소요시간 기준 단순 가산값이며, 미래 시각의 실제 배차 간격이나 첫차/막차는 반영되지 않습니다.`
* **정렬 기준 및 동률 처리 (Tie-Breaking)**:
  - `빠른 순 (기본)`: 1순위 소요시간 오름차순 ➔ 2순위 환승횟수 오름차순 ➔ 3순위 도보거리 오름차순
  - `환승 적은 순`: 1순위 환승횟수 오름차순 ➔ 2순위 소요시간 오름차순 ➔ 3순위 도보거리 오름차순
  - `도보 적은 순`: 1순위 도보거리 오름차순 ➔ 2순위 소요시간 오름차순 ➔ 3순위 환승횟수 오름차순
* **결측치 처리 규격**:
  - 소요 시간 결측(<= 0): 결과 목록에서 완전 제외.
  - 도보 거리 결측(`undefined`): 정렬 시 최하위(Infinity)로 처리하여 실제 0m(도보 없음)와 구분.
  - 요금 결측: "요금 정보 없음" 표시 (0원으로 무료 오인 차단).

### 2.5 조건 임시 변경 바텀시트 (TempConditionSheet)
* **조정 가능 항목**:
  1. 출발지 / 2. 목적지 / 3. 출발 날짜([오늘]/[내일]) 및 시각(HH:mm) / 4. 교통수단 필터 / 5. 정렬 기준
* **분기 액션**:
  - `[이 조건으로 1회 검색]`: 원본 `SavedRoute`는 일체 수정하지 않고 메모리 상 파라미터만 오버라이드하여 검색.
  - `[저장 내용 업데이트]`: 사용자가 명시적으로 선택 시에만 원본 레코드 영구 갱신.

### 2.6 장소 참조 및 스냅샷 Fallback 규칙 (출발지/목적지 대칭 적용)
* **대칭 적용 원칙**:
  - **출발지와 목적지의 모든 `placeId`(`origin.placeId`, `destination.placeId`) 참조에 동일한 최신 장소 조회 규칙을 적용한다.** (집, 회사, 일반 즐겨찾기 모두 해당)
* **참조 우선 원칙**:
  - `placeId`가 존재하고 실제 `SavedPlace`에 해당 ID가 등록되어 있으면 ➔ **참조 장소의 최신 좌표와 주소를 우선 사용하여 검색**.
* **스냅샷 Fallback 및 명시적 사용자 안내**:
  - 사용자가 집/회사를 삭제했거나, 백업 파일 복원 시 참조 장소가 누락된 경우 ➔ `SavedRoute` 내에 복사 보관되어 있는 **스냅샷 좌표/주소로 Fallback**하여 검색을 수행.
  - 이 경우 결과 화면 상단에 **`[저장 당시 위치 사용 (참조 장소 삭제됨)]`** 뱃지와 저장 당시 주소를 명시적으로 표시하여, 예전 주소로 잘못 이동하는 오인을 원천 방지함.

### 2.7 설정 화면: 데이터 백업/복원 및 초기화 상세 규칙
* **백업 파일 구조 (JSON)**:
  ```json
  {
    "schemaVersion": 1,
    "exportedAt": 1772947200000,
    "tables": {
      "savedRoutes": [ ... ],
      "savedPlaces": [ ... ],
      "settings": [ ... ]
    }
  }
  ```
  *(최근 검색 장소 `recentPlaces`는 휘발성 로컬 캐시 성격이므로 백업 대상에서 명시적으로 제외함)*
* **복원 파일 사전 검증 규칙 (Validation Rules)**:
  1. **스키마 버전 확인**: `schemaVersion === 1` 확인 (상위 버전 거부).
  2. **필수 필드 확인**: `tables` 내 `savedRoutes`, `savedPlaces`, `settings` 배열 존재 여부 확인.
  3. **유효 좌표 범위 확인**: 각 장소의 `lat` (33.0 ~ 43.0), `lng` (124.0 ~ 132.0) 대한민국 영토 범위 유효성 검증.
  4. 검증 실패 시: 즉시 작업을 중단하고 "유효하지 않은 백업 파일입니다" 에러 안내 및 롤백.
* **복원 및 충돌 처리 옵션**:
  1. **덮어쓰기 (Overwrite - 기본 권장)**:
     - 기존 `savedRoutes`, `savedPlaces`, `settings`를 백업 파일 데이터로 전체 원자적 교체.
     - **`recentPlaces` 보존**: 최근 검색어는 기기 고유의 로컬 캐시이므로 삭제하지 않고 보존.
  2. **병합 (Merge)**:
     - 고유 ID('HOME', 'WORK') 충돌 시: **"가져온 백업 파일 값 우선 적용"** vs **"기존 기기 값 유지"** 선택 팝업 제공.
     - 경로 목록은 충돌하지 않는 경우 `orderIndex`를 기존 최대 순번 뒤로 순차 재배치하여 추가.
* **전체 초기화 (Reset Database)**:
  - **삭제 대상**: `savedRoutes`, `savedPlaces`, `recentPlaces`, `settings` 전체.
  - **확인 절차**: 2차 경고 다이얼로그 표시 후 사용자의 명시적 "초기화 진행" 승인 시 전체 삭제 수행.

---

## 3. 비동기 경쟁 상태(Race Condition) 방지 규격

1. **`AbortController`**: 새 검색 요청 시 이전 요청 즉시 `abort()` 호출.
2. **`latestRequestId` (시퀀스 번호 추적)**: 최신 요청 번호와 일치하지 않는 과거 응답은 완전히 폐기.

---

## 4. 데이터 도메인 모델 정의

```typescript
// 저장 장소 모델
export interface SavedPlace {
  id: string;              // 'HOME' | 'WORK' | UUID
  type: 'HOME' | 'WORK' | 'FAVORITE';
  name: string;            // "집", "회사", "강남 피트니스"
  address: string;
  roadAddress?: string;
  lat: number;
  lng: number;
  createdAt: number;
  updatedAt: number;
}

// 저장 경로 모델 (기본 재사용 템플릿)
export interface SavedRoute {
  id: string;              // UUID
  name: string;            // "회사 출근", "퇴근길"
  origin: {
    placeId?: string;      // 'HOME' | 'WORK' | UUID 참조 (우선 적용)
    name: string;
    address: string;
    lat: number;
    lng: number;
    isDynamicLocation: boolean; // true면 재호출 시 GPS 최신화
  };
  destination: {
    placeId?: string;      // 참조 우선 적용
    name: string;
    address: string;
    lat: number;
    lng: number;
  };
  transportFilter: 'ALL' | 'SUBWAY' | 'BUS';
  timeCondition: {
    type: 'NOW' | 'SPECIFIC';
    timeString?: string;   // "HH:mm" (예: "08:30")
  };
  sortType: 'FASTEST' | 'MIN_TRANSFER' | 'MIN_WALK';
  orderIndex: number;
  createdAt: number;
  updatedAt: number;
}

// 실제 검색 요청 파라미터 (특정 확정 일시 포함)
export interface RouteSearchParams {
  origin: { lat: number; lng: number; name: string };
  destination: { lat: number; lng: number; name: string };
  filter: 'ALL' | 'SUBWAY' | 'BUS';
  departureTime?: {
    dateString: string; // "YYYY-MM-DD" (검색 시작 시점 고정)
    timeString: string; // "HH:mm"
  };
  sortType: 'FASTEST' | 'MIN_TRANSFER' | 'MIN_WALK';
}
```
