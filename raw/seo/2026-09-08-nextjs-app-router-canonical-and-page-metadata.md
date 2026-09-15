# Next.js App Router에서 self-canonical과 페이지별 metadata 구현

> Source: 내부 작업 기록 (`private/2026-09-08-nextjs-app-router-canonical-and-page-metadata.md`)
> Collected: 2026-09-08
> Published: 2026-09-08

> **상태: 미검토 작업 메모.** 위키 검토를 거치지 않았고 `wiki/` 에 등재되지 않았다.
> **외부 원본 없음.** 이 글은 ingest 한 자료가 아니라, koreawhc.com 의 TNSP-1417
> (canonical 누락으로 9개 URL 이 중복 페이지 처리되던 문제) 을 구현하며 남긴 내부 기록이다.
> 대응하는 `raw/` 원본이 없으므로 ingest 문서의 `Sources` 표기를 쓰지 않는다.
> **근거는 구현 저장소.** 코드 인용은 fe_be 저장소 브랜치 `feat/TNSP-1417-canonical-and-page-metadata`,
> 실측 수치는 그 브랜치를 `next build && next start` 로 띄워 로컬 DB 로 잰 값이다.
> 검토 후 등재한다면 `wiki/seo/` 가 자리이고, 기존 `wiki/seo/canonical-url-consolidation.md` 의 구현 편에 해당한다.
> 작성: 2026-09-08

## Overview

[canonical URL을 일관되게 지정하는 방법](canonical-url-consolidation.md)이 "무엇을 해야 하는가"라면, 이 글은 Next.js App Router에서 "어떻게 만드는가"다. 핵심 제약은 **루트 layout의 `generateMetadata`가 현재 요청 경로를 모른다**는 점이고, 이것을 middleware가 넘긴 요청 헤더로 푼다. 그 위에 metadata 상속 규칙, 페이지별 문구, 사이트맵 정리를 얹으면 사이트 전체가 self-canonical을 갖는다.

## 문제: 색인에서 빠지는 세 가지 경로

Search Console의 「사용자가 선택한 표준이 없는 중복 페이지」는 Google이 이 페이지의 대표 URL을 스스로 정하지 못했다는 뜻이다. 실제 사이트에서 관측된 원인은 셋이었고, 셋 모두 **사이트맵의 200 응답 URL 214개 중 canonical을 선언한 페이지가 하나도 없다**는 공통점에서 나왔다.

| 원인 | 무슨 일이 일어나는가 |
|---|---|
| 307 임시 리디렉션이 사이트맵에 실림 | 307은 "원래 URL을 계속 쓰라"는 뜻이다. Google이 출발지(`/ko/tour`)를 대표로 삼고 도착지(`/ko/tour/package`)를 중복으로 제외한다 |
| 쿼리스트링이 붙은 URL | `/stay`와 `/stay?page=1`의 본문이 같으면 파라미터가 붙은 쪽이 중복으로 제외된다 |
| 여러 페이지가 같은 title을 씀 | 고정 경로 70개 중 48개가 같은 title을 공유하면 Google이 서로 다른 페이지로 구분하지 못한다 |

self-canonical(자기 자신을 가리키는 canonical)은 셋 모두에 동시에 듣는다. 도착지가 자기를 대표로 선언하면 출발지와 경쟁하지 않고, 파라미터가 붙은 URL이 파라미터 없는 경로를 가리키면 중복이 합쳐진다.

## 왜 layout이 경로를 모르는가

App Router의 `generateMetadata`는 인자로 `params`(동적 세그먼트)와 `searchParams`(페이지 한정)만 받는다. **전체 pathname을 주는 인자는 없다.** 루트 `app/layout.tsx`는 모든 요청을 거치는 유일한 지점이지만, 정작 자기가 지금 어느 경로를 그리는 중인지 모른다.

Server Component에서 `headers()`는 읽을 수 있으므로, middleware가 경로를 헤더에 실어 넘기면 된다. Next.js는 이 목적의 헤더를 표준으로 제공하지 않아 직접 붙인다.

```ts
// src/middleware.ts
export function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl;

  // 클라이언트가 위조해 보낸 값은 먼저 지운다 — 이 헤더는 middleware만 채운다
  const fwd = new Headers(req.headers);
  fwd.delete('x-locale');
  fwd.delete('x-pathname');
  fwd.set('x-pathname', pathname);

  // ... 이후 모든 분기가 fwd를 그대로 넘긴다
  return NextResponse.next({ request: { headers: fwd } });
}
```

세 가지가 중요하다.

