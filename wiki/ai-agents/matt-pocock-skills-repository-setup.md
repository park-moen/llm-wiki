# Matt Pocock Skills의 Repository 설정 방식

> Sources: Matt Pocock Skills, Unknown; skills.sh, Unknown
> Raw: [setup-matt-pocock-skills](../../raw/ai-agents/setup-matt-pocock-skills.md)
> Updated: 2026-09-15

## Overview

`setup-matt-pocock-skills`는 기능을 구현하는 Skill이 아니라, 다른 Matt Pocock engineering Skill이 repository의 issue tracker, triage label과 domain 문서 위치를 일관되게 찾도록 설정하는 초기화 Skill이다. 현재 구조를 먼저 조사하고 사용자와 선택을 확인한 뒤 `AGENTS.md` 또는 `CLAUDE.md`와 `docs/agents/` 아래의 작은 설정 문서를 만든다.

## 설치와 Setup은 다른 단계다

설치는 Agent가 Skill 자체를 찾을 수 있게 복사하는 과정이다.

```text
npx skills add https://github.com/mattpocock/skills --skill setup-matt-pocock-skills
```

Setup은 설치된 Skill을 특정 repository에 맞추는 과정이다. Repository마다 issue tracker와 문서 구조가 다르므로 처음 사용할 때 한 번 실행한다. `disable-model-invocation: true`로 선언돼 있어 Agent가 임의로 적용하기보다 사용자가 명시적으로 호출하는 흐름을 전제로 한다.

```text
Skill 설치
  → repository 구조 조사
  → 사용자와 세 가지 선택 확인
  → 설정 초안 검토
  → instruction file과 docs/agents 작성
  → 다른 engineering Skill이 설정을 읽음
```

## 가장 먼저 기존 구조를 조사한다

Setup은 빈 repository라고 가정하지 않는다. Remote, 기존 instruction file, domain 문서, ADR, 이전 setup 결과와 monorepo 신호를 먼저 읽는다.

이 순서가 중요한 이유는 기존 팀 규칙을 보존하기 위해서다. 이미 `CLAUDE.md`가 있는데 `AGENTS.md`를 새로 만들거나, 기존 `## Agent skills` block을 두 번째로 추가하면 어느 규칙이 정본인지 모호해진다. 이 Skill은 기존 file을 우선하고 해당 block만 갱신하도록 설계됐다.

## Repository마다 결정하는 세 가지

### Issue tracker

다른 Skill이 issue를 어디서 읽고 쓸지 기록한다.

| 환경 | 기본 선택 | 실행 방식 |
|---|---|---|
| GitHub remote | GitHub Issues | `gh` CLI |
| GitLab remote | GitLab Issues | `glab` CLI |
| Remote가 없는 개인 project | Local Markdown | `.scratch/<feature>/` |
| Jira·Linear·사내 도구 | Custom workflow | 사용자의 설명을 문서로 기록 |

Jira가 기본 구현으로 내장된 것은 아니다. Jira를 선택하면 project가 사용하는 connector, API, CLI, issue 단위와 승인 규칙을 `docs/agents/issue-tracker.md`에 구체적으로 적어야 downstream Skill이 추측하지 않는다.

### Triage label vocabulary

`triage` Skill이 실제로 설치돼 있을 때만 label mapping을 설정한다. 설치되지 않은 기능을 위해 불필요한 label 문서를 만들지 않는다. 이미 조직에서 다른 label을 사용한다면 기본 label을 새로 만드는 대신 기존 이름과 canonical role의 대응 관계를 기록한다.

### Domain docs

일반 repository는 root `CONTEXT.md`와 `docs/adr/`를 사용하는 single-context가 기본이다. Monorepo 신호가 확인된 경우에만 `CONTEXT-MAP.md`에서 package별 `CONTEXT.md`를 가리키는 multi-context를 검토한다.

Context가 많을수록 좋다고 가정하면 안 된다. 작은 repository를 여러 context로 나누면 Agent가 어느 문서를 읽어야 하는지 판단하는 비용만 늘어날 수 있다.

## 만들어지는 설정 구조

Setup 결과는 대략 다음과 같다.

```text
AGENTS.md 또는 CLAUDE.md
└─ ## Agent skills
   ├─ Issue tracker → docs/agents/issue-tracker.md
   ├─ Triage labels → docs/agents/triage-labels.md
   └─ Domain docs → docs/agents/domain.md

docs/agents/
├─ issue-tracker.md
├─ domain.md
└─ triage-labels.md  # triage Skill이 있을 때만
```

Root instruction file에는 짧은 pointer만 두고 자세한 규칙을 별도 문서로 분리한다. 여러 Skill이 같은 문서를 읽으므로 tracker나 domain layout을 바꿀 때 각 Skill 본문을 따로 수정할 필요가 없다.

## 이 구조를 읽는 downstream Skill

설정 문서는 다음과 같은 작업에서 공통 context가 된다.

- `to-spec`: Spec을 저장하거나 issue tracker에 게시할 위치 결정
- `to-tickets`: 작업을 어떤 issue 형식으로 나눌지 결정
- `triage`: 팀의 label vocabulary 적용
- `tdd`, `diagnose`: domain 용어와 기존 결정 참고
- `improve-codebase-architecture`: context 경계와 ADR을 참고해 개선 후보 탐색

즉 setup의 가치는 file을 몇 개 만드는 데 있지 않다. 여러 Skill이 repository의 작업 위치와 공통 언어를 같은 방식으로 해석하게 만드는 데 있다.

## Prompt-driven Setup의 한계

이 Skill은 deterministic installer가 아니다. Agent가 repository를 탐색하고 문서를 작성하므로 같은 입력에서도 표현과 세부 결과가 달라질 수 있다.

다음 항목은 사용자가 직접 확인해야 한다.

- 감지한 remote와 issue tracker가 실제 운영 방식과 일치하는가?
- 기존 `AGENTS.md` 또는 `CLAUDE.md`의 규칙을 덮어쓰지 않았는가?
- Custom tracker 설명에 인증, 승인과 외부 변경 제한이 포함됐는가?
- Monorepo가 아닌데 불필요한 multi-context 구조를 만들지 않았는가?
- Downstream Skill이 실제로 존재하며 문서 경로를 읽는가?

Skill이 초안을 먼저 보여주는 이유도 이 경계 때문이다. 사용자가 확인하기 전에는 repository 규칙을 확정해서는 안 된다.

## 기존 Workflow에 적용할 때

이미 팀 전용 Jira·commit·MR Skill과 `AGENTS.md` 규칙이 있는 repository라면, 이 setup을 그대로 기본 설정으로 받아들이기보다 연결 계층으로 사용한다.

1. 기존 `AGENTS.md`, Jira workflow와 문서 위치를 정본으로 둔다.
2. `issue-tracker.md`에는 팀 Skill을 우회하지 않도록 실제 호출 규칙을 적는다.
3. Matt Pocock Skill과 기존 Skill의 역할이 겹치면 어느 쪽이 우선하는지 명시한다.
4. `triage`를 쓰지 않으면 label 설정을 생략한다.
5. 처음에는 `grill-with-docs`나 `diagnose`처럼 기존 workflow의 빈 부분만 연결한다.

이렇게 해야 setup이 새로운 병렬 규칙 체계를 만드는 것이 아니라, 기존 규칙을 다른 engineering Skill이 읽을 수 있는 형태로 연결한다.

## See Also

- [gstack으로 AI 개발 Workflow 이해하기](gstack-ai-engineering-workflow.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](../software-career/ai-coding-software-fundamentals.md)
