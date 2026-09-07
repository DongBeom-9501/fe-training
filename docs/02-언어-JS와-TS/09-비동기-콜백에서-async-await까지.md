# 9. 비동기 — 콜백에서 async/await까지

> **이 장의 질문**: 스레드가 하나뿐인데 어떻게 "기다리는" 코드를 읽기 좋게 쓰는가?

6장에서 배운 이벤트 루프와 8장에서 배운 함수 값이 여기서 만납니다.

## BE로 치면 이런 것

백엔드에서 DB를 호출할 때 당신은 그냥 씁니다.

```kotlin
val orders = orderRepository.findAll()   // 여기서 스레드가 멈춰서 기다린다
println(orders.size)
```

스레드가 블로킹되지만 괜찮습니다. 스레드가 200개 있으니까요.

브라우저에서 화면을 다루는 스레드는 하나뿐입니다. 여기서 멈추면 화면 전체가 멈춥니다. 그래서 브라우저의 **네트워크 API에는 블로킹 방식이 없습니다.** "기다린다" 대신 **"끝나면 이걸 해줘"** 를 등록하는 방식으로 갑니다.

> 물론 JS에서 아무것도 멈추지 않는다는 뜻은 아닙니다. 무거운 반복문은 그대로 메인 스레드를 붙잡고, `alert()` 같은 몇몇 API도 멈춥니다. **막히지 않는 것은 네트워크와 타이머 쪽** 입니다.

## 1세대: 콜백

```js
// "요청이 끝나면 이 함수를 불러줘"
fetchOrders((orders) => {
  console.log(orders.length);
});
console.log('먼저 찍힘');
```

동작은 합니다. 문제는 **연달아 해야 할 때** 생깁니다.

```js
// 주문 조회 → 고객 조회 → 등급 조회
fetchOrder(id, (order) => {
  fetchCustomer(order.customerId, (customer) => {
    fetchGrade(customer.gradeId, (grade) => {
      render(order, customer, grade);
    }, onError);
  }, onError);
}, onError);
```

들여쓰기가 오른쪽으로 계속 밀립니다. **콜백 지옥(callback hell)** 이라고 불렀습니다. 더 나쁜 건 에러 처리입니다. 각 단계마다 따로 처리해야 하고, `try/catch`로는 잡히지 않습니다 (콜백이 실행될 때는 이미 `try` 블록을 빠져나온 뒤니까요).

## 2세대: Promise

**Promise는 "아직 없는 값"을 나타내는 객체입니다.** Java의 `CompletableFuture`와 같은 자리입니다.

```js
const promise = fetchOrder(id);   // 즉시 반환된다. 값은 아직 없다.
```

Promise는 세 상태 중 하나입니다.

```
pending (대기)  ──┬──> fulfilled (성공)  → .then(값)
                  └──> rejected  (실패)  → .catch(에러)
```

이제 체인으로 이어집니다.

```js
fetchOrder(id)
  .then(order => fetchCustomer(order.customerId))
  .then(customer => fetchGrade(customer.gradeId))
  .then(grade => render(grade))
  .catch(err => showError(err));      // ← 어느 단계에서 실패해도 여기로
```

중첩이 사라지고, **에러 처리가 한 곳으로 모였습니다.** `.then` 안에서 또 Promise를 반환하면 그게 풀려서 다음 `.then`으로 넘어갑니다.

## 3세대: async / await

Promise도 여전히 콜백을 씁니다. `async/await`은 **Promise를 동기 코드처럼 보이게** 만들어 줍니다.

```js
async function load(id) {
  try {
    const order = await fetchOrder(id);
    const customer = await fetchCustomer(order.customerId);
    const grade = await fetchGrade(customer.gradeId);
    render(order, customer, grade);
  } catch (err) {
    showError(err);
  }
}
```

**이게 실무에서 쓰는 형태입니다.** 위아래로 읽히고, `try/catch`가 정상 동작하고, 변수 스코프도 자연스럽습니다.

두 가지 규칙만 기억하면 됩니다.

1. `await`은 `async` 함수 안에서만 쓸 수 있다 (예외: ESM 모듈의 최상위에서는 `async` 없이도 됩니다)
2. `async` 함수는 **항상 Promise를 반환한다** (`return 1`이라고 써도 `Promise<number>`)

## Kotlin 코루틴과 나란히

```kotlin
suspend fun load(id: Long) {
  try {
    val order = api.getOrder(id)
    val customer = api.getCustomer(order.customerId)
    render(order, customer)
  } catch (e: Exception) { showError(e) }
}
```
```js
async function load(id) {
  try {
    const order = await api.getOrder(id);
    const customer = await api.getCustomer(order.customerId);
    render(order, customer);
  } catch (e) { showError(e); }
}
```

**모양은 거의 같습니다. 하지만 같은 개념은 아닙니다.** 이 비유는 코드를 읽기 위한 앵커일 뿐이고, 실행 모델은 꽤 다릅니다.

| | Kotlin `suspend` | JS `async` |
| --- | --- | --- |
| 호출하면 | 값이 나올 때까지 중단 | **즉시 Promise를 반환** |
| 돌아올 스레드 | 디스패처가 정함 (여러 개 가능) | 항상 그 하나 |
| 취소 | 구조적 동시성으로 전파됨 | 없음. 직접 `AbortController`를 써야 함 |
| 스코프 | 부모-자식 관계가 있음 | 없음. 각자 떠다님 |

