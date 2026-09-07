# 수정 필요 사항

## 1. 기술적 사실 오류·낡은 설명

- `docs/01-브라우저라는-런타임/03-URL을-입력하면-벌어지는-일.md:57-65` — CSS는 HTML 파서를 멈추게 하는 리소스가 아니라 **렌더 차단** 리소스이고, 기본 스크립트는 파서와 렌더를 막는다. `async` 스크립트도 내려받기가 끝나는 순간 HTML 파서를 중단하고 실행할 수 있으므로 “안 막는다”라고 표기하면 틀린다. 표를 `HTML 파싱`과 `첫 렌더`를 분리해 다시 써야 한다.
- `docs/01-브라우저라는-런타임/03-URL을-입력하면-벌어지는-일.md:89` — CORS 실패가 항상 “서버는 200으로 응답했고 브라우저만 막았다”는 것은 아니다. preflight `OPTIONS`가 실패해 본 요청 자체가 가지 않을 수 있고, 인증·WAF·네트워크 실패도 가능하다. “CORS는 브라우저가 응답을 JS에 노출할지를 강제하는 정책이며, Network 탭에서 preflight와 실제 요청을 함께 확인한다”로 고쳐야 한다.
- `docs/01-브라우저라는-런타임/05-픽셀이-그려지기까지.md:60,70,76-86` — 레이아웃이 항상 가장 비싸거나, `transform`/`opacity`가 항상 합성만 하고 가장 싸다는 보장은 없다. 해당 속성도 페인트가 필요할 수 있고, 레이어 생성·큰 GPU 텍스처·필터 때문에 합성이 병목이 될 수 있다. “대체로 레이아웃을 피하는 데 유리하나 DevTools로 실제 paint/composite를 확인한다”로 조건을 붙여야 한다.
- `docs/01-브라우저라는-런타임/06-싱글-스레드와-이벤트-루프.md:11-22` — “브라우저의 JavaScript에는 스레드가 하나뿐”, “DOM 때문에 언어 설계 단계에서 하나를 택했다”는 설명은 사실을 단순화하다 틀렸다. 메인 문서의 DOM 접근은 한 스레드에서 직렬화되지만 Web Worker·Shared Worker·Worklet은 별도 JS 실행 컨텍스트/이벤트 루프를 가진다. 공유 DOM이 단일 UI 실행 모델의 중요한 제약인 것은 맞지만 JavaScript 언어 자체가 단일 스레드인 이유로 단정하면 안 된다.
- `docs/01-브라우저라는-런타임/06-싱글-스레드와-이벤트-루프.md:53-69` — “마이크로태스크 큐 하나 → 태스크 큐 하나 → 매번 렌더” 모델은 정확하지 않다. HTML에는 task source가 여럿이고, 한 task 뒤 마이크로태스크 checkpoint를 비운 뒤에 **렌더 기회가 있을 수** 있을 뿐 매 반복 렌더가 보장되지 않는다. 이 단순 도식은 유지하되 “학습용 축약”을 표시하고 렌더 타이밍을 보장처럼 가르치지 말아야 한다.
- `docs/02-언어-JS와-TS/07-JS에서-놀라는-것들.md:87-96` — `number`는 IEEE 754 배정밀도 부동소수점 하나지만 JavaScript의 숫자 계열 전체가 하나는 아니다. `BigInt`는 별도 원시 타입이고 `number`와 혼합 연산도 되지 않는다. 제목과 문장을 “`number` 타입은 한 종류”로 한정해야 바로 아래의 `BigInt` 권고와 모순되지 않는다.
- `docs/02-언어-JS와-TS/07-JS에서-놀라는-것들.md:129-132` — ESM/엄격 모드에서 `const f = user.greet; f()`는 `this`가 `undefined`여서 `this.name`에서 `TypeError`가 난다. “undefined”라는 출력은 보장되지 않는다. 엄격 모드 예제로 오류를 보여 주거나 `console.log(this?.name)`으로 의도를 맞춰야 한다.
- `docs/02-언어-JS와-TS/08-함수가-일급-시민이라는-것.md:21-32` — 함수 선언과 화살표 함수는 “전부 같은 말”이 아니다. 함수 선언은 hoisting되고 동적 `this`/`arguments`를 가지며, 화살표 함수는 lexical `this`이고 생성자로 쓸 수 없다. 바로 앞 7장에서 `this` 차이를 가르친 내용과도 충돌한다. “반환값이 같은 단순 함수에서는 비슷하게 쓸 수 있다”로 범위를 좁혀야 한다.
- `docs/02-언어-JS와-TS/09-비동기-콜백에서-async-await까지.md:18` — “JavaScript에는 블로킹 호출 자체가 없다”는 절대 표현은 틀렸다. CPU 동기 작업은 이미 메인 스레드를 막고, `alert`/`confirm`, 일부 동기 Web API, Worker에서의 `Atomics.wait` 같은 예외도 있다. 브라우저 메인 스레드의 네트워크 I/O는 보통 비동기 API로 제공된다는 설명으로 한정해야 한다.
- `docs/02-언어-JS와-TS/09-비동기-콜백에서-async-await까지.md:93` — `await`은 `async` 함수 안에서만 쓴다는 규칙에는 모듈의 top-level `await` 예외가 있다. “일반 함수 안에서는 `async`가 필요하고, ESM 최상위에서는 가능하다”로 고쳐야 한다.
- `docs/02-언어-JS와-TS/09-비동기-콜백에서-async-await까지.md:117` — `suspend ↔ async`는 잘못된 대응이다. Kotlin의 `suspend`는 중단 가능한 함수의 타입/호출 규약이고, JS `async`는 즉시 Promise를 반환하는 함수 표지다. JS `await`은 Promise continuation을 등록하는 지점일 뿐 Kotlin의 구조적 동시성·취소·dispatcher를 제공하지 않는다. 코드 모양만 비교하고 동등한 개념처럼 가르치지 말아야 한다.
- `docs/02-언어-JS와-TS/09-비동기-콜백에서-async-await까지.md:131-139` — `Promise.all`은 Promise를 시작시키지 않고 이미 만든 Promise를 기다릴 뿐이다. `fetchOrders()`처럼 호출이 I/O를 시작하는 API를 먼저 호출했기 때문에 동시 요청이 되는 것이다. “Promise는 만들어지는 순간 시작”도 Promise 일반의 성질이 아니다.
- `docs/02-언어-JS와-TS/09-비동기-콜백에서-async-await까지.md:163` — `fetch`는 HTTP 4xx/5xx에는 reject하지 않지만, reject 사유가 네트워크 단절만은 아니다. abort, CORS에 의해 표면화되는 네트워크 오류 등도 reject된다. “HTTP 오류 상태는 reject하지 않고, 네트워크 계열 실패·abort는 reject될 수 있다”로 수정해야 한다.
- `docs/02-언어-JS와-TS/09-비동기-콜백에서-async-await까지.md:178` — TanStack Query가 모든 fetch를 자동 취소하는 것은 아니다. query function이 제공받은 `AbortSignal`을 실제 요청에 전달해야 취소가 전파된다. 이 조건을 쓰지 않으면 독자가 화면 전환만으로 서버 작업까지 자동 중단된다고 오해한다.
- `docs/02-언어-JS와-TS/10-모듈-시스템과-package-json.md:13,36-42` — `import`가 “패키지 이름이 아니라 파일 경로”라는 설명과 바로 아래 bare package import가 모순된다. 상대/절대(또는 alias) specifier는 파일·URL을, bare specifier는 패키지 또는 import map이 해석하는 이름을 가리킨다고 구분해야 한다.
- `docs/02-언어-JS와-TS/10-모듈-시스템과-package-json.md:46` — 순환 ESM import가 곧바로 `undefined`를 만든다는 설명은 틀렸다. ESM의 live binding은 초기화 전에 읽으면 대개 TDZ `ReferenceError`가 나며, 초기화 순서가 안전하면 순환 자체는 동작할 수 있다. CommonJS의 부분 초기화와도 구분해서 “초기화 순서 의존 버그”로 설명해야 한다.
- `docs/02-언어-JS와-TS/10-모듈-시스템과-package-json.md:88-89,150-151` — `devDependencies`는 “빌드/테스트에만 필요”가 아니다. Next.js 앱은 배포 빌드 단계에서 TypeScript·빌드 도구가 필요할 수 있고, 의존성 분류가 브라우저 번들 크기를 직접 결정하지도 않는다. 런타임 설치/배포 환경과 실제 import를 기준으로 번들러가 결정한다는 두 축으로 고쳐야 한다.
- `docs/02-언어-JS와-TS/10-모듈-시스템과-package-json.md:140` — `NEXT_PUBLIC_` 값 전체가 빌드 결과물에 자동으로 들어가는 것이 아니라, 클라이언트 코드에서 참조한 값이 빌드 시 인라인되어 공개된다. 그래도 비밀을 두면 안 된다는 결론은 유지하되 노출 메커니즘을 정확히 써야 한다.
- `docs/02-언어-JS와-TS/10-모듈-시스템과-package-json.md:70-78` — 기준 스택이 현재 Next.js App Router인데 예제는 Next `15.1.0`, React `19.0.0`, TypeScript `5.7.0`으로 고정되어 있다. 현재 기준 버전(Next 16 계열)으로 올리거나 “역사적 예시이며 실제 버전은 생성 시점의 lockfile을 따른다”를 명시해야 한다. 특히 Next 16의 Cache Components와 `proxy.ts` 등은 뒤 목차의 캐싱·라우팅 설명에 영향을 준다.
- `docs/02-언어-JS와-TS/11-TypeScript.md:9-11` — 타입 주석·interface·type 같은 **type-level** 구성은 소거되지만 TypeScript 문법이 전부 사라지는 것은 아니다. `enum`, `namespace`, decorators 등은 JavaScript 출력에 런타임 코드를 만들 수 있다. “타입 정보는 소거된다”로 정확히 한정해야 한다.
- `docs/02-언어-JS와-TS/11-TypeScript.md:69` — `strict: true`가 Kotlin과 같은 null 안전성을 준다는 표현은 과장이다. 핵심 옵션은 `strictNullChecks`이고, `any`, 타입 단언, 잘못된 타입 선언, 선택적 속성 등의 탈출구 때문에 TypeScript는 본질적으로 완전한 null-safe 언어가 아니다.
- `docs/03-화면의-재료/12-HTML-시맨틱과-폼.md:93` — “`h1`은 페이지당 하나”는 HTML/접근성 표준의 요구사항이 아니다. 페이지의 주제에 해당하는 명확한 `h1` 하나를 두는 것이 일반적인 작성 규칙이라고 안내하되, 섹션마다 제목을 둘 수 있고 레벨은 문서 구조로 결정된다고 고쳐야 한다.
- `docs/03-화면의-재료/12-HTML-시맨틱과-폼.md:153` — 시맨틱 태그만으로 접근성 대부분이 저절로 해결되지는 않는다. 포커스 이동·visible focus·대비·오류 메시지 연결·동적 상태 알림·모달 포커스 관리가 남는다. “강한 기본값을 제공하지만 점검의 시작점”으로 낮춰야 한다.
- `docs/03-화면의-재료/12-HTML-시맨틱과-폼.md:178` — React의 `htmlFor`/`className`은 단순히 `for`·`class`가 JavaScript 예약어라서 생긴 규칙이 아니다. JSX에서 React DOM property 이름을 쓰는 API 설계이며, JSX 속성은 JavaScript 변수 선언이 아니다. 이유를 DOM property/API 호환성으로 바로잡아야 한다.
- `docs/03-화면의-재료/13-CSS-기초.md:34-48,73-82` — 캐스케이드를 `!important → 특이성 → 선언 순서` 세 단계로 가르치면 Tailwind를 쓰는 독자에게도 틀린 모델이 된다. 적용 가능성, origin+importance, cascade layer, specificity, scope proximity, source order 순이며, `!important`도 user-agent/user origin·transition·layer에 대해 “무조건” 이기지 않는다. 최소화하더라도 inline style과 `@layer`가 특이성/순서보다 먼저 결정된다는 점은 넣어야 한다.
- `docs/03-화면의-재료/13-CSS-기초.md:158-164` — absolute positioned 요소의 기준은 “가장 가까운 `relative` 조상”이 아니라 가장 가까운 **position이 static이 아닌** 조상이며, `transform`·`contain` 등도 containing block을 만들 수 있다. relative 부모가 없으면 initial containing block을 기준으로 배치된다. 문장을 이 규칙으로 고쳐야 한다.
- `docs/03-화면의-재료/14-레이아웃-Flexbox와-Grid.md:91-99` — 카드 수가 적을 때 남는 칸까지 접어 카드를 넓히는 의도라면 `auto-fill`이 아니라 `auto-fit`이다. `auto-fill`은 빈 track을 유지할 수 있어 “남는 공간 균등 분배” 설명과 다르게 보인다. 목적을 선택지로 설명하거나 코드와 설명을 `auto-fit`으로 맞춰야 한다.
- `docs/03-화면의-재료/15-CSS의-전역-네임스페이스-문제와-Tailwind.md:91-101` — CSS-in-JS 전체가 렌더마다 스타일을 계산·주입하고 Server Components와 맞지 않는다는 서술은 틀렸다. 이는 runtime CSS-in-JS의 일부 trade-off이고, compile-time CSS-in-JS도 있으며 Next.js에서 runtime 라이브러리도 registry/SSR 통합을 제공한다. “runtime CSS-in-JS의 경우”로 대상을 한정해야 한다.
- `docs/03-화면의-재료/15-CSS의-전역-네임스페이스-문제와-Tailwind.md:111,117-121,179-180` — Tailwind utility가 항상 CSS 속성 하나와 일대일인 것은 아니다(`text-xl`처럼 font-size와 line-height를 함께 설정할 수 있다). Tailwind도 전역 CSS selector를 생성하고, arbitrary value·theme 확장이 가능하며, 소스에서 완전한 클래스명을 찾을 수 있을 때만 unused CSS를 제거한다. “이름을 안 짓고 남는 CSS가 없다/정해진 값만 쓴다” 같은 절대 표현을 이 제약까지 포함해 고쳐야 한다.
- `docs/부록/A-FE-BE-용어-대응표.md:25,28-30,41` — 본문에서 이미 부정확한 대응을 다시 굳힌다. Java 람다가 “값”만 캡처한다는 말은 mutable 객체 참조의 변화를 볼 수 있어 틀리고, `async/await = suspend`, `import = 파일 경로 기반`도 각각 9장·10장의 오류를 반복한다. React props도 런타임에 readonly로 동결되는 값이 아니라 컴포넌트가 불변 스냅샷처럼 취급해야 하는 입력이다. 표를 각 본문의 수정된 표현과 일치시켜야 한다.
- `react-guide/docs/01-React의-계약/02-React-이전-명령형-DOM-조작.md:37` — 외부 값으로 `innerHTML`을 조립해 넣는 예제가 XSS 취약 코드다. 명령형 UI의 문제를 보이려는 예제라도 `td.textContent = o.id`처럼 노드를 만들고 textContent를 설정해야 한다. 그렇지 않으면 React의 장점으로 XSS 방지가 암시되는 잘못된 학습이 된다.
- `react-guide/docs/01-React의-계약/02-React-이전-명령형-DOM-조작.md:75,119` — React가 항상 “바뀐 부분만” DOM에 갱신하거나 수동 코드가 항상 전부 지우는 것은 보장되지 않는다. React의 reconciliation은 DOM 변경을 최소화하려고 하지만 public API가 최소 변경 집합을 보장하지 않고, 명령형 코드도 상태 모델·정교한 patching을 만들 수 있다. 비교 대상을 “이 예제의 방식”으로 한정해야 한다.
- `react-guide/docs/01-React의-계약/02-React-이전-명령형-DOM-조작.md:155` — 직접 바꾼 DOM을 React가 다음 렌더에서 반드시 덮어쓴다는 말도 너무 강하다. React가 그 노드/속성을 다음 출력에서 관리할 때 덮어쓸 수 있는 것이며, 핵심 문제는 두 소유자가 같은 DOM을 관리해 불일치가 난다는 점이다.
- `react-guide/docs/01-React의-계약/03-UI는-state의-함수다.md:15` — React가 컴포넌트 함수를 “순서를 바꿔서라도” 호출한다는 표현은 Hooks 규칙과 충돌해 들린다. React는 렌더 작업을 시작·중단·폐기·재시도할 수 있지만, 한 컴포넌트 안의 Hook 호출 순서는 개발자가 고정해야 한다. 두 종류의 ‘순서’를 명확히 분리해야 한다.
- `react-guide/docs/01-React의-계약/03-UI는-state의-함수다.md:91-100` — 부수 효과를 놓을 자리가 “딱 두 곳”은 아니다. 공식 권고는 렌더 밖에서, 보통 이벤트 핸들러에 두고 외부 시스템 동기화가 필요할 때 Effect를 쓰라는 것이다. 서버 렌더/모듈 초기화/라이브러리 경계 등까지 배제하는 규칙처럼 단정하지 말고, React 컴포넌트 내부의 일반적 선택지라고 한정해야 한다.
- `react-guide/docs/01-React의-계약/04-가상-DOM의-진실.md:7,51-64,70-81` — “가상 DOM은 직접 DOM 조작보다 항상 느리다”와 “대충 짠 수동 DOM보다 빠른 경우가 많다”는 둘 다 보편 명제가 될 수 없다. 동일한 결과를 내는 최소 수동 DOM update와 비교하면 React에는 렌더·재조정 비용이 추가되지만, 실제 앱의 비용은 작업량·배칭·DOM 변화·브라우저 최적화에 따라 달라진다. 직접 DOM 변경도 항상 layout/paint를 유발하지 않는다. 이 조건부 비교로 바꿔야 한다.
- `react-guide/docs/01-React의-계약/04-가상-DOM의-진실.md:91-94` — 서버 HTML 렌더링(SSR)과 React Server Components(RSC)를 같은 것으로 묶었다. RSC는 RSC Payload를 만들고, Next.js는 Client Component/RSC Payload를 이용해 HTML을 pre-render하며, 클라이언트는 Client Component만 hydrate한다. “서버 HTML(24장 RSC)”를 SSR·RSC·hydration의 관계로 분리해야 한다.
- `react-guide/docs/01-React의-계약/04-가상-DOM의-진실.md:115` — 조건문에 따라 컴포넌트 위치가 바뀐다고 성능과 상태 보존이 항상 무너지는 것은 아니다. 같은 트리 위치에서 같은 type/key가 유지되면 조건부 렌더도 상태를 보존한다. 상태 reset을 일으키는 tree position/type/key 변경을 구체적인 예제로 설명해야 한다.
- `react-guide/docs/01-React의-계약/04-가상-DOM의-진실.md:145-147` — Svelte/Solid가 “실제로 더 빠르다”는 보편 결론과 React Compiler가 그 프레임워크처럼 가상 DOM을 없애는 방향이라는 설명은 부정확하다. 프레임워크 성능은 workload별 벤치마크 문제이고, React Compiler는 React의 모델을 바꾸지 않고 build-time memoization으로 불필요한 재계산/리렌더를 줄인다.

