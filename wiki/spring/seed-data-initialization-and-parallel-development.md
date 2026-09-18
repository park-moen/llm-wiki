# 시드 데이터 초기화와 병렬 개발

> Sources: User-provided project note, 2026-09-17; Spring Boot Docs, Unknown; Spring Framework Docs, Unknown; Martin Fowler·Pramod Sadalage, Unknown; Microsoft Learn, Unknown
> Raw: [프로젝트의 시드 데이터 용어와 병렬 개발 설명](../../raw/spring/2026-09-17-project-seed-data-note.md); [Spring Boot Database Initialization 발췌](../../raw/spring/spring-boot-database-initialization-extract.md); [Spring Framework Environment Profiles 발췌](../../raw/spring/spring-framework-environment-profiles-extract.md); [Evolutionary Database Design 발췌](../../raw/software-career/evolutionary-database-design-seed-data-extract.md); [EF Core Data Seeding 주의사항 발췌](../../raw/software-career/ef-core-data-seeding-caution-extract.md)
> Updated: 2026-09-17

## Overview

`seed data`는 개발·검증에 필요한 초기 데이터 집합이고, 이를 넣는 코드는 database initializer의 한 형태다. `seeder`는 여러 생태계에서 관습적으로 쓰이는 구현체 이름이지만 Spring의 공식 추상화 이름은 아니다. 한국어 기획·설계 문서에서는 `시더`보다 `시드 데이터` 또는 `시드 데이터 초기화 코드`라고 쓰고, `SiteSeeder`처럼 실제 class를 가리킬 때만 identifier를 그대로 적는 편이 의미가 분명하다.

## 용어를 구분한다

| 표현 | 의미 | 문서에서의 권장 용도 |
|---|---|---|
| `seed data`·시드 데이터 | 새 database나 특정 환경에 미리 넣는 초기 데이터 | 일반 개념과 데이터 자체 |
| 시드 데이터 초기화 코드 | 시드 데이터를 생성·저장하는 code | 구현 방식을 설명할 때 |
| `SiteSeeder` | 프로젝트에 실제 존재한다고 제시된 Kotlin class 이름 | 해당 symbol을 정확히 가리킬 때 |
| database initialization | schema 또는 데이터를 초기 상태로 만드는 넓은 절차 | Spring 공식 기능과 운영 절차를 설명할 때 |
| fixture | 한 test나 test suite가 전제로 삼는 입력과 상태 | test 전용 데이터를 설명할 때 |

`시더`라고만 쓰면 데이터인지 실행 code인지 알기 어렵다. `seeder`로 바꾸는 것도 영어 identifier를 음역 대신 표기하는 정도라, 처음 읽는 사람이 역할을 바로 이해하게 하지는 못한다. 일반 문장에서는 역할까지 드러내는 `시드 데이터 초기화 코드`가 가장 명확하다.

## Spring에서 `@Profile("init")`이 뜻하는 것

Spring의 profile은 특정 profile이 활성화됐을 때만 bean definition을 등록하는 조건이다. 따라서 사용자 설명에 나온 `@Profile("init")`은 Spring이 제공하는 seeder 기능이 아니라, `init` profile에서만 `SiteSeeder` 같은 프로젝트 component를 등록하기 위한 경계로 해석해야 한다.

이 구조는 다음 두 개념의 조합이다.

```text
프로젝트 코드
└── SiteSeeder: 어떤 데이터를 어떤 순서로 저장할지 정의

Spring profile
└── init: 해당 component를 어느 환경에서 실행할지 제한
```

Spring Boot 자체도 SQL database의 schema와 데이터를 초기화하는 기능을 제공한다. 그러나 Kotlin class에서 repository를 호출하는 방식, SQL script, Flyway·Liquibase migration은 서로 다른 구현 전략이다. 프로젝트 문서에서는 무엇을 쓰는지 명시하고 여러 초기화 경로가 같은 데이터를 중복 관리하지 않게 해야 한다.

## 시드 데이터가 병렬 개발을 가능하게 하는 방식

사용자가 제시한 사례의 원래 의존 관계는 다음과 같다.

```text
어드민 가격 화면(041)
└── EventTicket·TicketPrice 입력
    └── 공개 가격 조회(018) 개발·확인
```

공개 조회는 조회할 데이터가 있어야 동작을 확인할 수 있으므로, 사람이 어드민 화면을 완성하고 데이터를 입력할 때까지 기다리면 순차 작업이 된다. 같은 데이터를 시드 데이터 초기화 코드로 먼저 만들면 의존 관계가 달라진다.

```text
EventTicket·TicketPrice schema와 조회 계약 확정
├── 시드 데이터 초기화 코드
│   └── 공개 가격 조회(018) 개발·확인
└── 어드민 가격 화면(041) 개발
```

