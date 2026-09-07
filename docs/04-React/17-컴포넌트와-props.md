# 17. 컴포넌트와 props — 단방향 데이터 흐름

> **이 장의 질문**: 컴포넌트가 여러 개일 때 데이터는 어느 방향으로 흐르는가?

## BE로 치면 이런 것

당신이 레이어드 아키텍처를 지킬 때 지키는 규칙이 있습니다. **의존성은 한 방향으로만 흐른다.** 컨트롤러는 서비스를 알지만 서비스는 컨트롤러를 모릅니다. 이 규칙을 어기면 순환 참조가 생기고, 어디서 무엇이 바뀌는지 추적이 불가능해집니다.

React는 화면에 대해 같은 규칙을 강제합니다.

> **데이터는 위에서 아래로만 흐른다.**

## props — 부모가 자식에게 주는 것

```tsx
// 부모
function OrderPage() {
  const orders = [{ id: 'ORD-1', status: 'PAID', amount: 15000 }];
  return <OrderTable orders={orders} title="최근 주문" />;
}

// 자식
function OrderTable({ orders, title }: { orders: Order[]; title: string }) {
  return (
    <section>
      <h2>{title}</h2>
      <table>
        <tbody>
          {orders.map(o => <OrderRow key={o.id} order={o} />)}
        </tbody>
      </table>
    </section>
  );
}
```

props는 **함수의 파라미터** 입니다. 그 이상도 이하도 아닙니다. 컴포넌트가 함수라고 했으니(16장) 당연한 이야기죠.

뒤에 나오는 `React.ReactNode`는 **React가 화면에 그릴 수 있는 것** 을 가리키는 타입입니다. JSX뿐 아니라 문자열, 숫자, 배열, `null`도 여기 들어갑니다.

### props는 읽기 전용으로 다룬다

**이것이 규칙의 핵심입니다.** 문법이 막아주지는 않습니다. JavaScript는 얼마든지 고치게 놔둡니다. 다만 고치는 순간 아래 일이 벌어져서, 팀이 지키기로 한 약속입니다.

```tsx
// ❌ 절대 하면 안 됨
function OrderTable({ orders }: { orders: Order[] }) {
  orders.sort((a, b) => b.amount - a.amount);   // 부모의 배열을 파괴한다
  orders.push(newOrder);                        // 부모가 모르는 변경
  return ...;
}
```

```tsx
// ✅ 복사해서 쓴다
function OrderTable({ orders }: { orders: Order[] }) {
  const sorted = [...orders].sort((a, b) => b.amount - a.amount);
  return ...;
}
```

8장에서 배운 `sort`가 원본을 바꾼다는 사실이 여기서 실전 버그가 됩니다. 자식이 props를 수정하면 **부모는 그 사실을 모른 채 데이터가 바뀝니다.** 화면이 언제 갱신될지 아무도 예측할 수 없게 됩니다.

Java로 치면 메서드에 넘긴 컬렉션을 그 안에서 정렬해 버리는 것과 같습니다. 백엔드에서도 하면 안 되는 일이죠.

## 그럼 자식은 부모에게 어떻게 말하나

데이터가 아래로만 흐른다면, 사용자가 자식에서 버튼을 눌렀을 때 어떻게 알릴까요?

**부모가 함수를 내려주고, 자식은 그 함수를 호출합니다.** 8장의 콜백이 여기서 쓰입니다.

```tsx
// 부모가 "무엇을 할지"를 정하고, 함수를 내려준다
function OrderPage() {
  const [selectedId, setSelectedId] = useState<string | null>(null);

  return (
    <OrderTable
      orders={orders}
      onSelect={(id) => setSelectedId(id)}   {/* 함수를 내려준다 */}
    />
  );
}

// 자식은 언제 호출할지만 안다. 무엇을 하는지는 모른다.
function OrderTable({ orders, onSelect }: Props) {
  return (
    <table>
      <tbody>
        {orders.map(o => (
          <tr key={o.id}>
            {/* 행 전체가 아니라 버튼에 건다 — 키보드로도 눌러야 하니까 (12장) */}
            <td><button onClick={() => onSelect(o.id)}>{o.id}</button></td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```
        데이터(props)  ↓         ↑  이벤트(콜백 호출)
   부모  ─────────────────────────  자식
```

**흐름은 여전히 한 방향입니다.** 자식이 부모를 직접 바꾸는 게 아니라, 부모가 미리 준 함수를 부를 뿐입니다. 최종 결정권은 항상 부모에게 있습니다.

이건 전략 패턴을 인터페이스 없이 하는 것과 같습니다. Java에서 `Consumer<String>`을 넘기던 자리에 그냥 함수를 넘깁니다.

> **명명 관례**: props로 받는 콜백은 `onXxx`, 그 구현 함수는 `handleXxx`로 짓는 것이 널리 쓰입니다.

## 상태 끌어올리기

두 형제 컴포넌트가 같은 데이터를 봐야 한다면? 형제끼리는 직접 대화할 수 없습니다.

```
      ❌ 형제끼리 직접        ✅ 공통 부모로 올린다

   ┌──────┐  ┌──────┐          ┌───────────┐
   │Filter│──│Table │          │  부모      │ ← state는 여기
   └──────┘  └──────┘          └─┬───────┬─┘
                                 ↓       ↓
                            ┌──────┐ ┌──────┐
                            │Filter│ │Table │
                            └──────┘ └──────┘