- **`delete` 먼저.** 클라이언트가 `x-pathname: /admin` 같은 헤더를 직접 보낼 수 있다. 신뢰 값으로 덮어쓰기 전에 지운다. `set`이 같은 이름을 이미 덮으므로 동작상 중복이지만, 이 헤더가 요청에서 온 값이 아니라는 의도를 코드에 남긴다.
- **`nextUrl.pathname`에는 쿼리스트링이 없다.** `?page=1`은 애초에 들어오지 않으므로 canonical에서 파라미터가 자동으로 떨어진다. 별도 처리가 필요 없다.
- **모든 반환 분기가 같은 `fwd`를 넘겨야 한다.** 한 분기라도 `NextResponse.next()`를 인자 없이 호출하면 그 경로만 헤더를 잃는다.

## metadata 병합 규칙

App Router의 metadata는 루트 layout에서 시작해 하위 세그먼트로 내려가며 병합된다. 규칙 두 개만 알면 된다.

**필드 단위로 덮어쓴다.** 하위가 선언한 필드만 교체되고 나머지는 상위 값이 남는다. 객체 안쪽을 깊게 합치지 않는다 — 하위가 `alternates`를 선언하면 상위의 `alternates` **전체**가 그 값으로 바뀐다.

**`title.template`은 하위의 문자열 title에 적용된다.** 루트가 `template: '%s | 사이트명'`을 두면 하위 페이지의 `title: '축전 개요'`는 `축전 개요 | 사이트명`으로 렌더된다. 접미사를 붙이고 싶지 않으면 `title: { absolute: '...' }`를 쓴다.

이 두 규칙 덕분에 루트 layout에 self-canonical을 한 번만 넣으면 사이트 전체가 덮이고, 예외가 필요한 페이지는 자기 `alternates`를 선언해 그 자리만 바꾼다.

```ts
// src/app/layout.tsx
const SITE = (process.env.NEXT_PUBLIC_SITE_URL ?? 'http://localhost:3000').replace(/\/$/, '');

async function selfCanonical(): Promise<string | null> {
  const pathname = (await headers()).get('x-pathname') ?? '';
  // 헤더 부재 = middleware matcher 제외 경로, /admin = 색인 대상 아님
  if (!pathname.startsWith('/') || pathname.startsWith('/admin')) return null;
  return `${SITE}${pathname === '/' ? '' : pathname}`;
}

export async function generateMetadata(): Promise<Metadata> {
  const canonical = await selfCanonical();
  return {
    title: { default: title, template: `%s | ${title}` },
    ...(canonical ? { alternates: { canonical } } : {}),
  };
}
```

예외 페이지는 자기 값을 선언한다. 아래는 축전 랜딩을 메인과 같은 문서로 통합하는 경우다.

```ts
// app/(public)/[locale]/festival/page.tsx
export async function generateMetadata({ params }) {
  const { locale } = await params;
  return { alternates: { canonical: `${SITE}/${locale}` } };  // 루트 값을 대체
}
```

`headers()`를 읽으면 그 라우트는 동적 렌더링으로 고정된다. 이미 `force-dynamic`인 사이트라면 손실이 없지만, 정적 생성에 의존하는 사이트라면 루트 layout 전체가 동적으로 바뀌는 대가를 먼저 따져야 한다.

## 페이지별 문구를 한곳에 모으기

자기 metadata가 없는 라우트는 루트의 사이트 공통 title을 그대로 물려받는다. 페이지가 늘수록 같은 title을 쓰는 URL이 함께 는다. 라우트마다 `generateMetadata`를 손으로 쓰면 문구가 파일 15개에 흩어져 검토와 수정이 어려워진다.

문구 표를 한 파일에 모으고 팩토리로 `generateMetadata`를 만들면 각 페이지는 한 줄로 끝난다.

```ts
// src/lib/page-meta.ts
const COPY: Record<string, Record<Locale, { title: string; description: string }>> = {
  'festival/overview': { ko: { ... }, en: { ... } },
  'people/preservation': { ko: { ... }, en: { ... } },
};

export function localeMeta(key: string, paramName?: string) {
  return async function generateMetadata({ params }): Promise<Metadata> {
    const p = await params;
    const locale: Locale = isLocale(p.locale) ? p.locale : 'ko';
    const fullKey = paramName ? `${key}/${p[paramName]}` : key;   // /people/[tab] → 'people/preservation'
    const copy = COPY[fullKey]?.[locale];
    return copy ? { title: copy.title, description: copy.description } : {};
  };
}
```

```ts
// 각 페이지
export const generateMetadata = localeMeta('festival/overview');   // 고정 경로
export const generateMetadata = localeMeta('people', 'tab');       // /people/[tab] — 4개 탭
```

