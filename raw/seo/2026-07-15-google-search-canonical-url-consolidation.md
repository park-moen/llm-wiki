# rel="canonical" 및 다른 메서드로 표준 URL을 지정하는 방법

> Source: https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls?hl=ko
> Collected: 2026-09-08
> Published: Unknown

Google 검색에서 중복되거나 매우 비슷한 페이지의 표준 URL을 지정할 때, 여러 가지 방법을 사용해 자신이 선호하는 표준 URL이 무엇인지 나타낼 수 있습니다. 다음 방법을 사용해 자신이 선호하는 표준 URL을 알릴 수 있으며, 가장 큰 영향을 미치는 방법부터 순서대로 나열되어 있습니다.

- 리디렉션: 리디렉션 대상이 표준 URL이 되어야 하는 강력한 신호입니다.
- `rel="canonical"` `link` 주석: 지정된 URL이 표준 URL이 되어야 하는 강력한 신호입니다.
- 사이트맵 포함: 사이트맵에 포함된 URL이 표준이 되도록 하는 약한 신호입니다.

이러한 방법들은 중첩하여 사용할 수 있으므로 함께 사용하면 더 효과적입니다. 즉, 두 가지 이상의 방법을 사용하면 검색 결과에 선호하는 표준 URL이 표시될 가능성이 높아집니다. 표준 URL을 지정하지 않아도 Google에서 어떤 버전의 URL이 Google 검색에서 사용자에게 표시하기에 가장 적합한 버전인지 식별합니다.

## 표준 URL을 지정해야 하는 이유

- 어떤 URL이 검색 결과에서 사람들에게 표시될지 지정합니다.
- 유사하거나 중복된 페이지와 관련된 신호를 선호하는 단일 URL로 통합합니다.
- 단일 콘텐츠와 관련된 측정항목의 추적을 단순화합니다.
- Googlebot이 같은 콘텐츠의 중복된 버전을 모두 크롤링하는 데 시간을 낭비하지 않도록 합니다.

## 권장사항

- 표준화를 목적으로 `robots.txt` 파일을 사용하면 안 됩니다. Google은 콘텐츠 없이 `robots.txt`에서 허용되지 않는 URL의 색인을 생성할 수 있습니다.
- 표준화를 위해 URL 삭제 도구를 사용해서는 안 됩니다. 이 도구를 사용하면 Google 검색에서 모든 버전의 URL을 숨깁니다.
- 다른 표준화 기술을 사용하여 서로 다른 URL을 같은 페이지의 표준 URL로 지정하면 안 됩니다. 예를 들어 사이트맵에서 URL 하나를 지정하고 `rel="canonical"`을 사용하여 같은 페이지에 다른 URL을 지정하면 안 됩니다.
- Google은 일반적으로 URL 프래그먼트를 지원하지 않으므로 URL 프래그먼트를 표준으로 지정하지 마세요.
- 표준 페이지 자체(자체 참조 표준)에 `rel="canonical"` 링크를 포함하세요.
- `noindex`를 사용하여 단일 사이트 내에서 표준 페이지가 선택되지 않도록 하면 페이지가 Google 검색에서 완전히 차단될 수 있으므로 권장하지 않습니다. `rel="canonical"` `link` 주석을 사용하는 것이 좋습니다.
- `hreflang` 요소를 사용한다면 같은 언어로 표준 페이지를 지정하세요. 같은 언어의 표준 페이지가 존재하지 않는 경우 가장 유사한 언어로 된 표준 페이지를 지정하세요.
- 사이트 내에서 연결할 때는 중복 URL이 아닌 표준 URL에 연결하세요.
- JavaScript로 클라이언트 측 렌더링을 사용하는 경우 HTML 소스 코드에서 표준 URL을 지정하고 JavaScript가 표준 링크 요소를 변경하지 않도록 하는 것이 좋습니다.

## 표준화 방법 비교

### `rel="canonical"` `link` 요소

표준 페이지로 연결되는 모든 중복 페이지의 코드에 `<link>` 요소를 추가합니다. 무한히 많은 중복 페이지를 매핑할 수 있지만, 용량이 큰 사이트 또는 URL이 자주 변경되는 사이트의 매핑을 유지하기가 복잡할 수 있습니다. HTML 페이지에만 작동하며 PDF와 같은 파일에는 작동하지 않습니다. 이 경우 `rel="canonical"` HTTP 헤더를 사용할 수 있습니다.

### `rel="canonical"` HTTP 헤더

페이지 응답에 `rel="canonical"` 헤더를 전송합니다. 페이지 크기가 커지지 않고 무한히 많은 중복 페이지를 매핑할 수 있지만, 용량이 큰 사이트 또는 URL이 자주 변경되는 사이트의 매핑을 유지하기가 복잡할 수 있습니다.

### 사이트맵

사이트맵에서 표준 페이지를 지정합니다. 특히 용량이 큰 사이트에서 쉽게 구현하고 관리할 수 있는 방법입니다. 다만 Google에서 사이트맵에서 선언된 표준 페이지와 관련된 중복 페이지가 어떤 것인지 판단해야 하며, `rel="canonical"` 매핑 방법에 비해 Google에 덜 강력한 신호를 줍니다.

### 리디렉션

영구 리디렉션을 사용하여 Google에 리디렉션된 URL이 리디렉션 대상 URL보다 더 낮은 버전임을 알립니다. 이 방법은 중복 페이지를 더 이상 사용하지 않는 경우에만 사용하세요.

## `rel="canonical"` `link` 주석 사용