## 2. 앞에서 정의하지 않은 용어·코드가 먼저 나오는 곳

- `docs/01-브라우저라는-런타임/06-싱글-스레드와-이벤트-루프.md:54,64,122` — Promise를 9장에서 처음 정의하면서 6장에서는 Promise 콜백을 마이크로태스크의 대표로 사용한다. 큐 우선순위의 이유를 이해하려면 “Promise는 나중에 끝날 값을 나타내며 `.then` continuation이 microtask로 예약된다”는 한 문장 정의가 이 장에 먼저 필요하다.
- `docs/01-브라우저라는-런타임/06-싱글-스레드와-이벤트-루프.md:137-141,152` — 메모이제이션, 가상 스크롤, CPU-bound가 정의 없이 실무 처방 표에 들어간다. 메모이제이션·가상 스크롤은 각각 35장/20장으로 보내더라도, CPU-bound는 현재 문맥에서 ‘계산으로 메인 스레드를 점유하는 작업’이라고 정의해야 한다.
- `docs/02-언어-JS와-TS/08-함수가-일급-시민이라는-것.md:180-188` — React를 배우기 전인데 JSX, 컴포넌트, `props`, 부모/자식 단방향 흐름, `onClick`을 한 코드 블록에 넣는다. 17장까지 독자가 해석할 수 없는 코드다. 일반 JavaScript 콜백 예제로 바꾸고, React 예시는 “17장에서 다시 볼 미리보기”로 별도 상자에 최소 문법 설명과 함께 격리해야 한다.
- `docs/02-언어-JS와-TS/11-TypeScript.md:94-117` — TypeScript 장의 내로잉 예제가 React JSX 컴포넌트(`Empty`, `Profile`, `Spinner`, `Table`)와 화면 상태를 전제한다. React Part 4 전이라 `return <Empty />`가 무엇인지조차 정의되지 않았다. 순수 TypeScript 반환값 예제로 바꾸거나 React 예시는 Part 4 뒤로 옮겨야 한다.
- `docs/03-화면의-재료/15-CSS의-전역-네임스페이스-문제와-Tailwind.md:159-175` — `cn`, `clsx`, `isActive`, dark mode, 반응형 variant를 한꺼번에 도입한다. `cn`은 내장 함수가 아니고 구현도 import도 없으며, 이 스택에 포함되지 않은 관례다. `cn`의 정의/구성(`clsx`와 `tailwind-merge` 여부)을 먼저 쓰거나 단순 조건식으로 바꿔야 한다.
- `react-guide/docs/01-React의-계약/02-React-이전-명령형-DOM-조작.md:92-107` — `Result` 타입이 정의되어 있지 않고, 입문 11장의 `Result` 예제와도 `idle`/`orders` 대 `loading`/`data`로 모양이 다르다. `useState`, `Result`, `Spinner` 등 imports/최소 선언을 보이거나 “의사 코드”라고 명시해야 한다. 현재대로는 TypeScript 예제가 컴파일되지 않는다.
- `react-guide/docs/01-React의-계약/04-가상-DOM의-진실.md:97-99` — CS 기초가 없다는 전제에서 O(n³), O(n), 휴리스틱을 처음 쓰고 정의하지 않는다. `n`이 노드 수일 때 입력이 커질수록 비교 횟수가 어떻게 늘어나는지 2~3문장으로 설명하거나 표기 자체를 빼야 한다.

