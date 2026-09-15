# 해외 현업 중심 AI 네이티브 엔지니어링 학습 로드맵

> Sources: [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](current-engineering-sources-and-ai-native-development.md); [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](martin-fowler-ai-software-engineering-study-guide.md)
> Archived: 2026-08-16

## Overview

AI와 LLM을 실제 software engineering에 적용하는 흐름을 해외 현업 사례부터 학습하기 위한 순서다. 먼저 Martin Fowler로 판단 기준을 세우고, Thoughtworks의 실패 실험과 Stripe·Uber의 production workflow를 비교한다. AI의 구현 위임, 검증과 운영 책임을 충분히 이해하고 직접 재현한 뒤에 카카오·우아한형제들·NAVER 사례로 확장한다. 한국 자료를 뒤로 배치한 것은 가치가 낮아서가 아니라, 현재 목표가 해외 선도 조직의 사고방식과 workflow를 먼저 흡수하는 것이기 때문이다.

## 전체 학습 순서

```text
1. Martin Fowler — 판단 기준
2. Thoughtworks — Agent 자율성의 실패와 한계
3. Stripe — 구현 위임과 Human review
4. Uber — AI review와 Evaluation pipeline
   ↓ 해외 사례 이해·재현 점검
5. 카카오 — AI-native SDLC
6. 우아한형제들 — 실패 후 Human·Tool·AI 역할 재설계
7. NAVER·WOOWACON — 관심 분야의 Production 사례로 확장
```

## 1. Martin Fowler: 먼저 판단 기준을 세운다

