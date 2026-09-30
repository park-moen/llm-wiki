# DB 콘텐츠 다국어화 저장 구조 선택

> Sources: Ruby on Rails Guides, Unknown; Mobility, Unknown; Wagtail Documentation, Unknown; Shopify Developers, Unknown; Hibernate ORM User Guide, Unknown; PostgreSQL Global Development Group, Unknown; GitLab, Unknown
> Raw: [Rails I18n과 모델 콘텐츠](../../raw/database/rails-i18n-model-content.md); [Mobility 저장 방식](../../raw/database/mobility-translation-backends.md); [Wagtail 콘텐츠 번역](../../raw/database/wagtail-content-translations.md); [Shopify Translation](../../raw/database/shopify-translation-outdated.md); [Shopify Markets 폴백](../../raw/database/shopify-markets-localized-fallback.md); [Hibernate 조회 전략](../../raw/database/hibernate-fetch-strategies-n-plus-one.md); [PostgreSQL 키와 외래 키](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)
> Updated: 2026-09-30

## Overview

DB에 저장된 글·상품 설명처럼 **운영 중 바뀌는 콘텐츠**를 다국어로 제공할 때는 번역 컬럼, 도메인별 번역 테이블, JSON 저장 중 조회와 관리 방식에 맞는 구조를 선택한다. 언어 수만으로 결론을 내리기보다 원본 언어의 위치, 번역 승인·재검토 단위, 목록 조회와 정렬, 누락 시 폴백, DB 제약을 함께 결정해야 한다. UI 버튼·오류 문구를 locale 파일로 번역하는 일과 DB 콘텐츠를 번역하는 일은 별도 문제다. [Rails I18n과 모델 콘텐츠](../../raw/database/rails-i18n-model-content.md); [Mobility 저장 방식](../../raw/database/mobility-translation-backends.md)

## 먼저 무엇을 번역하는지 구분하기

**i18n (국제화)**은 여러 언어·지역 형식을 지원할 준비이고, 실제 번역문과 형식을 제공하는 일은 **L10n (현지화)**이다. UI 고정 문구는 locale 파일 같은 방식이 자연스럽지만, 운영자가 수정하는 게시글 제목은 DB 콘텐츠의 저장·조회·발행 규칙이 필요하다. Rails 공식 가이드도 일반 I18n API와 DB에 저장된 모델 콘텐츠 번역을 구분한다. [Rails I18n과 모델 콘텐츠](../../raw/database/rails-i18n-model-content.md)

## 저장 방식별로 유리한 조건

| 방식 | 형태 | 고려할 상황 | 비용과 제약 |
| --- | --- | --- | --- |
| 원본 테이블의 언어별 컬럼 | `title_ko`, `title_en` | 언어가 적고 고정돼 있으며 같은 행에서 바로 읽는 단순한 운영 | 언어·번역 필드·상태가 늘 때 원본 테이블의 컬럼과 마이그레이션이 늘어남 |
| 도메인별 번역 테이블 | `post` + `post_translation(post_id, locale, title, ...)` | 언어 추가나 번역 상태·발행 절차가 필요하고 원본 테이블 변경을 제한할 때 | 별도 행을 읽어야 하므로 JOIN·일괄 조회·폴백을 설계해야 함 |
| 원본 행의 JSON 컬럼 | `post.translations` 안의 locale별 값 | 번역 구조가 유동적이고 번역 필드별 검색·DB 제약이 적을 때 | locale별 값의 고유성·검증·검색 인덱스는 DB 기능과 조회 방식에 맞춰 별도 검토해야 함 |

Mobility는 이 세 저장 방식을 모두 지원한다. 위 표의 **적합한 상황과 비용은 저장 형태에서 도출한 설계 판단**이며, 어느 방식이 항상 빠르거나 옳다는 공식 성능 순위는 아니다. 언어가 적고 고정돼 있다면 컬럼 추가가 가장 단순할 수도 있고, 언어가 늘 수 있다는 이유만으로 번역 테이블을 택할 필요도 없다. [Mobility 저장 방식](../../raw/database/mobility-translation-backends.md)

