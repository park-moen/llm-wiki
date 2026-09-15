# Spring Boot 프로젝트 생성과 첫 실행

> Sources: 김영한, 2026-01-30
> Raw: [프로젝트 환경 설정: 스프링 부트 프로젝트 생성](../../raw/spring/project-setup-and-spring-initializr.md); [프로젝트 환경 설정: 라이브러리 살펴보기](../../raw/spring/spring-boot-library-dependency-overview.md); [View 환경 설정: Welcome Page와 Thymeleaf](../../raw/spring/spring-view-environment-setup.md); [프로젝트 환경 설정: 빌드하고 실행하기](../../raw/spring/spring-boot-build-and-run.md); [강의 전체 참고 PDF companion](../../raw/spring/spring-course-overview.md); [프로젝트 환경 설정 PDF companion](../../raw/spring/spring-project-environment-setup.md)
> Updated: 2026-08-12

## Overview

Spring Boot 프로젝트는 `start.spring.io`에서 빌드 도구, 언어, Spring Boot 버전과 의존성을 선택해 생성할 수 있다. 생성된 프로젝트를 IDE로 열어 Gradle 설정과 기본 디렉터리 구조를 확인한 뒤 메인 메서드를 실행하면, 내장 Tomcat과 Spring 애플리케이션이 함께 기동된다. 오래된 영상의 버전 선택은 그대로 따르지 말고 강의의 최신 수정 이력을 우선해야 한다.

## 2026년 기준 버전 지침

영상 본문은 Java 11과 Spring Boot 2.3.1을 사용해 프로젝트를 생성한다.

> **Status: Outdated** (2023-11-27)
> 강의 수정 이력에 따르면 `start.spring.io`의 Spring Boot 2.x 지원은 종료되었으며, Spring Boot 3.0 이상과 Java 17 이상을 사용해야 한다. 이후 v2026-01-30에는 Spring Boot 4에서 강의 소스 코드가 작동함을 확인했다.

따라서 영상에서 선택하는 숫자를 그대로 재현하기보다 수정 이력과 현재 프로젝트 생성기가 허용하는 조합을 기준으로 환경을 구성한다.

## Spring Initializr에서 프로젝트 만들기

강의의 프로젝트 생성 흐름은 다음과 같다.

1. `start.spring.io`에 접속한다.
2. 빌드 도구로 Gradle을 선택한다. 수정 이력에는 프로젝트 선택 항목으로 `Gradle - Groovy`가 추가되어 있다.
3. 언어로 Java를 선택한다.
4. 정식 릴리스된 Spring Boot 버전 중 현재 지침에 맞는 버전을 선택한다. Snapshot이나 M1은 정식 릴리스가 아닌 개발 단계 버전으로 설명된다.
5. 프로젝트 metadata를 입력한다. 강의 예제는 group을 `hello`, artifact를 `hello-spring`으로 설정한다.
6. dependencies로 Spring Web과 Thymeleaf를 추가한다.
7. Generate로 압축 파일을 내려받고 압축을 푼다.

최신 참고 PDF는 Packaging으로 `Jar`, Java로 17 또는 21을 제시한다. Spring Boot 3.x는 Java 17 이상이 필요하며 Java EE package namespace가 `javax`에서 `jakarta`로 변경된다는 점도 함께 안내한다.

Spring Web은 웹 애플리케이션 구성을 위해, Thymeleaf는 HTML을 생성하는 template engine으로 선택한다. Template engine은 회사와 프로젝트에 따라 다른 선택지가 있을 수 있으므로 강의에서는 우선 Thymeleaf를 사용한다.

직접 선택한 Starter가 실제로 가져오는 library와 전이 의존성의 구조는 [Spring Boot Starter와 의존성 구조](spring-boot-starter-dependencies.md)에서 이어서 정리한다.

## IDE에서 열기와 첫 의존성 다운로드

강의는 IntelliJ IDEA 사용을 권장하지만 Eclipse 사용도 가능하다고 설명한다. IntelliJ IDEA에서는 압축을 푼 프로젝트의 `build.gradle`을 열어 project로 import한다. 처음 열 때 외부 라이브러리를 내려받으므로 network 연결이 필요하고 초기 loading에 시간이 걸릴 수 있다.

IntelliJ IDEA가 실행과 test를 Gradle을 통해 처리해 느릴 경우, 강의에서는 build와 run 설정을 IntelliJ IDEA로 변경해 Java를 직접 실행하는 방법을 소개한다. IDE 버전에 따라 설정 화면과 항목 이름은 달라질 수 있으므로 핵심은 Gradle 경유 실행과 IDE 직접 실행의 차이를 이해하는 것이다.

## IntelliJ 사용 팁과 단축키

강의는 프로젝트를 진행하면서 IntelliJ 단축키를 함께 익히는 방식으로 구성된다. 현재 수집된 강의에서 확인되는 IntelliJ 관련 조작은 다음과 같다.

