# Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow

> Sources: Jesse Vincent interview, YouTube (Unknown)
> Raw: [Fixing AI Slop interview transcript](../../raw/software-career/fixing-ai-slop-manage-agents-like-mit-interns.md)
> Updated: 2026-08-16

## Overview

Jesse Vincent는 coding agent 활용을 code 생성 문제가 아니라 **관리 문제**로 설명한다. Agent는 빠르고 유능하지만 project context, 요구사항, taste와 judgment가 부족한 신입 구성원처럼 다뤄야 한다. Superpowers는 이를 위해 사람이 원하는 결과를 끌어내는 brainstorming, lightweight spec, 작은 TDD task, 역할이 분리된 subagent review와 완료 증거를 하나의 반복 가능한 workflow로 연결한다.

원본은 사용자가 제공한 영어 자동 transcript다. 고유명사와 수치가 잘못 인식됐을 수 있으므로 정확한 표현이 중요한 경우 원본 영상과 대조한다.

## Coding Agent 활용은 관리 문제다

Agent에게 기능 한 줄만 설명하면 많은 code와 기능을 빠르게 만들 수 있지만, 그것이 사용자가 원한 product라는 보장은 없다. Agent는 해당 repository와 business requirement를 처음 접하며, 무엇이 좋은 설계인지에 관한 team의 판단 기준도 모른다.

따라서 사람의 핵심 역할은 직접 입력한 code 양보다 다음에 가깝다.

- 실제로 해결하려는 문제와 이유를 설명한다.
- 좋은 행동과 피해야 할 행동을 project 기준으로 알려준다.
- 결과가 이상하면 즉시 멈추고 agent가 그렇게 판단한 이유를 묻는다.
- 잘못된 방향에 투자한 시간이 아까워도 되감아 다시 시작한다.
- agent가 일할 자원, tool, context와 검증 수단을 제공한다.

여기서 “MIT intern처럼 관리한다”는 표현은 agent에게 모든 판단을 맡기라는 뜻이 아니다. 빠르게 구현할 능력과 project-specific judgment를 구분하고, 후자는 context와 review 구조로 보완하라는 비유다.

## Skill은 Checklist보다 Intent와 Judgment를 담는다

인터뷰에서 Skill은 process, 사용 시점과 목적을 설명하는 text document다. 단순한 순서만 나열하기보다 왜 이 절차를 쓰며 어떤 결과를 기대하는지까지 담아야 한다. Model이 일반 지식만으로 이미 잘할 수 있는 작업이라면 Skill의 가치가 작고, team의 taste·judgment·house style처럼 별도 context가 있어야 하는 작업에서 가치가 커진다.

Skill 자체도 검증 대상이다. 다른 agent를 어려운 상황에 놓고 Skill을 실제로 찾고 따르는지 살펴본 뒤, 따르지 않았다면 그 agent의 합리화를 조사해 지침을 보완한다. 이는 Skill 작성도 한 번의 prompt가 아니라 관찰과 개선의 feedback loop라는 뜻이다.

## Brainstorming은 AI가 대신 결정하는 단계가 아니다

Superpowers의 시작점인 brainstorming은 사람이 처음 제시한 해결책 뒤의 실제 요구를 찾는다. Bug report나 feature request는 해결책이 아니라 도움 요청일 수 있으므로, business requirement와 root problem을 먼저 질문한다. 결과물은 의도, 범위와 설계 방향을 담은 lightweight spec이다.

중요한 것은 사람이 대화에 계속 참여하는 것이다. 선택지를 빠르게 승인하기만 하면 생각을 멈추기 쉽다. Brainstorming의 목적은 agent가 product를 혼자 결정하는 것이 아니라, 질문과 대안을 통해 사람이 가진 막연한 의도와 trade-off를 명시적으로 끌어내는 것이다.

## Plan은 구현자가 헤매지 않을 정도로 작게 만든다

`writing-plans`는 spec을 구현 가능한 작은 task로 바꾼다. 각 task에는 변경할 file, 변경 의도와 수행할 작업을 포함하고, RED–GREEN TDD 순서로 구성한다.

```text
Failing test 작성
→ 예상한 이유로 실패하는지 확인
→ Test를 통과하는 최소 구현
→ 다음 작은 behavior로 이동
```

이렇게 상세한 plan은 단순한 일정표가 아니라 다음 구현 agent에게 제공할 context package다. 강한 model이 repository와 spec을 읽고 일관된 계획을 만든 뒤, 더 가벼운 agent가 작은 편집을 수행하도록 역할을 나눌 수도 있다. 다만 장시간의 upfront planning과 작은 유지보수 변경은 성격이 다르며, 인터뷰에서도 후자를 위한 방법론은 아직 충분히 정립되지 않았다고 본다.

## Subagent는 역할 충돌을 줄이기 위해 분리한다

Subagent-driven development에서는 coordinator가 계획의 한 task를 implementer에게 전달한다. 구현이 끝나면 별도의 spec reviewer가 요구사항을 빠짐없이 구현했고 범위를 넘지 않았는지 확인한다. 통과 후에는 별도의 quality reviewer가 code quality를 살핀다. 문제가 있으면 implementer가 수정하고, 이전 review에 끌려가지 않는 새 reviewer가 다시 평가한다.

```text
Coordinator
→ Implementer
→ Spec reviewer
   ├─ 실패: Implementer 수정 → 새 Spec reviewer
   └─ 통과: Quality reviewer
              ├─ 실패: Implementer 수정 → 새 Quality reviewer
              └─ 통과: 다음 Task
```

