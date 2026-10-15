# Spring Data JPA Auditing으로 생성·수정 정보 기록

> Sources: Spring Data JPA, Unknown
> Raw: [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md); [Spring Data bulk 변경과 callback](../../raw/spring/spring-data-jpa-query-methods-reference.md)
> Updated: 2026-10-15

## Overview

Auditing은 엔티티를 누가 언제 만들고 바꿨는지 기록하는 기능이다. 날짜를 각 Service에서 반복 대입하는 대신 auditing metadata와 listener를 구성한다. **일반적인 생성·수정 정보 기록과 변경 이력 전체를 저장하는 기능은 구분한다.** [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md)

## 날짜만 기록하는 Kotlin 예제

아래는 일반 클래스에 직접 생성자와 `open`을 적은 학습용 엔티티다. JPA 의존성 외에 공식 문서가 안내하는 `spring-aspects`가 classpath에 있는지 확인한다. listener는 엔티티별로 등록하거나 전역 `orm.xml`에서 등록하는 방식을 선택한다. 이 예제는 엔티티별 등록을 사용한다. [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md)

```kotlin
@Configuration
@EnableJpaAuditing
class JpaAuditConfiguration

@Entity
@EntityListeners(AuditingEntityListener::class)
open class AuditNote protected constructor() {
    @field:Id
    @field:GeneratedValue
    open var id: Long? = null
        protected set

    open var title: String = ""

    @field:CreatedDate
    open var createdAt: Instant? = null
        protected set

    @field:LastModifiedDate
    open var updatedAt: Instant? = null
        protected set

    constructor(title: String) : this() {
        this.title = title
    }
}
```

`CreatedDate`·`LastModifiedDate`는 `org.springframework.data.annotation`, `EnableJpaAuditing`은 `org.springframework.data.jpa.repository.config`, `AuditingEntityListener`는 `org.springframework.data.jpa.domain.support`에서 가져온다. `Instant`는 `java.time.Instant`다. 날짜만 기록하면 `AuditorAware` 구현은 필요하지 않다. [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md)

## 작업자까지 기록하기

`@CreatedBy`·`@LastModifiedBy`를 사용하면 현재 작업자를 제공하는 `AuditorAware<T>`를 구현한다. `T`는 기록할 프로퍼티 타입과 맞춘다. 인증된 사용자 ID를 기록한다면 애플리케이션의 인증 정보를 읽어 그 ID를 반환하도록 구성한다. 공식 문서의 Spring Security principal 예제를 애플리케이션 타입에 맞춰야 한다. [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md)

인증이 없는 배치 작업에서는 어떤 작업자를 반환할지도 별도로 정해야 한다. 이는 `AuditorAware`가 현재 사용자 또는 시스템을 알려 주는 SPI라는 설명에서 도출한 설계 질문이다. [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md)

## 어디까지 자동으로 기록되는가

Auditing listener는 엔티티 생명주기 흐름에서 동작한다. bulk 변경 쿼리는 같은 엔티티 callback 경로를 거치지 않으므로, bulk UPDATE의 수정 날짜도 항상 자동으로 채워진다고 기대하지 않는다. 별도 SQL로 바꾼 행까지 listener가 감지하는 기능도 아니다. 날짜가 필요한 bulk 작업은 쿼리에 포함하는 등 별도 정책을 정한다. [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md); [Spring Data bulk 변경과 callback](../../raw/spring/spring-data-jpa-query-methods-reference.md)

생성일·최종 수정일 필드는 변경 전 값을 모두 보관하지 않는다. 누가 어떤 필드를 어떤 값에서 바꿨는지까지 필요하다면 변경 이력 저장을 별도 요구사항으로 다룬다. 이는 최종 metadata 필드만 기록하는 구조에서 도출한 구분이다. [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md)

## 직접 확인하는 연습

새 노트를 저장하고 flush한 뒤 생성·수정 날짜가 채워지는지 확인한다. 새 트랜잭션에서 제목을 바꾸고 다시 조회하여 생성일은 유지되고 수정일이 갱신되는지 살펴본다. 시간 비교를 안정적으로 검증해야 한다면 공식 확장 지점인 `DateTimeProvider`를 검토한다. [Spring Data Auditing](../../raw/spring/spring-data-jpa-auditing-reference.md)

## See Also

- 이전: [사용자 정의 Repository](spring-data-jpa-custom-repository-fragments.md)
- 다음 심화: [낙관적 락과 비관적 락](../database/optimistic-vs-pessimistic-locking.md)
- [학습 순서로 돌아가기](jpa-learning-roadmap.md)
