# HTTP 307 redirect와 rewrite의 네트워크 흐름

> Sources: IETF / RFC 9110, Unknown; Vercel / Next.js 공식 문서, Unknown; 사용자 제공 처리 선택지 화면, 2026-10-13
> Raw: [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md); [Next.js redirects](../../raw/frontend/nextjs-redirects.md); [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md); [NextResponse](../../raw/frontend/nextjs-next-response.md); [미지원 언어 처리 선택지](../../raw/frontend/2026-10-13-unsupported-locale-routing-options.md)
> Updated: 2026-10-13

## Overview

**307 redirect는 클라이언트에게 다른 URL로 다시 요청하라고 응답하는 방식이고, rewrite는 요청한 URL을 유지하면서 서버에서 처리할 경로를 바꾸는 방식이다.** 307은 HTTP 상태 코드이고, rewrite는 Next.js의 routing·proxy 동작이다. rewrite 자체에 대응하는 별도의 HTTP redirect 상태 코드가 있는 것은 아니다. [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md); [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md); [NextResponse](../../raw/frontend/nextjs-next-response.md)

아래 예제는 제공된 화면의 `/ja/notice`를 `/notice`로 처리하는 선택지를 사용한다. 일반적인 브라우저 문서 탐색, redirect 자동 추적, 캐시에서 응답하지 않는 상황을 가정하며 CSS·JS·이미지·Next.js의 prefetch 요청은 생략한다. 실제 사이트의 응답을 측정한 결과가 아니라 공식 규칙을 적용한 설명용 흐름이다. [미지원 언어 처리 선택지](../../raw/frontend/2026-10-13-unsupported-locale-routing-options.md)

## 요청과 응답이 어떻게 달라지는가

### 307 redirect

```text
브라우저 → 서버: GET /ja/notice
서버 → 브라우저: 307 Temporary Redirect
                 Location: /notice
브라우저 → 서버: GET /notice
서버 → 브라우저: 200 OK + 국문 페이지

최종 주소창: /notice
```

서버의 첫 응답은 최종 페이지가 아니라 새 목적지를 알리는 응답이다. RFC는 서버가 `Location` 헤더로 목적지를 제공하도록 권고하며, 클라이언트는 이를 따라 새 요청을 보낼 수 있다고 정의한다. 브라우저가 자동으로 따라가는 위 예제에서는 최초 요청과 목적지 요청이 순서대로 필요하다. 목적지 페이지가 정상 응답한다고 가정한 `200 OK`는 설명용 값이며, 실제 결과는 목적지의 처리 결과에 달려 있다. [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md); [Next.js redirects](../../raw/frontend/nextjs-redirects.md)

### 내부 rewrite

```text
브라우저 → 서버: GET /ja/notice
서버 내부: /ja/notice를 /notice 처리 경로로 매핑
서버 → 브라우저: 200 OK + 국문 페이지

최종 주소창: /ja/notice
```

rewrite 자체는 브라우저에 `/notice`를 다시 요청하라고 지시하지 않는다. Next.js가 목적지 경로의 내용을 원래 요청에 대한 응답으로 제공한다. 내부 경로 매핑은 브라우저가 별도 HTTP 요청을 보내는 동작과 다르다. 여기서도 `200 OK`는 목적지 페이지가 정상 처리된다는 가정이며, rewrite가 항상 성공 상태를 보장하는 것은 아니다. [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md); [NextResponse](../../raw/frontend/nextjs-next-response.md)

| 비교 대상 | 307 redirect | 내부 rewrite |
| --- | --- | --- |
| 목적지로 이동하는 주체 | 클라이언트가 `Location`을 따라 요청한다. | 서버가 내부 처리 경로를 매핑한다. |
| 위 예제의 브라우저 요청 | 원래 주소 요청 + 목적지 요청 | 원래 주소 요청 |
| 문서 탐색 후 주소창 | `/notice` | `/ja/notice` |
| 응답의 의미 | 먼저 임시 이동을 알리고 목적지 응답을 별도로 받는다. | 원래 요청에 목적지의 처리 결과를 돌려준다. |
| 다음에 공유하는 주소 | 주소창을 복사하면 `/notice` | 주소창을 복사하면 `/ja/notice` |

