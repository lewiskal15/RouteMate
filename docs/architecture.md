# RouteMate — architecture.md (시스템 아키텍처 정의서)

> **문서 상태**: [Gate 3 승인 대기]  
> **문서 버전**: 1.3.1  
> **최종 수정일**: 2026-09-07  
> **승인 이력**: (v1.3.0 검토 피드백 반영, v1.3.1 Gate 3 검토 중)

---

## 1. 아키텍처 개요 및 계층 구조

RouteMate는 빠른 UI 반응성과 단일 코드베이스 운영을 위해 **React 18 + TypeScript + Vite + Capacitor** 기반의 하이브리드 아키텍처를 채택한다.  
데이터 제공사의 변경이나 네트워크 환경에 유연하게 대응하기 위해 **Service-Adapter 패턴**을 적용한다.

```text
┌────────────────────────────────────────────────────────┐
│                   Presentation Layer                   │
│        (React 18 + TypeScript + Tailwind CSS)          │
│  HomeScreen │ RouteResultScreen │ RouteDetailScreen    │
│  SavedRoutesScreen │ SettingsScreen │ NearbyTransitSheet │
└──────────────────────────┬─────────────────────────────┘
                           │
┌──────────────────────────▼─────────────────────────────┐
│                    Service Layer                       │
│  LocationService │ RouteQueryService │ StorageService  │
│  NearbyTransitService │ CacheManager                   │
└────────────┬─────────────┴─────────────┬───────────────┘
             │                           │
┌────────────▼────────────┐ ┌────────────▼───────────────┐
│     Adapter Layer       │ │     Persistence Layer      │
│  ┌───────────────────┐  │ │  ┌──────────────────────┐  │
│  │ IPlaceSearchProvider││ │  │   IndexedDB (Dexie)  │  │
│  │ (KakaoLocalAdapter) ││ │  │ - savedRoutes        │  │
│  ├───────────────────┤  │ │  │ - savedPlaces        │  │
│  │ ITransitRouteProvider│ │  │ - recentPlaces       │  │
│  │ (ODsayTransitAdapter)│ │  │ - settings           │  │
│  ├───────────────────┤  │ │  └──────────────────────┘  │
│  │ INearbyTransitProvider │  JSON Import/Export (트랜잭션)│
│  │ (ODsayNearbyAdapter) │ │                            │
│  └───────────────────┘  │                              │
└────────────┬────────────┘                              │
             │                                           │
┌────────────▼───────────────────────────────────────────▼┐
│             Network & Platform Execution Layer          │
│  - Dev Web: Vite Proxy (vite.config.ts)                 │
│  - Dev Mobile: CapacitorHttp (@capacitor/core)          │
│  - Production: Cloudflare Worker API Proxy              │
└─────────────────────────────────────────────────────────┘
```

---

## 2. 레이어별 설계 명세

### 2.1 Service Layer 파라미터 타입
```typescript
export interface RouteSearchParams {
  origin: {
    lat: number;
    lng: number;
    name: string;
  };
  destination: {
    lat: number;
    lng: number;
    name: string;
  };
  filter: 'ALL' | 'SUBWAY' | 'BUS';
  departureTime?: {
    dateString: string; // "YYYY-MM-DD"
    timeString: string; // "HH:mm"
  };
  sortType: 'FASTEST' | 'MIN_TRANSFER' | 'MIN_WALK';
}
```

### 2.2 Adapter Layer (추상화 인터페이스)
```typescript
// 1. 장소 및 주소 검색 어댑터 (카카오 연동)
export interface IPlaceSearchProvider {
  searchKeyword(query: string, userLat?: number, userLng?: number): Promise<PlaceItem[]>;
  searchAddress(address: string): Promise<PlaceItem[]>;
  coordToAddress(lat: number, lng: number): Promise<string>;
}

// 2. 대중교통 복합 경로 검색 어댑터 (ODsay 연동)
export interface ITransitRouteProvider {
  searchRoutes(params: RouteSearchParams, signal?: AbortSignal): Promise<TransitRouteResult[]>;
}

// 3. 주변 역/정류장 조회 어댑터 (ODsay pointSearch 연동 - REQ-04, REQ-06)
export interface INearbyTransitProvider {
  getNearbyStations(lat: number, lng: number, radiusMeters?: number): Promise<NearbyTransitStation[]>;
}
```

---

## 3. 데이터 영속성 (Persistence Layer) 및 트랜잭션 설계

### 3.1 Dexie.js (IndexedDB) 스키마
```typescript
import Dexie, { Table } from 'dexie';
import { SavedRoute, SavedPlace, RecentPlace } from './models';

export interface UserSetting {
  key: string;
  value: any;
}

export class RouteMateDB extends Dexie {
  savedRoutes!: Table<SavedRoute, string>;
  savedPlaces!: Table<SavedPlace, string>;
  recentPlaces!: Table<RecentPlace, string>;
  settings!: Table<UserSetting, string>;

  constructor() {
    super('RouteMateDB');
    this.version(1).stores({
      savedRoutes: 'id, name, orderIndex, createdAt, updatedAt',
      savedPlaces: 'id, type, name, createdAt',
      recentPlaces: 'id, name, searchedAt',
      settings: 'key'
    });
  }
}

export const db = new RouteMateDB();
```

