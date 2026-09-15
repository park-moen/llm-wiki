# Software Engineering을 Time·Scale·Trade-off로 이해하기

> Sources: Titus Winters·ACM Tech Talk (Unknown)
> Raw: [Software Engineering at Google transcript](../../raw/software-career/software-engineering-at-google-tech-talk.md)
> Updated: 2026-08-16

## Overview

이 발표는 software engineering을 code를 작성하는 기술보다 넓게 본다. 핵심은 여러 사람이 여러 version에 걸쳐 system을 계속 변경할 수 있게 만드는 것이다. 올바른 practice는 항상 같지 않으며 code의 예상 수명, 조직과 사용자의 규모, 현재 evidence와 비용의 trade-off에 따라 달라진다. 장기 운영할 software라면 change가 가능하도록 test, upgrade, automation, visibility와 유지보수 책임을 설계해야 한다.

원본은 사용자가 제공한 영어 자동 transcript다. 중복 문장과 고유명사 오인식이 포함될 수 있으므로 모호한 표현은 발표 영상이나 관련 서적과 대조해야 한다.

## Programming과 Software Engineering의 차이

Programming은 현재 문제를 code로 해결하는 데 집중할 수 있다. Software engineering은 그 해결책이 필요한 기간 동안 계속 작동하고 변경될 수 있는지까지 다룬다.

짧게 쓰고 버릴 script에는 다음 운영체제 upgrade나 장기 dependency 관리가 중요하지 않을 수 있다. 반대로 끝을 예측하기 어려운 service나 open-source project는 language version, security vulnerability와 dependency 변화에 계속 대응해야 한다. 같은 code quality practice도 예상 수명에 따라 투자 가치가 달라진다.

따라서 작업을 시작할 때 먼저 code의 예상 사용 기간을 판단해야 한다. 수명을 명시하지 않으면 일회성 code에 과도한 process를 적용하거나, 장기 system을 prototype 수준의 구조로 방치할 수 있다.

## Time이 문제를 어렵게 만드는 이유

시간이 지나면 code 자체뿐 아니라 주변 조건도 바뀐다.

- 사용자가 늘고 예상하지 못한 동작에 의존한다.
- Language, library와 operating system이 바뀐다.
- Security issue 때문에 계획에 없던 upgrade가 필요해진다.
- Team 구성원과 업무 지식이 바뀐다.
- Schema와 API의 backward compatibility가 중요해진다.

한동안 upgrade를 미루면 숨은 의존성이 쌓이고, team은 upgrade 경험과 policy를 갖지 못하며, 한 번에 여러 version의 변화를 흡수해야 한다. 이런 어려움이 서로 결합하면 다음 upgrade도 미루고 결국 rewrite에 의존하는 악순환이 생긴다.

지속 가능성(sustainability)은 모든 변화를 즉시 수행한다는 뜻이 아니다. 필요한 변화가 왔을 때 안전하게 대응할 능력을 유지한다는 뜻이다. 변경하지 않기로 선택할 수는 있지만 변경할 수조차 없는 상태는 위험하다.

## Hyrum’s Law와 숨은 의존성

사용자가 충분히 많으면 문서에 약속하지 않은 observable behavior에도 누군가 의존하게 된다. 이것이 발표에서 설명하는 Hyrum’s Law의 핵심이다.

예를 들어 API가 반환 순서를 보장하지 않았더라도 항상 같은 순서가 관찰되면 client가 그 순서에 의존할 수 있다. 이후 내부 구현을 개선하며 순서가 달라지면 계약상 문제없는 변경처럼 보여도 실제 사용자는 깨진다.

이 법칙은 모든 변경을 금지하라는 의미가 아니다. 다음을 설계에 포함하라는 뜻이다.

- 실제 사용 방식과 dependency에 대한 visibility
- 작은 incremental upgrade
- 변경 전후 behavior를 확인하는 test
- Migration path와 compatibility 전략
- 영향을 받는 사용자를 함께 고칠 수 있는 도구와 책임 구조

Compatibility는 변경 하나의 속성이 아니라 제공자와 사용자 관계의 속성이다. 사용자를 보지 못한 채 하위 호환 변경이라고 선언하는 것만으로는 충분하지 않다.

## Scale은 Code보다 Human Process에서 먼저 무너질 수 있다

조직과 codebase가 성장할 때 반복 작업의 비용이 어떻게 증가하는지 봐야 한다. Team과 code가 커질수록 더 많은 heroics, broadcast communication이나 중앙 담당자의 수작업이 필요한 process는 오래 유지하기 어렵다.

발표가 제시하는 기준은 반복 업무가 증가하더라도 human effort와 communication이 그보다 빠르게 증가해서는 안 된다는 것이다. 다음과 같은 process는 확장 위험 신호다.

- 모든 team이 모이는 merge 회의가 필요하다.
- 한 사람만 release나 upgrade를 수행할 수 있다.
- API 변경 때마다 모든 사용자가 각자 migration 방법을 다시 배운다.
- 공지 message만으로 수많은 project의 변경을 강제한다.
- Codebase가 커질수록 build와 test 시간이 통제 없이 늘어난다.

반복 작업은 전문가의 학습, automation과 표준화로 더 싸져야 한다. 같은 문제를 수많은 비전문가가 각자 해결하도록 분산하면 조직 전체 비용이 커질 수 있다.

## Churn Rule: 변경을 만든 Team이 비용을 책임한다

기존 API를 없애고 새 API로 이동할 때 모든 사용자에게 알아서 고치라고 요구하면 변경을 시작한 team의 비용은 작아 보인다. 그러나 조직 전체에서는 많은 사람이 같은 migration 지식을 따로 배우고, downstream project가 연쇄적으로 깨질 수 있다.

