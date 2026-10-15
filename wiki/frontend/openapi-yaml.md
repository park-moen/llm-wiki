# openapi.yaml로 HTTP API 명세 읽고 작성하기

> Sources: OpenAPI Initiative 공식 명세·학습 문서, Unknown; Swagger UI 공식 README, Unknown; Orval 공식 문서, Unknown
> Raw: [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md); [Introduction](../../raw/frontend/openapi-introduction.md); [문서 구조](../../raw/frontend/openapi-description-structure.md); [Best Practices](../../raw/frontend/openapi-best-practices.md); [Swagger UI](../../raw/frontend/swagger-ui-overview.md); [Orval Overview](../../raw/frontend/orval-overview.md)
> Updated: 2026-10-13

## Overview

`openapi.yaml`은 HTTP API를 OpenAPI 규칙에 맞춰 YAML로 기술한 명세 파일의 일반적인 이름이다. 어떤 URL에 어떤 메서드로 요청하는지, 요청에 필요한 값과 응답 데이터 구조, 인증 방식 등을 사람이 읽고 도구가 처리할 수 있는 형태로 정의한다. 표준은 명세의 시작 문서 이름으로 `openapi.yaml` 또는 `openapi.json`을 권장한다. 파일명 자체가 API 동작을 만드는 것은 아니다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md); [문서 구조](../../raw/frontend/openapi-description-structure.md)

FE에서 TypeScript 함수의 입력·반환 타입을 함께 정하는 것처럼, 프런트엔드와 백엔드가 HTTP 요청·응답 계약을 공유하는 데 사용한다고 이해할 수 있다. 다만 URL·HTTP 메서드·응답 상태·인증도 포함하며, 실제 서버와 일치하는지는 별도로 검증해야 한다. 이 비유는 공식 문서의 기계가 읽을 수 있는 API 계약 설명을 바탕으로 한 학습용 설명이다. [Introduction](../../raw/frontend/openapi-introduction.md); [Best Practices](../../raw/frontend/openapi-best-practices.md)

## 이름과 도구의 역할을 구분한다

| 이름 | 역할 |
| --- | --- |
| OpenAPI Specification | API 명세를 작성하는 표준 규칙이다. |
| `openapi.yaml` | 그 규칙으로 작성한 YAML 문서이며, 다른 파일명도 가능하다. |
| `openapi.json` | 같은 명세 구조를 JSON 문법으로 표현한다. |
| Swagger UI | OpenAPI 명세를 읽어 API 문서 화면과 상호작용 기능을 제공한다. |
| Orval | 명세를 읽어 TypeScript 모델·요청 함수와 설정에 따른 mock 등을 생성한다. |

[OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md); [Swagger UI](../../raw/frontend/swagger-ui-overview.md); [Orval Overview](../../raw/frontend/orval-overview.md)

명세의 저장 형식과 API의 전송 형식은 별개다. 명세가 YAML이어도 API 응답은 JSON·XML·바이너리 등일 수 있으며, 실제 형식은 해당 요청·응답의 `content`에 기술한다. 또한 하나의 파일로 작성하거나 `$ref`로 여러 문서를 연결할 수 있다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

## 주요 필드 읽는 순서

| 필드 | 무엇을 설명하는가 | 읽을 때 확인할 점 |
| --- | --- | --- |
| `openapi` | 문서가 따르는 OpenAPI 규격 버전 | `info.version`과 별개이며 도구 지원 버전과 맞춰야 한다. |
| `info` | 명세의 제목·버전 등 메타데이터 | `title`과 `version`이 필요하다. |
| `servers` | API 서버 주소 | 어떤 서버를 대상으로 하는지 확인한다. |
| `paths` | 경로별 HTTP operation | 같은 경로에서도 `get`과 `post`는 별개 operation이다. |
| `parameters` | 경로·query·header·cookie의 입력 | `name`, `in`, `required`, `schema` 등을 확인한다. |
| `requestBody` | 요청 본문 | `content`의 media type과 본문 schema를 확인한다. |
| `responses` | 응답 상태별 설명·데이터 | 성공 응답뿐 아니라 필요한 오류 응답도 기술한다. |
| `components.schemas` | 재사용하는 데이터 구조 | `$ref`가 가리키는 정의와 제약을 확인한다. |
| `components.securitySchemes`와 `security` | 인증 방식의 정의와 적용 | 방식을 정의한 것과 실제 operation에 적용한 것을 구분한다. |

[OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

`openapi`와 `info`는 루트의 필수 필드다. 수집한 규격에서는 `paths`, `components`, `webhooks` 중 적어도 하나가 필요하므로, 모든 명세에 위 표의 필드가 전부 있어야 하는 것은 아니다. 일반적인 요청·응답 API를 배우는 이 문서에서는 `paths`를 중심으로 읽는다. [문서 구조](../../raw/frontend/openapi-description-structure.md)

**버전은 서로 다른 대상을 가리킨다.** `openapi`는 규격의 버전이고, `info.version`은 명세 문서의 버전이며, 실제 API 구현 버전과도 구별된다. 이 문서의 예제는 수집한 OpenAPI 3.1.1을 기준으로 작성했다. 실제 프로젝트에서는 validator·Swagger UI·코드 생성기가 지원하는 버전을 함께 확인한다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md); [Swagger UI](../../raw/frontend/swagger-ui-overview.md)

## 사용자 조회 API 예제

다음은 공식 필드 규칙을 적용해 만든 **가상 학습 예제**다. 실제 서비스나 호출 가능한 서버를 나타내지 않는다. 서버 주소·operation 이름·응답 문구는 예제용이며, `GET /users/{id}`의 경로 입력과 응답 구조를 한 문서에서 보여준다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

```yaml
openapi: 3.1.1
info:
  title: Example User API
  version: '1.0.0'
servers:
  - url: https://api.example.com
paths:
  /users/{id}:
    get:
      operationId: getUser
      summary: 사용자 조회
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: 조회 성공
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: 사용자를 찾을 수 없음
components:
  schemas:
    User:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
        name:
          type: string
```

읽는 순서는 서버 주소 → 경로 → 메서드 → 입력 → 응답이다. `{id}`는 `in: path`인 parameter와 연결되고, 경로 parameter는 `required: true`여야 한다. 성공 응답은 JSON이며, 데이터 구조는 `User`를 참조한다. `User.required`는 응답 객체에 `id`와 `name`이 있어야 한다는 뜻이다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

`operationId`는 operation을 식별하는 문자열이며, 문서 내에서 유일해야 한다. 도구가 이를 식별자로 사용할 수 있지만, 모든 코드 생성기가 같은 함수명을 만든다고 보장하는 규칙은 아니다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

## parameters와 requestBody의 차이

`/users/{id}`의 `id`는 경로 입력이고, `/users?active=true`의 `active`는 query 입력이다. JSON으로 사용자 이름을 제출하는 경우는 `requestBody`로 표현한다. 아래는 사용자 생성 operation의 **일부만 보여주는 가상 예제**이며, 독립적인 OpenAPI 문서가 아니다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

```yaml
requestBody:
  required: true
  content:
    application/json:
      schema:
        type: object
        required:
          - name
        properties:
          name:
            type: string
```

여기서 `requestBody.required: true`는 본문 자체가 필요하다는 뜻이고, schema의 `required: [name]`은 본문 객체에 `name` 필드가 있어야 한다는 뜻이다. 필드의 존재 여부와 `null` 허용 여부도 별개다. 또한 request body의 `required`는 기본적으로 `false`이므로, 필요한 본문이라면 명시한다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

## schema와 실제 구현의 관계

schema는 데이터의 타입·필수 필드·제약을 기술하며, DB 테이블 정의와 동일한 개념이 아니다. API가 어떤 데이터를 주고받는지를 설명하므로 DB entity의 모든 필드를 그대로 노출해야 할 이유는 없다. 이는 명세가 외부 API 계약을 기술한다는 사실에서 도출한 설계 판단이다. [Introduction](../../raw/frontend/openapi-introduction.md); [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md)