## 3. BE 비유가 잘못된 모델을 만드는 곳

- `docs/01-브라우저라는-런타임/06-싱글-스레드와-이벤트-루프.md:96-111` — `async/await`을 Kotlin coroutine과 “거의 같은 모양”으로 놓고 차이를 스레드 수 하나로만 축소한다. Kotlin의 `suspend` 함수는 호출자에게 `Deferred`를 만들지 않고 structured concurrency·cancellation·dispatcher 문맥을 가지며, JS `async`는 호출 즉시 Promise를 돌려준다. 이 비유는 9장까지 반복되므로 “문법 독해 앵커일 뿐 실행·취소 모델은 다르다”는 비교표가 필요하다.
- `docs/02-언어-JS와-TS/08-함수가-일급-시민이라는-것.md:152-158` — Java는 “값”, JS는 “변수 자체”를 캡처한다는 표는 Java의 effectively-final **참조**가 가리키는 객체 변경을 여전히 관찰할 수 있다는 점을 지운다. primitive 재할당과 mutable object mutation을 나눈 예제로 바꿔야 stale closure의 원인을 정확히 전달한다.
- `react-guide/docs/01-React의-계약/02-React-이전-명령형-DOM-조작.md:7-9` — 트랜잭션 없이 UPDATE를 나열하는 비유는 원자성/rollback 문제를 React의 선언형 UI 문제와 혼동시킨다. React는 DB transaction처럼 중간 DOM 변경을 rollback하는 도구가 아니다. “화면 상태를 직접 저장·동기화하는 명령이 분산된다”는 BE의 projection/view model 또는 단일 source of truth 비유로 교체해야 한다.
- `react-guide/docs/01-React의-계약/04-가상-DOM의-진실.md:11-17` — 가상 DOM을 ORM과 “정확히 같은 트레이드오프”라고 하는 비유는 과하다. ORM은 DB 질의/영속성 추상화이고, virtual DOM은 UI 선언을 DOM commit으로 재조정하는 구현 세부다. 특히 ORM의 타입 안전성은 보편 이득도 아니며 React의 예측 가능성은 virtual DOM 단독이 아니라 state/렌더 규칙에서 나온다. “수동 최적화보다 일관된 선언 모델을 택하는 비용” 정도의 느슨한 앵커로 낮춰야 한다.

