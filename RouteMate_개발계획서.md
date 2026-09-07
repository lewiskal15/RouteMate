# RouteMate 종합 개발계획서 (요약본)

> **문서 상태**: [Gate 1~3 검토용 요약서]  
> **버전**: 1.3.1  
> **최종 수정일**: 2026-09-07  
> *본 문서는 프로젝트 전체 마일스톤과 아키텍처 방향을 파악하기 위한 종합 요약본입니다. 구현 및 검증에 관한 상세 규격은 [`/docs`](./docs/) 폴더의 각 전문 문서를 기준으로 관리됩니다.*

---

## 1. 프로젝트 개요

* **프로젝트명**: **RouteMate**
* **부제**: 내가 자주 가는 길을 가장 쉽게 (스마트 대중교통 동반자)
* **목적**: 기존 종합 지도 앱의 복잡한 길찾기를 복제하는 것이 아니라, 사용자가 자주 이용하는 대중교통 이동 조건을 저장하고, 홈 화면에서 1~2회 터치만으로 오늘 최적 경로를 빠르게 비교·재사용하는 개인용 도우미 앱을 구축한다.

---

## 2. 핵심 사양 및 제안 요약 (REQ-01 ~ REQ-13)

1. **출발지/목적지 간편 선택**: 현재 위치(재호출 시 동적 GPS 갱신), 장소/주소 검색(카카오), 저장 장소(집/회사), 최근 검색
2. **주변 역·정류장 조회**: 출발지 및 목적지 주변 반경 500m 이내 지하철역 및 버스정류장 동시 조회 (ODsay `pointSearch` 연동, 클라이언트 하버사인 직선거리 계산, 상위 10개 + 더보기)
3. **대중교통 복합 경로 탐색**: 지하철 전용, 버스 전용, 혼합(버스+지하철) 경로 탐색 (ODsay `searchPubTransPathT` 연동, 지하철 노선명 `lane[].name` 매핑)
4. **결과 비교 및 정렬**: 빠른 순(기본), 환승 적은 순, 도보 적은 순 정렬 (소요시간 결측치 제외, 도보 결측치 Infinity 최하위 처리)
5. **시간 조건 기준 통일 (제안)**: 출발 장소에서 이동을 시작하는 시각 기준 단순 가산 도착시각 표시 및 배차 미반영 안내 배너 의무화
6. **경로 저장 및 1-Tap 재호출**: 홈 카드에서 원터치로 오늘 기준 최적 경로 재검색
7. **장소 참조 대칭 연동**: 출발지/목적지 모두 최신 장소 주소 대칭 연동, 장소 삭제 시 스냅샷 fallback 및 안내 뱃지 노출
8. **유연한 조건 임시 변경**: 원본을 유지하면서 오늘만 출발지/목적지/시간/수단을 1회성 변경 검색
9. **데이터 영속성 및 백업 (REQ-13)**: IndexedDB (`Dexie.js`) 스토리지 + 설정 화면 JSON 사전검증/백업/복원 (원자적 트랜잭션, 충돌 해결, 롤백 보장, `recentPlaces` 보존)

---

## 3. 기술 구조 및 운영 보안 요약

* **프론트엔드**: React 18, TypeScript, Vite, Tailwind CSS
* **모바일 패키징**: Capacitor (Android 우선 타깃, `@capacitor/geolocation`)
* **데이터 저장소**: IndexedDB (`Dexie.js`) + JSON 백업/복원
* **통신 및 키 보안**:
  - 모바일 통신: `@capacitor/core`의 `CapacitorHttp`로 웹뷰 CORS 원천 회피.
  - 보안 정책: Cloudflare Worker 무인증 API 프록시 (IP별 분당 30회 + 일일 900회 이중 제한)를 경유하여 배포 APK 번들에 Secret Key 미포함 보장.

---

## 4. 개발 마일스톤 및 승인 Gate

| 단계 | 산출물 | 핵심 내용 | 승인 기준 |
|---|---|---|---|
| **Phase 0** | [`docs/intent.md`](./docs/intent.md) | 왜 만드는가, 요구사항(REQ-01~13), Non-Goals 표 | **Gate 1 승인 대기** |
| **Phase 1** | [`docs/spec.md`](./docs/spec.md) | 화면별 UI, 시간 안내 문구, 장소참조 대칭/삭제안내, 백업 트랜잭션 | **Gate 2 승인 대기** |
| **Phase 2** | [`docs/architecture.md`](./docs/architecture.md)<br>[`docs/api.md`](./docs/api.md)<br>[`docs/plan.md`](./docs/plan.md)<br>[`docs/test-plan.md`](./docs/test-plan.md) | 아키텍처, 지하철 lane[].name, 프록시 이중제한, 캐시 실시간 계산, 테스트 10종 | **Gate 3 승인 대기** |
| **Phase 3** | UI Prototype | Mock Adapter를 통한 실제 2-Tap UX 및 터치 동선 검증 | **Gate 4 승인** |
| **Phase 4~6** | API 연동 & 경로 비교 | 카카오 검색 + 주변 교통 + ODsay 경로 탐색 + Worker 프록시 연동 | **Gate 5 승인** |
| **Phase 7** | 저장/임시변경/백업 | 조건 임시 변경 및 MVP 최종 통합 검수 (APK 키 미포함 검증) | **Gate 6 승인 (MVP 1.0)** |

---

## 5. 상세 문서 참조 안내

각 영역의 구체적인 세부 규격은 다음 전문 문서를 참조하십시오:
* 제품 의도 및 요구사항 정의: [`docs/intent.md`](./docs/intent.md)
* 상세 기능 및 UI 명세서: [`docs/spec.md`](./docs/spec.md)
* 시스템 아키텍처 정의서: [`docs/architecture.md`](./docs/architecture.md)
* 외부 API 연동 규격서: [`docs/api.md`](./docs/api.md)
* 구현 로드맵 및 체크리스트: [`docs/plan.md`](./docs/plan.md)
* 테스트 및 품질 검증 계획서: [`docs/test-plan.md`](./docs/test-plan.md)
* 현재 상태 및 다음 액션: [`PROJECT_STATUS.md`](./PROJECT_STATUS.md)