자료: [How AI will change software engineering – with Martin Fowler](https://www.youtube.com/watch?v=CQmI4XKTa0U)

첫 단계에서는 특정 AI coding tool의 사용법보다 AI 시대에도 software engineering이 필요한 이유를 이해한다.

- LLM의 non-determinism과 전통적인 code의 deterministic 성질을 구분한다.
- Vibe coding이 허용될 수 있는 disposable prototype과 지속 운영할 software를 구분한다.
- AI가 code를 생성해도 test, refactoring, feedback loop와 human review가 필요한 이유를 이해한다.
- 주니어가 AI를 사용하면서 learning loop를 유지하는 방법을 정리한다.

이 기준이 있어야 뒤에 나오는 높은 자동화 수준을 전면 위임이나 개발자 대체로 오해하지 않는다.

## 2. Thoughtworks: Agent의 실패를 먼저 본다

자료: [How far can we push AI autonomy in code generation?](https://martinfowler.com/articles/pushing-ai-autonomy.html)

Agent에게 application 구현을 상당 부분 맡긴 실험에서 자율성이 높아질수록 어떤 문제가 생기는지 살펴본다.

- 요구사항의 빈칸을 agent가 임의로 채우는 문제
- 요청하지 않은 기능과 불필요한 code 생성
- Test 실패 상태에서도 성공을 선언하는 문제
- Reusable prompt, reference application과 static analysis의 역할
- Generate-review loop와 human supervision의 필요성

목표는 AI agent가 무능하다는 결론이 아니다. 어떤 실패를 예상하고 guardrail을 어디에 두어야 하는지 배우는 것이다.

## 3. Stripe: 구현을 위임하되 변경 책임은 유지한다

자료: [Stripe Dev Blog](https://stripe.dev/blog)

Stripe의 Minions 사례에서는 agent가 처음부터 끝까지 code를 작성하는 높은 수준의 구현 위임을 살펴본다. 동시에 사람이 pull request를 review하고 merge를 결정하는 책임 구조에 주목한다.

```text
사람이 문제와 성공 조건 정의
→ Agent가 repository context를 사용해 구현
→ CI와 자동 검증
→ 사람이 diff와 위험을 review
→ 승인 후 merge
```

관찰할 핵심은 agent의 수나 생성 속도가 아니다. Agent가 독립적으로 일할 수 있도록 repository와 tooling을 어떻게 준비하고, 결과를 어떤 단위로 사람에게 돌려주는가다.

## 4. Uber: AI를 하나의 Model이 아니라 System으로 본다

자료: [uReview: Scalable, Trustworthy GenAI for Code Review at Uber](https://www.uber.com/us/en/blog/ureview/)

Uber의 uReview에서는 AI code review를 production 규모로 운영하기 위한 pipeline을 본다.

```text
Diff 입력
→ 검토 대상 선별
→ 분야별 comment 생성
→ Confidence 평가
→ 중복·저가치 결과 제거
→ 개발자에게 제시
→ Feedback과 평가 dataset에 반영
```

여기서 얻어야 할 관점은 좋은 prompt 하나보다 generation, filtering, validation, deduplication과 feedback을 연결한 전체 system이 중요하다는 것이다. AI는 human reviewer를 없애기보다 검토를 보조하는 또 하나의 계층으로 배치된다.

## 해외 단계 완료 기준

다음 질문에 답하고 작은 project에서 재현할 수 있을 때 한국 사례로 넘어간다.

- LLM 출력과 deterministic tool의 차이를 설명할 수 있는가?
- AI가 구현한 code를 읽고 변경 영향과 failure mode를 설명할 수 있는가?
- Agent의 완료 보고가 아니라 실제 command와 exit code로 검증하는가?
- 하나의 feature를 review 가능한 thin slice로 나눌 수 있는가?
- Human review, test, static analysis, CI와 monitoring의 역할을 구분할 수 있는가?
- AI 결과의 false positive를 측정할 작은 evaluation set을 만들 수 있는가?
- AI가 실패했을 때 직접 debug하고 복구할 수 있는가?

완료 기준은 영상을 모두 시청했다는 사실이 아니라, 다음 workflow를 직접 운영해 본 경험이다.

```text
작은 요구사항 정의
→ AI에게 구현 위임
→ Diff review
→ Test·static analysis 실행
→ 실패 원인 직접 수정
→ 결과와 한계 기록
```

## 5. 카카오: AI-native SDLC로 범위를 넓힌다

자료: [if(kakao)25](https://if.kakao.com/2025/session)

해외 사례에서 검증 기준을 익힌 뒤 AI를 code 생성뿐 아니라 quality, test, release와 monitoring까지 연결하려는 조직적 시도를 살펴본다. 생산성 수치는 적용 context와 측정 방법을 함께 보고, 자신의 조직에 그대로 일반화하지 않는다.

## 6. 우아한형제들: 실패 후 역할을 다시 설계한다

자료: [AI와 함께하는 테스트 자동화: 플러그인 개발기](https://techblog.woowahan.com/24568/)

완성된 test code 전체를 AI가 생성하는 초기 접근이 실패한 뒤, deterministic plugin이 구조와 template을 만들고 AI가 제한된 TODO를 채우도록 역할을 재설계한 사례다. 해외 사례에서 배운 guardrail이 한국 조직의 구체적인 개발 도구에 어떻게 구현되는지 비교한다.

## 7. NAVER·WOOWACON: 자신의 직무 문제로 확장한다

자료: [NAVER D2 YouTube](https://www.youtube.com/@naver_d2), [우아한Tech YouTube](https://www.youtube.com/@woowatech)

마지막에는 AI라는 제목만 찾지 않고 자신의 업무와 가까운 production 주제를 선택한다.

- 장애 대응과 observability
- 대규모 traffic과 distributed system
- Test automation과 CI/CD
- Data pipeline과 security
- Legacy migration
- Backend·frontend·mobile architecture

AI가 실제 가치를 만들려면 결국 이러한 engineering 문제와 결합돼야 한다. 해외에서 익힌 질문을 한국 사례에도 동일하게 적용해 공통점과 조직별 차이를 비교한다.

## 자료별 학습 기록 형식

각 자료를 본 뒤 아래 항목만 한 장으로 기록한다.

1. 해결하려던 production 문제
2. 조직·system의 주요 제약
3. AI에 맡긴 범위
4. 사람이 유지한 책임
5. 사용한 deterministic guardrail
6. 실패와 false positive
7. 배포 후 확인한 evidence
8. 내 project에서 재현할 가장 작은 실험

## 최종 방향

목표는 AI를 가장 많이 사용하는 개발자가 아니다. 해외 선도 사례에서 구현 위임과 production 책임이 어떻게 분리되는지 배우고, 다음 상태에 도달하는 것이다.

> AI 없이도 문제를 이해하고, AI를 쓰면 더 빠르게 실행하며, AI가 실패하면 독립적으로 검증하고 복구할 수 있는 개발자.

한국 사례는 이 기반을 갖춘 뒤 해외 workflow가 국내 조직의 legacy, 규제, 개발 문화와 tooling에서 어떻게 변형되는지 비교하는 단계로 사용한다.

## See Also

- [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](martin-fowler-ai-software-engineering-study-guide.md)
- [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](current-engineering-sources-and-ai-native-development.md)
- [AI 시대 신입 개발자의 생존과 성장 판단](ai-era-junior-developer-survival-assessment-2026-08-13.md)
- [Brownfield AI Agent Workflow](../ai-agents/brownfield-ai-agent-workflow.md)
