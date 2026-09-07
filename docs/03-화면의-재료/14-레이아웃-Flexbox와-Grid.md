# 14. 레이아웃 — Flexbox / Grid / 반응형

> **이 장의 질문**: 요소를 원하는 자리에 놓으려면 무엇을 알아야 하는가?

## BE로 치면 이런 것

레이아웃 도구를 고르는 것은 **자료구조를 고르는 것** 과 비슷합니다. `List`를 쓸지 `Map`을 쓸지가 데이터의 모양으로 정해지듯, Flex를 쓸지 Grid를 쓸지도 배치의 모양으로 정해집니다.

기준은 딱 하나입니다.

| | 언제 |
| --- | --- |
| **Flexbox** | 한 방향으로 늘어놓을 때 (1차원) |
| **Grid** | 행과 열을 동시에 정할 때 (2차원) |

실무 비중은 **Flex가 80%, Grid가 20%** 정도입니다. 대부분의 UI는 "가로로 나열", "세로로 쌓기"이기 때문입니다.

## Flexbox — 한 방향 배치

부모에 `display: flex`를 주면 자식들이 한 줄로 늘어섭니다.

```css
.toolbar {
  display: flex;
  gap: 12px;                    /* 자식들 사이 간격 — margin보다 이걸 쓰세요 */
  align-items: center;          /* 교차축 정렬 (세로 가운데) */
  justify-content: space-between; /* 주축 정렬 (양끝으로 밀기) */
}
```

**축 개념만 잡으면 나머지는 다 파생됩니다.**

```
flex-direction: row (기본값)
  주축 →  가로
  교차축 ↓ 세로

flex-direction: column
  주축 ↓  세로
  교차축 → 가로
```

| 속성 | 방향 | 값 |
| --- | --- | --- |
| `justify-content` | **주축** | `flex-start` `center` `flex-end` `space-between` `space-around` |
| `align-items` | **교차축** | `flex-start` `center` `flex-end` `stretch` `baseline` |

> **가장 헷갈리는 지점**: `flex-direction: column`을 주면 `justify-content`가 세로가 됩니다. "justify는 가로"라고 외우면 틀립니다. **justify는 주축, align은 교차축** 입니다.

### 자식 쪽 속성

```css
.sidebar { flex: 0 0 240px; }   /* 늘지도 줄지도 않고 240px 고정 */
.content { flex: 1; }           /* 남은 공간을 다 차지 */
```

`flex: 1`이 실무에서 가장 많이 쓰입니다. "나머지 다 먹어"라는 뜻입니다.

### 실전 패턴 세 개

이 셋이면 대부분의 화면이 만들어집니다.

```css
/* 1. 가로 배치 + 양끝 정렬 (헤더) */
.header { display: flex; justify-content: space-between; align-items: center; }

/* 2. 세로 스택 + 균일 간격 (폼) */
.form { display: flex; flex-direction: column; gap: 16px; }

/* 3. 정중앙 정렬 */
.center { display: flex; justify-content: center; align-items: center; }
```

세 번째는 CSS 역사에서 오래 어려웠던 문제인데, Flexbox 이후로 두 줄이 됐습니다.

## Grid — 행과 열을 동시에

```css
.dashboard {
  display: grid;
  grid-template-columns: 240px 1fr;   /* 사이드바 고정 + 본문 가변 */
  gap: 16px;
}
```

`fr`은 Grid 전용 단위로 **"남은 공간의 비율"** 입니다. `1fr 2fr`이면 1:2로 나눕니다.

### 실전 패턴 두 개

```css
/* 1. 반응형 카드 그리드 — 미디어 쿼리 없이 자동으로 줄바꿈 */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 16px;
}
```

이 한 줄이 강력합니다. "카드 최소 너비 280px, 들어갈 수 있는 만큼 한 줄에, 남는 공간은 균등 분배"라는 뜻이라 **화면 크기별 분기를 쓸 필요가 없습니다.**

`auto-fit`과 `auto-fill` 두 가지가 있는데 헷갈리기 쉽습니다.

| | 카드가 칸보다 적을 때 |
| --- | --- |
| `auto-fit` | 빈 칸을 접어서 **카드를 넓힙니다** |
| `auto-fill` | 빈 칸을 그대로 두어 **카드가 원래 크기로 남습니다** |

카드가 화면을 꽉 채우길 원하면 `auto-fit`, 항상 같은 크기이길 원하면 `auto-fill`입니다.

```css
/* 2. 페이지 전체 골격 */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "nav    main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}
.layout > header { grid-area: header; }
.layout > nav    { grid-area: nav; }
.layout > main   { grid-area: main; }
.layout > footer { grid-area: footer; }
```

`grid-template-areas`는 **레이아웃을 그림으로 그리는** 문법입니다. 읽기가 아주 쉬워서 페이지 골격에 적합합니다.

## 반응형 — 화면 크기가 제각각인 세상

입문 2장에서 말한 "실행 환경이 하나가 아니다"가 여기서 실무 문제가 됩니다.

### 모바일 퍼스트

```css
/* 기본: 모바일 (좁은 화면) */
.grid { display: grid; grid-template-columns: 1fr; }

/* 768px 이상에서 덮어쓰기 */
@media (min-width: 768px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}

/* 1024px 이상 */
@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); }
}
```

