# Hibernate @GeneratedColumn과 DB 생성 열

> Sources: Hibernate ORM, Unknown; Jakarta Persistence, Unknown; PostgreSQL Global Development Group, 2026-08-13
> Raw: [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md); [Hibernate Generated Javadoc](../../raw/spring/hibernate-7-1-generated-javadoc.md); [Jakarta Persistence GeneratedValue Javadoc](../../raw/spring/jakarta-persistence-3-2-generatedvalue-javadoc.md); [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)
> Updated: 2026-09-29

## Overview

`@GeneratedColumn`은 **JPA 표준 주석이 아니라 Hibernate의 `org.hibernate.annotations.GeneratedColumn` 주석**이다. DB의 `GENERATED ALWAYS AS` 또는 이에 해당하는 생성 열을 entity 속성에 매핑한다. 주석의 필수 값에는 열을 계산할 SQL 식을 쓰며, Hibernate는 `INSERT`나 `UPDATE` 뒤 DB가 계산한 값을 다시 가져와 entity 상태에 반영한다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md)

## 무엇을 생성하는가

DB 생성 열은 다른 열을 바탕으로 값을 계산한다. PostgreSQL 문서에서는 행을 쓸 때 계산해 저장하는 생성 열과 읽을 때 계산하는 가상 생성 열을 구분한다. 일반적인 `DEFAULT` 값은 행을 처음 넣을 때 평가되지만, 생성 열은 바탕이 되는 행의 값이 바뀌면 다시 계산된다. PostgreSQL의 생성 열에는 값을 직접 `INSERT`하거나 `UPDATE`할 수 없다. [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)

Hibernate의 `@GeneratedColumn("계산식")`은 이 **DB 생성 열의 정의를 DDL에 포함**하고, 생성된 값을 Java 객체로 가져오도록 지정한다. DDL의 정확한 문법과 계산식 제약은 DB에 따라 다르므로 실제 사용하는 DB와 Hibernate dialect에서 확인해야 한다. 이는 Hibernate의 주석 설명과 PostgreSQL의 생성 열 규칙을 함께 적용한 판단이다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md); [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)

## 매핑 예시

아래 코드는 PostgreSQL 문서의 길이 환산 생성 열을 Hibernate entity에 매핑한 **설계 예시**다. 식의 `height_cm`은 Java 필드 이름이 아니라 DB 열 이름이다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md); [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)

```java
import java.math.BigDecimal;
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import org.hibernate.annotations.GeneratedColumn;

@Entity
public class Person {
    @Id
    private Long id;

    @Column(name = "height_cm")
    private BigDecimal heightCm;

    @Column(name = "height_in")
    @GeneratedColumn("height_cm / 2.54")
    private BigDecimal heightIn;
}
```

PostgreSQL 문서의 대응하는 DB 식은 `height_in numeric GENERATED ALWAYS AS (height_cm / 2.54)`이다. 이 예시의 Java 코드와 실제 생성 DDL이 목표 DB에서 맞는지는 사용하는 Hibernate 버전과 dialect로 확인해야 한다. [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md); [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md)

### 같은 행의 값으로 합계를 계산하는 경우

단가와 수량을 같은 행에 저장하고 행 합계를 DB가 계산하게 할 수도 있다. 아래는 PostgreSQL의 "현재 행만 참조" 규칙과 Hibernate의 생성 열 매핑 방식에 맞춰 작성한 **설계 예시**다. 실제 DDL과 타입은 사용하는 DB와 dialect에서 확인해야 한다. [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md); [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md)

```java
@Column(name = "unit_price")
private BigDecimal unitPrice;

@Column(name = "quantity")
private Integer quantity;

@Column(name = "line_total")
@GeneratedColumn("unit_price * quantity")
private BigDecimal lineTotal;
```

`lineTotal`은 애플리케이션이 직접 쓰는 값이 아니라 DB가 `unit_price`와 `quantity`에서 계산하는 값이다. Hibernate는 행을 넣거나 수정한 뒤 생성된 값을 다시 가져온다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md); [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)

## 언제 사용하는가

- **같은 행의 값에서 항상 다시 계산해야 할 때:** 길이 환산값이나 위 행 합계처럼 원본 열이 바뀌면 결과도 바뀌어야 하는 경우다. PostgreSQL의 생성 열은 원본 행이 바뀔 때 값을 다시 계산한다. [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)
- **DB가 계산한 값을 쓰기 직후 entity에서도 사용해야 할 때:** Hibernate가 `INSERT`와 `UPDATE` 뒤 생성값을 다시 읽는다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md)

PostgreSQL에서는 다른 테이블의 값, 하위 쿼리 또는 현재 시각처럼 변하는 함수에 의존하는 식을 생성 열에 쓸 수 없다. 이런 요구가 있다면 해당 DB의 생성 열이 맞는지 먼저 확인해야 한다. [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)

## 이름이 비슷한 주석과의 차이

| 주석 | 출처 | 목적 |
|---|---|---|
| `@GeneratedColumn` | Hibernate | `GENERATED ALWAYS AS` 계열의 DB 생성 열을 정의·매핑하고, 쓰기 뒤 계산된 값을 다시 가져온다. |
| `@Generated` | Hibernate | trigger나 DB 기본값 등 DB가 생성한 값을 일반적으로 매핑한다. 생성 시점을 지정할 수 있다. |
| `@GeneratedValue` | Jakarta Persistence | `@Id`와 함께 기본 키의 생성 전략을 지정한다. |

`@GeneratedColumn`의 계산식은 DB가 열 값을 생성하는 규칙이다. `@GeneratedValue`의 생성 전략은 entity의 기본 키를 만드는 규칙이므로 서로 바꿔 쓸 수 없다. `@Generated`는 DB가 값을 생성한다는 사실을 Hibernate에 알리는 범용 주석이며, Hibernate 문서는 `GENERATED ALWAYS AS` 열에는 `@GeneratedColumn`을 우선 사용하라고 안내한다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md); [Hibernate Generated Javadoc](../../raw/spring/hibernate-7-1-generated-javadoc.md); [Jakarta Persistence GeneratedValue Javadoc](../../raw/spring/jakarta-persistence-3-2-generatedvalue-javadoc.md)

## 적용 전 확인할 점

- **DB 계산식 제약:** PostgreSQL의 생성식은 현재 행만 참조할 수 있고, 불변 함수만 사용할 수 있으며, 다른 생성 열을 참조할 수 없다. 다른 DB의 규칙은 별도로 확인한다. [PostgreSQL 생성 열](../../raw/spring/2026-08-13-postgresql-18-generated-columns.md)
- **스키마 일치:** Hibernate가 DDL을 생성하지 않고 별도 migration으로 스키마를 관리한다면, entity의 계산식과 실제 DB 생성 열 정의를 맞춰야 한다. 이는 주석이 DDL 식을 지정한다는 설명에서 나오는 적용상 판단이다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md)
- **값을 다시 읽는 비용:** Hibernate는 쓰기 뒤 생성값을 가져온다. DB dialect의 `returning`이나 JDBC 기능을 사용할 수 없으면 별도 `select`가 필요할 수 있으므로, 성능이 중요한 쓰기 경로에서는 실제 SQL을 확인한다. [Hibernate GeneratedColumn Javadoc](../../raw/spring/hibernate-7-1-generatedcolumn-javadoc.md); [Hibernate Generated Javadoc](../../raw/spring/hibernate-7-1-generated-javadoc.md)

## See Also

- [Spring DB 접근 기술 비교](spring-database-access-technologies.md)
