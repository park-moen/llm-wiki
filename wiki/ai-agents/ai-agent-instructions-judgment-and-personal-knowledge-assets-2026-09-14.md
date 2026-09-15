# AI Agent 지침과 개인 지식 자산 운영 원칙

> Sources: [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md); [AI Coding Autonomy Experiment와 Human-in-the-loop](ai-coding-autonomy-experiment.md); [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md); [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md); [AI를 활용한 개발자 성장과 Career 판단](../software-career/ai-assisted-engineering-growth-and-career-judgment.md); [Karpathy LLM Wiki 운영 워크플로우](../llm-wiki/karpathy-llm-wiki-workflow.md)
> Official references: [Codex Memories](https://learn.chatgpt.com/docs/customization/memories); [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md); [Scheduled tasks](https://learn.chatgpt.com/docs/automations) (확인일: 2026-09-14)
> Origin: 2026-09-14 AI Agent 강의 내용에 관한 사용자 대화 (not stored in `raw/`)
> Archived: 2026-09-14

## Overview

AI Agent를 잘 쓴다는 것은 좋은 모델을 구독하거나 긴 prompt를 작성하는 데 그치지 않는다. Agent가 최신 정본을 확인하게 하고, 사용자의 전제를 비판할 수 있는 지침을 주며, 결과를 독립적으로 검증하고, 반복해서 얻은 판단을 계정 밖의 문서로 축적해야 한다. 구독 계정과 memory는 편의 기능이고, Markdown·Git으로 관리하는 지침·결정·실패 기록과 이를 다루는 사람의 판단 능력이 이전 가능한 개인 자산이다.

## 공식 문서를 기반으로 설계한다는 의미

“OpenAI와 Anthropic 공식 문서를 기반으로 설계하라”는 말은 특정 회사의 구조를 그대로 복제하라는 뜻보다, agent의 작업 지침에 **정본 확인 절차**를 넣으라는 의미로 해석하는 편이 실용적이다.

```markdown
설계하기 전에 OpenAI와 Anthropic의 관련 공식 문서를 확인한다.
모델의 기억만으로 API, 지원 기능과 제한을 추측하지 않는다.
공식 문서에서 확인한 사실과 현재 작업에 맞춘 판단을 구분한다.
두 문서가 다르면 공통점과 차이를 밝히고 선택 이유를 기록한다.
문서 전체를 넣지 말고 현재 결정에 필요한 부분만 읽는다.
```

이 방식은 검색, 문서 읽기와 비교가 추가되므로 시간과 token을 더 사용할 수 있다. 모델 자체가 학습되어 더 똑똑해지는 것은 아니다. 필요한 시점에 더 정확하고 최신인 context를 제공하여 추측을 줄이고 판단 근거를 남기는 방식이다.

```text
기억에 의존한 요청
→ 바로 설계
→ 빠르지만 오래된 정보와 추측이 섞일 수 있음

공식 문서 기반 요청
→ 정본 탐색 → 관련 부분 확인 → 차이 비교 → 설계
→ 비용은 늘지만 정확성과 추적 가능성이 좋아짐
```

모든 공식 문서를 항상 prompt에 붙이면 중요한 지침이 묻히고 context 비용만 커질 수 있다. 많은 정보를 미리 넣는 것보다 agent가 필요한 순간에 정본을 찾도록 만드는 편이 낫다. 이는 [AI Coding Autonomy Experiment와 Human-in-the-loop](ai-coding-autonomy-experiment.md)의 reference application 활용 원칙과도 연결된다.

## Agent는 사용자의 수준을 넘지 못한다는 말

“AI Agent는 사용하는 사람보다 똑똑해질 수 없다”는 문장을 문자 그대로 받아들이면 틀리다. AI는 사용자가 모르는 API, 설계 대안과 구현 방법을 제시할 수 있고 좁은 과제에서는 사용자의 현재 지식보다 나은 결과를 만들 수 있다.

다만 결과가 사용자의 사고 범위와 승인 기준을 따라가기 쉽다는 경험칙은 유효하다.

- 사용자가 제시한 해결책을 전제로 받아들이기 쉽다.
- 대화의 표현과 feedback에서 원하는 설명 깊이와 형식을 추정한다.
- 프로젝트 고유의 목표·취향·위험 기준은 context로 제공하지 않으면 알 수 없다.
- 사용자가 오류를 발견하지 못하면 잘못된 결과도 승인되어 다음 작업의 기준이 될 수 있다.
- 빠른 완료만 보상하면 반론·대안·검증보다 구현 속도에 맞춘 답변이 반복될 수 있다.

예를 들어 `JWT로 로그인을 구현해줘`라는 요청은 JWT 채택을 이미 확정한 것으로 읽힐 수 있다. 다음처럼 전제 검토를 지시하면 사고 범위가 달라진다.

```markdown
내가 제시한 해결책을 정답으로 가정하지 않는다.

설계하기 전에 다음을 확인한다.

1. 요구사항에 포함된 숨은 가정
2. 놓친 대안과 반대 근거
3. 이 접근이 실패하는 조건
4. 숙련된 개발자가 먼저 물어볼 질문
5. 공식 문서와 실제 코드로 검증할 항목
6. 현재 정보만으로 결정할 수 없는 부분

내 의견에 동의하는 것보다 잘못된 전제를 찾는 일을 우선한다.
```

결과 품질은 모델 성능 하나로 결정되지 않는다.

```text
결과 품질
≈ 모델 능력
× 제공된 context
× 질문이 허용한 사고 범위
× 검증 기준
× feedback 품질
```

이 식은 측정 공식이 아니라 관계를 이해하기 위한 표현이다. 핵심은 agent의 능력이 사용자 지식에 의해 절대적으로 제한되는 것이 아니라, 사용자가 만든 문제의 틀과 완료 기준이 실제로 꺼내 쓰는 능력의 범위를 제한한다는 점이다. [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)는 agent에게 부족한 프로젝트의 취향과 판단 기준을 context와 review로 보완해야 한다고 설명한다.

## 구독 계정과 개인 지식 자산을 구분한다

AI를 오래 사용하면 여러 종류의 상태가 쌓인다. 이를 모두 하나의 “계정 지능”으로 보면 무엇을 소유하고 무엇을 잃을 수 있는지 판단하기 어렵다.

| 층 | 실제 역할 | 이전 가능성 |
|---|---|---|
| 구독권과 model 접근 | 일정 기간 model과 제품 기능을 사용할 권리 | 구독 종료·계정 변경에 종속됨 |
| Session history | 이전 대화를 다시 열고 이어 가는 기록 | 제품과 계정 또는 local profile에 종속될 수 있음 |
| Codex memories | 일부 이전 대화에서 추출한 context를 다음 session에 주입 | 생성된 상태이며 정본으로 삼기 어려움 |
| `AGENTS.md`·Skill·playbook | 반복 작업의 지침, 판단 기준과 절차 | File과 Git으로 이전 가능 |
| 결정·실패·검증 기록 | 무엇을 왜 선택했고 무엇이 작동하지 않았는지에 관한 근거 | 공개 형식으로 보관하면 다른 agent에서도 사용 가능 |
| 사람의 mental model | 문제 분해, 설계 판단, 검증과 복구 능력 | 도구와 계정이 바뀌어도 남음 |

Codex 공식 문서 기준으로 local memories는 기본적으로 꺼져 있다. 활성화하면 적합한 과거 chat에서 유용한 context를 추출해 기본 Codex home의 `memories/`에 local file로 저장하고 이후 session에 주입할 수 있다. 모든 대화를 그대로 기억하거나 개인 전용 model을 계속 fine-tuning하는 기능은 아니다.

대화 당시 현재 Codex profile을 확인한 결과도 다음과 같았다.

```text
memories    stable    false
```

따라서 해당 profile은 이 시점에 local memories를 축적하고 있지 않았다. 이 값은 2026-09-14의 local 상태이므로 이후 설정 변경을 반영하지 않는다. Memory를 활성화하더라도 생성된 보조 상태로 취급하고, 장기 지식의 정본은 사람이 검토할 수 있는 저장소에 둔다.

## 개인 Git 저장소를 정본으로 삼는다

회사 자료의 개인 보관이 명시적으로 허용되었다는 전제라면, 개인 Codex가 업무 session을 정리하고 private Git 저장소에 축적하는 방식은 계정 의존성을 낮춘다. 핵심은 “개인 Codex를 계속 학습시킨다”가 아니라 **업무 경험을 provider와 구독에 종속되지 않는 file로 변환한다**는 것이다.

```text
회사·개인 Agent session
        ↓
launchd로 다음 날 정리 실행
        ↓
날짜별 source와 daily digest
        ↓
주간 통합
        ↓
Pattern · playbook · AGENTS.md · Skill · Wiki
        ↓
Private Git remote에 이력 보존
```

### 매일 수집하고 주기적으로 통합한다

날짜별 요약만 계속 쌓으면 검색하기 어려운 일기 보관함이 된다. 수집과 지식 통합을 분리한다.

1. **매일:** 전날 session의 문제, 결정, 실패, 검증 결과와 남은 질문을 보존한다.
2. **매주:** 반복된 판단을 pattern과 playbook으로 통합한다.
3. **반복 규칙:** 여러 작업에서 효과가 확인된 절차만 `AGENTS.md`나 Skill로 승격한다.
4. **검증:** Agent가 작성한 요약을 실제 code, 문서와 실행 결과에 대조한다.
5. **이력:** 변경 이유가 드러나도록 Git commit을 남기고 private remote에 보존한다.

예를 들어 매일 기록에서 다음 실패가 반복될 수 있다.

```text
- Agent가 공식 문서를 확인하지 않고 API를 추측함
- Test가 실패했는데 작업을 완료했다고 보고함
```

이를 장기 자산으로 바꾸면 다음과 같다.

```text
Pattern
→ 최신 API 작업은 공식 문서를 먼저 확인한다.
→ 완료 선언과 실제 검증 결과를 분리한다.

영속 지침
→ AGENTS.md에 source 확인 규칙 추가
→ Repository verify command와 completion gate 추가
```

`AGENTS.md`는 Codex가 작업 전에 읽는 프로젝트 지침이므로, 계정 memory에 맡기는 것보다 적용 범위와 변경 이력이 분명하다. 단, 지침은 agent 행동을 유도하는 산문 규칙이다. 반드시 지켜야 하는 test·보안·완료 조건은 CI, hook이나 검증 command 같은 독립된 장치로 보강한다. 자세한 구분은 [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)를 참고한다.

### 저장 구조 예시

```text
ai-career-memory/
├── inbox/       # 아직 분류하지 않은 session 요약
├── daily/       # 날짜별 결정·실패·학습 기록
├── decisions/   # 중요한 선택과 대안
├── patterns/    # 여러 작업에서 반복된 관찰
├── playbooks/   # 재현 가능한 작업 절차
├── skills/      # Agent가 실행할 지침과 자료
└── AGENTS.md    # 저장소 전체의 기본 작업 규칙
```

현재 LLM Wiki를 그대로 사용한다면 session export나 제공된 요약을 `raw/`에 불변 원본으로 보관하고, 여러 날에 걸쳐 검증된 지식을 일반 `wiki/`에 통합하며, 특정 시점의 결론만 Archive로 남기는 구조를 사용할 수 있다. 세 형식의 차이는 [Karpathy LLM Wiki 운영 워크플로우](../llm-wiki/karpathy-llm-wiki-workflow.md)에 정리되어 있다.

## Scheduled task를 운영할 때의 기준

Codex 공식 문서에 따르면 같은 chat에 연결된 scheduled task는 기존 chat context를 이어서 사용하고, 독립된 scheduled task는 매 실행을 별도 run으로 시작한다. 어느 방식을 사용하든 장기 지침은 task prompt나 Skill에, source는 접근 가능한 project·upload·연결 서비스에 두는 편이 안정적이다.

매일 오전 10시 9분에 전날 기록을 정리하는 작업에는 다음 조건을 둔다.

- 처리 대상 날짜를 명시하여 같은 session을 중복 정리하지 않는다.
- 입력이 없으면 새 문서를 만들지 않고 상태만 기록한다.
- 원본 요약과 agent가 도출한 판단을 구분한다.
- 기존 pattern과 충돌하면 자동으로 덮어쓰지 않고 검토 대상으로 표시한다.
- 인증 정보와 secret은 source·요약·Git history에 저장하지 않는다.
- 자동 생성 결과는 바로 영속 지침으로 승격하지 않고 반복 여부를 확인한다.
- 실패한 실행과 부분 완료도 log로 남겨 다음 실행이 성공으로 오인하지 않게 한다.

## 최종 운영 원칙

- 공식 문서는 agent가 필요한 순간에 찾아 쓰는 정본이다.
- 많은 token 사용은 더 많은 탐색과 검증의 비용일 수 있지만 지능 향상 자체는 아니다.
- AI는 사용자의 지식을 넘는 답을 낼 수 있지만 사용자의 좁은 전제와 낮은 승인 기준에 맞춰질 수 있다.
- 구독과 memory는 편의 기능이며 개인 model을 소유하는 것과 다르다.
- Session history는 기록이지만 그대로는 재사용 가능한 지식이 아니다.
- 반복되는 결정과 실패를 문서화하고 Git으로 관리해야 이전 가능한 자산이 된다.
- `AGENTS.md`와 Skill은 작업 방식을 전달하고, test·CI·hook은 중요한 조건을 독립적으로 검증한다.
- 최종 자산은 대화량이 아니라 설명하고 검증하고 다시 적용할 수 있는 판단 기준이다.

## See Also

- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
- [AI Coding Autonomy Experiment와 Human-in-the-loop](ai-coding-autonomy-experiment.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI를 활용한 개발자 성장과 Career 판단](../software-career/ai-assisted-engineering-growth-and-career-judgment.md)
- [Karpathy LLM Wiki 운영 워크플로우](../llm-wiki/karpathy-llm-wiki-workflow.md)
