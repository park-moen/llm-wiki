# AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵

> Sources: [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md); [취업 초기 주니어를 위한 AI-Native 개발 프로세스](junior-ai-native-development-process.md); [AI 시대 신입 개발자의 생존과 성장 판단](ai-era-junior-developer-survival-assessment-2026-08-13.md); [스프링 입문 학습 로드맵](../spring/spring-learning-roadmap.md); [Software Engineering을 Time·Scale·Trade-off로 이해하기](software-engineering-time-scale-tradeoffs.md)
> Archived: 2026-08-23

## Overview

JS·React·Next.js와 Kotlin·Spring Boot를 사용하는 입사 초기 개발자가 AI 중심의 실무 환경에서도 독립적인 software engineering 역량을 성장시키기 위한 6개월 로드맵이다. 목표는 AI를 능숙하게 지시하는 데 머물지 않고, 작은 기능의 요구사항과 실행 경로를 이해하고, 설계·구현·검증·복구까지 책임질 수 있는 개발자가 되는 것이다. React·Next.js는 실무를 통해 유지하면서 초보 단계인 Kotlin·Spring의 실행 모델을 깊이의 기준점으로 삼고, 회사의 AI 활용 방식은 인상보다 PR·CI·test·배포·장애 대응 evidence로 평가한다.

## 학습 우선순위

```text
1. 회사 code와 개발 흐름 파악
2. Kotlin·Spring 실행 모델 확보
3. AI 기반 작은 기능 End-to-End 소유
4. Refactoring·OOP·Architecture를 실제 code에 적용
5. 검증 가능한 AI workflow와 운영 판단 구축
```

언어와 framework API를 처음부터 끝까지 암기하지 않는다. 실제 업무 흐름을 따라가면서 state, dependency, transaction, boundary, testability와 failure mode를 이해한다.

## 회사의 AI 환경을 판단하는 기준

AI 사용량 자체는 성장 가능성을 판정하지 않는다. 중요한 것은 AI가 만든 변경을 사람이 책임질 수 있는 구조가 있는지다.

### 성장 가능한 환경

- AI가 만든 변경을 사람이 설명할 수 있다.
- Test, type check, lint와 CI가 실제로 실행된다.
- 중요한 설계와 요구사항은 사람이 결정한다.
- 장애가 발생하면 원인 분석과 재발 방지 활동을 한다.
- 모든 line을 읽지 않더라도 interface, test와 위험 영역을 검토한다.
- 주니어가 작은 기능을 분석부터 배포까지 소유할 기회를 얻는다.

### 성장 위험이 있는 환경

- 화면이 작동하면 완료로 처리한다.
- AI의 성공 보고를 증거 없이 신뢰한다.
- Test가 없거나 AI가 만든 test의 의미를 검토하지 않는다.
- 오류가 발생하면 원인을 찾지 않고 prompt만 반복한다.
- 실제 호출 흐름과 architecture를 설명할 사람이 없다.
- Production 장애와 rollback의 책임 구조가 없다.
- 주니어가 code를 이해하거나 직접 debug할 기회를 얻지 못한다.

입사 한 달 시점의 인상만으로 회사와 사수를 확정적으로 평가하지 않는다. 이후 1~2개월 동안 실제 PR, review comment, CI, test, 배포와 장애 대응 기록을 관찰한다.

## 1단계: 회사 System 지도 만들기 — 1~2주

별도 강의보다 실제 업무 system을 첫 학습 자료로 사용한다.

### Frontend 흐름 추적

작은 사용자 행동 하나를 선택한다.

```text
Button click
→ React component
→ state 변경
→ API 호출
→ Next.js route 또는 backend endpoint
→ response
→ 화면 갱신
```

다음을 확인한다.

- 상태는 누가 소유하는가
- Server와 client의 경계는 어디인가
- API 호출은 어디에서 시작되는가
- Loading, error와 empty state는 어디에서 처리하는가
- Cache가 있다면 언제 무효화되는가
- 실패하면 사용자는 무엇을 보는가

### Backend 흐름 추적

```text
HTTP request
→ Controller
→ Service
→ Repository
→ Database
→ Transaction 종료
→ HTTP response
```

다음을 확인한다.

- Request DTO는 어디에서 검증되는가
- Business rule은 어느 계층에 있는가
- Entity와 response model은 분리되는가
- Transaction boundary는 어디인가
- Exception은 어떻게 HTTP status로 변환되는가
- 어떤 test가 이 흐름을 보호하는가

### 산출물

- Frontend request 흐름도 1개
- Backend request 흐름도 1개
- Frontend와 backend의 검증 command 목록

Repository 전체를 한 번에 이해하려 하지 않는다. 현재 요청과 연결된 entry point부터 실제 호출 경로를 따라간다.

## 2단계: Kotlin·Spring 실행 모델 — 1~2개월

### Kotlin 학습 순서

1. Null safety
2. 함수와 named·default argument
3. Class, data class와 constructor
4. Interface와 상속
5. Collection
6. Lambda와 higher-order function
7. Extension function
8. `object`, companion object와 sealed class
9. Exception과 type modeling
10. Coroutine은 실제 project에서 사용할 때 집중

