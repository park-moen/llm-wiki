# Vercel Agent Skills·Superpowers·gstack·Harness Engineering 비교

> Sources: [find-skills로 Agent Skill 탐색과 설치하기](find-skills-discovery-and-installation.md); [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md); [gstack으로 AI 개발 Workflow 이해하기](gstack-ai-engineering-workflow.md); [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
> Archived: 2026-09-16

## Overview

Vercel Agent Skills, Superpowers, gstack과 Harness Engineering은 경쟁 제품이라기보다 서로 다른 계층을 담당한다. Vercel Agent Skills는 React·Next.js·Vercel 같은 특정 영역의 전문 지식을 제공하고, Superpowers는 개발 작업의 순서와 승인 지점을 규정한다. gstack은 역할별 Skill에 browser·QA·출하 도구를 결합하며, Harness Engineering은 이 모든 요소 아래에서 tool·권한·검증·retry를 실제로 통제한다.

## 두 Vercel 저장소의 역할

이름이 비슷하지만 역할은 다르다.

| 저장소 | 역할 |
|---|---|
| [`vercel-labs/agent-skills`](https://github.com/vercel-labs/agent-skills) | Vercel이 관리하는 실제 Skill 모음 |
| [`vercel-labs/skills`](https://github.com/vercel-labs/skills) | `npx skills` CLI와 `find-skills` 등 Skill 탐색·설치 생태계 |

`find-skills`는 `vercel-labs/skills`에 있고, `react-best-practices` 같은 기술 Skill은 `vercel-labs/agent-skills`에 있다.

## 네 가지 계층 비교

| 구분 | Vercel Agent Skills | Superpowers | gstack | Harness Engineering |
|---|---|---|---|---|
| 주된 질문 | 이 기술을 어떻게 잘 사용할까? | 개발 작업을 어떤 순서로 진행할까? | 제품팀의 여러 역할을 어떻게 Agent workflow로 만들까? | Agent가 규칙을 지키고 성공했는지 어떻게 통제할까? |
| 중심 자산 | 기술별 규칙·checklist·script | brainstorming·계획·TDD·review 절차 | 제품·설계·review·browser QA·출하 명령 | tool registry·context·trace·verifier·retry·권한 |
| 적용 범위 | React, Next.js, Vercel, UI와 문서 등 특정 영역 | 범용 software 개발 lifecycle | 제품 개발 lifecycle과 실제 browser 작업 | Agent 실행 환경 전체 |
| 규칙 성격 | 전문 지식과 검토 기준 | 강하게 정해진 workflow | 역할별 workflow와 실행 도구의 결합 | code·hook·CI를 통한 기계적 통제 |
| 주요 위험 | 특정 stack에 치우치거나 규칙을 과하게 적용할 수 있음 | 작은 작업에도 절차가 무거워질 수 있음 | browser session·배포 등 권한 범위가 큼 | 잘못된 verifier가 잘못된 결과를 안정적으로 반복할 수 있음 |

## Vercel Agent Skills: 기술별 전문 지식

`vercel-labs/agent-skills`는 Agent Skills 형식에 맞춘 `SKILL.md`와 선택적인 `scripts/`, `references/`로 Agent의 특정 기술 역량을 확장한다. 대표적인 범주는 다음과 같다.

- `react-best-practices`: React·Next.js 성능 규칙
- `composition-patterns`: 확장 가능한 React component 합성
- `web-design-guidelines`: 접근성·UX·성능 검토
- `react-native-guidelines`: React Native·Expo 지침
- `writing-guidelines`: Vercel 문서 작성 기준
- `vercel-optimize`: 실제 Vercel 지표를 바탕으로 비용·성능 조사
- `vercel-deploy-claimable`: Vercel 배포

따라서 Vercel Agent Skills는 “이 Next.js 구현에서 어떤 패턴을 따라야 하는가?”에는 강하지만, 실제 요구사항을 찾고 대안을 비교해 전체 개발 순서를 정하는 범용 workflow를 기본 목표로 삼지는 않는다. 다만 `vercel-optimize`처럼 지표 수집부터 병목 조사까지 포함하는 Skill은 단순 지식집보다 workflow 성격이 강하므로 각 Skill의 실제 내용과 실행 범위를 따로 확인해야 한다.

## Superpowers: 반복 가능한 개발 방법론

Superpowers는 특정 framework 지식보다 개발 단계를 분리하는 데 초점을 둔다.

```text
brainstorming
→ writing-plans
→ executing-plans
→ test-driven-development
→ requesting-code-review
→ verification-before-completion
```

이 workflow는 구현 전에 실제 문제와 요구사항을 확인하고, 승인된 설계를 작은 task와 RED–GREEN TDD로 옮기며, 구현자와 reviewer의 역할을 분리한다. 완료 선언에는 새로 실행한 검증 증거를 요구한다.

Vercel Skill과 Superpowers는 대체 관계가 아니다. Superpowers가 무엇을 어떤 순서로 만들지 관리하고, Vercel Skill이 React·Next.js 구현의 세부 품질 기준을 제공하는 방식으로 조합할 수 있다.

## gstack: 역할별 Skill과 실행 도구의 결합

gstack은 Markdown 지침만 모아 둔 저장소보다 넓다. 제품 탐색, engineering plan review, design review, code review, 실제 browser 기반 QA와 출하 흐름을 역할별 명령으로 제공한다.

```text
/office-hours
→ /plan-eng-review
→ 구현
→ /review
→ /qa-only
→ /ship
```

browser daemon, 로그인 상태 유지, screenshot과 실제 화면 검증처럼 실행 code도 포함한다. Vercel Agent Skills가 전문 reference에 가깝다면 gstack은 제품 책임자·engineering manager·designer·reviewer·QA의 작업 절차를 묶은 작은 software factory에 가깝다. 실제 계정, browser session과 배포에 접근할 수 있으므로 명령 이름뿐 아니라 권한과 외부 변경 범위를 확인해야 한다.

## Harness Engineering: 산문 지침을 실행 통제로 보강

Skill에 “반드시 test를 실행한다”거나 “실패하면 완료하지 않는다”고 적어도 그 자체는 자연어 지침이다. Agent가 규칙을 건너뛰었을 때 Skill 문서만으로 실행을 차단하거나 독립된 실패 기록을 남기지는 못한다.

Harness는 Agent 바깥에서 완료 여부를 다시 판정한다.

```text
Agent가 완료 선언
→ Harness가 test·lint·typecheck·build 실행
→ exit code와 diff 확인
   ├─ 성공: 완료 허용
   └─ 실패: 완료 거부 후 제한된 retry 또는 사람에게 escalation
```

Harness Engineering에는 tool과 권한, context 구성, tool trace, 위험 명령 차단, deterministic verification, retry 상한, human escalation과 secret 경계가 포함된다. Skill이 판단 방법을 제공한다면 harness는 그 판단이 실제 행동과 결과로 이어졌는지 통제한다.

## 유명 Engineering Skill 모음과의 차이

범용 Engineering Skill 모음은 대체로 다음 판단 과정을 독립 Skill로 나눈다.

- 요구사항 명확화
- 설계 후보와 trade-off 비교
- API와 interface 경계 설계
- 작업 분해
- root cause debugging
- TDD와 완료 검증
- code review
- security·performance 검토
- ADR 작성

이들은 Vercel Agent Skills보다 software engineering의 판단 workflow에 가깝다. 반대로 Vercel Agent Skills는 특정 기술에서 어떤 구현이 바람직한지에 집중한다. 어느 쪽이 더 상위인 것이 아니라, 범용 workflow와 domain 지식을 함께 사용하되 동일한 역할을 가진 Skill을 중복 실행하지 않는 것이 중요하다.

## 권장 조합

```text
개발 판단 workflow
├── Superpowers 또는 선별한 Engineering Skill

기술별 전문 지식
├── Vercel react-best-practices
├── composition-patterns
└── web-design-guidelines

제품·UI 검증
└── gstack /review, /qa-only

실제 완료 판정
├── Repository verify command
├── Git hook
└── CI

최종 결정
└── 사람
```

Vercel Agent Skills는 좋은 기술 지식을 제공하고, Superpowers는 좋은 개발 순서를 제공하며, gstack은 역할과 실행 도구를 묶는다. Harness는 이 요소들이 실제로 지켜졌는지 통제한다. 공통 workflow를 Skill에서 빌리더라도 프로젝트의 성공 조건, 검증 command와 최종 책임까지 외부 Skill에 넘기지는 않는다.

## See Also

- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [Superpowers를 지속 사용하는 Harness 운영 전략](superpowers-continuous-use-harness-strategy.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [Matt Pocock Skills의 Repository 설정 방식](matt-pocock-skills-repository-setup.md)
