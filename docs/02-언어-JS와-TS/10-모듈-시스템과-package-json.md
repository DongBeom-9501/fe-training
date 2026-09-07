# 10. 모듈 시스템과 package.json 읽는 법

> **이 장의 질문**: 프론트엔드 프로젝트를 처음 열었을 때 루트에 있는 파일들은 무엇이고, `import`는 어떻게 동작하는가?

## BE로 치면 이런 것

Kotlin 프로젝트를 처음 열면 당신은 `build.gradle.kts`부터 봅니다. 의존성이 뭐고, 어떤 플러그인이 붙어 있고, 태스크가 뭐가 있는지. 그러면 프로젝트의 성격이 대충 파악되죠.

프론트엔드에서 그 자리에 있는 파일이 **`package.json`** 입니다. 다만 하는 일이 조금 더 많습니다 — 의존성 선언 + 스크립트 실행기 + 패키지 메타데이터를 겸합니다.

## import / export

Java의 `import`와 결정적으로 다른 점: **가리키는 대상이 두 종류** 라는 것입니다. `./`로 시작하면 파일 경로이고, 그렇지 않으면 패키지 이름입니다.

```js
// 이름 있는 내보내기 (named export) — 하나의 파일에서 여러 개 가능
export function formatDate(d) { ... }
export const MAX_PAGE = 100;

// 가져오기 — 이름이 정확히 일치해야 한다
import { formatDate, MAX_PAGE } from './utils/date';
```

```js
// 기본 내보내기 (default export) — 파일당 하나
export default function OrderTable() { ... }

// 가져오기 — 이름은 아무거나 붙일 수 있다
import OrderTable from './components/OrderTable';
```

**실무 규칙**: 팀마다 다르지만, **named export를 기본으로 쓰는 팀이 늘고 있습니다.** 이름이 강제되어 자동완성과 리팩터링이 안정적이기 때문입니다. 다만 Next.js의 페이지 파일처럼 **프레임워크가 default export를 요구하는 자리** 가 있습니다(23장).

### 경로 세 가지

```js
import { formatDate } from './utils/date';        // 상대 경로 — 내 코드
import { formatDate } from '@/utils/date';        // 별칭 — 프로젝트 루트 기준 (설정 필요)
import { useQuery } from '@tanstack/react-query'; // 패키지 이름 — node_modules에서 찾음
```

`./`나 `../`로 시작하지 않으면 브라우저/번들러는 **패키지**로 간주하고 `node_modules`에서 찾습니다. `@/`로 시작하는 것은 프로젝트가 설정한 별칭인데, `../../../utils/date` 같은 지옥을 피하려고 씁니다. Next.js는 기본으로 제공합니다.

### 순환 참조

A가 B를 import하고 B가 A를 import하면, Java에서는 문제없지만 JavaScript에서는 **초기화 순서에 따라 결과가 달라집니다.** 운이 좋으면 그냥 동작하고, 나쁘면 "초기화되기 전에 접근했다"는 `ReferenceError`가 납니다. 에러가 나는 지점과 원인이 멀어서 찾기 어렵습니다.

> **예방법**: 의존성 방향을 한쪽으로 유지하세요. 백엔드에서 도메인 → 서비스 → 컨트롤러 방향을 지키는 것과 같은 원칙입니다. 린터로 잡을 수 있습니다(33장).

## package.json 읽기

실제 파일을 위에서부터 읽어봅시다.

```json
{
  "name": "admin-console",
  "version": "0.1.0",
  "private": true,
  "type": "module",

  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint .",
    "test": "vitest"
  },

  "dependencies": {
    "next": "15.1.0",
    "react": "^19.0.0",
    "@tanstack/react-query": "^5.62.0"
  },

  "devDependencies": {
    "typescript": "^5.7.0",
    "eslint": "^9.17.0",
    "vitest": "^2.1.0"
  }
}
```

> 아래 버전 숫자는 **작성 시점의 예시** 입니다. 실제 버전은 프로젝트를 만드는 시점의 최신을 따르고, 정확한 값은 락 파일이 정합니다.

| 필드 | 뜻 | Gradle로 치면 |
| --- | --- | --- |
| `private: true` | npm 공개 저장소에 실수로 배포되는 것 방지 | — |
| `type: "module"` | ESM(`import`) 사용. 없으면 옛 CommonJS(`require`) | — |
| `scripts` | `npm run dev` 로 실행 | Gradle task |
| `dependencies` | 배포된 앱을 돌리는 데 필요 | `implementation` |
| `devDependencies` | 개발·빌드·테스트에만 필요 | `testImplementation`, 플러그인 |