## 도메인별 번역 테이블을 고른다면

원본 언어를 `post.title`에 두고 다른 언어만 번역 테이블에 넣는 방식은 기존 원본 데이터와 국문 조회 경로를 유지하기 쉽다. 반대로 모든 언어를 locale별 행으로 다루면 원본 언어도 같은 조회 모델에 넣을 수 있다. Wagtail은 콘텐츠의 `locale`과 공통 `translation_key`를 사용하고, 그 조합의 중복을 막는 실제 사례다. **기본 언어를 어디에 둘지는 두 모델 중 선택할 설계 결정**이다. [Mobility 저장 방식](../../raw/database/mobility-translation-backends.md); [Wagtail 콘텐츠 번역](../../raw/database/wagtail-content-translations.md)

아래 SQL은 **원본 언어를 원본 테이블에 두는 가상 설계 예시**다. `(post_id, locale)`를 키로 두면 글과 언어의 조합을 한 행으로 제한하고, `post_id`에 외래 키를 두면 원본 글과의 관계를 DB에서도 검사한다. 번역 상태와 원문 버전은 필요한 운영 규칙에 맞춰 조정할 수 있는 예시 필드다. [PostgreSQL 키와 외래 키](../../raw/database/postgresql-18-primary-and-foreign-keys.md); [Shopify Translation](../../raw/database/shopify-translation-outdated.md)

```sql
CREATE TABLE post (
    id bigint PRIMARY KEY,
    title text NOT NULL
);

CREATE TABLE post_translation (
    post_id bigint NOT NULL REFERENCES post (id),
    locale text NOT NULL,
    title text,
    review_status text NOT NULL,
    source_version text,
    PRIMARY KEY (post_id, locale)
);
```

`post_translation`처럼 도메인마다 테이블을 두는 것과, 모든 도메인의 번역을 `source_type`·`source_id`로 한 테이블에 모으는 것은 다른 결정이다. 여러 테이블을 가리키는 종류·ID 쌍은 일반적인 외래 키로 참조 무결성을 보장하기 어렵다. 공통 코드나 공통 entity 클래스를 재사용하더라도 DB 테이블까지 하나로 합칠 필요는 없다. [GitLab 다형 연관 지침](../../raw/database/gitlab-polymorphic-associations.md)

## 목록 조회와 폴백의 비용

원본 언어가 원본 테이블에 있으면 원본 언어 목록은 번역 테이블 없이 읽을 수 있다. 다른 언어의 목록은 원본 목록에 해당 locale의 번역 행을 `LEFT JOIN`하거나, 원본 목록의 ID를 모아 번역을 일괄 조회해 합칠 수 있다. 반면 각 행마다 번역을 단건 조회하면 N+1 문제가 생긴다. **테이블을 분리했다는 사실만으로 조회 횟수가 정해지지는 않는다.** Hibernate 공식 가이드도 개별 `SELECT`와 `JOIN`·`BATCH` 조회를 구분한다. [Hibernate 조회 전략](../../raw/database/hibernate-fetch-strategies-n-plus-one.md)

아래는 번역이 없거나 아직 게시 가능한 상태가 아닐 때 원본 제목을 보여 주는 **가상 조회 예시**다. 폴백을 허용할지, 번역 중인 콘텐츠를 숨길지는 제품 규칙으로 먼저 정해야 한다. [Shopify Markets 폴백](../../raw/database/shopify-markets-localized-fallback.md)

```sql
SELECT p.id, COALESCE(t.title, p.title) AS display_title
FROM post p
LEFT JOIN post_translation t
  ON t.post_id = p.id
 AND t.locale = :locale
 AND t.review_status = 'READY';
```