Google에서는 RFC 6596에 설명된 것과 같이 명시적인 `rel` canonical `link` 주석을 지원합니다. 페이지의 대체 버전을 제안하는 `rel="canonical"` 주석은 무시됩니다. 구체적으로는 `hreflang`가 있는 `rel="canonical"` 주석입니다. `lang`, `media`, `type` 속성은 표준화에 사용되지 않습니다. 언어 및 국가 주석에는 `link rel="alternate" hreflang`을 사용하세요.

`rel="canonical"` `link` 주석은 다음 두 방법으로 제공할 수 있습니다.

- HTML 내의 `rel="canonical"` `link` 요소
- `rel="canonical"` `link` HTTP 헤더

두 방법 중 하나를 선택한 다음 지속적으로 사용하는 것이 좋습니다. 두 방법을 동시에 사용하는 것도 지원되기는 하지만 오류가 발생하기 쉽습니다. 예를 들어 HTTP 헤더에 한 URL을 제공하고 `rel="canonical"` `link`에 다른 URL을 제공하는 경우 오류가 발생합니다.

`rel="canonical"` `link` 요소는 HTML의 `<head>` 섹션에 사용되며, 페이지에 있는 콘텐츠를 대표하는 다른 페이지가 있음을 나타냅니다. 중복 페이지의 `<head>` 섹션에 표준 페이지를 가리키는 요소를 추가합니다.

```html
<link rel="canonical" href="https://example.com/dresses/green-dresses" />
```

표준 페이지 자체에도 동일한 자체 참조 `rel="canonical"` 링크 요소를 추가하는 것이 좋습니다. 표준 페이지에 별도의 URL에 있는 모바일 변형이 있는 경우 모바일 버전의 페이지를 가리키는 `rel="alternate"` `link` 요소를 해당 페이지에 추가하고, 페이지에 적합한 `hreflang` 또는 기타 요소를 추가합니다.

`rel="canonical"` `link` 요소에는 상대 경로보다 절대 경로를 사용하세요. 상대 경로도 Google에서 지원하지만 장기적으로 문제가 발생할 수 있으므로 권장되지 않습니다. `rel="canonical"` `link` 요소는 HTML의 `<head>` 섹션에 포함되어 있는 경우에만 허용됩니다.

## `rel="canonical"` HTTP 헤더

서버 구성을 변경할 수 있다면 HTML 요소 대신 `rel="canonical"` 타겟 속성이 있는 `link` HTTP 응답 헤더를 사용하여 PDF 파일 등 HTML이 아닌 문서의 표준 URL을 표시할 수 있습니다. Google은 웹 검색 결과에만 이 방법을 지원합니다.

각 자체 URL에 PDF나 Microsoft Word와 같은 여러 파일 형식으로 콘텐츠를 게시하는 경우 `rel="canonical"` HTTP 헤더를 반환하여 Googlebot에 HTML이 아닌 파일의 표준 URL을 알릴 수 있습니다.

```http
HTTP/1.1 200 OK
Content-Length: 19
Link: <https://www.example.com/downloads/white-paper.pdf>; rel="canonical"
```

`rel="canonical"` HTTP 헤더에도 절대 URL을 사용하세요.

## 사이트맵과 리디렉션 사용

각 페이지의 표준 URL을 선택하고 이를 사이트맵을 통해 제출합니다. 사이트맵에 명시된 모든 페이지는 표준 페이지로 제안됩니다. 중복 페이지가 있는 경우 Google이 콘텐츠의 유사성을 기준으로 어떤 페이지가 중복인지 판단합니다.

기존의 중복 페이지를 폐기하고 싶은 경우 리디렉션을 사용하세요. 모든 영구 리디렉션 방법이 Google 검색에 미치는 영향은 동일하지만, 검색엔진에서 각각 다른 리디렉션 방법을 알아차리는 데 걸리는 시간은 서로 다를 수 있습니다. 가장 빠르게 효과를 내려면 HTTP(서버 측이라고도 함) 리디렉션을 사용하세요.

## 기타 신호

Google은 명시적으로 제공되는 방법 외에도 일반적으로 사이트 설정을 기반으로 하는 표준화 신호 집합(HTTP보다 HTTPS 및 `hreflang` 클러스터의 URL 선호)을 사용합니다.

Google은 다음과 같은 문제나 충돌하는 신호가 있는 경우가 아니라면 HTTP 페이지보다 HTTPS 페이지를 표준 페이지로 선호합니다.

- HTTPS 페이지에 잘못된 SSL 인증서가 있습니다.
- HTTPS 페이지에 보안이 취약한 종속 항목(이미지 제외)이 있습니다.
- HTTPS 페이지에서 사용자를 HTTP 페이지로 또는 HTTP 페이지를 통해 리디렉션합니다.
- HTTPS 페이지에 HTTP 페이지로 연결되는 `rel="canonical"` `link`가 있습니다.

HTTP 페이지에서 HTTPS 페이지로 연결되는 리디렉션을 추가하고, HTTP 페이지의 `rel="canonical"` `link`를 HTTPS 페이지에 추가하며, HSTS를 구현하면 이 선호도를 강화할 수 있습니다. 잘못된 TLS/SSL 인증서, HTTPS에서 HTTP로의 리디렉션, 사이트맵의 HTTP 버전 또는 `hreflang` 주석, 잘못된 호스트 변형과 관련된 SSL/TLS 인증서는 피하세요.

Google에서는 사이트 현지화 작업을 도울 수 있도록 표준화 과정에서 `hreflang` 클러스터에 포함된 URL을 선호합니다.

최종 업데이트: 2026-07-15(UTC)
