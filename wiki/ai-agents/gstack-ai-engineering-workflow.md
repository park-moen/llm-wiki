# gstack으로 AI 개발 Workflow 이해하기

> Sources: Garry Tan·gstack, GitHub (Unknown)
> Raw: [gstack README](../../raw/ai-agents/gstack-readme.md); [gstack Builder Ethos](../../raw/ai-agents/gstack-builder-ethos.md); [gstack Architecture](../../raw/ai-agents/gstack-architecture.md)
> Updated: 2026-09-08

## Overview

gstack은 AI coding agent에게 역할별 지침과 실행 도구를 제공해 아이디어 검토부터 배포 후 확인까지 이어 주는 open source workflow다. 핵심은 agent에게 한 번에 “알아서 개발해”라고 맡기는 대신, 제품 판단·설계·구현 계획·code review·실제 화면 검사·배포를 서로 다른 Skill로 나누는 것이다. 주니어 개발자에게는 빈 prompt에서 시작하지 않아도 된다는 장점이 있지만, Skill의 결론을 정답으로 받아들이지 않고 변경 범위와 검증 결과를 직접 이해해야 한다.

## 한 문장으로 이해하기

일반적인 AI coding 도구가 한 명의 빠른 개발자라면, gstack은 그 도구에 **제품 책임자, engineering manager, designer, reviewer, QA와 release engineer의 작업 절차를 붙인 운영 체계**에 가깝다.

gstack의 README는 이를 “virtual engineering team”과 “open source software factory”라고 설명한다. 여기서 실제 사람이 여러 명 생기는 것은 아니다. 하나의 AI agent가 `/office-hours`, `/plan-eng-review`, `/review`, `/qa`, `/ship` 같은 Skill을 차례로 읽으며 역할과 점검 기준을 바꾸는 방식이다.

## 전체 흐름

gstack이 제시하는 sprint 순서는 다음과 같다.

```text
Think → Plan → Build → Review → Test → Ship → Reflect
```

각 단계의 산출물이 다음 단계의 입력이 된다.

| 단계 | 대표 Skill | 주니어 개발자가 확인할 것 |
|---|---|---|
| 문제 탐색 | `/office-hours` | 사용자가 겪는 문제가 무엇인지, 제안한 기능이 정말 필요한지 |
| 범위 결정 | `/plan-ceo-review` | 지금 만들 범위와 나중으로 미룰 범위가 분리됐는지 |
| 기술 계획 | `/plan-eng-review` | data flow, 실패 경로, edge case와 test 방법이 적혀 있는지 |
| 구현 | 일반 coding agent 작업 | plan과 다른 file까지 바꾸지 않았는지 |
| code review | `/review` | CI가 놓칠 수 있는 production bug와 빠진 요구사항이 있는지 |
| 사용자 흐름 검사 | `/qa` 또는 `/qa-only` | 실제 browser에서 핵심 흐름이 동작하는지 |
| 출하 | `/ship` | test와 변경 범위를 확인한 뒤 PR을 만들 준비가 됐는지 |
| 회고 | `/retro` | 반복되는 실패와 다음에 개선할 작업 방식이 무엇인지 |

모든 작업에 전체 흐름을 적용할 필요는 없다. README도 단순한 오타 수정에는 gstack이 필요하지 않을 수 있다고 예시를 든다. 작은 변경에 긴 기획 절차를 붙이면 검토 비용이 구현 비용보다 커질 수 있다.

## Skill과 실제 도구가 함께 있는 구조

gstack은 Markdown 지침만 모아 둔 저장소가 아니다.

- **역할별 Skill**은 agent가 어떤 질문을 하고 어떤 순서로 판단할지 설명한다.
- **설치·생성 과정**은 같은 원본 template을 Claude Code와 Codex를 포함한 여러 host에 맞는 Skill 문서로 만든다.
- **browser 계층**은 기획 조사, 화면 확인, screenshot과 QA에 실제 browser 상태를 제공한다.
- **실행 code와 test**는 browser daemon, 인증, 문서 생성, 검증 gate와 설치 동작을 담당한다.

macOS에서 Aside가 열려 있으면 사용자의 실제 로그인 session과 tab을 우선 사용한다. Aside가 없거나 실행 중이 아니면 gstack이 제공하는 persistent Chromium으로 자동 전환한다. fallback browser는 command마다 새 browser를 띄우지 않고 daemon을 유지해 cookie, localStorage와 tab 상태를 보존한다.

이 구조에서 중요한 점은 **Skill의 산문 지침**과 **실제로 실행을 막는 code**를 구분하는 것이다. “반드시 review한다”는 문장이 있어도 agent가 건너뛸 수 있다면 권고에 가깝다. 반면 command의 exit code, 인증 실패, 허용된 endpoint와 같은 검사는 runtime에서 기계적으로 판정된다.

## 주니어 개발자에게 도움이 되는 이유

### 빈 Prompt 대신 질문 순서를 제공한다

처음부터 좋은 요구사항과 설계 질문을 모두 떠올리기 어렵다. gstack의 역할별 Skill은 문제, 범위, data flow, 실패 처리와 test를 차례로 검토하게 해 빠뜨린 질문을 줄인다.

### Code 생성 뒤의 검증 단계를 눈에 보이게 만든다

