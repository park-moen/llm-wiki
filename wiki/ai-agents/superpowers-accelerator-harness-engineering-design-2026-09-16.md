# Superpowers 기반 Accelerator Harness Engineering 설계

> Sources: [Superpowers를 지속 사용하는 Harness 운영 전략](superpowers-continuous-use-harness-strategy.md); [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md); [Personal Blueprint 변경 세트와 정합성 종료 Gate 설계](personal-blueprint-change-set-consistency-gate-2026-09-15.md); [교체 가능한 Personal Blueprint Harness 설계](replaceable-personal-blueprint-harness-architecture-2026-09-15.md); [취업 초기 주니어를 위한 AI-Native 개발 프로세스 2: Harness Engineering](../software-career/junior-ai-native-development-harness-engineering.md); [Vibecoder에서 Accelerator로 전환하는 실천 가이드](../software-career/vibecoder-to-accelerator-transition-guide-2026-09-16.md)
> Archived: 2026-09-16

## Overview

Superpowers 기반 개인 harness에 accelerator 방식을 결합할 때는 Superpowers를 fork하거나 내부 절차를 직접 수정하기보다, Superpowers를 workflow engine으로 유지하고 Personal Harness가 `Accelerator profile`과 성공 조건을 소유하는 구조가 적절하다. Superpowers는 작업 순서를 제공하고, accelerator profile은 사람이 이해하고 승인해야 할 판단을 정의하며, repository script·hook·CI는 agent가 건너뛸 수 없는 최소 안전선을 담당한다.

```text
Personal Harness Core
├─ Accelerator profile
│  ├─ 사람이 소유할 판단
│  ├─ 이해 checkpoint
│  └─ 작업 규모별 workflow
│
├─ Superpowers adapter
│  ├─ brainstorming
│  ├─ writing-plans
│  ├─ test-driven-development
│  ├─ requesting-code-review
│  └─ verification-before-completion
│
└─ Repository enforcement
   ├─ scope 검사
   ├─ test·lint·build
   ├─ test 삭제 감지
   ├─ diff·검증 hash
   └─ retry 상한과 human escalation
```

## Workflow 상태 모델

권장 상태 흐름은 다음과 같다.

```text
TASK_DEFINED
→ BASELINED
→ INVESTIGATED
→ DESIGN_APPROVED
→ PLAN_APPROVED
→ IMPLEMENTING
→ REVIEW_READY
→ EXPLAINED
→ VERIFIED
→ CLOSED
```

`EXPLAINED`를 `VERIFIED`와 분리하는 것이 핵심이다. Test가 통과했다는 사실은 사람이 구현을 이해하고 이후 변경과 장애를 책임질 수 있다는 의미가 아니다.

## Task Contract를 사람이 먼저 작성한다

Superpowers를 호출하기 전에 사람이 목표, 범위와 자신의 현재 mental model을 작성한다.

```yaml
goal: 무엇을 해결하는가

behaviors:
  success:
  failure:

scope:
  include:
  exclude:

my_current_model:
  expected_entry_point:
  expected_flow:
  unknowns:

verification:
  target_tests:
  regression_candidates:

risk:
  level: normal
```

빈칸은 허용하되 AI가 임의로 채워 확정하지 못하게 한다. `unknowns`는 `brainstorming`에서 해결할 질문이 된다.

## Baseline을 구현 전에 고정한다

Harness는 구현 전에 다음 상태를 기록한다.

- 현재 branch와 Git status
- 기존 test·lint·build 결과
- 이미 실패 중인 항목
- 변경 전 commit
- 허용된 file 범위

Baseline이 있어야 새 회귀와 기존 문제를 구분할 수 있고, agent가 원래 있던 실패를 자신의 변경 결과로 잘못 해석하는 일을 줄일 수 있다.

## `brainstorming`을 읽기 전용 조사로 제한한다

Accelerator profile에서는 `brainstorming`에 다음 계약을 적용한다.

```text
파일을 수정하지 않는다.
Code에서 확인한 사실과 제안을 구분한다.
각 사실에는 근거 file과 symbol을 붙인다.
사용자가 적은 예상과 다른 부분을 표시한다.
미결정 domain rule을 임의로 확정하지 않는다.
```

출력 형식은 다음처럼 정규화한다.

```yaml
current_flow:
confirmed_facts:
assumptions:
open_questions:
regression_candidates:
recommended_slice:
```

