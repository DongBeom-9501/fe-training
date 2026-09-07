# 2차 리뷰: 수정 필요 사항

## A. 1차 지적 반영 후 남은 문제·재발 문제

- `docs/01-웹-브라우저와-인터넷/03-브라우저가-화면을-그리는-과정.md:64-72` — `async` 스크립트를 “첫 렌더링을 막을 수 있음”으로 분류하면 CSS의 렌더 차단과 같은 종류로 읽힌다. `async` 리소스 자체가 첫 페인트를 기다리게 하는 것은 아니고, 다운로드가 끝난 시점의 실행이 파서를 멈추고 메인 스레드를 점유할 수 있다는 뜻으로 분리해 설명하라.

- `docs/01-웹-브라우저와-인터넷/05-렌더링-성능과-최적화.md:70,88,140` — 앞에서 조건부라고 완화했지만 다시 “합성이 가장 싸다”, “`transform`은 항상 합성”, “대체로 가장 싸다”라고 단정한다. 레이어 생성·래스터화·메모리·기기 상태에 따라 비용이 달라지므로, 속성 이름만으로 비용 순위를 보장하지 않는다고 일관되게 고쳐라.

- `docs/01-웹-브라우저와-인터넷/06-JavaScript-실행-모델과-비동기.md:98-115,147-148` — `async`/`await`과 코루틴의 차이를 사실상 “스레드 수 하나”로 축소했고, `async` 자체를 “양보”라고 적었다. `async` 함수 호출은 즉시 Promise를 반환할 뿐이며, 실행을 중단하는 지점은 `await`다. 취소·구조화된 동시성·스케줄러·dispatcher가 다른 별개 모델임을 명시하고, 비유 표를 교체하라.

- `docs/02-JavaScript와-TypeScript/10-모듈과-패키지.md:13,42` — import 대상을 `./`와 “그 외 패키지” 두 종류로 나누면 `../` 상대 경로와 설정된 alias/import map을 설명하지 못한다. “상대 경로(`./`, `../`) / 패키지 또는 도구가 해석하는 별칭”으로 바로잡아라.

- `docs/03-HTML과-CSS/13-CSS-선택자와-우선순위.md:34-51` — 인라인 스타일, `!important`, 명시도, 선언 순서만으로 우선순위를 설명하고 “`!important`는 항상 이긴다”고 결론낸다. origin, cascade layer, transition 등이 남고, author `!important`는 normal inline보다 우선한다. 이 단순 순서를 “동일 origin·layer에서의 축약”으로 한정하거나 정확한 cascade 순서를 제시하라.

- `docs/03-HTML과-CSS/13-CSS-선택자와-우선순위.md:162-167` — absolute 위치 기준을 “가장 가까운 `position`이 `static`이 아닌 조상”으로만 설명한다. `transform`, `filter`, `perspective`, `contain` 등도 containing block을 만들 수 있으므로 “대표적인 경우”로 좁히거나 해당 조건을 추가하라.

- `docs/03-HTML과-CSS/15-Tailwind-CSS.md:97-101,117-123,152-153` — runtime CSS-in-JS가 매 렌더마다 스타일을 계산·주입한다고 일반화하고, Tailwind의 충돌·CSS 증가·영향 범위·임의값 사용을 절대적으로 말한다. 라이브러리 캐싱 여부와 일반 CSS의 전역 선택자 가능성을 반영하고, Tailwind 임의값(`w-[...]` 등)이 가능한데도 “필요 없다”고 한 문장을 정정하라.

- `docs/부록/A-FE-BE-용어-대응표.md:25,28-30,41` — 본문에서 완화한 오류가 부록에 그대로 남아 있다. closure를 항상 reference capture로, `async`/`await`을 코루틴 suspend로, event loop를 thread pool이 없는 모델로, import를 파일 경로만으로, props를 런타임 불변 값으로 대응시키지 말고 본문과 같은 한계·차이를 반영하라.

- `react-guide/docs/01-React-사고방식/02-React는-무엇을-해결하는가.md:81,147-148` — 코드가 `replaceChildren()`로 바뀌었는데 서술은 여전히 `tbody.innerHTML = ''`를 가리킨다. 또 React가 DOM 직접 조작을 “못 하게 막는다”고 쓰지 말고, 가능하지만 React가 관리하는 DOM과 충돌해 source of truth가 둘이 된다고 설명하라.

