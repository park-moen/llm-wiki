# Orval로 OpenAPI 클라이언트 생성하기

> Sources: Orval 공식 문서, Unknown
> Raw: [Overview](../../raw/frontend/orval-overview.md); [Installation](../../raw/frontend/orval-installation.md); [Quick Start](../../raw/frontend/orval-quick-start.md); [Basics](../../raw/frontend/orval-basics.md); [MSW](../../raw/frontend/orval-msw.md); [Input](../../raw/frontend/orval-input-validation.md); [Output](../../raw/frontend/orval-output-limitations.md)
> Updated: 2026-09-29

## Overview

Orval은 OpenAPI v3 또는 Swagger v2 명세를 읽어 TypeScript 모델과 타입이 붙은 HTTP 요청 함수를 생성한다. 설정에 따라 React Query hook과 MSW mock도 만들 수 있다. 프런트엔드와 백엔드가 OpenAPI를 계약으로 사용한다면 요청·응답 타입과 호출 함수를 수동으로 중복 작성하는 일을 줄이는 도구다. 단, 생성 결과는 입력 명세에 의존한다. [Overview](../../raw/frontend/orval-overview.md); [Quick Start](../../raw/frontend/orval-quick-start.md)

## 왜 사용하는가

API 명세가 있어도 프런트엔드에서 요청 함수, 요청·응답 타입, 테스트용 mock을 따로 작성하면 같은 계약을 여러 번 옮겨 적어야 한다. Orval은 이 산출물을 명세에서 생성해 API 변경을 코드에 반영하는 경로를 만든다. 공식 문서는 TypeScript 모델, HTTP 요청 함수, MSW mock 생성을 핵심 목표로 설명한다. React Query hook이 필요하면 `client: 'react-query'`를 선택할 수 있다. [Overview](../../raw/frontend/orval-overview.md); [Quick Start](../../raw/frontend/orval-quick-start.md)

**적합한 상황:** 백엔드가 유효한 OpenAPI 명세를 제공하고, 프런트엔드가 그 명세를 기준으로 타입이 있는 호출 코드나 mock을 반복 생성해야 할 때다. 생성 코드와 실제 서버 응답의 일치를 보장하려면 명세를 서버 변경과 함께 갱신하고 통합 테스트로 확인해야 한다. 마지막 문장은 생성 코드가 명세를 입력으로 삼는다는 사실에서 도출한 운영 판단이다. [Overview](../../raw/frontend/orval-overview.md)

## 시작 방법

Orval을 개발 의존성으로 설치한다. 현재 공식 설치 문서는 Orval 8+에 Node.js 22.18 이상이 필요하다고 안내한다. 더 오래된 Node LTS를 쓰는 프로젝트에는 공식 Docker image로 생성기를 실행하는 방법을 제시한다. [Installation](../../raw/frontend/orval-installation.md)

```sh
npm install orval -D
```

먼저 YAML 또는 JSON OpenAPI 명세를 준비한다. 설정 파일 없이 한 번 생성하려면 공식 Quick Start의 명령을 실행한다. `--input`은 명세 경로, `--output`은 생성 파일 경로다. [Quick Start](../../raw/frontend/orval-quick-start.md)

```sh
orval --input ./petstore.yaml --output ./src/petstore.ts
```

설정을 반복해서 사용할 때는 프로젝트 루트에 `orval.config.ts`를 둔다. 다음은 공식 Basics 예시처럼 React Query hook을 Fetch 기반으로 만들고, 모델과 mock도 생성하는 설정이다. `client`는 생성할 API 형태를, `httpClient`는 요청 전송 방식을 고른다. 이 둘은 별도 설정이다. [Basics](../../raw/frontend/orval-basics.md)

```ts
import { defineConfig } from 'orval';

export default defineConfig({
  petstore: {
    output: {
      mode: 'single',
      target: './src/petstore.ts',
      schemas: './src/model',
      client: 'react-query',
      httpClient: 'fetch',
      mock: true,
    },
    input: {
      target: './petstore.yaml',
    },
  },
});
```

설정 파일을 만들었다면 `orval`로 생성한다. Orval은 프로젝트 루트의 `orval.config.ts` 등을 자동으로 찾는다. `mock: true`는 MSW handler와 Faker factory를 함께 만든다. MSW handler만 필요하면 `mock: { generators: [{ type: 'msw' }] }`로 범위를 좁힐 수 있다. [Quick Start](../../raw/frontend/orval-quick-start.md); [MSW](../../raw/frontend/orval-msw.md)

## 선택할 때의 트레이드오프

| 선택 | 얻는 것 | 확인할 점 |
| --- | --- | --- |
| 명세에서 코드 생성 | 요청 함수·타입·mock을 한 계약에서 만든다. | 입력 명세가 실제 API와 함께 관리돼야 한다. 이는 생성 방식에서 도출한 운영 판단이다. |
| React Query와 MSW 생성 | hook과 개발·테스트용 handler를 직접 작성하는 양을 줄인다. | 필요한 산출물에 맞춰 `client`, `httpClient`, `mock`을 각각 정한다. `mock: true`는 Faker factory도 만든다. |
| 출력 파일 구성 선택 | 기본 `single`은 한 파일에 생성한다. | API 규모와 리뷰 방식에 따라 `split` 또는 tag 기반 mode가 필요한지 판단한다. |

[Overview](../../raw/frontend/orval-overview.md); [Basics](../../raw/frontend/orval-basics.md); [MSW](../../raw/frontend/orval-msw.md); [Output](../../raw/frontend/orval-output-limitations.md)

명세가 유효하지 않을 때 `unsafeDisableValidation`으로 검증을 끌 수 있지만, 공식 문서는 이 경우 코드 생성의 동작을 보장하지 않고 사소한 버전 변경에서도 깨질 수 있다고 경고한다. 명세를 수정할 수 있다면 먼저 `override.transformer`로 검증 전에 바로잡는 쪽을 권한다. 외부 `$ref`는 기본적으로 거부되며, 허용 목록을 지정해야 한다. `['*']`로 모두 허용하면 명세에 적힌 임의의 로컬 파일이나 URL을 읽을 수 있으므로 신뢰하는 문서만 허용하는 편이 안전하다. [Input](../../raw/frontend/orval-input-validation.md)

경로 변수에 `urlEncodeParameters`를 켜더라도 배열·객체 경로 변수는 OpenAPI의 `style` 규칙대로 직렬화하지 않고 `String(value)`로 처리한다. 그런 매개변수를 사용하는 API라면 생성 결과의 URL을 실제 요청으로 확인해야 한다. [Output](../../raw/frontend/orval-output-limitations.md)
