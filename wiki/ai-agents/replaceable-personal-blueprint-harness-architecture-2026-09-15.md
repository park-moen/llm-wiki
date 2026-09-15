# 교체 가능한 Personal Blueprint Harness 설계

> Sources: [개인 AI Engineering Harness 주말 구축 시뮬레이션](personal-ai-engineering-harness-weekend-simulation-2026-09-15.md); [Superpowers를 지속 사용하는 Harness 운영 전략](superpowers-continuous-use-harness-strategy.md); [Matt Pocock Skills의 Repository 설정 방식](matt-pocock-skills-repository-setup.md); [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md); [효과적인 Software Design Document 작성법](../software-career/effective-software-design-document.md)
> Archived: 2026-09-15

## Overview

회사 Blueprint를 개인용으로 단순 복제하기보다, 사람이 읽기 쉬운 기획 체계와 교체 가능한 Skill adapter, 단계적으로 강화하는 hook을 결합한 **Personal Blueprint Lab**으로 재설계한다. 개인 Blueprint가 직접 소유할 것은 workflow 단계, 산출물 계약, 추적성, 상태와 검증 규칙이다. Superpowers, gstack, Matt Pocock Skills와 이후 등장할 Skill은 특정 capability를 제공하는 교체 가능한 provider로 연결한다. 이 구조의 목적은 만능 기획 Skill 하나를 만드는 것이 아니라 여러 Skill을 안전하게 경험·비교하고, hook이 AI를 실제로 어디까지 통제할 수 있는지 관찰하는 것이다.

## 설계 목표

1. `SCR`, `TC`, `BR` 같은 ID를 유지하면서도 사람이 별도 문서를 찾지 않고 의미를 이해할 수 있게 한다.
2. 회사 Blueprint의 좋은 구조는 참고하되 다른 Skill의 접근법을 실험할 수 있게 한다.
3. 특정 Skill 이름과 구현에 core가 직접 의존하지 않게 한다.
4. 새로운 Skill을 기존 provider와 비교한 뒤 안전하게 교체하거나 함께 사용할 수 있게 한다.
5. 산문 지침과 hook의 실제 차단 능력을 분리해 실험한다.

## 시리얼 번호를 사람이 읽을 수 있게 만든다

`REQ`, `FEAT`, `SCR`, `BR`, `TC` 같은 안정적인 ID는 문서 간 추적성과 자동 검증에 유용하다. 문제는 ID 자체가 아니라 의미를 생략하고 ID만 표시하는 방식이다.

기계 중심 표기는 다음과 같다.

```text
REQ-014 → FEAT-009 → SCR-003 → BR-017 → TC-UI-021
```

개인 Blueprint에서는 항상 이름을 붙인다.

```text
회원 상태 필터 요구사항 (REQ-014)
→ 회원 상태 필터 기능 (FEAT-009)
→ 관리자 회원 목록 화면 (SCR-003)
→ 비활성 회원 제외 규칙 (BR-017)
→ 상태 필터 전환 테스트 (TC-UI-021)
```

문서 링크도 ID만 노출하지 않는다.

```markdown
[관리자 회원 목록 화면 (SCR-003)](06-screen-spec.md#scr-003-관리자-회원-목록)
```

### 표시 원칙

- ID를 단독으로 표시하지 않는다.
- 처음 등장할 때는 `이름 (ID)` 형식을 사용한다.
- 같은 문서에서 반복할 때는 의미가 명확하면 이름만 사용한다.
- 표에서는 `ID`와 `이름`을 별도 열로 둔다.
- Traceability Matrix는 기계 검증용 정본으로 유지한다.
- 사람이 읽는 문서에는 관련 항목의 한 줄 설명을 함께 표시한다.

| ID | 이름 | 한 줄 설명 |
|---|---|---|
| `SCR-003` | 관리자 회원 목록 | 회원을 조회하고 상태별로 필터링하는 화면 |
| `BR-017` | 비활성 회원 제외 | 기본 조회에서는 비활성 회원을 표시하지 않음 |
| `TC-UI-021` | 상태 필터 전환 | 전체·활성·비활성 필터와 URL 상태를 검증 |

ID는 컴퓨터를 위해 유지하고 이름과 설명은 사람을 위해 추가한다.

## Core는 Skill 이름이 아니라 Capability를 요청한다

Personal Blueprint core에서 `/gstack:plan-eng-review` 같은 실제 Skill 이름을 직접 호출하면 upstream 구조가 바뀔 때 core도 함께 깨진다. Core는 필요한 능력만 선언하고 adapter가 현재 설치된 Skill과 연결하게 한다.