`components`에 정의한 schema나 인증 방식은 참조하거나 적용해야 효과가 있다. 예를 들어 `securitySchemes`에 Bearer 인증을 선언한 것과 `security`로 이를 요구하는 것은 다른 단계다. 명세에 인증을 적었다고 서버의 인증 검사가 자동으로 구현되는 것도 아니다. 구현과 명세의 일치는 별도 검증 도구와 테스트로 확인한다. [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md); [Best Practices](../../raw/frontend/openapi-best-practices.md)

## 어떤 흐름으로 관리하는가

- **Design-first:** API 명세를 먼저 합의하고 구현·문서·클라이언트 생성에 사용한다.
- **Code-first:** 구현 코드·annotation 등을 바탕으로 명세를 생성한다.

공식 Best Practices는 Design-first를 권장하지만 두 접근을 모두 설명한다. 어느 쪽이든 원본으로 관리할 대상을 분명히 해야 한다. 생성한 명세를 수동으로 수정하면서 annotation도 따로 유지하면 서로 달라질 수 있으므로, 원본을 하나로 두거나 CI에서 일치를 검사한다. [Best Practices](../../raw/frontend/openapi-best-practices.md)

Orval을 쓰는 흐름은 **OpenAPI 명세 → TypeScript 모델·요청 함수 → FE에서 API 호출**이다. Swagger UI는 같은 명세를 사람이 읽고 API와 상호작용할 수 있는 화면으로 표현한다. 명세를 공통 입력으로 사용하면 문서와 호출 코드를 반복해서 옮겨 적는 작업을 줄일 수 있다. [Orval Overview](../../raw/frontend/orval-overview.md); [Swagger UI](../../raw/frontend/swagger-ui-overview.md)

## 검증할 때 구분할 것

1. **YAML 문법:** 들여쓰기·배열·문자열 등이 올바른지 확인한다.
2. **OpenAPI 구조:** 지정한 버전의 필드 규칙·참조·operation 구성이 유효한지 확인한다.
3. **팀 규칙:** 이름·설명·오류 응답 등 프로젝트 규칙을 lint로 확인한다.
4. **실제 API 동작:** 서버 요청·응답이 명세의 계약과 일치하는지 테스트한다.

이는 공식 문서의 문법·규격 검증과 CI 운영 권고를 학습용으로 나눈 확인 순서다. **명세 파일을 validator가 통과시킨 것과 실제 서버가 그 명세를 지키는 것은 별개의 검증이다.** 코드 생성 역시 입력 명세를 기준으로 하므로, 명세가 실제 서버와 다르면 생성한 타입과 호출 함수도 실제 응답에 맞지 않을 수 있다. [Best Practices](../../raw/frontend/openapi-best-practices.md); [Orval로 OpenAPI 클라이언트 생성하기](orval-openapi-client-generation.md)

YAML의 들여쓰기는 구조에 영향을 주고 필드 이름은 대소문자를 구분한다. 응답 상태 키는 `'200'`처럼 문자열로 적으며, `$ref`를 사용하면 참조 대상 파일과 경로가 존재하는지 확인한다. OpenAPI로 모든 가능한 HTTP API를 완전히 표현할 수 있는 것은 아니므로, 명세로 표현하기 어려운 동작은 별도 설명과 검증이 필요하다. [문서 구조](../../raw/frontend/openapi-description-structure.md); [OpenAPI Specification](../../raw/frontend/openapi-specification-3-1-1.md); [Best Practices](../../raw/frontend/openapi-best-practices.md)

## See Also

- [Orval로 OpenAPI 클라이언트 생성하기](orval-openapi-client-generation.md)
- [Spring MVC API 응답과 HttpMessageConverter](../spring/spring-mvc-api-response.md)
