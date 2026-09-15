# 디자이너 없는 팀을 위한 AI 디자인 레퍼런스 가이드

> Sources: [개발자를 위한 한국어 UX 도서 근거 가이드](developer-ux-book-evidence-guide.md); [gstack으로 AI 개발 Workflow 이해하기](../ai-agents/gstack-ai-engineering-workflow.md); [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](../ai-agents/superpowers-agent-management-and-spec-driven-development.md)
> Archived: 2026-09-09

## Overview

디자이너가 없는 팀에서 Claude의 디자인 기능은 초안 생성, 대안 탐색, 구현과 일관성 검토를 크게 보조할 수 있다. 그러나 사용자 문제 정의, 정보 우선순위, 브랜드 방향, 접근성 판단과 최종 책임까지 자동으로 대신하지는 않는다. Pinterest 하나에 의존하기보다 **실제 제품 흐름, 화면 패턴, 디자인 시스템, 시각적 분위기와 접근성 기준**을 서로 다른 출처에서 모아 Claude에 명시적인 근거로 제공하는 방식이 현실적이다.

## 결론: Pinterest만 볼 필요는 없다

Pinterest는 색감, 분위기, 일러스트와 전체적인 인상을 빠르게 모으는 moodboard에는 유용하다. 반면 이미지가 실제 제품 흐름에서 분리돼 있어 다음 질문에 답하기 어렵다.

- 사용자는 이 화면에 어떻게 들어왔는가?
- 입력 오류나 빈 데이터는 어떻게 처리하는가?
- 가입, 결제와 설정 변경은 몇 단계인가?
- desktop과 mobile에서 정보 구조가 어떻게 달라지는가?
- 이 component를 keyboard와 screen reader로 사용할 수 있는가?

따라서 Pinterest는 **시각적 방향을 찾는 보조 도구**로 두고, 제품 UI는 실제 screen과 flow를 모은 서비스를 먼저 참고하는 편이 좋다.

## 용도별 레퍼런스 지도

