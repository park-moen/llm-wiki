# Spring Boot Gradle 빌드와 JAR 실행

> Sources: 김영한, 2026-01-30; Spring Boot, Unknown; Gradle, Unknown; Oracle, Unknown
> Raw: [프로젝트 환경 설정: 빌드하고 실행하기](../../raw/spring/spring-boot-build-and-run.md); [프로젝트 환경 설정 PDF companion](../../raw/spring/spring-project-environment-setup.md); [Spring Boot Gradle 실행 JAR 패키징](../../raw/spring/spring-boot-gradle-packaging-executable-archives.md); [Gradle Java plugin task](../../raw/spring/gradle-java-plugin-tasks.md); [Spring Boot Gradle 애플리케이션 실행](../../raw/spring/spring-boot-gradle-running-application.md); [Java JAR 파일 형식](../../raw/spring/java-jar-file-specification.md); [Java JAR 파일 소개](../../raw/spring/java-packaging-programs-in-jar-files.md); [Java의 JAR 실행 명령](../../raw/spring/java-command-jar-execution.md); [Gradle 기본 개념](../../raw/spring/gradle-core-concepts.md)
> Updated: 2026-10-04

## Overview

**JAR은 여러 파일을 하나로 묶은 파일이고, `bootJar`는 Spring Boot 애플리케이션을 실행할 수 있는 JAR을 만드는 Gradle 작업이다.** `./gradlew build`를 실행하면 보통 이 작업도 함께 실행된다. 생성된 JAR은 서버에 복사해 `java -jar`로 실행할 수 있다.

## 먼저, JAR이란?

JAR은 ZIP 형식을 바탕으로 여러 파일을 하나로 묶은 파일이다. Java 프로그램의 코드가 컴파일된 파일과 설정·이미지 같은 자료를 함께 담을 수 있다. 파일명은 보통 `hello-spring.jar`처럼 `.jar`로 끝난다. **JAR은 파일의 형식**이지, 그 자체로 Spring Boot 전용 기술이나 실행 명령은 아니다.

`java -jar hello-spring.jar`는 Java에게 그 파일 속 프로그램을 시작하라는 명령이다. 이 방식으로 실행하려면 JAR 안에 시작할 class를 알려주는 정보가 있어야 한다. 프로그램이 사용하는 library도 실행할 때 찾을 수 있어야 한다. 따라서 **모든 JAR 파일이 파일 하나만으로 실행되는 것은 아니다.**

## `jar`와 `bootJar`는 무엇이 다른가?

Gradle은 프로젝트를 build하는 도구다. Gradle에서 **task는 한 가지 작업의 이름**이고, **plugin은 필요한 작업들을 Gradle에 추가하는 기능**이다. 이름이 비슷하지만 `jar`는 파일 확장자이기도 하고, 일반 JAR을 만드는 Gradle task의 이름이기도 하다.

| 이름 | 뜻 | 결과 |
|------|----|------|
| `.jar` | 묶음 파일의 형식 | `hello-spring.jar` 같은 파일 |
| `jar` | Gradle Java plugin의 작업 | 주로 프로젝트의 코드와 자료를 담은 일반 JAR |
| `bootJar` | Spring Boot Gradle plugin의 작업 | 코드와 실행에 필요한 library를 함께 담은 실행 가능한 JAR |

## `bootJar`의 의미와 사용 이유

Spring Boot plugin과 Gradle `java` plugin을 적용하면 `bootJar` 작업이 생긴다. 이 작업은 Spring Boot 애플리케이션의 코드와 자료를 `BOOT-INF/classes`에, 실행에 필요한 library를 `BOOT-INF/lib`에 넣어 JAR을 만든다. 서버에는 이 JAR을 복사하고 `java -jar`로 실행할 수 있어서 배포가 간단해진다.

일반 `jar` 작업은 프로젝트의 코드와 자료를 묶는다. Spring Boot plugin의 기본 설정에서는 이 파일명에 `plain`이 붙어 `bootJar` 결과물과 구분된다. 예를 들어 `hello-spring-plain.jar`가 보이면 Spring Boot 실행용 JAR이라고 생각하고 배포하지 않는다.

## `bootJar`를 사용하지 않으면?

상황을 구분해야 한다.

1. **명령을 직접 호출하지 않은 경우:** `./gradlew build` 또는 `./gradlew assemble`을 실행하면 기본 task 연결에 따라 `bootJar`도 실행된다. 별도로 `./gradlew bootJar`를 입력할 필요는 없다.
2. **`bootJar`를 비활성화하거나 실행하지 않아 실행 JAR을 만들지 않은 경우:** `jar` task가 활성화되어 있다면 일반 JAR은 만들 수 있지만, 그 파일만 복사해 `java -jar`로 Spring Boot 애플리케이션을 실행하는 배포 방식에는 사용할 수 없다. 별도의 의존성 classpath와 실행 설정이 필요하다.
3. **JAR 없이 개발 중 실행하려는 경우:** `./gradlew bootRun`을 사용하면 archive를 먼저 만들지 않고 runtime classpath로 애플리케이션을 실행할 수 있다.

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