> **가져갈 것**: `await`이 보이면 "여기서 기다리는구나"라고 읽으면 됩니다. 다만 **코루틴의 취소나 스코프 같은 것은 따라오지 않습니다.**

## 병렬로 기다리기 — 실무에서 가장 흔한 실수

```js
// 나쁜 예: 순차 실행. 각 200ms면 총 600ms
const orders = await fetchOrders();
const users = await fetchUsers();
const stats = await fetchStats();
```

세 요청이 **서로 의존하지 않는데도** 줄 서서 기다립니다. `await`을 쓰는 순간 거기서 다음 줄로 안 넘어가기 때문입니다.

```js
// 좋은 예: 병렬 실행. 총 200ms
const [orders, users, stats] = await Promise.all([
  fetchOrders(),
  fetchUsers(),
  fetchStats(),
]);
```

핵심은 **요청이 시작되는 시점이 `fetchOrders()`를 호출하는 순간** 이라는 것입니다. `await`은 시작 신호가 아니라 "결과를 여기서 받겠다"는 표시일 뿐입니다. 그래서 세 개를 먼저 다 호출해 놓고 나중에 한꺼번에 기다리면 병렬이 됩니다. `Promise.all`이 병렬로 만들어 주는 게 아니라, **이미 시작된 것들을 모아서 기다려 줄 뿐** 입니다.

관련 도구가 몇 개 더 있습니다.

| | 동작 |
| --- | --- |
| `Promise.all([...])` | 전부 성공하면 결과 배열. **하나라도 실패하면 즉시 실패** |
| `Promise.allSettled([...])` | 전부 끝날 때까지 기다리고 성공/실패를 각각 담아 반환 |
| `Promise.race([...])` | 가장 먼저 끝난 것 하나 |
| `Promise.any([...])` | 가장 먼저 **성공한** 것 하나 |

부분 실패를 허용해야 하는 대시보드 같은 화면에서는 `allSettled`가 맞습니다.

## fetch — 브라우저의 HTTP 클라이언트

```js
// 주문 목록을 가져온다
async function fetchOrders() {
  const res = await fetch('/api/orders');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);   // ← 이 줄이 중요
  return res.json();
}
```

**`fetch`의 가장 큰 함정**: 404나 500을 받아도 **에러를 던지지 않습니다.** "응답을 받았다"는 사실 자체는 성공이라고 보기 때문입니다. reject되는 것은 응답을 아예 못 받은 경우 — 네트워크 단절, 요청 취소, CORS 차단 등입니다.

그래서 `res.ok`(상태 코드 200~299)를 직접 확인해야 합니다. 실무에서는 이 처리를 한 곳에 모아 두고 씁니다 — 30장의 API 레이어가 그 이야기입니다.

## 취소

3초 걸리는 요청을 보냈는데 사용자가 다른 페이지로 가버렸다면, 그 응답은 이제 쓸모없습니다. 취소는 `AbortController`로 합니다.

```js
const controller = new AbortController();
fetch('/api/orders', { signal: controller.signal });

controller.abort();   // 취소
```

직접 쓸 일은 드뭅니다. TanStack Query 같은 라이브러리가 관리해 주거든요(27장). 다만 **라이브러리가 건네주는 `signal`을 실제 요청에 넘겨야** 취소가 전달됩니다. 그냥 쓰면 화면만 사라지고 요청은 계속 날아갑니다. 다만 **"화면이 사라졌는데 응답이 늦게 도착하는 상황"이 실재한다** 는 건 알아 두세요. 2장의 "오래 사는 프로세스"가 만드는 문제 중 하나입니다.

## 흔한 오해

**"`await`을 쓰면 화면이 멈춘다"**
아닙니다. `await`은 이벤트 루프에 제어권을 돌려줍니다. 화면을 멈추는 건 `await` 없는 무거운 동기 계산입니다.

**"`async`를 붙이면 병렬로 돈다"**
아닙니다. `await`을 연달아 쓰면 순차입니다. 병렬은 `Promise.all`로 명시해야 합니다.

**"fetch가 실패하면 catch로 간다"**
HTTP 에러 상태 코드는 catch로 가지 않습니다. `res.ok`를 직접 확인해야 합니다.

**"`.then`은 옛날 방식이라 안 써도 된다"**
`async/await`을 기본으로 쓰되, `Promise.all`이나 라이브러리 API에서는 여전히 만납니다. 읽을 줄은 알아야 합니다.

## 핵심 3가지

- **네트워크는 블로킹으로 기다리지 않습니다.** "끝나면 이걸 해줘"를 등록하는 방식이고, 콜백 → Promise → `async/await` 순으로 읽기 좋아졌습니다.
- **`await`을 연달아 쓰면 순차입니다.** 서로 의존하지 않는 요청은 `Promise.all`로 묶으세요.
- **`fetch`는 404·500에 예외를 던지지 않습니다.** `res.ok`를 직접 확인해야 하고, 이 처리는 API 레이어 한 곳에 모읍니다.

## 스스로 답해보기

1. 서로 무관한 API 세 개를 `await`으로 연달아 호출하면 무엇이 문제이고 어떻게 고치는가?
2. `fetch`로 받은 응답이 500인데 `catch` 블록이 실행되지 않았다. 왜인가?

---

**다음 장 예고** — 코드를 파일로 나누고 남의 코드를 가져다 쓰는 법. `import`가 어떻게 동작하는지, 그리고 `package.json`이라는 파일에 무엇이 적혀 있는지 읽어봅니다.
