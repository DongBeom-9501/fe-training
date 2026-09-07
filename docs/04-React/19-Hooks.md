# 19. Hooks — useState / useEffect / useRef

> **이 장의 질문**: `use`로 시작하는 함수들은 무엇이고, 왜 규칙이 붙어 있는가?

## BE로 치면 이런 것

Spring에서 `@Transactional`을 붙이면 메서드 실행 전후로 트랜잭션이 열리고 닫힙니다. 당신은 그 코드를 쓰지 않았지만 프레임워크가 **정해진 지점에 개입** 합니다. 그게 "훅"입니다.

React의 Hooks도 같은 개념입니다. **컴포넌트의 생애에 개입해서, 함수 컴포넌트가 원래 못 하던 일(상태 기억, 외부 시스템 연결)을 하게 해줍니다.**

이름이 전부 `use`로 시작합니다. 관례가 아니라 **린터가 이 접두사로 규칙 위반을 검사** 하기 때문에 지켜야 하는 약속입니다.

## Hooks의 두 가지 규칙

```
1. 컴포넌트 함수의 최상위에서만 호출한다
   → 조건문, 반복문, 중첩 함수 안에서 호출하면 안 된다
2. React 컴포넌트나 커스텀 훅 안에서만 호출한다
   → 일반 함수에서는 안 된다
```

**1번을 어기면 안 되는 이유** 를 알아두면 나머지가 다 이해됩니다.

React는 훅을 **이름이 아니라 호출 순서로 구분합니다.** 컴포넌트 안의 첫 번째 `useState`, 두 번째 `useState`... 이런 식으로 배열에 순서대로 담아 둡니다.

```tsx
function Form() {
  const [name, setName] = useState('');     // 0번 슬롯
  const [email, setEmail] = useState('');   // 1번 슬롯
}
```

그런데 조건문에 넣으면 렌더마다 순서가 달라집니다.

```tsx
// ❌
function Form({ isEdit }) {
  if (isEdit) {
    const [name, setName] = useState('');   // 어떤 때는 0번, 어떤 때는 없음
  }
  const [email, setEmail] = useState('');   // 어떤 때는 1번, 어떤 때는 0번
}
```

`isEdit`이 바뀌는 순간 `email`이 `name`의 값을 읽게 됩니다. 그래서 금지입니다.

> **해결은 간단합니다. 훅을 조건 밖으로 빼고, 조건은 훅 안이나 JSX에서 처리하세요.**

## useState

18장에서 다뤘습니다. 여기서는 놓치기 쉬운 것 두 개만.

```tsx
// 초기값 계산이 무거우면 함수로 넘긴다 (매 렌더마다 재계산되지 않음)
const [rows, setRows] = useState(() => parseHugeCsv(raw));

// ❌ 이러면 렌더마다 parseHugeCsv가 실행된다 (결과는 첫 번째 것만 쓰이면서)
const [rows, setRows] = useState(parseHugeCsv(raw));
```

## useRef — 리렌더를 일으키지 않는 상자

```tsx
const ref = useRef<HTMLInputElement>(null);
```

`useRef`는 두 가지 용도로 쓰입니다.

**용도 1: DOM 요소를 직접 잡기**

16장에서 말한 "명령형이 필요한 경계"가 여기입니다.

```tsx
function SearchBar() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>검색창으로</button>
    </>
  );
}
```

포커스 이동, 스크롤 위치, 미디어 재생 같은 것은 React가 대신해 줄 수 없어서 직접 내려가야 합니다. **나쁜 게 아니라 정당한 용법입니다.**

**용도 2: 렌더 사이에 값을 기억하되, 바뀌어도 리렌더는 안 시키기**

```tsx
const timerRef = useRef<number | null>(null);
timerRef.current = setTimeout(...);   // 바꿔도 리렌더 안 일어남
```

| | useState | useRef |
| --- | --- | --- |
| 값이 살아남나 | ✅ | ✅ |
| 바뀌면 리렌더 | ✅ | ❌ |
| 언제 쓰나 | 화면에 보이는 값 | 화면과 무관한 값 (타이머 ID, 이전 값, DOM 참조) |

**기준은 하나입니다. 이 값이 바뀌면 화면이 달라져야 하는가?** 그렇다면 state, 아니면 ref.

