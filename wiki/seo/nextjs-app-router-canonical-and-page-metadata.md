# Next.js App Router에서 canonical과 페이지별 metadata 구현

> Sources: TNSP-1417 내부 작업 기록, 2026-09-08
> Raw: [Next.js App Router에서 self-canonical과 페이지별 metadata 구현](../../raw/seo/2026-09-08-nextjs-app-router-canonical-and-page-metadata.md)
> Updated: 2026-09-08

## Overview

Next.js App Router에서 사이트 전체의 자체 참조 canonical을 만들려면 현재 요청 경로를 metadata 생성 지점까지 전달하고, 하위 route가 metadata를 덮어쓰는 규칙을 고려해야 한다. 이 문서는 koreawhc.com의 TNSP-1417 작업에서 사용한 middleware 기반 구현과 production build 검증법을 정리하되, 운영 설정과 원문 검증 script에서 발견한 주의점도 함께 기록한다.

## 문제를 한 종류로 뭉치지 않는다

내부 점검에서는 사이트맵의 200 응답 URL 214개에 canonical 선언이 없었고, 고정 경로 70개 중 48개가 같은 title을 사용했다. 이 상태에서 리디렉션 URL, 쿼리 매개변수가 붙은 URL, 서로 다른 페이지의 중복 title이 함께 관찰됐다.

해결책은 원인별로 나눈다.

| 문제 | 주된 조치 |
| --- | --- |
| 같은 내용이 쿼리 매개변수 유무로 나뉨 | 매개변수를 제거한 대표 경로로 canonical을 통일한다. |
| 리디렉션 응답 URL이 사이트맵에 포함됨 | 사이트맵에서는 최종 200 응답 URL만 제공한다. |
| 서로 다른 페이지가 같은 title을 사용함 | route와 locale별 title·description을 작성한다. |

자체 참조 canonical만으로 서로 다른 페이지의 중복 title이 해결되는 것은 아니다. canonical 정리와 페이지별 metadata 작성을 함께 적용해야 한다. 또한 307이라는 상태 코드만으로 Google이 어느 URL을 대표로 선택할지 단정하지 않고, 리디렉션·사이트맵·canonical이 같은 최종 URL을 가리키게 만드는 데 집중한다.

## pathname을 root layout에 전달하기

작업 당시 App Router의 root layout `generateMetadata`에는 전체 pathname이 직접 전달되지 않았다. 모든 공개 route에 공통 canonical을 만들기 위해 middleware가 `req.nextUrl.pathname`을 내부 요청 헤더에 넣고, Server Component의 `headers()`에서 읽는 구조를 사용했다.

```ts
export function middleware(req: NextRequest) {
  const forwarded = new Headers(req.headers);
  forwarded.delete('x-pathname');
  forwarded.set('x-pathname', req.nextUrl.pathname);

  return NextResponse.next({ request: { headers: forwarded } });
}
```

구현할 때 확인할 점은 다음과 같다.

- `nextUrl.pathname`에는 query string이 포함되지 않으므로 `/stay?page=1`도 `/stay`를 기준으로 canonical을 만들 수 있다.
- middleware의 모든 공개 route 분기가 같은 전달 헤더를 사용해야 한다.
- 클라이언트가 보낸 같은 이름의 헤더를 재사용하지 않고 middleware가 계산한 값으로 덮어쓴다.
- `/admin`처럼 색인 대상이 아닌 route와 matcher 제외 경로에서는 canonical을 만들지 않는다.

## metadata 상속과 예외 처리

root layout에 기본 title template과 canonical을 선언하고, 하위 route에는 페이지별 title·description을 둔다. 하위 route가 `alternates`를 선언하면 상위의 `alternates` 객체 전체를 대체할 수 있으므로, 통합 대상처럼 canonical 예외가 필요한 페이지에서만 명시적으로 덮어쓴다.

```ts
export async function generateMetadata(): Promise<Metadata> {
  const pathname = (await headers()).get('x-pathname') ?? '';
  const canonical = isIndexablePath(pathname)
    ? new URL(pathname, siteUrl).toString()
    : null;

  return {
    title: { default: title, template: `%s | ${title}` },
    ...(canonical ? { alternates: { canonical } } : {}),
  };
}
```

`siteUrl`은 production에서 반드시 검증된 설정값을 사용한다. 원문의 `http://localhost:3000` fallback은 환경 변수 누락을 숨겨 운영 HTML에 잘못된 canonical을 내보낼 수 있으므로, production에서는 시작 단계에서 설정 오류를 드러내거나 canonical 생성을 중단하는 편이 안전하다.

