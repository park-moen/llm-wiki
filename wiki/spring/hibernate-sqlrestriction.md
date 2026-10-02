# Hibernate `@SQLRestriction`: 고정 조회 조건의 동작과 한계

> Sources: Hibernate ORM, Unknown
> Raw: [SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md); [Hibernate 필터링 가이드](../../raw/spring/hibernate-7-1-sqlrestriction-user-guide.md); [SoftDelete Javadoc](../../raw/spring/hibernate-7-1-softdelete-javadoc.md); [Jakarta ManyToOne](../../raw/spring/jakarta-persistence-3-2-manytoone-javadoc.md); [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md)
> Updated: 2026-10-02

## Overview

`@SQLRestriction`은 Hibernate의 `org.hibernate.annotations.SQLRestriction`이다. 엔티티나 컬렉션을 읽을 때 Hibernate가 **생성하는 SQL에 항상 붙일 고정 조건**을 선언한다. 조건은 JPQL 속성명이 아닌 **DB 열을 사용하는 native SQL 술어**로 작성한다. 예를 들어 삭제 표시가 된 행을 평소 조회에서 숨길 수 있다. 이 어노테이션은 조회 조건을 추가할 뿐, 삭제 표시를 기록하거나 DB의 고유성 규칙을 만들지는 않는다. [SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md); [SoftDelete Javadoc](../../raw/spring/hibernate-7-1-softdelete-javadoc.md)

## 어디에 붙이는가

| 위치 | 적용 범위 | 예시 |
| --- | --- | --- |
| 엔티티 클래스 | 그 엔티티의 Hibernate 생성 조회 SQL | `Document`에서 삭제된 행 숨기기 |
| 연관 컬렉션 필드·메서드 | 그 컬렉션을 읽는 SQL | `owner.documents`에서 삭제된 문서만 제외 |
| `@ManyToMany`의 연결 테이블 | `@SQLJoinTableRestriction` 사용 | 연결 행의 상태로 필터링 |

Hibernate Javadoc은 엔티티와 `@OneToMany` 컬렉션에 같은 조건을 적용하는 예시를 각각 제시한다. 연결 테이블 행 자체를 제한할 때는 `@SQLJoinTableRestriction`을 구분해서 사용한다. [SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md)

여기서 **엔티티**는 `Document` 같은 객체 타입이고, **연관 컬렉션**은 `Owner.documents`처럼 여러 `Document`를 담도록 선언한 `List<Document>`·`Set<Document>` 필드다. `@ManyToOne Document document`는 객체 하나를 가리키므로 컬렉션이 아니다. 컬렉션은 DB 열 하나를 뜻하지도 않는다. 코드와 DB 행의 대응은 [JPA 엔티티·필드·연관 컬렉션과 DB 구조의 대응](jpa-entity-association-collection-mapping.md)에 설명했다. [Jakarta ManyToOne](../../raw/spring/jakarta-persistence-3-2-manytoone-javadoc.md); [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md)

```java
@Entity
@SQLRestriction("status <> 'DELETED'")
class Document {
    @Enumerated(EnumType.STRING)
    Status status;
}
```

위 조건은 `Document`를 읽을 때 삭제 상태의 행을 제외하려는 설정이다. 실제 컬럼명·저장 값·DB의 SQL 문법에 맞춰 작성해야 한다. Java 코드의 `status`라는 속성명과 DB 열 이름이 다르면 SQL에는 **DB 열 이름**을 써야 한다. 이는 술어가 native SQL이라는 정의에서 도출한 사용 규칙이다. [SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md)

## 내부 동작을 SQL로 보기

Hibernate 가이드에서는 `Account` 엔티티에 `@SQLRestriction("active = true")`, `Client.debitAccounts` 컬렉션에 `@SQLRestriction("account_type = 'DEBIT'")`를 선언한다. 엔티티 쿼리에는 `active = true`가 붙고, `debitAccounts`를 읽을 때는 **활성 조건·계정 종류 조건·부모 외래 키 조건**이 함께 붙는다. [Hibernate 필터링 가이드](../../raw/spring/hibernate-7-1-sqlrestriction-user-guide.md)

```sql
-- 가이드에 나온 SQL의 조건 부분
-- Account 목록
WHERE ( a.active = true )

-- Client.debitAccounts 컬렉션
WHERE ( d.active = true and d.account_type = 'DEBIT' ) AND d.client_id = 1
```

동작을 개념적으로 풀면 **매핑 시 고정 술어 등록 → Hibernate가 엔티티·컬렉션 SQL 생성 → 적용 대상의 술어를 다른 조회 조건과 조합 → DB 실행** 순서다. 위 SQL은 조건이 Java 컬렉션을 가져온 뒤 메모리에서 걸러지는 것이 아니라, DB로 보내는 SQL에 포함됨을 보여 준다. 이는 공식 가이드의 생성 SQL에서 확인한 동작이며, Hibernate 내부 코드의 모든 처리 단계를 기술한 것은 아니다. [Hibernate 필터링 가이드](../../raw/spring/hibernate-7-1-sqlrestriction-user-guide.md)

## 언제 쓰고, 언제 다른 방법을 쓰는가

| 요구 | 적합한 방법 | 이유 |
| --- | --- | --- |
| 모든 일반 조회에서 같은 고정 조건 적용 | `@SQLRestriction` | 매핑에 선언한 술어가 항상 적용됨 |
| 관리자 화면에서 조건을 끄거나 사용자마다 값을 바꿈 | `@Filter` 검토 | `@SQLRestriction`은 비활성화·매개변수 전달이 불가능함 |
| 삭제 요청 때 행을 실제 삭제하지 않고 표시 열을 바꿈 | `@SoftDelete` 등 삭제 동작 별도 설계 | `@SQLRestriction` 자체는 조회 조건임 |
| 활성 행의 중복을 DB에서 금지 | DB 고유 제약·조건부 고유 인덱스 | 조회 필터는 쓰기 시 중복을 거부하지 않음 |

Hibernate는 `@SQLRestriction`을 **정적 필터**, `@Filter`를 실행 시 켜고 값을 설정할 수 있는 **동적 필터**로 구분한다. `@SQLRestriction`은 항상 적용되고 끌 수 없으며 매개변수도 받을 수 없다. 소프트 삭제를 구현하려면 행에 삭제 표시를 기록하는 기능이 별도로 필요하고, Hibernate의 `@SoftDelete`는 그 표시 열을 다루는 기능이다. [SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md); [Hibernate 필터링 가이드](../../raw/spring/hibernate-7-1-sqlrestriction-user-guide.md); [SoftDelete Javadoc](../../raw/spring/hibernate-7-1-softdelete-javadoc.md)

`@SQLRestriction`의 문서상 적용 대상은 **Hibernate가 생성하는 SQL**이다. 직접 작성한 native SQL이나 DB 도구의 쿼리에도 같은 조건이 자동으로 붙는다고 가정하지 말고 별도로 확인한다. DB의 조건부 고유성은 [부분 유니크 인덱스와 MariaDB의 조건부 고유성 구현](../database/partial-unique-indexes-and-mariadb-alternatives.md)에서 다룬다. [SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md)

## See Also

- [JPA 엔티티·필드·연관 컬렉션과 DB 구조의 대응](jpa-entity-association-collection-mapping.md)
- [JPA 1:1 매핑 선택과 지연 로딩](jpa-to-one-lazy-loading-and-unique-foreign-key.md)
- [부분 유니크 인덱스와 MariaDB의 조건부 고유성 구현](../database/partial-unique-indexes-and-mariadb-alternatives.md)
