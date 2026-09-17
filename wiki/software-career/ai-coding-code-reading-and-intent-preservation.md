# AI Coding에서 Code Reading과 Intent 보존

> Sources: zanlib, 2026-09-14
> Raw: ["Do You Still Read the Code?"](../../raw/software-career/2026-09-14-do-you-still-read-the-code.md)
> Updated: 2026-09-16

## Overview

AI-assisted programming에는 적어도 두 가지 서로 다른 운영 경로가 있다. 하나는 AI를 구현 가속기로 사용하면서 개발자가 code의 작동 원리와 변경 이유를 계속 이해하는 방식이고, 다른 하나는 구현 세부를 AI에 위임하고 specification·context·evaluation을 주된 관리 대상으로 삼는 방식이다. 차이는 AI가 작성한 code의 양이 아니라 사람이 구현을 이해하고 유지할 책임을 어떻게 배치하는지에 있다. 어느 쪽을 택하든 팀은 변경을 병합하기 전에 유지보수 방식과 책임 경계를 명시해야 한다.

## Accelerator와 Vibecoder는 생성량으로 구분되지 않는다

`accelerator`는 자신의 이해를 code로 옮기는 속도를 높이는 데 AI를 사용한다. AI가 feature의 거의 모든 code를 만들더라도 개발자가 구현 이유를 설명하고 변경의 영향을 예상하며 이후 수정까지 책임지려 한다면 이 경로에 속한다. 생성된 code를 읽는 일은 구현과 mental model이 어긋나는 지점을 찾기 위한 핵심 활동이다.

`vibecoder`는 구현과 이후 수정을 AI에 위임한다. 사람의 관심은 원하는 behavior를 명시하고, 필요한 domain context를 공급하며, 결과가 만족스러운지 판정하는 체계를 만드는 쪽으로 이동한다. 모든 구현 세부를 이해하는 것은 이 방식의 목표가 아니다. 대신 durable specification, acceptance criteria와 evaluation이 구현을 재생성하고 검증할 수 있을 만큼 강해야 한다.

따라서 두 경로는 AI 사용량의 많고 적음이나 숙련도의 높고 낮음으로 구분되지 않는다. 핵심은 구현 이해를 사람이 유지할 것인지, 아니면 구현을 교체 가능한 산출물로 취급하고 specification과 evaluation에 유지보수 책임을 둘 것인지다.

## Code는 Domain Theory를 형성하는 작업 공간이다

이 글은 programming을 완성된 specification을 기계적으로 번역하는 일로만 보지 않는다. 구현 과정에서 모든 case와 예외를 다루다 보면 요구사항에 대한 기존 이해가 깨지고 domain model 자체가 바뀔 수 있다. Code 작성과 검토는 이미 정해진 이론을 표현하는 단계인 동시에, 그 이론을 발견하고 수정하는 과정이다.

이 관점에서 중요한 자산은 source code만이 아니다. 개발자가 system이 현실의 어떤 문제를 어떻게 모델링하는지 설명하고, 각 부분이 왜 그렇게 구성됐는지 답하며, 요구 변화에 맞춰 model을 고칠 수 있게 하는 이해가 함께 남아야 한다. 실행 결과와 test만으로는 software가 specification대로 움직이는지는 확인할 수 있어도 specification이 현실의 domain과 맞는지는 보장할 수 없다.

## Code Reading은 이해를 지키지만 Intent Debt까지 없애지는 못한다

생성된 code를 전부 읽는 것은 사람이 모르는 사이 구현 이해를 포기하는 drift를 줄인다. Diff를 훑기만 하다가 나중에는 자신이 무엇을 만들었는지 agent에게 되묻고도 답을 검증할 수 없는 상태로 가는 것을 막는 반복적인 점검 지점이 된다.

하지만 line-by-line review만으로는 결정의 근거까지 복원되지 않는다. Code에는 어떤 값과 구조가 남아 있어도 그것이 business requirement, 기존 convention, 의식적인 trade-off 또는 검토되지 않은 추측에서 나왔는지는 나타나지 않을 수 있다. 이런 연결이 사라지는 `intent debt`는 code를 읽는 것만으로 갚기 어렵다.