`headers()` 사용은 route의 rendering 전략에 영향을 준다. 이미 동적 rendering을 사용하는 서비스에는 추가 비용이 없을 수 있지만, 정적 생성에 의존한다면 root layout 전체에 적용하기 전에 build 결과와 cache 동작을 확인해야 한다.

## 페이지별 metadata를 중앙에서 관리하기

공통 metadata만 상속하면 서로 다른 route가 같은 title을 갖게 된다. route와 locale을 key로 사용하는 문구 표와 factory를 두면 metadata 문구가 흩어지는 것을 막을 수 있다.

```ts
export function localeMeta(key: string, paramName?: string) {
  return async function generateMetadata({ params }): Promise<Metadata> {
    const values = await params;
    const locale: Locale = isLocale(values.locale) ? values.locale : 'ko';
    const fullKey = paramName ? `${key}/${values[paramName]}` : key;
    const copy = COPY[fullKey]?.[locale];

    return copy ? { title: copy.title, description: copy.description } : {};
  };
}
```

다국어 route에서는 화면 label이 같더라도 검색 결과에서 페이지와 언어를 구분할 수 있는 title인지 별도로 검토한다. DB 콘텐츠의 제목 자체가 중복되거나 번역 fallback 때문에 locale별 제목이 같다면 metadata code와 콘텐츠 운영 문제를 분리해 다룬다.

## 사이트맵은 최종 URL만 제공한다

사이트맵에는 색인할 최종 200 응답 URL만 넣는다. 하위 첫 페이지로 redirect하는 section root는 제외하고, 도착 URL의 canonical과 내부 link도 같은 주소를 사용한다.

307을 308로 바꾸는 일은 별도 결정이다. 영구 redirect는 client cache와 향후 route 재사용에 영향을 줄 수 있으므로, 단순히 Search Console 경고를 없애기 위한 수단으로 먼저 선택하지 않는다.

## production 응답으로 검증하기

개발 server는 첫 요청 시 route를 compile하므로 metadata 전수 검사 결과를 왜곡할 수 있다. 이 작업에서는 `next dev` 검사에서 title 없음으로 잡힌 URL 46개가 production build에서는 정상으로 확인됐다. 검증은 `next build && next start`로 띄운 응답 HTML을 대상으로 한다.

최소 검증 항목은 다음과 같다.

1. 사이트맵의 각 URL이 200으로 응답한다.
2. 각 HTML에 canonical이 하나 있고 의도한 절대 URL을 가리킨다.
3. query string을 붙여도 canonical은 대표 경로를 가리킨다.
4. 서로 다른 페이지의 title·description이 route와 locale을 구분한다.
5. 의도적으로 통합한 페이지는 자체 참조 대신 정한 대표 URL을 가리킨다.

### title 중복 집계 순서

원문 예시의 다음 pipeline은 먼저 URL을 포함한 전체 행을 정렬한 뒤 title을 추출한다.

```bash
LC_ALL=C sort probe.tsv | cut -f3 | LC_ALL=C uniq -c
```

`uniq`가 같은 title을 세려면 title을 먼저 추출한 뒤 정렬해야 한다.

```bash
cut -f3 probe.tsv | LC_ALL=C sort | LC_ALL=C uniq -c | awk '$1 > 1'
```

한글 title을 byte 순서로 안정적으로 집계하려면 `LC_ALL=C`를 유지한다. 전수 요청이 middleware의 rate limit에 걸리면 요청 간격을 두고, HTTP 상태 코드는 HTML 본문과 별도로 수집한다. 단순한 `grep` 기반 HTML 추출은 빠른 점검에는 쓸 수 있지만 attribute 순서나 markup 형태가 달라질 수 있으므로 최종 검증에서는 HTML parser나 test code를 쓰는 편이 안정적이다.

## 적용 순서

1. 대표 URL 정책과 색인 제외 route를 정한다.
2. middleware가 검증된 pathname을 root layout에 전달하게 한다.
3. root layout에 기본 canonical과 title template을 둔다.
4. route·locale별 title과 description을 작성한다.
5. 사이트맵에서 redirect URL을 제거한다.
6. production build의 sitemap URL과 query variant를 전수 검사한다.
7. 배포 후 Search Console 재검증은 재크롤링과 별개라는 점을 고려해 기다린다.

## See Also

- [canonical URL: 중복 페이지의 대표 주소 정하기](canonical-url-consolidation.md)
