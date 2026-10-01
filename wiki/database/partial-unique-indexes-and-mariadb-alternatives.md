# 부분 유니크 인덱스와 MariaDB의 조건부 고유성 구현

> Sources: PostgreSQL Global Development Group, Unknown; MariaDB Documentation, Unknown; MariaDB Server Jira, 2018-01-31
> Raw: [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md); [PostgreSQL Unique Constraints](../../raw/database/postgresql-18-partial-unique-constraints.md); [MariaDB Unique Index](../../raw/database/mariadb-conditional-uniqueness-virtual-columns.md); [MariaDB Generated Columns](../../raw/database/mariadb-generated-columns-index-support.md); [MariaDB Partial Index Feature Request](../../raw/database/2018-01-31-mariadb-partial-filtered-index-request.md)
> Updated: 2026-10-01

## Overview

**부분 유니크 인덱스(partial unique index)**는 테이블의 모든 행이 아니라 `WHERE` 조건을 만족하는 행에서만 중복을 금지한다. PostgreSQL은 `CREATE UNIQUE INDEX ... WHERE ...`로 이를 직접 지원한다. MariaDB는 같은 구문을 지원하지 않지만, 조건에 따라 값 또는 `NULL`을 만드는 **생성 열과 `UNIQUE` 인덱스**를 조합해 조건부 고유성 규칙을 구현할 수 있다. 두 방식은 같은 업무 규칙을 표현할 수 있어도 인덱스의 저장·조회 특성까지 동일한 것은 아니다. [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md); [MariaDB Unique Index](../../raw/database/mariadb-conditional-uniqueness-virtual-columns.md); [MariaDB Partial Index Feature Request](../../raw/database/2018-01-31-mariadb-partial-filtered-index-request.md)

## 언제 필요한가

전체 행에서 이메일이 고유해야 하면 일반 `UNIQUE (email)`이면 충분하다. **일부 상태의 행끼리만 고유해야 할 때** 부분 유니크 인덱스를 검토한다. PostgreSQL 공식 예시는 `(subject, target)` 조합에 대해 `success = true`인 결과만 하나로 제한하고 실패 결과는 여러 개 허용한다. 탈퇴하지 않은 회원의 이메일만 고유하게 두는 아래 예시는 그 규칙을 회원 테이블에 적용한 **가상 설계**다. [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md); [PostgreSQL Unique Constraints](../../raw/database/postgresql-18-partial-unique-constraints.md)

| `email` | `deleted_at` | 같은 이메일의 새 활성 행 |
| --- | --- | --- |
| `a@example.com` | `NULL`(활성) | 거부 |
| `a@example.com` | 값 있음(탈퇴) | 허용 |

이 예시에서 `email`은 `NOT NULL`이라고 가정한다. 이메일 자체가 `NULL`이면 고유성 규칙이 기대와 다르게 작동할 수 있으므로 별도 정책이 필요하다. PostgreSQL의 일반 고유성 규칙은 기본적으로 두 `NULL`을 같은 값으로 취급하지 않는다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-18-partial-unique-constraints.md)

## PostgreSQL에서 만드는 법

아래 SQL은 **가상 회원 테이블**의 `deleted_at IS NULL`인 행에만 `email` 고유성을 적용한다. PostgreSQL 공식 문서의 `CREATE UNIQUE INDEX ... WHERE ...` 구조를 회원 예시로 바꾼 것이다. 인덱스의 **키**는 `email`이고, `WHERE`는 인덱스에 포함할 **행의 조건**이다. [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md)

```sql
CREATE TABLE members (
    id bigint PRIMARY KEY,
    email text NOT NULL,
    deleted_at timestamp NULL
);

CREATE UNIQUE INDEX uq_members_active_email
    ON members (email)
    WHERE deleted_at IS NULL;
```

활성 회원에게 같은 이메일을 한 번 더 저장하면 DB가 거부한다. 기존 회원의 `deleted_at`에 값을 넣으면 그 행은 조건에서 빠지고, 같은 이메일을 새 활성 회원에게 사용할 수 있다. 탈퇴 행을 다시 활성으로 바꿀 때 같은 이메일의 활성 행이 이미 있다면 그 변경도 거부된다. 이는 조건에 맞는 행 사이의 고유성을 계속 강제한다는 규칙에서 도출한 결과다. [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md)

일반 `UNIQUE` 제약은 테이블 전체의 고유성을 선언한다. PostgreSQL 공식 문서도 일부 행에만 적용하는 고유성은 일반 `UNIQUE` 제약으로 작성할 수 없고, **고유 부분 인덱스**로 강제한다고 구분한다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-18-partial-unique-constraints.md)

## PostgreSQL에서 확인할 한계

