# Superpowers를 지속 사용하는 Harness 운영 전략

> Sources: [Superpowers Brownfield 실전 가이드](superpowers-brownfield-field-guide.md); [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md); [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md); [취업 초기 주니어를 위한 AI-Native 개발 프로세스 2: Harness Engineering](../software-career/junior-ai-native-development-harness-engineering.md)
> Archived: 2026-08-16

## Overview

Superpowers는 계속 사용해도 되지만 완성된 정답이나 품질 책임을 대신하는 시스템으로 보지 않는다. 공통 workflow는 Superpowers에서 빌리고, 프로젝트 context·verification·gate·domain 승인은 repository와 팀에 맞게 붙인다. 새 harness를 처음부터 다시 만들기보다 실제 Issue에서 Skill 수행 여부와 검증 증거를 관찰하고, 반복되는 마찰만 제한적으로 보완하는 운영이 적절하다.

## 계속 사용해도 되는 이유

Superpowers의 재사용 가치는 개별 prompt 문구보다 개발 단계를 분리하는 workflow에 있다.

```text
brainstorming
→ writing-plans
→ executing-plans
→ test-driven-development
→ requesting-code-review
→ receiving-code-review
→ finishing-a-development-branch
```

이 구조는 다음 원칙을 반복하게 돕는다.

- 구현 전에 문제·요구사항·설계를 분리한다.
- Plan과 구현을 별도 checkpoint로 다룬다.
- TDD를 구현 단계 안의 feedback loop로 두다.
- Review 요청과 review 의견의 수용을 분리한다.
- 구현 완료, 검증 통과와 branch 통합을 다른 상태로 본다.

다만 현재 Wiki의 실측은 특정 Brownfield repository의 품질 baseline Issue를 local에서 실행한 관찰이다. Remote CI와 다양한 기능 Issue까지 보편적으로 검증된 결과로 확대하지 않는다.

## Superpowers에 맡길 것과 남겨둘 것

Superpowers는 주로 **workflow layer**로 사용한다. 프로젝트의 실제 성공 조건은 다른 계층에서 담당한다.

```text
Superpowers
└── 작업 순서와 Skill workflow

Repository instruction
└── 용어, 구조, 제약과 팀 규칙

Verify command
└── test·lint·typecheck·build

Hook·CI
└── 기계적으로 판정할 수 있는 실패 차단

Human checkpoint
└── 요구사항, domain, 설계, security와 production 책임
```

Skill에 TDD와 review가 적혀 있어도 agent가 단계를 건너뛸 수 있다면 산문 지침이다. 반드시 통과해야 하는 조건은 repository command, hook이나 CI가 모델의 자기 보고와 독립적으로 판정하게 한다.

## 작업별로 Skill을 선택한다

Superpowers를 사용한다고 모든 작업에 전체 pipeline을 강제하지 않는다.

### 작은 Bug 수정

```text
systematic-debugging
→ test-driven-development
→ verification-before-completion
```

### 새 기능

```text
brainstorming
→ writing-plans
→ executing-plans
→ test-driven-development
→ requesting-code-review
→ finishing-a-development-branch
```

### Code review 지적 반영

```text
receiving-code-review
→ 지적 재현
→ 수정
→ 전체 verify
```

### 복잡하고 독립적인 작업

```text
brainstorming
→ writing-plans
→ using-git-worktrees
→ subagent-driven-development
→ 통합 review
```

Worktree와 subagent는 작업이 독립적이고 사람이 각 plan·diff·merge를 감당할 수 있을 때만 추가한다.

## 새 Harness를 만들기 전에 보완할 신호

다음 현상이 반복될 때 Superpowers 전체를 교체하기보다 해당 경계를 보완한다.

| 반복 문제 | 보완 방법 |
|---|---|
| 프로젝트 정보를 매번 다시 설명 | Repository instruction |
| 검증 command 누락 | 단일 `verify` command |
| Test 실패에도 완료 선언 | Completion hook·CI gate |
| 위험 command 실행 | Tool 실행 전 gate |
| Review가 형식적으로 끝남 | Review checklist·advisory agent |
| Worktree의 branch·base 혼동 | 생성 script·Run metadata |
| 복잡한 Skill이 작은 작업을 방해 | 작업 위험도별 preset |
| 실패 loop가 종료되지 않음 | Retry 상한·loop guard·human escalation |

중요한 원칙은 workflow instruction과 enforcement layer를 분리하는 것이다. 그러면 Superpowers를 업데이트하거나 다른 harness로 교체해도 프로젝트의 검증 command와 gate를 재사용할 수 있다.

## 실제 Issue를 학습 단위로 삼는다

각 Run에서 다음을 기록한다.

1. 어떤 Skill을 사용했는가?
2. Skill의 목적을 실제로 달성했는가?
3. 어떤 단계가 불필요했거나 누락됐는가?
4. 사람이 어디서 중단·승인·재계획했는가?
5. 어떤 실패를 script·hook·CI가 자동으로 막아야 했는가?
6. 실행한 command, exit code와 남은 위험은 무엇인가?

관찰 결과는 다음과 같이 축적한다.

```text
반복해서 유용한 workflow
→ 유지

반복해서 불필요한 단계
→ 해당 작업 preset에서 제외

반복해서 누락되는 기계적 검사
→ script·hook·CI로 이동

매번 다른 trade-off가 필요한 판단
→ human checkpoint로 유지
```

## 결론

Superpowers는 계속 사용하되 **workflow는 빌리고 프로젝트의 성공 조건과 책임까지 넘기지 않는다.** 사용자의 Harness Engineering 학습은 Superpowers를 재구현하는 데서 시작하는 것이 아니라, 실제 Run에서 산문 workflow와 기계적 enforcement의 경계를 관찰하고 필요한 부분만 보완하는 것에서 시작한다.

## See Also

- [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](superpowers-brownfield-practice-workflow-initial-design.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md)
