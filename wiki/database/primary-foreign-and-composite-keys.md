# Primary Key, Foreign Key와 복합 키

> Sources: PostgreSQL Global Development Group, Unknown; Microsoft, 2026-07-20; Jakarta Persistence, Unknown; Jakarta Persistence Team, 2024-04-10; Kotlin, 2026-08-12; Kotlin, Unknown; Spring Data JPA, Unknown
> Raw: [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [Microsoft Primary and Foreign Key Constraints](../../raw/database/2026-07-20-microsoft-primary-and-foreign-key-constraints.md); [Jakarta Persistence IdClass Javadoc](../../raw/spring/jakarta-persistence-3-2-idclass-javadoc.md); [Jakarta Persistence EmbeddedId Javadoc](../../raw/spring/jakarta-persistence-3-2-embeddedid-javadoc.md); [Jakarta Persistence 키 규칙](../../raw/spring/jakarta-persistence-3-2-entity-and-composite-id-rules.md); [Kotlin no-arg plugin](../../raw/spring/2026-08-12-kotlin-no-arg-compiler-plugin.md); [Kotlin data class](../../raw/spring/kotlin-data-classes.md); [Spring Data JPA Repository](../../raw/spring/spring-data-jpa-repository-core-concepts.md)
> Updated: 2026-09-29

## Overview

`PRIMARY KEY`(PK)는 한 테이블의 행을 식별하는 열 또는 열 조합이고, `FOREIGN KEY`(FK)는 다른 테이블의 행을 참조하는 열 또는 열 조합에 거는 제약이다. **복합 키**는 여러 열을 함께 키로 쓰는 형태를 가리킨다. 따라서 PK도, FK도 여러 열로 구성할 수 있다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md)

| 개념 | 확인하는 것 | PostgreSQL 선언 예 |
|---|---|---|
| PK | 행을 식별하는 값이 고유하고 `NULL`이 아닌가 | `product_no integer PRIMARY KEY` |
| FK | 참조 열의 값이 참조 대상 테이블의 행과 맞는가 | `product_no integer REFERENCES products (product_no)` |
| 복합 PK | 여러 열의 **조합**이 행을 식별하는가 | `PRIMARY KEY (a, c)` |
| 복합 FK | 여러 참조 열의 **조합**이 대상 키와 맞는가 | `FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)` |

표의 구문은 PostgreSQL 문서의 예시다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md)

## PK: 행을 식별하는 기준

PK를 이루는 값은 고유해야 하고 `NULL`일 수 없다. PostgreSQL에서는 테이블에 PK를 하나만 둘 수 있지만, 그 PK가 여러 열로 구성될 수 있다. 여러 열에 각각 PK를 선언한다는 뜻이 아니다. PK를 선언하면 그 열 또는 열 조합에 고유 B-tree 인덱스가 자동으로 만들어진다. Microsoft도 SQL Server에서 PK 선언 시 고유 인덱스가 생성된다고 설명한다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [Microsoft Primary and Foreign Key Constraints](../../raw/database/2026-07-20-microsoft-primary-and-foreign-key-constraints.md)

```sql
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);
```

## FK: 다른 테이블의 행을 가리키는 기준

FK는 참조하는 테이블에 선언한다. 아래 예시에서 `orders.product_no`는 `products.product_no`를 참조하므로, 상품 테이블에 없는 값을 주문의 `product_no`에 넣을 수 없다. 다만 `product_no`가 `NULL`인 경우는 이 예시의 FK 검사 대상에서 제외된다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md)

```sql
CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    product_no integer REFERENCES products (product_no),
    quantity integer
);
```

PostgreSQL에서 FK의 참조 대상은 PK 외에도 `UNIQUE` 제약을 구성하는 열이나 일부 조건을 만족하는 고유 인덱스의 열이 될 수 있다. FK를 선언해도 **참조하는 쪽 열의 인덱스는 자동으로 생기지 않는다**. Microsoft도 SQL Server의 FK 인덱스가 자동 생성되지 않는다고 설명한다. 참조 대상 행의 삭제·키 변경 또는 FK를 이용한 조회 패턴에 따라 인덱스를 별도로 검토한다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [Microsoft Primary and Foreign Key Constraints](../../raw/database/2026-07-20-microsoft-primary-and-foreign-key-constraints.md)

