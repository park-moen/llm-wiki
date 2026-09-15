# Spring MVC View 렌더링과 Thymeleaf

> Sources: 김영한, 2026-01-30
> Raw: [View 환경 설정: Welcome Page와 Thymeleaf](../../raw/spring/spring-view-environment-setup.md); [프로젝트 환경 설정: 스프링 부트 프로젝트 생성](../../raw/spring/project-setup-and-spring-initializr.md); [프로젝트 환경 설정 PDF companion](../../raw/spring/spring-project-environment-setup.md); [스프링 웹 개발 기초 PDF companion](../../raw/spring/spring-web-development-basics.md); [스프링 웹 개발 기초 - 정적 콘텐츠 강의](../../raw/spring/spring-web-basics-static-content-lecture.md); [MVC와 템플릿 엔진 강의](../../raw/spring/spring-mvc-template-engine-lecture.md)
> Updated: 2026-08-12

## Overview

Spring Boot의 첫 화면은 정적 `index.html`로 만들 수 있고, 동적으로 변하는 화면은 Controller가 Model에 data를 담아 View 이름을 반환한 뒤 View Resolver와 Thymeleaf가 HTML을 렌더링하는 방식으로 구성할 수 있다. 이 둘의 차이는 server가 저장된 파일을 그대로 전달하는지, 요청을 처리한 결과로 template을 변환해 전달하는지에 있다.

## Spring Web 응답 방식의 큰 분류

입문 강의는 웹 개발 방식을 다음 세 가지 큰 흐름으로 나눈다.

1. 정적 콘텐츠: server가 저장된 file을 별도 처리 없이 그대로 반환한다.
2. MVC와 template engine: Controller와 Model이 요청을 처리하고 template engine이 data를 반영한 HTML을 만들어 반환한다.
3. API: HTML 대신 data를 HTTP response body로 전달해 client가 직접 사용한다.

이 분류는 입문 단계에서 요청이 어떤 경로로 처리되고 최종 응답이 file, 렌더링된 HTML, data 중 무엇인지 구분하기 위한 큰 그림이다. API 방식의 처리와 사용 맥락은 [Spring MVC API 응답과 HttpMessageConverter](spring-mvc-api-response.md)에서 이어진다.

## 강의의 문서 버전 주의

영상은 Spring Boot 2.3.1 reference documentation을 기준으로 Welcome Page와 template engine 설정을 찾는다.

> **Status: Outdated** (2023-11-27)
> 강의 수정 이력에 따르면 `start.spring.io`의 Spring Boot 2.x 지원은 종료되었으며 Spring Boot 3.0 이상과 Java 17 이상을 사용해야 한다. 화면 구성의 개념은 유지하되, 세부 설정과 documentation 경로는 사용하는 Spring Boot version의 reference documentation에서 확인한다.

## 정적 Welcome Page

`src/main/resources/static/index.html`을 만들면 root domain에 접속했을 때 Welcome Page로 사용된다. 이 방식은 별도 program logic을 실행하지 않고 web server가 저장된 HTML file을 browser에 그대로 응답하는 정적 content이다.

강의 예제에서는 `index.html`에 `/hello`로 이동하는 link를 만들고 server를 다시 시작한 뒤 `localhost:8080`에서 첫 화면을 확인한다. 아직 `/hello` handler가 없다면 link를 눌렀을 때 error page가 나타나는 것이 정상이다.

Spring ecosystem은 기능이 방대하므로 동작을 모두 암기하기보다 공식 reference documentation에서 `index.html`, Welcome Page, static content location 같은 keyword로 필요한 규칙을 찾는 습관이 중요하다.

### 일반 정적 콘텐츠 요청

Welcome Page가 아닌 일반 정적 file도 `src/main/resources/static` 아래에 둘 수 있다. PDF 예제는 `hello-static.html`을 만들고 `localhost:8080/hello-static.html`로 요청한다.

요청이 들어오면 Spring container에서 관련 Controller를 먼저 찾고, mapping된 Controller가 없으면 static resource에서 같은 경로의 file을 찾는다. 발견한 file은 별도 template 처리 없이 browser에 반환된다.

## Thymeleaf를 이용한 동적 View

동적 화면 예제는 Controller와 template으로 나뉜다.

### MVC의 역할과 책임

과거의 Model 1 방식처럼 JSP 하나에서 화면 출력, DB 접근, Controller와 business logic까지 모두 처리하면 file이 커지고 유지보수하기 어려워진다. MVC는 관심사를 다음과 같이 분리한다.

- Model: View를 그리는 데 필요한 data를 담아 전달한다.
- View: 전달받은 data를 사용해 화면을 표시하는 데 집중한다.
- Controller와 business logic: 요청을 받고 내부 처리를 수행한 뒤 View에 필요한 data를 준비한다.

