# Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드

> Sources: The Pragmatic Engineer·Martin Fowler (2025-11-19)
> Raw: [인터뷰 요약·공식 chapter](../../raw/software-career/2025-11-19-martin-fowler-ai-interview.md); [사용자 제공 전체 transcript](../../raw/software-career/2025-11-19-martin-fowler-ai-interview-full-transcript.md)
> Updated: 2026-08-16

## Overview

이 인터뷰는 AI coding tool의 사용법보다 software engineering이 왜 필요한지를 설명한다. Martin Fowler의 중심 주장은 AI가 code 작성을 빠르게 만들더라도 불확실한 출력을 이해하고 검증하며 계속 수정할 수 있는 구조가 더 중요해진다는 것이다. 주니어에게 필요한 결론은 AI를 피하라는 것이 아니라, AI를 사용하면서도 learning loop를 잃지 말라는 것이다.

전체 영어 transcript는 위의 사용자 제공 Raw에서 바로 읽을 수 있다. 확장 프로그램이 추출한 자동 자막이므로 고유명사 오인식, 잘못된 문장 분할과 중복은 원문 보존을 위해 그대로 남겨 두었다. 이해가 모호한 대목은 [The Pragmatic Engineer 공식 episode page](https://newsletter.pragmaticengineer.com/p/martin-fowler)나 YouTube 영상의 발음과 대조해야 한다.

## 먼저 알아야 할 배경

### Martin Fowler는 어떤 사람인가

Martin Fowler는 refactoring, enterprise application architecture와 Agile 분야에서 큰 영향을 준 software engineer이자 저자다. 인터뷰는 한 도구의 기능보다 수십 년 동안 software development 방식이 어떻게 바뀌었는지를 비교하는 관점에서 진행된다.

### 이 영상이 어렵게 느껴지는 이유

서로 다른 층의 이야기가 한꺼번에 등장하기 때문이다.

- Programming language와 abstraction
- LLM의 확률적 동작
- Test와 refactoring 같은 개발 practice
- Legacy system과 enterprise organization
- Agile, feedback loop와 delivery
- Junior의 학습과 mentoring

이것을 “AI가 code를 잘 만드는가?”라는 하나의 질문으로 들으면 흐름이 복잡해진다. 아래 개념을 순서대로 분리해서 보면 이해하기 쉽다.

## 1. Deterministic과 non-deterministic

### Deterministic system

같은 초기 상태와 입력에서 같은 규칙을 실행하면 같은 결과를 기대할 수 있는 system이다. 일반적인 application code의 `if`, loop와 function은 여기에 가깝다. Bug가 없다면 test는 같은 조건에서 반복 가능한 결과를 판정한다.

### Non-deterministic AI

LLM은 같은 질문에도 다른 code나 설명을 만들 수 있고, 그럴듯하지만 틀린 결과를 낼 수 있다. Fowler가 AI 변화를 assembly에서 high-level language로의 전환과 비교하면서도 더 중요한 차이로 non-determinism을 드는 이유다.

여기서 “AI는 무작위라 쓸 수 없다”는 결론이 나오지는 않는다. 대신 software engineer가 다음을 설계해야 한다.

- 허용 가능한 오차와 실패 범위
- 결과를 확인하는 test와 evaluation
- 중요한 변경을 승인하는 human review
- 실패 시 피해를 제한하는 rollout과 rollback
- 같은 작업을 반복해도 품질을 유지하는 deterministic guardrail

건축에서 재료가 평균적인 강도를 가진다고 가정하고 한계까지 설계하지 않듯, AI도 가장 그럴듯한 정상 출력만 보고 production에 넣어서는 안 된다는 비유다.

## 2. Abstraction은 단순히 자연어 prompt가 아니다

High-level language의 가치는 CPU instruction을 덜 쓰는 것만이 아니다. Function, class와 domain model처럼 문제를 설명하는 새로운 building block을 만들 수 있다는 데 있다.

좋은 abstraction은 다음 두 일을 함께 한다.

1. 현재 문제를 해결한다.
2. 비슷한 문제를 더 정확하게 설명할 language를 만든다.

LLM에게 자연어로 길게 지시하는 것만으로는 이 language가 생기지 않는다. Domain term, type, interface, test와 작은 DSL처럼 사람과 AI가 같은 의미로 사용할 구조를 만들어야 한다. Prompt를 잘 쓰는 능력보다 codebase 안에 정확한 language와 boundary를 세우는 능력이 장기적으로 더 중요하다는 뜻이다.

## 3. Vibe coding과 learning loop

인터뷰에서 vibe coding은 AI가 만든 code를 거의 읽지 않고 결과만 사용하는 원래 의미로 한정된다.

이 방식이 적합할 수 있는 대상은 다음과 같다.

- 버릴 prototype
- 아이디어를 확인하는 짧은 exploration
- 실패해도 피해가 작은 일회성 tool

반대로 계속 수정하고 운영해야 하는 software에는 위험하다. 생성 결과의 구조를 배우지 않으면 다음 요구사항이 생겼을 때 어디를 바꿔야 하는지 판단할 수 없다. 결국 조금 수정하지 못하고 전체를 다시 생성하는 패턴에 빠진다.

Learning loop는 다음 순환이다.

1. 문제와 해결 방법을 예상한다.
2. Code로 시도한다.
3. 실행 결과와 실패를 관찰한다.
4. 이해를 수정하고 다시 설계한다.

AI가 2번을 빠르게 대신할 수는 있지만 1·3·4번까지 생략하면 개발자의 mental model이 자라지 않는다. 주니어에게는 생성 속도보다 이 loop를 유지하는 것이 특히 중요하다.

## 4. AI 변경은 작은 Pull Request처럼 다룬다

Fowler는 AI의 변경을 빠르게 code를 생산하지만 그대로 믿기 어려운 collaborator의 작업에 가깝게 본다. 따라서 한 번에 큰 feature를 맡기기보다 review 가능한 thin slice로 나눈다.

예를 들어 회원가입 전체를 한 번에 생성하는 대신 다음처럼 나눌 수 있다.

1. Email value object와 validation test
2. 회원 저장 use case와 repository contract
3. 중복 email 실패 test
4. HTTP endpoint와 integration test
5. Logging, metric과 error response

각 slice가 test를 통과하고 사람이 설명할 수 있을 때 다음 단계로 간다. 이것은 AI 때문에 새로 생긴 원칙이 아니라 Agile과 continuous delivery의 feedback loop를 AI workflow에 적용한 것이다.

## 5. Test는 AI의 말을 확인하는 장치다

LLM이 “모든 test가 통과했다”고 말하는 것과 실제 command 결과는 다를 수 있다. Test는 AI에게 보여주기 위한 문서가 아니라 system이 요구한 동작을 하는지 독립적으로 판정하는 executable evidence다.

AI와 작업할 때는 다음을 구분한다.

- AI가 test code를 작성했다.
- Test process가 실제로 실행됐다.
- Test가 올바른 요구사항을 검증한다.
- 기존 동작을 깨뜨리지 않았다.

첫 번째만으로 나머지가 보장되지 않는다. Terminal의 exit code, 실패 test의 내용과 변경 전후 diff를 사람이 확인해야 한다.

## 6. Refactoring은 다시 만드는 일이 아니다

Refactoring은 겉으로 관찰되는 동작을 보존하면서 내부 구조를 개선하는 작은 변경이다. 큰 rewrite나 새 기능 추가를 모두 refactoring이라고 부르는 것은 정확하지 않다.

핵심은 두 가지다.

- 한 단계가 매우 작다.
- 작은 단계들을 안전하게 조합할 수 있다.

AI는 많은 code를 빨리 만들지만 구조 품질은 일정하지 않다. 그래서 refactoring의 가치는 줄어들기보다 커질 수 있다. 다만 class rename처럼 IDE가 정확하게 수행하는 작업까지 LLM에 맡길 필요는 없다.

유망한 조합은 다음과 같다.

- LLM: 의도 파악, 후보 탐색과 변경 계획
- IDE refactoring·codemod: 반복 가능한 구조 변경
- Compiler·static analysis: type와 rule 검사
- Test suite: behavior 보존 확인
- Human: 설계 방향과 trade-off 판단

## 7. Enterprise software가 별도 주제인 이유

Startup prototype과 은행·항공·공공기관 system은 같은 risk tolerance를 가질 수 없다. Enterprise에는 오래된 code뿐 아니라 regulation, 조직별 업무 규칙, 예외와 과거 의사결정이 축적돼 있다.

따라서 기술 선택은 “어떤 도구가 가장 새롭나?”가 아니라 다음 질문으로 판단한다.

- 실패하면 누가 어떤 피해를 입는가?
- 되돌릴 수 있는 변경인가?
- Domain expert가 결과를 검토할 수 있는가?
- 기존 system과 organization의 맥락을 충분히 이해했는가?
- Audit, security와 compliance evidence를 남길 수 있는가?

Fowler가 단순한 정답이나 cookbook을 경계하는 이유도 조직마다 이 맥락이 다르기 때문이다.

## 8. Agile은 회의 방식이 아니라 feedback loop다

Agile을 sprint, stand-up과 Jira 운영 방식으로만 이해하면 인터뷰의 의미를 놓친다. 여기서 중요한 것은 큰 계획을 먼저 확정하는 대신 작은 기능을 만들고 실제 feedback으로 다음 결정을 수정하는 능력이다.

AI가 구현 속도를 높인다면 한 cycle에 더 많은 code를 밀어 넣기보다 cycle 자체를 더 짧게 만드는 데 사용할 수 있다.

- 더 작은 변경
- 더 빠른 review
- 더 이른 user feedback
- 더 잦고 안전한 deployment
- 더 빠른 학습

Specification-driven development도 큰 specification을 한 번에 작성하면 waterfall로 돌아갈 수 있다. 최소 spec, 구현, test와 feedback을 작은 단위로 반복하는 것이 핵심이다.

## 9. 주니어에게 한 조언의 의미

Fowler는 주니어도 AI 도구를 사용하고 실험해야 한다고 본다. 문제는 경험이 적을수록 AI 출력의 품질을 판별할 기준도 부족하다는 점이다.

그래서 좋은 senior mentor를 찾는 일이 중요하다. Mentor는 답을 대신 주는 사람이 아니라 다음 판단 과정을 보여주는 사람이다.

- 왜 이 설계를 선택했는가?
- 어떤 대안을 버렸으며 이유는 무엇인가?
- 이 test가 놓치는 failure mode는 무엇인가?
- 이 변경은 production에서 어떻게 관찰할 것인가?
- AI의 설명 중 무엇을 의심해야 하는가?

AI에게도 같은 태도를 적용한다. 답만 받지 말고 근거, source, context와 반대 조건을 묻는다. 그리고 가능한 부분은 code, documentation과 실행 결과로 직접 확인한다.

인터뷰 앞부분의 Fowler 경력 이야기는 이 조언이 추상론이 아님을 보여준다. 그는 초기 경력의 Jim Odell과 이후 함께 일한 Kent Beck을 자신의 사고를 크게 확장한 mentor로 설명한다. 주니어가 우선해야 할 것은 유명인의 정답을 외우는 일이 아니라, 실제 code와 판단을 함께 검토하며 “왜”를 설명해 주는 사람과 반복적으로 일하는 환경이다.

## 10. 신뢰할 만한 기술 자료를 고르는 기준

Fowler가 좋은 source에서 찾는 특징은 강한 확신이 아니라 불확실성을 정직하게 드러내는 태도다. Software engineering의 답은 조직 규모, 규제, 기존 system, 팀 역량과 실패 비용에 따라 달라지기 때문이다.

자료를 볼 때는 다음 순서로 판단한다.

1. 결론이 적용된 context가 구체적으로 설명돼 있는가?
2. 장점뿐 아니라 비용, failure mode와 반대 조건도 다루는가?
3. “항상”이나 “절대” 대신 선택에 영향을 주는 factor를 제시하는가?
4. 실제 production 경험, code, test나 관찰 결과로 주장을 뒷받침하는가?
5. 아직 모르는 부분과 앞으로 검증할 부분을 구분하는가?

따라서 “모든 개발자는 AI agent에 전부 위임해야 한다”와 “AI는 절대 쓰면 안 된다”는 주장은 방향만 반대일 뿐 같은 문제를 가진다. 믿을 만한 자료는 어느 상황에서 어떤 trade-off를 감수하고 그 방법을 택했는지 보여준다.

## 11. Technology Radar에서 배우는 학습 관리법

Thoughtworks Technology Radar 이야기는 빠르게 변하는 기술을 따라가는 한 방법을 보여준다. 핵심은 위에서 정한 유행 목록이 아니라 여러 project에서 일하는 practitioner의 관찰을 모으고, 토론을 통해 현재 조직의 관점으로 정리하는 bottom-up 지식 공유다.

여기서 중요한 한계도 있다. 한 조직의 Radar는 그 조직의 client, 기술 stack과 risk tolerance를 반영한다. 다른 팀이 그대로 복사할 보편적 정답이 아니다. 개인 학습에도 같은 원리를 적용할 수 있다.

| 상태 | 개인 학습에서의 의미 |
|---|---|
| Adopt | 실제 project에서 반복 사용했고 선택 이유와 위험을 설명할 수 있음 |
| Trial | 작은 실제 과제에 적용하며 효과와 한계를 측정 중 |
| Assess | 읽거나 실험할 가치가 있지만 아직 사용 근거가 부족함 |
| Hold | 과대광고, 높은 위험 또는 더 나은 대안 때문에 당분간 피함 |

분기마다 목록을 갱신하되 도구 이름만 적지 않는다. 사용 context, 직접 확인한 evidence, 실패 조건과 다음 검토 시점을 함께 기록해야 학습이 구독 목록 수집으로 끝나지 않는다.

## 12. Design pattern은 암기 문제가 아니라 공통 언어다

Fowler에게 pattern의 가치는 system에 가능한 한 많이 집어넣는 데 있지 않다. 복잡한 설계 대안에 이름을 붙여 팀이 더 짧고 정확하게 대화하도록 만드는 데 있다. Pattern은 특정 context에서만 유용하므로 이름을 아는 것보다 언제 적용하지 말아야 하는지 설명하는 능력이 중요하다.

최근 pattern 이야기가 덜 보이는 이유를 “architecture가 사라졌다”로 해석해서는 안 된다. Cloud managed service가 일부 설계 결정을 미리 담아 제공하기도 하고, 각 회사가 내부 service와 역사에 맞춘 고유 jargon을 만들기도 한다. 문제와 trade-off는 여전히 존재하며 표현 방식과 추상화 위치가 바뀐 것이다.

주니어는 interview용 pattern 암기보다 다음을 연습하는 편이 낫다.

- 이 pattern이 해결하려는 반복 문제를 자신의 말로 설명한다.
- 적용 전후의 coupling, 변경 비용과 failure mode를 비교한다.
- 팀의 domain term과 system 이름이 code·문서에서 같은 뜻인지 확인한다.
- 외부의 pattern 이름과 회사 내부 jargon을 연결해 신규 구성원도 이해할 수 있게 한다.

## 13. AI 시대의 경력 전망을 읽는 법

Fowler는 AI가 software development를 크게 바꾸리라고 보면서도 개발 자체를 없앨 것이라고 단정하지 않는다. 인터뷰 당시의 채용 위축을 AI 하나로 설명하지 않고 투자 환경과 거시경제 변화가 동시에 작용한다고 구분한다. 이 대목은 시점 의존적인 전망이므로 예언처럼 인용하기보다 원인을 분리해서 보는 사고법을 배워야 한다.

그가 더 오래 유지될 능력으로 보는 것은 code 생성량이 아니다. 무엇을 만들어야 하는지 알아내는 능력, user와 domain expert의 언어를 이해하는 능력, 다른 사람과 협업하는 능력이다. AI 도구 사용법을 배우는 일과 CS·설계·test·communication을 기르는 일은 경쟁 관계가 아니다. 전자는 계속 바뀌는 leverage이고, 후자는 그 leverage의 방향과 안전성을 판단하는 기반이다.

## 핵심 용어 사전

| 용어 | 쉬운 설명 | 이 영상에서의 의미 |
|---|---|---|
| Abstraction | 복잡한 세부사항을 의미 있는 단위로 감추는 것 | Function·domain model처럼 문제를 설명하는 language 만들기 |
| Deterministic | 같은 조건에서 예측 가능한 결과를 내는 성질 | Compiler, IDE refactoring과 test처럼 반복 가능한 도구 |
| Non-deterministic | 같은 요청에서도 결과가 달라질 수 있는 성질 | LLM 출력에 검증과 tolerance가 필요한 이유 |
| Tolerance | 예상 편차를 견딜 수 있도록 둔 여유 | AI가 틀려도 큰 사고로 이어지지 않게 설계한 범위 |
| Learning loop | 시도와 결과를 통해 이해를 계속 고치는 순환 | Code를 읽지 않는 vibe coding이 약화시키는 과정 |
| Disposable code | 짧게 쓰고 버리기로 결정한 code | Vibe coding을 비교적 안전하게 적용할 수 있는 영역 |
| Legacy system | 오래 운영돼 업무 규칙과 제약이 축적된 system | LLM이 이해를 도울 수 있지만 변경에는 더 큰 주의가 필요한 대상 |
| Refactoring | 동작을 보존하며 내부 구조를 개선하는 작은 변경 | AI 생성 code의 유지보수성을 회복하는 discipline |
| Thin slice | 독립적으로 검증 가능한 작은 기능 단위 | AI 변경을 review하고 feedback을 빨리 받는 방법 |
| DSL | 특정 domain을 표현하기 위한 작은 language | 사람·domain expert·AI가 더 엄밀하게 대화하는 수단 |
| Ubiquitous language | 업무와 code에서 같은 뜻으로 쓰는 공통 용어 | Specification과 code가 서로 멀어지지 않게 하는 기준 |
| Human in the loop | 중요한 판단과 승인을 사람이 담당하는 구조 | AI 출력을 production에 연결할 때 필요한 책임 경계 |
| Technology Radar | 기술을 현재 판단 상태별로 분류한 지식 공유 도구 | 유행을 복사하지 않고 context와 현장 evidence를 축적하는 방법 |
| Design pattern | 반복되는 설계 문제와 해법에 붙인 공통 이름 | 정답 목록이 아니라 대안과 적용 context를 토론하는 vocabulary |
| Trade-off | 한 장점을 얻기 위해 다른 비용이나 위험을 받아들이는 관계 | 단일 cookbook 대신 상황별 판단이 필요한 이유 |

## 추천 시청 순서

처음부터 연속해서 보기보다 다음처럼 세 번 나눠 보는 편이 좋다.

### 1회차: AI 핵심 주장

- 16:45 High-level language와 AI 비교
- 25:08 Non-determinism
- 33:38 Vibe coding
- 43:25 Testing with LLMs

목표는 “AI가 얼마나 똑똑한가?”가 아니라 “왜 검증 방식이 달라져야 하는가?”에 답하는 것이다.

### 2회차: 실무 engineering

- 50:45 Enterprise software
- 56:38 Refactoring의 역사
- 1:02:15 AI 시대의 refactoring
- 1:06:10 Deterministic tool과의 결합
- 1:18:26 Agile과 feedback loop

자신의 project에서 test, IDE refactoring, 작은 PR과 deployment가 각각 어떤 역할을 하는지 연결한다.

### 3회차: 성장 방향

- 1:28:35 Fowler의 학습법
- 1:34:58 Junior engineer 조언
- 1:37:44 기술 산업과 개발자의 미래

좋은 source와 mentor를 어떻게 찾을지, AI가 대신할 수 없는 판단 능력이 무엇인지 기록한다.

## 직접 해볼 학습 실습

### 실습 1: Non-determinism 관찰

같은 작은 기능을 AI에게 별도 session에서 여러 번 요청한다. File 구조, error handling과 test가 어떻게 달라지는지 비교한다. 결과 차이를 보고 prompt만으로 일관성을 보장할 수 있는지 판단한다.

### 실습 2: Learning loop 유지

AI가 만든 function마다 다음을 직접 작성한다.

- 입력과 출력
- 정상 경로
- 실패 조건
- 외부 dependency
- 이해되지 않는 한 줄

설명하지 못하는 부분이 남아 있으면 완료로 보지 않는다.

### 실습 3: LLM과 deterministic tool 비교

Class rename을 AI agent와 IDE Rename refactoring으로 각각 수행한다. 변경 파일, 실행 시간, 놓친 reference와 test 결과를 비교한다. 어떤 종류의 작업을 어느 도구에 맡겨야 하는지 기록한다.

### 실습 4: 작은 slice 만들기

하나의 feature를 한 번에 생성하지 말고 독립적으로 test 가능한 단계로 나눈다. 각 단계마다 diff를 읽고 test를 실행한 뒤 다음 단계로 넘어간다.

### 실습 5: 기술 콘텐츠 source audit

최근 본 AI 개발 영상 하나를 고른다. 발표자의 조직과 project context, 직접 제시한 evidence, 언급하지 않은 비용, 적용이 깨질 조건을 한 장에 정리한다. 결론만 강하고 context가 없다면 실행 지침이 아니라 탐색 후보로 낮춘다.

### 실습 6: 개인 Technology Radar 만들기

현재 관심 기술을 Adopt·Trial·Assess·Hold로 나눈다. 각 항목에 “왜 이 상태인가?”, “직접 확인한 것은 무엇인가?”, “다음 작은 실험은 무엇인가?”를 한 줄씩 쓴다. 다음 갱신 때 실제 evidence가 없는 항목은 승격하지 않는다.

## 스스로 답해볼 질문

1. 내가 작성하는 application code 중 deterministic하다고 말할 수 없는 부분은 무엇인가?
2. AI가 틀렸을 때 피해를 막는 마지막 장치는 무엇인가?
3. 현재 작업은 disposable prototype인가, durable software인가?
4. AI가 만든 code에서 내가 새로 이해한 것은 무엇인가?
5. Test가 통과한다는 사실과 요구사항이 맞다는 사실은 어떻게 다른가?
6. LLM보다 compiler, IDE나 script가 더 잘하는 작업은 무엇인가?
7. 내 팀의 domain language가 code에 드러나는가?
8. 큰 변경을 더 작은 feedback loop로 나누려면 어떻게 해야 하는가?
9. 내 판단을 검토해 줄 senior mentor나 신뢰할 source가 있는가?
10. 내가 따르는 기술 조언은 어떤 조직과 risk tolerance에서 나온 것인가?
11. 최근 배운 도구를 유행이 아니라 직접 확인한 evidence로 분류할 수 있는가?
12. 내가 외운 pattern 이름을 적용하지 말아야 할 상황도 설명할 수 있는가?

## 결론

이 인터뷰는 “AI를 쓰지 말라”는 주장이 아니다. AI를 사용하되 이해, test, refactoring, feedback과 communication을 포기하지 말라는 주장이다.

주니어에게 가장 위험한 것은 AI가 code를 작성한다는 사실이 아니라, 그 과정에서 자신이 배우고 있다고 착각하면서 실제 learning loop를 생략하는 것이다. 반대로 AI 결과를 작은 변경으로 받고 직접 읽고 검증하며 mentor의 feedback을 얻는다면 AI는 학습 속도를 높이는 도구가 될 수 있다.

## See Also

- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](current-engineering-sources-and-ai-native-development.md)
- [AI 시대 신입 개발자의 생존과 성장 판단](ai-era-junior-developer-survival-assessment-2026-08-13.md)
- [Brownfield AI Agent Workflow](../ai-agents/brownfield-ai-agent-workflow.md)
- [AI Agent 산문 게이트와 결정적 게이트](../ai-agents/ai-agent-prose-vs-deterministic-gates.md)