**`scripts`가 사실상의 진입점입니다.** 낯선 프로젝트를 받으면 여기부터 보세요. 어떻게 띄우고, 어떻게 빌드하고, 어떻게 테스트하는지가 다 적혀 있습니다.

## 버전 표기 읽는 법

시맨틱 버저닝은 `MAJOR.MINOR.PATCH`입니다. 앞에 붙는 기호가 "얼마나 자동 업데이트를 허용하는가"를 정합니다.

| 표기 | 허용 범위 | 뜻 |
| --- | --- | --- |
| `5.62.0` | 정확히 그것만 | 고정 (pin) |
| `~5.62.0` | `5.62.x` | 패치만 |
| `^5.62.0` | `5.x.x` (`6.0.0` 미만) | **마이너까지** — 기본값 |
| `*` | 아무거나 | 쓰지 마세요 |

`^`가 기본값이라는 게 중요합니다. **`package.json`만으로는 실제로 설치될 버전이 정해지지 않습니다.** 오늘 설치한 사람과 내일 설치한 사람이 다른 버전을 받을 수 있습니다.

그래서 **락 파일** 이 있습니다.

| 파일 | 도구 |
| --- | --- |
| `package-lock.json` | npm |
| `pnpm-lock.yaml` | pnpm |
| `yarn.lock` | yarn |

락 파일에는 **실제로 설치된 정확한 버전** 이 전부 적혀 있습니다. Gradle의 lockfile과 같은 역할입니다.

> **규칙 두 개**
> 1. 락 파일은 **반드시 커밋**합니다.
> 2. CI에서는 `npm ci`(락 파일 그대로 설치)를 쓰고 `npm install`(락 파일을 갱신할 수 있음)을 쓰지 않습니다.

## 프로젝트 루트에 있는 다른 파일들

Next.js + TypeScript 프로젝트를 열면 대략 이렇습니다.

```
admin-console/
├── package.json           의존성과 스크립트
├── pnpm-lock.yaml         정확한 버전 (커밋 필수)
├── tsconfig.json          TypeScript 컴파일러 설정
├── next.config.ts         프레임워크 설정
├── eslint.config.mjs      린트 규칙
├── .env.local             환경변수 (커밋 금지)
├── node_modules/          설치된 패키지 (커밋 금지, .gitignore)
├── public/                그대로 서빙되는 정적 파일
└── src/
    ├── app/               라우팅 (23장)
    ├── components/        재사용 컴포넌트
    └── lib/               유틸리티, API 클라이언트
```

**환경변수 하나만 미리 짚어 둡니다.** `NEXT_PUBLIC_`으로 시작하는 환경변수를 **브라우저에서 도는 코드가 참조하면, 빌드할 때 그 값이 코드 안에 문자열로 박힙니다.** 사용자가 그대로 읽을 수 있다는 뜻입니다. 2장에서 말한 "프론트엔드에 넣은 비밀은 비밀이 아니다"가 이것입니다. API 키를 여기 넣으면 안 됩니다.

## 흔한 오해

**"node_modules를 커밋해야 하나?"**
아닙니다. 락 파일만 커밋합니다. `node_modules`는 수만 개 파일이 들어 있습니다.

**"`^`가 붙어 있으니 자동으로 최신을 쓰겠네"**
락 파일이 있으면 락 파일이 이깁니다. 업데이트는 명시적으로(`npm update`, 또는 Renovate 같은 봇으로) 합니다.

**"devDependencies에 넣으면 번들에서 빠진다"**
아닙니다. 브라우저로 전송될 번들에 무엇이 들어가는지는 **실제로 `import` 했는지** 를 보고 번들러가 정합니다. 이 구분은 번들 크기가 아니라 "배포 환경에 무엇을 설치해야 하는가"의 문제입니다.

## 핵심 3가지

- **`import`는 두 종류입니다.** `./`로 시작하면 파일 경로, 아니면 `node_modules`의 패키지 이름입니다.
- **`package.json`이 프로젝트의 얼굴입니다.** 낯선 프로젝트를 받으면 `scripts`부터 보세요.
- **버전 표기는 범위일 뿐입니다.** 실제 설치 버전은 락 파일이 정합니다. 반드시 커밋하세요.

## 스스로 답해보기

1. 락 파일을 커밋하지 않으면 팀에서 무슨 일이 벌어지는가?
2. 낯선 프론트엔드 프로젝트를 받았을 때 `package.json`에서 가장 먼저 봐야 할 필드는?

---

**다음 장 예고** — Part 2의 마지막입니다. 지금까지 본 JavaScript의 헐거운 부분들을 TypeScript가 어떻게 조여주는지, 그리고 Java의 타입 시스템과 어디서 갈라지는지 봅니다.
