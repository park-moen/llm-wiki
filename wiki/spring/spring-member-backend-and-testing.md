# Spring 회원 관리 백엔드와 테스트

> Sources: 김영한, 2026-01-30
> Raw: [회원 관리 예제 - 백엔드 개발 PDF companion](../../raw/spring/spring-member-backend-development.md); [회원 관리 예제 - 비즈니스 요구사항 강의](../../raw/spring/spring-member-management-requirements-lecture.md); [회원 Domain과 Repository 구현 강의](../../raw/spring/spring-member-domain-repository-implementation-lecture.md); [회원 Repository 테스트 케이스 강의](../../raw/spring/spring-member-repository-test-lecture.md); [회원 Service 구현 강의](../../raw/spring/spring-member-service-implementation-lecture.md); [회원 Service 테스트 강의](../../raw/spring/spring-member-service-test-lecture.md)
> Updated: 2026-08-13

## Overview

입문 회원 관리 예제는 Domain, Repository, Service를 분리하고 test로 각 계층의 책임을 확인한다. Repository interface 덕분에 저장 방식과 business logic을 분리할 수 있고, constructor injection으로 Service와 test가 같은 Repository instance를 사용한다. Test는 실행 순서나 이전 data에 의존하지 않도록 매번 독립된 fixture를 구성하고 저장소를 정리해야 한다.

## 최소 비즈니스 요구사항

입문 예제는 Spring 생태계의 전체 개발 흐름을 익히는 데 집중하기 위해 business 범위를 의도적으로 작게 잡는다.

- 회원 data: `id`, `name`
- 기능: 회원 등록, 회원 조회
- 개발 순서: 요구사항 정리 → Domain·Repository 구현 → Repository test → Service 구현 → Service test
- Test framework: JUnit

회원·주문·상품이 복잡하게 연결되는 실무형 domain을 다루기보다, 단순한 요구사항을 통해 계층 분리와 저장 기술 교체 과정을 관찰하는 예제다.

## 계층과 책임

요청 흐름은 Controller에서 Service, Repository, DB로 이어진다. Domain 객체는 이 계층 사이에서 business data를 표현한다.

- `Member`: `id`, `name`을 가진 domain 객체
- `MemberRepository`: `save`, `findById`, `findByName`, `findAll`이라는 저장·조회 계약
- `MemoryMemberRepository`: memory `Map`과 sequence를 사용하는 임시 구현
- `MemberService`: 회원 가입, 중복 회원 검사와 회원 조회 같은 business 기능

Service는 저장 방식이 아니라 Repository interface에 의존한다. 이 분리는 이후 memory 구현을 JDBC·JPA 구현으로 바꿀 수 있는 기반이 된다.

## 저장 기술이 정해지지 않은 설계 시나리오

강의는 개발을 시작해야 하지만 data 저장소가 아직 선정되지 않은 상황을 가정한다. 후보는 관계형 database와 NoSQL이며, 구체적인 접근 기술도 JDBC, MyBatis, JPA 등에서 나중에 결정될 수 있다.

이 불확실성을 business logic 안에 직접 넣지 않고 `MemberRepository` interface 뒤로 격리한다. 초기에는 빠르게 만들 수 있는 memory 구현체를 사용하고, 저장 기술이 정해지면 interface 구현체를 교체한다. 따라서 `MemberService`는 현재 저장 방식이 memory인지 database인지 알 필요 없이 Repository 계약에만 의존한다.

이 시나리오는 interface가 단순한 문법 요소가 아니라 변경 가능성이 큰 외부 저장 방식과 application의 핵심 logic 사이에 경계를 만드는 수단임을 보여준다.

## Member Domain과 Repository 계약

`Member`는 `id`와 `name`을 가진다. Name은 회원 가입 시 사용자가 입력하지만 id는 사용자가 정하는 값이 아니라 저장 시 system이 생성해 data를 식별하는 값이다. 입문 예제는 이해하기 쉬운 구현을 위해 두 field에 Getter와 Setter를 둔다.

`MemberRepository`의 네 method는 다음 계약을 표현한다.

- `save`: system id를 설정해 회원을 저장하고 저장된 회원을 반환한다.
- `findById`: id로 회원을 조회해 `Optional<Member>`를 반환한다.
- `findByName`: name으로 회원을 조회해 `Optional<Member>`를 반환한다.
- `findAll`: 저장된 모든 회원을 `List<Member>`로 반환한다.

조회 결과가 없을 수 있는 method는 null을 직접 반환하는 대신 Java 8의 `Optional`로 감싼다. Memory 구현의 `findById`는 `Optional.ofNullable(store.get(id))`를 사용하고, `findByName`은 `store.values()`를 filter한 뒤 `findAny()` 결과를 반환한다.

## Memory Repository 구현 흐름

Memory 저장소는 `Map<Long, Member>`와 증가하는 `sequence`로 구성된다.

