# 관계형 데이터베이스 정규화 원칙

> Sources: Microsoft, Unknown; Oracle, Unknown
> Raw: [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md); [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md)
> Updated: 2026-09-28

## Overview

정규화는 관계형 데이터에서 **하나의 사실을 어디에 저장할지** 정하고, 중복 저장으로 생기는 삽입·수정·삭제 이상을 줄이는 설계 과정이다. 테이블을 무조건 잘게 나누는 규칙이라기보다, 각 열이 해당 행의 식별자에 어떤 의미로 의존하는지 확인하는 과정이다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md); [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md)

## 정규화가 막으려는 문제

고객의 **현재 주소**를 고객 테이블과 여러 주문 행에 같은 의미로 저장하고 모두 최신 상태로 유지하려면, 주소 변경 때 모든 사본을 함께 수정해야 한다. 일부만 수정되면 어느 값이 맞는지 알 수 없게 된다. 이를 수정 이상이라고 한다. 중복된 사실을 한곳으로 모으면 삽입과 삭제 때도 특정 사실이 다른 행의 존재에 종속되는 문제를 줄일 수 있다. [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md); [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md)

핵심 질문은 **값이 같은가**보다 **같은 시점의 같은 사실을 뜻하는가**이다. 예를 들어 상품의 현재 표시명은 상품에 관한 사실이고, 주문 당시 실제 판매 가격은 주문 항목에 관한 사실이다. Oracle의 판매 주문 예시는 상품명을 상품 쪽으로 분리하면서도 주문별 할인 등에 따라 달라지는 판매 가격은 주문 항목에 둔다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md)

## 정규형을 읽는 실무 기준

| 단계 | 확인할 질문 | 판매 주문 예시 |
|---|---|---|
| 제1정규형 | 반복되는 열 묶음이나 한 칸에 여러 항목을 넣었는가? | 주문의 상품들을 `item1`, `item2`로 두지 않고 주문 항목 행으로 분리한다. |
| 제2정규형 | 복합 키를 쓰는 행의 비키 열이 키 전체에 의존하는가? | 상품명처럼 상품 자체에 속한 값은 주문 항목에서 상품으로 분리한다. |
| 제3정규형 | 비키 열이 다른 비키 열을 거쳐 간접적으로 결정되는가? | 고객 ID로 결정되는 현재 고객 정보는 주문 행에 반복해서 넣지 않는다. |

위 구분은 Oracle과 Microsoft의 입문용 설명을 요약한 것이다. 실제 판정에서는 먼저 후보 키와 업무상 함수 종속을 정의해야 한다. 열 이름이나 테이블 수만으로 정규형을 판정할 수 없다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md); [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md)

## 의도적인 비정규화

조회 성능이나 분석 편의를 위해 같은 현재 사실을 여러 곳에 물리적으로 저장할 수 있다. Oracle은 데이터 웨어하우스의 비정규화 테이블을 이런 사례로 설명한다. 이 경우 중복 값의 갱신 책임과 불일치 가능성을 설계에서 다뤄야 한다. 정규화를 포기하는 이유가 있는지와 그 비용을 확인하는 것이 우선이다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md); [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md)

반면 거래가 확정될 때 보존한 판매 가격처럼 **시점과 대상이 다른 사실**은 현재 상품 가격과 값이 겹친다고 해서 곧바로 비정규화가 되지 않는다. 이 구분은 [거래 시점 스냅샷과 비정규화](transaction-snapshot-and-denormalization.md)에서 다룬다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md)

## See Also

- [거래 시점 스냅샷과 비정규화](transaction-snapshot-and-denormalization.md)
