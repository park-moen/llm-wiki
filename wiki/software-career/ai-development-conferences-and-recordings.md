# AI 활용·개발 workflow 컨퍼런스 가이드

> Sources: Tessl·AI Engineer·Spring I/O, Unknown; Alina Dolgikh / JetBrains, 2025-08-22
> Raw: [AI Native DevCon](../../raw/software-career/ai-native-devcon.md); [AI Engineer Code Summit](../../raw/software-career/ai-engineer-code.md); [AI Engineer World's Fair](../../raw/software-career/ai-engineer-worlds-fair.md); [AI Engineer — Events and Talks](../../raw/software-career/ai-engineer.md); [Kotlin on the Backend — What’s New From KotlinConf 2025](../../raw/software-career/2025-08-22-kotlin-on-the-backend-from-kotlinconf.md); [Sessions — Spring I/O 2026](../../raw/software-career/spring-io-2026-sessions.md)
> Updated: 2026-10-21

## Overview

AI를 활용하는 방법은 개발팀의 조사·구현·검증 방식을 개선하는 일과, 서비스 안에 RAG·agent 같은 AI 기능을 구축하는 일로 나눠 살펴볼 수 있다. AI Native DevCon·AI Engineer Code Summit은 전자에, World’s Fair는 후자까지 넓혀 보는 데 적합하다는 것이 이 문서의 추천이다. 공식 행사 주제와 영상 안내를 바탕으로 한 학습 가이드이며 성능·생산성 순위는 아니다.

## AI Native DevCon: 개인의 AI 사용을 팀의 개발 방식으로 연결

Tessl의 공식 안내는 agent를 실제 코드베이스에 적용하는 개발자와 팀을 대상으로 한다. Context & Skills Engineering, agent의 보안·신뢰, 공통 스킬·workflow의 공유와 버전 관리, 지속적인 평가와 개선을 주제로 제시한다.

따라서 레슨을 스킬로 축적하거나, 기획·구현·검증 단계에서 AI의 역할을 정하는 방법을 고민할 때 우선 탐색할 만하다. 이는 행사 주제와 학습 목적을 연결한 추천이며, 특정 발표 방식의 효과를 검증했다는 뜻은 아니다.

### 무료 시청과 이전 녹화본

- [공식 AI Native DevCon 안내](https://tessl.io/devcon)
- [London 발표 재생목록](https://www.youtube.com/watch?v=KpfnldjO3Iw&list=PLISstAySqk7KtlYPFps1ZnK9xNIMSVkDD): 공식 London highlights가 연결한다.
- [Meta — AI Native Engineering](https://www.youtube.com/watch?v=NVzKtCSeoRA): 공식 Previous Speakers에 연결된 발표다.
- [Thoughtworks — State of Play: AI Coding Assistants](https://www.youtube.com/watch?v=tFffUnSq7VA): 공식 페이지에 연결된 발표다.

공식 안내의 무료 Virtual ticket은 keynote 생중계, 선별된 온라인 세션과 행사 후 녹화본을 포함한다. 이전 행사 YouTube 공개와 향후 행사의 Virtual 제공 범위를 구분하며, 모든 발표가 YouTube에 공개된다고 단정하지 않는다.

## AI Engineer Code Summit: AI를 이용한 소프트웨어 개발

[Code Summit 공식 안내](https://ai.engineer/code/2026)는 coding agent와 내부 agent를 만들거나 조직에 도입하는 엔지니어·리더를 대상으로 실제 workflow와 경험을 다룬다고 설명한다. 일반적인 모델 소개보다 AI를 개발 도구로 쓰는 방식에 관심이 있을 때 적합하다.

다만 공식 안내는 기초를 이미 알고 있는 실무자를 대상으로 한다고 명시한다. 처음 접한다면 모든 발표를 따라가기보다 현재 자신의 도구·작업과 가까운 사례를 고르는 편이 낫다. 행사 참가 조건과 공개 영상 시청 조건도 구분한다.

## AI Engineer World’s Fair: AI 기능을 실제 서비스로 구축·운영

[World’s Fair 공식 안내](https://ai.engineer/worldsfair/2026)는 production AI 시스템, agent, copilot, 평가, RAG pipeline과 인프라를 다룬다. AI를 코딩에 활용하는 방법뿐 아니라, 서비스 안에 AI 기능을 만들고 운영하는 방법으로 관심을 넓힐 때 적합하다.

- [공식 AI Engineer YouTube](https://www.youtube.com/@aiDotEngineer)
- [World’s Fair 공식 재생목록](https://www.youtube.com/playlist?list=PLcfpQ4tk2k0V1LNigteMgExP1rb4Hy8wn)

공식 페이지는 행사 종료 후 발표를 YouTube에 계속 공개한다고 설명한다. 공개된 영상을 찾을 수 있다는 안내이며, 아직 공개되지 않은 세션까지 모두 볼 수 있다는 보장은 아니다.

## 바로 탐색할 AI Engineer 발표

[AI Engineer 공식 발표 모음](https://ai.engineer/)에는 다음 자료가 연결돼 있다.

| 발표 | 선택할 때의 관심사 |
|---|---|
| [Full Walkthrough: Workflow for AI Coding — Matt Pocock](https://ai.engineer/talks/-QFHIoCo-Ko-ai-coding-workflow) | AI coding workflow를 살펴보기 |
| [Claude Code & the evolution of agentic coding — Boris Cherny](https://ai.engineer/talks/Lue8K2jqfKk-claude-code-evolution-agentic-coding) | agentic coding 도구와 활용 방식의 변화 |
| Don’t Build Agents, Build Skills Instead — Barry Zhang & Mahesh Murag | agent와 재사용 가능한 스킬의 역할 구분 |

이 표는 공식 목록의 제목을 기준으로 고른 탐색 출발점이다. 이 ingest에서는 발표 영상·transcript 전체를 수집하지 않았으므로 세부 주장을 검증한 요약으로 읽지 않는다. 링크된 페이지에서 원영상과 발표 당시의 도구·버전을 확인한다.

## Kotlin·Spring에서 AI를 적용할 때

AI 개발 workflow뿐 아니라 Kotlin·Spring 애플리케이션 안에 AI 기능을 구현하려는 경우, KotlinConf와 Spring I/O를 함께 살펴볼 수 있다.

JetBrains의 [KotlinConf 백엔드 발표 안내](https://blog.jetbrains.com/kotlin/2025/08/kotlin-on-the-backend-what-s-new-from-kotlinconf-2025/)에는 Koog·Mellum·Junie가 소개되어 있다. 원문은 Koog를 Kotlin으로 AI agent를 만들고 실행하는 framework로 설명한다. AI 도구로 개발을 돕는 일과 애플리케이션 자체에 agent 기능을 구현하는 일을 구분해서 읽는다.

[Spring I/O 공식 세션 목록](https://2026.springio.net/sessions/)에는 다음 발표가 있다.

| 발표 | 제목을 기준으로 한 탐색 목적 |
|---|---|
| [The Spring AI Ecosystem in 2026: From Foundations to Agents](https://2026.springio.net/sessions/the-spring-ai-ecosystem-in-2026-from-foundations-to-agents/) | Spring AI의 생태계와 agent 관련 주제 |
| [Supercharging Spring AI: Scalable AI Agents with Koog + Spring AI + Spring Boot](https://2026.springio.net/sessions/supercharging-spring-ai-scalable-ai-agents-with-koog-spring-ai-spring-boot/) | Koog·Spring AI·Spring Boot를 연결하는 주제 |
| [Skyrocket Developer Productivity with Spring Boot & IntelliJ IDEA](https://2026.springio.net/sessions/skyrocket-developer-productivity-with-spring-boot-and-intellij-idea/) | Spring Boot와 IntelliJ IDEA 개발 환경 활용 |

세션 제목과 공식 소개를 바탕으로 고른 출발점이며, 영상·transcript 전체를 수집하거나 발표의 성능·생산성 효과를 검증한 요약은 아니다. 녹화본 경로는 [일반 컨퍼런스 가이드의 Kotlin·IntelliJ·Spring 항목](developer-conferences-and-recordings.md)을 참고한다.

## AI를 어떻게 활용할지 배우는 관찰 기준

다음은 원문의 행사 주제를 참고한 이 문서의 학습 제안이다. 발표를 볼 때 아래 질문을 기록하고 자신의 작은 작업으로 검증한다.

1. 해결하려는 문제와 성공 조건은 무엇인가?
2. AI에 맡긴 일과 사람이 판단한 일은 무엇인가?
3. agent에게 어떤 코드·문서·지침·도구를 제공했는가?
4. 테스트·평가·리뷰에서 무엇을 확인하고 어떤 실패를 발견했는가?
5. 유용했던 경험을 다음 작업에 재사용하고, 잘못된 지침을 고칠 방법이 있는가?

예를 들어 개발 workflow가 목적이라면 작은 기능 하나의 요구사항 확인·구현·검증 흐름을 관찰한다. RAG 챗봇이 목적이라면 검색·답변 평가와 비용·운영 제약을 설명하는 세션을 고른다. 발표자가 한 번 성공한 데모와 반복해서 유지할 수 있는 운영 방식은 구분한다.

무료 영상을 볼 수 있다는 사실이 내용의 정확성이나 자신의 프로젝트에서의 효과를 보장하지는 않는다. 주최사의 제품·후원 관계와 발표 당시의 조건을 함께 확인한다. 영상 재생과 한국어 자막 제공 여부는 이번 수집에서 시험하지 않았다.

## See Also

- [해외 개발자 컨퍼런스와 녹화본 탐색 가이드](developer-conferences-and-recordings.md)
- [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](current-engineering-sources-and-ai-native-development.md)
- [해외 현업 중심 AI 네이티브 엔지니어링 학습 로드맵](overseas-ai-native-engineering-learning-path.md)
- [실행 경험을 레슨 스킬로 축적하는 Agent 운영](../if(kakao)-26/learning-from-experience-with-agent-skills.md)
- [RAG 구축·비용·답변 범위와 평가](../ai-engineering/rag-design-cost-and-evaluation.md)
