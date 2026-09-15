# 회원 Domain과 Repository 구현

> Source: Inflearn 김영한 「스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술」 강의 스크립트
> Collected: 2026-08-12
> Published: Unknown

네 이제 앞에 설명드렸던 내용을 바탕으로 회원 Domain과 Repository를 실제 coding해 볼게요.

먼저 `domain` package를 만들고 `Member` class를 만들 겁니다. 요구사항에는 id 식별자와 이름이라는 두 가지가 있습니다.

id는 고객이 정하는 id가 아니라 database나 data 저장소에서 sequence로 증가하는 등 system이 저장하고 data를 구분하기 위해 정하는 id입니다. Name은 회원의 이름입니다. 여기서는 가장 쉬운 예제로 Getter와 Setter를 모두 만들겠습니다.

그다음 `repository` package에 `MemberRepository` interface를 만듭니다. Repository는 저장소라는 뜻이고 회원 객체를 저장하는 저장소를 만들 겁니다.

Repository에는 네 가지 기능을 만듭니다.

- `save`: 회원을 저장하고 저장된 회원을 반환합니다.
- `findById`: id로 회원을 찾습니다.
- `findByName`: name으로 회원을 찾습니다.
- `findAll`: 지금까지 저장된 모든 회원 list를 반환합니다.

`findById`와 `findByName`은 `Optional<Member>`를 반환합니다. 조회 결과가 없으면 null일 수 있는데, 요즘에는 null을 그대로 반환하는 대신 Java 8에 들어간 `Optional`로 감싸서 반환하는 방법을 많이 선호합니다.

구현체로 `MemoryMemberRepository` class를 만들고 `MemberRepository` interface를 `implements`합니다. 여기서 `Option + Enter`를 누르면 Implement Methods를 실행할 수 있고, method를 전부 선택해 생성합니다.

Memory 저장소는 `Map<Long, Member>`를 사용하고 이름은 `store`로 합니다. Import가 필요할 때 강의에서는 `Ctrl + Space`나 `Option + Enter`를 사용합니다.

예제에서는 `new HashMap`을 사용합니다. 실무에서 공유되는 변수는 동시성 문제가 있을 수 있어 `ConcurrentHashMap`을 사용해야 하지만 여기서는 단순한 예제를 위해 `HashMap`을 사용합니다.

`Long sequence`도 만듭니다. Sequence는 0, 1, 2처럼 key 값을 생성합니다. 실무에서는 단순 `long`보다 동시성 문제를 고려해 `AtomicLong` 등을 사용해야 하지만 여기서는 단순하게 구현합니다.

`save`할 때 sequence 값을 하나 증가시키고 `member.setId`로 id를 설정합니다. Name은 회원 가입 시 사용자가 입력해 이미 넘어온 상태입니다. System이 id를 정한 뒤 `store.put(member.getId(), member)`로 저장하고 저장된 Member를 반환합니다.

`findById`는 `store.get(id)`로 찾습니다. 결과가 없으면 null일 수 있으므로 `Optional.ofNullable`로 감싸서 반환합니다. 그러면 client가 Optional을 사용해 조회 결과가 없을 때를 처리할 수 있습니다.

`findByName`은 Java 8 lambda를 사용합니다. `store.values()`를 순회하고 `member.getName()`이 parameter의 name과 같은지 filter한 다음 `findAny()`로 하나를 반환합니다. 결과는 Optional입니다.

`findAll`은 Map의 `store.values()`를 `new ArrayList`로 만들어 반환합니다. 실무에서는 순회하기 편한 List를 많이 사용한다고 설명합니다.

구현이 끝나면 이 code가 실제로 정상 동작하는지 test case로 검증해야 합니다. 다음 시간에는 회원 Repository test code를 작성합니다. 감사합니다.
