# DB 설계에서 다형 참조

> Sources: Ruby on Rails, Unknown; GitLab, Unknown; PostgreSQL Global Development Group, 2026-08-13
> Raw: [Rails 다형 연관](../../raw/database/rails-active-record-polymorphic-associations.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md); [PostgreSQL 제약 조건](../../raw/database/2026-08-13-postgresql-constraints.md)
> Updated: 2026-09-28

## Overview

**다형 참조(polymorphic reference)**는 한 행이 서로 다른 종류의 테이블 중 하나를 가리키도록, 대상의 종류와 ID를 함께 저장하는 설계다. Rails의 `Picture` 예시는 `imageable_type`에 `Employee` 또는 `Product`를, `imageable_id`에 해당 행의 ID를 저장한다. 애플리케이션에서는 하나의 연관 관계처럼 다룰 수 있지만, 이 두 열은 일반적인 데이터베이스 외래 키 하나와 같지 않다. [Rails 다형 연관](../../raw/database/rails-active-record-polymorphic-associations.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)

## 형태와 얻는 편의

```text
pictures(id, imageable_type, imageable_id, ...)
                 └─ Employee 또는 Product
```

Rails에서는 `belongs_to :imageable, polymorphic: true`로 여러 모델을 하나의 연관 관계로 다룬다. 서로 다른 종류의 대상에 같은 부속 자료를 붙일 때 애플리케이션 모델을 간단하게 만들 수 있다. 조회할 때는 종류와 ID를 함께 사용하고, 보통 두 열을 묶은 인덱스를 검토한다. [Rails 다형 연관](../../raw/database/rails-active-record-polymorphic-associations.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)

여기서 `imageable_id`를 Rails 문서가 “foreign key column”이라고 부르는 것은 **연관 대상을 찾는 열의 역할**을 설명한다. `imageable_type` 값에 따라 참조 테이블이 바뀌는 관계에 데이터베이스의 일반 `FOREIGN KEY ... REFERENCES 한_테이블` 제약이 자동으로 생긴다는 뜻은 아니다. PostgreSQL의 외래 키는 선언할 때 참조할 테이블을 정하고, GitLab은 다형 연관에서 외래 키로 무결성을 강제할 수 없다고 명시한다. [Rails 다형 연관](../../raw/database/rails-active-record-polymorphic-associations.md); [PostgreSQL 제약 조건](../../raw/database/2026-08-13-postgresql-constraints.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)

## DB 설계에서 부담이 되는 지점

- **참조 무결성:** 존재하지 않는 대상 ID나 종류와 ID가 맞지 않는 조합을 일반 외래 키만으로 막을 수 없다. 이를 허용한다면 애플리케이션이나 별도 DB 로직이 검증을 떠맡는다. [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)
- **종류 이름의 변경:** Rails 구현은 모델 클래스명을 종류 열에 저장하므로 클래스명을 바꾸면 저장된 값도 마이그레이션해야 한다. [Rails 다형 연관](../../raw/database/rails-active-record-polymorphic-associations.md)
- **조회와 인덱스:** 대상을 찾을 때 종류와 ID를 함께 조건으로 사용한다. GitLab은 이에 맞는 복합 인덱스와 복잡한 쿼리의 비용을 검토하라고 설명한다. [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)

이것은 다형 참조가 언제나 잘못된 모델이라는 뜻은 아니다. Rails는 실제로 이 기능을 제공하고 사용 사례를 설명한다. GitLab의 “별도 테이블을 사용하라”는 문장은 **GitLab의 데이터베이스 개발 지침**으로 읽어야 한다. 선택할 때는 애플리케이션 모델의 편의와 DB 수준 무결성 요구를 함께 판단한다. [Rails 다형 연관](../../raw/database/rails-active-record-polymorphic-associations.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)

## 외래 키를 유지하는 대안

| 대안 | 구조 | 고려할 점 |
|---|---|---|
| 대상 종류별 테이블 | `employee_pictures`, `product_pictures`처럼 각 테이블에 명확한 외래 키를 둔다. | 테이블과 공통 조회의 조합이 늘지만, 참조 무결성을 DB에서 강제할 수 있다. GitLab이 권하는 방향이다. |
| 고정된 대상별 외래 키 열 | `pictures`에 `employee_id`, `product_id`를 각각 두고 둘 중 하나만 채우도록 `CHECK`를 둔다. | 대상 종류가 적고 고정돼 있을 때 검토할 수 있다. 새 대상 종류가 생기면 스키마를 바꿔야 한다. |

두 번째 대안은 PostgreSQL의 **각 외래 키가 한 테이블을 참조한다**는 규칙과 **`CHECK`가 같은 행의 열 조합을 검사할 수 있다**는 기능을 조합한 설계 예시다. `CHECK`만으로 다른 테이블의 행 존재를 검사하는 방식은 PostgreSQL 문서가 지원하지 않는다고 설명한다. 따라서 대상 존재 여부는 각 외래 키가 확인하고, `CHECK`는 어느 외래 키 열을 사용할지만 제한한다. [PostgreSQL 제약 조건](../../raw/database/2026-08-13-postgresql-constraints.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)

대상 종류가 `employees`와 `products`로 고정돼 있다면 PostgreSQL에서는 다음처럼 표현할 수 있다. 이는 공식 문서의 외래 키와 `CHECK` 규칙을 조합한 **설계 예시**다.

```sql
CREATE TABLE pictures (
    id bigint PRIMARY KEY,
    employee_id bigint REFERENCES employees(id),
    product_id bigint REFERENCES products(id),
    CHECK (
        (employee_id IS NOT NULL AND product_id IS NULL)
        OR (employee_id IS NULL AND product_id IS NOT NULL)
    )
);
```

이 예시에서는 각각의 외래 키가 대상 행의 존재를 검사하고, `CHECK`가 한 행에서 참조 열 하나만 채워졌는지 검사한다. [PostgreSQL 제약 조건](../../raw/database/2026-08-13-postgresql-constraints.md)

## 판단 순서

1. 참조할 대상 종류가 고정돼 있는지와 새 종류가 얼마나 자주 생기는지 확인한다.
2. 잘못된 참조를 DB 외래 키로 차단해야 하는지 결정한다.
3. 종류별 테이블 또는 고정된 외래 키 열로 요구사항을 표현할 수 있는지 검토한다.
4. 다형 참조를 택한다면 종류·ID를 함께 조회하고, 종류 이름 변경과 무효 참조를 어떻게 다룰지 정한다.

이 순서는 Rails의 기능 설명, GitLab의 운영 지침, PostgreSQL의 제약 조건을 합쳐 만든 **설계 검토용 질문**이다. [Rails 다형 연관](../../raw/database/rails-active-record-polymorphic-associations.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md); [PostgreSQL 제약 조건](../../raw/database/2026-08-13-postgresql-constraints.md)

## See Also

- [관계형 데이터베이스 정규화 원칙](database-normalization-principles.md)
