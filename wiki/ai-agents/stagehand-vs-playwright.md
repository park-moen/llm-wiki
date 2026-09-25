# Stagehand: Playwright와의 차이와 시작 방법

> Sources: Browserbase, Unknown; GeekNews, Unknown; Browserbase Stagehand Docs, Unknown
> Raw: [Stagehand README 예시](../../raw/ai-agents/stagehand-readme.md); [Stagehand README 설치·기능](../../raw/ai-agents/stagehand-readme-install-and-usage.md); [GeekNews 소개·토론](../../raw/ai-agents/stagehand-v4-geeknews.md); [Stagehand Quickstart 발췌](../../raw/ai-agents/stagehand-quickstart-extract.md); [Codex 연동 문서 발췌](../../raw/ai-agents/stagehand-codex-integration-extract-2.md); [속도 최적화 문서 발췌](../../raw/ai-agents/stagehand-speed-optimization-extract.md)
> Updated: 2026-09-25

## Overview

Stagehand는 브라우저를 사용하는 AI agent용 SDK다. Playwright와 비슷한 `goto`, `locator`, `click`, `screenshot` API에 자연어 기반 `act()`, `observe()`, `extract()`를 더한다. 따라서 선택자를 이미 아는 동작은 코드로 지정하고, 페이지 구조를 해석해야 하는 동작은 모델에 맡기는 식으로 함께 쓸 수 있다.

## Playwright와 무엇이 다른가

| 관점 | Playwright | Stagehand |
| --- | --- | --- |
| 주된 출발점 | 웹 테스트와 명시적 브라우저 제어 | 브라우저를 사용하는 AI agent와 데이터 추출 |
| 요소 조작 | 코드에서 locator와 동작을 지정 | Playwright 스타일 API를 제공하고 `observe()`로 자연어에 맞는 요소의 실제 selector를 찾거나 `act()`로 자연어 지시를 실행 |
| 데이터 추출 | 페이지 요소를 코드로 읽고 가공 | `extract()`로 요청한 데이터를 스키마에 맞춰 추출 |
| 실행 선택 | 개발자가 대상과 절차를 명시 | 명시적 조작과 모델 기반 조작을 한 흐름에서 조합 |
| 실행 환경 | 자체 브라우저 자동화 도구 | 로컬 Chrome 또는 Browserbase 브라우저 사용; TypeScript, Python, Go SDK 제공 |

Stagehand가 Playwright 전체 API와 완전히 호환된다는 뜻은 아니다. README는 **Playwright 스타일 API**와 별도의 마이그레이션 가이드를 소개한다. 기존 테스트를 옮길 때는 사용 중인 API의 대응 여부를 확인해야 한다.

Stagehand v4는 브라우저 시작 시 확장 프로그램을 로드해 브라우저 안에서 명령을 처리하고, 여러 동작을 묶어 왕복 지연을 줄인다고 설명한다. 접근성 트리에서 모델에 전달할 정보를 줄이는 방식도 사용한다. GeekNews는 v4가 Playwright보다 **2배 빠르고 토큰 사용량이 80% 적다**는 주장을 전한다. README의 속도 비교 대상은 **Browserbase에서 실행하는 Stagehand와 동등한 클라우드 Playwright 브라우저**다. 로컬 실행이나 모든 작업에서 같은 개선이 나온다는 근거로 일반화하면 안 된다. 이 수치는 제공 측의 비교 결과이며, 여기서는 독립 검증하지 않았다.

## 핵심 API와 선택 기준

- `page.goto()`, `page.locator(...).fill()` 등: URL이나 selector가 분명하고 동작을 직접 통제할 때 쓴다.
- `observe("find the email input")`: 자연어로 요소를 찾고 실제 selector를 받는다. 로그인 예시처럼 비밀번호 값은 모델 호출에 넣지 않고, 반환된 selector에 코드로 입력할 수 있다.
- `act("click the sign in button")`: 자연어 지시에 맞춰 페이지를 조작한다. README는 사이트가 바뀌면 실행 방법을 다시 찾는다고 설명한다.
- `extract("extract every invoice in the table", schema)`: 페이지 내용을 요청한 구조로 추출한다. TypeScript 예시는 Zod, Python 예시는 Pydantic 스키마를 쓴다.

선택자가 안정적이고 검증 결과를 재현해야 하는 단계에는 명시적 API가 적합하다. 화면 구조가 자주 바뀌거나 의미로 요소를 찾아야 하는 단계에는 `observe()`와 `act()`가 유용할 수 있다. 이는 두 출처의 API 설명을 바탕으로 한 사용 판단이며, 실제 안정성은 대상 사이트에서 확인해야 한다.

## 로컬에서 시작하기: TypeScript

README의 설치 명령은 다음과 같다. 로컬 실행에는 Chrome이 필요하다.

```sh
pnpm add @browserbasehq/stagehand 'zod@~4.4.3'
```

모델 API 키를 환경 변수 `OPENAI_API_KEY`로 설정한 뒤, 다음처럼 브라우저와 Stagehand를 만든다. 아래 URL은 실행할 서비스의 주소로 바꿔야 한다.

```ts
import { localBrowser, Stagehand } from "@browserbasehq/stagehand";
import { z } from "zod/v4";

const browser = await localBrowser.launch({ userDataDir: "./browser-data" });
const stagehand = await Stagehand.create({
  browser,
  model: { modelName: "openai/gpt-5.4-mini", apiKey: process.env.OPENAI_API_KEY },
});

try {
  const [page] = await browser.context.pages();
  await page.goto("https://app.example.com/login");

  const { data: email } = await stagehand.observe("find the email input");
  const { data: password } = await stagehand.observe("find the password input");
  await page.locator(email[0].selector).fill(process.env.APP_EMAIL!);
  await page.locator(password[0].selector).fill(process.env.APP_PASSWORD!);
  await stagehand.act("click the sign in button");
  await stagehand.act("open the billing page");

  const { data } = await stagehand.extract(
    "extract every invoice in the table",
    z.object({
      invoices: z.array(z.object({
        number: z.string(),
        amount: z.number(),
        paid: z.boolean(),
      })),
    }),
  );
  console.log(data.invoices);
} finally {
  await stagehand.close();
  await browser.close();
}
```

