# 비디자이너를 위한 디자인 레퍼런스 도구함

> Sources: [디자이너 없는 팀을 위한 AI 디자인 레퍼런스 가이드](non-designer-ai-design-reference-guide-2026-09-09.md); [비디자이너를 위한 디자인 시안 가이드 작성법](ieve-design-reference-selection-guide-2026-09-09.md); [개발자를 위한 한국어 UX 도서 근거 가이드](developer-ux-book-evidence-guide.md)
> Origin: 사용자 제공 디자인 레퍼런스 메모 (2026-09-09; not stored in `raw/`)
> Archived: 2026-09-09

## Overview

디자이너가 아닌 개발자가 디자인 시안을 준비할 때는 한 종류의 레퍼런스만 보지 않는다. 실제 서비스의 전체 layout, 한국어 정보 밀도, component의 설계 이유, 기본 원칙과 구현용 asset을 서로 다른 출처에서 찾는다. Pinterest와 Dribbble은 분위기를 잡는 보조 자료로 제한하고, 실제 운영 사이트와 디자인 시스템 문서를 시안의 주된 근거로 삼는다.

## 레퍼런스는 목적에 따라 나눠 본다

```text
전체 화면과 흐름    → 실제 서비스 gallery
한국어 정보 밀도    → 국내 site와 경쟁 서비스
Component 판단 기준 → 디자인 시스템 문서
기본 원칙           → 개발자 대상 입문 자료
구현 재료           → font·icon·color·image asset
```

이 구분이 없으면 멋진 화면을 많이 모아도 무엇을 적용해야 할지 결정하기 어렵다. 레퍼런스를 저장할 때는 `왜 골랐는가`, `무엇을 가져오는가`, `무엇은 제외하는가`를 함께 기록한다.

## 1. 실제 사이트 갤러리: 전체 배치와 흐름

실제 운영되는 사이트를 수집한 gallery는 hero 하나가 아니라 page 전체의 정보 구조와 반복 pattern을 보는 데 적합하다.

