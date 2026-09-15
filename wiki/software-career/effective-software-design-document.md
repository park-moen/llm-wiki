# 효과적인 Software Design Document 작성법

> Sources: Michael Lynch, 2026-06-24
> Raw: [How to Write an Effective Software Design Document](../../raw/software-career/2026-06-24-effective-software-design-document.md)
> Updated: 2026-09-15

## Overview

Software Design Document는 구현 내용을 미리 길게 적는 문서가 아니다. 잘못 선택했을 때 되돌리기 어렵거나 비용이 큰 결정을 구현 전에 드러내고, 동료와 관련 팀이 검토할 수 있게 만드는 문서다. 문서의 길이와 section 수는 정답이 아니라 프로젝트의 복잡성, 위험과 협업 범위에 맞춰 정한다.

## 먼저 문서가 필요한 작업인지 판단한다

모든 변경에 설계 문서가 필요한 것은 아니다. 다음 조건이 많을수록 작성할 가치가 커진다.

- 여러 개발자가 구현을 나눠 맡아야 한다.
- 다른 팀이나 외부 system과 조율해야 한다.
- 장기간 개발하거나 production에서 오래 운영할 예정이다.
- 목표와 요구사항이 아직 모호하다.
- 보안, 개인정보, 법률 또는 큰 운영 장애 위험이 있다.

반대로 한 사람이 짧게 끝낼 수 있고, 선택이 틀려도 쉽게 바꿀 수 있는 작업이라면 짧은 메모나 issue만으로 충분할 수 있다. 문서 작성 비용도 engineering cost이므로 형식을 갖추는 것 자체를 목표로 삼지 않는다.

## 무엇을 쓸지는 ‘틀렸을 때의 비용’으로 결정한다

설계 문서에 모든 구현 세부 사항을 적으면 설계 단계에서 구현을 다시 하는 셈이 된다. 포함할 결정은 선택이 틀렸을 때 나중에 바꾸는 비용을 기준으로 고른다.

언어, 저장 방식, system 경계, 공개 interface와 보안 경계처럼 교체하기 어려운 결정은 일찍 검토한다. 버튼 배치나 쉽게 바꿀 수 있는 표시 방식처럼 복구 비용이 작은 결정은 구현과 사용자 feedback에 맡긴다.

이 기준은 문서를 짧게 만들기 위한 요령이 아니라 review 시간을 중요한 결정에 집중시키는 방법이다.

## 첫 page에서 독자의 공통 이해를 만든다

독자가 별도의 구두 설명 없이도 문서를 이해할 수 있어야 한다. 첫 부분에는 다음 내용을 둔다.

| Section | 답해야 할 질문 |
|---|---|
| Title | 대화에서 이 프로젝트를 무엇이라고 부를 것인가? |
| Metadata | 작성자, 작성일, 정본 URL, 상태와 승인자는 누구인가? |
| Objective | 이 프로젝트의 목적을 한 문장으로 설명하면 무엇인가? |
| Background | 왜 지금 이 문제를 해결하며 이전에는 무엇을 시도했는가? |
| Related documents | 요구사항, test plan과 관련 system 문서는 어디에 있는가? |

Objective에는 기술 선택보다 목적을 쓴다. Kubernetes 도입은 구현 수단이고, 배포 장애 감소는 달성하려는 결과다.

## Scope와 사용 모습을 구체화한다

### Goals와 Non-goals

Goals는 구현이 끝난 뒤 사용자, 팀 또는 회사에 어떤 변화가 생겨야 하는지 설명한다. Non-goals는 독자가 당연히 포함됐다고 오해할 만한 범위를 명시적으로 제외한다.

Non-goals는 “하지 않을 일”을 나열하는 방어 문구가 아니다. 이번 설계의 경계를 합의하고 review가 옆 문제로 퍼지는 것을 막는 장치다.

### Scenarios

추상적인 기능 이름만으로 동작을 이해하기 어렵다면 사용자의 실제 흐름을 짧은 scenario로 쓴다.

```text
1. 사용자가 보고서를 만든다.
2. 공유 URL을 생성한다.
3. 동료에게 URL을 전달한다.
4. 동료는 읽기 전용 보고서를 본다.
```

Scenario는 UI 명세를 대신하지 않는다. 설계가 현실에서 어떤 결과를 만드는지 reviewer가 같은 장면을 떠올리게 하는 역할을 한다.

## 구조와 운영을 함께 설계한다

필요한 section만 골라 사용하되 구현 구조와 production 운영을 분리하지 않는다.

