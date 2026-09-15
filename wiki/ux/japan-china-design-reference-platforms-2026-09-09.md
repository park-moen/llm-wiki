# 일본·중국 디자인 레퍼런스 플랫폼 활용법

> Sources: [비디자이너를 위한 디자인 레퍼런스 도구함](non-designer-design-reference-toolkit-2026-09-09.md); [비디자이너를 위한 디자인 시안 가이드 작성법](ieve-design-reference-selection-guide-2026-09-09.md)
> Origin: 사용자 제공 일본·중국 디자인 레퍼런스 메모 (2026-09-09; not stored in `raw/`)
> Archived: 2026-09-09

## Overview

한국·미국 중심의 레퍼런스 목록에 일본과 중국을 더하면 CJK 조판, 동아시아권의 정보 밀도와 시각 표현을 폭넓게 비교할 수 있다. 일본은 실제 운영 사이트의 layout과 interaction을 찾는 gallery가 발달해 있고, 중국은 portfolio·UI 화면·moodboard 중심의 community가 그 역할을 나눠 맡는다. 두 지역은 같은 방식으로 찾기보다 결과물의 형태에 맞춰 활용한다.

## 일본: 실제 사이트의 layout과 interaction

일본 gallery는 실제 운영 중인 website를 업종, 분위기와 interaction 유형으로 나눠 보여 준다. Awwwards나 SiteInspire처럼 완성된 site를 탐색하되, CJK text가 들어간 화면을 직접 비교할 수 있다는 점이 특징이다.

| 사이트 | 적합한 용도 | 살펴볼 항목 |
|---|---|---|
| **SANKOU!** | 업종·분위기뿐 아니라 hover, slide와 carousel 같은 interaction 유형으로 사례를 좁혀 찾는다. | Motion의 목적, desktop·mobile 차이와 구현 비용을 함께 확인한다. |
| **MUUUUU.ORG** | 세로로 긴 branding site와 typography·사진 중심 사례를 찾는다. 행사·기업 site의 section 흐름을 비교하기 좋다. | 첫 화면만 보지 말고 정보가 쌓이는 순서와 CTA 반복 방식을 본다. |
| **ikesai.com** | 일본어 website 사례를 폭넓게 살펴보며 시기별·업종별 표현 차이를 확인한다. | 오래된 사례는 현재 운영 여부와 responsive 동작을 원본 site에서 다시 확인한다. |
| **Web Design Clip** | PC, smartphone과 세로형 사례를 나눠 보며 device별 구성을 비교한다. | 같은 service의 breakpoint별 화면인지, 서로 다른 사례인지 구분해서 본다. |

### 한글 시안에 일본 사례가 유용한 이유

영문 중심의 수상작은 alphabet의 글자 폭을 전제로 heading의 자간, 행간과 줄바꿈을 설계한 경우가 많다. 같은 영역에 한글을 넣으면 문장이 예상보다 길어지거나 줄 수가 늘어 hierarchy가 달라질 수 있다.

일본 사례도 한글과 완전히 같지는 않지만 CJK 문자를 사용한 조판 문제를 다룬 결과물이라는 점에서 직접적인 비교 대상이 된다. 다음 항목을 중심으로 본다.

- 긴 heading을 몇 줄까지 허용하는가
- 한자·가나·영문·숫자가 섞일 때 크기와 굵기를 어떻게 나누는가
- 좁은 mobile 화면에서 제목과 metadata를 어떤 순서로 배치하는가
- 정보량이 많아도 여백과 section 구분이 유지되는가
- Motion이 text 읽기를 방해하지 않는가

일본 사례가 서구권 사례를 항상 대체하는 것은 아니다. 국문 content가 많고 행사·기관 성격이 강한 화면에서는 조판 후보를 거르는 데 더 직접적인 근거가 될 수 있다.

## 중국: community와 시안 중심 레퍼런스

중국은 하나의 live site gallery보다 designer community, UI 전문 platform과 moodboard service가 서로 다른 용도를 담당한다. 실제 URL보다 완성 화면이나 concept image 형태의 작업물이 많으므로 시각 방향을 찾는 데 활용하고, 실제 동작과 사용성은 별도로 검증한다.

