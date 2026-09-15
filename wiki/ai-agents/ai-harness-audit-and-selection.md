# AI Harness 실측과 선택 가이드

> Sources: Personal in-house harness audit, 2026-08-10; Tejas Kumar·IBM, YouTube (Unknown)
> Raw: [사내 하네스 3종 실측](../../raw/ai-agents/2026-08-10-inhouse-harness-audit.md); [Harnesses in AI transcript](../../raw/software-career/harnesses-in-ai-a-deep-dive-tejas-kumar-ibm.md); [Original YouTube source provenance](../../raw/software-career/harnesses-in-ai-tejas-kumar-source-provenance.md)
> Updated: 2026-08-16

## Overview

AI harness는 기능 수나 agent 수가 많다고 항상 더 적합한 것이 아니다. 개인 운영에서는 작업이 greenfield인지 brownfield인지, 한 사람이 단일 task를 처리하는지 여러 작업자가 병렬 개발하는지, 실제 차단 hook이 필요한지를 기준으로 도구의 무게와 입력 계약을 맞춰야 한다.

## 성격과 유효기간

이 문서는 특정 시점의 로컬 설치 상태와 plugin source를 정적으로 조사한 **개인 도구 평가**다. 조사 대상은 빠르게 갱신되므로 구체적인 version, agent 수, 문서량과 hook 구성은 만료될 수 있다. 아래 선택 원칙은 재사용하되 실제 도입 전에는 재조사 절차로 현재 상태를 확인한다.

## 무게는 용도와 함께 판단한다

조사에서는 `itnew-forge`와 `itnew-dev`가 단일 task에 투입하는 agent 수와 검증 단계에서 큰 차이를 보였다. 이를 단순히 무겁고 가벼운 도구의 우열로 해석하지 않는다.

- `itnew-forge`는 여러 작업자가 여러 session에서 병렬 개발하는 무거운 orchestration에 가깝다.
- `itnew-dev`는 기본 설정에서 단일 task를 가볍게 처리하고 필요할 때 addon을 붙이는 구조에 가깝다.
- 한 사람의 작은 수정에 대규모 병렬 harness를 적용하면 coordination overhead가 실제 구현보다 커질 수 있다.
- 복잡한 병렬 개발에서는 다중 검증과 역할 분리가 합리적인 비용일 수 있다.

따라서 agent 개수나 checklist 길이가 아니라 현재 작업에서 실제로 필요한 coordination, isolation, verification 수준을 먼저 정한다.

## Harness 구성요소를 실행 기준으로 살펴본다

특정 도구가 harness라고 부르는지보다 다음 구성이 runtime에서 실제로 어떻게 연결되는지 확인한다.

- Tool registry: model이 어떤 행동을 실행할 수 있는가?
- Context management: task·history·compaction을 누가 관리하는가?
- Guardrail: step·message·cost·위험 행동의 상한은 무엇인가?
- Trace: 실제 tool 호출과 상태 변화를 재현할 수 있는가?
- Verify: agent의 완료 문장과 독립적으로 성공을 판정하는가?
- Retry·escalation: 반복 상한과 사람에게 넘길 조건이 있는가?
- Secret boundary: credential이 model context에 노출되지 않는가?

Prompt·Skill이 작업 순서를 설명하는 것과 harness code가 실제 성공 조건을 재판정하는 것을 별도 항목으로 audit한다.

## 문서에 적힌 gate와 실제 hook을 구분한다

조사 시점의 대상들은 TDD와 hard gate를 표방했지만 주요 강제 절차가 Markdown과 main agent의 자가 검사에 의존했다. 좋은 검사 규칙이 있어도 실행 여부를 모델이 선택한다면 결정적 강제가 아니다.

개인 audit에서는 다음을 직접 확인한다.

- `hooks.json`에 실제로 어떤 event가 등록되어 있는가
- 차단을 만드는 `exit 2`나 deny decision이 존재하는가
- 설정 key가 runtime hook인지 prompt가 해석하는 절차인지
- gate가 기본으로 활성화되는가
- gate 미발화가 log나 changelog에서 관측되는가

## Greenfield 편향은 입력 계약에서 생긴다

조사한 harness들은 PRD, spec, task breakdown 같은 기획 산출물을 필수 입력으로 삼았다. 이런 도구는 정본을 사람이 미리 작성한 문서로 설정했기 때문에 문서가 없는 인수인계 project에서 진입 단계가 막힌다.

이 제약을 harness 전체의 본질로 보지 않는다. 실행 agent가 기존 code path를 처리할 수 있더라도 task를 생성하는 입구가 planning document만 받도록 설계되면 전체 workflow가 greenfield에 편향될 수 있다. 도구 적용 범위는 결국 무엇을 source of truth로 선택하는지에 크게 좌우된다.

## 폐기 문서와 현행 기능을 교차 확인한다

Repository 검색에서 `brownfield`, `reverse` 같은 이름이 발견되어도 현재 기능이 존재한다고 단정하지 않는다. 과거 skill의 guide가 남아 있을 수 있으므로 실제 `skills/` 구조, 현재 entry point와 실행 code를 함께 확인해야 한다.

개인 규칙은 다음과 같다.

1. 문서에서 기능 이름을 찾는다.
2. 현재 설치된 plugin인지 확인한다.
3. 현재 `skills/` 또는 runtime code에 구현이 남아 있는지 확인한다.
4. 필수 입력과 중단 조건을 확인한다.
5. 간단한 실제 session으로 동작을 계측한다.

## 작업별 선택 기준

| 작업 상황 | 우선 선택 |
|---|---|
| 대규모 greenfield를 여러 agent가 병렬 구현 | 역할 분리와 검증이 강한 orchestration harness 검토 |
| 한 사람이 명확한 단일 task 수정 | 가벼운 workflow 또는 단일 agent 우선 |
| 읽기 전용 조사·분석 | 필요할 때만 subagent를 사용하고 무거운 harness는 피함 |
| 기획 문서 없는 brownfield 인수인계 | greenfield harness를 억지로 통과시키지 않고 code를 정본으로 삼는 workflow 사용 |
| 위험 명령·TDD 조건을 반드시 차단 | harness의 문구보다 실제 hook과 enforcement layer 확인 |

## 재조사 시점

다음 변화가 보이면 이 문서의 실측 부분을 다시 확인한다.

- plugin의 major workflow나 version이 크게 변경됨
- `PreToolUse`, `PostToolUse`, `Stop` hook이 새로 추가됨
- brownfield 또는 code reverse entry point가 추가됨
- planning document 필수 조건이 완화됨
- agent 수, dispatch 구조나 기본 preset이 바뀜
- marketplace source와 실제 installed plugin 상태가 달라짐

## 한계

Agent spawn 수는 source의 분기와 호출 경로를 센 추정이며 실제 session telemetry가 아니다. 정적 분석에서 발견한 gate도 runtime에서 어떻게 발화하는지는 별도 계측이 필요하다. 무거운 harness 자체가 결함이라는 결론이 아니라, **도구의 전제와 현재 작업의 규모를 맞추라**는 개인 지침이다.

## See Also

- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md)
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
