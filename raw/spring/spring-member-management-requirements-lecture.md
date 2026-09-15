# 회원 관리 예제 - 비즈니스 요구사항 정리

> Source: Inflearn 김영한 「스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술」 강의 스크립트
> Collected: 2026-08-12
> Published: Unknown

네 이번 시간에는 이제 드디어 회원 관리 예제를 한번 만들어 보겠습니다.

먼저 비즈니스 요구사항을 정리할 거고요. 그다음에 회원 domain과 회원 domain 객체를 저장하고 불러올 수 있는 저장소라고 하는 Repository 객체를 만들 겁니다. 그리고 이 회원 Repository가 정상 동작하는지 test case를 만들고요. 실제 business logic이 있는 회원 Service를 만들고 이것이 정상적으로 동작하는지 test를 만들 겁니다. 여기서 test는 JUnit이라는 test framework로 만들 거예요.

먼저 이번 시간에는 비즈니스 요구사항을 정리할 건데요. 이 비즈니스 요구사항은 가장 쉬운 것으로 할 거예요. 정말 가장 단순한 것으로, data도 회원의 id와 이름밖에 없고요. 기능도 회원을 등록하고 조회하는 딱 두 가지 기능밖에 없어요.

왜냐하면 이 강의의 목표 자체가 복잡한 business를 하는 강의가 아니라 정말 단순한 예제를 가지고 Spring 생태계 전반적으로 어떤 식으로 개발이 일어나고 동작하는지를 알아보는 것이기 때문입니다. 그래서 정말 단순한 business를 할 겁니다.

만약에 복잡한 business를 가지고 실제 실무에서 회원도 있고 주문도 있고 상품도 있고 그렇게 복잡하게 엮이는 business에 대해서 공부하고 싶으면 활용 1편 강의를 들어보시면 됩니다.

추가로 아직 data 저장소가 선정되지 않았다는 가상 시나리오가 있습니다. 개발자는 개발을 해야 되는데 아직 DB가 선정되지 않은 거예요. 성능이 중요한 database로 할지, 일반적인 관계형 database로 할지, NoSQL로 할지 아직 정해지지 않은 상황입니다. 그런데 개발은 해야 되는 상황이라고 보면 될 것 같아요.

Spring의 특성을 더 잘 설명하기 위해 이런 가상의 시나리오를 만들었습니다. 이렇게 정말 단순한 시나리오로 시작하겠습니다.

일반적인 Web application의 계층 구조는 보통 Controller, Service, Repository, Domain 객체, Database로 구성됩니다.

Controller는 앞에서 본 Web MVC의 control 역할이나 API를 만들 때 Controller 역할을 합니다.

Service class에는 핵심 business logic이 들어갑니다. 예를 들어 회원은 중복 가입이 안 된다는 logic들이 Service 객체에 들어갑니다.

Domain은 회원, 주문, 쿠폰처럼 database에 주로 저장하고 관리되는 business domain 객체입니다. Service는 이 business domain 객체를 가지고 핵심 business logic이 동작하도록 구현한 객체라고 보면 됩니다.

그래서 Controller, Service, Repository, Domain으로 구성되어 있습니다. 저희도 이런 일반적인 계층형 구조를 따라갈 겁니다.

Class 의존관계를 보면 회원 business logic에는 `MemberService`가 있고요. 회원을 저장하는 `MemberRepository`는 interface로 설계할 거예요. 아직 data 저장소가 선정되지 않았기 때문입니다.

`MemberRepository` interface를 만들고 구현체는 우선 memory 구현체로 만들 거예요. 일단 개발은 해야 되니까 memory로 단순하게 저장하고 꺼낼 수 있는 구현체를 만듭니다. 향후 RDB로 할지 JPA로 할지 구체적인 기술이 선정되면 이것을 바꿔 끼울 거예요. 바꿔 끼우기 위해 interface를 정의합니다.

아직 data 저장소가 선정되지 않아서 interface를 통해 구현 class를 변경할 수 있도록 설계하고, data 저장소는 RDB, NoSQL 등 다양한 저장소를 고민 중인 상황으로 가정합니다. 개발을 진행하기 위해 초기 개발 단계에서는 가벼운 memory 기반 data 저장소를 사용합니다.

나중에 단순히 JDBC를 쓸지 MyBatis를 쓸지 JPA를 쓸지 등 저장 기술이 정해지면 바꿀 수 있다는 가정하에 설계했습니다.

대략적인 내용과 큰 그림은 이렇게 설명했고, 구체적인 code를 다음 시간부터 만들어 보겠습니다. 감사합니다.