### 3.2 저장소 영속성 (`persist()`) 규칙 복원
* 앱 구동 시 `navigator.storage?.persist()`를 호출하여 브라우저의 저장 데이터 영속 유지를 요청한다.
* `persist()` 결과가 `false`로 거부된 경우:
  - 브라우저나 OS가 저장 공간 부족 시 임의로 IndexedDB를 정리할 수 있음을 인지하고, 설정 화면에 **"데이터 자동 정리 가능성 있음 (주기적 JSON 백업 권장)"** 안내를 표시한다.

### 3.3 백업 덮어쓰기/병합 트랜잭션 및 충돌 해결 로직
* **DB 전체 덮어쓰기 (Overwrite)**:
  - 백업 파일 내 `savedRoutes`, `savedPlaces`, `settings`를 전체 교체한다.
  - **`recentPlaces` 보존 규칙**: 최근 검색어는 기기 고유의 로컬 캐시이므로 전체 덮어쓰기 시에도 삭제하지 않고 그대로 유지한다.
* **병합 (Merge) 충돌 해결 및 트랜잭션 기록**:
  - 충돌 선택("가져온 값 적용" vs "기존 값 유지")에 따라 메모리에서 최종 반영 리스트를 먼저 구성한 후 원자적으로 기록한다.
  ```typescript
  export async function executeRestoreMerge(
    backup: BackupData,
    conflictDecisions: Map<string, 'OVERWRITE' | 'KEEP'>
  ) {
    // 1. 복원 전 임시 백업 보관
    const preBackup = await createPreRestoreBackup();

    try {
      await db.transaction('rw', [db.savedRoutes, db.savedPlaces, db.settings], async () => {
        const existingPlaces = await db.savedPlaces.toArray();
        const existingRoutes = await db.savedRoutes.toArray();
        const maxOrderIndex = existingRoutes.reduce((max, r) => Math.max(max, r.orderIndex), 0);

        // A. 장소 충돌 해결
        for (const newPlace of backup.tables.savedPlaces) {
          const conflict = existingPlaces.find(p => p.id === newPlace.id);
          if (!conflict) {
            await db.savedPlaces.add(newPlace);
          } else if (conflictDecisions.get(newPlace.id) === 'OVERWRITE') {
            await db.savedPlaces.put(newPlace);
          }
          // 'KEEP' 선택 시 아무 작업도 하지 않고 기존 기기 값 유지
        }

        // B. 경로 병합 및 orderIndex 재배치
        let addedCount = 0;
        for (const newRoute of backup.tables.savedRoutes) {
          const conflict = existingRoutes.find(r => r.id === newRoute.id);
          if (!conflict) {
            newRoute.orderIndex = maxOrderIndex + 1 + (++addedCount);
            await db.savedRoutes.add(newRoute);
          } else if (conflictDecisions.get(newRoute.id) === 'OVERWRITE') {
            await db.savedRoutes.put(newRoute);
          }
        }

        // C. 설정 키 병합
        for (const setting of backup.tables.settings) {
          await db.settings.put(setting);
        }
      });
    } catch (error) {
      // 에러 발생 시 자동 롤백되며 기존 데이터 100% 보존
      console.error('복원 트랜잭션 실패, 롤백 수행:', error);
      throw error;
    }
  }
  ```

---

## 4. 캐시 관리 및 통신 아키텍처 (Cache & Proxy)

### 4.1 캐시 관리 (CacheManager)
* **캐시 키**: `route_${originLat.toFixed(4)}_${originLng.toFixed(4)}_${destLat.toFixed(4)}_${destLng.toFixed(4)}_${filter}`
* **도착 시각 실시간 계산 원칙**: 캐시에는 경로의 소요시간과 스텝 정보만 저장하며, **"예상 도착 시각"은 조회 시점의 출발 시각에 소요시간을 가산하여 실시간 계산**함으로써 이전 검색 시각이 재사용되는 오류를 원천 차단함.
* **Fresh 캐시 (3분)** 및 **Stale 캐시 (30분, HTTP 429 또는 오프라인 시 사용)** 운영.

### 4.2 운영 배포 프록시 아키텍처 (Cloudflare Worker)
* **프록시 주소**: `https://routemate-proxy.workers.dev/api/...` (예시이며 배포 시 실제 URL 확정)
* **이중 트래픽 제한**:
  1. **IP별 분당 제한 (30회/분)**: 단시간 내 비정상적 반복 호출 차단.
  2. **제공사별 일일 총량 제한 (900회/일)**: ODsay 무료 일일 1,000건 한도 안전 마진 보호.
* **허용 엔드포인트 화이트리스트**: 카카오 로컬 검색 및 ODsay pointSearch, searchPubTransPathT 경로만 중계하며 그 외 요청은 403 Forbidden 반환.
