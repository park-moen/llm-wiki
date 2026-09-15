# 최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링

> Sources: NAVER D2 (2026-05-28); 우아한형제들 기술블로그 (2025-11-19, 2025-12-04); kakao tech (2025-11-17); Toss SLASH24 (2024-09-12); 당근 기술 블로그 (2026-04-22); Stripe Dev Blog (2026-02-19); Uber Engineering (2025-08-12); Birgitta Böckeler·Thoughtworks (2025-08-05); The Pragmatic Engineer·Martin Fowler (2025-11-19); Matt Pocock (Unknown); Boris Cherny interview, YouTube (Unknown); Pasha interview·Beyond Coding (Unknown); Tejas Kumar·IBM, YouTube (Unknown); Jesse Vincent interview, YouTube (Unknown)
> Raw: [NAVER Engineering Day](../../raw/software-career/2026-05-28-naver-engineering-day-public-sessions.md); [WOOWACON 2025](../../raw/software-career/2025-11-19-woowacon-2025.md); [우아한형제들 AI 테스트 자동화](../../raw/software-career/2025-12-04-woowa-ai-test-automation.md); [카카오 AI Native SDLC](../../raw/software-career/2025-11-17-kakao-ai-native-sdlc.md); [Toss SLASH24 테스트 플랫폼](../../raw/software-career/2024-09-12-toss-slash24-test-platform.md); [당근 AI 데이터 팀](../../raw/software-career/2026-04-22-daangn-ai-data-team.md); [Stripe Minions](../../raw/software-career/2026-02-19-stripe-minions-coding-agents.md); [Uber uReview](../../raw/software-career/2025-08-12-uber-ureview.md); [AI autonomy 실험 요약](../../raw/software-career/2025-08-05-ai-autonomy-human-in-loop.md); [AI autonomy 실험 전체 원문](../../raw/software-career/2025-08-05-ai-autonomy-human-in-loop-2.md); [Martin Fowler AI interview](../../raw/software-career/2025-11-19-martin-fowler-ai-interview.md); [Matt Pocock software fundamentals talk](../../raw/software-career/software-fundamentals-matter-more-than-ever.md); [Building Claude Code with Boris Cherny transcript](../../raw/software-career/building-claude-code-boris-cherny.md); [Original YouTube source provenance](../../raw/software-career/building-claude-code-boris-cherny-source-provenance.md); [From Backend Engineer to Head of Mobile transcript](../../raw/software-career/from-backend-engineer-to-head-of-mobile-lessons-uber.md); [Harnesses in AI transcript](../../raw/software-career/harnesses-in-ai-a-deep-dive-tejas-kumar-ibm.md); [Original Harnesses in AI YouTube source provenance](../../raw/software-career/harnesses-in-ai-tejas-kumar-source-provenance.md); [Fixing AI Slop interview transcript](../../raw/software-career/fixing-ai-slop-manage-agents-like-mit-interns.md)
> Updated: 2026-08-16

## Overview

한국 빅테크의 현업 개발 자료가 모두 오래됐다는 판단은 정확하지 않다. NAVER, 우아한형제들, 카카오와 당근은 최근 자료를 계속 공개하고 있다. 다만 공개 위치가 YouTube 한곳으로 통일되지 않고 기술 블로그, 연례 conference 다시보기와 채용 인터뷰로 분산돼 있어 오래된 채널만 구독하면 흐름을 놓치기 쉽다.

최근 사례가 보여주는 방향도 사람이 이해하지 않은 채 AI에게 전부 위임하는 vibe coding과는 다르다. 구현 위임은 커지지만 production에서는 문제 정의, context 제공, review, test, release, monitoring과 운영 책임을 인간과 조직의 검증 체계가 감싼다. 더 정확한 표현은 **AI-native software engineering**이다.

## 한국 기업 자료의 최신성 지도

