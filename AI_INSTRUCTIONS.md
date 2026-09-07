# RouteMate — AI_INSTRUCTIONS.md (AI 코딩 에이전트 상세 작업 지침)

> 본 문서는 RouteMate 프로젝트에 참여하는 AI 코딩 에이전트가 코드 작성 및 변경 시 반드시 따라야 하는 10대 세부 작업 수칙입니다.

---

## 1. 10대 작업 수칙

1. **문서 기반 개발 (Spec-First)**
   - 코드를 작성하기 전에 반드시 [`docs/intent.md`](./docs/intent.md)와 [`docs/spec.md`](./docs/spec.md)의 명세를 먼저 숙지한다.
2. **범위 확장 금지 (No Scope Creep)**
   - `intent.md`의 보류 사항(지도 렌더링, 경유지 분할검색, 도착시간 역산 등)을 사용자의 사전 승인 없이 임의로 구현하지 않는다.
3. **승인 Gate 준수 (Wait for Gates)**
   - 한 단계가 완료되었다고 다음 단계의 구현 코드를 승인 없이 미리 작성하지 않는다. 각 Gate 승인을 거친다.
4. **추상화 계층 유지 (Adapter Pattern)**
   - 외부 API(카카오, ODsay)는 컴포넌트나 화면에서 직접 호출하지 않고 반드시 `services/adapter/` 계층의 인터페이스를 거친다.
5. **보안 및 환경변수 엄수 (Zero Secret Leak)**
   - API Key를 소스 코드에 하드코딩하지 않는다. 모바일 `CapacitorHttp`는 CORS 우회 통신 수단일 뿐 키 은닉 수단이 아니므로, 배포 시 서버리스 프록시를 경유하도록 아키텍처를 유지한다. ([`docs/architecture.md`](./docs/architecture.md) 참조)
6. **CORS 회피 전략 준수**
   - 개발 환경에서는 `vite.config.ts` Proxy 설정을, 모바일에서는 `@capacitor/core`의 `CapacitorHttp`를 사용하여 브라우저 CORS 에러를 원천 방지한다.
7. **저장소 규칙 준수 (Dexie.js & JSON 백업)**
   - IndexedDB 기반 `Dexie.js`를 사용하며, `persist()` 한계를 감안하여 설정 화면의 JSON 백업/복원 기능을 무결하게 지원한다.
8. **UI/UX 원칙 (One-Tap & Fast Comparison)**
   - 불필요한 중간 단계를 배제하고, 저장된 경로는 홈 카드에서 최대 1~2 터치로 결과를 볼 수 있도록 인터랙션을 최소화한다.
9. **사후 검증 및 테스트 필수 (Always Verify)**
   - 코드 변경 후에는 [`docs/test-plan.md`](./docs/test-plan.md)의 테스트 케이스(환승수 계산, 시간대, 원본보존 등)를 실행하여 리그레션을 방지한다.
10. **명확한 변경 보고 (Transparent Reporting)**
    - 파일 변경 내역과 설계 의도를 사용자에게 명확하고 간결하게 설명한다.
