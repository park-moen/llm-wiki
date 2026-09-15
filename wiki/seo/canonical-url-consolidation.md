# canonical URL: 중복 페이지의 대표 주소 정하기

> Sources: Google Search Central, 2026-07-15; J. Ohye and J. Kupke, 2012-04
> Raw: [rel="canonical" 및 다른 메서드로 표준 URL을 지정하는 방법](../../raw/seo/2026-07-15-google-search-canonical-url-consolidation.md); [RFC 6596: The Canonical Link Relation](../../raw/seo/2012-04-rfc-6596-canonical-link-relation.md)
> Updated: 2026-09-08

## Overview

같은 페이지가 여러 주소로 열리면 Google은 어떤 주소를 검색 결과에 보여줄지 판단해야 한다. 예를 들어 쿼리 매개변수가 붙은 주소, `www` 유무, HTTP와 HTTPS 주소가 같은 내용을 보여줄 수 있다. canonical URL은 이때 "이 주소를 대표 주소로 봐 주세요"라고 Google에 알려 주는 URL이다.

## 먼저 알아둘 두 가지

- **canonical URL**은 대표로 정한 주소다. 예: `https://example.com/products/green-dress`
- **canonical tag**는 그 대표 주소를 페이지에 적는 HTML 요소다. `rel="canonical"`이라고도 부른다.

canonical tag는 Google에 보내는 **선호 표시**다. Google은 tag뿐 아니라 리디렉션, 사이트맵, 내부 링크 등 여러 정보를 보고 최종 대표 URL을 고른다. 따라서 서로 다른 곳에서 서로 다른 주소를 가리키면 안 된다.

## 가장 기본적인 설정

중복 페이지의 `<head>` 안에 대표 URL을 적는다.

```html
<link rel="canonical" href="https://www.example.com/products/green-dress" />
```

대표 페이지 자신도 같은 tag를 넣는다. 이를 **자체 참조 canonical**이라고 한다. 상대 경로보다 `https://`부터 적는 절대 URL을 사용하고, JavaScript가 tag 값을 바꾸지 않게 한다.

## Search Console에서 canonical 문제가 보일 때

아래 순서대로 같은 대표 URL을 가리키는지 확인한다.

1. **대표 주소 하나를 정한다.** HTTPS 여부, `www` 유무, 경로, trailing slash, 쿼리 매개변수 처리 방식을 먼저 정한다.
2. **중복 페이지에 canonical tag를 넣는다.** 대표 주소를 절대 URL로 적고, 페이지마다 하나만 선언한다.
3. **더 이상 쓸 주소는 영구 리디렉션한다.** 리디렉션 대상도 대표 주소와 같아야 한다.
4. **사이트맵과 내부 링크를 고친다.** 중복 주소가 아니라 대표 주소만 넣는다.

`robots.txt`, URL 삭제 도구, `noindex`는 canonical의 대체 수단이 아니다. 특히 `noindex`는 페이지를 Google 검색에서 완전히 막을 수 있다.

## 자주 만나는 중복 주소

| 상황 | 대표 주소를 정한 뒤 할 일 |
| --- | --- |
| 쿼리 매개변수가 붙은 주소 | 원래 페이지를 대표 URL로 정하고, 매개변수 주소에 canonical tag를 넣는다. |
| `www`와 non-`www`가 모두 열림 | 하나를 대표로 정하고 다른 쪽을 대표 주소로 리디렉션한다. |
| HTTP와 HTTPS가 모두 열림 | HTTPS를 대표로 정하고 HTTP에서 HTTPS로 리디렉션한다. |
| URL 끝의 `/` 유무가 다름 | 하나의 형식을 정하고 내부 링크·사이트맵·canonical tag를 모두 맞춘다. |

대표 URL은 현재 페이지와 같은 내용이거나 현재 페이지의 내용을 모두 포함해야 한다. 서로 다른 제품·글처럼 내용이 다른 페이지를 억지로 하나의 canonical URL로 묶으면 안 된다.

## 이런 설정은 피한다

- 한 페이지에 canonical tag를 여러 개 넣는 경우
- 사이트맵, canonical tag, 리디렉션이 서로 다른 대표 주소를 가리키는 경우
- 대표 주소가 영구 리디렉션의 시작점이거나 HTTP 4xx 오류를 반환하는 경우
- `page-2`를 `page-1`의 canonical로 지정하는 경우처럼, 대표 페이지가 현재 페이지의 내용을 담지 않는 경우
- URL fragment를 대표 URL로 쓰는 경우

잘못된 대상이나 canonical chain을 설정하면 검색엔진이 canonical 설정을 무시하고 자체 판단을 할 수 있다.

## 심화: 특수한 경우

### 여러 페이지로 나눈 글

`page-1`, `page-2`처럼 글을 나누었다면 모든 내용을 포함한 view-all 페이지를 대표 URL로 둘 수 있다. 다만 읽기 어려워지거나 로딩 시간이 길어질 수 있으므로 사용자 경험도 함께 고려한다.

### 다국어 페이지

`hreflang`을 사용한다면 같은 언어의 페이지를 canonical URL로 지정한다. 언어·국가별 페이지를 서로 연결하는 일은 `hreflang`이 담당한다.

### PDF 같은 HTML이 아닌 파일

HTML `<head>`에 tag를 넣을 수 없으므로 HTTP 응답 헤더를 사용한다.

```http
Link: <https://www.example.com/downloads/white-paper.pdf>; rel="canonical"
```

RFC 6596은 상대 URL도 허용하지만, Google은 장기적인 운영 문제를 줄이기 위해 절대 URL을 권장한다. 실무에서는 Google 권장사항을 따른다.

## 핵심만 기억하기

1. 같은 내용의 여러 주소 중 대표 주소를 하나 정한다.
2. 중복 페이지의 `<head>`에 그 주소를 가리키는 canonical tag를 넣는다.
3. 리디렉션·사이트맵·내부 링크도 모두 같은 대표 주소로 맞춘다.

## See Also

- [Next.js App Router에서 canonical과 페이지별 metadata 구현](nextjs-app-router-canonical-and-page-metadata.md) — middleware 기반 자체 참조 canonical과 production 응답 검증 사례
- [Content Security Policy (CSP)](../web-security/content-security-policy-csp.md) — HTTPS 전환과 HSTS가 함께 언급되는 웹 보안 운영 맥락
