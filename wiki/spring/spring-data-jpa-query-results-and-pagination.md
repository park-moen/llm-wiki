# Spring Data JPA 조회 반환 타입과 페이징

> Sources: Spring Data JPA, Unknown; Spring Data Commons, Unknown
> Raw: [Spring Data 조회 메서드](../../raw/spring/spring-data-jpa-query-details-reference.md); [Spring Data 반환 타입](../../raw/spring/spring-data-jpa-query-returns-reference.md); [Spring Data null 처리](../../raw/spring/spring-data-jpa-null-handling-reference.md); [PageRequest API](../../raw/spring/spring-data-page-request-javadoc.md)
> Updated: 2026-10-15

## Overview

Repository의 반환 타입은 단순한 Kotlin 문법이 아니라 **조회 결과가 없거나 여러 개인 경우의 계약**이다. 목록 화면에서는 전체 건수가 필요한지에 따라 `Page`, `Slice`, `List`를 선택한다. 메서드 이름으로 조건을 만드는 기초는 기존 [Spring Data JPA 입문](spring-database-access-technologies.md)에 있으므로, 여기서는 결과와 조회 범위를 설계한다. [Spring Data 반환 타입](../../raw/spring/spring-data-jpa-query-returns-reference.md)

## 단건과 목록을 구분하기

| 반환 형태 | 결과 없음 | 결과가 여러 개일 때 |
| --- | --- | --- |
| `Member?` | null | 단건 기대를 위반하면 예외 |
| `Member` | Kotlin non-null 계약에 따라 예외 | 단건 기대를 위반하면 예외 |
| `Optional<Member>` | empty | 단건 기대를 위반하면 예외 |
| `List<Member>` | 빈 목록 | 여러 결과 반환 |

Kotlin nullable 반환은 여러 일치 결과 중 임의의 하나를 선택한다는 뜻이 아니다. 이메일 단건 조회라면 DB 제약으로 유일성도 맞춰야 한다. 컬렉션·wrapper 반환은 결과가 없을 때 null 대신 빈 형태를 사용한다. Kotlin nullability를 해석하려면 공식 문서가 안내하는 `kotlin-reflect` 구성도 확인한다. [Spring Data null 처리](../../raw/spring/spring-data-jpa-null-handling-reference.md); [Spring Data 반환 타입](../../raw/spring/spring-data-jpa-query-returns-reference.md)

## 페이징과 정렬 예제

[공통 Member 엔티티](jpa-persistence-context-and-entity-lifecycle.md)를 사용하는 학습용 코드다.

```kotlin
interface MemberRepository : JpaRepository<Member, Long> {
    fun findByEmail(email: String): Member?

    fun findByActive(active: Boolean, pageable: Pageable): Page<Member>

    fun findByNameContaining(name: String, pageable: Pageable): Slice<Member>
}
```

```kotlin
val pageable = PageRequest.of(
    0,
    20,
    Sort.by("name").ascending().and(Sort.by("id").ascending())
)
val page = memberRepository.findByActive(true, pageable)
```

`PageRequest`의 페이지 번호는 첫 페이지에서 `0`으로 시작하며, 예제의 크기는 학습을 위해 정한 값이다. 정렬의 프로퍼티 이름은 엔티티 기준이다. 이름이 같은 회원의 순서까지 정하기 위해 고유 ID를 뒤에 붙였다. 이 정렬 조합은 예제의 설계 선택이며, 동시 데이터 변경 중 여러 페이지의 완전한 일관성을 보장하는 장치는 아니다. [PageRequest API](../../raw/spring/spring-data-page-request-javadoc.md); [Spring Data 조회 메서드](../../raw/spring/spring-data-jpa-query-details-reference.md)

## Page·Slice·List 선택

| 형태 | 추가 정보 | 조회 비용을 볼 때 |
| --- | --- | --- |
| `Page` | 전체 건수·전체 페이지 | 전체 개수를 얻기 위한 count가 필요할 수 있음 |
| `Slice` | 다음 구간의 존재 여부 | 전체 건수 없이 이어 읽기 |
| `List` + `Pageable` | 해당 범위의 데이터 | Page용 count metadata를 만들지 않음 |

`Page`를 쓰면 항상 SQL이 정확히 같은 횟수 실행된다고 외우지 않는다. 중요한 것은 전체 건수를 제공하기 위해 추가 count가 발생할 수 있다는 점이다. 단순 더 보기라면 `Slice`가 필요한 계약을 더 잘 표현할 수 있다. [Spring Data 조회 메서드](../../raw/spring/spring-data-jpa-query-details-reference.md)

## 주의할 점

단건을 기대하는 메서드와 `First`·`Top`으로 조회 수를 제한한 메서드는 다르다. 제한을 쓰면 유일성이 깨진 데이터를 조용히 하나만 가져올 수 있으므로, 실제 요구사항이 단건인지 최신 한 건인지 먼저 구분한다. 이 문장은 결과 제한 규칙에서 도출한 설계 지침이다. [Spring Data 조회 메서드](../../raw/spring/spring-data-jpa-query-details-reference.md)

`Stream` 반환은 리소스 수명을 가진다. 주변 트랜잭션이 필요하고 사용 후 닫아야 하며, 반환 타입이 Stream이라는 이유만으로 DB에서 조금씩만 읽는다고 보장되지 않는다. [Spring Data 반환 타입](../../raw/spring/spring-data-jpa-query-returns-reference.md)

## 직접 작성하는 연습

같은 조건을 `Page`와 `Slice`로 각각 선언해 조회한다. SQL 로그에서 count 여부를 확인하고, 화면에 “전체 회원 수”가 없다면 어떤 반환 타입이 충분한지 설명한다. 컬렉션 fetch join을 페이징에 그대로 섞지 않는 이유는 [fetch 계획](jpa-fetch-plans-and-n-plus-one.md)에서 복습한다.

## See Also

- 이전: [save와 새 엔티티 판별](spring-data-jpa-save-and-is-new.md)
- 다음: [직접 작성하는 조회·수정 쿼리](spring-data-jpa-declared-and-modifying-queries.md)
