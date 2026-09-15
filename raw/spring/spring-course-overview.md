# 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술

> Source: [강의 참고 PDF](spring-course-overview.pdf)
> Collected: 2026-08-12
> Published: 2026-01-30

이 문서는 같은 디렉터리의 원본 PDF를 검색하고 근거 검사에 사용할 수 있도록 옮긴 companion text다. 문서의 목차와 버전 수정 이력을 보존했다.

## 강의 정보

- 강사: 김영한
- 과정: 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술
- 목차: 프로젝트 환경 설정, 스프링 웹 개발 기초, 회원 관리 예제(백엔드), 스프링 빈과 의존관계, 회원 관리 예제(웹 MVC), 스프링 DB 접근 기술, AOP, 다음으로

## 버전 수정 이력

- v2026-01-30: 스프링 부트 4 소스 코드 작동 확인
- v2025-06-15: 예제의 동시성 문제 안내, 참고 내용 포맷 수정
- v2024-01-20: `MemoryMemberRepositoryTest` 코드 오류 수정
- v2023-11-27: 스프링 부트 3.2 업데이트 관련 이슈, `start.spring.io`의 스프링 부트 2.x 지원 종료, 스프링 부트 3.0 이상 및 Java 17 이상 사용, 강의 매뉴얼 분할과 링크 안내
- v2023-08-16: 메뉴 링크 이동
- v2022-11-28: 스프링 부트 3.0 내용과 `Gradle - Groovy` 선택 안내 추가
- v2022-09-04: `helloConroller`를 `helloController`로, `template/`를 `templates/`로 수정
- v2021-12-01: H2는 1.4.200을 설치하고, 2.0.206을 이미 실행했다면 재설치 후 `~/test.mv.db`를 삭제하라는 당시 버전용 주의 추가. 관련 오류: `General error: "The write format 1 is smaller than the supported format 2 [2.0.206/5]" [50000-202] HY000/50000`
- v2021-07-18: 스프링 부트 최신 버전 선택 설명 추가
- v2021-03-03: Windows의 `h2.bat` 실행 안내 추가
- v2021-02-11: 회원 리포지토리 테스트에서 `result`, `member` 위치 수정
- v1.7(2020-11-23): 스프링 부트 2.4부터 `spring.datasource.username=sa`가 필요하다는 당시 안내 추가
- v1.6(2020-10-14): `helloController`를 `memberController`로 이미지 수정
- v1.5(2020-10-10): IntelliJ JDK 설치 확인 추가
- v1.4(2020-09-18): IntelliJ Community에서 `application.properties` key가 회색으로 표시될 수 있다는 설명 추가
- v1.3(2020-09-07): Windows 명령 표기를 `gradlew.bat`에서 `gradlew`로 수정
- v1.2(2020-08-28): Windows용 IntelliJ 단축키 조회 방법 추가
- v1.1(2020-08-28): Windows에서 macOS의 iTerm을 대신하는 방법 안내 추가
- v1.0(2020-07-20): 강의 오픈
