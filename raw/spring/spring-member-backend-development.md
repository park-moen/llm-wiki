# 회원 관리 예제 - 백엔드 개발

> Source: [강의 참고 PDF](spring-member-backend-development.pdf)
> Collected: 2026-08-12
> Published: 2026-01-30

이 문서는 같은 디렉터리의 원본 PDF를 검색하고 근거 검사에 사용할 수 있도록 옮긴 companion text다. 설계, 주요 API와 테스트 원칙을 보존했다.

## 구조와 도메인

- 일반적인 웹 애플리케이션 계층: Controller → Service → Repository → DB
- Domain 객체는 여러 계층에서 사용된다.
- 예제 `Member`는 `Long id`와 `String name`을 가진다.
- `MemberRepository`는 `save`, `findById`, `findByName`, `findAll`을 정의한다.
- `MemoryMemberRepository`는 `Map<Long, Member>`와 증가하는 `sequence`를 사용한다.
- 실무 동시성 환경에서는 `HashMap`과 단순 `long` 대신 `ConcurrentHashMap`과 `AtomicLong` 같은 동시성 대응이 필요하다는 주의가 있다.

## Repository 테스트

- JUnit test는 `save`, `findByName`, `findAll` 동작을 검증한다.
- AssertJ의 `assertThat(actual).isEqualTo(expected)`를 사용한다.
- test 실행 순서는 보장되지 않으므로 공유 저장소 상태를 각 test 뒤에 비운다.

```java
@AfterEach
public void afterEach() {
    repository.clearStore();
}
```

테스트는 서로 의존하지 않고 독립적으로 실행되어야 한다.

## Service와 테스트

- `MemberService.join`은 같은 이름의 회원을 확인하고 중복이면 `IllegalStateException`을 발생시킨 뒤 저장한다.
- `findMembers`와 `findOne`은 Repository 조회를 위임한다.
- Service의 method 이름은 business 용어를, Repository는 data access 용어를 사용하는 차이를 설명한다.
- `MemberService`는 constructor로 `MemberRepository`를 전달받아 같은 Repository instance를 공유한다.
- `@BeforeEach`에서 `MemoryMemberRepository`와 `MemberService`를 새로 만들고 `@AfterEach`에서 저장소를 비운다.
- 중복 회원 test는 `assertThrows(IllegalStateException.class, () -> memberService.join(member2))`와 exception message를 검증한다.