`userDataDir`를 지정하면 쿠키를 보관해 다음 실행에서도 로그인 상태를 사용할 수 있다. 예시는 `APP_EMAIL`과 `APP_PASSWORD`도 환경 변수에서 읽는다.

Python은 `pip install stagehand`, Go는 README의 `go get` 명령으로 설치할 수 있다. Browserbase에서 실행하려면 `browserbase.launch()`에 `BROWSERBASE_API_KEY`를 전달한다. Browserbase의 Model Gateway와 서버 측 캐시는 선택 기능이다. Browserbase의 호스팅 MCP 서버를 연결하면 코딩 agent에서 `navigate`, `act`, `observe`, `extract`를 사용할 수도 있다.

## AI agent 개발 시뮬레이션: 수정한 화면을 브라우저로 검증하기

다음은 **구현 예시를 바탕으로 구성한 가상 작업**이다. Stagehand와 Playwright를 실제로 나란히 실행해 시간을 측정한 결과가 아니다. [공식 Codex 연동 문서](https://docs.stagehand.dev/v4/integrations/codex)의 실험적 연동은 Codex에 지속되는 브라우저 세션과 `run`, `snapshot`, `screenshot` 도구를 제공한다. 한 MCP 서버 프로세스가 브라우저를 유지하므로 도구를 다시 호출해도 페이지 상태가 남는다.

예를 들어 개발자가 agent에게 로컬 앱의 회원가입 화면에서 모바일 폭일 때 제출 버튼이 가려지는 문제를 고치고 브라우저에서 다시 확인하도록 요청했다고 가정한다.

| 순서 | AI agent가 하는 일 | Stagehand가 맡는 부분 |
| --- | --- | --- |
| 재현 | 개발 서버와 관련 코드를 확인하고 로컬 앱을 연다. | 연결된 브라우저에서 화면을 열고 `snapshot` 또는 `screenshot`으로 현재 상태를 확인한다. |
| 원인 확인 | 화면 상태와 코드를 함께 보고 버튼이 가려지는 조건을 찾는다. | `run`으로 화면 크기 변경·요소 확인 같은 브라우저 조작을 실행하고 필요하면 다시 캡처한다. |
| 수정 | agent가 앱의 CSS 또는 component 코드를 바꾼다. | Stagehand는 코드 수정 도구가 아니라 수정 후 브라우저 검증에 쓰인다. |
| 재검증 | 같은 화면과 조건을 다시 확인하고 결과를 보고한다. | 유지된 세션에서 새로고침·조작·캡처를 반복한다. |

이 표의 화면과 결함은 설명용이다. 실제 agent의 tool 호출과 결과는 연동 방식, 앱 구조, 로그인 상태에 따라 달라진다. 로컬 앱을 대상으로 한다면 Codex 연동의 로컬 Chrome 모드를 사용할 수 있다. 공식 예제는 Stagehand 저장소를 빌드한 뒤 MCP 서버를 연결하는 방식이며, 이 연동은 실험적이라고 명시한다. 앞의 TypeScript 예제는 **앱 코드에서 Stagehand SDK를 직접 호출하는 방법**이고, 이 시뮬레이션은 **코딩 agent에 브라우저 도구를 붙이는 방법**이다.

### 이 상황에서 무엇이 빨라질 수 있나

Stagehand가 제시한 **2배 빠른 실행**은 Browserbase의 클라우드 브라우저에서 같은 스크립트를 실행한 Playwright와 비교한 주장이다. 따라서 위의 **로컬 앱 개발 전체가 2배 빨라진다는 의미는 아니다**. 코드 탐색·수정·빌드·테스트 시간까지 포함한 비교 자료는 확인한 출처에서 찾지 못했다.

개발 중 브라우저를 여러 번 조작할 때 Stagehand의 확장 프로그램 기반 실행과 명령 묶음은 브라우저와의 왕복 지연을 줄일 수 있다. 특히 원격 브라우저에서는 그 차이가 누적될 수 있다. 반면 화면 확인 과정에 모델 추론이 들어가면 그 시간도 고려해야 한다. Stagehand 문서는 `observe()`로 여러 동작을 한 번 계획한 다음 반환된 action을 `act()`로 재생하면 추가 모델 추론을 줄일 수 있다고 설명한다. 이 최적화도 위의 가상 개발 작업에서 측정한 결과는 아니다.

이미 selector와 검증 절차가 정해진 회귀 테스트에는 Playwright 스크립트가 직접적이다. agent가 화면을 탐색하며 위치를 찾고 변경 뒤 다시 확인하는 작업에는 Stagehand 도구가 편할 수 있다. 어느 쪽이 더 빠른지는 같은 앱·같은 작업·같은 브라우저 환경에서 재현, 수정, 재검증까지 걸린 시간을 비교해야 판단할 수 있다.

## 읽을 때 유의할 점

GeekNews의 속도·토큰 수치는 제품 발표와 소개 글의 주장이다. 실제 선택에서는 현재 Playwright 코드가 쓰는 API, 로컬 또는 클라우드 실행 환경, 모델 호출 비용, 대상 사이트의 변경 빈도를 함께 확인해야 한다. README 예제의 로그인 주소와 청구 데이터는 설명용이므로 실제 사이트에 바로 실행할 수 있는 완성 예제는 아니다.
