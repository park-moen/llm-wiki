# 스프링 핵심 원리 기본편 학습 안내

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [0. 스프링 핵심 원리 - 기본편 v2023-12-04 PDF companion](../../raw/spring/spring-core-course-overview.md); [1. 객체 지향 설계와 스프링 PDF companion](../../raw/spring/spring-core-object-oriented-design-outline.md); [10. 다음으로 PDF companion](../../raw/spring/spring-core-next-steps.md); [객체 지향 설계와 스프링 슬라이드 PDF companion](../../raw/spring/spring-core-object-oriented-design-slides.md)
> Updated: 2026-10-11

## Overview

스프링 핵심 원리 기본편은 객체 지향 설계에서 출발해 Spring container가 객체를 생성하고 연결하며 관리하는 이유를 다룬다. 입문편의 웹·DB 기능 실습 다음에, 역할과 구현의 분리 및 Bean 관리 원리를 이해하는 과정이다. 이 Wiki에 보관한 자료는 사용자 제공 `spring-basic-v20231204` 폴더의 PDF와 이후 추가한 객체 지향 설계 슬라이드이며, 자료에 적힌 버전 안내와 현재 환경의 권장 설정을 구분한다.

## 자료의 학습 순서

| 학습 단계 | 연결 문서 |
|---|---|
| 객체 지향 설계와 Spring | [객체 지향 설계와 SOLID](spring-object-oriented-design-and-solid.md) |
| 회원·주문 예제 만들기 | [기본편 회원·주문 도메인 설계](spring-core-member-order-design.md) |
| 객체 지향 원리 적용 | [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md), [할인 정책 확장과 테스트](discount-policy-extension-and-testing.md) |
| container와 Bean | [Spring container와 Bean 조회](spring-container-bean-lookup.md) |
| singleton container | [Singleton과 Configuration](spring-singleton-and-configuration.md) |
| component scan | [Component scan의 범위와 충돌](spring-component-scan.md) |
| 자동 의존관계 주입 | [자동 주입과 Bean 선택](spring-autowiring-and-bean-selection.md) |
| Bean 생명주기 | [초기화와 종료 콜백](spring-bean-lifecycle.md) |
| Bean scope | [Scope와 Provider](spring-bean-scopes-and-provider.md) |

## 별도 슬라이드의 보완 상태

「객체 지향 설계와 스프링」 PDF에는 강의 목차와 별도 PPT 자료를 참고하라는 안내만 있다. 사용자 제공 폴더에는 해당 PPT가 없어 Spring의 탄생 배경이나 SOLID 전체 설명을 이 PDF의 본문처럼 작성하지 않는다. 뒤의 예제 자료에서 실제로 설명하는 SRP·DIP·OCP 적용은 별도 문서에 정리한다.

> **Status: Outdated** (2026-10-11)
> 위 문장은 최초 자료 폴더를 반영했을 때의 상태다. 이후 사용자가 별도로 제공한 「객체지향설계와스프링-225321.pdf」를 보관해 Spring의 탄생 배경, 역할·구현 분리와 SOLID 설명을 보완했다. 최초 폴더의 목차 PDF는 그대로 두고, 새 슬라이드는 별도 원본으로 연결한다.

현재 첫 단원의 본문은 [객체 지향 설계와 Spring의 SOLID 원칙](spring-object-oriented-design-and-solid.md)에서 읽을 수 있다.

## 자료 버전의 의미

전체 목차 PDF에는 `v2023-12-04` 수정 이력이 있으며 Spring Boot 3.2 사용 시 IntelliJ의 실행 설정에서 Gradle을 사용하라는 안내가 추가되어 있다. 이는 해당 버전 자료의 실습 안내다. 기본편과 이미 보관한 입문편의 수정 이력은 서로 다르므로, 자료의 버전을 섞어 현재 제품의 전체 지원 범위로 해석하지 않는다.

## 원리 학습 이후의 진행 방향

마무리 PDF는 객체 지향 설계, 역할과 구현의 분리, OCP·DIP와 DI container를 핵심 원리로, Bean 등록·singleton·scan·자동 주입·생명주기·scope를 핵심 기능으로 정리한다. 이 기반에서 웹 MVC, 데이터 접근과 Spring Boot를 배우는 흐름을 제시한다.

추천 코스는 입문 → 기본편 → Spring Boot와 JPA 실무 로드맵이다. 먼저 활용 예제를 경험하고 JPA 기본을 공부한 뒤 활용 예제를 복습하며 API·성능 최적화, Spring Data JPA, Querydsl로 이어가는 방법을 설명한다. 자료 속 강의의 오픈·예정 표기는 당시 안내이므로 현재 출시 상태로 해석하지 않는다.

## See Also

- [스프링 입문 학습 로드맵](spring-learning-roadmap.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [객체 지향 설계와 Spring의 SOLID 원칙](spring-object-oriented-design-and-solid.md)