일괄 조회 방식에서는 목록 ID 집합과 locale을 함께 조건으로 넘긴다. 이 조회 경로를 실제로 구현해야 목록 건수에 비례한 단건 조회를 피할 수 있으며, `IN` 조건의 크기나 페이지네이션 방식도 확인해야 한다. 번역 제목으로 검색·정렬해야 한다면 원본 제목 폴백과 locale별 정렬까지 포함해 쿼리와 인덱스를 따로 검증한다. [Hibernate 조회 전략](../../raw/database/hibernate-fetch-strategies-n-plus-one.md)

폴백 순서도 명시한다. Shopify는 요청 지역 locale → 같은 언어의 기본 locale → 시장의 기본 언어 → 상점의 원본 언어 순서로 찾는 사례를 제공한다. 이는 Shopify의 제품 규칙이며, 자체 서비스에서는 필요한 단계와 번역이 없을 때의 동작을 따로 정해야 한다. **폴백 순서는 저장 방식만으로 자동 결정되지 않는다.** [Shopify Markets 폴백](../../raw/database/shopify-markets-localized-fallback.md)

## 원문 변경과 번역 상태

번역 행이 있다는 사실과 **현재 원문을 반영한 번역인지**는 다르다. Shopify의 번역 모델은 원문이 번역 이후 바뀌었는지 나타내는 `outdated` 필드를 둔다. 자체 DB에서도 원문 변경 시 재검토 상태로 전환하거나 원문 버전·digest를 기록할 수 있다. 이는 Shopify 방식을 그대로 복제해야 한다는 뜻이 아니라, 번역을 언제 다시 검토할지 정하라는 설계 판단이다. [Shopify Translation](../../raw/database/shopify-translation-outdated.md)

제목만 바뀌어도 본문까지 재검토해야 하는지에 따라 상태 단위를 **번역 행 전체** 또는 **번역 필드별**로 정한다. 행 단위는 단순하지만 불필요한 재검토가 늘 수 있고, 필드 단위는 정확하지만 상태 데이터와 처리 규칙이 늘어난다. 번역 누락·재검토 현황을 여러 도메인에 걸쳐 집계해야 한다면 도메인별 테이블의 결과를 모아야 하므로, 번역 테이블을 쓴다는 이유만으로 전체 현황이 단일 쿼리에서 자동으로 해결되지는 않는다. [Shopify Translation](../../raw/database/shopify-translation-outdated.md); [Mobility 저장 방식](../../raw/database/mobility-translation-backends.md)

## 결정 전에 확인할 질문

1. 번역 대상은 UI 고정 문구인가, 운영 중 수정되는 DB 콘텐츠인가? [Rails I18n과 모델 콘텐츠](../../raw/database/rails-i18n-model-content.md)
2. 언어·지역 수가 고정돼 있는가, 번역 상태와 발행 절차가 필요한가? [Mobility 저장 방식](../../raw/database/mobility-translation-backends.md); [Shopify Translation](../../raw/database/shopify-translation-outdated.md)
3. 기본 언어를 원본 테이블에 둘 것인가, 모든 언어를 같은 locale 모델로 다룰 것인가? [Wagtail 콘텐츠 번역](../../raw/database/wagtail-content-translations.md)
4. 목록·검색·정렬에서 어떤 언어를 얼마나 자주 읽고, 누락·재검토 시 무엇을 보여 줄 것인가? [Hibernate 조회 전략](../../raw/database/hibernate-fetch-strategies-n-plus-one.md); [Shopify Markets 폴백](../../raw/database/shopify-markets-localized-fallback.md)
5. 원본 테이블 변경, 조회 쿼리, 번역 관리 기능의 구현·운영 비용 중 무엇이 현재 제약인가? [Mobility 저장 방식](../../raw/database/mobility-translation-backends.md)

## See Also

- [Primary Key, Foreign Key와 복합 키](primary-foreign-and-composite-keys.md)
- [DB 설계에서 다형 참조](polymorphic-references.md)