## 복합 키: 여러 열을 함께 쓰기

복합 PK에서는 열 하나하나가 아니라 **열 조합**이 행을 식별한다. PostgreSQL은 `PRIMARY KEY (a, c)`처럼 테이블 제약으로 선언한다. 복합 FK도 `FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)`처럼 테이블 제약으로 선언하며, 양쪽 열의 수와 타입이 맞아야 한다. 참조 대상 열 조합은 PK 또는 PostgreSQL이 허용하는 고유 제약·인덱스 조건을 충족해야 한다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md)

Microsoft의 `Purchasing.ProductVendor` 예시에서는 `ProductID`와 `VendorID`가 함께 복합 PK를 이룬다. 각 열의 값은 다른 행에서 반복될 수 있지만, 같은 `ProductID`·`VendorID` 조합을 가진 행은 다시 넣을 수 없다. 이 예시는 복합 키의 고유성이 개별 열이 아닌 **열 조합 전체**에 적용된다는 점을 보여 준다. [Microsoft Primary and Foreign Key Constraints](../../raw/database/2026-07-20-microsoft-primary-and-foreign-key-constraints.md)

```sql
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    PRIMARY KEY (a, c)
);
```

PK와 FK는 역할이고, 복합 키는 키를 이루는 열의 개수에 관한 설명이다. 따라서 복합 PK와 복합 FK는 서로 다른 관계를 나타낸다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md)

## 번역 테이블의 복합 PK

글 하나에 언어별 번역을 저장한다면 `post_id`만으로는 번역 행을 구별할 수 없다. `lang_code`만으로도 구별할 수 없다. 두 열을 묶어 `PRIMARY KEY (post_id, lang_code)`로 선언하면 같은 글·언어 조합의 번역을 중복 저장할 수 없다. 다음은 앞의 복합 PK 규칙을 번역 테이블에 적용한 **설계 예시**다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [Microsoft Primary and Foreign Key Constraints](../../raw/database/2026-07-20-microsoft-primary-and-foreign-key-constraints.md)

```sql
CREATE TABLE t_post (
    id bigint PRIMARY KEY
);

CREATE TABLE t_post_translation (
    post_id bigint NOT NULL REFERENCES t_post (id),
    lang_code text NOT NULL,
    title text,
    PRIMARY KEY (post_id, lang_code)
);
```

여기서 `(post_id, lang_code)`는 번역 행의 **복합 PK**이고, `post_id`는 `t_post`를 참조하는 **FK**이기도 하다. PK와 FK는 같은 열에 동시에 적용될 수 있지만 맡는 검사는 다르다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md)

## JPA에서 복합 PK를 표현하는 방법

JPA는 entity의 필드와 DB 열을 연결하는 매핑 규칙을 제공한다. 단일 PK는 `@Id` 필드로 지정한다. 복합 PK는 별도 키 클래스를 정의하고, entity의 각 키 필드에 `@Id`를 붙이는 `@IdClass` 또는 키 객체 하나에 `@EmbeddedId`를 붙이는 방식으로 표현한다. 같은 entity에서 두 방식을 동시에 사용하지 않는다. [Jakarta Persistence 키 규칙](../../raw/spring/jakarta-persistence-3-2-entity-and-composite-id-rules.md); [Jakarta Persistence EmbeddedId Javadoc](../../raw/spring/jakarta-persistence-3-2-embeddedid-javadoc.md)

아래 두 Kotlin 구현은 **같은 `t_post_translation` 테이블을 매핑하는 대안**이다. 각각 독립적인 예제로 보고 한 방식만 선택한다. 예제의 `@field:`는 JPA 주석을 Kotlin의 필드에 붙여 필드 접근 방식을 분명히 한다.

### 방법 A: `@IdClass` — 키 필드를 entity에 직접 두기