사람이 실제 file을 열어 호출 경로를 확인하고 domain 결정과 scope를 승인해야 `DESIGN_APPROVED`로 이동한다.

## `writing-plans`에 Accelerator 계약을 추가한다

각 구현 단계는 Superpowers의 RED–GREEN task 형식을 따르되 다음 항목을 포함한다.

```yaml
behavior:
why:
red_test:
minimal_implementation:
expected_files:
verification_command:
architecture_decision:
human_question:
```

`human_question`은 구현 방법이 아니라 사람이 결정해야 할 책임·경계·trade-off를 드러낸다. 다음 질문에 답하지 못하면 plan을 승인하지 않는다.

- 어떤 behavior가 달라지는가?
- 어느 interface가 영향을 받는가?
- 이 책임이 왜 해당 layer에 있는가?
- 어떤 test가 성공을 증명하는가?
- 회귀 후보는 무엇인가?

## 순차 TDD를 기본값으로 둔다

초기 accelerator profile에서는 `subagent-driven-development`를 기본값에서 제외한다.

```text
Task 1
├─ RED 확인
├─ GREEN 확인
├─ Diff 확인
└─ 사람 checkpoint

Task 2
├─ RED 확인
├─ GREEN 확인
├─ Diff 확인
└─ 사람 checkpoint
```

한 task가 끝날 때마다 다음 증거를 남긴다.

- 실제 실패 output
- 실제 통과 output
- 변경 file
- 예상 밖 변경
- 새로 발견된 가정
- 다음 task에 영향을 주는 결정

병렬화는 codebase와 domain이 익숙하고 사람이 모든 plan과 diff를 감당할 수 있을 때 별도 profile로 연다.

## Teach-back Gate

구현 후 agent가 설명하는 것만으로는 accelerator의 이해를 확인할 수 없다. 사람이 자신의 말로 설명하는 `Teach-back Gate`를 별도 checkpoint로 둔다.

```yaml
teach_back:
  changed_behavior:
  request_flow:
  state_changes:
  failure_modes:
  important_decisions:
  test_meaning:
  regression_risks:
  incident_starting_point:
  unresolved_questions:
```

사람은 최소한 다음 내용을 설명한다.

- Entry point부터 database 또는 외부 API까지의 흐름
- 상태가 언제 어디서 바뀌는지
- 중요한 조건과 exception의 의미
- Test가 어떤 잘못된 구현을 막는지
- 장애가 발생하면 어디부터 조사할지
- 요구사항이 조금 바뀌면 어디를 수정할지

이 단계는 자동 채점하지 않는다. Agent가 작성한 설명을 그대로 승인하게 되면 이해 gate가 아니라 새로운 AI 문서 생성 단계가 된다. Agent에는 사람의 설명에서 빠진 흐름과 모순을 질문하고 확인할 file 위치를 제시하는 역할만 맡긴다.

## Recovery Drill

Recovery Drill은 익숙하지 않은 영역이나 중요한 기능에 조건부로 적용한다. 다음 중 하나를 사람이 직접 수행한다.

- Test case 추가
- 작은 조건 변경
- 이름 또는 interface 개선
- 실패한 test의 root cause 추적
- AI가 만든 잘못된 가정 수정
- Debugger로 실제 data flow 확인

이 절차가 모든 구현을 혼자 작성할 수 있음을 증명하지는 않는다. Mental model이 수동적인 설명 청취에만 머물지 않았는지 확인하는 역할을 한다.

## Deterministic Gate와 Human Checkpoint를 분리한다

자동화할 항목과 사람이 판단할 항목을 섞지 않는다.

| Deterministic gate | Human checkpoint |
|---|---|
| Test·lint·typecheck·build | 요구사항이 올바른가 |
| 예상 밖 file 변경 | Domain rule이 맞는가 |
| Test 삭제·비활성화 | Test가 의미 있는가 |
| 위험 Git 명령 | Architecture 선택이 타당한가 |
| 허용 범위 밖 변경 | Security·운영 위험이 수용 가능한가 |
| 검증 이후 diff 변경 | 구현을 실제로 이해했는가 |
| Retry 횟수 초과 | 재계획할지 중단할지 |

설명 가능성 자체를 script로 진짜 검증하기는 어렵다. Harness는 사람이 승인했는지와 승인 이후 diff가 바뀌지 않았는지만 기계적으로 확인한다.

## Review와 완료 계약

`requesting-code-review`에는 일반적인 품질 검토 외에 accelerator 기준을 추가한다.