| 사이트 | 성격 | 적합한 용도 | 한계 |
|---|---|---|---|
| **站酷 ZCOOL** | 폭넓은 분야를 다루는 designer community | `网页` category에서 web design 시안, branding과 visual tone을 탐색한다. | 게시된 이미지가 실제 운영 site와 같다고 단정할 수 없다. |
| **UI中国** | UI·UX 중심 community | 화면 단위 UI, mobile pattern과 세부 표현을 비교한다. | 전체 flow와 edge case가 생략될 수 있다. |
| **花瓣网 Huaban** | Board 방식의 image curation service | 주제별 moodboard를 만들고 color·photography·composition 후보를 모은다. | 출처와 사용 조건이 분명하지 않은 image는 그대로 asset으로 사용하지 않는다. |
| **设计导航** | Design resource를 모아 둔 link directory | 목적에 맞는 중국 platform과 도구를 찾는 출발점으로 사용한다. | Directory 자체를 품질 검증이나 원출처로 보지 않는다. |

### Live site보다 시안 이미지가 많은 점을 고려한다

제공된 메모는 중국의 상업 traffic이 일반 web보다 app과 WeChat mini program으로 흐르는 생태계 특성을 배경으로 제시한다. 이 설명을 모든 중국 service에 적용되는 규칙으로 단정하지는 않는다. 실무에서는 다음 차이를 확인하는 기준으로 사용한다.

```text
일본 gallery  → 실제 site의 layout·interaction·responsive 동작 확인
중국 community → visual tone·화면 구성·moodboard 후보 수집
```

중국 community에서 찾은 시안은 다음 질문을 통과해야 구현 근거로 사용할 수 있다.

- 실제 service에 적용되거나 운영된 화면인가
- 앞뒤 화면과 사용자 flow를 확인할 수 있는가
- loading·empty·error 상태가 있는가
- Mobile과 desktop 가운데 어느 환경을 전제로 했는가
- Image, font와 icon을 실제 제품에서 사용할 권리가 있는가

## IEVE 같은 행사 사이트에 적용하는 순서

1. **SANKOU!와 MUUUUU.ORG에서 구조 후보를 찾는다.** Hero, 행사 개요, program, 참가 등록과 sponsor section의 순서를 비교한다.
2. **일본 사례로 CJK 조판을 점검한다.** 긴 행사명, 날짜·장소와 국영문 병기에서 줄바꿈이 무너지지 않는지 확인한다.
3. **花瓣网과 站酷에서 시각 방향을 넓힌다.** Color, photo treatment와 key visual 후보를 moodboard로 묶는다.
4. **실제 경쟁 행사 site로 동작을 검증한다.** Gallery와 시안 이미지에서 보이지 않는 navigation, form, responsive 동작과 예외 상태를 확인한다.
5. **가져올 것과 제외할 것을 기록한다.** 화면을 그대로 복제하지 않고 현재 content, 기술 제약과 접근성 기준에 맞춰 선택한다.

## 빠른 선택표

| 찾으려는 것 | 먼저 볼 곳 | 추가 검증 |
|---|---|---|
| Interaction과 page 흐름 | SANKOU!, MUUUUU.ORG | 원본 site의 mobile·keyboard·성능 확인 |
| CJK typography와 정보 밀도 | SANKOU!, ikesai.com, Web Design Clip | 실제 한글 content를 넣어 줄바꿈 확인 |
| Visual tone과 moodboard | 花瓣网, 站酷 | 원출처와 asset license 확인 |
| UI 화면의 세부 표현 | UI中国 | 전체 flow와 상태 화면 보완 |
| 중국 design resource 탐색 | 设计导航 | 각 원본 platform에서 정보 재확인 |

핵심은 국가별 platform의 차이를 순위로 판단하지 않는 것이다. 일본 gallery는 실제 동작과 CJK 조판을 확인하는 자료로, 중국 community는 visual exploration을 넓히는 자료로 사용한다. 최종 시안은 실제 content와 service flow를 넣어 별도로 검증한다.

## See Also

- [비디자이너를 위한 디자인 레퍼런스 도구함](non-designer-design-reference-toolkit-2026-09-09.md)
- [비디자이너를 위한 디자인 시안 가이드 작성법](ieve-design-reference-selection-guide-2026-09-09.md)
- [디자이너 없는 팀을 위한 AI 디자인 레퍼런스 가이드](non-designer-ai-design-reference-guide-2026-09-09.md)
