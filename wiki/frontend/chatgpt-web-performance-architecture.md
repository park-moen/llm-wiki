# ChatGPT 웹의 성능 중심 아키텍처

> Sources: emewjin, 2026-09-02
> Raw: [ChatGPT 웹 리버스 엔지니어링 번역](../../raw/frontend/2026-09-02-chatgpt-web-reverse-engineering.md)
> Updated: 2026-09-08

## Overview

ChatGPT 웹은 로그인하지 않은 신규 사용자가 최대한 빨리 입력하고 첫 답변을 받게 하는 목표를 중심으로 설계되어 있다. 관찰된 구조는 서버 렌더링한 셸, 입력 가능 시점을 우선하는 점진적 로딩, 서버에서 평가한 기능 플래그, 미리 실행하는 악용 방지 검사와 서버 전송 이벤트(SSE) 스트리밍을 결합한다. 이 문서의 구현 세부 사항은 OpenAI 공식 아키텍처 문서가 아니라 공개된 HTML·JavaScript·CSS·네트워크 요청을 분석한 게시 시점의 외부 관찰 결과다.

## 제품 목표가 아키텍처를 결정한다

핵심 사용자 흐름은 낯선 사용자가 `chatgpt.com`을 열고 계정 생성이나 로그인 없이 질문을 보내는 과정이다. 이 목표에서는 초기 JavaScript를 모두 받은 뒤 화면을 그리는 클라이언트 사이드 렌더링보다, 실제 입력 화면을 HTML로 먼저 보내는 서버 사이드 렌더링이 유리하다. 대신 서버와 브라우저 양쪽의 실행 환경, 하이드레이션, 캐시와 렌더링 동작을 함께 다뤄야 하는 복잡성을 감수한다.

리버스 엔지니어링 결과에 따르면 ChatGPT 웹은 React 19와 React Router 7 프레임워크 모드, Vite를 사용해 스트리밍 서버 렌더링을 구성한다. 클라이언트 상태에는 TanStack Query를 사용하고, 스타일은 Tailwind CSS와 의미 기반 디자인 token을 조합한다. 메뉴와 팝오버 같은 UI는 Radix UI primitive, 입력창은 ProseMirror, 답변의 코드 블록은 CodeMirror 6을 활용한다.

## 서버 렌더링과 점진적 부팅

ChatGPT는 2022년 11월 30일 Next.js 12 Pages Router 앱으로 출시된 뒤, 2024년에 Remix를 거쳐 React Router 7로 이동했다. 현재 관찰된 문서의 `window.__reactRouterContext`에는 `ssr: true`와 `isSpaMode: false`가 들어 있으며, 로그아웃 화면의 셸도 서버가 실제 HTML로 렌더링한다.

분석자가 측정한 로그아웃 문서는 압축 상태에서 84KB였고, 가까운 Cloudflare edge에서 첫 바이트까지 약 50~65ms가 걸렸다. 응답에는 약 30KB의 앱 셸 마크업과 필요한 스타일이 포함되며, 하이드레이션 뒤의 DOM node는 548개였다. 이 수치는 보편적인 성능 보장이 아니라 특정 위치와 시점에서 얻은 관찰값이다.

서버는 React의 스트리밍 렌더링으로 기본 셸을 먼저 전송하고, 준비된 Suspense 영역과 route loader data를 이어서 보낸다. 서버가 `shouldPrefetchModels`, `shouldPrefetchHistory`, `shouldPrefetchStarredConversations` 같은 값을 내려 사용자별 prefetch 계획도 결정한다. 브라우저는 서버가 정한 초기 계획을 실행하고, TanStack Query는 서버가 제공한 초기값을 이어받는다.

## 입력 가능 시점을 우선하는 리소스 전략

부팅 과정의 우선순위는 전체 기능을 한꺼번에 준비하는 것이 아니라 입력창을 먼저 사용할 수 있게 만드는 것이다. `deferStartupImportsUntilComposerTTFI` 플래그는 composer가 상호작용 가능한 상태가 될 때까지 시작 단계의 import를 미룬다. 콜드 로딩에서 JavaScript chunk가 100개 넘게 필요하더라도 문서는 진입점과 React vendor chunk, 현재 route에 필요한 module 등 핵심 파일 14개만 module preload한다.

대화 화면의 작은 core는 `conversation-small` chunk로 여러 route에서 재사용한다. code block, chain-of-thought 관련 view 등은 실제 답변에 필요할 때 불러온다. JavaScript, stylesheet와 font는 대부분 `chatgpt.com/cdn/assets`라는 first-party origin에서 30일 cache header와 함께 제공해 별도 DNS 조회와 TLS 연결을 줄인다. 서비스 worker는 사용하지 않고 일반 HTTP cache에 의존한다.

