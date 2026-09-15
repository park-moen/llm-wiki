# 회원 관리 예제 - 웹 MVC 개발

> Source: [강의 참고 PDF](spring-member-web-mvc.pdf)
> Collected: 2026-08-12
> Published: 2026-01-30

이 문서는 같은 디렉터리의 원본 PDF를 검색하고 근거 검사에 사용할 수 있도록 옮긴 companion text다.

## 홈 화면

- `HomeController`의 `@GetMapping("/")` method는 `home` View를 반환한다.
- `home.html`은 `/members/new` 회원 가입과 `/members` 회원 목록 link를 제공한다.

## 회원 등록

- GET `/members/new`: `MemberController.createForm()`이 `members/createMemberForm`을 반환한다.
- Form의 `<input type="text" id="name" name="name">` 값이 `MemberForm.name`에 binding된다.
- POST `/members/new`: `MemberController.create(MemberForm form)`이 `Member`를 만들고 `memberService.join(member)`을 호출한 뒤 `redirect:/`를 반환한다.
- GET은 조회, POST는 data 전달·등록에 사용한다는 HTTP method 구분을 설명한다.

## 회원 조회

- GET `/members`: `memberService.findMembers()` 결과를 Model의 `members` attribute에 넣고 `members/memberList` View를 반환한다.
- Thymeleaf template은 `th:each="member : ${members}"`로 목록을 순회하고 `th:text="${member.id}"`, `th:text="${member.name}"`으로 출력한다.
