# 요구사항 분석부터 설계 다이어그램까지: Spring·React 통합 학습 로드맵

> Sources: [Spring 회원 관리 백엔드와 테스트](../spring/spring-member-backend-and-testing.md); [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md); [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md); [Frontend OCP·DIP와 React 설계 패러다임 통합 가이드](frontend-ocp-dip-and-react-software-engineering-guide-2026-08-24.md); [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](../ai-agents/superpowers-brownfield-practice-workflow-initial-design.md)
> Archived: 2026-08-24

## Overview

Spring 입문 강의에서 비즈니스 요구사항을 정리하고 class·object diagram으로 구조를 먼저 보여주는 방식은 코드를 읽기 전에 책임, 관계와 실행 시점의 조립을 파악하게 해준다. 그러나 모든 프로젝트에서 모든 다이어그램을 먼저 완성하는 것이 목적은 아니다. 요구사항, domain, 실행 순서, 상태, 시스템 경계와 data flow 중 현재 가장 불확실한 대상을 골라 적합한 표현을 사용하는 것이 핵심이다. 이 문서는 요구사항 분석부터 UML·C4·ERD·API 설계까지의 학습 순서와 React에서 유용한 User Flow·Component Tree·State Machine·FE–BE Sequence Diagram을 연결한다.

## 다이어그램은 정답 문서가 아니라 사고 도구다

다이어그램은 코드를 대신하는 완성 설계도가 아니다. 다음 질문에 빠르게 답하기 위한 압축된 mental model이다.

- 어떤 책임과 개념이 존재하는가?
- 서로 어떤 관계와 dependency를 가지는가?
- 요청 하나가 어떤 순서로 실행되는가?
- 상태는 어떤 사건에 의해 변하는가?
- Frontend, Backend, database와 외부 system의 경계는 어디인가?
- 변경이 발생하면 어느 module과 계약에 영향을 주는가?

Spring 입문 예제에서 요구사항과 diagram을 먼저 보여주는 것은 실제 설계 과정이면서 학습자가 `Controller → Service → Repository` 구조를 code보다 먼저 이해하도록 돕는 교육 장치이기도 하다. 실무에서는 diagram을 많이 만드는 것보다 중요한 불확실성을 줄이는 diagram을 선택하고 code 변화에 맞춰 필요한 만큼 유지하는 편이 낫다.

## 질문에 따라 다이어그램을 선택한다

| 알고 싶은 것 | 적합한 표현 |
|---|---|
| 사용자는 무엇을 하는가? | User Story, Use Case, User Flow |
| 비즈니스 개념과 관계는 무엇인가? | Domain Model, UML Class Diagram |
| 특정 시점의 실제 객체는 어떻게 조립되는가? | Object Diagram, Dependency Graph |
| 요청 하나는 어떤 순서로 실행되는가? | Sequence Diagram |
| 객체나 화면의 상태는 어떻게 변하는가? | State Machine, State Diagram |
| 전체 system의 경계는 어디인가? | C4 Model |
| Database에는 어떤 data가 저장되는가? | ERD |
| 화면과 정보는 어떻게 구성되는가? | Information Architecture, Component Tree |
| Data가 Frontend와 Backend를 어떻게 통과하는가? | Data Flow Diagram, Sequence Diagram |
| 왜 이 설계를 선택했는가? | ADR(Architecture Decision Record) |

Class diagram 하나로 모든 관점을 표현하려 하면 관계가 지나치게 많아지고 실제 실행 흐름이나 사용자 경험은 오히려 감춰질 수 있다.

## 1단계: 요구사항 분석

Code structure를 결정하기 전에 해결할 문제와 기대 behavior를 명확히 한다.

### 학습 키워드

- `Requirements Engineering`
- `Functional Requirements`
- `Non-functional Requirements`
- `User Story`
- `Acceptance Criteria`
- `Use Case Modeling`
- `Business Rule`
- `Invariant`
- `Failure Scenario`

회원가입 기능이라면 다음과 같이 정상·실패 동작과 유지해야 할 규칙부터 표현한다.

```text
사용자는 이름으로 회원가입할 수 있다.
동일한 이름의 회원은 가입할 수 없다.
가입에 성공하면 회원 식별자가 생성된다.
```

