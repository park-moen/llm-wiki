# Spring Boot Gradle 빌드와 JAR 실행

> Sources: 김영한, 2026-01-30
> Raw: [프로젝트 환경 설정: 빌드하고 실행하기](../../raw/spring/spring-boot-build-and-run.md); [프로젝트 환경 설정 PDF companion](../../raw/spring/spring-project-environment-setup.md)
> Updated: 2026-08-12

## Overview

Spring Boot application은 IDE 안에서만 실행하는 것이 아니라 Gradle Wrapper로 build해 실행 가능한 JAR로 만들 수 있다. 생성된 JAR에는 application과 실행에 필요한 구성이 함께 들어가므로 server에 복사한 뒤 `java -jar`로 실행할 수 있다. Build 전에 IDE에서 실행 중인 application을 종료해 같은 port를 중복 사용하지 않도록 해야 한다.

## Build 전에 IDE 실행 종료

강의는 terminal build를 시작하기 전에 IntelliJ에서 실행한 Spring Boot application을 먼저 종료하라고 강조한다. IDE process를 그대로 둔 채 build한 JAR을 실행하면 두 application이 같은 8080 port를 사용하려 해 오류가 발생할 수 있다.

이 단계는 IntelliJ keyboard shortcut 설명이 아니라 실행 process 관리다. 이번 강의 구간에서는 application 종료를 위한 특정 key combination을 제시하지 않는다.

## Gradle Wrapper로 build

Project root에서 운영체제에 맞는 Gradle Wrapper로 `build` task를 실행한다. 강의 내용을 명령 형태로 정리하면 다음과 같다.

macOS:

```bash
./gradlew build
```

Windows:

```bat
gradlew build
```

Windows의 실제 wrapper launcher 파일은 `gradlew.bat`이며, shell 환경에 따라 `gradlew.bat build`로 직접 실행해도 된다. PDF는 command를 `gradlew build`로 표기하고 Windows의 directory 목록 명령은 `ls` 대신 `dir`이라고 안내한다.

필요한 library가 local에 없으면 build 과정에서 내려받을 수 있다. Build가 끝나면 `build` directory가 생성되고, 실행할 JAR은 `build/libs` 아래에서 찾는다.

## JAR 실행

생성된 JAR이 있는 directory에서 다음 형태로 실행한다.

```bash
java -jar <JAR 파일명>
```

Application이 시작되면 IDE에서 실행했을 때와 마찬가지로 Spring Boot와 내장 web server가 동작한다. 이미 다른 process가 8080 port를 사용하고 있다면 같은 port에 두 server를 동시에 띄울 수 없으므로 기존 process를 종료해야 한다.

## Clean build

Build가 정상적으로 되지 않거나 이전 결과물을 제거하고 다시 만들고 싶다면 `clean`과 `build` task를 함께 실행한다.

macOS:

```bash
./gradlew clean build
```

Windows:

```bat
gradlew clean build
```

`clean` task는 기존 `build` directory를 제거하고, 이어지는 `build` task가 산출물을 새로 만든다. 완료 후 `build/libs`에서 새 JAR을 확인한다.

## 배포 단위

강의가 설명하는 기본 배포 방식은 build된 JAR을 server에 복사하고 `java -jar`로 실행하는 것이다. 별도 Tomcat을 server에 설치하고 application archive를 배치하던 방식과 달리, Spring Boot JAR은 내장 web server를 포함한 실행 단위로 사용할 수 있다.

## 환경 설정 단계의 마무리

이 build 과정은 다음 환경 설정 흐름을 마무리한다.

1. `start.spring.io`에서 project 생성
2. 주요 library와 dependency 구조 확인
3. Welcome Page와 Thymeleaf View 실행
4. Gradle build와 JAR 실행

이후 강의는 정적 content, MVC와 template engine, API를 포함하는 Spring 웹 개발 기초로 넘어간다.

## See Also

- [Spring Boot 프로젝트 생성과 첫 실행](spring-boot-project-setup.md)
- [Spring Boot Starter와 의존성 구조](spring-boot-starter-dependencies.md)
- [Spring MVC View 렌더링과 Thymeleaf](spring-mvc-view-rendering.md)
- [Spring MVC API 응답과 HttpMessageConverter](spring-mvc-api-response.md)
