# JPA 엔티티·필드·연관 컬렉션과 DB 구조의 대응

> Sources: Jakarta Persistence, Unknown; Hibernate ORM, Unknown; PostgreSQL Global Development Group, Unknown; MariaDB, Unknown
> Raw: [Jakarta Table](../../raw/spring/jakarta-persistence-3-2-table-javadoc.md); [Jakarta Column](../../raw/spring/jakarta-persistence-3-2-column-javadoc.md); [Jakarta ManyToOne](../../raw/spring/jakarta-persistence-3-2-manytoone-javadoc.md); [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md); [Jakarta 컬렉션 속성 규칙](../../raw/spring/jakarta-persistence-3-2-collection-valued-attributes.md); [Hibernate 양방향 OneToMany](../../raw/spring/hibernate-7-1-bidirectional-onetomany-guide.md); [PostgreSQL Schemas](../../raw/database/postgresql-18-schemas.md); [MariaDB Database vs. Schema](../../raw/database/mariadb-database-vs-schema.md); [Hibernate SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md)
> Updated: 2026-10-02

## Overview

JPA는 Java 객체 모델과 관계형 DB 구조를 연결한다. **엔티티 클래스는 보통 테이블에, 엔티티 객체 하나는 그 테이블의 row 하나에, 단순 값 필드는 column에 대응한다.** 다만 이것은 이해를 돕는 기본형이다. 엔티티가 추가 테이블을 사용할 수도 있으므로 `엔티티 = 테이블`을 항상 성립하는 등식으로 쓰면 안 된다. `schema`는 엔티티나 테이블의 다른 이름도 아니다. [Jakarta Table](../../raw/spring/jakarta-persistence-3-2-table-javadoc.md); [Jakarta Column](../../raw/spring/jakarta-persistence-3-2-column-javadoc.md)

## 먼저 용어를 분리하기

| 코드에서 보는 것 | DB에서 가까운 것 | 예시 |
| --- | --- | --- |
| 엔티티 클래스 | 주로 테이블에 매핑되는 객체 타입 | `Post` ↔ `post` |
| 엔티티 객체 하나 | 테이블의 row 하나 | `Post(id=5)` ↔ `post.id = 5`인 row |
| 단순 값 필드 | column에 매핑되는 값 | `Comment.body` ↔ `comment.body` |
| `@ManyToOne Post post` | 다른 엔티티 **하나**를 가리키는 연관 필드; 보통 자식 테이블의 외래 키 | `comment.post_id` |
| `@OneToMany List<Comment> comments` | 연결된 엔티티 **여러 개**를 담는 연관 컬렉션 | `post_id = 5`인 `comment` row의 모음 |
| schema | DB 제품에 따라 테이블을 묶는 이름 공간 또는 database의 동의어 | PostgreSQL의 `RECORDS.CUST`에서 `RECORDS` |

`@Column`은 단순 값 필드·속성이 매핑될 column을 지정한다. 반면 `@ManyToOne`은 객체 참조 필드이고, `@OneToMany`는 여러 객체를 담는 필드다. **컬렉션 필드 하나가 부모 테이블의 column 하나라는 뜻은 아니다.** `@OneToMany`는 보통 자식 테이블의 외래 키를 통해 여러 자식 row와 연결되고, 매핑에 따라 연결 테이블을 쓸 수도 있다. [Jakarta Column](../../raw/spring/jakarta-persistence-3-2-column-javadoc.md); [Jakarta ManyToOne](../../raw/spring/jakarta-persistence-3-2-manytoone-javadoc.md); [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md)

`schema`의 뜻은 DB 제품에 따라 다르다. PostgreSQL에서는 테이블 등을 담는 이름 공간이고 `schema.table` 형태로 쓴다. MariaDB에서는 `schema`와 `database`가 같은 뜻이다. JPA의 `@Table(name = "CUST", schema = "RECORDS")`도 **테이블 이름과 schema 이름을 별도로 지정**한다. 일상적인 DB 설계 대화에서 말하는 ‘스키마’는 테이블·column·제약의 구조 전체를 뜻할 수도 있으므로 문맥을 확인한다. [PostgreSQL Schemas](../../raw/database/postgresql-18-schemas.md); [MariaDB Database vs. Schema](../../raw/database/mariadb-database-vs-schema.md); [Jakarta Table](../../raw/spring/jakarta-persistence-3-2-table-javadoc.md)

