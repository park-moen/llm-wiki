# 거래 시점 스냅샷과 비정규화

> Sources: Oracle, Unknown; Microsoft, Unknown
> Raw: [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md); [Oracle 원래 가격과 현재 가격](../../raw/database/oracle-original-versus-current-pricing.md); [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md)
> Updated: 2026-09-28

## Overview

주문에 상품 가격이나 이름을 다시 저장하는 방식을 흔히 “스냅샷”이라고 부르지만, **스냅샷이라는 이름만으로 정규화 위반을 판정할 수는 없다.** 현재 값을 계속 복제한 데이터라면 중복 관리가 생긴다. 반면 주문 당시 확정된 가격은 현재 상품 가격과 다른 사실이다. Oracle의 정규화 예시도 상품명은 상품 쪽으로 분리하고, 주문마다 달라질 수 있는 판매 가격은 주문 항목에 남긴다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md)

## 먼저 값의 의미와 갱신 규칙을 정한다

| 저장 값 | 뜻 | 갱신 규칙 | 정규화 관점 |
|---|---|---|---|
| `product.current_name` | 지금 판매 중인 상품의 이름 | 상품 정보가 바뀌면 갱신 | 상품에 속하는 현재 사실 |
| `order_line.product_name_copy` | 상품의 **현재 이름**을 주문에도 복사 | 상품 이름이 바뀔 때 함께 갱신해야 함 | 같은 현재 사실을 중복 저장하는 비정규화 |
| `order_line.name_at_purchase` | 구매 당시 주문서에 표시해 확정한 이름 | 이후 상품명 변경과 독립적으로 보존 | 업무상 별도의 과거 사실이면 중복 저장으로 단정할 수 없음 |
| `order_line.charged_unit_price` | 해당 주문 항목에 실제 적용한 단가 | 주문 확정 후 과거 거래 기준으로 보존 | 주문 항목에 속하는 거래 사실 |

위 표의 열 이름은 설명을 위한 예시다. 세 번째 행의 해석은 **업무에서 구매 당시 표시명을 실제로 보존해야 한다는 조건이 있을 때** 성립한다. 단지 화면 조회를 빠르게 하려고 현재 상품명을 복사하고 계속 동기화한다면 두 번째 행에 가깝다. 이는 Oracle의 주문 모델과 Microsoft의 중복 데이터 설명을 적용한 설계 판단이다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md); [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md)

## 실제 거래 가격을 남기는 이유

Oracle의 주문 예시에서는 할인 등에 따라 **같은 상품도 주문마다 청구 가격이 달라질 수 있으므로** 판매 가격을 주문 항목에 둔다. Oracle Commerce 문서는 이미 제출된 주문을 수정하거나 교환 주문의 가격을 계산할 때 원래 주문에 적용한 가격을 사용하고, 새로 추가한 상품에는 현재 가격을 적용한다고 설명한다. 과거 주문의 결제·반품 근거를 현재 상품 가격으로 다시 계산하면 당시 거래와 달라질 수 있다는 뜻이다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md); [Oracle 원래 가격과 현재 가격](../../raw/database/oracle-original-versus-current-pricing.md)

예를 들어 `product.list_price`가 지금의 판매 기준 가격이고 `order_line.charged_unit_price`가 주문 때 확정된 단가라면, 두 열은 한때 숫자가 같아도 갱신 규칙이 다르다. `product_id`만으로 주문별 실제 단가가 결정되지 않으므로, 주문 항목의 가격을 상품 테이블로 옮기는 것은 거래 사실을 잃게 만든다. 이 설명은 Oracle의 주문별 가격 사례에서 도출한 예시다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md)

## 설계할 때 확인할 질문

1. 저장하려는 값이 **현재의 상품 사실**, **주문 당시 확정된 사실**, **조회용 사본** 중 무엇인가?
2. 원본이 바뀌었을 때 이 값도 바뀌어야 하는가? 함께 바뀌어야 한다면 중복 저장과 동기화 비용을 검토한다.
3. 주문 당시 값을 나중에 재구성할 수 있는가? 재구성할 수 없거나 거래 증빙에 당시 값이 필요하다면 주문 항목에 기록할 근거가 있다.
4. 주문 시점의 가격·이름을 기록하기로 했다면 어떤 시점에 확정하며, 취소·정정은 기존 기록을 바꿀지 별도 이력으로 남길지 업무 규칙을 정한다.

이 질문들은 원문에 있는 공식 체크리스트가 아니라, 정규화 원칙과 Oracle의 주문 가격 동작을 실제 스키마 검토에 적용한 **실무용 판단 절차**다. 스냅샷 열 전체를 “정규화 위반”이라고 부르면 현재 사실의 사본과 과거 거래 사실을 혼동하기 쉽다. [Oracle 논리 설계](../../raw/database/oracle-data-warehousing-logical-design.md); [Oracle 원래 가격과 현재 가격](../../raw/database/oracle-original-versus-current-pricing.md); [Microsoft 정규화 기초](../../raw/database/microsoft-database-normalization-basics.md)

## See Also

- [관계형 데이터베이스 정규화 원칙](database-normalization-principles.md)