- **외래 키의 참조 대상:** PostgreSQL 외래 키는 기본 키, 고유 제약 또는 **부분 인덱스가 아닌** 고유 인덱스의 열을 참조할 수 있다. 따라서 위 부분 유니크 인덱스만으로 `email`을 다른 테이블의 외래 키 참조 대상으로 만들 수 없다. 회원 참조에는 별도의 기본 키 `id`를 쓰는 편이 명확하다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-18-partial-unique-constraints.md)
- **조회에 쓰일지 여부:** 부분 인덱스를 만들었다고 모든 조회가 그 인덱스를 쓰지는 않는다. PostgreSQL이 조회의 `WHERE` 조건에서 인덱스의 조건을 추론할 수 있어야 하며, 일반적으로 조건을 쿼리에 맞춰 써야 한다. 인덱스 조건을 증명할 부분이 실행 시점 매개변수에 달려 있으면 계획 시점에 이를 증명하지 못할 수 있다. **인덱스를 통한 조회 최적화와 중복 금지 규칙은 별개**이므로 실제 조회 계획을 확인한다. [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md)
- **성능 기대:** 일부 행만 인덱싱하면 인덱스 크기와 갱신 비용을 줄이는 데 도움이 될 수 있지만, PostgreSQL 문서는 일반 인덱스보다 유리한 폭이 대부분의 경우 작을 수 있다고 경고한다. 고유성 규칙이 목적이라면 필요한 제약부터 정하고, 성능 이점은 별도로 측정한다. [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md)

## MariaDB에서는 어떻게 같은 규칙을 표현하나

MariaDB의 [부분/필터 인덱스 기능 요청](../../raw/database/2018-01-31-mariadb-partial-filtered-index-request.md)은 열린 상태이며, MariaDB 공식 가이드는 조건부 고유성에 **생성 열과 `UNIQUE`**를 사용하는 예시를 제공한다. 공식 예시는 활성·보류 사용자의 이름 중복을 막고 삭제된 사용자의 중복은 허용한다. 아래는 그 방식을 앞의 회원 예시에 적용한 **가상 SQL**이다. 실제 운영 DB의 버전·스토리지 엔진·문자 비교 규칙에서 DDL을 검증해야 한다. [MariaDB Unique Index](../../raw/database/mariadb-conditional-uniqueness-virtual-columns.md); [MariaDB Generated Columns](../../raw/database/mariadb-generated-columns-index-support.md); [MariaDB Partial Index Feature Request](../../raw/database/2018-01-31-mariadb-partial-filtered-index-request.md)

```sql
CREATE TABLE members (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    deleted_at DATETIME NULL,
    active_email VARCHAR(255)
        AS (CASE WHEN deleted_at IS NULL THEN email ELSE NULL END) PERSISTENT,
    UNIQUE KEY uq_members_active_email (active_email)
);
```

생성 열 `active_email`은 활성 행에서 실제 이메일, 탈퇴 행에서 `NULL`이 된다. MariaDB의 `UNIQUE`는 `NULL` 중복을 허용하므로 탈퇴 행은 같은 이메일이 여러 개여도 되고, 활성 행 사이에서는 중복이 거부된다. `email NOT NULL`은 활성 행이 `NULL`로 고유성 검사를 피하지 않도록 둔 설계 조건이다. 여러 열을 함께 고유하게 해야 한다면 `UNIQUE (tenant_id, active_email)`처럼 조합을 정할 수 있다. 이는 MariaDB 공식 예시의 원리를 확장한 설계 예시다. [MariaDB Unique Index](../../raw/database/mariadb-conditional-uniqueness-virtual-columns.md)

**중요한 차이:** 이 방법은 조건부 **고유성 규칙**을 흉내 내지만, PostgreSQL처럼 `WHERE`를 붙여 **일부 행만 인덱싱하는 기능**과 같지 않다. MariaDB 생성 열에 고유 인덱스를 만드는 방식이므로 PostgreSQL 부분 인덱스의 크기·조회 성능 이점까지 같다고 가정하면 안 된다. MariaDB 공식 문서도 이를 생성 열을 이용한 대체 방식으로 소개한다. [MariaDB Generated Columns](../../raw/database/mariadb-generated-columns-index-support.md); [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md)

MariaDB 생성 열은 `VIRTUAL`과 `PERSISTENT` 중 선택할 수 있고 둘 다 인덱싱할 수 있다. 위 SQL은 공식 조건부 고유성 예시처럼 `PERSISTENT`를 사용했다. 생성식은 같은 행의 값으로 계산해야 하며, 생성 열에 인덱스를 추가한 뒤에는 스키마 변경 방식에 제약이 생길 수 있다. 실제 DB 버전과 마이그레이션 도구에서 적용 가능 여부를 확인한다. [MariaDB Generated Columns](../../raw/database/mariadb-generated-columns-index-support.md); [MariaDB Unique Index](../../raw/database/mariadb-conditional-uniqueness-virtual-columns.md)

## 선택 기준

| 요구사항 | 우선 검토할 방법 |
| --- | --- |
| 모든 행에서 중복 금지 | 일반 `UNIQUE` 제약 |
| PostgreSQL에서 조건에 맞는 행만 중복 금지 | 부분 유니크 인덱스 |
| MariaDB에서 같은 **고유성 규칙** 필요 | 생성 열 + `UNIQUE` 인덱스 |

DB가 중복을 거부하는 규칙과 조회를 빠르게 하는 인덱스 사용 여부를 분리해서 검토한다. 특히 DBMS를 바꾸는 경우에는 **허용·거부되어야 할 행의 예시를 먼저 정하고**, 생성 열의 `NULL` 처리와 실제 DDL·조회 계획을 대상 DB에서 확인한다. [PostgreSQL Partial Indexes](../../raw/database/postgresql-18-partial-indexes.md); [MariaDB Unique Index](../../raw/database/mariadb-conditional-uniqueness-virtual-columns.md)

## See Also

- [Database Unique Constraint](unique-constraints.md)
- [Hibernate @GeneratedColumn과 DB 생성 열](../spring/hibernate-generated-column.md)
