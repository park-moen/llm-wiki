# Spring MVC API 응답과 HttpMessageConverter

> Sources: 김영한, Unknown
> Raw: [스프링 웹 개발 기초 PDF companion](../../raw/spring/spring-web-development-basics.md); [스프링 웹 개발 기초 - 정적 콘텐츠 강의](../../raw/spring/spring-web-basics-static-content-lecture.md); [Spring Web API 방식 강의](../../raw/spring/spring-mvc-api-response-lecture.md)
> Updated: 2026-08-12

## Overview

Spring MVC Controller에서 `@ResponseBody`를 사용하면 View 이름을 반환하고 template을 렌더링하는 대신 method 반환값을 HTTP response body에 직접 기록한다. 문자열은 문자 converter가 처리하고 객체는 JSON converter가 처리하며, Spring은 요청의 Accept header와 Controller 반환 type을 바탕으로 적절한 `HttpMessageConverter`를 선택한다.

## View 응답과 API 응답의 차이

일반 MVC View 방식에서는 Controller가 View 이름을 반환하고 View Resolver가 template을 찾아 HTML을 렌더링한다. `@ResponseBody` 방식은 View Resolver를 거치지 않고 반환값을 HTTP body에 쓴다. 여기서 HTTP body는 HTML의 body tag와 다른 개념이다.

HTTP message는 header와 body로 구성된다. `@ResponseBody`는 Controller 반환값을 HTML의 `<body>` element에 넣는다는 뜻이 아니라 HTTP response의 body 영역에 직접 기록하도록 Spring MVC에 지시한다.

## API 방식이 사용되는 맥락

강의는 API 방식을 HTML 화면 대신 data가 필요한 client와 통신하는 방식으로 소개한다. 대표적인 예는 다음과 같다.

- Android나 iPhone application처럼 server에서 화면이 아닌 data를 받아야 하는 client
- Vue·React처럼 server에서 받은 data로 browser 화면을 구성하는 client-side application
- HTML을 주고받을 필요 없이 data 교환이 중요한 server 간 통신

과거에는 XML도 사용했지만 강의 예제는 객체를 JSON data 형식으로 변환하는 현재의 일반적인 흐름에 초점을 맞춘다. 이는 API 전체를 엄밀하게 정의한 것이 아니라 MVC·template 응답과 구분하기 위한 입문 수준의 설명이다.

## 문자열 직접 반환

`/hello-string` handler에 `@ResponseBody`를 적용하고 문자열을 반환하면 해당 문자열이 response body로 전달된다. PDF 예제는 `name` request parameter를 받아 `hello `와 결합한다.

실행 예제: `http://localhost:8080/hello-string?name=spring`

이 흐름에서는 View 이름을 해석하거나 template을 찾지 않는다. 문자열 반환은 기본적으로 `StringHttpMessageConverter`가 처리한다.

## 객체를 JSON으로 반환

`/hello-api` handler가 `Hello` 객체를 반환하고 `@ResponseBody`가 적용되어 있으면 객체가 JSON으로 변환된다. 예제의 `Hello` class는 `name` property와 getter·setter를 가지며, request parameter 값을 property에 설정해 반환한다.

실행 예제: `http://localhost:8080/hello-api?name=spring`

객체 처리는 기본적으로 `MappingJackson2HttpMessageConverter`가 담당한다. 처리 결과는 View가 아니라 JSON response body다.

### Getter·Setter와 JavaBean property

강의의 `Hello` 객체는 private `name` field와 `getName`, `setName` method를 둔다. 외부 code와 library는 이 method를 통해 값을 읽고 쓰며, 이런 getter·setter 기반 접근을 JavaBean 규약 또는 property 접근 방식으로 설명한다.

예제는 request parameter를 `setName`으로 객체에 저장하고 객체를 반환한다. JSON 결과의 `name` key는 이 객체의 property에 대응한다.

### JSON library

강의는 Java 객체를 JSON으로 바꾸는 대표 library로 Jackson과 Google의 Gson을 소개한다. Spring은 Jackson을 기본으로 선택하며, `MappingJackson2HttpMessageConverter`가 Jackson을 사용해 객체를 JSON 표현으로 변환한다. 다른 converter나 library로 변경할 수 있지만 입문 예제는 기본 구성을 그대로 사용한다.

## HttpMessageConverter 선택

Spring MVC는 다음 정보를 조합해 사용할 converter를 선택한다.

- Client가 보낸 HTTP Accept header
- Controller method의 반환 type

문자열과 객체 외에도 byte 등 여러 형태를 처리하는 `HttpMessageConverter`가 등록되어 있다. 따라서 `@ResponseBody`의 핵심은 특정 JSON library 자체보다, 반환값을 View rendering 없이 HTTP body 표현으로 변환하는 pipeline에 있다.

Client가 `Accept` header로 원하는 표현 형식을 제시하고 server에 해당 형식의 library와 converter가 준비되어 있으면 JSON 이외의 형식도 선택될 수 있다. 아무 형식이나 임의로 만드는 것이 아니라 request의 표현 형식 조건과 Controller 반환 type을 함께 판단한다.

## 처리 흐름

1. Browser나 client가 API URL을 요청한다.
2. 내장 Tomcat이 요청을 Spring container에 전달한다.
3. Mapping된 Controller method가 실행되어 문자열이나 객체를 반환한다.
4. `@ResponseBody`를 확인한 Spring MVC가 View Resolver 대신 `HttpMessageConverter`를 선택한다.
5. Converter가 반환값을 문자 또는 JSON으로 변환해 HTTP response body에 기록한다.

## IntelliJ 단축키

### 작성 중인 구문 완성

객체 생성문을 작성하는 과정에서 현재 구문을 완성하는 IntelliJ 기능을 소개한다.

- macOS: `Command + Shift + Enter`
- Windows: 강의 스크립트에는 실제 key 조합이 제시되지 않으므로 추정하지 않는다.

강의에서는 닫는 기호와 문장 끝을 직접 모두 입력하는 대신 이 조합으로 현재 구문을 완성하는 용도로 사용한다.

### Getter·Setter 생성

강의는 IntelliJ의 Getter and Setter 생성 기능도 사용하지만 실제 key 조합에 대한 발화가 불명확하다. 따라서 이를 확정 단축키로 기록하지 않으며, 현재 IntelliJ keymap에서 `Generate` 또는 `Getter and Setter` action을 조회한다.

## See Also

- [Spring MVC View 렌더링과 Thymeleaf](spring-mvc-view-rendering.md)
- [Spring Boot 프로젝트 생성과 첫 실행](spring-boot-project-setup.md)
