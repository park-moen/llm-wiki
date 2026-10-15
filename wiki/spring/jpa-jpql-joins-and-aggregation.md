# JPA JPQL 조인·집계·DTO 조회

> Sources: Jakarta Persistence, Unknown
> Raw: [Jakarta Persistence Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md)
> Updated: 2026-10-15

## Overview

JPQL은 테이블이 아니라 엔티티와 그 프로퍼티를 대상으로 조회한다. 단순한 이름 조건 조회는 기존 [DB 접근 기술 비교](spring-database-access-technologies.md)에 있으므로, 여기서는 **조인·집계·DTO 조회**를 중심으로 확장한다. JPQL과 Hibernate의 더 넓은 HQL 기능을 동일한 표준으로 취급하지 않는다. [JPA Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md)

## DB 이름 대신 엔티티 이름을 쓴다

[공통 Member 엔티티](jpa-persistence-context-and-entity-lifecycle.md)의 테이블 이름은 `members`지만 JPQL에서는 기본 엔티티 이름인 `Member`를 사용한다. 프로퍼티도 column 이름이 아닌 Kotlin 엔티티의 이름으로 적는다.

```jpql
select m
from Member m
where m.active = true
order by m.name asc
```

조건값은 문자열 연결 대신 `:name` 같은 매개변수로 전달한다. 엔티티·프로퍼티 이름은 대소문자를 구분하며, 별칭은 해당 조회 안에서 경로를 표현하는 데 쓴다. [JPA Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md)

## 연관 필드로 조인하기

`Purchase.member`가 회원을 참조하는 연관 필드라고 가정한 학습용 JPQL이다.

```jpql
select p
from Purchase p
join p.member m
where m.email = :email
```

`join p.member`는 매핑된 객체 관계를 따라 조인한다. `join`은 inner join이므로 연결된 회원이 없는 구매는 결과에서 빠진다. 연결이 없는 행까지 유지하려면 목적에 맞게 `left join`을 사용한다. [JPA Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md)

일반 조인과 fetch join의 목적은 다르다. 일반 조인을 썼다는 이유만으로 연관 엔티티가 초기화된다고 가정하지 않는다. fetch join은 조회 결과와 함께 연관 객체를 가져오는 용도이며 다음 [조회 성능 문서](jpa-fetch-plans-and-n-plus-one.md)에서 다룬다. [JPA Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md)

## 집계 결과를 DTO로 받기

다음은 `example` 패키지에 DTO가 있고 `Purchase.member` 관계가 있다는 가정의 학습 예제다.

```kotlin
package example

data class PurchaseCount(val memberId: Long, val count: Long)
```

```jpql
select new example.PurchaseCount(m.id, count(p))
from Purchase p
join p.member m
group by m.id
having count(p) > :minimum
```

`WHERE`는 그룹을 만들기 전 행의 조건을, `HAVING`은 그룹별 집계 결과의 조건을 표현한다. `COUNT`의 반환 타입은 `Long`이다. `select new`에는 DTO의 패키지를 포함한 클래스 이름과 생성자에 맞는 값의 순서를 적는다. 엔티티를 DTO로 선언하는 것이 아니라 조회 결과를 담을 별도 클래스를 만드는 것이다. [JPA Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md)

## 대량 수정·삭제는 별도 동작이다

JPQL의 bulk update·delete는 조건에 맞는 DB 데이터를 직접 변경한다. 영속성 컨텍스트의 관리 객체 상태가 자동으로 같은 값이 되는 것은 아니다. 같은 작업 안에서 bulk 변경과 엔티티 조회·수정을 섞으면 오래된 값을 읽을 수 있으므로 경계를 나누거나 동기화 전략을 정한다. Spring Data의 `@Modifying` 적용은 [직접 작성하는 쿼리](spring-data-jpa-declared-and-modifying-queries.md)에서 이어서 다룬다. [JPA Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md)

## 직접 작성하는 연습

회원별 구매 수를 반환하는 DTO 조회를 작성한다. `HAVING` 조건을 바꾸어 결과를 비교하고, 구매가 없는 회원까지 필요하다면 `Member`에서 시작하는 left join으로 쿼리를 바꾼다. 어떤 엔티티에서 조회를 시작하느냐에 따라 남는 행이 달라지는지 확인한다.

## See Also

- 이전: [cascade와 orphanRemoval](jpa-cascade-and-orphan-removal.md)
- 다음: [N+1과 fetch 계획](jpa-fetch-plans-and-n-plus-one.md)
- [Spring Data DTO Projection](spring-data-jpa-projections-and-dtos.md)