| 기업 | 최근 확인 자료 | 무엇을 볼 수 있나 | 추천 진입점 |
|---|---|---|---|
| NAVER | Engineering Day 2026 | 사내 기술 세션, AI workshop, 실무 기술 도입 | [NAVER D2 YouTube](https://www.youtube.com/@naver_d2), [D2](https://d2.naver.com/) |
| 우아한형제들 | WOOWACON 2025, AI 테스트 자동화 | 장애 대응, backend, AI agent, test와 개발 문화 | [우아한Tech YouTube](https://www.youtube.com/@woowatech), [기술블로그](https://techblog.woowahan.com/) |
| 카카오 | if(kakao)25 | AI-native SDLC, code quality, test, release와 monitoring | [if(kakao)25 세션](https://if.kakao.com/2025/session), [kakao tech YouTube](https://www.youtube.com/@kakaotech) |
| 토스 | SLASH24 | 대규모 test 자동화, infra, server, frontend와 QA | [SLASH24 테스트 플랫폼 세션](https://toss.im/slash-24/sessions/15), [toss tech](https://toss.tech/) |
| 당근 | 2026년 데이터 팀 사례 | AI prototype, MCP, governance와 팀의 문제 해결 방식 | [당근 기술 블로그](https://medium.com/daangn), [당근 채용 블로그](https://careers.daangn.com/blog/) |

이 표에서 중요한 점은 YouTube 활동량과 실무 정보의 신선도를 동일시하지 않는 것이다. NAVER·우아한형제들·카카오처럼 conference 영상을 공개하는 기업은 영상이 좋은 출발점이다. 당근처럼 최신 변화가 글과 인터뷰에 먼저 쌓이는 기업은 blog를 함께 추적해야 한다. 토스는 연례 SLASH session archive가 강하지만 확인된 최신 대표 개발 conference는 SLASH24이므로 다른 기업보다 시차를 감안한다.

## 현업에서 AI는 실제로 어디까지 맡는가

### 구현 위임은 이미 크게 늘었다

Stripe의 Minions는 agent가 한 변경의 코드를 처음부터 끝까지 작성한다. 카카오는 prototype, hackathon뿐 아니라 code quality, test, release와 monitoring까지 AI 적용 범위를 넓히고 있다. 당근도 prototype과 내부 데이터 도구를 실제 팀 workflow에 연결한다.

따라서 AI가 boilerplate와 작은 변경만 돕는다고 보는 것도 현실보다 보수적이다. Agent에게 상당한 구현을 맡기고 개발자는 여러 작업을 감독하는 방식은 이미 실제 조직에 존재한다.

### 그러나 production 책임까지 무검증으로 넘기지는 않는다

Stripe에서는 사람이 agent의 pull request를 review한다. Uber는 AI review 결과를 그대로 신뢰하지 않고 generation, filtering, validation, deduplication과 developer feedback을 별도 단계로 운영한다. 우아한형제들의 테스트 자동화도 완성 코드를 한 번에 생성하는 첫 접근이 실패한 뒤, deterministic template과 IDE context, 인간의 최종 검증으로 역할을 나눴다.

Thoughtworks의 Spring Boot 실험에서는 작은 CRUD application은 비교적 잘 생성했지만 entity와 관계가 늘어나자 가정의 일관성이 흔들리고, 증상만 누르는 수정과 test 실패 상태의 성공 선언이 나타났다. Stack별 지침, reference application, modularization과 generate-review 반복은 결과를 개선했지만 인간의 감독을 제거하지 못했다. 이 사례들은 “AI가 만들었으니 완료”가 아니라 다음 구조가 현업형에 가깝다는 것을 보여준다.

Claude Code 팀의 workflow는 더 높은 구현 자율성과 같은 원칙이 공존할 수 있음을 보여준다. 익숙하지 않은 codebase에서는 설명을 따라가며 학습하고, 익숙한 영역에서는 plan을 먼저 맞춘 뒤 여러 checkout이나 Worktree로 병렬 처리한다. 생성된 변경은 local test, type checker·linter·build, AI code review와 human engineer의 최종 review를 겹쳐 검증한다. 높은 agent 사용량은 fundamentals의 대체물이 아니라 강한 feedback과 review 기반 위의 운영 mode다.

Pasha의 interview는 이 원칙을 개인 개발 workflow에서 보완한다. AI를 junior·medium 동료처럼 두고 spec과 code를 반복 review하며, 프로젝트 Markdown으로 architecture와 세부 규칙을 제공한다. 하나의 feature도 planner·implementer·reviewer context로 분리할 수 있지만, 사람은 AI가 생성한 context 문서와 code를 실제 repository와 대조하고 끝까지 이해한다.

Tejas Kumar의 harness demo는 검증을 runtime 구조로 더 구체화한다. Tool registry, context management, guardrail, trace, deterministic verifier와 retry 상한을 model 주변에 두고, agent가 성공을 선언해도 실제 상태가 조건을 만족하지 못하면 실패로 판정한다. 이는 AI-native engineering이 prompt 작성법을 넘어 모델의 행동을 관찰·제한·검증하는 software system 설계임을 보여준다.

Jesse Vincent의 Superpowers workflow는 이 구조의 관리 측면을 보완한다. Brainstorming으로 사람의 intent를 spec으로 만들고, plan을 작은 TDD task로 분해하며, implementer·spec reviewer·quality reviewer의 역할을 분리한다. 다만 인터뷰 시점에는 별도 behavioral testing이 완전히 내장된 것은 아니므로 실제 사용자 흐름의 proof와 deterministic completion gate는 repository별 harness가 추가해야 한다.

Martin Fowler는 이 문제를 deterministic software engineering과 non-deterministic AI의 결합으로 설명한다. AI 변경을 review 가능한 작은 slice로 제한하고 test, IDE refactoring과 static analysis 같은 반복 가능한 도구로 감싸야 한다. 이때 목표는 한 번에 더 많은 code를 생성하는 것이 아니라 feedback loop를 더 짧게 만드는 것이다.

1. 사람이 문제와 성공 조건을 정의한다.
2. AI가 조사, 구현과 반복 작업을 수행한다.
3. test, static analysis와 policy가 기계적으로 검증한다.
4. 사람이 architecture, 위험과 변경 의미를 review한다.
5. 단계적 release와 monitoring으로 production 결과를 확인한다.

## 개발자가 발전해야 할 방향

AI 도구 사용법은 필요하지만 독립된 최종 목적은 아니다. 장기적으로 강화할 능력은 다음과 같다.

- 모호한 요구를 검증 가능한 성공 조건으로 바꾸기
- 기존 codebase의 구조와 변경 영향을 추적하기
- API, database, network, concurrency와 distributed system의 failure mode 이해하기
- unit·integration·E2E test와 observability로 결과를 증명하기
- security, privacy, cost와 운영 위험을 판단하기
- AI가 빠르게 일할 수 있도록 repository context와 deterministic guardrail 만들기
- AI가 틀렸을 때 직접 debug하고 복구하기

Matt Pocock의 발표는 이 방향을 codebase 설계 수준으로 구체화한다. 사람과 AI가 같은 `design concept`와 `ubiquitous language`를 공유하고, TDD로 작은 feedback loop를 강제하며, 복잡성을 단순한 interface 뒤에 숨기는 `deep module`을 설계해야 한다. AI가 tactical implementation을 빠르게 수행하더라도 module boundary와 public interface를 결정하는 전략적 책임은 사람이 소유한다.

AI를 쓰지 않는 개발자가 되는 것도, AI 출력에 판단을 포기하는 운영자가 되는 것도 목표가 아니다. **AI 없이도 문제를 이해하고, AI를 쓰면 더 빠르게 실행하며, AI가 실패하면 독립적으로 복구할 수 있는 개발자**가 현실적인 성장 방향이다.

## YouTube 학습 순서

### 1. 한국 기업의 실제 문제부터 본다

NAVER D2와 우아한Tech에서 익숙한 stack의 최근 conference session을 고른다. 도구 이름보다 production 문제와 해결 이유, 제약과 실패, 검증·운영 방식을 기록한다.

### 2. AI 성공담과 실패담을 짝지어 본다

카카오의 AI-native SDLC 사례를 본 뒤 우아한형제들의 테스트 자동화 실패·개선 사례를 읽는다. 빠른 prototype과 durable software의 기준이 다르다는 점을 비교한다.

### 3. 해외 대규모 조직의 검증 구조를 본다

Stripe와 Uber 사례에서 agent 수보다 review, evaluation dataset, filtering, CI와 feedback loop에 주목한다. Martin Fowler·Thoughtworks의 autonomy 실험으로 반대 증거도 확인한다.

### 4. 시청을 작은 재현으로 끝낸다

영상 한 편마다 자신의 project에 적용 가능한 한 가지를 고른다. 예를 들어 test fixture 자동화, pull request checklist, structured logging 또는 rollback 절차를 직접 구현하고 결과를 기록한다. 시청량보다 재현과 설명 가능성이 성장에 더 직접 연결된다.

## 자료를 평가하는 질문

- 발표자가 실제 production 규모와 제약을 설명하는가?
- 성공 수치의 측정 방법과 비교 기준이 공개돼 있는가?
- 실패, false positive, 운영 비용과 trade-off도 다루는가?
- test, review, rollout, monitoring과 incident 대응이 등장하는가?
- conference 홍보가 아니라 재현 가능한 architecture와 판단 근거가 남는가?
- 최신 영상이 없을 때 공식 기술 블로그나 session archive가 갱신되고 있는가?

## 결론

뒤처지고 있다는 불안 때문에 모든 일을 agent에게 넘길 필요는 없다. 현업의 변화는 분명 빠르지만, 경쟁력은 prompt 횟수보다 더 큰 범위의 engineering 책임을 맡는 데서 나온다. AI 위임 능력과 software engineering의 기초는 대체 관계가 아니라 결합 관계다.

한국 기업 자료도 충분히 최근 내용을 찾을 수 있다. 다만 YouTube 구독 목록만 보지 말고 conference archive와 기술 블로그를 한 묶음으로 추적해야 한다.

## See Also

- [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](claude-code-team-ai-native-development-workflow.md)
- [AI Coding Autonomy Experiment와 Human-in-the-loop](../ai-agents/ai-coding-autonomy-experiment.md)
- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](martin-fowler-ai-software-engineering-study-guide.md)
- [AI 시대 신입 개발자의 생존과 성장 판단](ai-era-junior-developer-survival-assessment-2026-08-13.md)
- [Brownfield AI Agent Workflow](../ai-agents/brownfield-ai-agent-workflow.md)
- [AI Agent 산문 게이트와 결정적 게이트](../ai-agents/ai-agent-prose-vs-deterministic-gates.md)
- [Agent Harness의 구조와 Deterministic Control Loop](../ai-agents/agent-harness-anatomy-and-deterministic-control-loop.md)
- [AI를 활용한 개발자 성장과 Career 판단](ai-assisted-engineering-growth-and-career-judgment.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](../ai-agents/superpowers-agent-management-and-spec-driven-development.md)