발표의 churn rule은 변경 비용의 대부분을 변경을 시작한 team이 부담해야 한다는 원칙이다. 이를 위해 transition plan을 검토하고, old와 new 사이에 실제 migration path가 존재하도록 새 설계를 제한하며, 가능한 사용처를 직접 수정한다.

이 방식은 변경 team에 더 많은 초기 비용을 요구한다. 대신 expertise와 automation을 한곳에 축적해 전체 조직의 반복 비용을 낮춘다.

## 위험한 작업을 통제할 것인가, 싸게 만들 것인가

큰 merge가 위험하다고 해서 merge 권한을 소수에게 집중하고 드물게 수행하면 merge 하나의 크기와 조정 비용이 더 커질 수 있다. 다른 전략은 merge를 작고 자주 수행하고, test와 automation으로 비용을 낮추는 것이다.

모든 위험에 같은 답이 적용되지는 않는다. 하지만 반복되는 핵심 process라면 드문 heroics로 통제하는 방식과 자주 연습해 싸게 만드는 방식을 비교해야 한다. Version control과 integration에서는 작은 변경, 짧은 branch와 자동 검증이 scale에 더 잘 맞을 수 있다.

## Shift Left: Feedback을 더 이른 시점으로 옮긴다

같은 defect도 production에서 발견할 때보다 개발 중에 발견할 때 수정 비용이 작다. Code review, pre-submit test, canary와 production monitoring은 서로 다른 비용과 fidelity를 가진 feedback layer다.

Shift left는 bug가 전혀 발생하지 않게 만든다는 약속이 아니다. 완벽한 defect prevention은 불가능하다. 대신 각 종류의 defect를 더 싸고 원인이 분명한 시점에 발견하도록 여러 proxy를 배치하는 것이다.

```text
개발 중 type·test
→ code review·pre-submit
→ canary
→ production monitoring
```

왼쪽 단계일수록 developer의 context가 아직 머릿속에 있고 변경 범위가 작다. 오른쪽 단계일수록 실제 사용자 환경에 가까워 fidelity는 높지만 실패 비용과 원인 탐색 비용도 커진다.

## Trade-off는 구호가 아니라 Evidence로 계산한다

Software engineering에는 모든 상황에 적용되는 하나의 silver bullet이 없다. 장기 유지보수, 빠른 delivery, 개발자 시간, compute resource와 reliability 사이에서 선택해야 한다.

“Hardware는 싸다”, “다들 이렇게 한다”, “내가 정했으니 따른다”는 말은 trade-off를 설명하지 못한다. 대신 예상 사용자 수, 운영 비용, engineer effort, failure impact와 software lifespan 같은 요소를 비교해야 한다.

한 번 옳았던 결정도 context가 바뀌면 다시 평가해야 한다. Evidence가 달라졌는데 과거 결정을 그대로 유지하는 것은 일관성이 아니라 경직성일 수 있다.

## Dependency와 Visibility

외부 dependency는 다른 team이나 organization이 control한다. 그들의 지원 기간, security 대응과 우선순위가 나의 project와 같다는 보장이 없다. 따라서 dependency management는 technical mechanism만으로 해결하기 어려운 관계와 priority 문제를 포함한다.

조직 내부에서는 가능한 한 사용처, version과 build 상태를 볼 수 있어야 변경 영향을 계산할 수 있다. 하나의 repository가 반드시 유일한 해법은 아니지만, code와 dependency 관계가 보이지 않는 구조에서는 안전한 migration과 compatibility 판단이 훨씬 어려워진다.

## Build System과 Constraint의 가치

작은 조직도 대규모 Google 전용 tooling을 그대로 복제할 필요는 없다. 발표에서는 일관된 tool 적용의 기반으로 좋은 build system을 강조한다.

좋은 build system은 code가 어떻게 구성되고 실행되는지 기계가 이해하게 하고, build와 test를 반복 가능하게 만든다. 명확한 constraint는 개발자의 자유를 무조건 빼앗는 것이 아니라 system을 예측 가능하게 만들어 automation과 scale을 가능하게 한다.

## 주니어 개발자를 위한 적용 질문

1. 이 code는 몇 번 실행하고 버릴 것인가, 계속 운영할 것인가?
2. 다음 language·library upgrade를 지금 구조에서 수행할 수 있는가?
3. 문서에 없는 behavior를 누군가 의존할 가능성이 있는가?
4. 반복 작업이 team 성장에 비례해 더 많은 수작업을 요구하는가?
5. 변경을 만든 사람이 migration 비용을 사용자에게 떠넘기고 있지 않은가?
6. Bug를 production보다 더 왼쪽에서 발견할 방법이 있는가?
7. 현재 결정을 뒷받침하는 evidence와 비용 계산이 있는가?
8. Build와 test가 누구의 환경에서도 같은 방식으로 실행되는가?

주니어가 모든 대규모 조직 문제를 미리 해결할 필요는 없다. 중요한 것은 현재 code의 예상 수명과 scale을 먼저 구분하고, 장기 운영할 가능성이 생기면 important-but-not-urgent한 upgrade, test와 구조 개선을 계속 추적하는 것이다.

## 결론

Software engineering의 목표는 code를 생산하는 것이 아니라 필요한 기간 동안 문제 해결책을 작동하게 유지하는 것이다. 이를 위해 time에 따른 변화, scale에 따른 human cost와 evidence 기반 trade-off를 함께 봐야 한다. 좋은 system은 절대 변하지 않는 system이 아니라, 필요한 변화가 왔을 때 안전하게 반응할 수 있는 system이다.

## See Also

- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](martin-fowler-ai-software-engineering-study-guide.md)
- [Brownfield AI Agent Workflow](../ai-agents/brownfield-ai-agent-workflow.md)
