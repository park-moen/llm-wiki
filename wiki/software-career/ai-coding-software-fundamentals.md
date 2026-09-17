# AI Coding에서 Software Fundamentals가 더 중요해지는 이유

> Sources: Matt Pocock (Unknown); Titus Winters·ACM Tech Talk (Unknown); Boris Cherny interview, YouTube (Unknown); Pasha interview·Beyond Coding (Unknown); Jesse Vincent interview, YouTube (Unknown); zanlib, 2026-09-14
> Raw: [Software Fundamentals Matter More Than Ever transcript](../../raw/software-career/software-fundamentals-matter-more-than-ever.md); [Software Engineering at Google transcript](../../raw/software-career/software-engineering-at-google-tech-talk.md); [Building Claude Code with Boris Cherny transcript](../../raw/software-career/building-claude-code-boris-cherny.md); [Original YouTube source provenance](../../raw/software-career/building-claude-code-boris-cherny-source-provenance.md); [From Backend Engineer to Head of Mobile transcript](../../raw/software-career/from-backend-engineer-to-head-of-mobile-lessons-uber.md); [Fixing AI Slop interview transcript](../../raw/software-career/fixing-ai-slop-manage-agents-like-mit-interns.md); ["Do You Still Read the Code?"](../../raw/software-career/2026-09-14-do-you-still-read-the-code.md)
> Updated: 2026-09-16

## Overview

Matt Pocock의 핵심 주장은 AI가 code를 빠르게 생산할수록 software fundamentals의 가치가 줄어드는 것이 아니라 커진다는 것이다. AI는 잘 설계된 codebase에서는 강력하지만, 변경하기 어려운 codebase에서는 더 많은 code와 더 빠른 변경이 구조적 혼란을 증폭시킨다. 따라서 사람은 specification 작성에 그치지 않고 요구사항의 의미, 공통 언어, module boundary, interface와 feedback loop를 계속 설계해야 한다.

원본은 사용자가 제공한 영어 자동 transcript다. 고유명사 오인식과 문장 분할 오류가 포함될 수 있으므로, 모호한 표현은 발표 영상과 대조해야 한다.

## Code는 왜 싸지 않은가

`specs to code` 접근은 specification을 AI에 전달해 code를 만들고, 문제가 생기면 code 대신 specification만 고쳐 다시 생성하려 한다. 발표자는 이 과정을 반복할수록 code가 나빠지는 경험을 설명한다. 새 요구사항 하나만 보고 변경하면 system 전체의 구조를 고려하지 못해 software entropy가 커지기 때문이다.

문제는 생성 비용보다 유지 비용이다. 좋은 codebase는 변경하기 쉽고, 나쁜 codebase는 작은 변경도 bug와 예상하지 못한 영향을 만든다. AI가 변경 속도를 높이면 나쁜 구조에서 발생하는 비용도 더 빨리 누적된다. 그러므로 code를 읽지 않는 방식은 durable software의 전략이 되기 어렵다.

> **Status: Disputed**
> Jesse Vincent는 큰 diff의 수동 line review보다 사용자 outcome, negative behavior, safety와 reliability를 검증하는 것이 중요하며, 자신은 code를 직접 읽는 시간을 크게 줄였다고 설명한다. 반면 이 문서의 다른 source들은 장기 software의 test·interface·module boundary 자체가 잘못됐는지 판단하려면 사람이 code와 system design을 이해해야 한다고 본다. 두 입장을 단순히 하나로 확정하지 않고, verifier가 강하고 위험이 낮은 내부 구현은 outcome 중심으로 더 위임하되 public interface·security·data integrity·검증 자산과 장기 구조는 사람이 깊게 review하는 위험 기반 절충을 현재 운영 기준으로 둔다.

이 차이는 AI가 작성한 code의 비율보다 유지보수 계약의 차이로 보는 편이 정확하다. 구현 이해를 계속 보유하려는 `accelerator`는 code reading을 통해 mental model과 구현의 drift를 찾는다. 구현을 교체 가능한 산출물로 다루는 `vibecoder`는 specification·context·evaluation에 유지보수 책임을 둔다. 두 번째 방식을 택하면서도 이를 뒷받침할 검증 체계를 만들지 않으면 구현 이해와 durable intent를 모두 잃을 수 있다.

## 사람과 AI가 같은 Design Concept을 가져야 한다

AI가 엉뚱한 결과를 내는 원인을 prompt 표현 하나의 문제로만 보면 안 된다. 사람도 자신이 원하는 것을 처음부터 완전히 알지 못하며, 사람과 AI 사이에 만들 대상에 대한 공통 mental model이 없을 수 있다. 발표에서는 이를 `design concept`으로 설명한다.