이때 제거되는 것은 **데이터 입력 UI에 대한 선행 의존성**이다. Domain model, database schema와 조회 계약까지 독립되는 것은 아니다. `EventTicket`과 `TicketPrice`의 관계나 필수 field가 정해지지 않았다면 두 기능을 동시에 시작해도 뒤에서 다시 충돌할 수 있다.

Martin Fowler와 Pramod Sadalage는 sample data를 source control에 두어 새 database를 재현할 수 있게 해야 한다고 설명한다. 병렬 개발에서도 같은 원칙이 적용된다. 두 작업자가 동일한 시드 데이터와 schema를 사용해야 각자의 local 결과를 통합할 수 있다.

## 병렬 진행 전에 고정할 계약

다음 조건을 먼저 합의해야 시드 데이터가 임시 우회가 아니라 공통 개발 기반이 된다.

- `EventTicket`·`TicketPrice`의 관계와 필수 field
- 공개 조회가 의존하는 식별자와 상태값
- 정상 가격, 판매 불가와 경계값을 대표하는 데이터
- 반복 실행 시 기존 데이터를 어떻게 식별할지에 관한 정책
- schema 변경 시 시드 데이터를 함께 갱신할 책임
- `init` profile을 허용할 환경과 production에서의 비활성화 조건

시드 데이터가 이 계약을 앞질러 임의의 domain rule을 확정해서는 안 된다. 두 기능이 공유할 최소 계약을 먼저 결정하고, 그 계약을 재현하는 작은 데이터 집합을 두는 것이 목적이다.

## 시드 데이터와 migration을 구분한다

모든 초기 데이터를 같은 수명으로 취급하지 않는다.

| 데이터 종류 | 예 | 관리 판단 |
|---|---|---|
| 필수 reference data | 변경이 드문 상태 code·분류 | migration 또는 명시적인 운영 초기화 검토 |
| 개발 sample data | 예시 공연·티켓·가격 | development profile의 초기화 코드에 적합 |
| test fixture | 특정 test의 성공·실패 조건 | test가 직접 소유하고 서로 격리 |
| production business data | 실제 공연·판매 가격 | 어드민 기능이나 승인된 운영 절차로 생성 |

Spring Boot 공식 문서는 database 초기화 경로를 제공하지만, 프로젝트가 선택한 migration 도구와 초기화 code의 책임을 섞어 쓰지 않는 편이 안전하다. 다른 생태계의 Microsoft 공식 문서도 seed code를 일반적인 앱 실행 경로에 두면 여러 instance가 동시에 실행될 때 충돌할 수 있다고 경고한다. 프로젝트의 `init` profile은 이런 위험을 줄이는 실행 경계로 사용할 수 있지만, profile 이름만으로 production 실행이 자동 방지되는 것은 아니다.

## 문서 표현 권장안

검토 대상 문장은 다음처럼 바꾸는 것이 가장 명확하다.

**권장 문장:** 시드 데이터 초기화 코드로 `EventTicket`·`TicketPrice`를 미리 생성하면, 어드민 가격 화면(041)과 공개 조회(018)를 병렬로 개발할 수 있다.

프로젝트 구현까지 함께 알려야 한다면 다음처럼 쓸 수 있다.

**구현 포함 문장:** `init` profile의 시드 데이터 초기화 코드로 `EventTicket`·`TicketPrice`를 미리 생성하면, 어드민 가격 화면(041)과 공개 조회(018)를 병렬로 개발할 수 있다.

`SiteSeeder`라는 실제 class를 직접 안내하는 문맥에서는 다음 표현이 적절하다.

**Class 명시 문장:** `SiteSeeder`가 `EventTicket`·`TicketPrice` 시드 데이터를 생성하므로, 어드민 가격 화면(041)과 공개 조회(018)를 병렬로 개발할 수 있다.

표기 통일은 단순 치환보다 의미 기준으로 수행한다. 데이터 자체는 `시드 데이터`, 실행 code는 `시드 데이터 초기화 코드`, 실제 class는 `SiteSeeder`처럼 구분하면 기존 문서의 `시더`를 일괄 변경할 때도 문맥을 보존할 수 있다.

## 한계

외부 자료는 seed data와 환경별 초기화의 일반 원칙을 뒷받침한다. `@Profile("init")`, `SiteSeeder`, `EventTicket`, `TicketPrice`와 기능 번호의 실제 구현 상태는 사용자가 제공한 설명에 근거하며 현재 Wiki 저장소에서 project code를 직접 검증한 결과는 아니다.

## See Also

- [Spring DB 접근 기술 비교](spring-database-access-technologies.md)
- [Spring 회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)
- [효과적인 Software Design Document 작성법](../software-career/effective-software-design-document.md)
