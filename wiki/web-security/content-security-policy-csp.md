# Content Security Policy (CSP)

> Sources: MDN contributors, Unknown
> Raw: [컨텐츠 보안 정책 (CSP)](../../raw/web-security/content-security-policy-csp.md)
> Updated: 2026-08-24

## Overview

Content Security Policy(CSP)는 브라우저가 페이지에서 불러오거나 실행할 수 있는 콘텐츠의 출처를 서버가 제한하는 추가 보안 계층이다. 교차 사이트 스크립팅(XSS)과 데이터 주입 공격의 실행 경로를 줄이고 위반을 보고하는 데 쓰이며, 리소스 종류별 허용 출처를 선언하는 정책 지시문(directive)으로 구성한다.

## CSP가 줄이는 위협

CSP의 핵심은 브라우저가 실행 가능한 스크립트의 신뢰 출처를 명시하는 것이다. 허용 목록 밖의 스크립트, 인라인 스크립트와 이벤트 처리기 등을 제한하면 서버 응답에 악성 코드가 섞이더라도 브라우저에서 실행될 가능성을 줄일 수 있다. 필요하면 스크립트 실행을 전역적으로 허용하지 않는 정책도 구성할 수 있다.

콘텐츠를 가져올 도메인뿐 아니라 사용할 프로토콜도 제한할 수 있다. 다만 CSP는 전송 보안 전체를 대신하지 않는다. HTTPS 적용, 쿠키의 `secure` 속성, HTTP에서 HTTPS로의 리디렉션, `Strict-Transport-Security` 같은 조치와 함께 사용해야 한다.

## 정책 전달 방식

일반적으로 웹 서버가 HTTP 응답에 다음 헤더를 반환한다.

```http
Content-Security-Policy: policy
```

HTML의 `<meta http-equiv="Content-Security-Policy">` 요소로도 정책을 선언할 수 있지만, 위반 보고서 전송 같은 일부 기능은 HTTP 헤더에서만 사용할 수 있다. 과거의 `X-Content-Security-Policy` 헤더는 더 이상 설정할 필요가 없다.

## 정책 작성 원리

각 지시문은 스크립트, 스타일, 이미지, 미디어, 프레임, 글꼴, 작업자(worker) 등 특정 리소스 유형이나 정책 영역을 담당한다.

- `default-src`는 별도 정책이 없는 리소스 유형에 적용되는 폴백(fallback)이다.
- `script-src`는 스크립트 출처를 제한한다. 인라인 스크립트와 `eval()` 실행을 막으려면 `default-src` 또는 `script-src` 정책이 필요하다.
- `style-src`는 `<style>` 요소와 `style` 속성 등 인라인 스타일의 적용을 제한한다.
- `'self'`는 현재 문서와 같은 출처를 뜻하고, `'none'`은 해당 리소스를 허용하지 않는 정책에 사용한다.

가장 단순한 같은 출처 정책은 다음과 같다.

```http
Content-Security-Policy: default-src 'self'
```

리소스 유형별로 출처를 나누려면 지시문을 세미콜론으로 연결한다.

```http
Content-Security-Policy: default-src 'self'; img-src *; media-src example.org example.net; script-src userscripts.example.com
```

이 예시는 기본적으로 같은 출처의 콘텐츠만 허용하되, 이미지는 모든 출처에서, 미디어는 지정한 두 도메인에서, 스크립트는 지정한 서버에서만 가져오도록 예외를 둔다.

## 안전한 배포와 테스트

새 정책은 먼저 보고 전용 모드로 관찰할 수 있다.

```http
Content-Security-Policy-Report-Only: policy
```

`Content-Security-Policy-Report-Only`는 위반 보고서를 만들지만 정책을 시행하지 않는다. 같은 응답에 `Content-Security-Policy`도 있으면 보고 전용 정책과 시행 정책이 모두 적용되며, 후자는 실제로 콘텐츠를 차단한다. 따라서 기존 사이트에서는 보고 전용 정책으로 예상 위반을 수집하고 필요한 출처를 정리한 뒤 시행 정책으로 전환하는 방식이 적합하다.

위반 보고는 기본적으로 전송되지 않는다. 보고를 받으려면 `report-to` 지시문으로 수신 위치를 지정하고 서버에서 보고 데이터를 처리해야 한다.

```http
Content-Security-Policy: default-src 'self'; report-to http://reportcollector.example.com/collector.cgi
```

## 위반 보고서 해석

위반 보고서에는 차단된 리소스의 URI인 `blocked-uri`, 정책이 보고용인지 시행용인지 나타내는 `disposition`, 위반 문서의 `document-uri`, 실제 위반을 일으킨 `effective-directive`, 원래 정책인 `original-policy`, HTTP 상태를 나타내는 `status-code` 등이 포함될 수 있다.

차단된 리소스가 위반 문서와 다른 출처라면 `blocked-uri`는 민감한 정보 유출을 막기 위해 전체 경로 대신 스키마, 호스트와 포트까지만 포함될 수 있다. 브라우저에 따라 실제 적용 지시문보다 더 세분화된 값을 `effective-directive`에 제공할 수도 있으므로 보고서를 해석할 때 구현 차이를 고려해야 한다.

## 운영 시 주의점

- CSP는 허용 출처를 넓게 잡을수록 방어 효과가 약해진다. 실제로 필요한 리소스 유형과 출처를 구분해 정책을 작성한다.
- 보고 전용 정책은 관찰 수단이지 차단 수단이 아니다. 최종적으로 시행 헤더가 필요하다.
- 일부 Safari 버전에는 CSP 헤더와 동일 출처 처리에 관한 비호환성이 보고되어 있으므로 대상 브라우저에서 정책과 위반 보고를 함께 검증한다.
