# Spring Data JPA 사용자 정의 Repository와 fragment

> Sources: Spring Data JPA, Unknown
> Raw: [Spring Data Custom Repository Implementations](../../raw/spring/spring-data-jpa-custom-repository-reference.md); [Spring Data Transactionality](../../raw/spring/spring-data-jpa-transactions-reference.md)
> Updated: 2026-10-15

## Overview

기본 CRUD나 조회 선언으로 필요한 동작을 표현하기 어렵다면 Repository에 사용자 정의 기능을 합성할 수 있다. **추가 기능 인터페이스, 구현 클래스, 이를 상속하는 Repository**를 나누는 fragment 방식이 공식 문서의 권장 모델이다. [Custom Repository Implementations](../../raw/spring/spring-data-jpa-custom-repository-reference.md)

## Kotlin 예제

[공통 Member](jpa-persistence-context-and-entity-lifecycle.md)를 조회하는 학습용 코드다. EntityManager를 직접 사용하는 구조를 보여 주기 위한 예제이며, 이 단순 조건 자체는 파생 메서드로도 표현할 수 있다.

```kotlin
interface MemberSearch {
    fun findActiveNamed(name: String): List<Member>
}

class MemberSearchImpl(
    private val entityManager: EntityManager
) : MemberSearch {
    override fun findActiveNamed(name: String): List<Member> =
        entityManager.createQuery(
            "select m from Member m where m.active = true and m.name = :name",
            Member::class.java
        ).setParameter("name", name).resultList
}

interface MemberRepository : JpaRepository<Member, Long>, MemberSearch
```

`MemberSearch`가 fragment 인터페이스이고 `MemberSearchImpl`이 구현이다. 기본 접미사는 `Impl`이며 설정으로 바꿀 수 있다. Spring Data가 탐색할 수 있는 Repository 관련 패키지에 배치한다. Service에서는 합성된 `MemberRepository`를 주입받아 추가 기능을 호출할 수 있다. [Custom Repository Implementations](../../raw/spring/spring-data-jpa-custom-repository-reference.md)

## Repository 이름 뒤에 Impl을 붙이면 되는가

과거의 단일 구현 탐색 방식은 최종 Repository 이름에 접미사를 붙이는 형태였다. 현재 공식 문서는 그 방식보다 fragment 단위 인터페이스와 구현을 합성하는 모델을 권장한다. 위 예제에서 구현 클래스 이름이 `MemberRepositoryImpl`이 아닌 이유다. [Custom Repository Implementations](../../raw/spring/spring-data-jpa-custom-repository-reference.md)

사용자 정의 구현은 일반 Spring Bean으로 다른 의존성을 주입받을 수 있다. 복잡한 조회를 위해 EntityManager나 별도 데이터 접근 도구를 사용할 자리지만, 이것이 모든 조회를 직접 구현해야 한다는 뜻은 아니다. 기존 기능이 요구사항을 충족하는지 먼저 확인한다. [Custom Repository Implementations](../../raw/spring/spring-data-jpa-custom-repository-reference.md)

## 트랜잭션과 구현 충돌

fragment를 추가했다는 사실만으로 모든 메서드에 원하는 트랜잭션 설정이 생기지 않는다. 여러 호출을 묶는 경계는 [Service 트랜잭션](spring-data-jpa-service-transactions.md)에서 명시한다. 트랜잭션이 선언된 fragment 메서드는 실제 구현의 속성을 따른다. [Spring Data Transactionality](../../raw/spring/spring-data-jpa-transactions-reference.md)

사용자 정의 구현은 기본 Repository 구현보다 우선할 수 있고, 여러 fragment가 같은 메서드 signature를 제공하면 선언 순서에 따른 충돌 해석도 생긴다. 입문 예제에서는 기본 `save()`를 재정의하지 않고 추가 기능만 분리한다. 뒤 문장은 우선순위 규칙에서 도출한 학습 범위 선택이다. [Custom Repository Implementations](../../raw/spring/spring-data-jpa-custom-repository-reference.md)

## 직접 작성하는 연습

위 세 타입을 Repository 탐색 범위에 두고 통합 테스트에서 사용자 정의 함수를 호출한다. 구현 클래스의 이름과 fragment 인터페이스 이름이 대응하는지 확인한다. 다음으로 복잡한 검색 한 가지를 추가하되 기본 CRUD 구현을 복사하지 않는다.

## See Also

- 이전: [동적 검색 조건](spring-data-jpa-dynamic-search.md)
- 다음: [Auditing](spring-data-jpa-auditing.md)