이 단계에서는 `MemberService`나 `MemberRepository` 같은 구현 구조를 먼저 확정하지 않아도 된다. 다음 질문을 통해 요구사항의 빈칸을 찾는다.

- 누구의 어떤 문제를 해결하는가?
- 정상 동작과 failure mode는 무엇인가?
- 범위에 포함되는 것과 제외되는 것은 무엇인가?
- 항상 유지해야 하는 invariant는 무엇인가?
- 어떤 trade-off와 미결정 사항이 있는가?

Spring 회원 관리 예제도 요구사항 정리 후 Domain·Repository 구현, Repository test, Service 구현과 Service test로 진행한다.

## 2단계: Domain Modeling

요구사항에서 핵심 명사, 행위, 책임과 규칙을 찾는다.

### 학습 키워드

- `Domain Modeling`
- `Domain-Driven Design Basics`
- `Ubiquitous Language`
- `Entity`
- `Value Object`
- `Aggregate`
- `Domain Service`
- `Anemic Domain Model`
- `Responsibility-driven Design`

처음부터 DDD 전체 pattern을 적용하기보다 다음 질문으로 시작한다.

```text
핵심 개념은 무엇인가?
각 개념은 어떤 data를 가지는가?
어떤 행위를 누가 책임지는가?
항상 지켜야 하는 business rule은 무엇인가?
```

회원 예제에서는 다음 정도의 model을 만들 수 있다.

```text
Member
- id
- name

MemberService
- 회원 가입
- 중복 회원 검사

MemberRepository
- 저장
- 조회
```

중요한 것은 class 이름을 빨리 만드는 것이 아니라 기획자, domain expert, Backend와 Frontend가 같은 용어를 같은 의미로 사용하는 것이다. Ubiquitous Language는 대화, diagram, API와 code의 이름을 연결한다.

## 3단계: 핵심 UML

UML 전체 표기법을 암기하기보다 Class, Sequence와 State Diagram을 먼저 익히고 Object Diagram은 DI와 runtime object graph를 이해하는 보조 도구로 사용한다.

### Class Diagram

학습 키워드:

- `UML Class Diagram`
- `Association Aggregation Composition`
- `Dependency Generalization Realization`
- `Multiplicity`

Class Diagram은 정적인 type, 책임과 관계를 보여준다.

```text
MemberService
    │ depends on
    ▼
MemberRepository
    △
    │ implements
MemoryMemberRepository
```

### Sequence Diagram

학습 키워드:

- `UML Sequence Diagram`
- `Message Lifeline Activation`
- `Synchronous Asynchronous Message`

Sequence Diagram은 요청이 실제로 통과하는 실행 순서를 보여준다.

```text
Browser
→ MemberController
→ MemberService
→ MemberRepository
→ Database
```

Class Diagram이 “누가 연결되어 있는가?”를 보여준다면 Sequence Diagram은 “무엇이 어떤 순서로 호출되는가?”를 보여준다. 기존 code를 이해할 때는 Sequence Diagram이 더 직접적인 경우도 많다.

### State Diagram

학습 키워드:

- `UML State Machine Diagram`
- `State Transition`
- `Finite State Machine`

주문, 결제, 가입 승인, upload처럼 상태 전이가 중요한 기능을 표현한다.

```text
Draft
→ Submitted
→ Approved
→ Completed
        └→ Cancelled
```

### Object Diagram

학습 키워드:

- `UML Object Diagram`
- `Runtime Object Graph`
- `Dependency Object Graph`

Object Diagram은 특정 시점에 실제 instance가 어떻게 연결됐는지 보여준다.

```text
memberService
    │
    ▼
memoryMemberRepository
```

DI를 처음 배울 때 interface와 실제 구현의 연결을 이해하는 데 유용하다. 지속적인 프로젝트 문서에서는 전체 structure를 설명하는 C4·Class Diagram이나 behavior를 설명하는 Sequence Diagram보다 사용 범위가 좁을 수 있다.

## 4단계: Architecture Modeling

Class보다 큰 system과 module boundary를 다룬다.

### 학습 키워드

- `Software Architecture Fundamentals`
- `C4 Model`
- `System Context Diagram`
- `Container Diagram`
- `Component Diagram`
- `Layered Architecture`
- `Hexagonal Architecture`
- `Clean Architecture`
- `Module Boundary`
- `Coupling Cohesion`