주소창 복사 결과와 요청 구성은 위 가정에서 도출한 설명이다. `fetch`의 redirect 추적은 요청의 목적지를 바꾸는 동작이며, 그 자체를 현재 문서의 주소창 이동과 동일하게 이해하면 안 된다. [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md); [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md)

## HTTP 요청과 네트워크 연결을 구분한다

307의 비용은 목적지 요청을 보내기 전에 이동 응답을 기다려야 한다는 순차적인 의존성에 있다. 캐시나 추가 redirect가 없는 위 예제에서는 내부 rewrite보다 브라우저와 서버 사이에 요청·응답 왕복이 더 필요하다. 다만 실제 시간 차이는 서버 처리 시간과 네트워크 조건에 따라 달라지므로 고정된 지연 수치로 표현할 수 없다. 이는 표준의 재요청 흐름에서 도출한 성능 판단이다. [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md)

**새 HTTP 요청이 필요하다는 말이 새 TCP 연결이나 TLS handshake를 반드시 만든다는 뜻은 아니다.** 307의 규정은 HTTP 메서드와 목적지에 관한 것이며, 새 transport 연결을 요구하는 규칙은 아니다. 따라서 요청 수만으로 DNS 조회나 연결 설정 횟수를 단정해서는 안 된다. 이는 HTTP 응답 의미와 연결 설정을 구분한 해석이다. [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md)

**rewrite도 네트워크 요청이 전혀 없다는 뜻은 아니다.** Next.js는 외부 URL로 rewrite할 수 있다. 이 경우 브라우저는 원래 서버에 요청하지만 서버가 외부 목적지로 proxy 요청을 보낼 수 있으므로, 브라우저가 보는 요청과 서버가 보내는 요청을 따로 센다. 내부 rewrite와 외부 rewrite의 비용을 같게 보면 안 된다. [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md)

## 307은 HTTP 메서드를 보존한다

307을 자동으로 따라갈 때 클라이언트는 메서드를 바꾸면 안 된다. GET은 GET으로, POST는 POST로 목적지에 다시 요청한다. 원래 요청을 다시 보내는 흐름이므로 POST의 본문도 재전송하는 것으로 이해하되, 모든 헤더가 그대로 복사된다고 보지는 않는다. RFC는 목적지에 따라 `Host`, `Authorization`, `Cookie` 등의 헤더를 제거하거나 다시 구성하는 규칙도 설명한다. [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md)

Next.js가 307과 308을 사용하는 이유도 메서드를 명확히 보존하기 위해서다. 전통적인 301·302에서는 POST가 GET으로 바뀌는 동작이 허용돼 왔다. **307의 temporary는 임시 URL 이동이라는 뜻이며, POST를 GET으로 바꾸거나 원래 URL을 영구적으로 대체한다는 뜻이 아니다.** [HTTP redirection과 307](../../raw/frontend/rfc-9110-redirection-and-307.md); [Next.js redirects](../../raw/frontend/nextjs-redirects.md)

따라서 `/ja/notice`처럼 페이지 조회 경로에서 GET 요청을 처리하는 선택과, 데이터를 제출하는 POST 경로에 같은 규칙을 적용하는 선택은 별도로 판단해야 한다. 이 문서는 제공된 화면의 페이지 탐색 예제를 설명하며, 서비스 전체에 redirect 규칙을 적용하는 구현 지시는 아니다. [미지원 언어 처리 선택지](../../raw/frontend/2026-10-13-unsupported-locale-routing-options.md)

## Next.js에서 표현하는 방법

아래 코드는 처리 방식의 차이를 보여주는 설명용 예제다. 원문 API 사용법을 화면의 목적지에 맞춰 바꿨으며, 미지원 언어 판별 조건이나 matcher를 포함한 완성 코드가 아니다. [NextResponse](../../raw/frontend/nextjs-next-response.md); [미지원 언어 처리 선택지](../../raw/frontend/2026-10-13-unsupported-locale-routing-options.md)