**`min-width`로 위로 쌓는 방식** 을 씁니다. 이유가 두 가지입니다.

1. 좁은 화면이 제약이 더 큽니다. 어려운 쪽을 먼저 풀고 넓힐 때 더하는 게 쉽습니다
2. 13장의 캐스케이드 — 나중에 선언한 게 이기므로, 넓은 화면 규칙이 뒤에 와야 자연스럽습니다

관례적인 분기점입니다. 외울 필요는 없고 Tailwind 같은 도구가 이 값을 기본으로 씁니다.

```
640px   모바일 가로 / 작은 태블릿
768px   태블릿
1024px  노트북
1280px  데스크톱
```

### 미디어 쿼리 없이 반응형 만들기

가능하면 분기 자체를 피하는 게 좋습니다. 유지보수할 게 줄어듭니다.

```css
.cards { grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); }  /* 자동 줄바꿈 */
.container { width: min(100%, 1200px); margin-inline: auto; }             /* 최대폭 + 가운데 */
.toolbar { display: flex; flex-wrap: wrap; gap: 8px; }                    /* 넘치면 다음 줄로 */
```

### 뷰포트 설정

이게 없으면 모바일에서 데스크톱 화면을 축소해서 보여줍니다. Next.js는 기본으로 넣어 줍니다.

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

## 흔한 함정

### 1. `height: 100%`가 안 먹는다

```css
.page { height: 100%; }   /* 아무 일도 안 일어남 */
```

`%` 높이는 **부모의 높이가 정해져 있어야** 계산됩니다. 부모가 `auto`면 기준이 없습니다.

```css
/* 해결 1: 화면 높이 기준 */
.page { min-height: 100vh; }

/* 해결 2: Flex/Grid로 부모가 높이를 나눠주게 */
.layout { display: grid; grid-template-rows: auto 1fr auto; min-height: 100vh; }
```

### 2. Flex 자식이 안 줄어든다

긴 텍스트나 테이블이 들어 있는 flex 자식은 **내용 최소 크기 밑으로 줄어들지 않습니다.** 그래서 부모를 뚫고 나갑니다.

```css
.content { flex: 1; min-width: 0; }   /* 이 한 줄이 해결 */
```

`min-width: 0`은 처음 보면 마법처럼 보이는데, **"내용보다 작아져도 된다"고 허락하는 것** 입니다. 실무에서 자주 만납니다.

### 3. 모바일에서 `100vh`가 잘린다

모바일 브라우저의 주소창 때문에 `100vh`가 실제 보이는 영역보다 큽니다.

```css
.full { min-height: 100dvh; }   /* dvh = 동적 뷰포트 높이 */
```

## Flex와 Grid 중 무엇을 쓸까

```
자식들을 한 줄(또는 한 열)로 늘어놓는다        → Flex
행과 열을 동시에 통제해야 한다                → Grid
카드가 화면 크기에 따라 자동으로 줄바꿈된다     → Grid (auto-fill)
페이지 전체 골격                             → Grid (template-areas)
버튼 묶음, 툴바, 폼 한 줄, 아이콘+텍스트        → Flex
```

**섞어 씁니다.** Grid로 페이지 골격을 잡고, 각 영역 안은 Flex로 채우는 게 가장 흔한 구성입니다.

## 흔한 오해

**"Grid가 Flex의 상위 호환이다"**
용도가 다릅니다. 1차원 배치에 Grid를 쓰면 오히려 장황합니다.

**"`float`로 레이아웃을 짠다"**
Flexbox 이전의 방식입니다. 오래된 코드에서만 만납니다.

**"반응형은 미디어 쿼리로 하는 것"**
`auto-fill`, `flex-wrap`, `min()` 으로 분기 없이 되는 경우가 많습니다. 분기는 적을수록 좋습니다.

**"`justify-content`는 가로 정렬이다"**
주축 정렬입니다. `flex-direction: column`이면 세로가 됩니다.

## 핵심 3가지

- **한 방향이면 Flex, 행과 열이면 Grid.** 실무 비중은 Flex가 압도적입니다.
- **축 개념 하나면 됩니다.** `justify-content`는 주축, `align-items`는 교차축. `direction: column`이면 주축이 세로가 됩니다.
- **함정 셋을 기억하세요.** `height: 100%`가 안 먹는 것, Flex 자식이 안 줄어드는 것(`min-width: 0`), 모바일에서 `100vh`가 잘리는 것(`100dvh`).

## 스스로 답해보기

1. 사이드바는 240px 고정, 본문은 나머지를 다 차지하게 하려면 어떻게 쓰는가? (Flex와 Grid 각각)
2. Flex 자식 안의 긴 텍스트가 부모를 뚫고 나간다. 무엇을 추가해야 하며 그것은 무슨 뜻인가?

---

**다음 장 예고** — Part 3의 마지막입니다. CSS를 쓸 줄 알게 됐는데도 실무 CSS는 왜 그렇게 잘 망가질까요? 원인은 문법이 아니라 **CSS가 전역이라는 구조적 성질** 에 있습니다.