C4 Model은 system을 확대 수준별로 나누어 본다.

```text
System Context
→ 어떤 사용자와 외부 system이 있는가?

Container
→ Frontend, Backend, database는 어떻게 나뉘는가?

Component
→ 각 container 내부의 주요 module은 무엇인가?
```

Code를 읽기 전에 전체 구조를 파악하려는 목적이라면 모든 class를 그리는 것보다 Context·Container Diagram이 더 빠른 mental map을 제공할 수 있다. Architecture를 folder template으로 보지 말고 변경 가능성, 독립 test, dependency 방향과 되돌리기 비용을 설명하는 도구로 사용한다.

## 5단계: Data와 API 설계

Domain object, 저장 model과 network contract를 구분한다.

### 학습 키워드

- `Data Modeling`
- `Conceptual Logical Physical Data Model`
- `Entity Relationship Diagram`
- `Database Normalization`
- `API Design`
- `REST Resource Modeling`
- `OpenAPI Specification`
- `API Contract`
- `Error Response Design`

각 표현의 목적은 다르다.

```text
Class Diagram
→ code와 domain 책임

ERD
→ 저장할 data와 관계

API Schema
→ system 경계를 통과하는 data contract
```

이 셋을 하나의 diagram으로 억지로 합치지 않는다. Domain model의 개념이 database table이나 API DTO와 항상 일대일로 대응해야 하는 것은 아니다.

## 6단계: 설계 결정 기록

Diagram은 구조를 보여주지만 그 구조를 선택한 이유까지 설명하지 못할 수 있다.

### 학습 키워드

- `Architecture Decision Record`
- `ADR`
- `Design Document`
- `Technical Specification`
- `RFC`
- `Trade-off Analysis`

짧은 ADR 예시는 다음과 같다.

```text
결정
Service가 Repository interface에 의존한다.

이유
- 저장 기술이 아직 결정되지 않았다.
- Memory 구현으로 먼저 개발해야 한다.
- 이후 JDBC나 JPA로 교체할 가능성이 있다.

대가
- Interface와 조립 code가 추가된다.
```

결정, 이유, 대안과 비용을 함께 기록하면 나중에 구조만 보고 당시 판단을 추측하는 일을 줄일 수 있다.

## React와 Frontend에서도 다이어그램을 사용한다

Frontend에서도 요구사항과 설계를 시각화한다. 다만 React는 class hierarchy보다 사용자 흐름, component 합성, state ownership과 FE–BE data flow가 중요하므로 Class Diagram의 비중은 상대적으로 낮고 다른 표현이 더 유용할 수 있다.

### User Flow와 Screen Flow

```text
상품 목록
→ 상품 상세
→ 장바구니
→ 주문서
→ 결제
→ 완료
```

학습 키워드:

- `User Flow Diagram`
- `Screen Flow`
- `Customer Journey`
- `Information Architecture`
- `Site Map`

기획, 디자인과 Frontend가 사용자의 목적과 이동 경로를 합의할 때 사용한다.

### Component Tree

```text
CheckoutPage
├── OrderSummary
├── ShippingForm
├── PaymentMethodSelector
└── PaymentActions
```

학습 키워드:

- `React Component Architecture`
- `Component Tree`
- `Component Boundary`
- `Container Presentational Pattern`
- `Composition Pattern`

Component Tree는 화면의 조립 구조를 보여주지만 data flow나 runtime 호출 순서까지 설명하지는 않는다.

### State Ownership Diagram

```text
CheckoutPage
├── order state
├── shipping state
└── payment state
      ↓
PaymentMethodSelector
PaymentActions
```

학습 키워드:

- `React State Ownership`
- `Lifting State Up`
- `Derived State`
- `Server State vs Client State`
- `State Colocation`
- `State Machine React`

React에서는 어떤 class가 method를 가지는가보다 어떤 component가 state를 소유하고 어디까지 전달하는가가 중요한 설계 질문이다.

### FE–BE Sequence Diagram

회원가입 흐름을 전체 system으로 연결할 수 있다.

```text
User
→ SignUpForm
→ useSignUp
→ UserRepository
→ HTTP API
→ Spring Controller
→ MemberService
→ MemberRepository
→ Database
```

