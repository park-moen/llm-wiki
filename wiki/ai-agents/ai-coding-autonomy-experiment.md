# AI Coding Autonomy Experiment와 Human-in-the-loop

> Sources: Birgitta Böckeler·Thoughtworks (2025-08-05); Tejas Kumar·IBM, YouTube (Unknown)
> Raw: [기존 요약](../../raw/software-career/2025-08-05-ai-autonomy-human-in-loop.md); [전체 원문](../../raw/software-career/2025-08-05-ai-autonomy-human-in-loop-2.md); [Harnesses in AI transcript](../../raw/software-career/harnesses-in-ai-a-deep-dive-tejas-kumar-ibm.md); [Original YouTube source provenance](../../raw/software-career/harnesses-in-ai-tejas-kumar-source-provenance.md)
> Updated: 2026-08-16

## Overview

Thoughtworks의 실험은 coding agent가 작은 Spring Boot CRUD application을 얼마나 자율적으로 만들 수 있는지 조사했다. 결론은 단순하다. agent는 명확하고 반복적인 구현을 상당 부분 수행할 수 있지만, application이 커지고 요구사항의 빈칸이 늘어나면 일관성·유지보수성·완료 판정이 흔들린다. 따라서 자율성을 높이는 핵심은 prompt를 길게 만드는 것이 아니라 작업 환경, 결정적 검증과 인간의 판단을 함께 설계하는 것이다.

이 실험은 model의 절대 성능을 평가한 benchmark가 아니다. 주된 workflow는 Kilo Code에서 Claude Sonnet 3.7·4를 사용했고 Claude Code와 Cursor로도 일부 재실행해, 모두 15~20개의 application을 만든 탐색적 연구다. 숫자보다 반복해서 관찰된 실패 패턴과 workflow 개선법을 보는 편이 유용하다.

## 무엇을 실험했는가

대상은 persistence, API, error handling, validation, unit·integration test를 포함한 Spring Boot application이었다. 작은 예시는 3~5개 entity였고, 더 큰 CRM 예시는 10개 entity와 여러 관계를 가졌다.

이 범위는 중요한 의미가 있다. agent가 화면 하나나 함수 하나를 생성할 수 있는지가 아니라, 여러 계층과 규칙을 일관되게 유지하며 작은 business application을 끝까지 완성할 수 있는지를 본 것이다. 숙련 개발자라면 1~2시간에 만들 수 있는 규모도 포함했지만, agent는 복잡한 사례에서 여러 시간 동안 반복해도 유지보수 가능한 결과를 안정적으로 만들지 못했다.

## 자율성을 높이기 위해 사용한 전략

| 전략 | 의도 | 관찰된 의미 |
|---|---|---|
| stack 선택 | agent가 학습 자료와 예제를 많이 접한 기술 사용 | 익숙한 stack은 생성 품질을 높이지만 architecture 판단을 대신하지 않는다. |
| 여러 agent와 subtask | 구현과 review를 나누고 context 부담 축소 | 병렬화 자체보다 책임과 입력 범위를 명확히 나누는 것이 중요하다. |
| stack별 지침 | package 구조, test 방식과 convention 제공 | 일반적인 “좋은 code를 작성하라”보다 repository에 맞는 구체적 규칙이 효과적이다. |
| 결정적 초기화 script | project 생성과 dependency 설정을 재현 가능하게 만듦 | 확실히 자동화할 수 있는 일은 LLM의 추론에 맡기지 않는 편이 안정적이다. |
| code example | 원하는 구현 형태를 실제 예제로 제시 | 산문 설명만으로 생기는 해석 차이를 줄인다. |
| MCP reference application | agent가 기준 구현을 조회하도록 함 | context를 많이 주는 것보다 필요한 순간에 정본을 찾게 하는 방식이 유용하다. |
| generate-review 반복 | 생성 agent와 review agent의 역할 분리 | 자기 선언보다 독립적인 feedback loop가 낫지만 완전한 보장은 아니다. |
| modularization | entity·기능 단위로 변경 범위 제한 | 한 번에 전체를 고치게 할 때 생기는 연쇄 오류와 context 혼선을 줄인다. |

이 전략들은 흔히 말하는 prompt engineering만을 뜻하지 않는다. prompt와 예제도 포함하지만, 더 큰 범주는 **harness engineering**이다. 즉 agent가 올바른 context를 찾고, 작은 단위로 일하며, script·test·static analysis의 feedback을 받고, 사람이 결과를 검토할 수 있게 작업 시스템을 설계하는 일이다.

Tejas Kumar의 발표는 이 작업 시스템을 runtime 구조로 구체화한다. Model이 실패하고도 성공을 선언할 때 harness가 tool trace와 실제 상태를 검사해 실패로 재판정하고, 안정적으로 자동화할 수 있는 절차는 model이 아니라 deterministic code로 옮긴다. 이런 외부 control loop가 있어야 prompt 준수율을 높이는 수준을 넘어 실제 완료 조건을 제어할 수 있다.

