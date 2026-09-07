# RouteMate Design System

> RouteMate UI/UX 기준 문서  
> 목적: AI 코딩 에이전트, 개발자, 디자이너가 동일한 화면 철학과 규칙을 따르도록 하는 단일 기준 문서(Source of Truth)

---

## 1. Product Design Intent

RouteMate는 버스, 지하철, 도보 등 대중교통 이동에 필요한 정보를 빠르고 명확하게 제공하는 이동 지원 앱이다.

사용자는 대개 다음 상황에서 앱을 사용한다.

- 출발 직전
- 길을 걷는 중
- 버스정류장 또는 지하철역에서 대기 중
- 환승 중
- 한 손으로 스마트폰을 조작하는 상황
- 주변이 밝거나 어두운 상황
- 시간이 촉박한 상황

따라서 RouteMate의 디자인은 다음 원칙을 최우선으로 한다.

1. **한눈에 이해되어야 한다.**
2. **한 손 조작이 편해야 한다.**
3. **정보의 우선순위가 명확해야 한다.**
4. **이동 중에도 오작동이 적어야 한다.**
5. **다음 행동이 무엇인지 사용자가 고민하지 않아야 한다.**
6. **화려함보다 신뢰성, 가독성, 예측 가능성을 우선한다.**

---

# 2. Core Design Principles

## 2.1 Clear First

화면에서 가장 중요한 정보는 항상 가장 먼저 보여야 한다.

예:

- 도착까지 남은 시간
- 출발/도착 위치
- 다음 버스 또는 지하철
- 환승 여부
- 예상 소요시간
- 도보 거리
- 출발 권장 시각

중요하지 않은 세부 정보는 보조 영역 또는 상세 화면에 배치한다.

---

## 2.2 One-Hand Friendly

모바일 사용을 기본으로 한다.

주요 액션은 가능한 화면 하단 또는 엄지손가락이 접근하기 쉬운 위치에 둔다.

주요 버튼 최소 높이:

- 기본: 44px 이상
- 핵심 CTA: 48px 이상
- 모바일 터치 영역 최소: 44 × 44px

작은 아이콘만 단독으로 배치하는 경우에도 실제 터치 영역은 44 × 44px 이상 확보한다.

---

## 2.3 Transit Information Hierarchy

대중교통 정보는 다음 우선순위를 따른다.

### Priority 1
현재 사용자가 지금 해야 할 행동

예:

- 3분 후 542번 버스 탑승
- 2번 출구로 이동
- 다음 역에서 하차
- 5분 후 환승

### Priority 2
시간 정보

- 도착 예정
- 출발 예정
- 남은 시간
- 전체 소요시간

### Priority 3
교통수단 정보

- 버스 번호
- 지하철 노선
- 정류장/역명

### Priority 4
보조 정보

- 정류장 번호
- 차량 혼잡도
- 배차 간격
- 도보 거리
- 요금

---

# 3. Visual Direction

## 3.1 Style

RouteMate는 다음 디자인 방향을 따른다.

**Modern Professional Transit UI**

키워드:

- Clean
- Calm
- Reliable
- Fast
- Accessible
- Minimal
- Information-first

참고 방향:

- Google Maps의 정보 구조
- Apple Maps의 단순성
- Citymapper의 대중교통 정보 밀도
- Linear의 정돈된 UI
- Material Design 3의 접근성

단, 특정 앱의 디자인을 그대로 복제하지 않는다.

---

## 3.2 Avoid

다음 스타일은 기본적으로 사용하지 않는다.

- 과도한 그라데이션
- 지나친 Glassmorphism
- 지나치게 둥근 카드
- 큰 그림자
- 장식 목적의 애니메이션
- 과도하게 작은 글씨
- 정보보다 아이콘이 더 강조되는 UI
- 컬러를 너무 많이 사용하는 화면
- 버튼처럼 보이지 않는 핵심 액션

---

# 4. Color System

색상은 가능한 CSS 변수 또는 디자인 토큰으로 관리한다.

직접 HEX 값을 화면마다 반복 작성하지 않는다.

## 4.1 Base Colors

```css
--color-bg: #F7F8FA;
--color-surface: #FFFFFF;
--color-surface-secondary: #F1F3F5;

--color-text-primary: #17191C;
--color-text-secondary: #5F6670;
--color-text-tertiary: #8A929C;

--color-border: #E1E5E9;
--color-divider: #ECEFF2;

--color-primary: #246BFD;
--color-primary-hover: #195CE6;
--color-primary-soft: #EAF1FF;

--color-success: #2E9D5B;
--color-warning: #E89A24;
--color-danger: #D94A4A;
--color-info: #3578E5;
```

