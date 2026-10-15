# Spring Data JPA Service 트랜잭션 경계와 readOnly

> Sources: Spring Data JPA, Unknown; Spring Framework, Unknown
> Raw: [Spring Data Transactionality](../../raw/spring/spring-data-jpa-transactions-reference.md); [Spring Transactional](../../raw/spring/spring-framework-transaction-annotations-reference.md); [Spring rollback 규칙](../../raw/spring/spring-framework-transaction-rollback-reference.md)
> Updated: 2026-10-15

## Overview

트랜잭션은 개별 SQL이나 Repository 메서드가 아니라 **업무상 함께 성공하거나 취소되어야 하는 작업 단위**로 잡는다. 여러 Repository를 호출하는 Service에 경계를 두면 객체 수정과 DB 작업이 같은 트랜잭션에 참여하게 할 수 있다. `@Transactional`을 처음 사용하는 예제는 기존 [DB 접근 기술 비교](spring-database-access-technologies.md)에 있으며, 이 문서는 경계와 예외 동작을 다룬다. [Spring Data Transactionality](../../raw/spring/spring-data-jpa-transactions-reference.md)

## Repository 기본 설정이 모든 작업을 묶지는 않는다

`CrudRepository`에서 상속한 조회 작업에는 기본적으로 read-only 설정이, 그 밖의 기본 변경 작업에는 일반 트랜잭션 설정이 적용된다. 그러나 직접 선언한 조회 메서드에는 같은 설정이 자동으로 적용되는 것이 아니다. 여러 메서드를 이어 호출한다고 저절로 하나의 업무 트랜잭션이 되지도 않는다. [Spring Data Transactionality](../../raw/spring/spring-data-jpa-transactions-reference.md)

```kotlin
@Service
class MemberService(private val members: MemberRepository) {
    @Transactional
    fun rename(id: Long, name: String) {
        val member = members.findById(id).orElseThrow()
        member.name = name
    }

    @Transactional(readOnly = true)
    fun find(id: Long): MemberSummaryDto {
        val member = members.findById(id).orElseThrow()
        return MemberSummaryDto(member.name, member.email)
    }
}
```

[공통 Member](jpa-persistence-context-and-entity-lifecycle.md)와 [조회 DTO](spring-data-jpa-projections-and-dtos.md)를 사용하는 학습용 코드다. 수정 메서드는 쓰기 트랜잭션에서 관리 객체를 수정한다. 외부 Service 트랜잭션에 참여하는 Repository 호출에서는 그 외부 경계의 설정이 중요하다. [Spring Data Transactionality](../../raw/spring/spring-data-jpa-transactions-reference.md)

## readOnly는 쓰기 금지 검증기가 아니다

`readOnly = true`는 JDBC driver와 JPA 구현체에 최적화 힌트를 전달한다. DB마다 다르지만 모든 쓰기를 반드시 예외로 막는 장치라고 보면 안 된다. Hibernate에서는 flush mode·변경 감지 관련 최적화가 적용될 수 있으므로, 이 경계 안에서 엔티티를 바꾼 뒤 자동 저장을 기대하지 않는다. [Spring Data Transactionality](../../raw/spring/spring-data-jpa-transactions-reference.md)

## 같은 객체 안의 함수 호출에 주의하기

기본 proxy 방식에서는 Spring proxy를 통과하는 외부 호출을 가로챈다. 같은 인스턴스의 함수가 다른 `@Transactional` 함수를 직접 호출하는 self-invocation만으로는 그 주석의 새 경계가 적용되지 않는다. 이미 바깥 트랜잭션이 있다면 그 안에서 실행될 수는 있지만, 내부 함수의 설정을 새로 적용한 결과는 아니다. [Spring Transactional](../../raw/spring/spring-framework-transaction-annotations-reference.md)

Kotlin에서는 Spring proxy가 적용될 수 있도록 프로젝트의 `kotlin-spring` 설정이나 클래스·메서드의 확장 가능성도 확인한다. 기본 설정 안내는 기존 [Kotlin JPA 설정](spring-database-access-technologies.md)에 연결한다.

## 예외와 rollback

기본 rollback 규칙은 `RuntimeException`과 `Error`를 대상으로 하며 checked exception은 기본적으로 rollback 대상이 아니다. 프로젝트가 전역 rollback 정책이나 `rollbackFor`를 설정했다면 그 설정이 달라질 수 있다. Kotlin에서 checked exception을 문법적으로 강제하지 않아도 Spring이 예외 타입으로 판단한다는 점은 그대로다. [Spring rollback 규칙](../../raw/spring/spring-framework-transaction-rollback-reference.md); [Spring Transactional](../../raw/spring/spring-framework-transaction-annotations-reference.md)

예외를 내부에서 잡고 정상 반환하면 proxy에 예외가 전달되지 않을 수 있다. 이때도 rollback-only로 이미 표시됐는지 등에 따라 결과가 달라지므로, 예외를 catch했다는 이유만으로 항상 commit된다고 판단하지 않는다. 이는 공식 rollback 처리 구조에서 도출한 주의점이다. [Spring rollback 규칙](../../raw/spring/spring-framework-transaction-rollback-reference.md)

## 직접 확인하는 연습

쓰기 Service에서 이름을 바꾼 뒤 unchecked 예외를 던지고, 호출이 끝난 뒤 새 트랜잭션에서 값을 확인한다. 테스트 자체의 rollback이 결과를 가리지 않게 구분한다. 다음으로 같은 클래스 내부 호출과 다른 Bean을 통한 호출을 비교하여 실제 트랜잭션 경계가 어디인지 설명한다.

## See Also

- 이전: [Projection과 DTO](spring-data-jpa-projections-and-dtos.md)
- 다음: [동적 검색 조건](spring-data-jpa-dynamic-search.md)
- [변경 감지와 flush](jpa-dirty-checking-and-flush.md)
