# RouteMate Design System (디자인 가이드라인)

> **문서 상태**: [UI/UX 디자인 기준 문서 (Source of Truth)]  
> **버전**: 1.0.0  
> **최종 수정일**: 2026-09-07  
> **목적**: AI 코딩 에이전트, 개발자가 동일한 화면 철학, 타이포그래피, 컬러, 컴포넌트 규칙을 따르도록 하는 단일 기준 문서

---

## 1. Product Design Intent (디자인 의도)

RouteMate는 버스, 지하철, 도보 등 대중교통 이동에 필요한 정보를 빠르고 명확하게 제공하는 개인용 대중교통 이동 도우미 앱이다.

사용자는 대개 다음 상황에서 앱을 사용한다:
- 출발 직전 (바쁘게 준비 중)
- 길을 걷는 중 (한 손 조작)
- 정류장/역에서 대기 중
- 환승 중 (시간이 촉박함)
- 주변이 밝거나 흔들리는 환경

따라서 RouteMate의 디자인은 다음 원칙을 최우선으로 한다:
1. **한눈에 이해되어야 한다.** (Clear First)
2. **한 손 조작이 편해야 한다.** (One-Hand Friendly, 엄지손가락 영역)
3. **정보의 우선순위가 명확해야 한다.** (소요시간 ➔ 환승 ➔ 노선)
4. **이동 중 오작동이 적어야 한다.** (최소 터치 타깃 44~48px)
5. **화려한 그래픽보다 신뢰성, 가독성, 반응속도를 우선한다.**

---

## 2. Core Visual Direction & Principles

### 2.1 Visual Direction
* **Modern Professional Transit UI**: Clean, Calm, Reliable, Fast, Minimal.
* **벤치마크**:
  - Toss: 극단적인 여백과 라운드 처리, 군더더기 없는 텍스트 위계
  - Apple Maps: 단순하고 세련된 미니멀 타임라인
  - Citymapper: 대중교통 정보 밀도와 식별성
* **Avoid (지양 사항)**: 과도한 그라데이션, 장식용 애니메이션, 12px 미만의 미세 폰트, 불필요한 테두리선 남발.

### 2.2 Transit Information Hierarchy (정보 우선순위)
1. **Priority 1 (지금 해야 할 행동 / 핵심 숫자)**: 총 소요시간("45분"), 다음 탑승할 버스/지하철 번호
2. **Priority 2 (시간 정보)**: 예상 도착 시각("09:15 도착"), 남은 시간, 도보 소요 시간
3. **Priority 3 (교통수단 정보)**: 노선 번호, 승하차 정류장/역명, 환승 횟수
4. **Priority 4 (보조 정보)**: 정류장 번호(ARS-ID), 요금, 도보 거리(m)

---

## 3. Color System (디자인 토큰)

### 3.1 Base Colors
```css
/* Background & Surface */
--color-bg: #F7F8FA;                  /* 연한 그레이 배경 */
--color-surface: #FFFFFF;             /* 메인 카드 표면 */
--color-surface-secondary: #F1F3F5;   /* 보조 영역/칩 */

/* Text */
--color-text-primary: #17191C;        /* 본문 핵심 텍스트 */
--color-text-secondary: #5F6670;      /* 레이블, 서브 텍스트 */
--color-text-tertiary: #8A929C;       /* 플레이스홀더, 부가정보 */

/* Borders & Dividers */
--color-border: #E1E5E9;
--color-divider: #ECEFF2;

/* Brand Accent */
--color-primary: #246BFD;             /* 신뢰감 있는 블루 */
--color-primary-hover: #195CE6;
--color-primary-soft: #EAF1FF;        /* 연한 블루 배경 칩 */

/* Status */
--color-success: #2E9D5B;
--color-warning: #E89A24;
--color-danger: #D94A4A;
```

### 3.2 Transit Brand Colors (공식 노선 색상)
대중교통 노선 식별용 공식 컬러를 타임라인 바와 뱃지에 적용한다:
* **수도권 지하철**:
  - 1호선: `#0052A4` | 2호선: `#3CB44A` | 3호선: `#F36E08` | 4호선: `#00A9E0`
  - 5호선: `#996CAC` | 7호선: `#747F00` | 9호선: `#BDB092` | 신분당선: `#D4003B` | 수인분당: `#F5A200`
* **버스 유형**:
  - 간선버스(파랑): `#0068B7` | 지선버스(초록): `#53B332`
  - 광역버스(빨강): `#E60012` | 순환버스(노랑): `#F2B700`

---

## 4. Typography (타이포그래피 규격)