| 필요한 것 | 우선 볼 곳 | 적합한 이유 |
|---|---|---|
| 실제 mobile·web 제품 화면 | [Mobbin](https://mobbin.com/), [Refero](https://refero.design/) | 실제 제품의 screen, pattern과 flow를 기능별로 탐색하기 좋다. 두 서비스 모두 AI 도구와 연결하는 방법도 제공한다. |
| 가입·결제·검색 같은 사용자 흐름 | [Page Flows](https://pageflows.com/) | 개별 화면만이 아니라 screen recording과 단계별 flow를 볼 수 있어 interaction을 이해하기 좋다. |
| SaaS dashboard·설정·가격·onboarding | [SaaSFrame](https://www.saasframe.io/) | SaaS website, product interface와 email을 page 유형과 flow로 나눠 찾을 수 있다. |
| Landing page와 시각적 방향 | [Lapa Ninja](https://www.lapa.ninja/), [Landbook](https://land-book.com/) | Hero, typography, section 구성과 marketing page의 분위기를 비교하기 좋다. |
| Web product의 구현 가능한 기준 | [Carbon Design System](https://carbondesignsystem.com/), [Material Design](https://m3.material.io/) | component, pattern, 사용 원칙과 code 자산을 함께 확인할 수 있다. |
| Apple platform 관례 | [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines) | iOS·iPadOS·macOS의 foundation, pattern, component와 input 관례를 확인할 수 있다. |
| 접근 가능한 interaction | [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) | dialog, tabs, combobox 같은 widget의 role, state와 keyboard interaction 예제가 있다. |
| UX 원칙의 짧은 복습 | [Laws of UX](https://lawsofux.com/) | 선택지, 인지 부하, 익숙한 pattern과 시각적 grouping을 설명하는 개념을 빠르게 찾기 좋다. |

디자이너가 아닌 개발자라면 처음부터 모든 서비스를 구독할 필요는 없다. 아래 조합이면 충분하다.

```text
실제 화면: Mobbin 또는 Refero 하나
+ 사용자 흐름: Page Flows
+ 구현 기준: Carbon 또는 Material Design 하나
+ 접근성: WAI-ARIA APG
+ Landing page가 필요할 때만 Lapa Ninja
```

## 무엇을 만들지에 따라 출처를 바꾼다

### 사내 관리자와 업무용 Web App

화려한 gallery보다 Carbon, Material Design과 실제 SaaS product screen을 우선한다. Table, filter, bulk action, form validation, empty state, 권한 부족과 긴 data를 어떻게 처리하는지 찾는다. 검색어도 `beautiful dashboard`보다 `audit log filter`, `bulk edit table`, `permission denied state`처럼 사용자의 작업으로 작성한다.

### 고객용 Mobile App

Mobbin·Refero에서 같은 업종보다 같은 행동을 먼저 찾는다. 예를 들어 서비스가 달라도 onboarding, 본인 인증, 결제 수단 추가와 알림 허용 flow는 비교할 수 있다. iOS는 Apple HIG, Android는 Material Design을 함께 확인해 platform 관례를 지킨다.

### 소개·홍보용 Landing Page

Lapa Ninja와 Landbook으로 전체 page의 hierarchy, section 순서와 typography를 비교한다. 멋있는 hero만 복사하지 말고 value proposition, product proof, CTA와 FAQ가 어떤 순서로 사용자의 의문을 해결하는지 본다.

## Claude에 이미지만 던지지 않는 방법

Claude는 레퍼런스가 많다고 자동으로 좋은 결정을 내리지 않는다. 각 자료에 **왜 선택했는지**를 붙이면 결과가 훨씬 안정적이다.

### Reference Brief

```markdown
## 사용자와 목적
- 사용자: 매일 주문을 처리하는 운영 담당자
- 핵심 작업: 처리되지 않은 주문을 빠르게 찾아 일괄 승인
- 성공 조건: 현재 상태와 다음 행동을 별도 설명 없이 알 수 있음

## 참고할 점
- Reference A: table의 정보 우선순위와 filter 배치
- Reference B: bulk selection과 action feedback
- Reference C: empty·loading·error state 처리

## 참고하지 않을 점
- 브랜드 색상과 logo
- 실제 업무에 없는 chart
- animation과 장식용 gradient

## 제약
- 기존 component library 사용
- keyboard만으로 핵심 작업 가능
- desktop 우선, 좁은 화면에서도 조회 가능
```

“이 화면처럼 만들어 줘”보다 **문제, 참고할 pattern, 제외할 요소와 검증 조건**을 함께 전달한다. 특정 제품을 그대로 복제하지 않고 여러 레퍼런스에서 각각 한 가지 이유만 가져오는 편이 좋다.

## 디자이너 없는 팀의 최소 Workflow

1. **문제를 먼저 적는다.** 화면 이름보다 사용자, 목표, 빈도와 실패 비용을 쓴다.
2. **같은 행동을 검색한다.** 경쟁사 이름뿐 아니라 onboarding, filter, approval, checkout처럼 flow와 pattern으로 찾는다.
3. **하나의 디자인 시스템을 기준으로 정한다.** 색·간격·component 규칙을 매 화면마다 새로 생성하지 않는다.
4. **레퍼런스를 소수로 좁힌다.** 각 레퍼런스에서 가져올 점과 버릴 점을 기록한다.
5. **Claude에 여러 안을 요청한다.** 첫 결과를 확정하지 말고 정보 구조나 interaction이 다른 안을 비교한다.
6. **상태를 빠짐없이 만든다.** 기본, loading, empty, error, success, disabled, 권한 부족과 responsive 상태를 확인한다.
7. **실제 사용 흐름으로 검증한다.** screenshot의 완성도보다 사용자가 목표를 달성하는 과정과 오류 회복을 본다.
8. **결정을 기록한다.** 선택한 안, 이유, trade-off와 이후 확인할 위험을 짧게 남긴다.

## Claude가 대체하기 쉬운 일과 어려운 일

| 비교적 잘 보조하는 일 | 사람이 계속 책임져야 하는 일 |
|---|---|
| 레퍼런스 분류와 공통 pattern 추출 | 실제 사용자 문제가 맞는지 판단 |
| 여러 layout과 visual direction 생성 | 정보의 business 우선순위 결정 |
| 디자인 시스템에 맞춘 component 조합 | 브랜드 고유성 및 법적·윤리적 판단 |
| 화면별 일관성·누락 상태 점검 | 사용자 조사와 이해관계자 조율 |
| HTML/CSS 구현과 반복 수정 | 접근성과 사용성의 최종 검증 |

따라서 “Claude가 디자이너를 완전히 대체한다”보다 **디자인 실행 능력은 Claude로 보강하고, 디자인 책임자는 팀 안에 명시적으로 남긴다**고 보는 편이 안전하다. 디자이너가 없다면 product owner나 개발자 한 명이 사용자 문제, 레퍼런스 선택과 최종 승인에 대한 책임을 맡아야 한다.

## 개발자가 먼저 익힐 판단 기준

좋아 보이는 화면을 많이 보는 것만으로는 충분하지 않다. 다음 질문을 반복해서 사용한다.

- 사용자가 첫 화면에서 현재 상태와 다음 행동을 알 수 있는가?
- 가장 중요한 정보와 action이 시각적으로 먼저 보이는가?
- 익숙한 pattern을 바꿔야 할 분명한 이유가 있는가?
- 선택지와 입력 항목을 줄일 수 있는가?
- 오류 원인과 복구 방법을 알려 주는가?
- keyboard, focus, contrast와 screen reader 동작을 확인했는가?
- 실제 data가 길거나 없거나 실패해도 layout이 유지되는가?
- 레퍼런스의 외형이 아니라 해결 방식을 가져왔는가?

기존 Wiki의 [개발자를 위한 한국어 UX 도서 근거 가이드](developer-ux-book-evidence-guide.md)는 화면 결정의 이유와 사용성 원칙을 배우는 입문 자료를 목적별로 정리한다. 레퍼런스 사이트로 사례를 찾고, 책과 디자인 시스템으로 그 선택의 이유를 설명하는 방식이 개발자에게 잘 맞는다.

## See Also

- [개발자를 위한 한국어 UX 도서 추천](developer-ux-book-recommendations-2026-08-21.md)
- [gstack으로 AI 개발 Workflow 이해하기](../ai-agents/gstack-ai-engineering-workflow.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](../ai-agents/superpowers-agent-management-and-spec-driven-development.md)
