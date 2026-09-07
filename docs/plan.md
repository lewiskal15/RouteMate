# RouteMate — plan.md (구현 로드맵 및 작업 계획서)

> **문서 상태**: [Gate 3 승인 대기]  
> **문서 버전**: 1.3.0  
> **최종 수정일**: 2026-09-07  
> **승인 이력**: (v1.2.0 검토 피드백 반영, v1.3.0 Gate 3 검토 중)

---

## 1. 개발 마일스톤 및 Gate 승인 체계

AI 에이전트는 기획/설계 단계의 문서 초안을 선행 작성할 수 있으나, **각 Gate의 사용자 명시적 승인이 완료되기 전에 다음 단계의 구현 코드를 작성할 수 없습니다.**

```text
[Phase 0] 프로젝트 의도 기획 ──▶ Gate 1 승인 (/docs/intent.md)
       │
[Phase 1] 화면 및 기능 명세  ──▶ Gate 2 승인 (/docs/spec.md)
       │
[Phase 2] 기술 설계 & 검증 계획 ──▶ Gate 3 승인 (/docs/architecture.md, api.md, plan.md, test-plan.md)
       │
[Phase 3] 프로젝트 부트스트랩 & UI Mock 프로토타입 ──▶ Gate 4 승인 (UX 및 터치 동선 검증)
       │
[Phase 4~6] 장소/주변교통/경로 API & 운영 프록시 ──▶ Gate 5 승인 (실제 경로 데이터 및 프록시 검증)
       │
[Phase 7] 저장/임시변경/백업 완성 ──▶ Gate 6 승인 (MVP 1.0 최종 검수 및 릴리즈)
```

---

## 2. 단계별 세부 작업 명세 및 체크리스트

### Phase 0: 프로젝트 의도 확정 (Gate 1)
* **착수 조건**: 사용자 요구사항 분석 시작
* **작업 체크리스트**:
  - [x] `docs/intent.md` 초안 작성 (REQ-01~13 정의, Non-Goals 표 복원)
  - [x] 출발시간 기준 단일화 및 경유지 Phase 8 분리 제안 작성
  - [ ] **사용자 Gate 1 검토 및 승인**

### Phase 1: 화면 및 기능 명세 (Gate 2)
* **착수 조건**: Gate 1 승인 완료
* **작업 체크리스트**:
  - [x] `docs/spec.md` 초안 작성 (화면별 UI 명세, 임시 조건 변경 로직)
  - [x] `origin.placeId` 통일, 장소 참조 우선 및 스냅샷 fallback 규칙
  - [x] 검색 시점 날짜 고정 및 시간 안내 배너 의무화
  - [x] 백업/복원 원자적 트랜잭션, HOME/WORK 충돌 해결 규칙
  - [ ] **사용자 Gate 2 검토 및 승인**

### Phase 2: 기술 구조 및 상세 설계 (Gate 3)
* **착수 조건**: Gate 2 승인 완료
* **작업 체크리스트**:
  - [x] `docs/architecture.md` (계층 구조, Dexie 트랜잭션, Cloudflare Worker 운영 프록시)
  - [x] `docs/api.md` (카카오 + ODsay pointSearch 하버사인 직선거리 계산, 환승 공식)
  - [x] `docs/test-plan.md` (REQ-01~13 추적표, 복원/신규 테스트 케이스 10종)
  - [ ] **사용자 Gate 3 검토 및 승인**

---

### Phase 3: 부트스트랩 & UI Mock 프로토타입 (Gate 4)
* **착수 조건**: Gate 3 승인 완료
* **작업 체크리스트**:
  - [ ] Vite + React 18 + TypeScript + Tailwind CSS 스캐폴딩
  - [ ] `@capacitor/core`, `@capacitor/android`, `@capacitor/geolocation` 초기화
  - [ ] 저장소 기반 구축: `Dexie.js` 스키마 초기화 (`RouteMateDB`)
  - [ ] `MockTransitAdapter` 및 `MockPlaceAdapter` 기반 화면 구현:
    - 홈 화면 (빠른 검색창, 저장 경로 카드, 주변 교통 요약)
    - 조건 임시 변경 바텀시트
    - 경로 결과 화면 (시간 안내 배너, 소요시간순 정렬 탭, 타임라인 요약 카드)
    - 경로 상세 화면
  - [ ] **사용자 Gate 4 검토 및 승인** (모바일 화면에서의 2-Tap 사용성 및 화면 흐름 검증)