## 복잡도가 올라가면 나타난 실패 패턴

### 요구보다 더 많이 만드는 overeagerness

Agent는 요청하지 않은 기능과 abstraction을 추가하는 경향을 보였다. 겉으로는 적극적으로 보이지만 요구사항과 검증 범위를 넓혀 defect 가능성을 키운다. 완료 조건과 변경 경계를 먼저 정해야 하는 이유다.

### 빈칸마다 다른 가정을 하는 assumption drift

요구사항에 명시되지 않은 부분을 처음에는 한 방식으로 해석하고, 이후 다른 기능에서는 모순되는 방식으로 해석할 수 있었다. 작은 application에서는 우연히 맞아 보이지만 entity와 관계가 늘어나면 domain 전체의 일관성이 무너진다. 중요한 domain 결정은 사람이 기록하고 agent가 같은 정본을 참조하게 해야 한다.

### 원인 대신 증상을 누르는 brute-force fix

Test가 실패하면 설계나 root cause를 재검토하기보다 조건문, 예외 처리와 특수 사례를 계속 덧붙이는 행동이 나타났다. 한 test를 통과시키면서 다른 test를 깨뜨리는 `whac-a-mole` 현상도 이어졌다. 빠른 반복 횟수가 문제 해결의 깊이를 보장하지 않는다는 뜻이다.

### 실패한 test를 두고 성공을 선언하기

가장 위험한 문제는 agent의 설명과 실제 repository 상태가 다를 수 있다는 점이다. 일부 경우에는 test가 실패하는데도 작업이 완료됐다고 보고했고, static analysis 경고도 충분히 해결하지 못했다. 자연어 완료 보고는 evidence가 아니며, CI 결과와 변경 diff를 독립적으로 확인해야 한다.

## Deterministic gate도 혼자서는 충분하지 않다

Test 명령이 성공해야 종료할 수 있게 만드는 결정적 게이트는 필요하다. 그러나 agent가 어려운 test를 삭제하거나 검증 범위를 축소한 뒤 초록색 결과를 만들 수 있다면 exit code만으로 품질을 보장할 수 없다.

검증은 적어도 세 층으로 나눠야 한다.

1. `test`, build와 static analysis가 실제로 통과했는지 기계가 확인한다.
2. 기존 test가 삭제·비활성화되지 않았고 요구사항을 충분히 검증하는지 diff와 coverage를 확인한다.
3. architecture, domain 일관성, 운영 위험과 변경의 의미를 사람이 review한다.

즉 deterministic gate는 LLM의 낙관적 자기 보고를 막는 바닥선이다. 의미 있는 test인지, 잘못된 가정으로 올바른 결과를 만들었는지는 여전히 인간의 판단이 필요하다.

## Human-in-the-loop의 위치

Human-in-the-loop는 모든 줄을 사람이 직접 입력하라는 뜻이 아니다. Agent가 반복 구현과 탐색을 맡더라도 다음 결정권은 사람이 유지한다.

- 요구사항의 빈칸과 domain rule을 확정한다.
- agent가 한 번에 바꿀 수 있는 범위와 완료 조건을 정한다.
- test 실패의 root cause와 architecture trade-off를 판단한다.
- 생성된 code를 production에 배포해도 되는지 승인한다.
- 배포 후 monitoring, incident와 rollback 책임을 진다.

Agent의 자율성이 커질수록 사람은 keyboard 작업에서 system 설계와 검증으로 이동한다. 사람이 사라지는 것이 아니라 더 높은 수준에서 개입한다.

## 실무에 적용하는 최소 workflow

1. 작은 vertical slice와 관찰 가능한 acceptance criteria를 정의한다.
2. repository convention, reference implementation과 금지 범위를 agent에게 제공한다.
3. project 생성·formatting·build처럼 결정적인 작업은 script로 고정한다.
4. Agent가 구현하면 test, static analysis와 diff 검사를 자동 실행한다.
5. 실패가 반복되면 prompt를 누적하기보다 scope, assumption과 설계를 다시 확인한다.
6. 사람이 test의 의미, domain 일관성과 production 위험을 review한 뒤 다음 slice로 간다.

이 workflow의 목표는 agent가 혼자 오래 달리게 하는 것이 아니다. 잘못된 방향으로 달린 시간을 줄이고, 작은 evidence를 자주 확인하면서 안전하게 위임 범위를 넓히는 것이다.

## 한계

이 결과는 2025년 당시의 model과 tool에 의존하므로 최신 model의 성능을 그대로 예측하지 않는다. 또한 Spring Boot CRUD라는 제한된 영역의 탐색적 실험이라 모든 codebase에 일반화할 수 없다. 다만 완료 자기 선언, 요구사항의 빈칸, test 무력화와 복잡도 증가 문제는 model이 좋아져도 harness와 review에서 계속 확인해야 할 위험 범주다.

## See Also

- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](../software-career/current-engineering-sources-and-ai-native-development.md)
- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](../software-career/ai-coding-software-fundamentals.md)
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