## useEffect — 생명주기가 아니라 동기화

가장 많이 쓰이고 **가장 많이 오용되는** 훅입니다.

```tsx
useEffect(() => {
  // 실행할 것
  return () => {
    // 정리(cleanup) — 다음 실행 전과 언마운트 시
  };
}, [deps]);   // 이 값들이 바뀌면 다시 실행
```

의존성 배열에 따라 동작이 달라집니다.

```tsx
useEffect(() => { ... });            // 매 렌더마다 (거의 안 씀)
useEffect(() => { ... }, []);        // 처음 한 번만
useEffect(() => { ... }, [orderId]); // orderId가 바뀔 때마다
```

### 무엇을 위한 도구인가

**`useEffect`는 "React 바깥의 시스템과 화면을 맞추는" 도구입니다.**

```tsx
// ✅ 정당한 용법들
useEffect(() => {
  const socket = connectWebSocket(roomId);
  return () => socket.close();          // 정리 필수
}, [roomId]);

useEffect(() => {
  document.title = `주문 ${count}건`;   // 브라우저 API
}, [count]);

useEffect(() => {
  const onResize = () => setWidth(window.innerWidth);
  window.addEventListener('resize', onResize);
  return () => window.removeEventListener('resize', onResize);
}, []);
```

공통점이 보이시나요. **웹소켓, document, window** — 전부 React가 관리하지 않는 외부 시스템입니다.

### 정리 함수를 빼먹으면

2장에서 말한 "죽지 않는 프로세스"의 대가가 여기서 나옵니다. 구독을 걸어 놓고 해제하지 않으면 **컴포넌트가 사라져도 계속 살아 있습니다.** 이게 프론트엔드의 메모리 누수입니다.

```tsx
// ❌ 화면을 100번 열었다 닫으면 리스너가 100개 쌓인다
useEffect(() => {
  window.addEventListener('resize', onResize);
}, []);
```

> **규칙**: `useEffect` 안에서 **무언가를 시작했다면 반드시 멈추는 코드** 를 반환하세요. 구독, 타이머, 연결, 이벤트 리스너 전부.

## useEffect를 쓰지 말아야 하는 경우

**이 절이 이 장에서 가장 중요합니다.** 초보자가 `useEffect`를 남용하는 이유는 "부수 효과 = useEffect"라고 외웠기 때문인데, 실제로는 대부분 필요 없습니다.

### 1. 사용자 행동에 대한 반응 → 이벤트 핸들러

```tsx
// ❌ 저장 여부를 state로 만들고 effect로 감시
const [shouldSave, setShouldSave] = useState(false);
useEffect(() => { if (shouldSave) { saveOrder(); setShouldSave(false); } }, [shouldSave]);

// ✅ 그냥 핸들러에서 호출
const handleSave = () => saveOrder();
```

**"사용자가 무언가 했기 때문에" 일어나는 일은 이벤트 핸들러에 씁니다.** `useEffect`는 "화면이 이 상태이기 때문에" 필요한 동기화에 씁니다.

### 2. 다른 state로부터 계산되는 값 → 그냥 계산

```tsx
// ❌ 파생 상태를 state로 두고 effect로 동기화
const [orders, setOrders] = useState([]);
const [total, setTotal] = useState(0);
useEffect(() => { setTotal(orders.reduce((s, o) => s + o.amount, 0)); }, [orders]);

// ✅ 렌더할 때 계산하면 끝
const total = orders.reduce((s, o) => s + o.amount, 0);
```

`useEffect` 버전은 렌더가 두 번 일어나고(orders 갱신 → effect → total 갱신), 중간에 값이 어긋나는 순간이 생깁니다. **다른 값으로부터 계산되는 것은 state가 아닙니다.** 26장에서 다시 다룹니다.

### 3. 데이터 페칭 → 라이브러리 또는 서버 컴포넌트

```tsx
// ❌ 흔하지만 문제가 많다
useEffect(() => {
  fetch(`/api/orders/${id}`).then(r => r.json()).then(setOrder);
}, [id]);
```

이 코드에는 빠진 것이 많습니다 — 로딩 상태, 에러 처리, **경쟁 조건**(id를 빠르게 바꾸면 옛 응답이 새 응답을 덮어씀), 취소, 캐시.