`PostTranslationId`의 `postId`, `langCode`는 entity의 `@Id` 필드와 **이름·타입이 같아야** 한다. `@IdClass` 키 클래스 자체에는 `@Embeddable`이 필요하지 않다. Kotlin `data class`는 `equals`와 `hashCode`를 생성하며, 기본값을 둔 생성자는 인자 없는 생성자도 제공한다. 키 값의 동등성은 DB의 키 비교와 일치해야 한다. [Jakarta Persistence IdClass Javadoc](../../raw/spring/jakarta-persistence-3-2-idclass-javadoc.md); [Kotlin data class](../../raw/spring/kotlin-data-classes.md)

```kotlin
import jakarta.persistence.*
import java.io.Serializable

data class PostTranslationId(
    var postId: Long = 0L,
    var langCode: String = "",
) : Serializable

@Entity
@Table(name = "t_post_translation")
@IdClass(PostTranslationId::class)
open class PostTranslation(
    @field:Id
    @field:Column(name = "post_id")
    open var postId: Long,
    @field:Id
    @field:Column(name = "lang_code")
    open var langCode: String,
    @field:Column(name = "title")
    open var title: String?,
) {
    protected constructor() : this(0L, "", null)
}
```

entity에서는 `translation.postId`처럼 키 필드에 바로 접근한다. 다만 필드 이름이나 타입을 바꾸면 `PostTranslationId`도 함께 맞춰야 한다. 키 클래스가 없어지는 방식은 아니다. [Jakarta Persistence IdClass Javadoc](../../raw/spring/jakarta-persistence-3-2-idclass-javadoc.md)

### 방법 B: `@EmbeddedId` — 키를 객체로 묶기

`@EmbeddedId` 필드의 타입은 `@Embeddable` 키 클래스여야 한다. 키 클래스는 값 동등성을 구현해야 하므로 아래 예시에서는 두 키 필드로 `equals`·`hashCode`를 정의했다. entity에는 `@Id`를 추가하지 않는다. [Jakarta Persistence EmbeddedId Javadoc](../../raw/spring/jakarta-persistence-3-2-embeddedid-javadoc.md)

```kotlin
import jakarta.persistence.*
import java.io.Serializable
import java.util.Objects

@Embeddable
open class TranslationId(
    open var sourceId: Long,
    @field:Column(name = "lang_code")
    open var langCode: String,
) : Serializable {
    protected constructor() : this(0L, "")

    override fun equals(other: Any?): Boolean =
        other is TranslationId && sourceId == other.sourceId && langCode == other.langCode

    override fun hashCode(): Int = Objects.hash(sourceId, langCode)
}

@Entity
@Table(name = "t_post_translation")
open class PostTranslation(
    @field:EmbeddedId
    @field:AttributeOverride(name = "sourceId", column = Column(name = "post_id"))
    open var id: TranslationId,
    @field:Column(name = "title")
    open var title: String?,
) {
    protected constructor() : this(TranslationId(0L, ""), null)
}
```

entity에서는 `translation.id.sourceId`처럼 키 객체를 거쳐 접근한다. 키 클래스를 여러 entity에서 공유한다면 클래스 안에서는 `sourceId`라는 공통 속성명을 쓴다. 하지만 실제 DB 열은 `@AttributeOverride`로 각 entity의 `post_id`, `hotel_id` 등에 매핑할 수 있다. 키 구조와 의미가 같은 테이블끼리 공유할지 판단하고, 각 entity에서 필요한 열 이름을 지정한다. [Jakarta Persistence EmbeddedId Javadoc](../../raw/spring/jakarta-persistence-3-2-embeddedid-javadoc.md)

### Spring Data JPA 저장소에서 조회하기

`JpaRepository`의 두 번째 타입 인수는 entity의 ID 타입이다. `@IdClass`를 선택했다면 `PostTranslationId`, `@EmbeddedId`를 선택했다면 `TranslationId`를 쓴다. 두 방식 모두 조회할 때 키 값 전체를 전달한다. 아래는 **각 방식의 대안**이며 한 프로젝트에 동시에 선언하는 코드가 아니다. [Spring Data JPA Repository](../../raw/spring/spring-data-jpa-repository-core-concepts.md)

