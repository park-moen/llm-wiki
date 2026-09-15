# Spring 회원 관리 웹 MVC

> Sources: 김영한, 2026-01-30
> Raw: [회원 관리 예제 - 웹 MVC 개발 PDF companion](../../raw/spring/spring-member-web-mvc.md); [회원 관리 Web MVC 회원 등록 강의 원문](../../raw/spring/spring-member-web-mvc-registration-lecture.md); [회원 관리 Web MVC 회원 목록 조회 강의 원문](../../raw/spring/spring-member-web-mvc-list-lecture.md)
> Updated: 2026-08-13

## Overview

회원 관리 Web MVC 예제는 home에서 회원 가입 Form과 회원 목록으로 이동하고, Controller가 GET·POST 요청을 구분해 Service를 호출한 뒤 Thymeleaf View를 렌더링한다. 입력 Form의 field는 form object에 binding되고, 조회 결과는 Model을 통해 template로 전달된다.

## Home 화면과 경로

`HomeController`는 GET `/` 요청에 `home` View를 반환한다. `home.html`은 다음 두 기능으로 연결된다.

- `/members/new`: 회원 가입 Form
- `/members`: 회원 목록

Spring MVC가 `/`를 처리하는 Controller를 찾으면 정적 Welcome Page보다 Controller의 mapping이 우선한다.

## 회원 가입: GET과 POST 분리

GET `/members/new`는 입력 화면인 `members/createMemberForm`을 반환한다. Form의 `name` attribute는 제출 시 `MemberForm.name`에 binding된다.

POST `/members/new`는 binding된 `MemberForm`에서 name을 꺼내 `Member`를 만들고 `memberService.join`으로 등록한다. 처리가 끝나면 `redirect:/`로 home에 돌아간다. 같은 URL이라도 GET은 화면 조회, POST는 data 제출·변경이라는 역할로 구분된다.

### Form field와 객체 binding

회원 등록 Form의 핵심 계약은 action·HTTP method·field name이다.

```html
<form action="/members/new" method="post">
    <input type="text" id="name" name="name" placeholder="이름을 입력하세요">
    <button type="submit">등록</button>
</form>
```

`name="name"`은 server로 전달되는 parameter의 key다. 사용자가 값을 입력하고 제출하면 Spring은 POST handler의 `MemberForm` 객체에 있는 `setName(...)`을 호출해 그 값을 넣는다. Controller는 `form.getName()`으로 값을 꺼내 새 `Member`에 설정한다.

```java
@PostMapping("/members/new")
public String create(MemberForm form) {
    Member member = new Member();
    member.setName(form.getName());
    memberService.join(member);
    return "redirect:/";
}
```

전체 요청 흐름은 다음과 같다.

1. Browser가 GET `/members/new`를 요청한다.
2. Controller가 `members/createMemberForm` View 이름을 반환하고 View Resolver와 Thymeleaf가 Form HTML을 렌더링한다.
3. 사용자가 Form을 제출하면 `action`에 지정된 `/members/new`로 POST 요청이 전송된다.
4. Spring이 request parameter `name`을 `MemberForm.name`에 binding한다.
5. Controller가 `Member`를 만들고 Service의 `join`을 호출한다.
6. 등록을 마치면 `redirect:/` 응답으로 home을 다시 요청하게 한다.

`id="name"`은 HTML element 식별에 쓰이고, server-side binding에서 핵심이 되는 값은 `name="name"`이다. `Member`를 생성할 때는 이름이 같은 다른 class가 아니라 예제에서 정의한 domain class가 import되었는지도 확인해야 한다.

## 회원 목록 렌더링

GET `/members`는 Service에서 회원 전체를 조회하고 Model의 `members` attribute에 담은 뒤 `members/memberList` View를 반환한다. Thymeleaf는 다음 표현으로 collection을 HTML table에 펼친다.

- `th:each="member : ${members}"`: 회원 반복
- `th:text="${member.id}"`: id 출력
- `th:text="${member.name}"`: 이름 출력

Controller의 핵심 처리는 다음과 같다.

```java
@GetMapping("/members")
public String list(Model model) {
    List<Member> members = memberService.findMembers();
    model.addAttribute("members", members);
    return "members/memberList";
}
```

`th:each`는 Model에서 `${members}` collection을 꺼내 Java의 for-each처럼 순회한다. 반복마다 현재 객체를 `member`에 담고 `member.id`, `member.name`을 평가해 table row를 만든다. HTML source에는 template 표현식 자체가 아니라 반복 결과가 반영된 최종 row가 나타난다.

### Thymeleaf의 Java property 접근

`Member`의 `id`와 `name` field가 `private`이어도 `${member.id}`, `${member.name}`으로 읽을 수 있다. Thymeleaf가 Java property 접근 방식에 따라 각각 `getId()`, `getName()`을 호출하기 때문이다. 따라서 template의 property 표기와 Java의 public getter가 연결된다.

### Memory repository의 생명주기

현재 예제의 회원 데이터는 memory에만 저장된다. Application process를 종료하고 server를 다시 시작하면 기존 회원 목록이 사라진다. 실행을 넘어 데이터를 유지해야 하는 실제 application에서는 file이나 database 같은 영속 저장소가 필요하다.

따라서 Controller는 요청과 use case를 조정하고, Service는 business logic을, View는 표시를 담당한다.

## IntelliJ IDEA 단축키

| 환경 | 단축키 | 기능 | 강의에서의 사용 맥락 |
|---|---|---|---|
| macOS | `Command + E` | 최근에 열어본 파일 목록 표시 | Template를 살펴보다 Controller의 `model.addAttribute("members", ...)` 위치로 돌아갈 때 사용 |

이번 강의 원문에는 Windows/Linux 대응 단축키가 제시되지 않았다. Thymeleaf 관련 IDE 탐색 지원은 강의에서 IntelliJ 유료 버전 기능으로 안내되며 무료 버전에서는 제한될 수 있다.

## See Also

- [Spring MVC View 렌더링과 Thymeleaf](spring-mvc-view-rendering.md)
- [Spring 회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
