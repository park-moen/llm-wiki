# Spring Data JPA 동적 검색: Specification과 Query by Example

> Sources: Spring Data JPA, Unknown
> Raw: [Spring Data Specifications](../../raw/spring/spring-data-jpa-specifications-reference.md); [Spring Data Query by Example](../../raw/spring/spring-data-jpa-query-by-example-reference.md)
> Updated: 2026-10-15

## Overview

이름·이메일·활성 상태처럼 선택적인 검색 조건을 조합하면 메서드 이름만으로 모든 경우를 선언하기 어렵다. `Specification`은 Criteria API의 조건을 조합하고, Query by Example(QBE)은 예시 객체의 프로퍼티를 바탕으로 조건을 만든다. **복잡한 조건을 표현하는 능력과 간편한 입력 방식은 서로 다른 선택 기준**이다. [Specifications](../../raw/spring/spring-data-jpa-specifications-reference.md); [Query by Example](../../raw/spring/spring-data-jpa-query-by-example-reference.md)

## Specification으로 조건 조합하기

다음은 [공통 Member](jpa-persistence-context-and-entity-lifecycle.md)를 사용하는 학습용 예제다. 최신 문서의 새 인터페이스 대신 오래전부터 제공된 `Specification`을 사용한다.

```kotlin
interface MemberRepository : JpaRepository<Member, Long>,
    JpaSpecificationExecutor<Member>

fun activeOnly(): Specification<Member> =
    Specification { root, _, cb ->
        cb.isTrue(root.get<Boolean>("active"))
    }

fun named(name: String): Specification<Member> =
    Specification { root, _, cb ->
        cb.equal(root.get<String>("name"), name)
    }

fun search(name: String?): List<Member> {
    val spec = if (name.isNullOrBlank()) {
        activeOnly()
    } else {
        activeOnly().and(named(name))
    }
    return memberRepository.findAll(spec)
}
```

마지막 함수는 Repository를 주입받은 Service 본문의 예시다. 조건을 함수로 분리하면 필요할 때 `and`·`or`로 조합할 수 있다. 문자열로 프로퍼티 이름을 지정하는 부분은 오타를 compiler가 모두 검증하지 않으므로 테스트로 확인하거나 metamodel 기반 접근을 검토한다. [Specifications](../../raw/spring/spring-data-jpa-specifications-reference.md)

공식 문서에는 Spring Data JPA `4.0`에서 도입한 `PredicateSpecification`과 query 유형별 변형도 소개되어 있다. 이 학습 예제와 새 API를 섞어 복사하지 말고 프로젝트 버전이 지원하는 타입을 선택한다. [Specifications](../../raw/spring/spring-data-jpa-specifications-reference.md)

## QBE는 예시 객체에서 조건을 만든다

```kotlin
val probe = Member("민", "")
val matcher = ExampleMatcher.matchingAll()
    .withIgnorePaths("id", "email", "active")
    .withMatcher("name", ExampleMatcher.GenericPropertyMatchers.startsWith())

val result = memberRepository.findAll(Example.of(probe, matcher))
```

probe는 검색할 값의 예시 객체다. matcher는 어떤 프로퍼티를 무시하고 문자열을 어떻게 비교할지 지정한다. 공통 엔티티의 `email` 기본값과 `active`가 원하지 않는 조건에 포함되지 않도록 명시적으로 제외했다. probe 객체를 DB에 저장하는 예제가 아니다. [Query by Example](../../raw/spring/spring-data-jpa-query-by-example-reference.md)

## 어떤 방식이 맞는가

| 요구사항 | 먼저 검토할 방법 |
| --- | --- |
| 단순 값 비교·문자열 검색과 무시할 필드 지정 | QBE |
| 조건을 함수로 재사용하고 논리적으로 조합 | Specification |
| 복잡한 조인·집계·조회 형태 제어 | 사용자 정의 Repository 등 별도 구현 |

QBE는 임의의 중첩 논리 그룹이나 컬렉션·Map 매칭을 모두 표현하는 도구가 아니다. null 처리와 primitive 프로퍼티의 기본 포함 여부도 확인해야 한다. 복잡한 검색까지 “예시 객체 하나면 끝난다”고 일반화하지 않는다. [Query by Example](../../raw/spring/spring-data-jpa-query-by-example-reference.md)

## 직접 작성하는 연습

활성 회원 검색에 선택적인 이름 조건을 추가한다. 이름이 없을 때와 있을 때의 SQL을 비교한다. 같은 요구사항을 QBE로 표현한 뒤 기본값 때문에 원치 않는 조건이 생기는지 확인하고, 두 방식 중 선택 이유를 적는다.

## See Also

- 이전: [Service 트랜잭션 경계](spring-data-jpa-service-transactions.md)
- 다음: [사용자 정의 Repository](spring-data-jpa-custom-repository-fragments.md)