| 관점 | 기록할 내용 |
|---|---|
| Diagrams | data flow, component 관계, 외부 dependency와 protocol |
| Glossary | 새 구성원이나 다른 팀이 모를 수 있는 용어 |
| Constraints | 예산, 고객, infrastructure와 dependency 제약 |
| SLO | 가용성, latency와 처리 규모처럼 측정 가능한 목표 |
| Monitoring / alerting | SLO 실패와 주요 장애를 어떻게 발견할지 |
| Timeline | 이해와 검증에 도움이 되는 중간 산출물과 milestone |
| Interfaces | UI, API, CLI와 file format의 contract |
| Dependencies / infrastructure | 언어, library, 실행 환경과 저장소 선택 |
| Security / privacy / legal | 공격면, 신뢰 경계, 민감 정보와 규제 조건 |
| Logging | 남길 event, level, 보존 기간, 접근 권한과 제외할 정보 |

Diagram은 아름답게 그리는 것보다 수정 가능해야 한다. source drawing이나 Mermaid 같은 생성 코드를 함께 연결해야 system이 바뀌었을 때 다시 만들 수 있다.

SLO와 monitoring도 짝으로 본다. 목표값만 적고 production에서 측정할 수 없다면 달성 여부를 알 수 없다. 보안과 개인정보도 구현 후 검사 항목이 아니라 data flow와 신뢰 경계를 결정하는 설계 입력이다.

## 미결정 사항과 대안을 보존한다

설계 중 답을 찾지 못한 문제를 숨기지 않고 `Open issues`에 기록한다.

```text
문제
- 아직 결정하지 못한 것은 무엇인가?

선택지
- 현실적인 대안은 무엇인가?

다음 행동
- 누가 무엇을 확인해야 결정을 내릴 수 있는가?
```

결정이 끝나면 항목을 삭제하지 않는다. 결론을 덧붙여 `Resolved issues`로 옮기면 나중에 같은 논의를 반복하지 않고 당시 판단 근거를 찾을 수 있다.

`Alternatives considered`에는 유력했지만 채택하지 않은 대안과 이유를 짧게 남긴다. 모든 아이디어의 역사를 적을 필요는 없다. Reviewer가 배제한 이유를 물을 가능성이 높은 대안에 집중한다.

## 주니어 개발자를 위한 최소 Template

작은 팀이나 단일 기능이라면 다음 구조로 시작할 수 있다.

```markdown
# 프로젝트 이름

## Metadata
- 작성자:
- 상태: Draft / In Review / Approved
- 관련 문서:

## Objective
누구나 이해할 수 있는 목적 한 문장

## Background
- 현재 문제
- 해결해야 하는 이유
- 이전 시도

## Goals
- 구현 후 달라져야 할 결과

## Non-goals
- 이번 범위에서 제외할 내용과 이유

## Scenarios
- 대표 사용자 또는 system 흐름

## Proposed design
- component와 data flow
- interface와 주요 dependency
- 수정 가능한 diagram source

## Operations and risks
- SLO와 monitoring
- security와 privacy
- logging과 장애 대응

## Open issues
- 문제 / 선택지 / 다음 행동

## Alternatives considered
- 대안 / 채택하지 않은 이유
```

먼저 이 최소 구조를 채운 뒤 프로젝트 위험에 따라 section을 추가한다. 빈 section을 형식적으로 유지하기보다 필요 없는 section은 제거하고 그 이유가 중요하다면 짧게 남긴다.

## Review 전 확인할 질문

- 첫 page만 읽어도 목적과 배경을 이해할 수 있는가?
- Goal이 구현 기술이 아니라 기대 결과로 쓰였는가?
- Scope로 오해할 만한 항목이 Non-goals에 있는가?
- 되돌리기 어려운 결정과 그 trade-off가 드러나는가?
- Diagram 원본을 나중에 수정할 수 있는가?
- 운영, 보안과 개인정보가 구현 이후의 일로 밀려나지 않았는가?
- Open issue마다 결정을 위한 다음 행동이 있는가?
- 강한 대안을 제외한 이유가 남아 있는가?

설계 문서는 완성된 정답을 선언하는 문서가 아니다. 중요한 가정과 결정을 검토 가능한 상태로 만들어, 잘못된 구현에 큰 비용을 쓰기 전에 feedback을 받는 도구다.

## See Also

- [요구사항 분석부터 설계 다이어그램까지: Spring·React 통합 학습 로드맵](requirements-modeling-and-design-diagrams-learning-roadmap-2026-08-24.md)
- [Software Engineering을 Time·Scale·Trade-off로 이해하기](software-engineering-time-scale-tradeoffs.md)