- `react-guide/docs/01-React-사고방식/04-가상-DOM과-성능.md:13-17,72-79,153-154` — ORM과 가상 DOM을 “정확히 같은 trade-off”로 비유한 문장과, DOM 변경이 곧 layout/paint이고 수동 DOM이 일반적으로 더 빠르다는 대비가 남아 있다. ORM은 SQL 생성/영속성 문맥이고 React는 트리 비교·commit 문맥이라는 차이를 먼저 밝히고, layout/paint는 변경 종류와 브라우저 최적화에 따라 발생한다고 고쳐라.

## B. 새 React 16–20장

- `docs/04-React/16-명령형에서-선언형으로.md:43` — 사용자 데이터인 `o.id`, `o.product`, `o.status`를 `insertAdjacentHTML()` 템플릿에 보간해 XSS가 다시 생겼다. 2장의 `createElement()`/`textContent` 예시처럼 DOM 노드를 만들거나, 이 명령형 예시 자체를 JSX 이전의 안전한 API로 작성하라.

- `docs/04-React/16-명령형에서-선언형으로.md:58-60` — “상태가 n개면 화면 조합이 2ⁿ개”는 n개의 독립 boolean일 때만 성립한다. `result`처럼 상호 배타적인 상태까지 포함한 일반 공식처럼 쓰지 말고 조건을 붙여라.

- `docs/04-React/16-명령형에서-선언형으로.md:66-70,110-116` — 첫 React 코드에서 `useState`와 `Result`를 정의 없이 사용하고 `Result` 선언은 한참 뒤에 나온다. 이 장이 Hooks 전이므로 먼저 “현재 값과 갱신 함수 쌍을 반환하는 React API”를 짧게 정의하고, `Result`를 코드보다 앞에 배치하라.

- `docs/04-React/16-명령형에서-선언형으로.md:93,146-149,174,189-190,204-211` — `UI = f(state)`를 모든 컴포넌트의 완전한 식처럼 제시하고, JSX가 `createElement`로 컴파일된다는 표현·`htmlFor`의 이유·“하나의 root 필수”·“JSX/null만 반환”·“React가 직접 DOM 조작을 막음”을 모두 일반 규칙으로 적었다. props/context도 출력에 영향을 준다는 단서를 넣고, 현대 JSX transform은 개념적으로만 `createElement`와 대응한다고 밝혀라. `htmlFor`는 DOM API 이름을 따른 것이며, JSX의 한 표현식 제약과 컴포넌트가 반환할 수 있는 React node(문자열·배열·`null` 포함)를 구분하고 직접 조작은 금지가 아닌 escape hatch라고 고쳐라.

- `docs/04-React/17-컴포넌트와-props.md:15-29,80-89` — `<tbody>`를 `<section>` 바로 아래에 두고, 독립 예시에서 `<tr>`을 렌더한다. 이는 유효한 HTML 구조가 아니다. 표 전체(`<table><tbody>…`)를 예시로 보이거나, 해당 컴포넌트가 table 안에서만 쓰인다는 문맥을 코드로 보장하라. 선택 동작도 클릭 가능한 `<tr>` 대신 셀 안의 `<button>`으로 제공해 키보드 접근성을 지켜라.

- `docs/04-React/17-컴포넌트와-props.md:140-161,184,205` — `React.ReactNode`를 정의·import 없이 도입하고, props 불변·상속 부재·자식의 부모 state 변경 불가를 언어/React의 강제 규칙처럼 쓴다. `ReactNode`가 “React가 렌더 가능한 값”임을 먼저 설명하고, props 불변은 지켜야 할 규약이며 class component 상속은 존재하되 React가 composition을 권장한다는 식으로 고쳐라. 부모가 setter를 props로 넘기면 자식도 갱신 요청을 할 수 있으므로 ownership 규칙과 금지 규칙을 구분하라.

- `docs/04-React/18-state와-렌더링-사이클.md:7-9` — JPA dirty checking을 “바뀐 열만 UPDATE”로 비유하면 Hibernate 기본 설정 등에서 틀릴 수 있다. dirty checking은 변경을 감지해 update를 예약한다는 수준으로 한정하고, 실제 SQL 열 구성은 provider/configuration에 따라 다르다고 밝혀라. React도 “바뀐 부분만 DOM 반영”은 구현 세부를 보장하는 계약이 아니라는 단서를 붙여라.

