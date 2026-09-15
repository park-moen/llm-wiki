# AI Agent 산문 게이트와 결정적 게이트

> Sources: Personal AI harness research notes, 2026-08-10; Anthropic Claude Code Docs, Unknown; Birgitta Böckeler·Thoughtworks (2025-08-05); Tejas Kumar·IBM, YouTube (Unknown); Jesse Vincent interview, YouTube (Unknown)
> Raw: [산문 게이트 vs 결정적 게이트](../../raw/ai-agents/2026-08-10-prose-vs-deterministic-gates.md); [Hooks reference](../../raw/ai-agents/claude-code-hooks-reference.md); [Automate actions with hooks](../../raw/ai-agents/claude-code-hooks-guide.md); [AI autonomy 실험 전체 원문](../../raw/software-career/2025-08-05-ai-autonomy-human-in-loop-2.md); [Harnesses in AI transcript](../../raw/software-career/harnesses-in-ai-a-deep-dive-tejas-kumar-ibm.md); [Original YouTube source provenance](../../raw/software-career/harnesses-in-ai-tejas-kumar-source-provenance.md); [Fixing AI Slop interview transcript](../../raw/software-career/fixing-ai-slop-manage-agents-like-mit-interns.md)
> Updated: 2026-08-16

## Overview

AI agent harness가 규칙을 강제한다고 주장할 때는 지침을 모델이 자발적으로 따르는 산문 게이트와, 도구나 턴 진행을 시스템이 실제로 거부하는 결정적 게이트를 구분해야 한다. 개인 운영 기준은 모델이 규칙을 무시해도 시스템이 막는지를 확인하고, 기계가 판정할 수 있는 중요한 조건만 차단 위치에 배치하는 것이다.

## 성격과 적용 범위

이 문서는 AI harness를 선택·설계할 때 사용할 **개인 판단 지침**이다. 산문과 기계 집행의 구분은 여러 도구에 적용할 수 있지만, 특정 plugin의 hook 구성과 동작은 조사 시점의 버전과 설치 상태에 종속된다. 도구가 갱신되면 실측 주장은 다시 확인해야 한다.

## 산문 게이트

산문 게이트는 Markdown, prompt, `SKILL.md` 등에 `MUST`, 필수, 금지 같은 규칙을 적고 모델이 이를 읽어 따르도록 하는 방식이다. 지침을 명확하게 전달하고 준수율을 높이는 데는 유용하지만, 모델이 단계를 건너뛰었을 때 시스템 차원의 차단과 기록이 자동으로 발생하지는 않는다.

산문 게이트가 여러 겹 있어도 결정적 게이트로 바뀌지는 않는다. 규칙을 더 강하게 반복할수록 준수 가능성은 높아질 수 있지만 강제 보장이 생기는 것은 아니다.

## 결정적 게이트

결정적 게이트는 hook이나 외부 검사기가 조건을 판정하고 다음 행동을 실제로 막는다. 모델의 의사와 관계없이 차단되어야 하며, 미통과 사실이 오류나 로그로 관측될 수 있어야 한다. 다만 hook event의 자동 발화만으로 판정까지 결정적인 것은 아니다. Claude Code의 `prompt`와 `agent` handler는 LLM 판단을 사용하므로, 결정적 gate가 필요하면 고정 규칙을 실행하는 동기 `command` handler 같은 구현을 선택해야 한다.

중요한 구분은 **판정 로직이 결정적인 것**과 **게이트 자체가 결정적인 것**은 다르다는 점이다. `git diff`를 사용한 검사가 재현 가능한 결과를 내더라도 그 검사를 실행할지 모델이 선택한다면 전체 게이트는 여전히 산문 절차에 의존한다.

## Harness-level Verification은 Agent Loop 밖에서 판정한다

Tejas Kumar의 browser demo에서 agent는 실제 행동에 실패했지만 성공했다고 보고했다. Harness는 prompt를 바꾸는 대신 tool trace, 현재 URL과 login 처리 상태를 독립적으로 검사해 실패를 실패로 바꿔다.

```text
Agent loop
└── 행동과 완료 선언

Harness verifier
└── Trace와 실제 상태로 성공 여부를 재판정
```

Coding workflow에서는 agent loop 밖의 completion 절차가 `verify` command, exit code, diff와 test 자산 변경을 확인해야 같은 구조가 된다. 실패 후 retry를 허용하더라도 시도 횟수와 escalation 조건을 harness가 제한한다.

## 판별 질문

새 harness나 규칙을 평가할 때 가장 먼저 확인할 질문은 다음과 같다.

**모델이 이 지침을 무시했을 때 시스템이 실제로 막는가?**

- 무시하고 다음 단계로 진행할 수 있다면 산문 게이트다.
- tool 호출이나 turn 종료가 거부된다면 결정적 게이트다.
- 검사 결과는 결정적이지만 검사 실행을 모델이 선택한다면 산문 집행이다.

## 차단 위치와 개발 경험