구현이 끝났다는 agent의 말만 듣지 않고 `/review`와 `/qa`를 별도 단계로 둔다. 특히 browser 기반 QA는 실제 사용 흐름과 화면 상태를 확인하는 데 도움이 된다.

### 역할을 분리해 관점을 바꾼다

구현한 agent는 자신의 결정을 방어하기 쉽다. 같은 model을 쓰더라도 제품 범위, engineering, design과 QA의 기준을 분리하면 한 관점으로만 작업을 끝내는 위험을 줄일 수 있다. 다만 역할 이름이 다르다고 독립적인 사람 여러 명의 review와 같아지는 것은 아니다.

## 처음 사용할 때의 권장 순서

처음부터 자동 배포나 여러 병렬 sprint를 실행하기보다 작은 기능 하나로 흐름을 익히는 편이 안전하다.

1. 실제 사용자 문제를 자신의 말로 한 문단 작성한다.
2. `/office-hours`로 문제와 해결책을 구분한다.
3. `/plan-eng-review` 결과에서 변경 file, data flow, 실패 경로와 test를 직접 설명해 본다.
4. 구현 후 `git diff`를 읽고 예상 밖 변경을 찾는다.
5. `/review`의 의견을 code와 test로 확인한다.
6. 배포하지 않은 local 또는 staging 환경에서 `/qa-only`로 먼저 검사한다.
7. test·lint·typecheck·build와 핵심 사용자 흐름을 직접 확인한 뒤에만 출하 단계를 검토한다.

`/qa-only`는 보고만 하고 code를 바꾸지 않으므로 처음 구조를 익힐 때 유용하다. `/qa`, `/review`, `/ship`, `/land-and-deploy`는 각각 code 수정, commit, push, PR 생성, merge 또는 배포까지 이어질 수 있으므로 Skill 이름만 보지 말고 실행 전 권한과 변경 범위를 확인한다.

## 안전하게 사용할 때 확인할 것

### 실제 계정에 미치는 행동

Aside 경로는 사용자가 이미 로그인한 browser를 사용한다. Architecture 문서는 읽기·이동·제출 전 form 입력과 실제 계정을 변경하는 행동을 구분하고, 외부 대상의 변경 작업에는 먼저 동의를 받는 원칙을 둔다. 그래도 agent가 실제 session에 접근한다는 사실은 변하지 않으므로 결제, 삭제, 전송과 production 관리 화면은 별도 승인이 필요한 영역으로 정한다.

### 자동 Update와 공급망

공유 저장소의 team mode는 주기적으로 update를 확인해 version drift를 줄인다. 반대로 upstream Skill이 바뀌면 agent의 행동 규칙도 바뀔 수 있다. 설치 source, 고정 version, update 방식과 변경 내역을 code 의존성처럼 검토한다.

### 검증 기준을 약화시키는 변경

Test command가 성공해도 agent가 test를 삭제하거나 검사 범위를 줄였다면 신뢰할 수 없다. 성공 여부와 함께 test diff, coverage 변화와 실행한 command를 확인하고, public API·security·data migration처럼 위험이 큰 변경은 사람이 더 깊게 review한다.

## gstack의 철학을 그대로 받아들이지 않아도 된다

Builder Ethos는 큰 범위를 두려워하지 말고, 새로 만들기 전에 재사용할 지식을 찾으며, 최종 결정권을 사용자에게 남기라고 주장한다. 이 원칙들은 서로 균형을 이뤄야 한다.

- 넓은 목표는 기존 code의 blast radius와 review 가능 범위로 잘게 나눈다.
- 빠른 생성을 생산성의 전부로 보지 않고 실제 사용자 결과, 결함과 유지보수 비용을 함께 본다.
- 여러 model이 같은 결론을 내도 추천일 뿐이며, business context를 가진 사용자가 결정한다.
- 기존 solution을 재사용할 때도 license, 보안과 project 적합성을 확인한다.

## 한계와 평가 시 주의점

- README의 생산성 배수와 출하량은 Garry Tan이 자신의 저장소를 대상으로 계산한 자체 측정이다. 독립적인 비교 실험이나 일반 개발팀의 성과 보장은 아니다.
- 역할별 Skill은 판단 누락을 줄일 수 있지만, 같은 잘못된 요구사항이나 부족한 context를 공유하면 여러 단계가 같은 방향으로 틀릴 수 있다.
- 실제 browser와 로그인 session을 이용하는 기능은 편리한 만큼 개인정보, 외부 전송과 계정 변경의 위험 범위도 커진다.
- 많은 Skill과 산출물은 복잡한 제품 작업에는 도움이 되지만 작은 유지보수에는 불필요한 절차가 될 수 있다.
- 저장소가 빠르게 변하므로 정확한 Skill 목록, 지원 host, 설치 방식과 보안 구조는 사용 전에 현재 README와 code를 다시 확인한다.

주니어 개발자에게 가장 중요한 학습 목표는 gstack 명령을 많이 외우는 것이 아니다. **문제를 정의하고, 변경 범위를 설명하고, 검증 증거를 읽고, 마지막 결정을 스스로 내리는 능력**을 기르는 데 workflow를 사용해야 한다.

## See Also

- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
