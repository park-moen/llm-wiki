# Spring Boot Starter와 의존성 구조

> Sources: 김영한, 2026-01-30
> Raw: [프로젝트 환경 설정: 라이브러리 살펴보기](../../raw/spring/spring-boot-library-dependency-overview.md); [프로젝트 환경 설정 PDF companion](../../raw/spring/spring-project-environment-setup.md)
> Updated: 2026-08-12

## Overview

Spring Boot 프로젝트에서 개발자가 `build.gradle`에 직접 선언하는 library는 적어도, 실제로는 Starter가 필요로 하는 library가 전이적으로 함께 추가된다. Gradle은 이 dependency graph를 해석해 Spring Web MVC, 내장 Tomcat, Spring Core, logging과 test 도구까지 가져오고 중복 dependency를 정리한다. 입문 단계에서는 모든 library를 외우기보다 Starter가 어떤 기능 묶음을 제공하는지 큰 구조를 파악하는 것이 중요하다.

## 직접 의존성과 전이 의존성

강의 예제에서 직접 선택하는 주요 dependency는 Spring Web, Thymeleaf와 test library이다. 그러나 Gradle의 External Libraries나 Dependencies tree에는 훨씬 많은 library가 나타난다. 직접 추가한 Starter 자체가 다른 library를 필요로 하고, 그 library도 다시 하위 dependency를 가지기 때문이다.

Gradle과 Maven 같은 build tool은 이 관계를 따라 필요한 library를 자동으로 내려받는다. 같은 library가 여러 경로에서 요구되면 dependency tree에서는 중복을 제거해 표현한다. 따라서 직접 선언한 항목과 실제 runtime classpath에 포함되는 전체 항목은 서로 다를 수 있다.

## 주요 Starter 구성

### Spring Boot Starter Web

Web Starter는 웹 애플리케이션에 필요한 묶음을 제공한다. 강의에서 강조하는 핵심 구성은 다음과 같다.

- Spring Web
- Spring Web MVC
- Spring Boot Starter Tomcat

Tomcat이 embedded library로 포함되므로 별도의 외부 Tomcat server를 설치하고 application code를 배포하는 과정 없이 Java `main` method 실행만으로 web server를 시작할 수 있다.

### Spring Boot Starter Thymeleaf

Thymeleaf Starter는 HTML을 생성하는 Thymeleaf template engine과 그 실행에 필요한 dependency를 가져온다. Starter 하나를 선언하면 관련 library를 개별적으로 찾아서 추가하지 않아도 된다.

### 공통 Spring Boot Starter

다른 Starter를 선택하면 공통 기반인 Spring Boot Starter도 전이적으로 포함된다. 이를 통해 Spring Boot, auto configuration, Spring Core와 logging 관련 library가 함께 설정된다.

## Logging 구성

강의는 실무 application에서 `System.out.println` 대신 logging framework를 사용해야 한다고 설명한다. Logging을 사용하면 log level에 따라 심각한 error를 분리하고, output을 log file로 관리할 수 있다.

Spring Boot Starter Logging에는 SLF4J와 Logback이 포함된다. 강의의 설명에서 SLF4J는 logging interface 역할을 하고 Logback은 실제 output을 담당하는 implementation으로 사용된다. 이 구분을 이해하면 application code가 특정 logging implementation에 직접 결합되지 않는 이유를 파악할 수 있다.

## Test library 구성

Test Starter가 제공하는 주요 도구는 다음과 같다.

- JUnit 5: Java test를 작성하고 실행하는 중심 framework
- Mockito: mock object를 사용한 test를 돕는 library
- AssertJ: assertion 작성을 편리하게 하는 library
- Spring Test: Spring과 통합한 test를 지원하는 library

입문 단계에서는 각 도구의 API를 미리 암기하기보다, JUnit이 test 실행의 중심이고 나머지 library가 mock·assertion·Spring integration을 보조한다는 역할 구분부터 익힌다.

## 의존성 구조를 확인하는 방법

IDE의 Gradle tool window에서 프로젝트의 Dependencies tree를 열면 직접 추가한 Starter와 하위 dependency 관계를 탐색할 수 있다. 이 tree는 다음 질문에 답할 때 유용하다.

- 직접 선언하지 않은 library가 왜 포함되었는가?
- 특정 Starter가 어떤 기능과 framework를 함께 가져오는가?
- 같은 dependency가 여러 경로에서 요구되는가?

처음부터 dependency graph 전체를 이해하려 하기보다, 필요한 시점에 특정 Starter에서 아래 방향으로 관계를 따라가는 방식이 적합하다.

### 강의에서 소개한 IntelliJ 접근 방법

강의에서는 좌측 하단의 Tool Window 아이콘을 눌러 Gradle 창을 표시하는 방법과 keyboard를 이용한 빠른 접근을 함께 소개한다. macOS에서는 `Command`를 두 번 누르는 동작을 보여주며, Windows의 `Alt` 두 번은 강사가 추정형으로 언급한다. 따라서 Windows 조합은 확정 단축키가 아니라 강의 중 참고 사항으로 보고, 동작하지 않으면 Tool Window 아이콘이나 IntelliJ의 현재 keymap을 사용한다.

## See Also

- [Spring Boot 프로젝트 생성과 첫 실행](spring-boot-project-setup.md)
