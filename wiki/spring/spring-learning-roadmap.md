# 스프링 입문 학습 로드맵

> Sources: 김영한, 2026-01-30
> Raw: [스프링 입문 첫 강의](../../raw/spring/spring-introduction-course-overview.md); [프로젝트 환경 설정: 스프링 부트 프로젝트 생성](../../raw/spring/project-setup-and-spring-initializr.md); [강의 전체 참고 PDF companion](../../raw/spring/spring-course-overview.md); [다음으로 PDF companion](../../raw/spring/spring-next-steps.md)
> Updated: 2026-08-12

## Overview

스프링 입문 학습의 출발점은 IoC, DI, AOP 같은 이론을 먼저 깊게 파는 것이 아니라, 실제로 동작하는 웹 애플리케이션을 직접 완성하며 기술이 쓰이는 위치와 이유를 파악하는 것이다. 입문 단계에서 전체 개발 사이클을 경험해 큰 그림을 만든 뒤, 핵심 원리와 웹·데이터 접근 기술을 차례로 깊게 학습하는 흐름을 권한다.

## 실습으로 먼저 전체 사이클 익히기

입문 강의는 간단한 웹 애플리케이션을 처음부터 끝까지 만들어 보는 방식으로 구성된다. 학습자는 다음 흐름을 직접 코딩하고 실행한다.

1. [Spring Initializr로 스프링 프로젝트 생성](spring-boot-project-setup.md)
2. 스프링 부트로 웹 서버 실행
3. [회원 도메인과 Repository·Service 작성 및 테스트](spring-member-backend-and-testing.md)
4. [스프링 웹 MVC로 회원 등록·조회 개발](spring-member-web-mvc.md)
5. [JDBC, JPA, Spring Data JPA를 이용한 데이터베이스 연동](spring-database-access-technologies.md)
6. 테스트 케이스 작성
7. [AOP로 공통 관심사 분리](spring-aop-cross-cutting-concerns.md)

이 순환을 먼저 경험하면 각각의 기술이 애플리케이션의 어느 지점에서 사용되는지 파악할 수 있고, 이후에 깊게 공부할 영역도 판단하기 쉬워진다.

## 학습 원칙

핵심 학습 방법은 강의의 코드를 처음부터 끝까지 직접 작성하는 것이다. 이론은 구현 과정 중 필요한 시점에 그림과 함께 소개되며, 기술 자체를 암기하기보다는 실제 애플리케이션에서 어떻게 사용하는지 이해하는 데 초점을 둔다.

강의 범위는 실무에서 주로 사용하는 기술을 중심으로 좁힌다. 오래되었거나 사용 빈도가 낮은 기술을 초반부터 모두 다루기보다, 실제 개발에 필요한 기술로 먼저 작동 경험과 학습 방향을 확보하는 접근이다.

## 완전 정복 로드맵의 진행 방향

입문 과정에서 스프링 전반의 큰 그림을 코드로 익힌 뒤에는 다음 영역으로 학습을 확장한다.

- 스프링 핵심 원리: 입문에서 사용한 기능이 어떤 원리로 동작하고 활용되는지 깊게 이해한다.
- 웹 MVC: 웹 개발 전반과 스프링의 웹 기술을 학습한다.
- DB 데이터 접근 기술: 스프링으로 데이터베이스에 접근하는 여러 방식과 실무의 선택 기준을 학습한다.
- 스프링 부트: 방대한 기능 중 실무에서 주로 사용하는 부분과 프로젝트 적용 방식을 학습한다.

전체 전략은 사용 경험에서 출발해 원리와 전문 영역으로 내려가는 것이다. 먼저 작동하는 시스템을 만들어 맥락을 확보하면, 뒤이어 배우는 핵심 이론을 실제 사용처와 연결해서 이해할 수 있다.

강의 마지막 참고 자료는 다음 학습 경로로 [스프링 완전 정복 로드맵](https://www.inflearn.com/roadmaps/373)과 [스프링 부트와 JPA 실무 완전 정복 로드맵](https://www.inflearn.com/roadmaps/149)을 제시한다. 입문에서 전체 흐름을 경험한 뒤 핵심 원리와 실무 JPA로 확장하는 순서다.

## See Also

- [Spring Boot 프로젝트 생성과 첫 실행](spring-boot-project-setup.md)
- [Spring 회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Spring 회원 관리 웹 MVC](spring-member-web-mvc.md)
- [Spring DB 접근 기술 비교](spring-database-access-technologies.md)
- [Spring AOP와 공통 관심사 분리](spring-aop-cross-cutting-concerns.md)
