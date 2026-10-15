# P6Spy SQL 로그 읽기와 설정

> Sources: P6Spy 공식 문서·설정 예제·구현 소스, Unknown
> Raw: [Configuration and Usage](../../raw/spring/p6spy-configandusage.md); [Installation](../../raw/spring/p6spy-install.md); [Known Issues](../../raw/spring/p6spy-knownissues.md); [spy.properties](../../raw/spring/p6spy-spy-properties.md); [시간 변환 구현](../../raw/spring/p6spy-log-time-implementation.md)
> Updated: 2026-10-13

## Overview

P6Spy는 JDBC 호출을 감싸 DB 관련 활동을 가로채고 로그로 기록하는 도구다. SQL과 바인딩 값, JDBC 메서드 호출 시간, 연결 식별자 등을 확인할 수 있다. 로그를 켜는 설정과 JDBC driver·DataSource에 P6Spy를 연결하는 설정이 함께 필요하다. JPA를 쓰는 애플리케이션에서도 JDBC 호출을 관찰하는 방식으로 이해한다. [Installation](../../raw/spring/p6spy-install.md); [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

FE의 요청 interceptor로 요청 내용을 관찰하는 것처럼, 애플리케이션과 JDBC driver 사이에서 DB 호출을 관찰한다고 생각할 수 있다. 이는 동작을 이해하기 위한 비유다. **로그에 보이는 시간은 API 전체 응답 시간이나 DB 내부 실행 계획을 대신하지 않는다.** [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

## 기본 로그의 항목 읽기

공식 문서는 기본 `SingleLineFormat`을 다음 순서로 설명한다. 실제 프로젝트에서 formatter를 교체하거나 SLF4J의 시간·thread·logger 접두를 추가하면 보이는 형태가 달라질 수 있으므로, 먼저 `logMessageFormat`을 확인한다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

```text
current time|execution time|category|connection id|statement SQL String|effective SQL string
```

| 항목 | 의미 | 해석할 때 주의할 점 |
| --- | --- | --- |
| current time | 로그가 기록되는 시각 | `dateformat` 설정에 따라 표현이 달라진다. |
| execution time | 해당 JDBC 메서드 호출에 걸린 시간 | SQL의 전체 비용이나 HTTP 요청 시간과 같지 않다. |
| category | `statement`, `commit`, `result` 등 활동 종류 | 로그 한 줄을 곧바로 SELECT 한 번으로 세면 안 된다. |
| connection id | 활동이 기록된 연결의 식별자 | 요청 ID·사용자 ID·트랜잭션 ID로 해석하지 않는다. |
| statement SQL string | statement에 전달된 원래 SQL | PreparedStatement에서는 `?`가 남아 있을 수 있다. |
| effective SQL string | 바인딩 값을 넣어 표시한 SQL | 읽기 위한 문자열이며, DB는 PreparedStatement를 받는다. |

[Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

연결 식별자가 HTTP 요청이나 트랜잭션을 고유하게 식별한다고 볼 근거는 없다. 요청별 분석을 하려면 애플리케이션 로그의 요청 식별자·처리 시각 등과 함께 대조해야 한다. 이는 공식 문서가 connection id를 연결 식별자로 정의한다는 사실에서 도출한 분석 지침이다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

## 바인딩 값이 들어간 SQL의 의미

PreparedStatement에서는 SQL 템플릿과 바인딩 값을 따로 전달한다. 공식 문서의 `SELECT * FROM MYTABLE WHERE THISCOL = ?`처럼 원래 SQL에 `?`가 남아 있는 것은 정상이다. P6Spy는 실제 값이 들어간 모양을 로그로 만들어 확인을 돕지만, 이것을 애플리케이션이 값을 문자열로 이어 붙여 실행했다는 증거로 읽으면 안 된다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

특히 **기본 형식을 설명하는 항목 이름과 custom formatter의 placeholder 이름을 구분한다.** `%(effectiveSql)`은 driver에 제출한 원래 SQL이고, `%(sql)`은 바인딩 값을 채운 표시용 SQL이다. 이름만 보고 반대로 해석하기 쉬우므로 설정할 때 아래 대응을 확인한다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md); [spy.properties](../../raw/spring/p6spy-spy-properties.md)

| CustomLineFormat placeholder | 내용 |
| --- | --- |
| `%(effectiveSql)` | driver에 전달한 원래 SQL |
| `%(effectiveSqlSingleLine)` | 원래 SQL에서 줄바꿈을 제거한 것 |
| `%(sql)` | 바인딩 값을 채운 표시용 SQL |
| `%(sqlSingleLine)` | 바인딩 값을 채우고 줄바꿈을 제거한 것 |

DB 콘솔에서 재현할 때도 로그 문자열만으로 원래 JDBC 호출을 완전히 재현했다고 단정하지 않는다. 바인딩 타입·연결 설정·트랜잭션 조건까지 같다는 보장은 없기 때문이다. 이는 로그 표현과 실제 PreparedStatement를 구분한 실무 판단이다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

## 실행 시간을 어디까지 믿을 것인가

공식 문서는 execution time을 **특정 메서드가 실행되는 데 걸린 시간**으로 설명한다. 예를 들어 PreparedStatement의 `.execute()` 호출과 ResultSet의 `.next()` 호출은 별도의 활동이다. 따라서 statement 로그의 시간 하나로 결과 전체 읽기, entity 변환, 비즈니스 로직, HTTP 응답까지 모두 포함됐다고 해석하면 안 된다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

수집한 `P6LogQuery` 구현은 나노초 단위로 받은 경과 시간을 `TimeUnit.NANOSECONDS.toMillis(elapsedNanos)`로 변환해 logger에 전달한다. **측정에 사용한 단위와 최종 로그의 표시 단위는 구분한다.** 설정 예제의 `useNanoTime` 설명만 보고 로그 숫자가 나노초라고 단정하지 않고, 적용 버전의 구현과 formatter를 확인한다. [시간 변환 구현](../../raw/spring/p6spy-log-time-implementation.md); [spy.properties](../../raw/spring/p6spy-spy-properties.md)

이 구현의 정수 밀리초 변환을 고려하면 로그에 `0`이 나온다고 실제 비용이 전혀 없다는 뜻은 아니다. 또한 JDBC 호출 경과 시간에는 driver와 DB 호출 경로가 포함되므로, 이를 DB 서버 내부의 순수 연산 시간으로 단정할 수 없다. 앞 문장은 시간 변환 코드에서, 뒤 문장은 JDBC 메서드 호출을 측정한다는 설명에서 도출한 해석이다. [시간 변환 구현](../../raw/spring/p6spy-log-time-implementation.md); [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

## 로그 출력 설정 예제

공식 설치 문서는 P6Spy와 `spy.properties`를 classpath에서 사용할 수 있게 하고, driver 또는 DataSource 경로에 P6Spy를 연결하도록 안내한다. driver 방식의 예로 JDBC URL에 `p6spy:`를 추가하고 `com.p6spy.engine.spy.P6SpyDriver`를 사용하는 방법이 있다. DataSource 방식은 별도의 wrapper 설정을 따른다. 아래 로그 설정만 추가하는 것으로 JDBC 호출이 자동 연결되는 것은 아니다. [Installation](../../raw/spring/p6spy-install.md)

다음은 공식 옵션을 조합한 **학습용 `spy.properties` 예제**다. 특정 Spring Boot starter의 설정이 아니라 P6Spy 자체 설정이며, 실제 애플리케이션의 연결 방식과 logging 설정에 맞춰 적용해야 한다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md); [spy.properties](../../raw/spring/p6spy-spy-properties.md)

```properties
appender=com.p6spy.engine.spy.appender.Slf4JLogger
logMessageFormat=com.p6spy.engine.spy.appender.CustomLineFormat
customLogMessageFormat=%(currentTime)|%(executionTime)|%(category)|connection%(connectionId)|%(sqlSingleLine)
excludecategories=info,debug,result,resultset,batch
```

`Slf4JLogger`는 SLF4J의 `p6spy` logger를 사용하므로 애플리케이션의 logging 설정에서 해당 logger가 출력되도록 해야 한다. `CustomLineFormat`을 선택해야 `customLogMessageFormat`이 적용된다. `MultiLineFormat`은 값을 채운 SQL을 별도 줄로 보여주고, `SingleLineFormat`은 기본 한 줄 형식이다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

위 예제의 제외 목록은 공식 기본 제외 목록을 명시한 것이다. `statement`뿐 아니라 `commit`·`rollback`도 남을 수 있으며, result·resultset·batch는 숨긴다. 필요한 진단이 무엇인지에 따라 제외 목록을 조정한다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md); [spy.properties](../../raw/spring/p6spy-spy-properties.md)

## 필터와 임계값이 분석에 미치는 영향

- `excludecategories`: 활동 종류별로 로그를 제외한다.
- `filter`와 `include`·`exclude`·`sqlexpression`: SQL 문자열 조건으로 출력 범위를 정한다.
- `executionThreshold`: 일정 시간 기준을 넘는 호출만 출력하는 데 사용한다.
- `excludebinary`: 바이너리 값을 `[binary]`로 표시할지 정한다.

[Configuration and Usage](../../raw/spring/p6spy-configandusage.md); [spy.properties](../../raw/spring/p6spy-spy-properties.md)

수집한 구현은 경과 시간을 밀리초로 변환한 값이 `executionThreshold`보다 **클 때** 출력 조건을 충족한다. 임계값이 양수가 아니면 이 시간 제한을 적용하지 않는다. 따라서 임계값을 켠 로그에서 보이는 SQL 수는 전체 실행 SQL 수와 다를 수 있다. 빠른 쿼리가 반복되는 문제를 조사할 때는 시간 임계값과 제외 조건을 먼저 확인한다. [시간 변환 구현](../../raw/spring/p6spy-log-time-implementation.md)

## N+1과 느린 SQL을 구분해 조사하기

같은 요청 처리 중 비슷한 SELECT가 바인딩 값만 바뀌며 반복되는지 확인하면 추가 조회 문제를 찾는 데 도움이 된다. 다만 반복 로그만으로 N+1을 확정하지 않고, 호출 범위와 의도된 조회인지 확인한다. 기존 JPA 문서는 역방향 연관관계의 추가 조회가 목록에서 반복될 수 있다고 설명한다. **P6Spy는 실제 호출을 관찰하는 근거이고, 원인과 해결책은 매핑·조회 로직을 함께 읽어 판단한다.** [JPA 지연 로딩](jpa-to-one-lazy-loading-and-unique-foreign-key.md); [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

느린 SQL이 있다면 SQL·바인딩 값·발생 빈도를 확보한 뒤 DB 실행 계획을 확인한다. 실행 시간 하나로 인덱스 누락이나 정렬 문제를 확정하지 않는다. P6Spy 로그와 `EXPLAIN`은 서로 다른 정보를 제공한다. 이 조사 순서는 JDBC 로그의 측정 범위와 기존 실행 계획 문서를 연결한 실무 판단이다. [MySQL EXPLAIN](../database/mysql-explain-index-and-filesort.md); [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

## 관찰 범위와 로그 공유 시 주의점

공식 Known Issues는 driver 고유 클래스로 직접 cast하는 코드가 P6Spy proxy 때문에 실패할 수 있다고 설명한다. `unwrap`으로 원래 객체를 얻을 수 있지만, 그 객체로 직접 수행한 동작은 P6Spy가 기록하지 못한다. 저장 프로시저의 OUT parameter도 로그 시점의 차이 때문에 기록되지 않는다고 명시한다. **로그가 없다는 것만으로 DB 활동이 없었다고 단정하지 않는다.** [Known Issues](../../raw/spring/p6spy-knownissues.md)

바인딩 값을 보여주는 로그에는 실제 데이터가 포함된다. 진단·공유할 때 필요한 값만 남기고 개인정보나 인증 관련 값은 별도로 가리는 편이 안전하다. 이는 P6Spy의 값 출력 기능에서 도출한 운영 판단이며, `excludebinary`가 문자열 값까지 모두 가려주는 설정은 아니다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md)

로그 출력량과 SQL 문자열 구성에는 비용이 들 수 있으므로, 로그를 켠 환경의 시간을 운영 환경의 비용으로 그대로 일반화하지 않는다. 공식 문서는 필터·출력 대상·형식을 조정하는 옵션을 제공하지만, 모든 환경에 적용되는 고정 성능 비용을 제시하지는 않는다. 측정 조건과 설정을 함께 기록한다. 이 문서는 실제 앱을 설치하거나 성능을 측정한 결과가 아니라 공식 문서와 구현을 정리한 자료다. [Configuration and Usage](../../raw/spring/p6spy-configandusage.md); [시간 변환 구현](../../raw/spring/p6spy-log-time-implementation.md)

## See Also

- [JPA 1:1 매핑 선택과 지연 로딩](jpa-to-one-lazy-loading-and-unique-foreign-key.md)
- [MySQL EXPLAIN의 인덱스와 Using filesort 읽기](../database/mysql-explain-index-and-filesort.md)
- [DB 콘텐츠 다국어화 저장 구조 선택](../database/multilingual-content-schema-design.md)
