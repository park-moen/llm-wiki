# Plan mode와 지속적인 이해 형성

> Sources: Ayman Nadeem, 2026-09-24; Boris Cherny interview, YouTube (Unknown)
> Raw: [Plan mode is dead](../../raw/ai-agents/2026-09-24-plan-mode-is-dead.md); [Building Claude Code with Boris Cherny transcript](../../raw/software-career/building-claude-code-boris-cherny.md)
> Updated: 2026-09-28

## Overview

Ayman Nadeem의 「Plan mode is dead」는 계획을 세우는 일을 없애자는 글이 아니다. 저자는 자신이 만든 Nuanced의 사용 경험을 바탕으로, 긴 AI 생성 계획서를 먼저 확정하고 구현으로 넘어가는 방식이 사람의 이해를 돕지 못했다고 설명한다. 작업 중 알게 된 사실에 따라 이해·시도·확인·조정을 반복하고, 중요한 결정을 사람이 계속 파악하는 흐름을 제안한다. 이 주장은 특정 제품과 저자의 경험에 근거하며 모든 Plan mode의 효과를 검증한 비교 실험은 아니다.

## 계획하는 과정과 계획 문서는 다르다

저자가 구분하는 것은 계획을 세우는 사고 과정과 그 결과로 남기는 긴 문서다. Nuanced에서는 대화로 모호함을 풀고 spec을 만든 뒤 검토·승인하고 구현했다. 그러나 사용자는 긴 spec을 읽기 어려워했고, 계획을 마쳐야 구현할 수 있는 흐름은 구현 중 새 질문이 나왔을 때 이전 단계로 돌아가기 어렵게 만들었다고 한다.

저자는 모델이 저장소를 탐색하고 합리적인 가정을 세우는 능력이 좋아지면서, agent를 위해 구현 지시를 극도로 자세히 적을 필요가 줄었다고 본다. 반면 사람이 시스템을 이해해야 할 필요는 더 커졌다고 말한다. Prompt에서 agent의 결정, 코드와 제품 동작으로 이어지는 관계를 파악하지 못하면 생성 속도가 빨라도 잘못된 가정과 유지보수 부담이 숨어 들어갈 수 있기 때문이다.

## 작업 중 이어지는 계획

저자가 제안하는 흐름은 현재 이해한 만큼 시도하고, 결과를 확인해 새로 드러난 질문에 답한 뒤 방향을 조정하는 것이다. 계획은 이 반복 안에서 계속 이루어진다. 큰 spec 하나를 읽는 일보다 필요한 순간에 중요한 결정과 그 근거를 드러내는 대화를 중시한다.

이는 **계획의 길이만 줄이면 된다**는 주장보다 넓다. 저자가 지적한 문제는 계획과 구현을 분리하는 도구의 흐름, AI가 만든 장문을 읽는 부담, 여러 agent가 만든 변경을 사람이 계속 이해하기 어려운 점이다. 저자도 시스템이 변할 때 사람의 mental model을 어떻게 최신 상태로 유지할지는 해결되지 않은 문제라고 인정한다.

## Plan mode를 먼저 쓰는 작업 방식과의 차이

Boris Cherny의 인터뷰에는 익숙한 코드베이스에서 Plan mode로 agent와 접근법을 맞춘 뒤 구현을 맡기고, 여러 작업을 병렬로 진행하는 사례가 있다. Nadeem은 계획서를 먼저 확정하고 구현으로 넘기는 흐름이 사고와 작업을 끊을 수 있다고 본다. 두 사람은 계획의 필요성보다 **계획을 언제 고정하고 사람이 변경을 어떻게 이해할지**에서 다른 경험을 말한다.

> **Status: Disputed**
> Boris Cherny는 자신의 익숙한 코드베이스에서 구현 전 Plan mode로 계획을 맞추는 방식을 효과적으로 사용한다고 설명한다. Ayman Nadeem은 Nuanced에서 계획을 별도 문서와 단계로 고정한 방식이 사용자 이해를 돕지 못했다고 보고한다. 작업 환경과 Plan mode의 구현 방식이 다르므로 어느 흐름이 일반적으로 더 낫다고 이 두 경험만으로 결론 내릴 수 없다.

## 적용할 때 확인할 질문

다음은 두 사례를 함께 읽고 도출한 적용 기준이다.

- 계획을 읽은 뒤 사람이 변경할 행동과 경계를 설명할 수 있는가?
- 구현 중 가정이 바뀌면 계획을 쉽게 고치고 다음 작은 작업에 반영할 수 있는가?
- Agent의 결정이 코드와 실제 동작에 어떻게 이어졌는지 확인할 수 있는가?
- 계획을 검토하는 비용이 작업의 위험과 규모에 비해 적절한가?

이 질문에 답하는 데 도움이 된다면 Plan mode를 사용할 수 있다. 긴 문서를 승인하는 절차가 이해를 늘리지 못한다면 대화와 작은 작업 단위의 반복으로 계획을 이어 가는 편이 이 글의 제안에 가깝다.

## See Also

- [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](../software-career/claude-code-team-ai-native-development-workflow.md)
- [LLM과 함께 프로그래밍의 즐거움과 주도권 지키기](../software-career/llm-assisted-programming-enjoyment-and-ownership.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
