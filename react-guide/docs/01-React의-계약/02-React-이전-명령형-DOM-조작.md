# 2. React 이전 — 명령형 DOM 조작이 무너지는 지점

> **이 장의 질문**: React 없이 화면을 만들면 정확히 무엇이 무너지는가?

## BE로 치면 이런 것

당신의 시스템에 **진실의 원천(single source of truth)** 이 없다고 상상해 보세요. 주문 상태가 DB에도 있고, 캐시에도 있고, 검색 인덱스에도 있고, 각각을 코드 여기저기서 따로 갱신합니다. 한 군데를 빼먹으면 셋이 어긋나고, 어느 쪽이 맞는지 아무도 모릅니다.

그래서 우리는 원본을 한 곳에 두고 나머지는 **거기서 파생되게** 만듭니다.

React 이전의 프론트엔드에는 그 원본이 없었습니다. **화면 자체가 원본이었습니다.**

## 문제 상황: 주문 목록 화면

요구사항은 소박합니다.

- 주문 목록을 보여준다
- 로딩 중에는 스피너
- 실패하면 에러 메시지
- 결과가 없으면 "주문이 없습니다"
- 새로고침 버튼

React 없이 쓰면 이렇게 됩니다.

```js
// ❌ 명령형: "무엇을 할지"를 순서대로 나열한다
async function loadOrders() {
  document.getElementById('spinner').style.display = 'block';
  document.getElementById('error').style.display = 'none';
  document.getElementById('empty').style.display = 'none';
  document.getElementById('refresh').disabled = true;

  try {
    const orders = await fetchOrders();
    const tbody = document.getElementById('tbody');
    tbody.replaceChildren();                           // 기존 것 지우고
    for (const o of orders) {                          // 다시 그리고
      const tr = document.createElement('tr');
      for (const value of [o.id, o.status]) {
        const td = document.createElement('td');
        td.textContent = value;                        // textContent — 문자열을 HTML로 해석하지 않는다
        tr.appendChild(td);
      }
      tbody.appendChild(tr);
    }
    if (orders.length === 0) {
      document.getElementById('empty').style.display = 'block';
    }
  } catch (e) {
    document.getElementById('error').textContent = e.message;
    document.getElementById('error').style.display = 'block';
  } finally {
    document.getElementById('spinner').style.display = 'none';
    document.getElementById('refresh').disabled = false;
  }
}
```

동작은 합니다. 문제는 **이제부터** 입니다.

## 무너지는 네 지점

### 1. 상태가 화면 여기저기에 흩어진다

"지금 로딩 중인가?"라는 질문에 답하려면 어디를 봐야 할까요? `spinner`의 `display` 속성입니다. **화면이 곧 데이터베이스가 된 것입니다.** 그리고 이 데이터베이스에는 스키마도 제약조건도 없습니다.

`spinner`는 보이는데 `refresh` 버튼은 활성화된 상태 — 이런 **불가능해야 할 조합** 이 얼마든지 만들어집니다. 코드 어딘가에서 한 줄을 빼먹기만 하면 됩니다.

### 2. 요구사항 하나가 늘면 수정 지점이 여러 곳으로 번진다

"로딩 중에는 필터 드롭다운도 비활성화" 한 줄이 추가되면, `loadOrders`의 시작과 `finally` 양쪽을 고쳐야 합니다. 그리고 다른 곳에서도 로딩을 시작한다면 거기도 고쳐야 합니다.

상태 하나가 늘 때마다 **고쳐야 할 곳이 곱셈으로 늘어납니다.** 이게 명령형 UI의 본질적 한계입니다.

### 3. 순서 의존성이 생긴다

위 코드는 `loadOrders`가 두 번 겹쳐 호출되면 깨집니다. 첫 번째 응답이 두 번째 응답보다 늦게 도착하면 **옛 데이터가 새 데이터를 덮어씁니다.** 백엔드로 치면 락 없이 read-modify-write를 하는 것과 같습니다.

### 4. 어디까지 그렸는지 아무도 모른다

위 코드가 `tbody.replaceChildren()`으로 **통째로 지우고 다시 그리는** 이유가 있습니다. **"무엇이 바뀌었는지"를 계산하는 게 너무 어렵기 때문입니다.** 그래서 다 지우고 다시 그리는데, 그러면 사용자가 입력하던 값이나 스크롤 위치, 포커스가 날아갑니다.

## 이 문제의 정체

네 가지는 사실 하나의 문제입니다.

> **화면(DOM)이 상태의 저장소 역할까지 하고 있다.**

백엔드에서 이런 코드를 본다면 당신은 바로 지적할 겁니다. "상태를 뷰에서 빼서 한 곳으로 모으고, 뷰는 그 상태로부터 계산되게 하세요."

**React가 한 일이 정확히 그것입니다.**

## React의 답

아래는 개념을 보여주기 위한 코드입니다. `useState`와 `Result` 타입은 각각 10장과 입문 가이드 11장에서 다룹니다.

