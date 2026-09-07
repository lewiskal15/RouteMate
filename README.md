# RouteMate (대중교통 도우미 앱)

> **"내가 자주 가는 길을 가장 쉽게"**  
> 자주 사용하는 이동 조건을 저장하고, 원터치로 비교하며, 오늘만 유연하게 조건을 임시 변경하는 개인용 대중교통 도우미 앱입니다.

---

> [!NOTE]
> **현재 프로젝트 단계 안내**:  
> 본 프로젝트는 **[기획 및 설계 검토 단계 (Gate 1 ~ Gate 3 사용자 승인 대기 중)]**입니다.  
> 실제 소스 코드(React/Capacitor)는 Gate 1~3 승인 완료 후 Phase 3 부트스트랩 단계부터 순차적으로 생성됩니다.

---

## 1. 권장 문서 읽기 순서

프로젝트 기획 및 구조를 검토하실 때 다음 순서로 읽으시는 것을 권장합니다:

1. [`RouteMate_개발계획서.md`](./RouteMate_개발계획서.md): 프로젝트 전체 개요 및 마일스톤 요약
2. [`PROJECT_STATUS.md`](./PROJECT_STATUS.md): 현재 Gate 승인 대기 현황 및 핵심 제안 사항
3. [`docs/intent.md`](./docs/intent.md): [Gate 1 대상] 왜 만드는가, 요구사항 정의(REQ-01~13), 핵심 제안
4. [`docs/spec.md`](./docs/spec.md): [Gate 2 대상] 화면별 상세 기능, 인터랙션 및 비즈니스 규칙
5. [`docs/architecture.md`](./docs/architecture.md): [Gate 3 대상] 시스템 계층도, Dexie DB, 통신/보안 아키텍처
6. [`docs/api.md`](./docs/api.md): [Gate 3 대상] ODsay 및 카카오 연동 규격, 환승 변환 공식
7. [`docs/plan.md`](./docs/plan.md): [Gate 3 대상] 단계별 구현 로드맵 및 체크리스트
8. [`docs/test-plan.md`](./docs/test-plan.md): [Gate 3 대상] 요구사항 추적 매트릭스 및 필수 품질 검수 시나리오
9. [`AGENTS.md`](./AGENTS.md) & [`AI_INSTRUCTIONS.md`](./AI_INSTRUCTIONS.md): AI 에이전트 행동 지침 및 구현 수칙

---

## 2. 프로젝트 핵심 가치

- **One-Tap Quick Search**: 지도 로딩 대기 없이 홈 카드의 [바로 검색] 터치 1회로 최적 경로 조회.
- **Timeline Comparison**: 지하철/버스/혼합 경로를 소요시간 순으로 한눈에 비교.
- **Flexible Tweaks**: 원본 설정을 훼손하지 않고 오늘만 시간/수단/출발지/목적지를 임시 변경하여 재검색.
- **Nearby Stations**: 출발지와 목적지 주변 반경 500m 내 버스 정류장과 지하철역을 한눈에 확인 (ODsay `pointSearch` 연동).

---

## 3. 기술 스택 요약

- **Frontend**: React 18, TypeScript, Vite, Tailwind CSS
- **Mobile Packaging**: Capacitor (Android)
- **Local Persistence**: IndexedDB (`Dexie.js`) + JSON 백업/복원
- **External APIs**:
  - 장소/주소 검색: 카카오 로컬 REST API
  - 복합 경로 탐색 & 주변 역/정류장: ODsay Lab REST API
- **Testing**: Vitest, React Testing Library