---

## 4.2 Transit Colors

대중교통 노선 고유 색상이 있는 경우 실제 노선 색상을 우선 사용한다.

예:

- 지하철 노선색
- 버스 유형색
- 환승 강조색

단, 노선색은 장식이 아니라 **식별 목적**으로만 사용한다.

노선색 위 텍스트는 WCAG 대비 기준을 만족하도록 한다.

---

## 4.3 Dark Mode

향후 다크모드를 지원할 수 있도록 구조적으로 토큰을 분리한다.

```css
--dark-bg: #111316;
--dark-surface: #191C20;
--dark-surface-secondary: #22262B;

--dark-text-primary: #F4F6F8;
--dark-text-secondary: #B9C0C8;

--dark-border: #30353B;
```

다크모드에서도 노선색의 의미가 유지되어야 한다.

---

# 5. Typography

한글 가독성을 최우선으로 한다.

권장 폰트:

1. Pretendard
2. SUIT
3. Noto Sans KR
4. 시스템 기본 sans-serif

기본 권장:

```css
font-family:
  Pretendard,
  "Noto Sans KR",
  -apple-system,
  BlinkMacSystemFont,
  "Segoe UI",
  sans-serif;
```

---

## 5.1 Type Scale

| 역할 | 크기 | 굵기 | 사용 예 |
|---|---:|---:|---|
| Display | 28px | 700 | 핵심 도착시간 |
| H1 | 24px | 700 | 페이지 제목 |
| H2 | 20px | 700 | 주요 섹션 |
| H3 | 18px | 600 | 카드 제목 |
| Body Large | 16px | 500 | 주요 정보 |
| Body | 14px | 400~500 | 일반 내용 |
| Caption | 12px | 400~500 | 보조 정보 |

12px 미만 텍스트는 원칙적으로 사용하지 않는다.

---

## 5.2 Numeric Emphasis

시간, 분, 거리 등 숫자는 매우 중요하다.

예:

**3분**
**12:45**
**1.2 km**

숫자는 주변 텍스트보다 1단계 높은 크기 또는 굵기를 사용할 수 있다.

---

# 6. Spacing System

기본 spacing 단위는 **4px**이다.

허용 권장값:

```text
4
8
12
16
20
24
32
40
48
```

임의의 13px, 17px, 19px 등의 간격 사용은 지양한다.

---

## 6.1 Page Spacing

모바일:

- 좌우 기본 여백: 16px
- 넓은 모바일: 20px
- 섹션 간격: 24~32px

태블릿/데스크톱:

- 콘텐츠 최대 폭: 1200~1440px
- 좌우 여백: 24~32px
- 주요 패널 간격: 24px

---

# 7. Radius

기본 Radius:

```text
Small: 6px
Medium: 8px
Large: 12px
XL: 16px
```

권장:

- Input: 8px
- Button: 8px
- Card: 12px
- Bottom Sheet: 상단 16px
- Modal: 12~16px

과도한 pill 형태는 태그, 필터, 상태표시 등에만 사용한다.

---

# 8. Shadows

그림자는 최소화한다.

기본적으로 카드 구분은 Border와 배경색으로 처리한다.

권장:

```css
--shadow-sm:
  0 1px 2px rgba(0,0,0,0.05);

--shadow-md:
  0 4px 12px rgba(0,0,0,0.08);
```

큰 그림자는 Modal, Bottom Sheet 등 화면 위 레이어에만 사용한다.

---

# 9. Layout

## 9.1 Mobile First

RouteMate는 Mobile First로 설계한다.

대표 기준 폭:

```text
360
390
412
768
1024
1440
```

기본 모바일 디자인 기준:

**390px**

---

## 9.2 Desktop

데스크톱에서는 모바일 화면을 단순히 크게 확대하지 않는다.

권장 구조:

```text
┌───────────────┬──────────────────────────────┐
│ Sidebar       │ Main Content                 │
│               │                              │
│ Search        │ Map / Route / Information    │
│ Favorites     │                              │
│ Recent        │                              │
└───────────────┴──────────────────────────────┘
```

지도 기반 화면의 경우:

- Desktop: 지도 + 정보 패널
- Mobile: 지도 + Bottom Sheet

구조를 권장한다.

---

# 10. App Shell