### Phase 4: 장소 검색, 플랫폼별 GPS 위치 획득 및 주변 교통 연동
* **착수 조건**: Gate 4 승인 완료
* **작업 체크리스트**:
  - [ ] 플랫폼별 GPS 위치 획득 파이프라인 구현:
    - 웹: `navigator.permissions` 질의 및 `navigator.geolocation.getCurrentPosition` 호출
    - Android: `@capacitor/geolocation`의 `checkPermissions()` 및 `requestPermissions()` 호출
    - 타임아웃 및 거부 실패 시 "마지막 확인 위치(시각) 표시 + 사용자 선택" fallback 구현
    - 출발지가 현재 위치일 때 검색 시점 동적 GPS 최신화 구현
  - [ ] `KakaoLocalAdapter` 구현 (키워드 검색, 주소 검색, 좌표-주소 변환)
  - [ ] 최근 검색어 자동 저장(Dexie DB 연동, 최대 10개 FIFO)
  - [ ] `ODsayNearbyAdapter` 구현 (ODsay `pointSearch` stationClass=1:2 기반 하버사인 직선거리 계산 및 상위 10개 + 더보기 구현)

### Phase 5: 대중교통 경로 탐색 연동 및 운영 프록시 구축 (Gate 5 준비)
* **착수 조건**: Phase 4 완료
* **작업 체크리스트**:
  - [ ] `ODsayTransitAdapter` 구현 (도보거리m 매핑, 소요시간 결측 제외, 환승수 공식 `Math.max(0, legCount - 1)`)
  - [ ] 로컬 개발: Vite Dev Proxy (`server.proxy`) 연동
  - [ ] **배포 운영용 Cloudflare Worker API 프록시 구축 및 배포**:
    - ODsay 및 카카오 Secret Key 주입, 엔드포인트 화이트리스트, 분당 호출 제한(Rate Limit) 설정
    - 앱이 프록시 엔드포인트를 호출하도록 환경별 BASE_URL 분기
    - 프록시 장애 시 화면 에러 처리 및 재시도 UI 구현

### Phase 6: 경로 비교 및 정렬 엔진 (Gate 5)
* **착수 조건**: Phase 5 완료
* **작업 체크리스트**:
  - [ ] 빠른 순, 환승 적은 순, 도보 적은 순(도보 결측치 Infinity 최하위 처리) 정렬 로직 구현
  - [ ] 수단 필터링 (전체, 지하철 전용, 버스 전용)
  - [ ] 3분 Fresh 캐시 및 30분 Stale Fallback 캐시(조회 시각 타임스탬프 뱃지) 구현
  - [ ] 최신 요청 번호(`latestRequestId`) 및 `AbortController` 기반 경쟁 상태 방지
  - [ ] **사용자 Gate 5 검토 및 승인** (실제 대중교통 경로 탐색 데이터 및 프록시 통신 검증)

### Phase 7: 저장/임시변경/백업 & 최종 통합 (Gate 6 — MVP 1.0 릴리즈)
* **착수 조건**: Gate 5 승인 완료
* **작업 체크리스트**:
  - [ ] 홈 화면 카드 [바로 검색] 1-Tap 재검색 파이프라인 완성
  - [ ] 저장 조건 임시 변경 로직 완성 (원본 유지 vs 덮어쓰기 분기 검증)
  - [ ] 저장 장소(집, 회사, 즐겨찾기) CRUD 및 저장 경로 최신 주소 참조 연동 완성
  - [ ] 설정 화면 데이터 백업/복원(JSON Export/Import 원자적 트랜잭션 및 충돌 팝업) 완성
  - [ ] **배포 APK 보안 검증: 디컴파일 및 바이너리 grep을 통해 외부 API Secret Key 미포함 확인**
  - [ ] 전체 단위 테스트 실행 및 UX 성능 실측
  - [ ] **사용자 Gate 6 최종 검수 및 승인 (MVP 1.0 릴리즈)**

---

## 3. 후속 단계 계획 (Phase 8 ~ 11)

- **Phase 8**: 사용자 지정 경유지(1개 지점) 및 경유지 주변 역·정류장 조회
- **Phase 9**: 실시간 버스 잔여 정류장 수 및 지하철 도착 정보
- **Phase 10**: 도착 희망 시각 역산 스마트 출발 알림
- **Phase 11**: AI 자연어 기반 이동 경로 요약 및 추천
