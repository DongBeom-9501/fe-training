# 11. TypeScript — 구조적 타이핑, 제네릭, 내로잉

> **이 장의 질문**: Java의 타입 시스템에 익숙한 사람이 TypeScript에서 다시 배워야 하는 것은 무엇인가?

## BE로 치면 이런 것

TypeScript는 **JavaScript + 타입** 입니다. 그리고 중요한 사실 하나.

> **TypeScript의 타입 정보는 컴파일하면 사라집니다.** 브라우저가 실행하는 건 타입이 지워진 순수 JavaScript입니다.

Java의 제네릭 소거(type erasure)를 아신다면 그 극단적인 버전이라고 생각하시면 됩니다. Java는 클래스와 필드 타입이 런타임에 남지만, TypeScript의 타입 표기·`type`·`interface`는 **흔적도 남지 않습니다.** 컴파일 타임에만 존재하는 주석에 가깝습니다.

여기서 실무 규칙이 하나 나옵니다.

> **서버 응답을 타입으로 선언했다고 해서 실제 그 모양이라는 보장은 없습니다.** 타입은 약속이지 검증이 아닙니다. 정말 검증이 필요한 경계(API 응답)에서는 런타임 검증 라이브러리(Zod 등)를 씁니다. 30장에서 다룹니다.

## 결정적 차이: 구조적 타이핑

Java는 **이름으로** 타입을 판단합니다(명목적 타이핑). `implements Printable`이라고 써야 `Printable`입니다.

TypeScript는 **모양으로** 판단합니다(구조적 타이핑).

```ts
type User = { id: string; name: string };

function greet(u: User) { console.log(u.name); }

// User라고 선언한 적 없지만, 모양이 맞으므로 통과한다
const someone = { id: '1', name: '김', role: 'admin' };
greet(someone);   // ✅ OK
```

Kotlin이었다면 `User` 타입이 아니라며 거부했을 코드입니다. TypeScript는 **필요한 필드가 다 있으면 통과** 시킵니다. 덕 타이핑에 컴파일 타임 검사를 붙인 셈입니다.

이 성질 덕분에 인터페이스를 미리 설계하지 않아도 되고, 남의 라이브러리 타입에 내 객체를 맞추기 쉽습니다. 대신 **"의미가 다른데 모양이 같은 것"을 구분해 주지 않습니다.**

```ts
type UserId = string;
type OrderId = string;
// 이 둘은 TypeScript에게 완전히 같은 타입이다. 섞어 써도 안 잡힌다.
```

## 기본 타입

```ts
let name: string;
let count: number;          // int/long/double 구분 없음 (7장)
let done: boolean;
let ids: string[];          // 또는 Array<string>
let pair: [string, number]; // 튜플

// 리터럴 타입 — Java의 enum 자리를 대신하는 경우가 많다
type Status = 'PENDING' | 'PAID' | 'SHIPPED';

// 객체
type Order = {
  id: string;
  status: Status;
  amount: number;
  memo?: string;          // ? = 있어도 되고 없어도 됨 (undefined 허용)
  readonly createdAt: string;
};
```

`type`과 `interface` 둘 다 있는데 대부분의 경우 같습니다. **`type`을 기본으로 쓰고, 라이브러리 타입을 확장해야 할 때만 `interface`** 정도로 정하고 넘어가세요. 팀 컨벤션의 문제이지 옳고 그름의 문제가 아닙니다.

## null 안전성

`tsconfig.json`에 `"strict": true`가 켜져 있으면(반드시 켜세요) Kotlin과 비슷한 수준의 null 검사를 얻습니다. 다만 `any`나 `as`로 언제든 빠져나갈 수 있어서 Kotlin만큼 촘촘하지는 않습니다.

```ts
function findUser(id: string): User | undefined { ... }

const user = findUser('1');
console.log(user.name);       // ❌ 컴파일 에러: undefined일 수 있음
console.log(user?.name);      // ✅
```

`User | undefined`가 Kotlin의 `User?`에 해당합니다.

## 내로잉 — 타입을 좁혀 나가기

TypeScript에서 가장 자주 쓰게 될 사고 방식입니다. **컴파일러가 코드 흐름을 따라가면서 타입을 좁힙니다.**

```ts
function format(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase();   // 여기서 value는 string으로 좁혀졌다
  }
  return value.toFixed(2);        // 여기서는 number
}
```

```ts
// null 체크도 내로잉이다
function displayName(user: User | undefined): string {
  if (!user) return '(없음)';
  return user.name;        // 여기 아래로는 user가 확실히 있다
}
```

**판별 유니온(discriminated union)** 은 이 사고방식의 정점이고, 프론트엔드에서 대단히 유용합니다.

```ts
type Result =
  | { status: 'loading' }
  | { status: 'success'; data: Order[] }
  | { status: 'error'; message: string };

function describe(r: Result): string {
  switch (r.status) {
    case 'loading': return '불러오는 중';
    case 'success': return `${r.data.length}건`;   // data가 있는 게 보장됨
    case 'error':   return r.message;              // message가 있는 게 보장됨
  }
}
```