```text
problem-discovery
requirements-critique
domain-review
implementation-planning
design-critique
browser-qa
code-review
```

하나의 capability에는 여러 provider가 대응할 수 있다.

```text
implementation-planning
├─ superpowers/writing-plans
├─ mattpocock/to-tickets
└─ future-skill/implementation-plan

browser-qa
├─ gstack/qa-only
├─ playwright-adapter
└─ future-browser-skill
```

전체 구조는 다음과 같다.

```text
Personal Blueprint Core
│
├─ Artifact Contract
├─ Workflow State
├─ Validation Rules
└─ Capability Request
          │
          ▼
    Provider Registry
     ├─ Superpowers adapter
     ├─ gstack adapter
     ├─ Matt Pocock adapter
     └─ 새로운 Skill adapter
```

### Core가 소유할 것

- Discover → Define → Design 같은 workflow 단계
- 산출물 형식
- 사람이 읽기 좋은 ID 표현
- 문서 간 추적성
- 상태와 변경 이력
- 사람의 승인 지점
- 검증 기준

### 외부 Skill에 맡길 것

- 문제를 탐색하는 질문 방식
- Plan을 비판하는 관점
- Domain model을 검토하는 방법
- Browser에서 실제 흐름을 확인하는 방법
- 산출물과 code를 review하는 기준

## Adapter는 공통 입출력 계약을 사용한다

Skill마다 입력과 출력 형식이 다르면 교체가 어렵다. Adapter가 provider별 결과를 공통 형식으로 변환한다.

```yaml
capability: implementation-planning
provider: superpowers/writing-plans
status: completed

inputs:
  - docs/planning/04-spec.md
  - docs/planning/06-screen-spec.md

outputs:
  artifact: experiments/run-001/implementation-plan.md

findings:
  - title: 비활성 상태 처리 누락
    severity: medium
    related:
      - "회원 상태 필터 기능 (FEAT-009)"

open_questions:
  - 기본 필터가 전체인지 활성 상태인지 결정 필요

mutations:
  canonical_documents_changed: false

evidence:
  - experiments/run-001/transcript.md
```

새로운 Skill이 등장하면 core를 수정하지 않고 adapter를 추가한다. Provider가 없거나 실행에 실패했을 때도 core가 중단되는 대신 fallback 또는 사람에게 넘기는 상태를 명확히 반환해야 한다.

## 새로운 Skill은 Shadow Mode로 먼저 경험한다

처음 설치한 Skill이 기획 정본을 바로 고치게 하지 않는다. Active provider와 shadow provider를 구분한다.

```text
Active provider
└─ 승인된 범위에서 실제 산출물 작성 가능

Shadow provider
└─ 같은 입력을 검토하지만 실험 결과만 별도로 작성
```

예를 들어 Personal Blueprint가 PRD를 만든 뒤 여러 Skill을 다음처럼 비교한다.

```text
Personal Blueprint Core
→ 정본 PRD 작성

Superpowers brainstorming
→ 빠진 질문만 experiments/에 기록

gstack plan review
→ 범위·위험 의견만 experiments/에 기록

Matt Pocock domain-modeling
→ 용어·경계 의견만 experiments/에 기록
```

사람은 각 결과를 비교해 유용한 부분만 정본에 반영한다. 유용성이 여러 Run에서 반복해서 확인된 provider만 active로 승격한다.

| 평가 항목 | 확인할 내용 |
|---|---|
| 새로운 발견 | 기존 workflow가 놓친 문제를 찾았는가 |
| 중복 | 이미 있는 질문이나 문서를 반복했는가 |
| 실행 가능성 | 의견을 요구사항이나 검증 조건으로 바꿀 수 있는가 |
| 비용 | 추가 context와 시간에 비해 가치가 있었는가 |
| 침범 | 정본이나 사용자의 결정을 임의로 바꾸려 했는가 |

## Profile로 Skill 조합을 바꾼다

Skill 조합을 하나로 고정하지 않고 실험 목적에 따라 profile을 둔다.

```yaml
profiles:
  stable:
    problem-discovery: personal-blueprint
    implementation-planning: superpowers
    browser-qa: gstack
    code-review: mattpocock

  exploration:
    problem-discovery:
      active: personal-blueprint
      shadow:
        - superpowers
        - gstack
    domain-review:
      shadow:
        - mattpocock

  lean:
    problem-discovery: personal-blueprint
    implementation-planning: built-in
    browser-qa: playwright
```