- `docs/04-React/18-state와-렌더링-사이클.md:106-113,132,142-149` — state 동일성 비교를 reference comparison이라 하고, `setState` 뒤 컴포넌트·모든 자식이 다시 렌더된다고 단정한다. React는 `Object.is`로 bailout하며, 자식은 memo/React Compiler 등으로 건너뛸 수 있고 render 시도와 DOM commit도 다르다. “갱신한 컴포넌트에서 렌더를 시작할 수 있으며, 실제 호출·commit은 최적화와 값에 따라 달라진다”로 단계별로 설명하라.

- `docs/04-React/18-state와-렌더링-사이클.md:185-189` — 목록 앞에 항목을 넣을 때 index key가 “모든 행을 다른 것으로 보고 완전히 다시 그린다”고 설명한 것은 반대에 가깝다. index key 때문에 기존 위치의 컴포넌트/DOM을 새 데이터에 재사용하여 입력값·local state가 다른 항목으로 옮겨갈 수 있다는 것이 핵심이다.

- `docs/04-React/19-Hooks.md:7-10,15-20` — Hooks를 `@Transactional` 같은 lifecycle interception으로 비유하면 AOP/annotation처럼 오해된다. 또한 모든 `use*` 호출이 조건문·반복문에서 금지된다고 쓰면 React 19의 `use` 예외와 충돌한다. Hook은 렌더 중 React에 state/effect 정보를 등록하는 호출이라고 설명하고, `use`는 Hook이 아니며 조건문·반복문에서 가능하되 `try/catch` 안에서는 안 된다는 예외를 넣어라.

- `docs/04-React/19-Hooks.md:107-122,150-159` — `useEffect(..., [])`를 “처음 한 번만”, cleanup을 “시작한 모든 Effect에 반드시 반환”이라고 단정한다. 전자는 mount마다 실행되고 Strict Mode 개발 환경에서는 setup → cleanup → setup 검증도 있다. 후자는 subscription, timer, request 등 정리할 외부 자원이 있을 때 대칭 cleanup이 필요하다고 써라.

- `docs/04-React/19-Hooks.md:190-224` — Server Component와 TanStack Query가 `useEffect` 데이터 요청의 문제를 “해결”한다고 묶고, 의존성 객체 문제의 해법을 primitive dependency나 `useMemo`/`useCallback`으로만 제시한다. 서버 데이터, 사용자 상호작용 후의 client fetch, cache 동기화는 서로 다른 경우다. 객체/함수는 Effect 안에서 만들거나 실제 primitive 의존성만 나열하는 방식을 먼저 제시하고, memoization은 의미 보장이 아니라 참조 안정화 도구임을 명시하라.

- `docs/04-React/19-Hooks.md:230-247` — `useWindowWidth`가 render/초기 state에서 `window.innerWidth`를 읽는다. Next.js App Router의 Client Component도 초기 HTML 생성 과정에서 서버에서 pre-render될 수 있으므로 `window is not defined`가 난다. 초기 fallback을 두고 Effect에서 측정하거나 `useSyncExternalStore` 기반 구현으로 바꾸고, hydration 시 값이 바뀔 수 있음을 설명하라.

- `docs/04-React/20-컴포넌트-설계.md:7-10,76-88` — 컴포넌트 경계를 곧 rerender 경계라고 설명하고 분리만으로 자식 렌더가 막히는 듯 서술한다. 부모 state 변경은 자식을 다시 호출할 수 있으며, state를 가까이 둬 갱신 시작 범위를 줄이는 것과 memo/Compiler bailout이 실제 호출을 줄이는 것을 구분하라.

- `docs/04-React/20-컴포넌트-설계.md:48-72` — “AddOrderButton 내부 state”가 왜 OrderList를 다시 렌더하지 않는지 말하지만, 정작 `AddOrderButton` 구현을 보여주지 않는다. Modal을 포함해 local state를 선언한 최소 컴포넌트 코드를 넣어 주장을 검증 가능하게 만들어라.

