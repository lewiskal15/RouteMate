# RouteMate — PROJECT_STATUS.md (프로젝트 상태 및 승인 관리)

> **최종 갱신일**: 2026-09-07  
> **현재 단계**: Phase 0 ~ Phase 2 문서 v1.3.1 정밀 보완 완료 ➔ **Gate 1 / Gate 2 / Gate 3 사용자 승인 대기**

---

## 1. 마일스톤 및 Gate 승인 현황

| Gate | 승인 대상 산출물 (버전) | 현재 상태 | 승인자 | 승인일 | 핵심 검토 및 승인 조건 |
|---|---|:---:|:---:|:---:|---|
| **Gate 1** | `docs/intent.md` (v1.3.0) | **검토 대기** | - | - | REQ-01~13 정의, Non-Goals 표, 시간 단일화/경유지 분리 제안 검토 |
| **Gate 2** | `docs/spec.md` (v1.3.1) | **검토 대기** | - | - | 화면 UI, 장소참조 대칭적용/삭제안내, 백업 사전검증 및 초기화 규격 검토 |
| **Gate 3** | `docs/architecture.md` (v1.3.1)<br>`docs/api.md` (v1.3.1)<br>`docs/plan.md` (v1.3.0)<br>`docs/test-plan.md` (v1.3.0) | **검토 대기** | - | - | 지하철 lane[].name 정정, 백업 병합 트랜잭션, 프록시 이중 제한, 캐시 실시간 계산 검토 |
| **Gate 4** | Phase 3 UI Mock Prototype | 미착수 | - | - | 프로젝트 스캐폴딩 및 가상 데이터 기반 2-Tap UX 검증 |
| **Gate 5** | Phase 4~6 API 연동 검증 | 미착수 | - | - | 카카오/ODsay 실제 대중교통 경로, 주변교통 및 Cloudflare Worker 프록시 검증 |
| **Gate 6** | Phase 7 MVP 1.0 최종 검수 | 미착수 | - | - | 조건 저장/임시변경/백업 및 Android APK 빌드/Secret Key 미포함 검증 |

---

## 2. v1.3.1 핀포인트 보완 내역 (2026-09-07)

1. **지하철 노선명 매핑 정정**:
   - `docs/api.md`에서 지하철 노선명 필드를 `subPath[].lane[].name`으로 정확히 복원 (주변시설 `laneName`과 분리).
2. **백업 병합 트랜잭션 코드 로직 정정**:
   - `docs/architecture.md`에서 사용자의 충돌 선택("가져온 값" vs "기존 값 유지")을 먼저 반영한 뒤 원자적으로 기록하는 실제 병합 로직 구현. (경로 `orderIndex` 재배치 포함, `recentPlaces` 보존).
3. **장소 참조 대칭 적용 및 삭제 안내**:
   - `docs/spec.md`에서 `origin.placeId`, `destination.placeId` 모두에 최신 장소 조회 대칭 적용.
   - 참조 장소 삭제 시 스냅샷 Fallback 검색을 수행하되 화면에 **`[저장 당시 위치 사용 (참조 장소 삭제됨)]`** 뱃지와 주소 명시.
4. **누락되었던 세부 규칙 복원**:
   - **캐시 정책 복원**: 캐시 키, Fresh 3분/Stale 30분, **캐시된 소요시간에 이번 검색 출발시각을 실시간 가산하여 도착시각 계산** 원칙 명시.
   - **백업 파일 사전 검증 복원**: `schemaVersion === 1`, 필수 필드, 대한민국 좌표 범위 유효성 검증.
   - **저장소 영속성 (`persist()`) 복원**: 브라우저 거부 시 안내 및 JSON 백업 연계.
   - **전체 초기화 규칙 복원**: 2차 위험 경고 및 전체 테이블 삭제.
5. **프록시 정책 통일**:
   - IP별 분당 제한(30회/분) + 제공사별 일일 총량 제한(900회/일), 허용 화이트리스트 및 에러 규격 통일.

---

## 3. 승인 후 변경 관리 규칙 (Change Control)

* 한 번 Gate 승인이 완료된 기획/스펙/아키텍처 문서는 임의로 수정할 수 없습니다.
* 구현 중 기술적 블로커나 요구사항 변경이 발생할 경우:
  1. 즉시 구현을 멈추고 관련 문서에 변경 제안을 기록.
  2. 사용자에게 변경 사유와 영향 범위를 보고하고 **재승인(Re-Gate)**을 득한 후 작업을 재개합니다.

---

## 4. 다음 즉시 작업 (Next Actions)

1. **사용자의 Gate 1 ~ Gate 3 문서 승인 획득**
2. 승인 완료 즉시 ➔ **Phase 3 착수**:
   - Vite + React 18 + TypeScript + Tailwind CSS 스캐폴딩
   - Capacitor 초기화 (`@capacitor/core`, `@capacitor/android`, `@capacitor/geolocation`)
   - `Dexie.js` 스키마 셋업 (`RouteMateDB`)
   - `MockTransitAdapter` 및 `MockPlaceAdapter` 기반 UI Mock 프로토타입 구현