API 성공·실패, loading, retry와 화면 갱신까지 표시하면 Backend class 관계만 보거나 Frontend component tree만 보는 것보다 전체 실행 경로를 이해하기 쉽다.

### Frontend State Machine

복잡한 form, 결제, 인증과 upload 과정에 유용하다.

```text
idle
→ validating
→ submitting
├── success
└── error
    └── retry → submitting
```

`isLoading`, `isSuccess`, `isError`, `isRetrying`, `isCancelled` 같은 boolean state의 가능한 조합을 설명하기 어려워질 때 명시적인 State Machine을 검토한다.

### Full-stack Data Flow

```text
Admin Form
→ API Request
→ Spring Backend
→ Database
→ Public API
→ React Query
→ Public UI
```

이 diagram은 Frontend만이 아니라 data가 전체 system을 통과하는 경로와 변경 영향 범위를 보여준다. Wiki의 기존 Brownfield 설계 사례도 Admin form model부터 Public rendering까지의 data path를 먼저 드러내는 방식을 사용한다.

## 추천 학습 순서

다음 순서로 진행하면 UML 표기법 암기에 매몰되지 않고 요구사항에서 code까지 연결할 수 있다.

```text
요구사항과 Acceptance Criteria
→ Use Case
→ Domain Model
→ Class Diagram
→ Sequence Diagram
→ State Diagram
→ C4 Model
→ ERD와 API Contract
→ ADR
```

React를 함께 공부할 때 다음을 추가한다.

```text
User Flow
→ Information Architecture
→ Component Tree
→ State Ownership
→ FE–BE Sequence Diagram
→ 복잡한 기능의 State Machine
```

## 다이어그램 선택 규칙

기능 하나마다 모든 diagram을 만드는 것은 설계가 아니라 문서 작업이 될 수 있다. 막힌 질문에 따라 선택한다.

- 구조와 책임이 헷갈리면 Class·Component Diagram
- 호출 순서가 헷갈리면 Sequence Diagram
- 상태 조합이 헷갈리면 State Diagram
- System 경계가 헷갈리면 C4 Diagram
- 사용자 이동이 헷갈리면 User Flow
- 저장 구조가 헷갈리면 ERD
- FE와 BE 사이의 contract가 헷갈리면 OpenAPI·API Schema
- 선택 이유가 사라질 것 같으면 ADR

Diagram은 code와 별개의 진실이 되지 않도록 구현과 대조한다. 모든 세부 class를 유지하기보다 public contract, 중요한 dependency와 위험한 실행 흐름처럼 오래 유효한 정보를 우선한다.

## 첫 실습 과제

현재 배우는 Spring 회원 관리 예제로 다음 세 장을 직접 만든다.

### 1. Domain·Class Diagram

`Member`, `MemberService`, `MemberRepository`, `MemoryMemberRepository`의 책임과 관계를 표현한다.

### 2. 회원가입 Sequence Diagram

Browser 또는 test에서 시작해 Controller, Service, Repository와 저장소까지의 호출 순서를 표현한다. 중복 회원인 실패 흐름도 함께 표시한다.

### 3. React–Spring Full-stack Data Flow

React 회원가입 form부터 HTTP API, Spring Controller·Service·Repository와 database를 거쳐 성공·실패 UI가 갱신되는 흐름을 그린다.

각 diagram을 만든 뒤 실제 code와 비교하며 다음을 확인한다.

- Diagram에 없는 dependency가 code에 숨어 있지 않은가?
- 요구사항의 business rule을 누가 책임지는가?
- Runtime 구현은 어디에서 조립되는가?
- Error와 state transition이 빠지지 않았는가?
- Diagram이 너무 상세해 code를 복제하고 있지는 않은가?

이 실습은 diagram이 code 전에 구조를 보여주는 장점과 code가 변하면 diagram도 오래된 설명이 될 수 있다는 한계를 함께 학습하게 한다.

## See Also

- [Spring 회원 관리 백엔드와 테스트](../spring/spring-member-backend-and-testing.md)
- [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md)
- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [Frontend OCP·DIP와 React 설계 패러다임 통합 가이드](frontend-ocp-dip-and-react-software-engineering-guide-2026-08-24.md)
- [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](../ai-agents/superpowers-brownfield-practice-workflow-initial-design.md)