| 사이트 | 적합한 용도 | 사용할 때 주의할 점 |
|---|---|---|
| [Refero](https://refero.design/) | 실제 서비스 화면을 page·component·pattern별로 찾는다. 가격표, FAQ, onboarding처럼 필요한 문제부터 좁혀 보기 좋다. | 개별 화면의 외형만 보지 말고 앞뒤 flow와 실제 제품의 목적을 함께 확인한다. |
| [Mobbin](https://mobbin.com/) | Mobile app과 web product의 실제 screen을 flow 단위로 비교한다. 가입, 결제, 검색과 설정 흐름을 볼 때 유용하다. | 유료 범위가 있으므로 필요한 platform과 flow를 얼마나 자주 조사할지 고려해 구독을 판단한다. |
| [Land-book](https://land-book.com/) · [Godly](https://godly.website/) | Landing page, 기업·행사 site의 전체적인 시각 방향과 section 구성을 찾는다. | 인상적인 hero보다 content 순서, CTA와 실제 mobile 화면을 함께 본다. |
| [SiteInspire](https://www.siteinspire.com/) | 산업, 유형과 style별로 site를 좁혀 차분한 기업형 방향을 비교한다. | 오래된 사례는 현재 동작과 responsive 상태를 원본 site에서 다시 확인한다. |
| [Awwwards](https://www.awwwards.com/) | 강한 visual concept, motion과 실험적인 interaction의 아이디어를 찾는다. | 표현이 화려한 사례가 많으므로 사용성·성능·접근성을 검토하지 않고 그대로 적용하지 않는다. |

갤러리는 정답 목록이 아니다. 같은 `dashboard`라도 사용자의 업무, data 밀도와 주요 행동이 다르면 적절한 구조도 달라진다.

## 2. 한국 사이트: 정보 밀도와 한글 typography

해외 레퍼런스만 보면 한국 사용자가 익숙한 정보량, 한글 줄바꿈과 공공·기업 사이트의 content 구조를 놓치기 쉽다. 국문 서비스라면 국내 사례를 별도 축으로 수집한다.

- [GDWEB](https://www.gdweb.co.kr/): 전시·공공·기업 등 분야별 국내 사례를 찾는다.
- [웹어워드코리아](https://www.i-award.or.kr/web/): 국내 수상작을 분야별로 비교한다.
- [DBCUT](https://www.dbcut.com/): 국내 web design 사례를 폭넓게 탐색한다.
- [Notefolio](https://notefolio.net/): 국내 창작자와 designer의 작업 과정, branding과 visual direction을 살핀다.

수상작과 portfolio는 품질 보증서가 아니다. 다음 항목을 관찰하는 출발점으로 사용한다.

- 한글 heading과 body text의 크기·행간
- 국문·영문이 함께 있을 때의 줄바꿈과 정렬
- 많은 메뉴와 content를 묶는 navigation 방식
- 기관·행사·기업 사이트에서 신뢰를 표현하는 방식
- Mobile에서 유지하는 정보와 제거하는 정보

## 갤러리보다 직접적인 경쟁 레퍼런스를 먼저 모은다

현재 제품과 비슷한 실제 서비스는 gallery의 인기 사례보다 더 직접적인 근거가 된다. 행사 site를 만든다면 CES, MWC, Web Summit, VivaTech, Slush와 서울모빌리티쇼처럼 규모와 목적이 다른 행사를 후보로 두고 비교한다.

처음에는 8~10개 정도를 후보로 모은 뒤, 다음 기준으로 줄인다.

| 비교할 항목 | 확인할 내용 |
|---|---|
| 첫 화면 | 행사의 성격, 일정·장소와 주요 CTA를 바로 알 수 있는가 |
| Program | 날짜·track·장소가 겹쳐도 탐색하기 쉬운가 |
| Exhibitor | 항목이 많을 때 검색·filter·분류가 작동하는가 |
| Registration | 단계, 비용, 오류와 완료 상태를 이해할 수 있는가 |
| Archive | 지난 회차와 현재 회차를 혼동하지 않는가 |
| Mobile | 현장에서 필요한 일정·장소·ticket 접근이 빠른가 |

페이지별 screenshot은 중요한 조사 자료지만 그 자체가 specification은 아니다. 각 screenshot에서 가져올 해결 방식과 제품 제약을 글로 연결해야 시안과 구현의 근거가 된다.

## 3. 디자인 시스템 문서: 개발자에게 맞는 판단 기준

디자인 시스템 문서는 component의 모양뿐 아니라 사용 시점, 상태, 접근성과 구현 방법을 함께 설명한다. 그림만 모았을 때 얻기 어려운 `왜 이렇게 하는가`의 근거를 제공한다.

| 디자인 시스템 | 우선 참고할 화면 |
|---|---|
| [Material Design 3](https://m3.material.io/) | 일반적인 web·Android UI, color·typography·상태 체계 |
| [Shopify Polaris](https://polaris.shopify.com/) | 상품·주문·설정처럼 운영 작업이 많은 admin UI |
| [Atlassian Design System](https://atlassian.design/) | 협업 도구, form, navigation과 복잡한 업무 흐름 |
| [IBM Carbon](https://carbondesignsystem.com/) | Enterprise product, table과 data 밀도가 높은 화면 |
| [GitHub Primer](https://primer.style/) | 개발 도구, repository와 설정처럼 조밀한 정보 화면 |
| [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines) | iOS·iPadOS·macOS의 navigation, input과 platform 관례 |

한 화면에 여러 디자인 시스템을 섞어 쓰기보다 현재 기술 stack과 제품 성격에 가까운 하나를 기본 문법으로 정한다. 나머지는 특정 문제를 해결하는 참고 자료로만 사용한다.

## 4. 기본 원칙을 빠르게 익히는 자료

Gallery를 따라 하기 전에 최소한의 판단 어휘를 익히면 레퍼런스의 장단점을 설명하기 쉬워진다.

- **Refactoring UI**: 개발자가 hierarchy, spacing, color와 typography를 실무 관점에서 배우는 입문 자료로 활용한다.
- **Practical UI**: UI를 구성하고 다듬는 구체적인 규칙을 확인한다.
- [Laws of UX](https://lawsofux.com/): 인지 부하, 선택지와 익숙한 pattern 같은 UX 원칙을 짧게 찾아본다.

어떤 책도 자동으로 좋은 시안을 만들어 주지는 않는다. 현재 작업 중인 화면 하나에 원칙을 적용하고, 실제 사용자 행동과 content로 검증해야 한다. 한국어 입문 자료는 [개발자를 위한 한국어 UX 도서 근거 가이드](developer-ux-book-evidence-guide.md)를 함께 참고한다.

## 5. 구현용 asset

Asset은 사용하기 쉬운지만 보지 말고 license, product tone과 기술 stack을 함께 확인한다.

| 종류 | 후보 | 확인할 점 |
|---|---|---|
| 한글 font | [눈누](https://noonnu.cc/), [Pretendard](https://github.com/orioncactus/pretendard) | 눈누의 각 font는 license가 다르므로 상업 이용과 webfont 허용 범위를 개별 확인한다. Pretendard도 배포 방식과 지원할 font weight를 정한다. |
| Icon | [Lucide](https://lucide.dev/), [Phosphor](https://phosphoricons.com/) | 한 제품에서는 stroke, fill과 크기 규칙을 통일하고 icon만으로 의미를 전달하지 않는다. |
| Color | [Radix Colors](https://www.radix-ui.com/colors) | 단계별 color scale을 구성할 때 유용하지만 접근성을 자동으로 보장하지는 않는다. 실제 foreground·background 조합의 contrast를 별도로 검사한다. |
| Image | [Unsplash](https://unsplash.com/), [Pexels](https://www.pexels.com/) | License와 인물·상표 노출을 확인한다. 행사·기관의 신뢰가 중요하면 generic stock image보다 실제 현장 사진을 우선한다. |

Asset을 정한 뒤에는 `font는 Pretendard`, `icon은 Lucide`처럼 이름만 적지 않는다. 사용할 weight, 기본 크기, stroke, color token과 예외를 함께 정의한다.

## 제품 유형별 최소 조합

### 행사·기관 website

```text
Land-book 또는 Godly
+ GDWEB·웹어워드코리아
+ 직접 수집한 유사 행사 site
+ Material Design 또는 기존 component library
```

Visual tone, 한글 정보 밀도와 실제 행사 flow를 따로 확인한다.

### Admin·CMS

```text
Refero
+ Polaris·Atlassian·Carbon·Primer 중 제품 성격에 가까운 하나
+ 실제 최대 data와 권한별 상태
```

멋있는 dashboard보다 table, form, bulk action, error와 권한 상태를 우선한다.

### Mobile app

```text
Mobbin
+ Apple HIG 또는 Material Design
+ 실제 onboarding·결제·설정 flow
```

Platform 관례와 실제 사용자 흐름을 먼저 맞춘 뒤 고유한 visual direction을 더한다.

## 레퍼런스 기록 양식

```markdown
## 사이트명 / URL

- 제품 유형:
- 참고할 page·flow:
- 해결하려는 문제:
- 가져올 것:
- 가져오지 않을 것:
- desktop·mobile 차이:
- loading·empty·error 상태:
- 실제 제품에 적용할 때의 위험:
- 연결할 화면 또는 요구사항:
```

`느낌이 좋다`에서 끝내지 않고 위치, 우선순위, 동작과 상태를 적는다. 예를 들어 `깔끔한 card`보다 `회차 label은 card 우상단에 두고, 종료 상태는 채도와 text를 함께 바꾼다`처럼 작성한다.

## 선택할 때의 최종 기준

- 실제 사용자가 해결하려는 문제와 연결되는가
- 이상적인 sample이 아니라 실제 data 규모를 견디는가
- 국문·영문 text와 mobile 화면에서도 유지되는가
- loading·empty·error·권한 상태를 확인할 수 있는가
- 기존 component와 기술 stack으로 구현할 수 있는가
- 접근성, 성능과 유지보수 비용을 감당할 수 있는가
- 가져올 점과 제외할 점을 다른 사람이 같은 뜻으로 이해할 수 있는가

레퍼런스의 수보다 **선택 이유를 설명할 수 있는가**가 중요하다. 갤러리는 후보를 찾는 도구이고, 디자인 시스템은 판단 기준이며, 실제 경쟁 서비스는 제품에 맞는 제약을 확인하는 근거다.

## See Also

- [비디자이너를 위한 디자인 시안 가이드 작성법](ieve-design-reference-selection-guide-2026-09-09.md)
- [디자이너 없는 팀을 위한 AI 디자인 레퍼런스 가이드](non-designer-ai-design-reference-guide-2026-09-09.md)
- [개발자를 위한 한국어 UX 도서 추천](developer-ux-book-recommendations-2026-08-21.md)