```

```tsx
function OrderPage() {
  const [keyword, setKeyword] = useState('');          // 공통 부모가 소유
  const filtered = orders.filter(o => o.id.includes(keyword));

  return (
    <>
      <SearchBar value={keyword} onChange={setKeyword} />
      <OrderTable orders={filtered} />
    </>
  );
}
```

**"상태 끌어올리기(lifting state up)"** 라고 부릅니다. 두 컴포넌트가 같은 상태를 필요로 하면, 그 상태는 **둘의 가장 가까운 공통 부모** 가 가져야 합니다.

> 반대로, 한 컴포넌트만 쓰는 상태를 부모에 두면 **불필요한 리렌더** 가 생깁니다. 상태는 필요한 만큼만 위로 올리세요. 26장에서 이 균형을 다시 다룹니다.

## children — 컴포넌트를 감싸기

`props.children`은 **여는 태그와 닫는 태그 사이의 내용** 을 받는 특별한 prop입니다.

```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <section className="rounded border p-4">
      <h3 className="font-bold">{title}</h3>
      <div>{children}</div>
    </section>
  );
}

// 사용하는 쪽
<Card title="주문 요약">
  <OrderTable orders={orders} />
  <TotalRow amount={total} />
</Card>
```

**이게 왜 중요하냐면**, `Card`가 안에 뭐가 들어올지 몰라도 되기 때문입니다. 껍데기와 내용이 완전히 분리됩니다.

props를 여러 단계 내려보내야 하는 상황(prop drilling)의 상당수가 `children`으로 해결됩니다. 20장에서 자세히 다룹니다.

## props 타입 정의

TypeScript로는 이렇게 씁니다.

```tsx
type OrderTableProps = {
  orders: Order[];
  title?: string;                       // 선택적
  onSelect: (id: string) => void;       // 콜백
  children?: React.ReactNode;           // JSX를 받는 자리
};

function OrderTable({ orders, title = '주문 목록', onSelect }: OrderTableProps) {
  ...
}
```

`title = '주문 목록'` 처럼 구조 분해에서 기본값을 줄 수 있습니다(8장).

## 컴포넌트 합성 vs 상속

React 공식 문서는 **컴포넌트 상속 대신 합성** 을 권장합니다. (예전 방식인 클래스 컴포넌트에서는 상속이 문법적으로 가능하긴 합니다. 다만 실무에서 쓰지 않습니다.)

Java에서 `AbstractBaseController`를 만들어 상속하던 패턴 대신, React에서는 **합성** 을 씁니다.

```tsx
// ❌ 상속으로 특수화 (React에는 이런 게 없다)

// ✅ 합성으로 특수화
function DangerButton({ children, ...rest }: ButtonProps) {
  return <Button variant="danger" {...rest}>{children}</Button>;
}
```

`{...rest}`는 전개 구문(8장)으로 나머지 props를 그대로 넘기는 관용구입니다.

## 흔한 오해

**"props를 자식에서 바꿔도 되지 않나?"**
문법적으로는 됩니다. 그래서 더 위험합니다. 부모가 모르는 변경이 생기고, 화면 갱신이 예측 불가능해집니다.

**"자식이 부모의 state를 직접 바꾸면 편한데"**
부모가 setter를 props로 내려주면 자식도 갱신을 **요청** 할 수 있습니다. 막혀 있는 게 아닙니다. 핵심은 **소유권** 입니다. 그 상태를 언제 어떻게 바꿀지 정하는 쪽은 언제나 부모입니다.

**"props가 많아지면 객체 하나로 묶어 넘기자"**
때에 따라 다릅니다. props가 10개를 넘어가면 대개 **컴포넌트를 잘못 나눈 신호** 입니다. 20장에서 다룹니다.

**"모든 상태는 최상위에 두는 게 안전하다"**
아닙니다. 불필요한 리렌더가 늘고, 컴포넌트 재사용이 어려워집니다.

## 핵심 3가지

- **데이터는 위에서 아래로만 흐릅니다.** props는 읽기 전용입니다. 자식이 배열을 정렬하거나 객체를 고치면 안 됩니다.
- **자식은 콜백으로 알립니다.** 부모가 함수를 내려주고 자식이 부릅니다. 그래서 흐름은 여전히 단방향입니다.
- **형제가 공유하는 상태는 위로 올립니다.** 가장 가까운 공통 부모가 소유합니다.

## 스스로 답해보기

1. 자식 컴포넌트가 `props.orders.sort()`를 호출하면 무엇이 문제인가?
2. 검색창과 결과 테이블이 같은 키워드를 공유해야 한다. 상태를 어디에 두어야 하며 왜인가?

---

**다음 장 예고** — 이제 state입니다. `setState`를 호출하면 정확히 무슨 일이 일어나는지, 그리고 React가 화면의 어느 부분을 다시 그릴지 어떻게 결정하는지 봅니다.
