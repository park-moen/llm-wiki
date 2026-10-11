# 기본편 회원·주문 도메인 설계

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [2. 스프링 핵심 원리 이해1 - 예제 만들기 PDF companion](../../raw/spring/spring-core-member-order-example.md)
> Updated: 2026-10-11

## Overview

기본편 예제는 저장 방식과 할인 정책이 확정되지 않은 상황에서 역할을 인터페이스로 분리하고 순수 Java로 회원 가입·조회와 주문 생성을 구현한다. Spring Boot는 프로젝트 준비에 쓰지만, 초기 도메인 구현은 Spring container 없이 실행한다. 역할과 구현을 분리했더라도 서비스 안에서 구체 구현을 직접 생성하면 변경 의존성은 남는다.

## 입문편과 구분할 모델과 API

| 항목 | 기본편의 계약 |
|---|---|
| `Member` | `id`, `name`, `grade`를 가지며 `Grade`는 BASIC과 VIP |
| `MemberRepository` | `void save(Member member)`, `Member findById(Long memberId)` |
| `MemberService` | `void join(Member member)`, `Member findMember(Long memberId)` |
| `OrderService` | `Order createOrder(Long memberId, String itemName, int itemPrice)` |
| `DiscountPolicy` | 회원과 주문 금액을 받아 할인 금액 반환 |
| `Order` | 회원·상품·주문 금액·할인 금액을 보관하고 `calculatePrice()`로 결제 금액 계산 |

입문편의 [회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)는 별도 예제다. 그 문서의 `Optional` 반환, 중복 이름 검증, 저장 시 ID 생성 계약을 기본편에 그대로 적용하지 않는다.

## 주문 생성의 책임 분리

주문 서비스는 회원 ID로 저장소에서 회원을 조회하고, 할인 계산을 `DiscountPolicy`에 위임한 뒤 할인 결과를 담은 `Order`를 반환한다. 결제 금액 계산은 `itemPrice - discountPrice`다. 예제는 복잡도를 줄이기 위해 주문 데이터를 DB에 저장하는 단계는 생략한다.

초기의 `FixDiscountPolicy`는 VIP에게 1000원을 할인하고 BASIC에게는 할인을 적용하지 않는다. 할인 계산을 정책 객체에 맡기므로 주문 서비스는 구체적인 할인 계산식을 몰라도 된다.

## 초기 구현에 남은 의존성

`MemoryMemberRepository`는 static `HashMap`에 회원을 저장한다. PDF는 동시성 문제가 생길 수 있으며 `ConcurrentHashMap`을 사용할 수 있다고 설명한다. 메모리 저장소는 아직 확정되지 않은 실제 저장 방식 대신 사용하는 실습 구현이다.

초기 `MemberServiceImpl`은 `new MemoryMemberRepository()`를, `OrderServiceImpl`은 저장소와 `new FixDiscountPolicy()`를 내부에서 생성한다. 필드 타입을 인터페이스로 선언해도 구현 클래스에 대한 의존성은 남는다. PDF가 설계상 문제를 질문하는 지점이며, 다음 [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)에서 생성·연결 책임을 분리한다.

## 실행과 테스트

`MemberApp`, `OrderApp`의 main으로 흐름을 확인한 뒤 JUnit 테스트로 옮긴다. 회원 가입 테스트는 저장한 회원과 조회한 회원이 같은지, 주문 테스트는 VIP 주문의 할인 금액이 맞는지 AssertJ로 검증한다. 전체 주문 흐름 검증과 [할인 계산만 검증하는 단위 테스트](discount-policy-extension-and-testing.md)는 범위가 다르다.

## 실습 환경과 자료 내부의 오래된 예시

이 자료의 프로젝트 안내는 Java 17 이상, Gradle - Groovy, Java 언어와 Jar 패키징을 사용하며 추가 의존성을 선택하지 않는다. Spring Boot 3.2 관련 안내에서는 IDE의 build·test 실행을 Gradle로 설정하도록 한다.

반면 본문의 `build.gradle` 예시는 `2.3.3.RELEASE`와 `sourceCompatibility = '11'`을 담고 있어 같은 자료의 새 환경 안내와 다르다.

> **Status: Outdated** (2023-12-04)
> 위 Gradle 예시는 이전 환경을 보여준다. 같은 PDF의 업데이트된 안내는 Spring Boot 3.0 이상과 Java 17 이상을 요구한다. 예전 설정을 전체 복사하기보다 해당 자료의 환경 안내와 구분한다.

자료는 JPA, lifecycle, validation 관련 `javax` 패키지를 `jakarta`로 바꾼 예를 보여준다. 이 설명은 PDF에 제시된 API의 전환 안내로 읽으며 모든 Java 패키지가 바뀐다는 규칙으로 확대하지 않는다. 현재 설치 권장 버전은 이 자료만으로 판단하지 않는다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [할인 정책 확장과 단위 테스트](discount-policy-extension-and-testing.md)
- [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)
