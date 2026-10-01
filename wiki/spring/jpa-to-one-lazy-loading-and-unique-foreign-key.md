# JPA 1:1 매핑 선택과 지연 로딩

> Sources: Jakarta Persistence, Unknown; Hibernate ORM, Unknown; Spring Data JPA, Unknown; PostgreSQL Global Development Group, Unknown
> Raw: [Jakarta 관계 매핑 규칙](../../raw/spring/jakarta-persistence-3-2-relationship-mapping-consistency.md); [Hibernate 양방향 OneToOne 지연 로딩](../../raw/spring/hibernate-7-1-one-to-one-lazy-association.md); [Hibernate to-one 연관관계 가이드](../../raw/spring/hibernate-7-1-to-one-association-guide.md); [Jakarta ManyToOne](../../raw/spring/jakarta-persistence-3-2-manytoone-javadoc.md); [Jakarta OneToOne](../../raw/spring/jakarta-persistence-3-2-onetoone-javadoc.md); [Jakarta JoinColumn](../../raw/spring/jakarta-persistence-3-2-joincolumn-unique-javadoc.md); [Spring Data JPA 쿼리 메서드](../../raw/spring/spring-data-jpa-query-methods.md); [PostgreSQL UNIQUE](../../raw/database/postgresql-unique-constraints.md); [PostgreSQL FK](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [PostgreSQL 부분 UNIQUE](../../raw/database/postgresql-18-partial-unique-constraints.md)
> Updated: 2026-10-01

## Overview

**선택 기준은 로딩 속도가 아니라 관계의 실제 개수다.** 전체 자식 행을 통틀어 부모당 최대 한 행이면 외래 키 소유 쪽에 `@OneToOne(fetch = LAZY)`와 DB 고유 제약을 둔다. 같은 부모에 이력 행이 여러 개 생길 수 있다면 `@ManyToOne(fetch = LAZY)`을 쓰고, 활성 행만 하나로 제한하는 규칙은 조건부 고유 인덱스로 표현한다. Jakarta Persistence 명세는 `@ManyToOne` 외래 키 **전체에** 고유 제약을 지정하는 매핑을 허용하지 않는다. 따라서 기존 문서의 `@ManyToOne` + 일반 `UNIQUE` 조합은 JPA 표준에 맞는 권장안이 아니다. [Jakarta 관계 매핑 규칙](../../raw/spring/jakarta-persistence-3-2-relationship-mapping-consistency.md); [Hibernate to-one 연관관계 가이드](../../raw/spring/hibernate-7-1-to-one-association-guide.md); [PostgreSQL 부분 UNIQUE](../../raw/database/postgresql-18-partial-unique-constraints.md)

## 선택 순서

| 확인할 질문 | 답 | 매핑과 DB 규칙 |
| --- | --- | --- |
| 같은 부모에 자식 행을 평생 최대 하나만 저장하는가? | 예 | 자식의 `@OneToOne(fetch = LAZY)`와 외래 키 `UNIQUE` |
| 삭제·교체 이력 등으로 같은 부모에 자식 행을 여러 개 저장하는가? | 예 | 자식의 `@ManyToOne(fetch = LAZY)`; 활성 행만 하나여야 하면 조건부 고유 인덱스 |
| 자식의 기본 키를 부모 기본 키와 같게 둘 수 있는가? | 예, 진짜 1:1 | `@OneToOne` + `@MapsId`도 검토 |
| 부모에서 선택적 자식을 속성으로 바로 탐색해야 하는가? | 예 | 역방향 `@OneToOne(mappedBy = ...)`의 추가 조회와 목록 조회 비용을 먼저 검증 |

첫 질문의 `UNIQUE`는 **전체 행**에 적용된다. 두 번째 질문의 조건부 고유 인덱스는 **조건을 만족하는 행만** 검사하므로, 이력 행 여러 개를 남기는 다대일 관계와 양립한다. DB별 구현은 [부분 유니크 인덱스와 MariaDB의 조건부 고유성 구현](../database/partial-unique-indexes-and-mariadb-alternatives.md)에 정리되어 있다. [PostgreSQL 부분 UNIQUE](../../raw/database/postgresql-18-partial-unique-constraints.md); [Jakarta 관계 매핑 규칙](../../raw/spring/jakarta-persistence-3-2-relationship-mapping-consistency.md)

## 예시 A: 게시글마다 설정 행을 최대 하나만 저장

`post_settings.post_id`가 `post.id`를 가리키고, 옛 설정을 별도 행으로 보관하지 않는다고 하자. `UNIQUE (post_id)`는 같은 게시글을 가리키는 두 설정 행을 거부한다. 설정이 없는 게시글은 허용된다. `optional = false`는 **설정 행에 부모 게시글이 반드시 있어야 한다**는 자식 쪽 조건이지, 모든 게시글에 설정 행이 있어야 한다는 뜻이 아니다. [PostgreSQL FK](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [PostgreSQL UNIQUE](../../raw/database/postgresql-unique-constraints.md); [Jakarta OneToOne](../../raw/spring/jakarta-persistence-3-2-onetoone-javadoc.md)

```sql
CREATE TABLE post (id bigint PRIMARY KEY);
CREATE TABLE post_settings (
    id bigint PRIMARY KEY,
    post_id bigint NOT NULL UNIQUE REFERENCES post (id)
);
```

```java
@Entity
@Table(name = "post_settings")
class PostSettings {
    @Id Long id;

    @OneToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "post_id", nullable = false, unique = true)
    Post post;
}
```

외래 키가 있는 `PostSettings.post`는 연결할 `Post`의 ID를 알고 시작하므로 Hibernate의 공식 예시처럼 `LAZY`를 요청할 수 있다. `@OneToOne` 자체가 지연 로딩을 막는 것은 아니다. `@JoinColumn(unique = true)`는 JPA 스키마 생성용 메타데이터이므로, migration으로 DB를 관리한다면 실제 스키마에도 위 고유 제약을 적용해야 한다. 두 to-one 어노테이션 모두 기본 fetch는 `EAGER`이며, JPA의 `LAZY`는 구현체에 주는 힌트다. [Hibernate to-one 연관관계 가이드](../../raw/spring/hibernate-7-1-to-one-association-guide.md); [Jakarta OneToOne](../../raw/spring/jakarta-persistence-3-2-onetoone-javadoc.md); [Jakarta JoinColumn](../../raw/spring/jakarta-persistence-3-2-joincolumn-unique-javadoc.md)

자식의 정체성이 부모와 같다면 별도 `id`와 `post_id` 대신 `post_id`를 기본 키이자 외래 키로 쓰는 `@MapsId` 방식도 있다. 이 경우 부모 ID로 자식을 직접 조회할 수 있다. [Hibernate to-one 연관관계 가이드](../../raw/spring/hibernate-7-1-to-one-association-guide.md); [Hibernate 양방향 OneToOne 지연 로딩](../../raw/spring/hibernate-7-1-one-to-one-lazy-association.md)

## 예시 B: 설정 변경 이력을 남기고 활성 행만 하나 허용

같은 게시글의 설정 변경 전후를 각각 행으로 남기면 `post_id` 전체를 고유하게 만들 수 없다. 관계는 여러 이력 행에서 한 게시글로 향하는 **다대일**이다. PostgreSQL에서는 활성 행에만 적용되는 고유 인덱스로 게시글마다 활성 설정을 최대 하나로 제한할 수 있다. 아래 SQL과 매핑은 공식 문서의 규칙을 조합한 **설명용 설계**다. [Jakarta 관계 매핑 규칙](../../raw/spring/jakarta-persistence-3-2-relationship-mapping-consistency.md); [PostgreSQL 부분 UNIQUE](../../raw/database/postgresql-18-partial-unique-constraints.md)

```sql
CREATE TABLE post_settings_revision (
    id bigint PRIMARY KEY,
    post_id bigint NOT NULL REFERENCES post (id),
    active boolean NOT NULL
);
CREATE UNIQUE INDEX one_active_settings_per_post
    ON post_settings_revision (post_id) WHERE active;
```

```java
@Entity
@Table(name = "post_settings_revision")
class PostSettingsRevision {
    @Id Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "post_id", nullable = false)
    Post post;

    boolean active;
}
```

여기에는 `@JoinColumn(unique = true)`를 붙이지 않는다. 그러면 이력 행도 같은 `post_id`를 쓸 수 없게 된다. 활성 설정을 부모에서 찾는 기능은 별도 조회로 구현할 수 있다. 예를 들어 Spring Data JPA에서 `@Query("select r from PostSettingsRevision r where r.post.id = :postId and r.active = true")`로 조회하면 부모에 선택적 역방향 속성을 둘 필요가 없다. 여러 게시글 목록에서 이 조회를 건별로 반복하지 않도록 일괄 조회 쿼리를 설계한다. [Spring Data JPA 쿼리 메서드](../../raw/spring/spring-data-jpa-query-methods.md); [Hibernate 양방향 OneToOne 지연 로딩](../../raw/spring/hibernate-7-1-one-to-one-lazy-association.md)

## 부모에서 자식을 읽을 때의 별도 판단

자식 행에는 `post_id`가 있지만 부모 `post` 행에는 자식 ID가 없다. 부모의 선택적 `@OneToOne(mappedBy = "post")` 속성이 `null`인지 알아내려면 Hibernate가 자식 테이블을 추가 조회할 수 있다. 이것이 목록에서 반복되면 N+1이 된다. **`@ManyToOne`으로 바꿔도 부모에 외래 키가 생기지는 않는다.** Hibernate는 양방향이 꼭 필요할 때 bytecode enhancement를 검토하고, 공유 기본 키가 가능하면 `@MapsId`를 권한다. 선택적 자식을 자주 읽는 화면은 매핑 이름보다 실행 SQL과 조회 방식을 기준으로 판단한다. [Hibernate 양방향 OneToOne 지연 로딩](../../raw/spring/hibernate-7-1-one-to-one-lazy-association.md); [Hibernate to-one 연관관계 가이드](../../raw/spring/hibernate-7-1-to-one-association-guide.md)

선택적이지 않은 역방향 `@OneToOne`에 `LAZY`를 쓰는 Hibernate 예시도 있으므로, 역방향이 항상 지연 로딩 불가능한 것은 아니다. 단, `optional = false`를 성능 목적으로 선언하려면 **실제로 모든 부모에 자식이 존재한다는 모델**이어야 한다. 실제 쿼리 수는 적용한 Hibernate 버전과 설정에서 확인한다. [Hibernate to-one 연관관계 가이드](../../raw/spring/hibernate-7-1-to-one-association-guide.md); [Hibernate 양방향 OneToOne 지연 로딩](../../raw/spring/hibernate-7-1-one-to-one-lazy-association.md)

## 기존 `@ManyToOne` + 일반 `UNIQUE` 주장 판정

- 외래 키 소유 쪽 `@OneToOne(fetch = LAZY)`도 Hibernate 공식 예시에 있으므로, **지연 로딩만을 위해 `@ManyToOne`을 선택할 근거는 없다.** [Hibernate to-one 연관관계 가이드](../../raw/spring/hibernate-7-1-to-one-association-guide.md)
- `@ManyToOne` 외래 키 전체에 고유 제약을 선언하는 매핑은 Jakarta Persistence 명세의 관계 개수 규칙에 어긋난다. 기존 문서의 `@ManyToOne` + `@JoinColumn(unique = true)` 예시는 표준 JPA 사용법으로 권장할 수 없어 `@OneToOne` 예시로 교체했다. [Jakarta 관계 매핑 규칙](../../raw/spring/jakarta-persistence-3-2-relationship-mapping-consistency.md)
- DB의 고유 외래 키가 보장하는 것은 **부모당 자식 최대 한 행**이다. 모든 부모에 자식이 반드시 있다는 보장은 별개다. [PostgreSQL UNIQUE](../../raw/database/postgresql-unique-constraints.md); [PostgreSQL FK](../../raw/database/postgresql-18-primary-and-foreign-keys.md)

## See Also

- [부분 유니크 인덱스와 MariaDB의 조건부 고유성 구현](../database/partial-unique-indexes-and-mariadb-alternatives.md)
- [Database Unique Constraint](../database/unique-constraints.md)
- [Spring DB 접근 기술 비교](spring-database-access-technologies.md)
