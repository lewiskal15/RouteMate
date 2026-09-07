# RouteMate — api.md (외부 API 연동 규격서)

> **문서 상태**: [Gate 3 승인 대기]  
> **문서 버전**: 1.3.1  
> **최종 수정일**: 2026-09-07  
> **승인 이력**: (v1.3.0 검토 피드백 반영, v1.3.1 Gate 3 검토 중)

---

## 1. 외부 API 연동 구성표 및 데이터 출처

| 기능 구분 | 제공사 | 공식 엔드포인트 | 조회 데이터 및 비고 |
|---|---|---|---|
| **장소 키워드 검색** | 카카오 로컬 | `GET /v2/local/search/keyword.json` | 장소명, 도로명/지번 주소, 좌표(WGS84) |
| **정밀 주소 검색** | 카카오 로컬 | `GET /v2/local/search/address.json` | 도로명/지번 단위 정밀 좌표 획득 |
| **좌표 ➔ 행정동 주소** | 카카오 로컬 | `GET /v2/local/geo/coord2address.json` | 현재 GPS 좌표를 문자열 주소로 변환 |
| **주변 역·정류장 조회** | **ODsay Lab** | `GET /v1/api/pointSearch` | **지하철역 및 버스정류장 반경 동시 조회 (REQ-04, REQ-06)** |
| **대중교통 복합 경로** | **ODsay Lab** | `GET /v1/api/searchPubTransPathT` | 지하철, 버스, 혼합 환승 경로 및 소요시간 |

---

## 2. 주변 역·정류장 조회 규격 (ODsay `pointSearch`)

### 2.1 요청 규격 및 파라미터
- **엔드포인트**: `GET https://api.odsay.com/v1/api/pointSearch`
- **요청 쿼리 파라미터**:
  - `apiKey`: 발급받은 ODsay API Key
  - `x`: 기준 경도 (lng, WGS84)
  - `y`: 기준 위도 (lat, WGS84)
  - `radius`: 검색 반경 (미터 단위, 기본 `500`)
  - `stationClass`: **`1:2`** (콜론 구분자로 버스정류장 `1`과 지하철역 `2` 동시 조회)
- **요청 URL 예시**:  
  `https://api.odsay.com/v1/api/pointSearch?apiKey={KEY}&x=127.0276&y=37.4979&radius=500&stationClass=1:2`

### 2.2 응답 구조 및 직선거리 계산 (하버사인 공식)
* **공식 규격 확인**: ODsay `pointSearch` 응답 객체(`result.station[]`)에는 **`distance` 필드가 제공되지 않습니다.**
* **클라이언트 직선거리 계산 필수 적용**:
  - 기준점 좌표(`userLat`, `userLng`)와 시설 좌표(`station.y`, `station.x`)를 바탕으로 앱 클라이언트에서 **하버사인 공식(Haversine Formula)**을 적용하여 `distanceMeters`를 산출합니다.
  ```typescript
  function calculateDistanceMeters(lat1: number, lon1: number, lat2: number, lon2: number): number {
    const R = 6371e3; // 지구 반경 (m)
    const phi1 = (lat1 * Math.PI) / 180;
    const phi2 = (lat2 * Math.PI) / 180;
    const deltaPhi = ((lat2 - lat1) * Math.PI) / 180;
    const deltaLambda = ((lon2 - lon1) * Math.PI) / 180;

    const a = Math.sin(deltaPhi / 2) ** 2 +
              Math.cos(phi1) * Math.cos(phi2) * Math.sin(deltaLambda / 2) ** 2;
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return Math.round(R * c);
  }
  ```

### 2.3 고유 식별 및 중복 처리 규칙
* **식별자 생성**: 동일 이름의 반대 방향 정류장(상/하행)이나 ARS-ID가 없는 지하철역이 누락되지 않도록 **`stationClass` + `_` + `stationID`** 복합 키로 고유 식별합니다. (명칭 기반 중복 제거 절대 금지)
* **표시 제한 및 더보기**:
  - 1차로 직선거리 오름차순 정렬 후 **상위 10개**를 기본 렌더링.
  - 검색된 총 시설이 10개를 초과할 경우 하단에 **`[더보기 (N개 전체보기)]`** 버튼을 제공하여 누르면 전체 목록을 전개.