## 같은 관계를 두 방향에서 보기

아래는 게시글과 댓글의 **설명용 매핑**이다. 개발자가 `comments` 필드를 선언하고 JPA에 관계를 알려 준다. `@OneToMany`가 Java 필드를 저절로 만들어 주는 것은 아니다. `mappedBy = "post"`는 자식의 `Comment.post` 필드가 관계를 소유한다는 뜻이다. [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md); [Hibernate 양방향 OneToMany](../../raw/spring/hibernate-7-1-bidirectional-onetomany-guide.md)

```java
@Entity
@Table(name = "post")
class Post {
    @Id Long id;

    @OneToMany(mappedBy = "post")
    List<Comment> comments = new ArrayList<>();
}

@Entity
@Table(name = "comment")
class Comment {
    @Id Long id;
    @Column(name = "body") String body;

    @ManyToOne
    @JoinColumn(name = "post_id")
    Post post;
}
```

| post.id | comment.id | comment.body | comment.post_id |
| --- | --- | --- | --- |
| 5 | 10 | 첫 댓글 | 5 |
| 5 | 11 | 둘째 댓글 | 5 |

위 표는 두 테이블의 관계를 한 줄씩 나란히 보여 주는 **가상 데이터**다. `Comment(id=10).post`는 `Post(id=5)` 객체 **하나**를 가리키고, `Post(id=5).comments`는 댓글 두 row를 나타내는 `List<Comment>`다. 코드에는 양쪽 탐색 필드가 있어도 이 예시의 DB 관계는 `comment.post_id` 외래 키 하나로 표현된다. [Jakarta ManyToOne](../../raw/spring/jakarta-persistence-3-2-manytoone-javadoc.md); [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md); [Hibernate 양방향 OneToMany](../../raw/spring/hibernate-7-1-bidirectional-onetomany-guide.md)

## 컬렉션은 배열이나 column을 뜻하는가

Jakarta Persistence가 말하는 컬렉션 속성은 `Collection`, `Set`, `List`, `Map` 같은 Java 컬렉션 인터페이스로 선언한 필드·속성이다. 위 `List<Comment>`는 **엔티티의 연관 컬렉션**이고, `@ManyToOne Post post`는 컬렉션이 아닌 **단일 객체 참조**다. 따라서 `@ManyToOne`을 썼다는 이유만으로 그쪽에 배열이나 컬렉션이 생기지는 않는다. [Jakarta 컬렉션 속성 규칙](../../raw/spring/jakarta-persistence-3-2-collection-valued-attributes.md); [Jakarta ManyToOne](../../raw/spring/jakarta-persistence-3-2-manytoone-javadoc.md); [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md)

컬렉션에도 종류가 있다. `@OneToMany List<Comment>`는 **엔티티 여러 개**를 담고, `@ElementCollection Set<String>`은 **기본값 여러 개**를 담는다. 둘 다 컬렉션 필드지만 매핑 대상이 다르다. `@ElementCollection`은 [JPA `@ElementCollection`](jpa-element-collection.md)에서 자세히 다룬다. [Jakarta 컬렉션 속성 규칙](../../raw/spring/jakarta-persistence-3-2-collection-valued-attributes.md)

이 구분은 `@SQLRestriction`에도 그대로 적용된다. **엔티티 클래스에 붙인 조건**은 그 엔티티의 Hibernate 생성 조회 SQL에, **`@OneToMany` 컬렉션 필드에 붙인 조건**은 그 연관 컬렉션을 읽는 SQL에 적용된다. 자세한 생성 SQL은 [Hibernate `@SQLRestriction`](hibernate-sqlrestriction.md)에 있다. [Hibernate SQLRestriction Javadoc](../../raw/spring/hibernate-7-1-sqlrestriction-javadoc.md); [Jakarta OneToMany](../../raw/spring/jakarta-persistence-3-2-onetomany-javadoc.md)

## See Also

- [Hibernate `@SQLRestriction`: 고정 조회 조건의 동작과 한계](hibernate-sqlrestriction.md)
- [JPA `@ElementCollection`](jpa-element-collection.md)
- [Primary Key, Foreign Key와 복합 키](../database/primary-foreign-and-composite-keys.md)