```ts
// proxy 또는 middleware에서 사용할 두 가지 대안
const destination = request.nextUrl.clone();
destination.pathname = '/notice';

// 대안 A: 브라우저에 목적지로 재요청하도록 알린다.
return NextResponse.redirect(destination, 307);

// 대안 B: 위 return 대신 사용하며, 브라우저 URL을 유지한다.
// return NextResponse.rewrite(destination);
```

`next.config.js`의 `redirects`는 `permanent: false`일 때 307을 사용하고, `rewrites`는 `source`와 `destination`을 매핑한다. 설정 파일의 정적 규칙과 `NextResponse`를 사용하는 조건부 처리 방식은 구현 위치가 다르지만, 브라우저를 이동시키는지와 원래 URL을 유지하는지의 구분은 같다. [Next.js redirects](../../raw/frontend/nextjs-redirects.md); [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md); [NextResponse](../../raw/frontend/nextjs-next-response.md)

## 화면의 선택지를 어떻게 읽을 것인가

제공된 화면은 `ko·en·zh`를 지원하는 것으로 설명하고, `/ja/notice` 같은 미지원 언어 접두를 제거하는 방식을 묻는다. 또한 국문 URL에는 접두를 두지 않아 `/ko/notice`도 `/notice`로 이동하는 예시를 제시한다. 이는 화면에 제안된 정책이며, 실제 배포된 응답이나 현재 route 목록을 검증한 사실은 아니다. [미지원 언어 처리 선택지](../../raw/frontend/2026-10-13-unsupported-locale-routing-options.md)

- **주소 자체를 국문 경로로 정리하려면 307:** 주소창과 복사한 링크가 `/notice`로 바뀌지만, redirect 응답 뒤 목적지 요청이 필요하다.
- **원래 주소를 유지하면서 국문 내용을 보여주려면 rewrite:** 브라우저의 redirect 재요청을 피하지만, `/ja/notice`가 주소창과 공유 링크에 남는다.

위 선택 기준은 요구사항과 HTTP 흐름을 연결한 설계 판단이다. 화면처럼 국문 접두 없는 주소로 정리하는 것이 목표라면 307이 그 의도에 맞는다. 반대로 미지원 언어 주소를 그대로 유지하는 것이 목표라면 rewrite가 맞는다. 화면의 `Recommended` 표시만으로 어느 서비스에서나 307이 더 옳다고 일반화하지 않는다. [미지원 언어 처리 선택지](../../raw/frontend/2026-10-13-unsupported-locale-routing-options.md); [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md)

## Network 탭에서 확인할 것

실제 앱을 검증할 때는 DevTools의 Network 탭에서 기록을 유지하고 `/ja/notice`에 직접 접속해 문서 요청을 확인한다. 아래는 이번 조사에서 실행한 검증 결과가 아니라, 위 흐름을 확인하기 위한 항목이다.

- 307 방식: 최초 응답의 상태와 `Location`, 이어지는 `/notice` 요청, 최종 주소창을 확인한다.
- rewrite 방식: 주소창은 `/ja/notice`인지, 원래 요청으로 국문 페이지가 반환되는지, 별도 redirect가 섞이지 않았는지 확인한다.
- 외부 rewrite: 브라우저의 Network 탭만으로 서버의 외부 요청 수를 알 수 없으므로 서버 로그나 tracing도 확인한다.
- 추가 요청: 페이지의 asset·prefetch 요청을 redirect 때문에 발생한 문서 재요청과 구분한다.

rewrite의 목적지에서 별도 redirect를 반환할 수도 있으므로, 실제 응답에서 307이 보인다는 이유만으로 rewrite 설정이 없다고 단정하지 않는다. 이 확인 방법은 proxy와 redirect를 함께 적용할 수 있다는 공식 동작에서 도출한 검증 지침이다. [Next.js rewrites](../../raw/frontend/nextjs-rewrites.md); [NextResponse](../../raw/frontend/nextjs-next-response.md)

## See Also

- [next-intl로 Next.js 다국어 화면 만들기](next-intl.md)
- [Next.js App Router에서 canonical과 페이지별 metadata 구현](../seo/nextjs-app-router-canonical-and-page-metadata.md)
