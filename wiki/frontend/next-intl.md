# next-intl로 Next.js 다국어 화면 만들기

> Sources: Jan Amann / next-intl 공식 문서, Unknown
> Raw: [Overview](../../raw/frontend/next-intl-overview.md); [App Router](../../raw/frontend/next-intl-app-router.md); [Translations](../../raw/frontend/next-intl-translations.md); [Routing setup](../../raw/frontend/next-intl-routing-setup.md); [Server & Client Components](../../raw/frontend/next-intl-server-client-components.md); [Request configuration](../../raw/frontend/next-intl-request-configuration.md)
> Updated: 2026-10-13

## Overview

`next-intl`은 Next.js의 국제화(i18n)를 위한 도구다. 언어별 번역 문구 표시, 날짜·숫자 등 지역에 맞는 형식, 언어별 URL 처리를 제공한다. React 화면에서 번역 문구를 사용하면서 Next.js의 App Router와 Server·Client Components에 맞춰 설정할 수 있다. [Overview](../../raw/frontend/next-intl-overview.md)

이 문서는 수집 시점의 공식 문서와 공식 저장소 `main`의 원문을 기준으로 App Router 사용 흐름을 정리한다. 기존 프로젝트에 적용할 때는 설치된 Next.js·next-intl 버전에 맞는 API와 설정을 확인해야 한다. 특히 routing과 정적 렌더링 설정은 아래 버전 관련 안내를 함께 읽는다. [App Router](../../raw/frontend/next-intl-app-router.md); [Routing setup](../../raw/frontend/next-intl-routing-setup.md)

## locale과 messages

**locale**은 언어와 표시 형식을 나타내는 식별자이며, `en-US`처럼 지역 정보를 포함할 수 있다. **messages**는 locale별로 묶은 번역 키와 문구의 쌍이다. 일반적으로 JSON 파일을 사용하지만, 원격 데이터 소스에서 불러오는 것도 가능하다. [Translations](../../raw/frontend/next-intl-translations.md); [Request configuration](../../raw/frontend/next-intl-request-configuration.md)

아래는 공식 문서의 메시지와 React 사용 예제다. `About`은 관련 문구를 묶는 namespace이고, `t('title')`은 그 안의 문구를 가져온다. namespace를 생략하면 `t('About.title')`처럼 전체 경로를 사용한다. [Translations](../../raw/frontend/next-intl-translations.md)

```json
{
  "About": {
    "title": "About us"
  }
}
```

```tsx
import {useTranslations} from 'next-intl';

function About() {
  const t = useTranslations('About');
  return <h1>{t('title')}</h1>;
}
```

messages는 중첩 객체로 구성할 수 있다. 키 이름 자체에는 `.`을 넣을 수 없으며, `.`은 중첩 경로를 구분할 때 사용한다. ICU message syntax를 통해 변수 삽입, 복수형, 값에 따른 문구 선택을 표현하고, `t.rich`로 번역 문구의 태그를 React 요소와 연결할 수 있다. [Translations](../../raw/frontend/next-intl-translations.md)

**번역 문구를 표시하는 기능과 번역문을 만드는 작업은 구분한다.** 공식 문서는 준비한 messages를 제공하고 표시하는 흐름을 안내하며, 번역 협업이나 자동화는 별도 관리 도구와 연결한다. 따라서 `next-intl`을 설치했다는 이유만으로 모든 문구의 번역이 완성된다고 볼 수 없다. 이는 공식 문서의 역할 구분에서 도출한 설명이다. [App Router](../../raw/frontend/next-intl-app-router.md); [Request configuration](../../raw/frontend/next-intl-request-configuration.md)

## App Router에서 시작하는 흐름

설치는 다음과 같다. [App Router](../../raw/frontend/next-intl-app-router.md)

```sh
npm install next-intl
```

| 파일 | 역할 |
| --- | --- |
| `messages/en.json` | 언어별 번역 문구를 보관한다. |
| `src/i18n/request.ts` | 요청별 locale과 messages 등 서버 설정을 제공한다. |
| `next.config.ts` | `createNextIntlPlugin()`으로 요청 설정 파일과 next-intl을 연결한다. |
| `app/layout.tsx` | Client Components가 번역을 사용할 경우 `NextIntlClientProvider`로 감싼다. |
| `app/page.tsx` | `useTranslations` 또는 `getTranslations`로 문구를 표시한다. |

[App Router](../../raw/frontend/next-intl-app-router.md)

공식 시작 예제는 먼저 locale을 `en`으로 고정하고 messages를 불러온다. 아래 경로는 `src/i18n/request.ts`에서 프로젝트 루트의 `messages/`를 읽는 구조다. 여러 언어를 지원하려면 이후 사용자 설정이나 routing에서 locale을 정하도록 바꾼다. [App Router](../../raw/frontend/next-intl-app-router.md)

```ts
import {getRequestConfig} from 'next-intl/server';

export default getRequestConfig(async () => {
  const locale = 'en';

  return {
    locale,
    messages: (await import(`../../messages/${locale}.json`)).default
  };
});
```

`getRequestConfig`의 설정은 요청별로 관리된다. Server Components에서 렌더링한 `NextIntlClientProvider`는 locale·messages·표시 형식 등의 설정을 상속하므로, 보통 같은 값을 다시 전달할 필요가 없다. [Request configuration](../../raw/frontend/next-intl-request-configuration.md)

## Server·Client Components에서 사용하는 방법