## 4. 난이도가 급격히 뛰는 곳

- `docs/01-브라우저라는-런타임/06-싱글-스레드와-이벤트-루프.md:46-69,115-130` — 콜 스택 → microtask/task → 렌더 기회 → Promise 출력 순서를 한 장에서 요구한다. Promise 정의도 뒤 장이라 독자는 용어와 메커니즘을 동시에 외우게 된다. 먼저 “한 JS 작업은 끝까지 실행된다 + 브라우저가 나중 할 일을 큐에 넣는다”까지만 확립하고, microtask 우선순위는 Promise 장 뒤의 보충 상자/별도 절로 내려야 한다.
- `docs/02-언어-JS와-TS/11-TypeScript.md:102-119` — 기본 내로잉 직후 판별 유니온, JSX, 로딩/성공/에러 상태 모델을 동시에 던진다. 타입의 분기와 React 화면 분기를 분리해 예시를 두 단계로 나누지 않으면 TypeScript를 배우는 장에서 React 문법 장벽에 막힌다.
- `docs/03-화면의-재료/14-레이아웃-Flexbox와-Grid.md:101-119` — `grid-template-areas` 예제는 신규 속성 6개와 3×2 ASCII 레이아웃을 한 번에 해석해야 하는데, 바로 앞에서 Grid의 column/gap만 설명했다. 1행 2열의 최소 예제에서 `grid-area`를 하나씩 붙인 뒤 전체 골격으로 확장해야 한다.
- `docs/03-화면의-재료/15-CSS의-전역-네임스페이스-문제와-Tailwind.md:157-175` — Tailwind 첫 실전 코드가 조건 병합·responsive variant·상태 variant·dark mode·정적 탐지 제약을 연속으로 넣는다. 기본 utility 하나 → variant 하나 → 조건부 클래스 하나 순으로 나누고, 도구 의존 `cn`은 마지막에 별도 선택 주제로 빼야 한다.
- `react-guide/docs/01-React의-계약/04-가상-DOM의-진실.md:95-129` — diff 복잡도, `key`, tree position/state reset, Fiber, concurrent interruption, `useTransition`, Suspense를 한 흐름으로 묶었다. 이 가이드의 Part 2가 render/commit과 reconciliation/key를 별도 장으로 예고하므로, 여기서는 “두 가지 휴리스틱이 있다”까지만 두고 Fiber/concurrency는 Part 2 뒤로 옮겨야 한다.