각 주제는 다음 기준으로 완료한다.

- 회사 code에서 사용 사례를 찾는다.
- Input, output과 상태 변화를 설명한다.
- 작은 예제를 직접 작성한다.
- 잘못 사용한 failure case를 만든다.
- AI가 제안한 code와 직접 작성한 code를 비교한다.

### Spring 학습 순서

1. Spring Boot project 구조와 실행
2. Bean과 dependency injection
3. MVC request·response 흐름
4. Validation과 exception handling
5. Repository와 database 연결
6. Unit test와 integration test
7. Transaction
8. JPA persistence context와 entity lifecycle
9. Spring Data JPA
10. Proxy와 AOP
11. Security는 project에서 필요해질 때 별도로 집중

특히 다음을 자신의 말로 설명할 수 있어야 한다.

- Class가 존재하는 것과 Spring Bean으로 관리되는 것의 차이
- Constructor injection이 object graph를 만드는 과정
- 직접 `new`한 객체와 container가 만든 객체의 차이
- `@Transactional`이 실제로 적용되는 범위
- Controller, Service와 Repository의 책임
- Test transaction의 rollback
- Proxy가 method 호출에 미치는 영향

### 주제별 학습 기록

```text
개념 설명 1개
실제 회사 code 사례 1개
작은 실행 실험 1개
Failure case 1개
검증하는 Test 1개
```

강의 수강 완료보다 이 산출물이 실제 이해의 기준이다.

## 3단계: AI와 작은 기능 소유 — 2~3개월

CRUD, validation이나 작은 UI 변경처럼 영향 범위가 좁은 실제 Issue 하나를 학습 단위로 사용한다.

### AI 실행 전 작성할 내용

```markdown
## 문제

사용자에게 어떤 문제가 있는가?

## 기대 동작

정상 입력이면 무엇이 발생하는가?

## 실패 동작

잘못된 입력, 권한 없음과 network failure에서는 어떻게 되는가?

## 변경 경계

어떤 frontend·backend module이 영향을 받는가?

## 검증

어떤 test와 실제 동작으로 완료를 확인하는가?
```

문서의 목적은 처음부터 완벽한 spec을 만드는 것이 아니라 모르는 부분과 가정을 드러내는 것이다.

### 역할 분리

AI에는 관련 entry point 조사, 모호한 부분 질문, 복수의 구현안, 작은 구현, test 후보와 diff 위험 분석을 맡긴다.

사람은 다음을 소유한다.

- 문제와 완료 조건
- Domain 용어
- Public interface
- Data와 transaction boundary
- Security와 권한
- Test가 검증해야 할 behavior
- 최종 diff와 배포 판단

### 구현 Loop

```text
Behavior 하나 선택
→ Test 또는 검증 방법 결정
→ AI가 작은 변경 구현
→ Diff 확인
→ Test 실행
→ 호출 흐름을 직접 설명
→ 다음 Behavior로 이동
```

기능 전체를 한 번에 생성시키지 않는다. 설명하지 못하는 code가 남았다면 위임 범위가 현재 학습 범위를 넘어선 것이다.

## 4단계: Refactoring과 OOP 적용 — 3~4개월

Refactoring, Clean Code와 OOP 자료를 실제 변경 경험에 연결한다.

```text
회사 code에서 불편한 변경 경험
→ 관련 설계 원리 학습
→ Characterization test 작성
→ AI에게 복수 개선안 요청
→ Coupling과 변경 비용 비교
→ 작은 Refactoring
→ 기존 Behavior 유지 확인
→ 적용 조건과 한계 기록
```

Service가 많은 책임을 가진 것처럼 보여도 즉시 class를 분리하지 않는다. 어떤 이유로 code가 함께 변경되는지, 기존 behavior를 어떤 test가 보호하는지, 분리 후 interface와 dependency가 실제로 단순해지는지 확인한다. File 수만 늘리는 shallow module은 피한다.

이 단계에서는 responsibility, cohesion, coupling, dependency direction, encapsulation, composition, polymorphism, interface, side effect와 behavior preservation을 학습한다. SOLID와 pattern 이름을 암기하는 대신 현재 변경 비용을 실제로 낮추는지 판단한다.

## 5단계: Architecture와 Agile — 4~6개월

### Architecture 질문

- System의 핵심 business rule은 어디에 있는가
- Framework를 바꾸면 무엇이 영향을 받는가
- Database 변경이 domain logic까지 전파되는가
- Module 간 dependency 방향은 무엇인가
- 어떤 interface가 외부에 공개돼 있는가
- 되돌리기 어려운 결정은 무엇인가
- 장애 원인을 관찰할 수 있는 log와 metric이 있는가
- Migration과 backward compatibility는 누가 책임지는가

Clean Architecture를 folder template이 아니라 변경 비용과 dependency를 설명하는 언어로 사용한다.

### Agile 질문

