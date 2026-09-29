# 외래 키의 `ON DELETE CASCADE`와 `ON DELETE SET NULL`

> Sources: PostgreSQL Global Development Group, Unknown
> Raw: [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)
> Updated: 2026-09-29

## Overview

`ON DELETE`는 **참조받는 행을 삭제할 때**, 그 행을 가리키는 외래 키가 있는 행을 어떻게 처리할지 정한다. 외래 키는 참조하는 테이블에 선언한다. `CASCADE`는 참조하는 행도 삭제하고, `SET NULL`은 참조하는 행을 남기면서 외래 키 열을 `NULL`로 바꾼다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)

| 동작 | 참조받는 행 삭제 시 참조하는 행 | 관계에 맞는 경우 |
|---|---|---|
| `ON DELETE CASCADE` | 함께 삭제 | 자식 행이 부모의 구성 부분이고 독립적으로 존재할 수 없을 때 |
| `ON DELETE SET NULL` | 남겨 두고 외래 키 값을 `NULL`로 변경 | 연결이 선택 사항이고 참조 대상이 없어져도 행은 유지해야 할 때 |

이 선택 기준은 PostgreSQL 문서의 주문·주문 항목과 상품·상품 담당자 예시를 요약한 것이다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)

## `ON DELETE CASCADE`: 주문을 지울 때 주문 항목도 삭제

PostgreSQL 문서에서는 `order_items.order_id`가 `orders`를 참조하며 `ON DELETE CASCADE`를 사용한다. 주문 항목은 주문의 구성 부분이므로 주문 행을 삭제하면 이를 참조하는 주문 항목 행도 자동으로 삭제된다. 반면 같은 주문 항목의 상품 참조에는 `ON DELETE RESTRICT`를 사용해, 상품 삭제가 주문 항목 삭제로 이어지지 않게 한다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)

```sql
CREATE TABLE order_items (
    product_no integer REFERENCES products ON DELETE RESTRICT,
    order_id integer REFERENCES orders ON DELETE CASCADE,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```

## `ON DELETE SET NULL`: 선택적인 연결만 끊기

PostgreSQL 문서는 상품에 연결된 담당자 행이 삭제되더라도 상품은 남겨야 하는 관계를 `SET NULL`의 사용 사례로 든다. 다음 SQL은 그 설명에서 만든 **설계 예시**다. 담당자 행을 삭제하면 상품 행은 유지되고 `manager_id`만 `NULL`이 된다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)

```sql
CREATE TABLE product_managers (
    manager_id integer PRIMARY KEY
);

CREATE TABLE products (
    product_no integer PRIMARY KEY,
    manager_id integer REFERENCES product_managers ON DELETE SET NULL
);
```

`SET NULL`을 선택해도 다른 열 제약은 지켜야 한다. 외래 키 열을 `NOT NULL`로 선언했다면 삭제 동작이 그 열을 `NULL`로 바꾸려 할 때 제약과 충돌한다. 따라서 연결을 끊은 뒤에도 행을 유지할 수 있는 관계인지, 외래 키 열에 `NULL`을 허용하는지 함께 확인한다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)

## 선택할 때 확인할 점

- **자식 행의 독립성:** 주문 항목처럼 부모 없이 의미가 없다면 `CASCADE`를 검토한다. 상품과 주문처럼 독립적인 대상이라면 한쪽 삭제가 다른 쪽의 기록을 지워도 되는지 먼저 판단한다. PostgreSQL 문서는 이런 관계에 `RESTRICT` 또는 `NO ACTION`을 더 적절한 선택으로 설명한다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)
- **선택적 연결 여부:** 참조 대상만 사라지고 행은 유지해야 한다면 `SET NULL`을 검토한다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)
- **기본 동작:** PostgreSQL에서 `ON DELETE`를 생략하면 `NO ACTION`이다. 참조하는 행이 남아 외래 키를 만족하지 못하면 보통 삭제가 오류로 끝난다. [PostgreSQL 외래 키 삭제 동작](../../raw/database/postgresql-18-foreign-key-on-delete-actions.md)

## See Also

- [DB 설계에서 다형 참조](polymorphic-references.md)