- Package 표시: 강사는 중간 package를 한 줄로 묶어 보여주는 `Compact Middle Packages` 옵션을 선호한다. 이는 source 구조를 바꾸는 기능이 아니라 Project view의 표시 방식이다.
- Refactor This: Windows `Ctrl + Alt + Shift + T`
- Project Structure: Windows `Ctrl + Alt + Shift + S`, macOS `⌘;`
- Settings 또는 Preferences: Windows `Ctrl + Alt + S`, macOS `⌘,`
- Gradle 창 열기: 좌측 하단 Tool Window 아이콘을 눌러 Gradle을 선택할 수 있다.
- Gradle 창 빠른 접근: 라이브러리 강의에서는 macOS의 `Command`를 두 번 누르는 방법을 보여주고, Windows에서는 `Alt`를 두 번 누르는 것으로 추정해 설명한다.
- View template 탐색: View 환경 설정 강의에서는 IntelliJ Enterprise edition에서 Controller가 반환한 View 이름에 macOS의 `Command` key를 사용해 연결된 template로 이동하는 기능을 보여준다. 무료 edition에서는 지원되지 않을 수 있으며, 이 구간에서는 Windows 대응 단축키를 설명하지 않는다.
- Build 강의: IntelliJ에서 실행 중인 application을 먼저 종료한 뒤 terminal에서 Gradle Wrapper를 사용한다. 이 구간에는 새 keyboard shortcut이 제시되지 않으며, IDE 조작과 terminal command를 구분해야 한다.

PDF에 명시된 Windows 단축키와 별개로, 강사가 영상에서 추정형으로 언급한 `Alt` 두 번 같은 조합은 확정 단축키로 보지 않는다. Keymap이 다르거나 동작하지 않으면 action 이름을 검색해 현재 설정을 확인한다. 강의 수정 이력에는 v1.2 - 2020-08-28에 Windows용 IntelliJ 단축키 조회 방법이 추가되었다고 기록되어 있다.

## 생성된 프로젝트 구조

- `src/main/java`: package와 application source가 들어간다.
- `src/main/resources`: Java source 이외의 configuration, properties, XML과 HTML 등이 들어간다.
- `src/test`: test source가 들어간다. Main과 test를 분리하는 구조는 기본적인 project convention이다.
- `build.gradle`: plugin, Java version, repository와 dependency 등 build 설정을 담는다.
- `.gitignore`: build result처럼 source control에 포함하지 않을 파일을 지정한다.
- Gradle wrapper와 `settings.gradle`: 프로젝트가 Gradle build를 실행하고 구성하는 데 사용한다.

초기 단계에서는 Gradle 전체를 깊게 학습하기보다 version과 library dependency를 관리하는 역할부터 이해한다. 공개 library는 기본적으로 Maven Central에서 내려받도록 설정되며, 필요하면 별도 repository URL을 지정할 수 있다.

## 실행 성공 확인

생성된 application class의 `main` method를 실행하면 `@SpringBootApplication`을 기준으로 Spring Boot가 시작되고 내장 Tomcat도 함께 실행된다. Log에서 Tomcat이 port 8080으로 시작되었는지 확인한 뒤 `localhost:8080`에 접속한다.

아직 mapping이나 화면을 만들지 않은 상태에서는 Whitelabel Error Page가 나타나도 server 기동에는 성공한 것이다. Application을 종료한 뒤 같은 주소에 연결되지 않는 것까지 확인하면 시작과 종료 동작을 구분할 수 있다.

다음 단계에서는 [Spring MVC View 렌더링과 Thymeleaf](spring-mvc-view-rendering.md)처럼 정적 Welcome Page를 추가하고 Controller와 template을 연결해 실제 화면을 만든다.

IDE에서 첫 실행까지 확인한 뒤에는 [Spring Boot Gradle 빌드와 JAR 실행](spring-boot-gradle-build-and-run.md)에 따라 실행 가능한 JAR을 만들고 IDE 밖에서 실행할 수 있다.

## 강의 수정 이력을 읽는 기준

강의 수정 이력은 최초 영상과 현재 실행 환경 사이의 차이를 보완한다. 환경을 구성할 때는 다음 순서로 판단한다.

- 최신 버전 항목을 먼저 확인한다.
- 특정 날짜의 호환성 경고는 당시 사용하던 version 조합에 관한 기록으로 구분한다.
- 오타와 image 수정 내역은 현재 교재나 source에 이미 반영되었는지 확인한다.
- v2025-06-15에는 예제의 concurrency 문제 안내가 추가되었으므로, 이후 회원 repository 예제를 학습할 때 해당 주의를 함께 확인한다.

예를 들어 v2021-12-01의 H2 1.4.200 설치 지침과 H2 2.0.206 호환성 오류는 당시 강의 환경을 위한 기록이다. 최신 환경에서 이를 무조건 적용하기보다 현재 Spring Boot·Java 조합과 강의의 최신 안내를 우선한다.

## See Also

- [Spring Boot Gradle 빌드와 JAR 실행](spring-boot-gradle-build-and-run.md)
- [Spring MVC API 응답과 HttpMessageConverter](spring-mvc-api-response.md)
- [Spring MVC View 렌더링과 Thymeleaf](spring-mvc-view-rendering.md)
- [Spring Boot Starter와 의존성 구조](spring-boot-starter-dependencies.md)
- [스프링 입문 학습 로드맵](spring-learning-roadmap.md)
- [Spring 회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)