Kotlin의 sealed class + `when`과 정확히 같은 패턴입니다. `status`를 확인하는 것만으로 나머지 필드가 있는지 없는지가 정해집니다.

> **왜 중요한가**: `loading`인 상태에는 `data` 필드가 아예 없기 때문에, "로딩 중인데 데이터를 읽으려 하는" 코드는 **컴파일이 안 됩니다.** 나중에 화면 상태를 다룰 때(16장) 이 패턴이 그대로 쓰입니다.

## 제네릭

Java와 거의 같습니다.

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}

const n = first([1, 2, 3]);        // number | undefined 로 추론됨
```

제약도 있습니다.

```ts
function byId<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}
```

`<T extends { id: string }>` 는 Kotlin의 `<T : Identifiable>` 자리인데, **구조적 타이핑이라 인터페이스를 선언할 필요가 없습니다.** "id라는 string 필드가 있기만 하면 된다"고 바로 쓸 수 있습니다.

## 실무에서 자주 쓰는 유틸리티 타입

기존 타입에서 새 타입을 만들어 냅니다. 중복 선언을 줄여 줍니다.

```ts
type Order = { id: string; status: Status; amount: number; memo?: string };

Partial<Order>              // 모든 필드가 선택적 — 수정 폼에 유용
Required<Order>             // 모든 필드가 필수
Pick<Order, 'id' | 'status'>  // 일부만 골라내기
Omit<Order, 'id'>           // 일부 빼기 — 생성 요청 DTO에 유용
Readonly<Order>             // 전부 읽기 전용
Record<string, Order>       // 맵 타입
```

```ts
// 실무에서 이렇게 쓴다
type CreateOrderRequest = Omit<Order, 'id' | 'createdAt'>;
type UpdateOrderRequest = Partial<CreateOrderRequest>;
```

DTO를 세 개 따로 만드는 대신 하나에서 파생시키는 것입니다. 필드가 추가되면 세 개가 같이 따라옵니다.

## `any`를 쓰지 않는 이유

```ts
const data: any = await res.json();
data.orders.map(...)        // 오타가 나도, 필드가 없어도 아무도 안 잡아준다
```

`any`는 **그 값에 대한 타입 검사를 전부 끕니다.** 그리고 전염됩니다 — `any`에서 나온 값도 `any`가 되어 검사가 계속 꺼진 채로 코드 여기저기로 퍼집니다.

정말 모르는 타입이라면 `unknown`을 쓰세요.

```ts
const data: unknown = await res.json();
data.orders;                      // ❌ 에러 — 좋은 신호다
if (isOrderList(data)) { ... }    // ✅ 확인한 뒤에 쓰게 강제한다
```

`any`는 "검사하지 마", `unknown`은 "검사하기 전엔 못 쓴다"입니다.

> **실무 규칙**: 린터에서 `no-explicit-any`를 켜 두고, 정말 필요한 곳에서만 주석과 함께 예외 처리합니다.

## `as`도 조심해야 한다

```ts
const user = data as User;   // "내가 책임질게" — 컴파일러는 믿어줍니다
```

타입 단언(`as`)은 검사가 아니라 **주장** 입니다. 틀리면 런타임에 터집니다. Java의 무검사 캐스팅과 같습니다. API 경계에서는 `as` 대신 런타임 검증을 쓰세요.

## 흔한 오해

**"TypeScript를 쓰면 런타임 에러가 없다"**
컴파일 타임 실수만 줄어듭니다. 서버 응답이 타입과 다르면 그대로 터집니다.

**"타입을 다 직접 써야 한다"**
대부분 추론됩니다. **변수에는 타입을 안 쓰고, 함수의 파라미터와 반환 타입에만 쓰는 것** 이 좋은 균형입니다.

**"`interface`가 `type`보다 정식이다"**
아닙니다. 팀 컨벤션의 문제입니다.

**"제네릭을 복잡하게 쓸수록 좋다"**
읽을 수 없는 타입은 없는 것만 못합니다. 조건부 타입이나 매핑 타입은 정말 필요할 때만 쓰세요.

## 핵심 3가지

- **타입 정보는 컴파일 후 사라집니다.** 타입은 약속이지 검증이 아닙니다. API 경계에서는 런타임 검증이 따로 필요합니다.
- **이름이 아니라 모양으로 판단합니다.** 인터페이스를 미리 선언할 필요가 없습니다. 대신 의미가 다른데 모양이 같은 타입은 구분해 주지 않습니다.
- **판별 유니온으로 상태를 모델링하세요.** Kotlin sealed class + `when`과 같은 안전성을 얻습니다. 그리고 `any` 대신 `unknown`을 씁니다.

## 스스로 답해보기

1. 서버 응답을 `Order[]` 타입으로 선언했는데 실제로는 필드 하나가 빠져 있었다. TypeScript가 이걸 못 잡는 이유는?
2. `any`와 `unknown`의 차이를 한 문장으로 설명해 보세요.

---

**다음 장 예고** — Part 2가 끝났습니다. 언어를 알았으니 이제 눈에 보이는 것을 만들 차례입니다. `div`만 쓰면 왜 안 되는지부터 시작합니다.