---

## 3. 대중교통 복합 경로 탐색 규격 (ODsay `searchPubTransPathT`)

### 3.1 요청 규격
- **엔드포인트**: `GET https://api.odsay.com/v1/api/searchPubTransPathT`
- **파라미터**:
  - `apiKey`: ODsay API Key
  - `SX`, `SY`: 출발지 경도/위도
  - `EX`, `EY`: 목적지 경도/위도
  - `SearchPathType`: `0`(전체/혼합), `1`(지하철 전용), `2`(버스 전용)
  - `SearchType`: `0`(도시 내 이동)

### 3.2 시간 파라미터 미지원과 클라이언트 처리 규칙
* **API 한계**: ODsay 공개 API는 미래 시각 파라미터를 받지 않으며 현재 기준 표준 스케줄을 반환합니다.
* **화면 필수 안내 문구**:  
  > `💡 예상 도착 09:15 · 현재 조회된 소요시간 기준 단순 가산값이며, 미래 시각의 실제 배차 간격이나 첫차/막차는 반영되지 않습니다.`

---

## 4. 데이터 매핑 및 변환 공식 (ODsay ➔ RouteMate)

| 내부 도메인 필드 | ODsay 원본 필드 | 타입/단위 | 변환 공식 및 결측치/이상치 처리 규칙 |
|---|---|---|---|
| `routeId` | 자체 생성 | string | `pathType_${pathType}_${index}_${totalTime}` 조합 고유 키 |
| **`totalTimeMinutes`** | `info.totalTime` | number (분) | **결측 또는 <= 0인 경우: 유효하지 않은 경로로 결과 목록에서 완전 제외** |
| **`totalWalkDistanceMeters`** | `info.totalWalk` | number (m) | **공식 원본은 도보 이동 거리(m). 결측 시 `undefined`** (정렬 시 최하위 처리) |
| `totalWalkMinutes` | `subPath[].sectionTime` | number (분) | `subPath` 중 `trafficType === 3`(도보)의 `sectionTime` 분 합산 |
| **`totalTransfers`** | `subPath[]` 계산 | number (회) | **하단 [4.1 환승 횟수 변환 공식] 참조** |
| **`fare`** | `info.payment` | number (원) | 결측 또는 0인 경우 `undefined` (UI에 **"요금 정보 없음"** 표시) |
| `steps[].stepType` | `subPath[].trafficType` | enum | `1` ➔ `SUBWAY`, `2` ➔ `BUS`, `3` ➔ `WALK` |
| `steps[].sectionTime` | `subPath[].sectionTime` | number (분) | 구간 소요 시간 |
| `steps[].distanceMeters`| `subPath[].distance` | number (m) | 구간 이동 거리 |
| **`steps[].routeName`** | `subPath[].lane[].busNo` / **`subPath[].lane[].name`** | string | **버스는 `busNo`, 지하철은 `lane[].name` 매핑** (주변시설의 laneName과 분리) |
| `steps[].startStationName` | `subPath[].startName` | string | 승차역/정류장명 |
| `steps[].endStationName` | `subPath[].endName` | string | 하차역/정류장명 |
| `steps[].stationCount` | `subPath[].stationCount` | number | 정차 역/정류장 수 |
| `steps[].arsId` | `subPath[].startArsID` | string | 버스 5자리 정류장 번호 (결측 시 null) |
| `steps[].way` | `subPath[].way` | string | 방면 정보 (결측 시 null) |

### 4.1 환승 횟수 계산 공식 (Transit Leg Formula)
```typescript
// 대중교통(지하철 또는 버스) 탑승 구간 개수 카운트
const transitLegs = subPath.filter(leg => leg.trafficType === 1 || leg.trafficType === 2);
const transitLegCount = transitLegs.length;

// 실제 환승 횟수 = 승차 횟수 - 1
const totalTransfers = Math.max(0, transitLegCount - 1);
```