문구를 정할 때 걸리는 함정 하나. **다국어 사이트에서 국문·영문 title이 같아지는 경우**를 확인해야 한다. 화면 라벨이 국문도 `FAQ`라고 해서 그대로 쓰면 두 로케일의 title이 완전히 같아져 로케일별 구분이 사라진다. 국문은 「자주 묻는 질문」처럼 다르게 잡는다.

## 사이트맵에서 리디렉션 URL 빼기

사이트맵은 "이 URL들을 색인해 달라"는 목록이다. 리디렉션으로 응답하는 URL을 넣으면 Google에게 출발지를 대표 후보로 제시하는 셈이 되고, 도착지와 경쟁시킨다. 출발지는 응답 본문이 없어 canonical을 선언할 수도 없다.

```ts
const STATIC_PATHS = [
  '',
  'festival/overview', 'festival/opening',
  // 섹션 대문 'tour' · 'people' · 'archive' · 'community' 는 하위 첫 페이지로 307만 하므로 넣지 않는다
  'tour/explore', 'tour/package',
  'people/preservation', 'people/symbol',
];
```

307을 308 영구 리디렉션으로 바꾸는 선택지도 있지만 신중해야 한다. 308은 브라우저가 영구 캐시해 되돌리기 어렵고, 그 경로에 나중에 실제 페이지를 만들 여지를 없앤다. 사이트맵에서 빼고 도착지에 self-canonical을 주는 편이 되돌릴 수 있는 조치다.

## 검증할 때 주의할 것

배포된 응답 HTML을 기준으로 확인한다. Search Console의 재검증은 Google의 재크롤링 주기에 달려 있어 수 주가 걸린다.

세 가지 함정이 있었다.

- **`next dev`로 재면 안 된다.** 개발 서버는 라우트를 첫 요청에서 컴파일하므로, 컴파일 중 응답에는 `<title>`이 비어 있을 수 있다. 실제로 dev에서 46개가 "title 없음"으로 잡혔지만 production build에서는 전부 정상이었다. `next build && next start`로 재야 한다.
- **macOS의 `sort`가 한글을 뭉갠다.** UTF-8 로케일에서 서로 다른 한글 제목을 동등하다고 보고 `uniq -c`가 합쳐 버린다. 「개막식」 21개 같은 거짓 중복이 나온다. 집계에는 `LC_ALL=C sort`를 쓴다.
- **자기 사이트의 레이트리밋에 걸린다.** middleware에 IP당 요청 제한이 있으면 전수 검사가 429를 받는다. 요청 사이에 간격을 둔다.

전수 검사는 이런 형태가 된다.

```bash
curl -s "$BASE/sitemap.xml" | grep -o '<loc>[^<]*</loc>' | sed 's/<[^>]*>//g' > urls.txt
while read -r u; do
  body=$(curl -s "$u")
  canon=$(printf '%s' "$body" | grep -o 'rel="canonical" href="[^"]*"' | head -1 | sed 's/.*href="//; s/"$//')
  title=$(printf '%s' "$body" | grep -o '<title>[^<]*</title>' | head -1 | sed 's/<[^>]*>//g')
  printf '%s\t%s\t%s\n' "$u" "$canon" "$title"
  sleep 0.45                      # 레이트리밋 회피
done < urls.txt > probe.tsv

LC_ALL=C sort probe.tsv | cut -f3 | LC_ALL=C uniq -c | awk '$1>1'   # title 중복
awk -F'\t' '$2 != $1' probe.tsv                                      # canonical 불일치
```

확인할 것은 네 가지다. 사이트맵 URL이 전부 200인가, 전부 canonical을 갖는가, canonical이 자기 URL을 가리키는가(의도한 통합은 예외), 쿼리스트링이 붙어도 canonical이 경로만 가리키는가.

## 남는 것

이 조치로 해결되지 않는 중복이 있다. **관리자가 입력한 콘텐츠 제목이 실제로 같은 경우**다. 자료실에 「병산서원 기록 사진」이 17개 있으면 코드는 DB 제목을 정상적으로 읽고 있을 뿐이고, 다국어 폴백 규칙(영문 미입력 시 한글 표시)까지 겹치면 국문·영문 쌍도 같아진다. 이건 코드가 아니라 콘텐츠 입력 문제라 운영 주체와 상의할 사안이다.

기술적 조치와 콘텐츠 조치를 구분하는 것이 중요하다. 섞으면 코드에서 못 고칠 것을 고치려다 확정된 데이터 구조를 건드리게 된다.

## See Also

- [Google Search에서 canonical URL을 일관되게 지정하는 방법](../wiki/seo/canonical-url-consolidation.md)