1. `save`가 sequence를 증가시켜 Member에 system id를 설정한다.
2. `store.put(member.getId(), member)`로 Map에 저장한다.
3. `findById`는 Map key로 조회한다.
4. `findByName`은 Map value를 순회하며 이름이 같은 회원을 찾는다.
5. `findAll`은 `store.values()`를 `ArrayList`로 변환한다.

## Memory Repository의 한계

입문 예제의 `HashMap`과 단순 `long sequence`는 구조를 학습하기 위한 구현이다. 여러 thread가 동시에 접근하는 실무 환경에서는 thread-safe하지 않으므로 `ConcurrentHashMap`, `AtomicLong` 같은 동시성 대응이 필요하다. 이는 DB로 교체하기 전 memory 구현 자체의 제약이다.

## Repository 테스트를 독립시키기

Main method나 Web Controller를 통해 기능을 직접 확인하면 준비와 반복 실행에 시간이 들고 여러 test를 한꺼번에 실행하기 어렵다. JUnit은 검증 code 자체를 반복 실행하고 여러 test 결과를 함께 확인할 수 있게 한다.

Test source는 main source가 아닌 test 영역에서 같은 package 구조를 사용하고, 대상 class 이름 뒤에 `Test`를 붙이는 convention을 따른다. 예제의 `MemoryMemberRepositoryTest`는 외부에서 사용할 class가 아니므로 `public`일 필요가 없다.

### Repository test 범위

- `save`: 저장한 Member를 id로 다시 조회했을 때 같은 객체인지 검증한다.
- `findByName`: `spring1`, `spring2` 회원을 저장하고 `spring1` 조회 결과가 첫 번째 Member인지 검증한다.
- `findAll`: 회원을 둘 저장한 뒤 결과 collection의 size가 2인지 검증한다.

`Optional.get()`은 일반 application code에서 바로 사용하는 좋은 방식은 아니지만, 값이 반드시 존재하도록 준비한 이 test 예제에서는 조회 결과를 꺼내는 용도로 사용한다.

### Assertion 선택

JUnit Jupiter의 `Assertions.assertEquals`로 expected와 actual을 비교할 수 있다. 강의는 이후 읽기 쉬운 AssertJ의 다음 형태로 전환한다.

```java
assertThat(result).isEqualTo(member);
```

AssertJ assertion을 static import하면 class 이름 없이 `assertThat()`부터 작성할 수 있다. 기대와 실제가 다르면 test는 실패하고 IDE에 빨간 표시가 나타나며, 일치하면 녹색으로 통과한다.

### Test 독립성과 상태 정리

Repository test에서 가장 중요한 규칙은 test 사이에 저장소 상태를 공유하지 않는 것이다.

JUnit은 test method 실행 순서를 보장하지 않는다. 앞선 test의 data가 남아 있으면 개별 test는 통과해도 전체 실행은 실패할 수 있다. `@AfterEach`에서 `clearStore()`를 호출해 매 test 뒤 상태를 초기화하면 이 순서 의존성을 제거할 수 있다.

각 test는 단독 실행과 전체 실행에서 같은 결과를 내야 한다. 이를 위해 하나의 test가 끝날 때 memory 저장소 같은 공용 상태를 비워 다음 test가 깨끗한 상태에서 시작하게 한다.

### 구현 후 테스트와 TDD

이 강의는 `MemoryMemberRepository`를 먼저 구현하고 나중에 test를 작성했으므로 TDD가 아니다. 반대로 검증할 test를 먼저 작성하고 그 test가 통과하도록 구현 class를 만드는 접근을 테스트 주도 개발(TDD)이라고 설명한다.

Test 수가 많아지면 IDE의 class 단위 실행이나 Gradle test task로 함께 실행할 수 있다. Build 과정과 test를 연결하면 실패한 test가 있을 때 다음 단계로 진행하지 않도록 할 수 있다.

## IntelliJ 단축키

이번 구현 강의에서 확인되는 IntelliJ 조작은 다음과 같다.

- Interface method 구현: macOS `Option + Enter`로 intention menu를 열고 `Implement Methods`를 선택한다.
- Import 보조: 강의에서는 `Ctrl + Space` 또는 macOS `Option + Enter`를 사용한다.
- 현재 구문 완성: macOS `Command + Shift + Enter`를 사용한다.
- AssertJ static import: macOS `Option + Enter`로 static import option을 선택한다.
- Rename refactoring: 강의 표기는 `Shift + F6`이며, 동일한 이름의 참조를 함께 변경한다.
- Return value를 local variable로 추출: macOS `Command + Option + V`를 사용한다.
- Refactor menu: 강의 표기는 `Ctrl + T`다.
- Extract Method: macOS `Command + Option + M`을 사용한다.
- Test class 생성: macOS `Command + Shift + T`로 `Create New Test`를 열고 JUnit 5를 선택한다.
- 이전 실행 반복: macOS `Ctrl + R`, Windows `Shift + F10`을 사용한다.
- Windows·Linux의 Test class 생성 대응 key는 이번 raw에 제시되지 않으므로 추정하지 않는다. 현재 IntelliJ keymap에서 `Implement Methods`, `Complete Current Statement`, `Static Import`, `Rename`, `Introduce Variable`, `Refactor This`, `Extract Method`, `Create New Test`, `Run Context Configuration` action을 조회한다.