#### 검증 예정 사례 (API 실제 응답 검증 대상):
- **사례 A (지하철 직통: 당산 ➔ 합정)**: 탑승 1구간 ➔ `transitLegCount = 1` ➔ `totalTransfers = 0` (직통)
- **사례 B (버스 2회 승차)**: 버스 2구간 ➔ `transitLegCount = 2` ➔ `totalTransfers = 1` (환승 1회)
- **사례 C (지하철 호선 환승: 2호선 ➔ 3호선)**: 지하철 2구간 ➔ `transitLegCount = 2` ➔ `totalTransfers = 1` (환승 1회)
- **사례 D (혼합: 버스 ➔ 지하철 ➔ 버스)**: 탑승 3구간 ➔ `transitLegCount = 3` ➔ `totalTransfers = 2` (환승 2회)

---

## 5. 통신 방식 및 운영 프록시 보안 정책

### 5.1 통신 방식 vs 키 보관 방식 분리
* **CapacitorHttp**: 웹뷰의 브라우저 CORS 제약을 회피하는 **네이티브 HTTP 통신 기술**로 사용.
* **운영 프록시 (Cloudflare Worker) 정책**:
  - **프록시 주소**: `https://routemate-proxy.workers.dev/api/...` (문서 표기는 예시이며 배포 후 확정된 실제 URL 사용).
  - **허용 엔드포인트 화이트리스트**:  
    `/api/kakao/search/keyword`, `/api/kakao/search/address`, `/api/kakao/geo/coord2address`, `/api/odsay/pointSearch`, `/api/odsay/searchPubTransPathT`. 그 외 요청은 `403 Forbidden` 차단.
  - **트래픽 및 쿼터 보호 이중화 정책**:
    1. **IP별 분당 제한 (Rate Limit: 30회/분)**: 짧은 시간 내 비정상적인 반복 호출 및 디도스 방지.
    2. **제공사별 일일 총량 제한 (Daily Quota Limit: 900회/일)**: ODsay 무료 일일 1,000건 한도를 초과하지 않도록 서버 차원에서 안전 마진 유지.
  - **통일 오류 응답 규격**:  
    ```json
    {
      "error": "Too Many Requests",
      "code": 429,
      "message": "일일 대중교통 조회 한도를 초과했습니다.",
      "retryAfter": 3600
    }
    ```

---

## 6. 캐싱 및 장애 Fallback 정책 (Cache Policy)

### 6.1 캐시 키 및 수명 주기
* **캐시 키 생성 규칙**:  
  `route_${originLat.toFixed(4)}_${originLng.toFixed(4)}_${destLat.toFixed(4)}_${destLng.toFixed(4)}_${filter}`  
  *(좌표는 소수점 4자리 반올림하여 반경 약 11m 이내 미세 이동 시 캐시 적중률 극대화)*
* **캐시 수명 주기**:
  1. **신선 캐시 (Fresh Cache, TTL 3분)**: 동일 조건 재검색 시 네트워크 호출 없이 즉시 반환.
  2. **장애/한도초과 캐시 (Stale Cache, TTL 30분)**: HTTP 429(일일 쿼터 초과) 또는 오프라인 장애 시 최근 30분 이내 성공 데이터를 반환하고, 화면에 **[오래된 데이터 (오프라인/한도초과)]** 뱃지 및 **조회 시점 타임스탬프**를 명시.

> **[중요] 캐시 저장 및 도착 시각 계산 원칙**:  
> 캐시 저장소에는 **"경로 원본 결과(소요시간, 환승 구간, 도보 거리 등)"**만 보관합니다.  
> **"예상 도착 시각"은 이번 검색의 출발 시각에 소요시간을 가산하여 매번 실시간으로 새로 계산**하여, 이전 검색의 과거 시각이 재사용되는 버그를 원천 차단합니다.