## Mobile

```text
┌─────────────────────────┐
│ Header                  │
├─────────────────────────┤
│                         │
│ Main Content            │
│                         │
├─────────────────────────┤
│ Bottom Navigation       │
└─────────────────────────┘
```

---

## Desktop

```text
┌─────────────────────────────────────┐
│ Top Header                          │
├───────────┬─────────────────────────┤
│ Sidebar   │ Main Content            │
│           │                         │
└───────────┴─────────────────────────┘
```

---

# 11. Navigation

Bottom Navigation 권장 메뉴:

1. 홈
2. 길찾기
3. 주변
4. 즐겨찾기
5. 설정

단, 실제 기능 수가 적은 초기 버전에서는 3~4개로 줄인다.

Bottom Navigation에서 현재 메뉴는 명확하게 강조한다.

아이콘 + 텍스트를 함께 사용한다.

---

# 12. Header

모바일 기본:

높이:

```text
56px
```

포함 가능:

- 뒤로가기
- 화면 제목
- 검색
- 즐겨찾기
- 더보기

Header에 너무 많은 기능을 배치하지 않는다.

---

# 13. Buttons

## Primary

주요 행동.

예:

- 경로 검색
- 출발하기
- 저장
- 길안내 시작

높이:

```text
48px
```

---

## Secondary

보조 행동.

예:

- 다른 경로 보기
- 상세 보기

---

## Tertiary

작은 보조 액션.

예:

- 수정
- 삭제
- 다시 검색

---

## Destructive

삭제, 초기화 등 위험 행동에는 Danger 색상 사용.

삭제는 가능한 확인 절차를 둔다.

---

# 14. Input Fields

기본 높이:

```text
44~48px
```

구성:

```text
Label
[ Icon | Input text | Clear ]
Helper / Error
```

Placeholder만으로 입력 의미를 설명하지 않는다.

---

# 15. Location Search

출발지 / 도착지 입력은 RouteMate의 핵심 UI이다.

권장 구조:

```text
● 출발지
  현재 위치

● 도착지
  어디로 갈까요?
```

출발지/도착지 전환 버튼 제공.

최근 검색은 입력 직후 바로 접근 가능해야 한다.

추천 순서:

1. 현재 위치
2. 즐겨찾기
3. 최근 검색
4. 검색 결과

---

# 16. Route Result Card

경로 카드에서는 다음 정보가 가장 중요하다.

```text
42분
12:15 → 12:57

도보 4분
↓
버스 542
↓
2호선
↓
도보 3분

환승 1회 · 1,500원
```

카드의 모든 정보를 같은 강조도로 표현하지 않는다.

권장 계층:

1. 총 소요시간
2. 출발/도착시간
3. 교통수단
4. 환승
5. 요금
6. 도보거리

---

# 17. Transit Timeline

경로 상세는 Timeline 형태를 권장한다.

```text
● 현재 위치
│
│ 도보 4분
│
● 서울역
│
│ 1호선
│  8개 역
│
● 시청역
│
│ 도보 3분
│
● 목적지
```

노선색을 Timeline에 활용한다.

---

# 18. Bus UI

버스 정보 표시 우선순위:

1. 버스 번호
2. 도착 예정 시간
3. 남은 정류장 수
4. 현재 위치
5. 혼잡도
6. 다음 차량

예:

```text
542

3분
2정거장 전

다음 버스 11분
```

"3분"이 가장 강하게 보여야 한다.

---

# 19. Subway UI

지하철 정보 우선순위:

1. 노선
2. 방향
3. 도착 예정
4. 탑승 위치
5. 환승역
6. 하차역

예:

```text
2호선
신도림 방면

2분 후 도착
```

---

# 20. Map UI

지도에는 너무 많은 UI를 겹치지 않는다.

지도 위 허용:

- 현재 위치 버튼
- 지도 확대/축소
- 검색
- 경로 Overlay
- Bottom Sheet

지도 핀 선택 시 상세 정보는 Bottom Sheet로 표시한다.

---

# 21. Bottom Sheet

모바일 지도 화면의 핵심 패턴으로 사용한다.

상태:

```text
Collapsed
Half
Expanded
```

사용자가 드래그하여 높이를 변경할 수 있다.

Bottom Sheet 내부 스크롤과 지도 드래그가 충돌하지 않도록 구현한다.

---

# 22. Cards

Card는 정보 그룹을 표현할 때 사용한다.

기본:

```text
background: surface
border: 1px solid border
radius: 12px
padding: 16px
```

카드 안에 카드가 반복되는 구조는 지양한다.

---

# 23. Lists

List Row 높이 권장:

```text
56~72px
```

구성:

```text
[Icon] Main text
       Secondary text       >
```

---

# 24. Icons

아이콘은 하나의 라이브러리를 일관되게 사용한다.

권장:

- Lucide
- Material Symbols

권장 크기:

```text
16
20
24
```

아이콘 스타일을 혼합하지 않는다.

예:

- Filled Material Icon
- Outline Lucide

를 한 화면에서 섞지 않는다.

---

# 25. States

모든 화면은 최소한 다음 상태를 고려한다.

## Loading

Skeleton UI를 우선 사용한다.

전체 화면 Spinner는 가능한 피한다.

---

## Empty

예:

```text
최근 검색이 없습니다.

목적지를 검색해 보세요.
```

항상 다음 행동을 제시한다.

---

## Error

좋지 않은 예:

```text
Error 502
```

좋은 예:

```text
교통 정보를 불러오지 못했습니다.

네트워크 연결을 확인한 후 다시 시도해주세요.

[다시 시도]
```

---

## Offline

인터넷 연결이 끊긴 경우 명확히 표시한다.

예:

```text
오프라인 상태입니다.
일부 실시간 정보가 표시되지 않을 수 있습니다.
```

---

# 26. Feedback

사용자의 행동에는 즉각 반응한다.

예:

- 즐겨찾기 저장
- 경로 복사
- 출발지 변경

Toast 권장 표시시간:

```text
2~3초
```

---

# 27. Animation

Animation은 정보 이해를 돕는 경우에만 사용한다.

권장:

```text
150~250ms
```

사용 예:

- Bottom Sheet
- Page Transition
- Dropdown
- Route Expand

장식 목적 애니메이션은 지양한다.

---

# 28. Accessibility

WCAG 2.2 AA 수준을 기본 목표로 한다.

필수:

- 텍스트 대비 확보
- 키보드 접근 가능
- Focus 표시
- Screen Reader Label
- Color만으로 상태 구분 금지
- 최소 터치영역 44px
- 확대 시 레이아웃 붕괴 방지

---

# 29. Responsive Rules

## ≤ 599px

Mobile

- Bottom Navigation
- Bottom Sheet
- 단일 열

## 600~1023px

Tablet

- 2열 가능
- 지도 + Floating Panel 가능

## ≥1024px

Desktop

- Sidebar
- 지도 + 정보 Panel
- Keyboard Shortcut 지원 가능

---

# 30. Core Screens

초기 RouteMate에서 디자인 우선순위가 높은 화면:

## P0

1. Home
2. 출발지/도착지 검색
3. 경로 검색 결과
4. 경로 상세
5. 실시간 이동 안내

## P1

6. 주변 정류장
7. 버스 도착 정보
8. 지하철 도착 정보
9. 즐겨찾기
10. 최근 검색

## P2

11. 설정
12. 알림
13. 개인정보/권한
14. 오류/오프라인 화면

---

# 31. Home Screen

권장 구성:

```text
RouteMate

[ 어디로 갈까요? ]

집      회사      즐겨찾기

최근 검색
----------------
서울역
강남역
인천공항

주변 교통
----------------
버스정류장 120m
지하철역 350m
```

홈 화면을 기능 메뉴판처럼 만들지 않는다.

사용자의 첫 행동은 목적지 검색이어야 한다.

---

# 32. Current Trip Screen

실시간 이동 화면은 앱에서 가장 중요한 화면 중 하나이다.

예:

```text
다음 행동

3분 후
542번 버스 탑승

서울역 버스정류장
120m

[경로 보기]
```

"다음 행동"이 항상 화면 상단에 나타나야 한다.

---

# 33. Permission Screens

위치 권한 요청은 갑자기 OS 팝업부터 띄우지 않는다.

먼저 RouteMate 화면에서 이유를 설명한다.

예:

```text
현재 위치를 사용하면
가까운 버스정류장과 지하철역을 찾을 수 있습니다.

[위치 사용]
```

---

# 34. Design Tokens

권장 구조:

```text
tokens/
├── colors
├── typography
├── spacing
├── radius
├── shadows
└── motion
```

코드에서도 동일 개념으로 관리한다.

---

# 35. Recommended Frontend Stack

React 기반 RouteMate 웹앱이라면:

```text
React
TypeScript
Tailwind CSS
shadcn/ui
Lucide Icons
```

