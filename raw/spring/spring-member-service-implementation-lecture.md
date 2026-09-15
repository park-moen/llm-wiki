# 회원 Service 구현

> Source: Inflearn 김영한 「스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술」 강의 스크립트
> Collected: 2026-08-12
> Published: Unknown

네 여러분 이제 이어서 회원 Service class를 만들어 보겠습니다. 이 회원 Service는 회원 Repository와 Domain을 활용해서 실제 business logic을 작성하는 쪽입니다. 먼저 `service` package를 만들고 `MemberService` class를 만듭니다.

회원 Service를 만들려면 회원 Repository가 있어야 됩니다. `MemberRepository memberRepository = new MemoryMemberRepository()`와 같이 만들 수 있습니다.

회원 가입부터 만듭니다. `public long join(Member member)`로 만들고, 회원 가입을 하면 저장된 회원의 id를 반환하도록 spec을 잡았습니다.

회원 가입 business rule로 같은 이름이 있는 중복 회원은 허용하지 않습니다. `memberRepository.findByName(member.getName())`으로 같은 이름의 회원을 찾습니다.

이때 강의에서 소개한 IntelliJ 단축키 `Command + Option + V`를 사용하면 method return 값을 local variable로 만들 수 있습니다. 조회 결과는 `Optional<Member>`입니다.

Optional에 값이 있으면 `ifPresent`가 실행됩니다. 이미 같은 이름의 Member가 있으면 `IllegalStateException`을 발생시키고 message는 `이미 존재하는 회원입니다.`로 합니다.

과거에는 null인지 직접 확인했지만 null일 가능성이 있으면 Optional로 감싸서 반환하고 `ifPresent` 같은 method를 사용할 수 있습니다. 그냥 꺼내고 싶으면 `get`을 사용할 수 있지만 직접 바로 꺼내는 것은 권장하지 않습니다. 값이 있으면 꺼내고 없으면 method를 실행해 default 값을 얻는 `orElseGet`도 많이 사용합니다.

Optional을 local variable로 바로 반환받아 사용하는 code보다 `findByName` 호출 뒤에 바로 `ifPresent`를 연결할 수 있습니다.

```java
memberRepository.findByName(member.getName())
        .ifPresent(m -> {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        });
```

`findByName` 뒤로 검증 logic이 이어지는 경우 method로 추출하는 것이 좋습니다. `Ctrl + T`를 누르면 refactoring 관련 menu가 나오고 `Extract Method`를 선택할 수 있습니다. 정확한 macOS 단축키는 `Command + Option + M`입니다. Windows는 다른 단축키일 수 있다고 안내합니다.

추출한 method 이름은 `validateDuplicateMember`로 합니다. `join`을 읽으면 중복 회원을 검증하고 통과하면 저장한다는 흐름을 바로 이해할 수 있습니다.

전체 회원 조회는 `findMembers`로 만들고 `memberRepository.findAll()`을 반환합니다. 회원 한 명 조회는 `findOne(Long memberId)`로 만들고 `memberRepository.findById(memberId)`를 반환합니다.

Repository는 `save`, `findById`, `findByName`, `findAll`처럼 data를 넣고 꺼내는 기술적인 용어를 사용합니다. Service class는 `join`, `findMembers`처럼 business에 가까운 용어를 사용합니다. Service는 business를 처리하고 Repository는 data를 저장하고 조회하는 역할에 맞춰 이름을 정합니다.

회원 가입에서 중복 회원이면 exception이 발생하는지 검증하려면 main method나 Controller로 직접 넣어보는 방법도 있지만, 이전에 배운 test case를 활용하는 방법이 가장 좋습니다. 다음 시간에는 회원 Service test를 작성합니다. 감사합니다.