작업을 바로 시작하기 전에 AI가 요구사항과 결정 사항을 집요하게 질문하도록 만들면 다음을 먼저 드러낼 수 있다.

- 사용자와 해결하려는 문제
- 범위에 포함되는 것과 제외되는 것
- 서로 의존하는 설계 결정
- 정상 동작과 failure mode
- 아직 답하지 못한 가정

질문의 목적은 긴 계획 문서를 빨리 만드는 것이 아니라, 구현 전에 사람과 AI가 같은 것을 상상하도록 만드는 것이다.

## Ubiquitous Language로 대화 비용을 줄인다

사람, domain expert와 AI가 서로 다른 단어를 사용하면 요구사항과 code가 어긋난다. Domain-Driven Design의 `ubiquitous language`는 대화, domain model과 code에서 같은 용어를 같은 의미로 사용하는 방법이다.

Project의 주요 용어를 짧은 문서로 관리하고 class, function, API와 대화에 일관되게 사용하면 AI가 불필요하게 장황해지는 문제를 줄이고 구현을 계획에 맞출 수 있다. 중요한 것은 용어 목록 자체보다 각 용어의 의미와 경계를 합의하는 것이다.

## Feedback 속도가 개발 속도의 상한이다

AI는 많은 code를 만든 뒤에야 type check나 test를 실행하려는 경향을 보일 수 있다. 그러나 feedback을 받기 전에 너무 멀리 진행하면 여러 오류가 한꺼번에 쌓이고 원인을 찾기 어려워진다.

발표가 권하는 흐름은 작은 단계를 강제하는 TDD다.

1. 기대 behavior를 test로 표현한다.
2. Test를 통과할 만큼만 구현한다.
3. Test가 보호하는 상태에서 구조를 refactoring한다.
4. 다음 behavior로 이동한다.

Static type, automated test와 browser feedback 같은 guardrail이 있어도 AI가 스스로 적절한 시점에 사용한다고 가정해서는 안 된다. Repository의 workflow가 작은 변경과 잦은 검증을 요구하도록 만들어야 한다.

## Deep Module과 단순한 Interface

발표는 test하기 쉬운 codebase의 구조로 `deep module`을 제시한다.

- Deep module: 많은 기능과 복잡성을 내부에 감추고 단순한 interface를 제공한다.
- Shallow module: 제공하는 기능은 적지만 외부에 복잡한 interface와 많은 의존성을 노출한다.

작고 얕은 module이 지나치게 많으면 사람과 AI 모두 여러 file과 dependency를 계속 따라가야 한다. 반대로 관련된 책임을 명확한 boundary 안에 모으고 단순한 interface를 제공하면, 외부에서는 내부 구현을 전부 알지 않고도 module을 사용하고 검증할 수 있다.

Deep module은 무조건 큰 file을 만들라는 뜻이 아니다. 핵심은 내부 복잡성에 비해 외부 interface가 단순하고, 책임과 변경 경계가 분명해야 한다는 것이다.

## Interface는 사람이 설계하고 구현은 선택적으로 위임한다

AI에 구현을 맡기더라도 module의 목적, public interface와 testable boundary는 사람이 주도해서 설계해야 한다. 중요도가 낮은 내부 구현은 interface를 통과하는 test로 검증하면서 AI에 더 많이 위임할 수 있다. 반면 금융처럼 실패 비용이 큰 영역은 내부 구현도 더 강하게 review해야 한다.

사람과 AI의 역할을 다음처럼 나눌 수 있다.

| 역할 | 주요 책임 |
|---|---|
| 사람 | 문제 정의, design concept, domain language, module map, interface와 위험 판단 |
| AI | Codebase 탐색, 반복 구현, test 후보와 tactical code change |
| Deterministic tool | Type check, test, lint와 반복 가능한 검증 |

사람은 전략적 설계를 소유하고 AI는 전술적 구현을 가속한다. 이 구분이 없으면 AI가 빠르게 만든 local change가 system 전체의 설계를 약화시킬 수 있다.

## 주니어 개발자를 위한 적용 순서

1. 구현 요청 전에 AI에게 요구사항의 모호함과 미결정 사항을 질문하게 한다.
2. Project 용어를 자신의 말로 정의하고 code의 이름과 일치시킨다.
3. Feature를 작은 behavior로 나누고 test부터 작성한다.
4. AI가 만든 변경 직후 type check와 test를 실행한다.
5. 관련 책임이 여러 작은 module에 흩어졌는지 살핀다.
6. Public interface는 직접 설명하고 검토한다.
7. 매일 작은 refactoring으로 system design에 투자한다.