```tsx
type Result =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; orders: Order[] }
  | { status: 'error'; message: string };

// ✅ 선언형: "어떤 상태일 때 화면이 어떻게 생겼는지"를 선언한다
function OrderList() {
  const [state, setState] = useState<Result>({ status: 'idle' });

  async function load() {
    setState({ status: 'loading' });
    try {
      const orders = await fetchOrders();
      setState({ status: 'success', orders });
    } catch (e) {
      setState({ status: 'error', message: (e as Error).message });
    }
  }

  if (state.status === 'loading') return <Spinner />;
  if (state.status === 'error')   return <Error msg={state.message} />;
  if (state.status === 'success' && state.orders.length === 0) return <Empty />;
  return <Table rows={state.status === 'success' ? state.orders : []} onRefresh={load} />;
}
```

바뀐 것을 하나씩 보면.

| 명령형 | React |
| --- | --- |
| 상태가 DOM 속성에 흩어짐 | 상태가 `state` 한 곳에 |
| 불가능한 조합이 만들어짐 | 판별 유니온이라 **타입이 막아줌** (입문 11장) |
| 요구사항 하나 = 수정 여러 곳 | 상태 정의 한 곳만 고침 |
| 무엇이 바뀌었는지 직접 계산 | **React가 계산** (6장) |
| 이 예제처럼 다 지우고 다시 그리기 쉬움 | 달라진 곳을 찾아 최소한만 손댐 |

**핵심은 "화면을 고치는 코드"를 아무도 안 쓴다는 것입니다.** 상태를 바꾸면 화면은 따라옵니다. 어떻게 따라오는지가 이 문서 Part 2의 내용입니다.

## 그런데 명령형이 사라진 건 아니다

React가 명령형을 없앤 게 아니라 **경계 안으로 밀어 넣은 것** 입니다. 실제 DOM 조작은 여전히 일어나야 하고, React 내부가 그걸 대신 합니다.

그래서 React가 대신해 줄 수 없는 일 — 포커스 이동, 스크롤 위치 조작, 외부 라이브러리 연결 — 에서는 **여전히 명령형으로 내려가야 합니다.** 그때 쓰는 도구가 `useRef`(13장)와 `useEffect`(11장)입니다.

> 이걸 미리 말해두는 이유가 있습니다. React를 배우는 사람들이 자주 하는 실수가 **"명령형은 나쁘다"고 배워서 `useRef`와 `useEffect`를 죄책감 없이 못 쓰는 것** 입니다. 나쁜 게 아니라 **경계에서만 쓰는 것** 입니다.

## 흔한 오해

**"React는 DOM 조작을 편하게 해주는 라이브러리다"**
반대에 가깝습니다. React는 DOM을 직접 만지지 **않아도 되게** 만드는 라이브러리입니다. 마음먹으면 만질 수는 있습니다. 다만 그러면 같은 DOM을 관리하는 주인이 둘이 되어 예측이 무너집니다. 그 절제가 이득의 원천입니다.

**"jQuery 시절 코드는 무조건 나쁘다"**
화면 요소 몇 개짜리 페이지에서는 명령형이 더 간결합니다. 문제는 상태가 늘어날 때 비용이 곱셈으로 커진다는 것입니다.

**"React를 쓰면 성능이 좋아진다"**
성능이 아니라 **예측 가능성**을 얻습니다. 성능은 오히려 손해 보는 경우도 있습니다. 4장에서 정면으로 다룹니다.

## 안티패턴 #1: DOM을 직접 만져서 상태를 표현하기

```tsx
// ❌ React 안에서 명령형으로 되돌아가기
function Toggle() {
  const [open, setOpen] = useState(false);
  const show = () => {
    document.getElementById('panel')!.style.display = 'block';  // React가 모르는 변경
  };
  ...
}
```

React도 그 요소를 관리하고 있기 때문에, 다음 렌더에서 이 변경이 지워질 수 있습니다. 근본 문제는 **같은 DOM을 두 주인이 관리하게 된 것** 입니다. 어느 쪽이 이길지 예측할 수 없습니다. **화면에 보이는 것은 항상 상태로부터 나와야 합니다.**

## 핵심 3가지

- **명령형 UI의 병은 하나입니다.** 화면이 상태 저장소 역할을 겸하는 것. 나머지 증상은 전부 여기서 파생됩니다.
- **React는 그 역할을 뺏습니다.** 상태를 한 곳으로 모으고, 개발자가 "화면을 고치는 코드"를 쓰지 않게 합니다.
- **명령형이 사라진 건 아닙니다.** 경계로 밀려났을 뿐입니다. 포커스·스크롤·외부 라이브러리에서는 여전히 내려갑니다.

## 스스로 답해보기

1. 명령형 UI에서 "요구사항 하나가 늘면 수정 지점이 곱셈으로 는다"는 말을 로딩 상태 예시로 설명해 보세요.
2. React가 명령형을 "없앤" 것이 아니라 "경계로 밀어냈다"는 말은 실무에서 무엇을 뜻하는가?

---

**다음 장 예고** — 그 계약을 한 줄로 쓰면 `UI = f(state)` 입니다. 함수라는 표현이 은유가 아니라 문자 그대로라는 것, 그리고 그 대가로 무엇을 포기해야 하는지 봅니다.