핵심은 file을 형식적으로 나누는 것이 아니라 화면 표현과 server 내부 처리의 책임을 분리하는 것이다.

### Controller

- Class에 `@Controller`를 선언한다.
- Method에 `@GetMapping("hello")`를 선언해 `/hello` 요청과 연결한다.
- Spring이 전달한 Model에 `addAttribute`로 key `data`와 value를 넣는다.
- View 이름으로 `hello`를 반환한다.

### Template

- `src/main/resources/templates/hello.html`을 만든다.
- Thymeleaf namespace를 선언한다.
- `th:text`에서 Model의 `${data}`를 참조한다.

Template에 적힌 고정 text는 Thymeleaf 처리 과정에서 Model의 value로 치환되고, 최종 HTML이 browser에 전달된다. Controller가 Model의 value를 바꾸면 같은 View에서도 다른 결과를 렌더링할 수 있다.

PDF의 `hello-mvc` 예제는 `@RequestParam("name")`으로 query parameter를 받아 Model의 `name` attribute에 넣고 View 이름 `hello-template`을 반환한다. `hello-mvc?name=spring`처럼 요청하면 Thymeleaf가 `${name}`을 사용해 HTML을 렌더링한다.

### `@RequestParam`의 필수 parameter

강의 예제에서 `@RequestParam`의 `required` option은 기본값이 `true`다. 따라서 `/hello-mvc`만 요청해 name을 생략하면 필수 parameter가 없다는 error가 발생한다. `/hello-mvc?name=spring`처럼 값을 전달하거나, 선택 parameter로 설계할 때는 `required = false`를 명시해야 한다.

### Thymeleaf의 HTML 미리보기

Thymeleaf template은 server를 거치지 않고 HTML file 자체를 열어도 `hello! empty` 같은 기본 text를 화면에서 확인할 수 있다. Server를 통해 template engine이 실행되면 `th:text`가 Model 값으로 이 기본 text를 치환한다. 따라서 markup 작업자는 정적 HTML 형태를 미리 볼 수 있고, 실제 실행에서는 동적 data가 반영된다.

## 요청에서 HTML 응답까지

`/hello` 요청의 처리 흐름은 다음과 같다.

1. Browser가 `localhost:8080/hello`로 HTTP GET 요청을 보낸다.
2. 내장 Tomcat이 요청을 받아 Spring에 전달한다.
3. Spring이 `@GetMapping("hello")`와 일치하는 Controller method를 실행한다.
4. Controller가 Model에 data를 넣고 View 이름 `hello`를 반환한다.
5. View Resolver가 View 이름에 해당하는 `templates/hello.html`을 찾는다.
6. Thymeleaf가 Model data를 template에 반영해 HTML을 렌더링한다.
7. 렌더링된 HTML이 browser에 응답된다.

핵심은 Controller의 문자열 반환값이 HTML body 자체가 아니라 View 이름이라는 점이다. Spring Boot의 기본 template 설정에 따라 View Resolver가 해당 이름을 resource의 template file과 연결한다.

## IntelliJ 단축키

### View 이름에서 template로 이동

강의에서는 Controller가 반환하는 View 이름 `hello`와 `templates/hello.html`이 연결된다는 점을 IntelliJ navigation 기능으로 보여준다. macOS에서 View 이름에 `Command` key를 사용하면 연결된 template 위치로 이동할 수 있다고 설명한다.

이 기능은 강의에서 IntelliJ Enterprise edition의 지원 기능으로 소개되며, 무료 edition에서는 지원되지 않을 수 있다. 강의는 이번 구간에서 Windows용 대응 단축키를 제시하지 않으므로 별도로 추정하지 않는다.

### Method parameter 정보 보기

MVC 강의에서는 annotation이나 method의 parameter option을 확인하는 Parameter Info 기능을 소개한다.

- macOS: `Command + P`
- Windows: 이번 강의 스크립트에는 실제 key 조합이 기록되어 있지 않으므로 추정하지 않는다. IntelliJ keymap에서 `Parameter Info` action을 조회한다.

예제에서는 `@RequestParam`에 cursor를 두고 이 기능으로 `required` 같은 option과 기본 동작을 확인한다.

## 개발 중 View 변경 반영

강의에서는 Spring Boot DevTools를 추가하면 HTML file을 compile한 뒤 server를 다시 시작하지 않고 View 변경을 반영할 수 있다고 안내한다. DevTools의 구체적인 설정과 현재 version에서의 동작은 사용하는 Spring Boot reference documentation을 기준으로 확인한다.

## See Also

- [Spring MVC API 응답과 HttpMessageConverter](spring-mvc-api-response.md)
- [Spring Boot 프로젝트 생성과 첫 실행](spring-boot-project-setup.md)
- [Spring Boot Starter와 의존성 구조](spring-boot-starter-dependencies.md)
- [Spring 회원 관리 웹 MVC](spring-member-web-mvc.md)
