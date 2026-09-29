# JPA `@ElementCollection`

> Sources: Jakarta Persistence, Unknown
> Raw: [Jakarta Persistence ElementCollection Javadoc](../../raw/spring/jakarta-persistence-3-2-elementcollection-javadoc.md)
> Updated: 2026-09-29

## Overview

`@ElementCollection`은 기본 타입이나 embeddable 객체의 컬렉션을 매핑할 때 사용하는 Jakarta Persistence 주석이다. 컬렉션 테이블을 이용해 매핑하려면 이 주석을 지정해야 한다. `@CollectionTable`은 그 컬렉션을 저장할 DB 테이블의 매핑을 지정한다.

## 매핑 대상과 예시

공식 문서는 `Set<String>` 필드에 `@ElementCollection`을 붙인 `Person.nickNames` 예시를 제시한다. 아래는 해당 예시에서 매핑에 필요한 부분만 추린 것이다.

```java
@Entity
public class Person {
    @Id
    protected String ssn;

    @ElementCollection
    protected Set<String> nickNames = new HashSet<>();
}
```

원소는 기본 타입 또는 embeddable 클래스다. 이 주석의 `targetClass`는 원소 타입을 지정하며, Java generics로 컬렉션의 원소 타입을 선언했다면 생략할 수 있다. generics를 사용하지 않았다면 `targetClass`를 지정해야 한다.

### embeddable 객체를 여러 개 담는 경우

주소처럼 여러 필드가 한 값을 이루고, 한 사람에게 그런 값이 여러 개 있을 때도 사용할 수 있다. 다음은 embeddable 클래스의 컬렉션을 매핑할 수 있다는 공식 문서의 설명에 따라 만든 **설계 예시**다. 실제 테이블·열 이름은 이 예시에서 지정하지 않았다.

```java
@Embeddable
public class Address {
    protected String city;
    protected String street;
}

@Entity
public class Person {
    @Id
    protected String ssn;

    @ElementCollection
    protected List<Address> addresses = new ArrayList<>();
}
```

`Set<String>` 별명 예시는 기본 타입의 모음, `List<Address>` 예시는 embeddable 값의 모음에 해당한다. 둘 다 `@ElementCollection`의 매핑 대상이다. [Jakarta Persistence ElementCollection Javadoc](../../raw/spring/jakarta-persistence-3-2-elementcollection-javadoc.md)

## 언제 사용하는가

- **한 entity에 기본 타입 값이 여러 개 필요할 때:** `Person.nickNames`처럼 별명을 문자열 값의 모음으로 저장하는 경우다. 공식 Javadoc의 예시다. [Jakarta Persistence ElementCollection Javadoc](../../raw/spring/jakarta-persistence-3-2-elementcollection-javadoc.md)
- **한 entity에 여러 embeddable 값이 필요할 때:** 위 `Person.addresses`처럼 주소를 값의 모음으로 모델링하는 경우다. 이는 Javadoc에 명시된 매핑 대상에서 만든 설계 예시다. [Jakarta Persistence ElementCollection Javadoc](../../raw/spring/jakarta-persistence-3-2-elementcollection-javadoc.md)

컬렉션 원소가 기본 타입이나 embeddable 클래스가 아니라 별도 entity라면 이 주석의 매핑 대상이 아니다. 원소를 어떤 타입으로 모델링할지 먼저 정한 뒤 주석을 선택한다. [Jakarta Persistence ElementCollection Javadoc](../../raw/spring/jakarta-persistence-3-2-elementcollection-javadoc.md)

## 조회 전략

`fetch`의 기본값은 `LAZY`다. Jakarta Persistence 문서에서 `LAZY`는 persistence provider에 대한 힌트이며, `EAGER`는 즉시 가져오도록 요구하는 설정이다. 따라서 `LAZY`를 선언하거나 기본값으로 두었다는 사실만으로 모든 실행 경로에서 지연 로딩이 보장된다고 가정하지 않는다.
