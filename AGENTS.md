# RouteMate — AGENTS.md (AI 에이전트 행동 규칙)

> 본 문서는 RouteMate 프로젝트에 참여하는 모든 AI 에이전트(Antigravity 등)가 작업 착수 전 반드시 읽고 따라야 하는 가이드라인입니다.

---

## 1. 에이전트 핵심 행동 원칙

1. **Gate 승인 준수 (Never Skip Gates)**
   - Gate 승인이 나지 않은 단계의 코드를 절대 미리 구현하지 마십시오.
   - 각 Gate 단계의 산출물(문서 또는 코드)을 완료한 후, 사용자의 명시적인 승인을 요청하십시오.
2. **요구사항 추적성 유지 (Traceability)**
   - 모든 구현과 테스트는 [`docs/intent.md`](./docs/intent.md)의 요구사항 식별자(`REQ-01` ~ `REQ-13`)를 기준으로 수행하십시오.
3. **가상 데이터 투명성 (Mock Transparency)**
   - API 연결 전 Mock 모드일 때는 화면 상단에 반드시 **[MOCK 가상 데이터 모드]** 뱃지를 표시하여 사용자가 실제 교통 데이터로 오인하지 않도록 하십시오.
4. **외부 API 직접 호출 금지 (Adapter Pattern)**
   - 화면 컴포넌트에서 카카오나 ODsay API를 직접 호출하지 마십시오. 반드시 `services/adapter/` 계층의 인터페이스를 통해 접근하십시오.
5. **키 보안 및 통신 정책 준수**
   - API Key를 소스 코드에 하드코딩하지 마십시오. 배포 환경에서는 서버리스 프록시를 경유해야 하며, 모바일 통신은 `CapacitorHttp`를 사용하십시오. ([`docs/architecture.md`](./docs/architecture.md) 참조)
6. **세부 작업 수칙**
   - 구체적인 10대 작업 원칙은 [`AI_INSTRUCTIONS.md`](./AI_INSTRUCTIONS.md)를 준수하십시오.

---

## 2. 참조 문서 맵 및 읽기 순서

- **기획 및 요구사항**: [`docs/intent.md`](./docs/intent.md) (Gate 1)
- **기능 명세서**: [`docs/spec.md`](./docs/spec.md) (Gate 2)
- **UI/UX 디자인 시스템**: [`docs/design.md`](./docs/design.md) (Design Source of Truth)
- **시스템 아키텍처**: [`docs/architecture.md`](./docs/architecture.md) (Gate 3)
- **외부 API 연동 규격**: [`docs/api.md`](./docs/api.md) (Gate 3)
- **구현 로드맵 및 일정**: [`docs/plan.md`](./docs/plan.md) (Gate 3)
- **테스트 계획서**: [`docs/test-plan.md`](./docs/test-plan.md) (Gate 3)
- **현재 프로젝트 상태**: [`PROJECT_STATUS.md`](./PROJECT_STATUS.md)