주니어에게 필요한 능력은 AI보다 code를 빨리 쓰는 것이 아니다. AI가 만든 변경이 system의 design concept, domain language와 module boundary를 지키는지 판단하고, 작은 feedback loop로 잘못된 방향을 수정하는 능력이다.

## AI를 동료로 쓰며 Code Reading을 훈련한다

Pasha의 인터뷰는 AI를 모든 구현을 대신하는 자율 agent보다 spec을 함께 다듬고 code·review 초안을 제공하는 junior·medium 동료처럼 사용하는 사례를 더한다. 사람은 AI가 작성한 code를 읽고 review하며, review comment도 정답으로 받지 않고 주의해야 할 위치를 알려주는 후보로 사용한다.

새 stack에서 AI가 문법과 SDK 사용을 도와줄 수 있어도 책임 분리, data flow, interface와 failure mode를 보는 fundamentals는 사람에게 있어야 한다. 초보자가 AI로 완성품만 얻고 구현 이유를 따라가지 않으면 전이 가능한 mental model을 만들지 못한다.

## AI가 Code를 써도 남는 사고 기술

Boris Cherny의 관점은 language·framework와 code style에 대한 강한 선호는 상대적으로 덜 중요해질 수 있지만, methodical하고 hypothesis-driven한 debugging과 types-first 사고는 계속 중요하다는 것이다.

이 둘은 구현을 직접 입력하는 기술과 구분된다.

```text
Hypothesis-driven debugging
→ 증상, 원인 가설, 실험과 evidence를 분리

Types-first 사고
→ Code body보다 input·output·invariant와 interface를 먼저 설계
```

AI가 오류를 한 번에 수정할 수 있어도 개발자가 재현과 근거를 이해하지 못하면 agent가 실패했을 때 복구할 수 없다. 반대로 framework 세부 문법을 모두 암기하지 않아도 type signature, domain boundary와 검증 방법을 설명할 수 있다면 AI 구현을 더 안전하게 위임할 수 있다.

또한 사용 중인 layer 아래를 이해한다는 전통적인 조언은 AI 시대에도 유지된다. Language runtime과 framework뿐 아니라 model의 context, tool use, permission과 non-deterministic failure mode가 새로 이해해야 할 아래 layer에 포함된다.

## Time·Scale·Trade-off가 AI 위임의 경계를 바꾼다

AI에게 얼마나 위임할지는 tool의 성능만으로 결정할 수 없다. Google의 software engineering 관점처럼 code의 예상 수명과 사용 규모를 먼저 구분해야 한다.

- 짧게 쓰고 버릴 script는 AI 생성 속도를 우선할 수 있다.
- 장기 운영할 service는 upgrade, backward compatibility와 hidden dependency를 고려해야 한다.
- 사용자가 많고 여러 team이 의존하는 interface는 변경 비용과 migration 책임이 커진다.
- 실패 비용이 큰 영역은 더 빠른 feedback, 강한 test와 human review가 필요하다.

AI는 code 생산 비용을 낮추지만 time에 따른 유지보수와 scale에 따른 coordination 비용을 없애지 않는다. 오히려 변경량이 늘면서 그 비용을 더 빨리 드러낼 수 있다. 따라서 AI의 구현 가능성뿐 아니라 변경을 필요한 기간 동안 안전하게 유지하고 반복할 수 있는지도 함께 판단해야 한다.

## 결론

AI 시대에 fundamentals가 중요해지는 이유는 AI가 약해서가 아니라 매우 빠르기 때문이다. 좋은 구조에서는 AI의 속도가 leverage가 되지만, 나쁜 구조에서는 같은 속도가 entropy를 키운다. Specification만 관리하며 code를 보지 않는 방식보다 사람이 전략적 설계를 소유하고, AI 구현을 단순한 interface와 반복 가능한 feedback loop 안에 두는 방식이 지속 가능한 AI coding에 가깝다.

## See Also

- [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](claude-code-team-ai-native-development-workflow.md)
- [Software Engineering을 Time·Scale·Trade-off로 이해하기](software-engineering-time-scale-tradeoffs.md)
- [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](martin-fowler-ai-software-engineering-study-guide.md)
- [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](current-engineering-sources-and-ai-native-development.md)
- [Brownfield AI Agent Workflow](../ai-agents/brownfield-ai-agent-workflow.md)
- [AI를 활용한 개발자 성장과 Career 판단](ai-assisted-engineering-growth-and-career-judgment.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](../ai-agents/superpowers-agent-management-and-spec-driven-development.md)
- [AI Coding에서 Code Reading과 Intent 보존](ai-coding-code-reading-and-intent-preservation.md)