직접 다 처리하는 대신 도구를 씁니다. **Next.js 서버 컴포넌트**(24장)나 **TanStack Query**(27장)가 이 문제를 통째로 해결합니다.

> **정리**: `useEffect`를 쓰기 전에 물어보세요. **"이건 React 바깥의 시스템과 맞추는 일인가?"** 아니라면 다른 방법이 있습니다.

## 의존성 배열 함정

```tsx
// ❌ 무한 루프
useEffect(() => {
  setData({ ...data, loaded: true });
}, [data]);   // effect가 data를 바꾸고 → data가 바뀌어서 effect가 또 돌고 ...
```

```tsx
// ❌ 객체·함수는 매 렌더 새로 만들어져서 항상 "바뀐 것"으로 판정된다
const options = { pageSize: 20 };
useEffect(() => { load(options); }, [options]);   // 매 렌더 실행
```

객체와 함수를 의존성에 넣으면 8장에서 배운 참조 비교 때문에 매번 다릅니다. 해법은 **원시값만 의존성에 넣거나**, `useMemo`/`useCallback`으로 참조를 고정하는 것입니다. 후자는 35장에서 다룹니다.

> **의존성 배열은 린터에게 맡기세요.** `eslint-plugin-react-hooks`가 빠진 의존성을 잡아 줍니다. **경고를 무시하거나 배열을 비워서 넘어가지 마세요.** 그건 대부분 버그를 숨기는 것입니다.

## 커스텀 훅 — 로직 재사용의 단위

`use`로 시작하는 함수를 직접 만들면 그게 커스텀 훅입니다. 훅을 다른 훅 안에서 부를 수 있습니다.

```tsx
// 화면 크기를 추적하는 로직을 한 곳에 모은다
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  useEffect(() => {
    const onResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', onResize);
    return () => window.removeEventListener('resize', onResize);
  }, []);
  return width;
}

// 쓰는 쪽은 한 줄
function Layout() {
  const width = useWindowWidth();
  return width < 768 ? <MobileNav /> : <DesktopNav />;
}
```

**커스텀 훅은 상태를 공유하지 않습니다. 로직을 공유합니다.** 두 컴포넌트가 `useWindowWidth()`를 부르면 각자 별도의 state를 가집니다. 함수를 두 번 호출하면 지역 변수가 두 벌 생기는 것과 같습니다.

## 흔한 오해

**"useEffect는 componentDidMount 같은 것이다"**
생명주기 훅이 아니라 **동기화 도구** 입니다. 이렇게 생각하면 남용하게 됩니다.

**"의존성 배열을 비우면 한 번만 실행되니까 안전하다"**
안에서 쓰는 값이 낡은 채로 고정됩니다(오래된 클로저). 린터 경고를 존중하세요.

**"useRef는 DOM 잡을 때만 쓴다"**
리렌더 없이 값을 기억하는 용도가 절반입니다.

**"커스텀 훅으로 상태를 공유할 수 있다"**
없습니다. 로직만 공유됩니다. 상태 공유는 상태 끌어올리기나 Context입니다(28장).

## 핵심 3가지

- **훅은 호출 순서로 구분됩니다.** 그래서 조건문 안에 넣으면 안 됩니다. 순서가 어긋나면 값이 뒤섞입니다.
- **state냐 ref냐의 기준은 하나입니다.** "이 값이 바뀌면 화면이 달라져야 하는가." 그렇다면 state, 아니면 ref.
- **`useEffect`는 생명주기 훅이 아닙니다.** 바깥 시스템과의 동기화 도구입니다. 사용자 행동은 이벤트 핸들러로, 파생 값은 그냥 계산으로 처리하세요.

## 스스로 답해보기

1. Hooks를 조건문 안에서 호출하면 안 되는 이유를 "슬롯"이라는 단어로 설명해 보세요.
2. `orders`로부터 `total`을 계산하는 데 `useEffect`를 쓰면 무엇이 문제인가?

---

**다음 장 예고** — Part 4의 마지막입니다. 컴포넌트를 언제, 어떤 기준으로 쪼개야 하는지 — 백엔드의 클래스 분리 감각이 어디까지 통하고 어디서 달라지는지 봅니다.