- 요구사항을 검증 가능한 작은 behavior로 나누었는가
- Feedback을 얼마나 빨리 받을 수 있는가
- 잘못된 가정을 작은 비용으로 발견하는가
- 긴 branch와 큰 diff를 피하는가
- 배포 후 실제 사용자 반응을 확인하는가
- 회고 결과가 다음 workflow에 반영되는가

Agile은 Jira와 ceremony 사용법이 아니라 feedback loop와 incremental delivery의 설계로 학습한다.

## React·Next.js 유지 전략

React·Next.js 입문 강의를 반복하기보다 실제 실행 모델을 강화한다.

### React

- Render와 event handler의 관계
- State ownership과 derived state
- Effect가 필요한 경우와 필요하지 않은 경우
- Component boundary
- Controlled form
- Loading, error와 empty state
- UI test가 검증해야 하는 behavior

### Next.js

- Server와 client 실행 경계
- Routing과 layout
- Data fetching 위치
- Cache와 freshness
- Authentication과 authorization의 위치
- API 또는 Server Action의 trust boundary
- Build time과 request time의 차이
- Error, not-found와 loading 처리

버전별 API는 필요할 때 공식 문서와 AI로 찾는다. Code가 어디에서 실행되고 누가 상태를 소유하는지를 먼저 이해한다.

## 매주 반복할 운영 Routine

### 실무 Issue마다

1. AI 실행 전에 문제와 기대 behavior를 짧게 작성한다.
2. 관련 호출 경로를 AI가 찾게 한 뒤 IDE에서 직접 확인한다.
3. Plan의 scope, test와 위험을 검토한다.
4. Behavior 하나씩 구현한다.
5. 매 변경 후 diff와 test를 확인한다.
6. 작업 종료 시 변경을 자신의 말로 설명한다.

### 개인 학습

- Kotlin·Spring 실행 실험
- 실제 회사 code 흐름 추적
- Software design 원리의 실제 적용
- AI 없이 작은 핵심 logic 작성 또는 bug 원인 가설 세우기

AI 없이 하는 연습은 과거의 생산 방식으로 돌아가기 위한 것이 아니다. 독립적으로 사고하고 복구하는 능력이 유지되는지 확인하는 훈련이다.

## 회사 환경 2개월 관찰표

| 관찰 대상 | 성장 신호 | 위험 신호 |
|---|---|---|
| PR | 요구사항, 위험과 test를 검토 | 생성 결과를 바로 merge |
| Test | 사람이 의미를 확인 | AI가 만든 test를 그대로 신뢰 |
| CI | Build, test와 lint가 실제 gate | 실패해도 우회하거나 실행하지 않음 |
| Architecture | 결정 이유와 trade-off가 있음 | 현재 구조를 아무도 설명하지 못함 |
| 장애 | 재현, 원인과 재발 방지를 기록 | Prompt를 바꿔 다시 배포 |
| 사수 | 호출 흐름과 판단을 설명 | AI 결과만 전달 |
| CTO | 품질과 운영 책임 구조를 설계 | 생성 속도만 평가 |
| 본인 | 작은 기능을 끝까지 소유 | Prompt 전달과 결과 확인만 수행 |

사수에게 code를 보는지 직접 추궁하기보다 다음처럼 완료와 검증 기준을 질문한다.

- 이 PR의 완료 기준은 무엇인가요?
- 이 변경에서 가장 위험한 부분은 어디인가요?
- 어떤 test가 회귀를 막고 있나요?
- 이 Service의 transaction boundary는 어디인가요?
- 장애가 발생하면 어떤 log와 metric부터 확인하나요?
- AI가 만든 변경을 merge하기 전에 무엇을 확인하나요?

답변과 실제 workflow가 일치하는지를 관찰한다.

## 6개월 후 완료 기준

- React 화면에서 Spring과 database까지 request 흐름을 추적한다.
- Kotlin code의 nullability, state와 type 의도를 설명한다.
- Spring Bean, DI, transaction과 proxy 동작을 설명한다.
- AI 실행 전에 요구사항의 빈칸을 발견한다.
- AI가 제안한 설계를 근거를 들어 거절하거나 수정한다.
- Test가 보장하는 것과 보장하지 않는 것을 설명한다.
- Production bug를 재현하고 원인 가설을 세운다.
- 작은 기능을 분석부터 검증까지 독립적으로 책임진다.
- Architecture 원칙을 pattern 이름보다 변경 비용으로 설명한다.
- AI가 실패해도 작업을 이어가고 복구한다.

최종 성장 기준은 AI가 만든 code의 양이 아니다. AI 없이도 문제를 이해하고, AI를 사용하면 더 빠르게 실행하며, AI가 실패하면 독립적으로 검증하고 복구할 수 있어야 한다.

## See Also

- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](claude-code-team-ai-native-development-workflow.md)
- [AI를 활용한 개발자 성장과 Career 판단](ai-assisted-engineering-growth-and-career-judgment.md)
- [Spring Bean과 의존관계 설정](../spring/spring-beans-and-dependency-injection.md)
- [Spring DB 접근 기술 비교](../spring/spring-database-access-technologies.md)