## Service의 business 규칙과 테스트

Service는 Domain과 Repository를 사용해 business use case를 구현한다. Repository가 `save`, `findById`, `findAll`처럼 data 접근 중심의 이름을 사용한다면 Service는 `join`, `findMembers`, `findOne`처럼 business 관계자가 이해하기 쉬운 이름을 사용한다.

### 회원 가입과 중복 검증

`MemberService.join`은 다음 순서로 동작한다.

1. `memberRepository.findByName(member.getName())`으로 같은 이름의 회원을 조회한다.
2. `Optional.ifPresent`로 값이 존재하는지 확인한다.
3. 중복이면 `IllegalStateException("이미 존재하는 회원입니다.")`을 발생시킨다.
4. 중복이 아니면 Repository에 저장하고 Member id를 반환한다.

중복 검증 code는 `validateDuplicateMember` method로 추출한다. 그러면 `join` method가 “중복 검증 후 저장”이라는 business 흐름을 직접 드러내고 세부 검증 logic은 별도 method에 감춰진다.

### Optional 사용

조회 결과가 없을 수 있을 때 Optional을 사용하면 null을 직접 비교하는 대신 `ifPresent` 같은 API로 존재 여부에 따른 logic을 표현할 수 있다. 강의는 `get()`으로 즉시 꺼내는 방식을 권장하지 않고, 필요한 경우 `orElseGet`처럼 값이 없을 때 대안을 제공하는 method도 소개한다.

중간 `Optional<Member>` variable이 꼭 필요하지 않으면 `findByName(...).ifPresent(...)`처럼 호출을 이어서 작성할 수 있다.

### 조회 기능과 Service 테스트

`findMembers`는 Repository의 `findAll`을, `findOne`은 `findById`를 위임해 반환한다.

Service test는 Given–When–Then 구조로 작성한다.

- Given: 검증에 필요한 Member와 같은 사전 조건을 준비한다.
- When: `join`처럼 검증 대상 business method를 실행한다.
- Then: 저장 결과나 exception을 assertion으로 확인한다.

이 구분은 test가 길어졌을 때 준비 data, 실행 대상과 검증부를 빠르게 파악하도록 돕는다. 모든 test에 항상 맞는 규칙은 아니므로 입문 단계의 기본 구조로 사용한 뒤 상황에 맞게 변형한다.

정상 가입 test는 `join`이 반환한 id로 회원을 다시 찾고 name이 같은지 확인한다. 하지만 회원 가입의 핵심 rule은 중복 이름을 막는 것이므로 정상 흐름만 검사하면 반쪽짜리 test다.

### 중복 회원 예외 검증

같은 name을 가진 두 Member를 준비하고 첫 회원을 가입시킨 뒤 두 번째 `join`에서 `IllegalStateException`이 발생해야 한다. Try–catch와 `fail()`로도 검사할 수 있지만 JUnit 5의 `assertThrows`가 의도를 더 직접적으로 표현한다.

```java
IllegalStateException e = assertThrows(
        IllegalStateException.class,
        () -> memberService.join(member2)
);
assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
```

Exception type뿐 아니라 message까지 검사하면 실제 중복 회원 rule 때문에 실패했는지 확인할 수 있다.

### Test fixture와 dependency injection

Service test는 `@BeforeEach`에서 새 `MemoryMemberRepository`를 만들고 그 instance를 `MemberService` constructor에 전달한다. Test와 Service가 서로 다른 Repository를 각각 `new`하면 현재 구현의 static store에서는 우연히 동작할 수 있어도 실제로는 다른 instance를 검사하게 된다.

Constructor로 외부의 Repository를 받게 하면 Service와 test가 같은 instance를 공유한다. Service가 dependency를 직접 생성하지 않고 외부에서 전달받는 이 방식을 dependency injection(DI)이라고 한다.

`@AfterEach`에서는 `repository.clearStore()`를 호출해 다음 test에 회원 data가 누적되지 않게 한다. `@BeforeEach`의 fixture 생성과 `@AfterEach`의 상태 정리를 함께 사용하면 각 test가 독립적으로 실행된다.

## See Also

- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Spring 회원 관리 웹 MVC](spring-member-web-mvc.md)
- [Spring DB 접근 기술 비교](spring-database-access-technologies.md)
- [스프링 입문 학습 로드맵](spring-learning-roadmap.md)