이 구조에서는 gstack을 제거해도 `browser-qa` capability 자체는 유지된다. 새로운 Skill이 더 적합하면 profile의 provider mapping만 교체한다.

## Hook은 단계적으로 통제력을 높인다

Hook으로 AI를 어디까지 통제할 수 있는지 알아보려면 처음부터 모든 위반을 차단하지 않는다. 같은 규칙을 관찰, 경고와 차단 단계로 올리며 실제 효과와 부작용을 기록한다.

```text
Level 0: Prompt
→ 지침만 제공

Level 1: Observe
→ 위반을 기록하지만 막지 않음

Level 2: Warn
→ Agent에게 수정 기회를 제공

Level 3: Block
→ 실제 tool 실행이나 완료를 차단
```

기획 문서 직접 수정 규칙은 다음처럼 발전시킬 수 있다.

```text
Observe
→ 어떤 Skill이 docs/planning을 직접 고치는지 기록

Warn
→ 변경 관리 절차를 안내

Block
→ 승인되지 않은 수정은 tool 단계에서 거부
```

| Hook | 책임 |
|---|---|
| `SessionStart` | 정본 위치, profile과 provider 상태 주입 |
| `PreToolUse` | 위험 Git 명령, 배포와 정본 무단 변경 차단 |
| `PostToolUse` | 문서 drift, 예상 밖 file과 test 삭제 감지 |
| `Stop` | 추적성 검사, 전체 verify와 증거 누락 확인 |

좋은 설계인지, 요구사항이 올바른지, test가 의미 있는지와 비용·일정의 trade-off는 단순한 exit code로 판정할 수 없다. 이런 판단은 사람과 reviewer에게 남기고 hook은 file 변경, command, exit code와 ID 누락처럼 관찰 가능한 조건을 맡는다.

## 권장 Repository 구조

```text
personal-blueprint/
├── core/
│   ├── workflow.md
│   ├── artifact-contracts/
│   ├── validation-rules/
│   └── capability-contracts/
├── adapters/
│   ├── superpowers/
│   ├── gstack/
│   ├── mattpocock/
│   └── built-in/
├── profiles/
│   ├── stable.yaml
│   ├── exploration.yaml
│   └── lean.yaml
├── hooks/
│   ├── session-start
│   ├── pre-tool-use
│   ├── post-tool-use
│   └── stop
├── scripts/
│   ├── validate-traceability
│   ├── check-planning-drift
│   └── collect-run-evidence
└── experiments/
    └── run-001/
```

핵심 정책과 validator는 특정 AI host 밖에 둔다. Claude Code, Codex나 이후 사용할 host에는 core를 호출하는 얇은 adapter만 둔다.

## 피해야 할 결합 방식

다음 구조는 Personal Blueprint가 특정 upstream 명령에 직접 의존하게 만든다.

```text
Personal Blueprint
└─ 내부에서 gstack·Superpowers 명령을 직접 호출
   └─ Upstream 변경 시 전체 workflow가 깨짐
```

대신 core는 capability를 요청하고 adapter가 provider별 차이를 흡수해야 한다.

```text
Personal Blueprint Core
└─ “기술 계획 검토가 필요하다”는 capability 요청
          ↓
Adapter가 현재 provider 선택
          ↓
결과를 공통 계약으로 변환
          ↓
Hook과 validator가 독립적으로 검증
```

회사 Blueprint를 그대로 fork해 수정하면 upstream 변경과 개인 변경의 차이를 계속 관리해야 한다. 회사 Blueprint는 `base class`라기보다 좋은 참고 구현으로 해체해 받아들이고, 개인 core와 계약은 독립적으로 설계하는 편이 적합하다.

## 최종 판단

Personal Blueprint는 새로운 만능 기획 Skill 하나보다 다음 세 가지를 함께 갖춘 환경이어야 한다.

1. 사람이 읽기 쉬운 개인 기획 체계
2. 여러 Skill을 안전하게 비교하고 교체하는 실험실
3. Agent 지침과 실제 시스템 통제의 차이를 측정하는 harness

장기적으로 남는 개인 자산은 특정 Skill의 prompt가 아니다. 기획 정본의 구조, 사람이 이해할 수 있는 추적성, capability 계약, provider 평가 기록, 권한 경계와 독립적인 완료 판정이 핵심 자산이다.

## See Also

- [gstack으로 AI 개발 Workflow 이해하기](gstack-ai-engineering-workflow.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
- [AI Agent 지침과 개인 지식 자산 운영 원칙](ai-agent-instructions-judgment-and-personal-knowledge-assets-2026-09-14.md)