```text
- Task Contract와 diff가 일치하는가
- 근거 없는 domain 결정이 추가됐는가
- 사람이 승인하지 않은 public interface 변경이 있는가
- 불필요한 abstraction이 생겼는가
- Test가 삭제되거나 약화됐는가
- 구현 이유가 기록되지 않은 선택이 있는가
```

Review 의견은 `receiving-code-review`를 거쳐 재현한 뒤에만 반영한다. 마지막 completion gate는 다음 조건을 확인한다.

```text
verify 통과
+ scope 검사 통과
+ test integrity 검사 통과
+ review 완료
+ teach-back 승인 존재
+ 미결정 사항 기록
+ approval 이후 diff 불변
= CLOSED 가능
```

Review나 teach-back 이후 code가 바뀌면 diff hash가 달라지므로 `EXPLAINED`와 `VERIFIED` 상태를 자동 만료한다.

## 권장 Repository 구조

```text
.harness/
├── core/
│   ├── workflow.md
│   ├── state-machine.md
│   └── completion-contract.md
├── profiles/
│   ├── accelerator-lean.yaml
│   ├── accelerator-standard.yaml
│   └── accelerator-high-risk.yaml
├── adapters/
│   └── superpowers.md
├── templates/
│   ├── task-contract.md
│   ├── investigation.md
│   ├── plan-review.md
│   └── teach-back.md
├── scripts/
│   ├── verify
│   ├── check-scope
│   ├── check-test-integrity
│   └── check-run-state
└── runs/
    └── <task-id>/
        ├── task.md
        ├── baseline.json
        ├── investigation.md
        ├── plan.md
        ├── decisions.md
        ├── review.md
        ├── teach-back.md
        └── evidence.json
```

Personal Harness core는 `implementation-planning`, `code-review` 같은 capability만 요청하고 adapter가 이를 `superpowers/writing-plans`, `superpowers/requesting-code-review`에 연결한다. Superpowers의 실제 명령 이름을 core에 직접 넣지 않아야 provider를 교체하거나 업데이트할 때 검증 계약을 유지할 수 있다.

## 작업별 Profile

### Lean

오타, 문구와 작은 설정처럼 영향이 명확한 작업에 사용한다.

```text
scope 확인
→ 수정
→ diff review
→ verify
→ 변경 이유 설명
```

### Standard

일반적인 기능과 bug 수정의 기본값이다.

```text
Task Contract
→ baseline
→ brainstorming
→ design 승인
→ writing-plans
→ plan 승인
→ 순차 TDD
→ code review
→ teach-back
→ verify
→ close
```

### High-risk

인증, 결제, 개인정보, migration과 public API 변경에 사용한다.

```text
Standard
+ 독립 security·domain review
+ migration·rollback plan
+ recovery drill
+ production monitoring 조건
+ 별도 human approval
```

## 단계별 구축 순서

완성형을 한 번에 만들지 않는다.

1. `Task Contract`, `Teach-back`, 완료 checklist를 Markdown으로 수동 운영한다.
2. 실제 작업에서 자주 빠지는 단계와 불필요한 절차를 기록한다.
3. 반복되는 test·lint·build를 단일 `verify` 명령으로 묶는다.
4. 예상 밖 file, test 삭제와 검증 누락을 `Observe` hook으로 기록한다.
5. 반복해서 유효했던 규칙만 `Warn` 단계로 올린다.
6. 위험 명령, 검증 실패와 approval 이후 diff 변경처럼 오탐이 적은 항목만 `Block`한다.
7. Single agent workflow가 안정된 뒤 독립 작업부터 Worktree와 subagent를 추가한다.

처음부터 모든 hook과 adapter를 만들면 harness를 개발하느라 실제 accelerator 훈련을 하지 못할 수 있다. 실제 작업을 `Standard` profile로 수동 실행한 뒤 반복해서 누락되는 부분만 자동화한다.

## 최종 계약

이 harness의 핵심 계약은 다음과 같다.

> Superpowers가 작업 순서를 관리하고, deterministic gate가 거짓 완료를 막으며, Accelerator profile은 사람이 구현 이해와 production 책임을 끝까지 소유하게 한다.

## See Also

- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [개인 AI Engineering Harness 주말 구축 시뮬레이션](personal-ai-engineering-harness-weekend-simulation-2026-09-15.md)
- [AI Coding에서 Code Reading과 Intent 보존](../software-career/ai-coding-code-reading-and-intent-preservation.md)