```kotlin
// @IdClass 구현을 선택한 경우
import org.springframework.data.jpa.repository.JpaRepository

interface PostTranslationRepository : JpaRepository<PostTranslation, PostTranslationId>

fun findTranslation(
    repository: PostTranslationRepository,
    postId: Long,
    langCode: String,
): PostTranslation? = repository.findById(PostTranslationId(postId, langCode)).orElse(null)
```

```kotlin
// @EmbeddedId 구현을 선택한 경우
import org.springframework.data.jpa.repository.JpaRepository

interface PostTranslationRepository : JpaRepository<PostTranslation, TranslationId>

fun findTranslation(
    repository: PostTranslationRepository,
    postId: Long,
    langCode: String,
): PostTranslation? = repository.findById(TranslationId(postId, langCode)).orElse(null)
```

Spring Boot에서 JPA entity를 Kotlin으로 작성할 때는 JPA가 사용할 인자 없는 생성자와 확장 가능한 entity 클래스를 준비해야 한다. 예제는 `open`과 보호된 기본 생성자를 명시했다. `kotlin("plugin.jpa")`를 구성하면 `@Entity`·`@Embeddable` 등에 합성 기본 생성자를 생성한다. 사용하는 Kotlin 버전에 맞춰 entity의 `open` 설정도 확인한다. [Jakarta Persistence 키 규칙](../../raw/spring/jakarta-persistence-3-2-entity-and-composite-id-rules.md); [Kotlin no-arg plugin](../../raw/spring/2026-08-12-kotlin-no-arg-compiler-plugin.md)

두 Kotlin 코드는 **복합 PK 매핑**을 보여 준다. `postId`라는 숫자 필드만 선언해도 `@ManyToOne` 관계나 DB의 FK 제약이 자동으로 생기는 것은 아니다. 위 SQL 예시처럼 DB에 FK를 따로 선언할 수 있고, entity에서 `Post` 객체와의 관계까지 매핑하려면 추가 JPA 매핑이 필요하다. `@EmbeddedId`로 부모 키를 공유하는 관계에는 `@MapsId`도 사용할 수 있다. [PostgreSQL Primary Keys and Foreign Keys](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [Jakarta Persistence EmbeddedId Javadoc](../../raw/spring/jakarta-persistence-3-2-embeddedid-javadoc.md)

## 두 방식의 선택 기준

| 기준 | `@IdClass` | `@EmbeddedId` |
|---|---|---|
| entity에서 읽기 | `translation.postId`처럼 바로 접근 | `translation.id.sourceId`처럼 키 객체를 거쳐 접근 |
| 키 정의 | entity와 키 클래스의 키 필드 이름·타입을 맞춰야 함 | 키 구조를 `@Embeddable` 클래스 하나에 모음 |
| 여러 테이블에서 키 클래스 사용 | entity마다 같은 이름·타입의 `@Id` 필드가 필요 | `@AttributeOverride`로 entity마다 DB 열 이름을 바꿀 수 있음 |
| 적합한 상황 | entity 필드를 평평하게 두고 직접 조회·사용하는 코드가 중심일 때 | 키 조합을 하나의 값으로 다루거나 같은 구조를 여러 매핑에 쓸 때 |

마지막 행은 공식 문서의 매핑 구조에서 도출한 **설계 판단**이다. `@IdClass`가 항상 우선이라는 표준 규칙은 없다. 키를 코드에서 어떻게 다루는지와 키 클래스의 재사용 범위를 기준으로 선택한다. [Jakarta Persistence IdClass Javadoc](../../raw/spring/jakarta-persistence-3-2-idclass-javadoc.md); [Jakarta Persistence EmbeddedId Javadoc](../../raw/spring/jakarta-persistence-3-2-embeddedid-javadoc.md)

## See Also

- [Database Unique Constraint](unique-constraints.md)
- [외래 키의 `ON DELETE CASCADE`와 `ON DELETE SET NULL`](foreign-key-on-delete-actions.md)
- [Spring DB 접근 기술 비교](../spring/spring-database-access-technologies.md)