을 기본 권장한다.

컴포넌트 라이브러리는 무분별하게 여러 개 혼용하지 않는다.

---

# 36. Component Architecture

공통 UI는 재사용 가능한 컴포넌트로 만든다.

예:

```text
components/
├── Button
├── IconButton
├── Input
├── SearchInput
├── Card
├── TransitBadge
├── RouteCard
├── Timeline
├── BottomSheet
├── Modal
├── Toast
├── EmptyState
├── LoadingState
└── ErrorState
```

페이지마다 동일한 버튼이나 카드 CSS를 새로 작성하지 않는다.

---

# 37. Figma Rules

Figma에서도 동일한 디자인 시스템을 유지한다.

필수:

- Auto Layout 사용
- Component 사용
- Variant 사용
- Variables 사용
- Text Style 사용
- Color Style / Variable 사용
- 동일 UI 직접 복제 금지

주요 Figma Components:

```text
Button
Input
Search
Card
Route Card
Bus Badge
Subway Badge
Bottom Navigation
Header
Timeline
Bottom Sheet
Dialog
Toast
Empty State
```

---

# 38. AI Coding Agent Rules

AI는 UI를 수정할 때 다음을 준수한다.

1. `design.md`를 먼저 읽는다.
2. 기존 디자인 시스템과 컴포넌트를 확인한다.
3. 공통 컴포넌트가 있으면 재사용한다.
4. 화면별 임의 색상값을 추가하지 않는다.
5. spacing은 디자인 토큰을 우선 사용한다.
6. 새로운 UI 패턴을 추가해야 할 경우 기존 패턴으로 해결 가능한지 먼저 확인한다.
7. 기능 변경과 대규모 디자인 변경을 동시에 하지 않는다.
8. 기존 기능을 디자인 변경 과정에서 삭제하지 않는다.
9. Desktop과 Mobile 레이아웃을 모두 확인한다.
10. 수정 후 실제 화면 캡처로 시각적 검증을 한다.

---

# 39. UX Review Checklist

화면 구현 후 다음을 확인한다.

### Information

- 가장 중요한 정보가 가장 먼저 보이는가?
- 숫자와 시간 정보가 잘 보이는가?
- 불필요한 정보가 너무 많지 않은가?

### Interaction

- 다음 행동이 명확한가?
- 버튼이 충분히 큰가?
- 한 손 조작이 가능한가?
- 클릭 횟수가 불필요하게 많지 않은가?

### Visual

- spacing이 일관적인가?
- 글자 크기가 일관적인가?
- 색상이 너무 많지 않은가?
- 카드가 과도하게 많지 않은가?

### Mobile

- 키보드가 중요한 UI를 가리지 않는가?
- Bottom Sheet 조작이 자연스러운가?
- Safe Area가 적용되는가?

### Accessibility

- 대비가 충분한가?
- 작은 글씨가 없는가?
- 색상만으로 상태를 구분하지 않는가?

---

# 40. Definition of Done — UI

UI 작업은 다음 조건을 만족해야 완료로 본다.

- design.md 규칙 준수
- Desktop 확인
- Mobile 확인
- 주요 상태 Loading / Empty / Error 확인
- 키보드 접근 확인
- 터치영역 확인
- 기존 기능 정상 작동
- Browser Console Error 없음
- 주요 화면 Screenshot 검토 완료
- 필요 시 Figma 기준 화면과 비교 완료

---

# 41. Design Change Policy

디자인 시스템 자체를 변경할 경우:

1. 변경 이유를 설명한다.
2. 기존 화면에 미치는 영향을 확인한다.
3. `design.md`를 먼저 수정한다.
4. Figma Component를 수정한다.
5. 코드 Component를 수정한다.
6. 각 화면을 순차 적용한다.

화면 하나 때문에 전체 디자인 규칙을 임의 변경하지 않는다.

---

# 42. Final Design Philosophy

RouteMate의 디자인 목표는

> "멋있어 보이는 앱"이 아니라  
> **"이동 중 필요한 정보를 가장 빠르게 이해하고 행동할 수 있는 앱"**

이다.

사용자가 RouteMate를 열었을 때 항상 다음 세 가지가 명확해야 한다.

1. **나는 지금 어디에 있는가**
2. **다음에 무엇을 해야 하는가**
3. **목적지까지 얼마나 남았는가**

모든 화면과 기능은 이 세 가지 질문을 기준으로 설계한다.