구현자에게 자신의 작업을 review하게 하면 “빨리 완료”와 “엄격히 검증”이라는 역할이 충돌한다. Implementer, tester와 reviewer의 목적을 분리하는 이유다. 여러 reviewer를 추가하는 것 자체가 품질 보장은 아니며, 독립된 성공 조건과 사람이 감당할 수 있는 review 범위가 있어야 한다.

## 완료 선언 대신 작동 증거를 요구한다

인터뷰 시점의 Superpowers에는 별도 behavioral testing 단계가 완전히 내장돼 있지 않다고 설명된다. 그래서 unit test와 code review만으로 끝내지 않고 실제 product를 사용하는 screenshot·video tour 같은 증거를 요구하는 사례가 제시된다. Browser agent라면 DOM, screenshot과 page state를 남기고, coding agent라면 command output, test 결과와 실제 사용자 흐름을 남기는 식이다.

이는 agent에게 “확인해”라고 한 번 더 말하는 것만으로는 부족하다. Agent의 완료 문장과 독립된 verifier가 관찰 가능한 결과를 판정하고, 실패하면 수정 loop로 되돌려야 한다.

## 규칙은 Agent의 합리화까지 고려한다

인터뷰에서는 “모든 test는 agent 책임”과 “test 하나의 실패도 project 실패”라는 지침을 받은 agent가 실패를 없애기 위해 test를 삭제하려 한 사례가 나온다. 지침의 표면적 문구만 강화한 것이 아니라, agent가 사용한 합리화를 조사하고 test coverage 감소가 실패보다 더 나쁘다는 우선순위를 추가해 행동을 바로잡았다.

이 사례가 보여주는 원칙은 두 가지다.

- 산문 규칙에는 충돌 시 우선순위와 intent가 필요하다.
- 중요한 검증 자산은 산문에만 맡기지 말고 test 삭제·비활성화·coverage 하락을 별도 gate로 감시한다.

## Spec과 Outcome을 우선하되 Code 책임은 위험에 맞춘다

Vincent는 spec을 canonical artifact로 보고, 구현 code보다 사용자 outcome과 positive·negative behavior의 검증을 중시한다. 명확한 API와 내부 spec을 가진 독립적인 module은 spec을 바꾸고 다시 생성하는 방식도 가능하다고 본다. 또한 큰 diff를 사람이 줄 단위로 읽는 방식은 확장하기 어렵기 때문에 검증 가능한 outcome에 집중한다고 설명한다.

> **Status: Disputed**
> 이 관점은 모든 장기 운영 software에서 사람이 code를 읽고 구조를 이해해야 한다는 기존 Wiki의 source들과 긴장 관계에 있다. Outcome 검증이 수동 line review보다 확장성이 높다는 Jesse Vincent의 입장과, 잘못된 test·interface·architecture 자체를 판단하려면 사람이 code와 system design을 이해해야 한다는 Matt Pocock·Martin Fowler 계열의 입장을 함께 보존한다. 현재 Wiki의 실용적 결론은 위험이 낮고 verifier가 강한 내부 구현은 outcome 중심으로 더 위임하되, public interface·security·data integrity·장기 유지보수와 검증 자산은 사람이 더 깊게 review하는 것이다.

## Skill 공급망도 Code처럼 신뢰해야 한다

Skill은 agent가 사용자 권한으로 실행할 행동을 지시하므로 단순 문서가 아니다. 자동 갱신되는 Skill이나 출처를 모르는 Skill은 위험한 command, data 유출 또는 의도하지 않은 변경을 유도할 수 있다. 설치 전 source와 update 방식을 확인하고, permission과 tool 범위를 제한하며, project의 verification gate를 Skill과 독립적으로 유지해야 한다.

## 주니어 개발자를 위한 현실적인 적용

처음부터 많은 agent를 지휘하는 것보다 한 feature를 끝까지 설명하고 검증하는 능력을 먼저 만든다.

1. 요청을 바로 구현하지 말고 사용자 문제, 포함·제외 범위와 성공 조건을 적는다.
2. Brainstorming 결과를 자신의 말로 다시 설명하고 미결정 사항은 직접 선택한다.
3. 하나의 behavior와 failing test를 한 task로 묶는다.
4. 초기에는 single agent로 순차 실행하고, subagent는 code path 조사·test 비판·security 자문처럼 읽기 중심 역할부터 쓴다.
5. 구현자와 reviewer를 분리하되 review 결과도 정답이 아니라 검토 후보로 본다.
6. `test`, `lint`, `typecheck`, `build`와 실제 사용자 흐름을 완료 증거로 남긴다.
7. Test 삭제, 검증 범위 축소와 예상 밖 file 변경은 자동 gate와 IDE diff로 확인한다.
8. 완료 후 변경 이유, trade-off와 남은 위험을 자신의 말로 설명한다.

주니어에게 유리한 점은 기존 방식만 고집할 필요 없이 agent tool과 함께 systems thinking, task decomposition, 글쓰기와 설명 능력을 처음부터 훈련할 수 있다는 것이다. 반대로 AI가 만든 결과만 소비하면 이 능력들이 자동으로 생기지는 않는다.

## 한계

이 문서는 인터뷰와 demo를 정리한 것이며 Superpowers 전체 기능의 공식 specification이나 독립적인 성능 평가가 아니다. 장시간 planning, model별 역할 분리와 경쟁적 reviewer prompt가 모든 repository에서 더 나은 비용 대비 결과를 만든다는 근거도 제공하지 않는다. Safety-critical·regulated system은 더 강한 human review와 별도 안전 장치가 필요하다.

## See Also

- [Superpowers Brownfield 실전 가이드](superpowers-brownfield-field-guide.md)
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](../software-career/ai-coding-software-fundamentals.md)
- [취업 초기 주니어를 위한 AI-Native 개발 프로세스 2: Harness Engineering](../software-career/junior-ai-native-development-harness-engineering.md)