| 실행 위치 | 기본 사용법 | 확인할 점 |
| --- | --- | --- |
| 일반 React component | `useTranslations('About')` | 서버에서도 사용할 수 있으며, import 경로에 따라 클라이언트에서 실행될 수도 있다. |
| async Server Component | `await getTranslations('About')` | async component에서는 hook 대신 awaitable API를 사용한다. |
| Client Component | 상위 `NextIntlClientProvider`와 `useTranslations` | 필요한 messages를 provider로 전달해야 한다. |

[Server & Client Components](../../raw/frontend/next-intl-server-client-components.md)

번역 때문에 모든 화면을 `'use client'`로 바꿀 필요는 없다. 공식 문서는 서버에서 번역한 문자열을 Client Component의 props나 children으로 전달하는 방식을 우선 제안한다. 클라이언트에서 직접 번역해야 한다면 필요한 messages만 제공하거나 전체 messages를 제공할 수 있다. [Server & Client Components](../../raw/frontend/next-intl-server-client-components.md)

전체 messages 전달은 시작하기 쉽고 많은 앱에서 합리적인 선택이다. 다만 전달한 messages가 페이지의 streaming 응답에 포함되므로 양이 많으면 초기 응답과 상호작용 성능에 영향을 줄 수 있다. 공식 문서는 먼저 측정하고, 필요할 때 전달 범위를 줄이라고 안내한다. [Server & Client Components](../../raw/frontend/next-intl-server-client-components.md)

## 언어별 URL은 필요할 때 추가한다

locale별 URL이 필수는 아니다. 같은 URL에서 cookie나 사용자 설정으로 locale을 정할 수도 있다. 언어별 주소가 필요하면 `/en/about` 같은 prefix 방식이나 언어별 domain 방식으로 구성하고, App Router의 최상위 `[locale]` segment를 사용한다. [App Router](../../raw/frontend/next-intl-app-router.md); [Routing setup](../../raw/frontend/next-intl-routing-setup.md)

routing 설정은 `defineRouting`으로 지원 locale과 기본 locale을 정의하고, `createMiddleware`로 요청 경로를 처리하며, `createNavigation`으로 설정에 맞는 `Link`·`useRouter` 등을 만든다. URL에서 얻은 locale은 지원 목록에 속하는지 검증한다. [Routing setup](../../raw/frontend/next-intl-routing-setup.md)

**버전에 따른 차이:** 현재 수집한 문서는 Next.js 16부터 `middleware.ts` 대신 `proxy.ts`라는 파일명을 사용한다고 설명한다. locale을 읽는 방식은 `next/root-params`를 안내하며, Next.js 16.3 이상에서 기본 제공되고 이전 버전에서는 `experimental.rootParams` 설정이 필요하다고 명시한다. `setRequestLocale`은 호환성을 위해 유지되는 legacy API로 분류한다. 기존 예제를 그대로 복사하기 전에 프로젝트 버전과 정적 렌더링 방식을 대조해야 한다. [Routing setup](../../raw/frontend/next-intl-routing-setup.md)

## 번역 누락과 오류 처리

번역 키를 찾지 못하거나 형식 처리가 실패하면 기본적으로 console에 오류를 출력하고 `${namespace}.${key}`를 표시한다. `onError`로 오류 처리를, `getMessageFallback`으로 오류 시 표시할 문구를 바꿀 수 있다. 다른 언어의 번역을 폴백으로 사용하려면 기본 locale의 messages와 현재 locale의 messages를 로딩 단계에서 병합하는 방식이 가능하다. `getMessageFallback` 자체를 다른 locale의 번역을 자동 조회하는 기능으로 이해하면 안 된다. [Request configuration](../../raw/frontend/next-intl-request-configuration.md)

`onError`와 `getMessageFallback`은 함수이므로 서버 설정에서 Client Components로 자동 상속되지 않는다. 클라이언트에도 필요하다면 `'use client'`인 provider wrapper 안에서 함수를 정의한다. [Request configuration](../../raw/frontend/next-intl-request-configuration.md)

Client Component에서 provider 관련 오류가 나면 상위에 `NextIntlClientProvider`가 있는지, 서버에서 실행된다고 생각한 component가 실제로 클라이언트 import 경로에 포함된 것은 아닌지 확인한다. 또한 `t.raw`와 `dangerouslySetInnerHTML`로 HTML을 표시한다면 원문을 sanitize해야 한다. [Server & Client Components](../../raw/frontend/next-intl-server-client-components.md); [Translations](../../raw/frontend/next-intl-translations.md)

## DB 콘텐츠 번역과의 관계

UI 버튼·오류 문구를 messages로 관리하는 것과 DB의 게시글·상품 설명을 언어별로 저장·조회하는 것은 별도 설계 문제다. next-intl은 원격 messages를 불러올 수 있지만, 번역 테이블 구조나 발행·재검토 상태까지 정해주지는 않는다. 이 구분은 next-intl의 messages 로딩 기능과 기존 DB 다국어 문서를 함께 해석한 설계 판단이다. [Request configuration](../../raw/frontend/next-intl-request-configuration.md); [DB 콘텐츠 다국어화 저장 구조 선택](../database/multilingual-content-schema-design.md)

## See Also

- [HTTP 307 redirect와 rewrite의 네트워크 흐름](redirect-vs-rewrite.md)
- [DB 콘텐츠 다국어화 저장 구조 선택](../database/multilingual-content-schema-design.md)
- [Next.js App Router에서 canonical과 페이지별 metadata 구현](../seo/nextjs-app-router-canonical-and-page-metadata.md)
