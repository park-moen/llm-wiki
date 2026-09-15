# 스프링 웹 개발 기초

> Source: [강의 참고 PDF](spring-web-development-basics.pdf)
> Collected: 2026-08-12
> Published: Unknown

이 문서는 같은 디렉터리의 원본 PDF를 검색하고 근거 검사에 사용할 수 있도록 옮긴 companion text다. Page 순서, code, URL과 diagram의 연결 관계를 보존했다.

## Page 1: 스프링 웹 개발 기초

- 정적 콘텐츠
- MVC와 템플릿 엔진
- API

### 정적 콘텐츠

스프링 부트 정적 콘텐츠 기능

https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content

`resources/static/hello-static.html`

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>static content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
정적 콘텐츠 입니다.
</body>
</html>
```

실행: http://localhost:8080/hello-static.html

## Page 2: 정적 콘텐츠 처리와 MVC

### 정적 콘텐츠 처리

Web browser가 `localhost:8080/hello-static.html`을 요청한다. 내장 Tomcat server가 요청을 받고, Spring container에서 `hello-static` 관련 Controller를 먼저 찾는다. 관련 Controller가 없으면 `resources/static/hello-static.html`을 찾아 browser에 반환한다.

### MVC와 템플릿 엔진

MVC: Model, View, Controller

Controller:

```java
@Controller
public class HelloController {

    @GetMapping("hello-mvc")
    public String helloMvc(@RequestParam("name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello-template";
    }
}
```

View: `resources/templates/hello-template.html`

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
```

## Page 3: MVC 처리와 문자 API

```html
</body>
</html>
```

실행: http://localhost:8080/hello-mvc?name=spring

MVC와 템플릿 엔진의 처리 흐름:

1. Web browser가 `localhost:8080/hello-mvc`를 요청한다.
2. 내장 Tomcat server가 요청을 Spring container로 전달한다.
3. `helloController`가 View 이름 `hello-template`과 Model `name:spring`을 반환한다.
4. `viewResolver`가 `templates/hello-template.html`을 찾는다.
5. Thymeleaf template engine이 처리한 HTML을 browser에 반환한다.

### API: @ResponseBody 문자 반환

```java
@Controller
public class HelloController {

    @GetMapping("hello-string")
    @ResponseBody
    public String helloString(@RequestParam("name") String name) {
        return "hello " + name;
    }
}
```

## Page 4: @ResponseBody 문자·객체 반환

`@ResponseBody`를 사용하면 View Resolver를 사용하지 않는다. 대신 HTTP body에 문자 내용을 직접 반환한다. 여기서 body는 HTML body tag를 의미하지 않는다.

실행: http://localhost:8080/hello-string?name=spring

### @ResponseBody 객체 반환

```java
@Controller
public class HelloController {

    @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

`@ResponseBody`를 사용하고 객체를 반환하면 객체가 JSON으로 변환된다.

실행: http://localhost:8080/hello-api?name=spring

## Page 5: @ResponseBody 사용 원리

Web browser가 `localhost:8080/hello-api`를 요청하면 내장 Tomcat server와 `helloController`를 거쳐 `@ResponseBody`가 적용된 `hello(name:spring)` 반환값을 처리한다. View Resolver 대신 `HttpMessageConverter`가 동작해 `{name: spring}` 형식의 응답을 browser에 전달한다.

- HTTP body에 문자 내용을 직접 반환한다.
- View Resolver 대신 `HttpMessageConverter`가 동작한다.
- 기본 문자 처리: `StringHttpMessageConverter`
- 기본 객체 처리: `MappingJackson2HttpMessageConverter`
- Byte 처리 등 기타 여러 `HttpMessageConverter`가 기본으로 등록되어 있다.
- Client의 HTTP Accept header와 server Controller의 반환 type 정보를 조합해 `HttpMessageConverter`가 선택된다.