Review에서는 최소한 다음을 구분하고 code 변화와 연결해야 한다.

- 명시된 요구사항
- 의식적으로 선택한 구현 결정과 근거
- 기존 codebase에서 물려받은 convention
- 근거가 기록되지 않은 우연한 선택

Agent의 요약 능력은 이 연결을 추적하고 설명이 비어 있는 결정을 드러내는 데 사용할 수 있다. 다만 요약이 새로운 정본이 되어서는 안 되며, session과 agent가 바뀌거나 구현이 재생성돼도 목표·제약·검증 기준과 관련 context가 유지돼야 한다.

## 두 경로에는 서로 다른 Debt와 실패 방식이 있다

Accelerator 방식에서는 생성 속도가 사람의 읽기와 이해 속도를 앞지를 때 `cognitive debt`가 쌓인다. 큰 diff를 계속 검토하는 일은 느리고 피로하며, 사람이 책임을 진다고 선언하는 것만으로 그 책임을 수행할 능력이 유지되지는 않는다. AI가 만든 code를 검토하는 것만으로 직접 구현하고 복구하는 기술까지 충분히 유지되는지도 확정되지 않았다.

Vibecoder 방식에서는 요구사항이 다시 쓰이거나 잊히고 session 사이에서 context가 흔들릴 때 `intent debt`가 쌓인다. 성공은 model 성능뿐 아니라 specification, context 관리와 evaluation harness의 품질에 크게 의존한다. 구현을 읽지 않기로 했다면 그 공백을 메울 외부 검증 자산을 의도적으로 구축해야 한다.

Throwaway prototype처럼 내부 구현을 장기간 유지할 필요가 없는 영역에서는 vibecoding을 선택할 수 있다. 반대로 production code처럼 사람이 장애와 변경을 책임져야 하는 영역에서는 구현 이해를 유지하는 비용을 감수할 이유가 커진다. 한 project 안에서도 module별로 다른 방식을 선택할 수 있지만, 어느 영역에 어떤 계약을 적용하는지는 분명해야 한다.

## 팀은 유지보수 계약을 먼저 합의해야 한다

가장 위험한 상태는 서로 다른 경로를 택한 사람을 같은 팀에 두면서 기대와 경계를 합의하지 않는 것이다. Accelerator는 작성자가 애초에 유지하지 않은 구현 이해를 나중에 복원해야 할 수 있다. Vibecoder는 specification과 evaluation으로 대체하려던 우연한 구현 결정을 설명하라는 요구를 받을 수 있다.

변경을 병합하기 전에 팀은 다음을 확인해야 한다.

- 이 영역은 개발자의 구현 이해를 통해 유지하는가
- Specification과 재생성·rigorous testing을 통해 유지하는가
- 두 방식을 함께 쓴다면 각각 어느 boundary까지 적용되는가
- Production 문제와 다음 변경의 최종 책임은 누가 지는가

Code를 읽지 않는 것 자체가 진보인 것은 아니다. 구현과의 상호작용을 줄이는 경로를 선택했다면, 그 선택이 동료에게 암묵적인 이해 복원 비용을 떠넘기지 않도록 별도의 유지보수 체계를 보여줘야 한다.

## 한계

이 글은 accelerator 경로를 선호하는 저자의 분석이며 두 방식의 장기 유지보수 비용을 비교한 실증 연구가 아니다. Vibecoder 방식이 발전해 code reading을 불필요하게 만들 가능성을 배제하지 않으며, accelerator 방식 역시 review 부담과 기술 유지 비용 때문에 지속하기 어려울 수 있음을 인정한다. 따라서 이 구분은 어느 한쪽의 승리를 선언하는 결론보다 팀의 책임과 도구 설계를 명시하기 위한 판단 틀로 사용하는 편이 적절하다.

## See Also

- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [AI를 활용한 개발자 성장과 Career 판단](ai-assisted-engineering-growth-and-career-judgment.md)
- [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](martin-fowler-ai-software-engineering-study-guide.md)
- [Agent Harness의 구조와 Deterministic Control Loop](../ai-agents/agent-harness-anatomy-and-deterministic-control-loop.md)