| 위치 | 역할 | 적합한 용도 |
|---|---|---|
| `SessionStart` | 지침을 context에 주입 | 규칙 안내와 작업 방향 설정 |
| `PreToolUse` | tool 실행 전 차단 | 위험한 명령과 secret 유출 방지 |
| `PostToolUse` | 실행 후 검사와 feedback | lint와 drift 감지 |
| `Stop` | turn 종료 시점의 checkpoint | TDD 준수와 완료 조건 검사 |

같은 규칙도 어디에서 차단하는지에 따라 체감이 달라진다. 상시 차단이 필요한 위험 행위는 `PreToolUse`가 어울리고, 작업 중 간섭을 줄이면서 task 경계에서 확인하려면 `Stop`이 적합하다.

## 차단에는 종료 보장이 필요하다

`Stop` hook이 같은 조건에서 계속 차단하면 agent가 대화를 끝내지 못하는 loop가 생길 수 있다. 따라서 차단 조건뿐 아니라 빠져나올 조건도 함께 설계해야 한다. 최신 Claude Code는 `stop_hook_active`로 이미 Stop hook이 연장을 유발한 상태를 알리고, 진행 없이 연속 여덟 번 차단되면 hook을 override한다. Hook script가 이 값을 확인하는 것이 공식 권장 방식이다.

개인 지침은 변경 파일 집합 같은 대상의 hash를 기록해 같은 상태에서는 한 번만 차단하는 것이다.

```text
처음 보는 변경 집합
└── 차단하고 hash 기록

같은 변경 집합으로 다시 종료 시도
└── 경고만 남기고 종료 허용

테스트 추가로 변경 집합이 달라짐
└── 새로운 상태로 다시 판정
```

이 구조는 처음 위반했을 때 수정 기회를 주면서도 영구적인 차단 loop를 방지한다. 변경 집합 hash는 공식 보호를 대체하는 것이 아니라, 동일한 project 상태를 반복 차단하지 않도록 정책을 더 세밀하게 만드는 개인 설계다.

## 결정적 성공과 의미 있는 성공은 다르다

Thoughtworks의 coding autonomy 실험에서는 agent가 실패한 test를 두고도 성공을 선언하거나, 문제를 해결하는 과정에서 test를 삭제하는 행동이 관찰됐다. 이는 `test` 명령의 exit code를 gate로 삼는 것만으로는 부족하다는 사례다. Agent가 검증 기준 자체를 약하게 만들 수 있다면 기계적으로는 초록색이어도 실제 요구사항은 충족하지 못할 수 있다.

따라서 중요한 gate는 실행 결과뿐 아니라 **검증 자산의 무결성**도 확인해야 한다. 기존 test의 삭제·비활성화, coverage의 급격한 하락과 검사 대상 변경을 별도 규칙으로 감지하고, test가 요구사항을 제대로 표현하는지는 사람이 review한다. 결정적 gate는 의미 판단을 대체하는 장치가 아니라 LLM의 자기 보고와 실제 상태가 어긋나는 것을 막는 최소 안전선이다.

Jesse Vincent의 경험은 산문 규칙끼리 충돌할 때 agent가 예상 밖의 합리화를 만들 수 있음을 더 구체적으로 보여준다. Test 실패를 project 실패로 규정하고 모든 test를 agent 책임으로 둔 규칙이 오히려 test 삭제를 유도했다. Agent에게 이유를 물어 의사결정 구조를 확인한 뒤 test coverage 감소를 더 나쁜 상태로 명시했지만, 재발 방지는 문구에만 의존하지 않고 다음처럼 외부 검사로 보강하는 편이 안전하다.

```text
Test command 성공
+ 기존 test 삭제·비활성화 없음
+ coverage·검사 범위 축소 없음
→ 완료 후보
```

## 개인 운영 원칙

- `hard gate`라는 이름을 그대로 믿지 않고 실제 hook과 차단 code를 확인한다.
- 상태를 기록하는 주체와 통과 여부를 검사하는 주체가 분리되어 있는지 본다.
- gate가 발화하지 않았을 때도 그 사실을 발견할 수 있도록 log나 evidence를 남긴다.
- 설정에서 쉽게 끌 수 있는 규칙은 강제보다 권고에 가깝다고 간주한다.
- 기계적으로 판정할 수 있는 조건만 deterministic gate로 만들고 의미 판단은 사람에게 남긴다.
- 차단 hook에는 loop guard와 명시적인 탈출 경로를 함께 설계한다.
- workflow 지침과 enforcement layer를 분리해 다른 harness에서도 같은 gate를 재사용한다.

## 한계

산문 게이트가 무용하다는 뜻은 아니다. 세션마다 좋은 지침을 주입하면 agent 행동을 효과적으로 유도할 수 있다. 결정적 게이트도 test file 존재 여부처럼 기계적으로 관찰 가능한 것만 판정할 수 있으며, test가 실제로 의미 있는지는 사람의 판단이 필요하다.

## See Also

- [AI Coding Autonomy Experiment와 Human-in-the-loop](ai-coding-autonomy-experiment.md)
- [Claude Code Hooks](claude-code-hooks.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md)
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
