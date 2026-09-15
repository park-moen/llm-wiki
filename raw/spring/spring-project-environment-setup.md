# 프로젝트 환경 설정

> Source: [강의 참고 PDF](spring-project-environment-setup.pdf)
> Collected: 2026-08-12
> Published: 2026-01-30

이 문서는 같은 디렉터리의 원본 PDF를 검색하고 근거 검사에 사용할 수 있도록 옮긴 companion text다. 페이지 순서에 따라 환경 요구사항, 명령과 IntelliJ 단축키를 보존했다.

## 프로젝트 생성과 버전

- 사전 준비: Java 17 이상, IntelliJ IDEA
- Spring Boot 3.x는 Java 17 이상이 필요하고 `javax` namespace가 `jakarta`로 변경된다.
- 강의 최신 안내: Spring Boot 3.0 이상을 사용하고 Java는 17 또는 21을 선택한다. Spring Boot 4에서도 소스 작동을 확인했다.
- Spring Initializr 선택: Gradle - Groovy, Java, 정식 Spring Boot 버전, Packaging Jar, Spring Web, Thymeleaf
- 예제 metadata: Group `hello`, Artifact `hello-spring`
- 영상 원본의 Spring Boot 2.3.1과 Java 11 표시는 과거 강의 환경이다.

## 프로젝트 구조와 실행

- `src/main/java`: Java source
- `src/main/resources`: Java 이외의 configuration과 HTML
- `src/test`: test source
- `build.gradle`: build 및 dependency 설정
- `@SpringBootApplication`이 있는 main method를 실행하면 내장 Tomcat이 port 8080에서 시작한다.
- mapping이 없을 때 `localhost:8080`의 Whitelabel Error Page는 server가 정상 기동했다는 확인으로 사용할 수 있다.

## IntelliJ 설정과 단축키

- Refactor This(Windows): `Ctrl + Alt + Shift + T`
- Project Structure(Windows): `Ctrl + Alt + Shift + S`
- Project Structure(macOS): `⌘;`
- Settings(Windows): `Ctrl + Alt + S`
- Preferences(macOS): `⌘,`
- `Project SDK`와 `Project language level`, Gradle JVM이 설치한 JDK를 가리키는지 확인한다.
- Gradle을 통한 실행이 느리면 Build and run 및 Run tests using을 IntelliJ IDEA로 변경하는 안내가 있다.

## 라이브러리와 화면

- `spring-boot-starter-web`: Spring Web MVC, 내장 Tomcat
- `spring-boot-starter-thymeleaf`: Thymeleaf template engine
- `spring-boot-starter`: Spring Core와 logging. SLF4J interface와 Logback implementation을 사용한다.
- `spring-boot-starter-test`: JUnit, Mockito, AssertJ, Spring Test
- `resources/static/index.html`은 Welcome Page가 된다.
- Controller가 View 이름을 반환하면 View Resolver가 `resources/templates/{ViewName}.html`을 찾고 Thymeleaf가 렌더링한다.
- DevTools를 사용하면 HTML 변경 후 compile하여 server 재시작 없이 반영할 수 있다.

## 빌드와 실행

macOS/Linux:

```bash
./gradlew build
cd build/libs
java -jar hello-spring-0.0.1-SNAPSHOT.jar
```

Windows:

```bat
gradlew build
cd build\libs
java -jar hello-spring-0.0.1-SNAPSHOT.jar
```

Windows에는 `gradlew.bat` 파일이 있고, `ls` 대신 `dir`을 사용한다. 문제가 있으면 `./gradlew clean build` 또는 Windows의 `gradlew clean build`로 기존 `build`를 지우고 다시 빌드한다.