스타일도 같은 원리를 따른다. route와 기능 단위로 CSS를 나누고, UI 본문에는 system font를 사용한다. theme은 의미 기반 CSS variable로 표현하며, 저장된 밝기·색상 설정은 첫 paint 전에 실행되는 inline script가 `html` 요소에 적용한다. 이 방식은 잘못된 theme이 잠깐 보이는 현상을 피하면서 component를 다시 렌더링하지 않고 theme을 바꿀 수 있게 한다.

## 스트리밍 답변은 별도의 렌더링 문제다

사용자가 메시지를 보내면 응답은 POST `fetch` 요청의 서버 전송 이벤트(SSE) stream으로 도착한다. 미리 렌더링한 셸에 token을 받는 즉시 표시하므로, 제출 뒤의 대기 경로에는 주로 model 응답 시간만 남는다.

도착 중인 Markdown은 닫히지 않은 code fence, 미완성 table, 짝이 없는 강조 marker처럼 불완전할 수 있다. ChatGPT 웹은 누적되는 메시지를 계속 parse하고 다시 렌더링하면서도 layout이 흔들리지 않게 처리해야 한다. code block을 CodeMirror editor로 만들고 수식을 KaTeX의 시각 표현과 MathML로 함께 제공하는 것도 답변 자체를 제품의 핵심 렌더링 대상으로 다룬 결과다.

## 서버 평가 기능 플래그와 실험

관찰 당시 서버 HTML에는 377KB의 bootstrap JSON이 포함되어 있었고, 그 안에 기능 gate 556개, 동적 설정 144개, 실험 layer 192개가 들어 있었다. 서버가 익명 ID와 지역을 바탕으로 플래그를 미리 평가하므로, browser는 별도 요청을 기다리거나 기본 UI를 그렸다가 뒤집을 필요가 없다.

플래그는 기능 노출뿐 아니라 `promoteCss`, `stripModulepreloadImports`처럼 로딩 전략도 제어한다. 실험 event는 Statsig의 third-party endpoint가 아니라 `chatgpt.com/ces/v1/`으로 보내 광고 차단기로 인해 측정 대상이 빠질 가능성을 낮춘다. 읽을 수 있는 기능 이름 대신 hash를 사용하는 것은 client bundle에서 출시 전 계획이 드러나는 일을 줄인다.

## 익명 사용을 가능하게 하는 보이지 않는 보호 계층

로그인 없는 접근은 마찰을 줄이지만 자동화된 악용과 무료 추론 비용을 키운다. 이를 막기 위해 페이지가 로드될 때 Cloudflare challenge가 실행되고, OpenAI의 Sentinel SDK가 sandbox iframe에서 browser 환경 정보를 수집한다고 분석됐다. 앱은 사용자가 입력하는 동안 `chat-requirements` 검사를 준비하고 완료해, Enter를 누른 뒤 보호 검사 때문에 생기는 지연을 줄인다.

별도의 `/backend-anon` API 영역은 익명 사용자에게도 ID, 속도 제한과 실험을 적용한다. 즉, 인증을 먼저 끝내고 화면을 여는 방식이 아니라 화면을 먼저 제공하면서 사용자에게 보이지 않는 곳에서 보호 절차를 앞당긴다. 빠른 진입 경험과 악용 방지는 서로 반대되는 선택지가 아니라 실행 시점을 조정해 함께 달성하는 목표다.

## 재사용할 수 있는 설계 원칙

- 제품의 가장 중요한 첫 행동을 정하고, resource loading과 측정 지표를 그 행동에 맞춘다.
- 화면에 필요한 HTML과 CSS를 먼저 보내고 부가 기능은 실제 사용 시점까지 미룬다.
- theme과 기능 플래그처럼 첫 화면을 바꾸는 결정은 paint와 hydration 전에 확정한다.
- data, JavaScript, CSS와 font를 가능한 한 같은 origin에서 제공해 연결 비용을 줄인다.
- 사용자가 다른 일을 하는 동안 prefetch와 보안 검사를 수행해 제출 뒤의 critical path를 짧게 만든다.
- 기성 library를 조합하더라도 route 분할, cache, streaming과 측정을 제품 목표에 맞게 세밀하게 조정한다.

## 해석할 때 주의할 점

이 구조는 공개 자산과 network trace를 바탕으로 추론한 결과다. 내부 source code, 운영 정책과 server architecture 전체를 확인한 것은 아니며, 실험에 따라 사용자별 결과가 달라질 수 있다. chunk 수, bootstrap 크기, 기능 플래그 수와 endpoint는 배포 시점마다 바뀔 수 있으므로 영구적인 제품 사양이 아니라 관찰 시점의 evidence로 사용해야 한다.