* **권장 폰트**: `Pretendard, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`
* **Type Scale**:
  - `Display` (28px, Bold): 핵심 소요시간 ("45분")
  - `H1` (24px, Bold): 페이지 헤더 ("회사 출근")
  - `H2` (20px, SemiBold): 섹션 타이틀 ("저장된 경로")
  - `H3` (18px, SemiBold): 경로 카드 메인 헤더
  - `Body Large` (16px, Medium): 입력창 텍스트, 주요 버튼
  - `Body` (14px, Regular/Medium): 정류장명, 노선 설명
  - `Caption` (12px, Regular): 보조 안내, 타임스탬프, 결측치 뱃지
* **숫자 강조 (Numeric Emphasis)**: 시간, 분, 거리는 주변 텍스트보다 굵거나 1단계 크게 강조.

---

## 5. Spacing, Radius & Shadows

* **Spacing Grid**: 4px 배수 시스템 (4, 8, 12, 16, 20, 24, 32px). 기본 모바일 좌우 패딩: **16px** (넓은 화면 20px).
* **Corner Radius**:
  - `Input / Button`: 8~12px
  - `Card`: 16px (`rounded-2xl`)
  - `Bottom Sheet`: 상단 20px (`rounded-t-3xl`)
  - `Tag / Badge`: 6px 또는 9999px (Pill)
* **Shadows**:
  - `Card`: `0 2px 8px rgba(0, 0, 0, 0.04)` (은은하고 자연스러운 그림자)
  - `Bottom Sheet / Floating Button`: `0 4px 20px rgba(0, 0, 0, 0.12)`

---

## 6. Core Screen UI Components (핵심 화면 컴포넌트 규격)

### 6.1 홈 화면 (HomeScreen)
1. **빠른 길찾기 바**:
   - 상단 고정, 출발지 ➔ 목적지 전환 버튼(`⇄`), 손가락이 닿기 쉬운 큼직한 인풋(높이 48px).
2. **저장된 경로 카드 (SavedRouteCard - 핵심)**:
   - 카드 상단: 아이콘 + 별칭("🏢 회사 출근") + 기준 시간 뱃지("08:30 출발").
   - 카드 중앙: 출발지 ➔ 목적지 요약.
   - 카드 우측 하단: 돋보이는 `[바로 검색]` 액션 버튼 (원터치 조회).
3. **주변 대중교통 요약 칩**:
   - "📍 주변 정류장 120m · 지하철 350m" 한눈에 확인 가능한 원클릭 칩.

### 6.2 경로 결과 카드 (RouteSummaryCard)
```text
┌────────────────────────────────────────────────────────┐
│ 45분                            09:15 도착 · 도보 5분 │
│ ────────────────────────────────────────────────────── │
│ [도보 3분] ── [🚌 541 (25분)] ── [도보 2분] ── [🚇 2호선] │
│ ────────────────────────────────────────────────────── │
│ 환승 1회 · 1,500원                                    │
└────────────────────────────────────────────────────────┘
```
- 소요시간("45분")을 좌측 상단에 가장 크고 선명하게 배치.
- 중앙에 노선 고유색이 들어간 미니 타임라인 바 배치.

### 6.3 타임라인 아코디언 (Transit Timeline)
- 도보 구간: 점선 및 걷는 사람 아이콘(`Walk`).
- 탑승 구간: 해당 호선/버스 색상의 실선, 승하차 정류장명, 경유 정류장 아코디언 접기/펼치기.

### 6.4 모바일 바텀시트 (Bottom Sheet)
- 아래로 쓸어내리는 제스처 드래그 핸들(`w-12 h-1.5 bg-gray-300 rounded-full mx-auto`) 상단 배치.
- 조건 임시 변경 시트: 날짜/시간 피커, 수단 필터 토글.

---

## 7. Recommended Frontend Stack (UI 구현)

* **Framework**: React 18, TypeScript, Vite
* **Styling**: Tailwind CSS
* **Icons**: `lucide-react` (단일 아이콘 라이브러리 일관성 준수)
* **BottomSheet**: Vaul (또는 Radix Dialog 기반 모바일 드로어)
* **Animation**: Framer Motion (150~250ms 가벼운 모바일 탭 터치 피드백)

---

## 8. UI 작업 완료 기준 (Definition of Done — UI)

- [ ] `design.md`의 타이포그래피, Spacing 토큰 준수
- [ ] 모바일 뷰포트(360px ~ 412px)에서 레이아웃 붕괴 없음
- [ ] 최소 터치 타깃(44 × 44px 이상) 확보
- [ ] 로딩 상태(Skeleton), 빈 상태(Empty), 에러 상태(Error) 대응
- [ ] 한 손 조작 주요 액션이 화면 하단 50% 영역 내 배치