- `docs/04-React/20-컴포넌트-설계.md:90-99,116-154` — container/presentation 분리와 RSC/Client Component 경계가 상당히 겹친다고 연결하고, `children` composition이 prop drilling 대부분을 해결하며 Context는 최후 수단처럼 제시한다. RSC/Client 경계는 브라우저 API·상호작용·module graph/bundle의 경계이지 표현/데이터 컴포넌트 분류가 아니다. 명시 props, slot composition, subtree-scoped Context, 외부 store의 선택 기준을 별도 표로 정리하라. React 19 기준 예시는 `<UserContext.Provider>` 대신 `<UserContext value={user}>`를 쓰거나 전자를 legacy 문법으로 표시하라.

- `docs/04-React/20-컴포넌트-설계.md:184` — “파일 하나 = 컴포넌트 하나 = default export”를 원칙처럼 제시하면서 10장의 named export 권장과 충돌한다. 팀 규약의 한 선택지로 낮추고, 이 자료에서 채택할 export 규칙을 하나로 통일하라.

## C. 한글 가독성

- `docs/04-React/16-명령형에서-선언형으로.md:58-60` — 상태, 예시, `n`, `2ⁿ`, 결론이 한 문단에 겹쳐 첫 독자가 전제를 놓친다. “독립 boolean flag가 늘면 조합이 늘어난다”와 “상호 배타적인 상태는 type으로 제한한다”를 두 문단으로 나눠라.

- `docs/04-React/16-명령형에서-선언형으로.md:196-211` — “이 대가”, “그렇다고”, “바로 이것”의 지시 대상이 각각 선언형 UI, 명령형 제어, React의 교환 조건 중 무엇인지 흔들린다. 각 문장에서 `선언형 렌더링의 대가`, `명령형 escape hatch`, `React가 관리하는 DOM`을 명사로 반복해 지시어를 없애라.

- `docs/04-React/18-state와-렌더링-사이클.md:142-149` — render, reconciliation, commit, “결과”, 화면 반영을 짧은 문장들에 연속으로 넣어 단계가 섞인다. “새 React element tree 계산 → 이전 tree와 비교 → 필요한 DOM 변경 commit”으로 번호를 붙이고, `결과`를 `계산된 element tree` 또는 `DOM 변경`으로 구체화하라.

- `docs/04-React/19-Hooks.md:150-159` — “시작했으면 반드시 멈춰야 한다”는 비유가 무엇을 시작한 것인지 불명확하고 cleanup의 적용 범위도 과장한다. timer·event listener·subscription처럼 종료가 필요한 자원과 로그처럼 cleanup 없는 Effect를 대조하는 두 문장으로 고쳐라.

- `docs/04-React/19-Hooks.md:253` — “`useEffect`는 componentDidMount가 아니다. 생명주기 훅으로 생각하면 …”은 부정한 개념을 바로 다음 문장에서 다시 전제해 읽기 걸린다. “`useEffect`를 componentDidMount 같은 생명주기 훅으로 취급하면 …”으로 주어를 명확히 바꿔라.

- `docs/04-React/20-컴포넌트-설계.md:114-129` — `children`을 slot으로 쓰는 경우, props 전달, Context의 관계가 “이걸로”, “그런데”로 이어져 선택 기준이 흐린다. 각 기법의 목적을 먼저 한 문장씩 쓰고, 뒤의 세 bullet을 “언제 props / 언제 composition / 언제 Context” 형식으로 바꿔라.

- `docs/03-HTML과-CSS/15-Tailwind-CSS.md:175-179` — `sm:`, `md:`, `hover:`, `dark:`가 무엇의 접두사인지 설명하지 않은 채 “이 문법”, “다시”라고 가리킨다. “Tailwind variant prefix”라고 명명한 뒤 breakpoint·상태·테마가 각각 어느 접두사인지 한 문장씩 분리해라.

- `react-guide/docs/01-React-사고방식/03-React가-필요한-이유.md:15` — 한 문장 안에 “입문 과정의 이후 장”, “Order 서비스”, “선택하지 않은 이유”, “복잡해지는 시점”이 함께 들어간다. 입문 가이드에서 배운 범위와 심화 가이드에서 다룰 범위를 두 문장으로 나누고, `그것` 같은 지시어 대신 `상태·렌더링·의존성`을 명시하라.
