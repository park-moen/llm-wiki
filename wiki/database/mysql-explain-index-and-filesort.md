# MySQL EXPLAIN의 인덱스와 Using filesort 읽기

> Sources: MySQL 8.4 Reference Manual, Unknown
> Raw: [EXPLAIN Statement](../../raw/database/mysql-8-4-explain-statement.md); [EXPLAIN Output Format](../../raw/database/mysql-8-4-explain-output-format.md); [ORDER BY Optimization](../../raw/database/mysql-8-4-order-by-optimization.md)
> Updated: 2026-09-29

## Overview

`EXPLAIN`은 MySQL이 SQL을 어떻게 처리할지 보여 주는 실행 계획이다. `key`는 선택한 인덱스, `rows`는 살펴볼 것으로 예상하는 행 수, `Extra`의 `Using filesort`는 결과 순서를 맞추기 위한 추가 정렬을 뜻한다. **행을 찾는 데 인덱스를 사용해도 정렬에는 별도 작업이 필요할 수 있다.** `filesort`라는 이름만으로 디스크에 파일을 썼거나 쿼리가 느리다고 판단할 수는 없다. [EXPLAIN Statement](../../raw/database/mysql-8-4-explain-statement.md); [EXPLAIN Output Format](../../raw/database/mysql-8-4-explain-output-format.md); [ORDER BY Optimization](../../raw/database/mysql-8-4-order-by-optimization.md)

## 먼저 SQL을 어떻게 처리하는지 보기

```sql
EXPLAIN
SELECT id, title
FROM t_post
WHERE author_id = 'kim'
ORDER BY title;
```

`EXPLAIN`은 이 SQL에 대해 MySQL optimizer가 고른 계획을 보여 준다. 일반 `EXPLAIN`의 `rows`는 추정치다. 실제 실행 시간과 처리 행 수까지 보려면 `EXPLAIN ANALYZE`를 쓸 수 있는데, 이 명령은 SQL을 **실제로 실행**한다. 아래처럼 `SELECT`를 대상으로 먼저 읽는 편이 안전하다. [EXPLAIN Statement](../../raw/database/mysql-8-4-explain-statement.md); [EXPLAIN Output Format](../../raw/database/mysql-8-4-explain-output-format.md)

```sql
EXPLAIN ANALYZE
SELECT id, title
FROM t_post
WHERE author_id = 'kim'
ORDER BY title;
```

## `key`와 `Using filesort`가 함께 나오는 시뮬레이션

아래 테이블과 행은 공식 문서의 **인덱스로 행을 찾지만 `ORDER BY`에는 추가 정렬이 필요한 경우**를 설명하려고 만든 가상 예시다. 실제 MySQL에서 측정한 실행 계획이나 시간은 아니다. [ORDER BY Optimization](../../raw/database/mysql-8-4-order-by-optimization.md)

| id | author_id | title |
| --- | --- | --- |
| a | kim | Zebra |
| b | lee | Garden |
| c | kim | Moon |
| d | kim | Apple |

`author_id`에만 인덱스 `idx_author`가 있다고 가정하자. 앞의 SQL은 `author_id = 'kim'`인 행을 찾은 뒤 `title` 오름차순으로 결과를 돌려줘야 한다.

1. MySQL이 `idx_author`를 **행 찾기**에 선택했다고 가정하면, `kim`에 해당하는 `Zebra`, `Moon`, `Apple`을 찾는다. 실행 계획의 `key`에는 `idx_author`가 표시될 수 있다.
2. `idx_author`는 `author_id`에 대한 인덱스이므로, 찾은 행들의 **`title` 순서**까지 맞춰 준다고 볼 수 없다.
3. MySQL이 결과를 `Apple`, `Moon`, `Zebra` 순서로 정렬하면 `Extra`에 `Using filesort`가 표시된다. 이 추가 정렬은 메모리에서 끝날 수도 있고, 데이터가 메모리에 들어가지 않으면 임시 디스크 파일을 사용할 수도 있다.

프런트엔드 코드에 비유하면 먼저 `authorId === 'kim'`으로 대상을 좁히고, 남은 배열을 `title`로 정렬하는 두 단계다. 인덱스는 앞 단계를 도왔지만 뒤 단계까지 없애지는 못한 셈이다. 이 설명은 MySQL의 실제 내부 코드를 그대로 옮긴 것이 아니라 두 작업의 차이를 보여 주는 비유다. [EXPLAIN Output Format](../../raw/database/mysql-8-4-explain-output-format.md); [ORDER BY Optimization](../../raw/database/mysql-8-4-order-by-optimization.md)

### 정렬까지 인덱스로 해결하려면

`(author_id, title)` 순서의 복합 인덱스가 있고 MySQL이 그 인덱스를 선택하면, 같은 `author_id`에 속하는 행을 `title` 순서로 읽어 추가 정렬을 피할 수 있다. 공식 문서도 인덱스의 앞부분을 상수로 제한하고 다음 부분으로 `ORDER BY`하는 경우를 인덱스 정렬의 예로 든다. 실제 선택은 쿼리와 데이터에 따라 달라지므로 인덱스를 추가한 뒤 다시 `EXPLAIN`으로 확인한다. [ORDER BY Optimization](../../raw/database/mysql-8-4-order-by-optimization.md)

```sql
CREATE INDEX idx_author_title ON t_post (author_id, title);
```

## 실행 계획에서 읽을 항목

| 항목 | 처음 읽을 때의 뜻 |
| --- | --- |
| `possible_keys` | MySQL이 행을 찾을 때 고려할 수 있는 인덱스 후보 |
| `key` | MySQL이 실제로 선택한 인덱스. `PRIMARY`라면 기본 키 인덱스이고, `NULL`이면 더 효율적인 실행에 사용할 인덱스를 찾지 못했다는 뜻 |
| `key_len` | 선택한 인덱스에서 사용한 길이. 복합 인덱스의 어느 부분까지 쓰는지 살피는 단서 |
| `rows` | MySQL이 살펴봐야 한다고 예상한 행 수. InnoDB에서는 실제 행 수와 다를 수 있음 |
| `Extra: Using filesort` | `ORDER BY` 순서를 만들기 위해 추가 정렬을 수행함 |
| `Extra: Using index` | 필요한 열을 인덱스에서만 읽음. `Using filesort`와 다른 의미 |

여기서 `key`는 **기본 키(PK)라는 뜻으로만 쓰는 말이 아니다.** MySQL이 선택한 인덱스의 이름이다. `Using index`도 단순히 `key`에 값이 있다는 뜻이 아니라, 필요한 열을 인덱스에서만 읽는 방식이라는 뜻이다. [EXPLAIN Output Format](../../raw/database/mysql-8-4-explain-output-format.md)

## 성능을 판단할 때

`Using filesort`가 보이면 **추가 정렬이 왜 필요한지**를 확인한다. `WHERE`와 `ORDER BY` 열이 어떤 인덱스에 들어 있는지, 정렬 대상이 얼마나 되는지, 실제 실행 시간이 어떤지 순서대로 본다. `filesort`가 메모리에서 처리될 수도 있으므로 문구 하나만으로 디스크 사용이나 병목을 확정하지 않는다. `EXPLAIN ANALYZE`는 예상 행 수와 실제 행 수·시간을 비교할 때 쓴다. [EXPLAIN Statement](../../raw/database/mysql-8-4-explain-statement.md); [ORDER BY Optimization](../../raw/database/mysql-8-4-order-by-optimization.md)