## 5. 입문·심화 가이드의 중복과 빠진 연결

- `README.md:35-40` ↔ `react-guide/README.md:27-44` — 입문 16장의 “명령형에서 선언형, `UI = f(state)`”과 심화 2~3장의 명령형 DOM/`UI = f(state)`가 동일한 문제·해답을 두 번 설명한다. 입문은 사용법과 최소 mental model, 심화 2~3장은 순수성·스냅샷·interruptible render라는 심화 결론으로 역할을 분리하고, 심화의 재도입은 1~2쪽 요약으로 줄여야 한다.
- `README.md:38-39` ↔ `react-guide/README.md:32-44` — 입문 18장의 재조정·key와 19장의 Hooks가 심화 5~14장에서 render/commit, key, state, Hooks로 다시 전개된다. 중복 자체보다 “입문에서 어디까지 보장하고 심화에서 무엇을 새로 증명하는지” 경계가 없다. 각 입문 장 끝에 심화 장 번호와 ‘여기서는 규칙, 심화에서는 내부 동작/예외’를 명시해야 한다.
- `README.md:43-48` — Next.js App Router 파트에 `Link` 기반 클라이언트 전환, prefetch, RSC Payload, 초기 로드 hydration과 이후 navigation의 차이가 없다. 파일 라우팅만 배운 독자는 `<a>`/`Link`, full reload/client transition, layout state 보존을 연결하지 못한다. 23장 범위에 이 흐름을 넣어야 한다.
- `README.md:43-48` — 21장의 CSR/SSR/SSG/ISR 지도와 24장의 RSC 구분 사이에 “SSR과 RSC는 독립 축이며 Client Component도 초기 요청에서 HTML로 pre-render될 수 있다”가 빠져 있다. 이 누락은 `'use client' = 브라우저에서만 실행`, `RSC = SSR`이라는 가장 흔한 App Router 오해를 만든다. 21장 또는 24장 목차에 이 문장을 명시해야 한다.
- `README.md:47` — “fetch 캐싱과 Server Actions”만으로는 현재 App Router의 cache model을 설명할 수 없다. request memoization, Data Cache, Full Route Cache, Router Cache/재검증의 범위와 `revalidatePath`/`revalidateTag`의 갱신 대상을 분리해야 하며, Next 16을 기준으로 할 경우 Cache Components(`use cache`)와의 관계도 버전 표기와 함께 넣어야 한다.
- `README.md:47,50-54` — Server Component의 서버 fetch/Next cache와 TanStack Query의 client server-state cache가 언제 함께 쓰이고 언제 중복되는지의 hand-off가 목차에 없다. 25장 끝 또는 27장 첫 절에 ‘초기 RSC 데이터, client-side refetch가 필요한 데이터, hydration/dehydration 여부’를 결정하는 표가 필요하다.
- `react-guide/README.md:56` — React 19 장의 항목이 `use`, Actions, `useOptimistic`, Compiler만 나열한다. Actions를 실제 폼 흐름으로 이해하는 데 필요한 `useActionState`, `useFormStatus`, form `action`/`formAction`, 그리고 React 19의 `ref` as prop·`useRef` 타입 변경이 누락되어 있다. 최소한 전자는 장 범위에 포함하고, 후자는 migration 주석으로 넣어야 한다.
