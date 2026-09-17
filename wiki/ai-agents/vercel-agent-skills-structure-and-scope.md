# Vercel Agent Skills의 구조와 활용 범위

> Sources: Vercel Labs, Unknown
> Raw: [Vercel Agent Skills README](../../raw/ai-agents/vercel-agent-skills-readme.md)
> Updated: 2026-09-16

## Overview

`vercel-labs/agent-skills`는 AI coding agent의 특정 기술 능력을 확장하는 공식 Skill 모음이다. 각 Skill은 Agent Skills 형식을 따르는 지침과 선택적인 script·참고 문서로 구성된다. 저장소의 중심은 범용 개발 lifecycle을 강제하는 단일 workflow가 아니라 React·Next.js 성능, UI 검토, 문서 작성, React Native와 Vercel 운영·배포처럼 목적이 분명한 능력을 필요한 작업에 연결하는 데 있다.

## 저장소의 역할

Vercel은 Skill을 Agent 능력을 확장하는 지침과 script의 묶음으로 정의한다. 저장소의 각 Skill은 관련 작업이 감지됐을 때 agent가 사용할 수 있으며, 하나의 Skill 디렉터리는 다음 요소를 가질 수 있다.

| 구성 | 역할 |
|---|---|
| `SKILL.md` | Skill의 사용 시점과 수행 지침 |
| `scripts/` | 반복 실행이나 자동화를 위한 보조 script |
| `references/` | 본문에서 필요할 때 읽는 추가 문서 |

이 구조는 모든 지침을 항상 context에 넣는 방식보다 작업에 맞는 Skill과 참고 자료를 선택적으로 불러오는 데 적합하다. 다만 실제 context loading 방식과 자동 trigger의 세부 동작은 설치된 agent host에 따라 확인해야 한다.

## 제공하는 Skill의 범위

README에 기록된 Skill은 크게 네 영역으로 묶을 수 있다.

### React와 Next.js 구현 품질

- `react-best-practices`는 React·Next.js 성능 지침을 8개 범주와 40+개 규칙으로 제공한다. waterfall 제거와 bundle 크기를 가장 높은 우선순위로 두고 server·client data fetching, re-render, rendering과 JavaScript 최적화를 다룬다.
- `composition-patterns`는 boolean prop이 늘어나는 component를 compound component, state lifting과 내부 합성으로 재구성한다.
- `react-view-transitions`는 React View Transition API와 Next.js 연결, shared element transition과 접근성 처리를 다룬다.

### UI와 콘텐츠 검토

- `web-design-guidelines`는 접근성, focus, form, animation, typography, image, navigation, theme, touch와 국제화를 포함한 100+개 규칙으로 UI code를 검토한다.
- `writing-guidelines`는 Vercel writing handbook을 기반으로 voice, 구조, code sample, typography와 AI workflow를 포함한 80+개 규칙으로 문서와 산문을 검토한다.

### React Native

- `react-native-guidelines`는 7개 영역과 16개 규칙으로 React Native·Expo의 성능, layout, animation, image, state, architecture와 platform별 패턴을 다룬다.

### Vercel 운영과 배포

- `vercel-optimize`는 먼저 배포 지표를 수집한 뒤 비용·성능·신뢰성·cache·function 사용량과 billing 개선 지점을 조사한다. 모든 file을 무차별적으로 읽기보다 지표가 가리키는 route와 file로 조사 범위를 좁히는 방식이다.
- `vercel-deploy-claimable`은 project를 package하고 framework를 감지해 Vercel에 올린 뒤 preview URL과 소유권을 이전할 claim URL을 반환한다. `package.json`에서 40+개 framework를 자동 감지한다고 설명한다.

## 설치와 배포 구조

저장소 전체는 다음 명령으로 설치한다.

```bash
npx skills add vercel-labs/agent-skills
```

README는 설치 후 관련 작업이 감지되면 Skill이 자동으로 사용된다고 설명한다. 저장소의 `main`에서 Skill이 변경될 때마다 immutable GitHub release, discovery index와 Skill별 artifact를 발행한다. 이 구조는 registry나 설치 도구가 특정 시점의 artifact를 발견하고 배포할 수 있게 한다.

Immutable release가 곧 자동 update의 안전성을 보장하는 것은 아니다. 설치 전에 선택할 Skill의 `SKILL.md`, 포함된 script, 외부 통신과 실행 권한을 확인하고, 설치 후에는 실제로 연결된 version과 update 방식을 별도로 확인해야 한다.

## `vercel-labs/skills`와의 차이

두 저장소는 이름이 비슷하지만 역할이 다르다.

```text
vercel-labs/agent-skills
└── React·UI·문서·Vercel 운영에 사용하는 실제 Skill 모음

vercel-labs/skills
└── npx skills CLI와 find-skills 등 탐색·설치 생태계
```

`find-skills`를 설치하거나 Skills CLI의 동작을 조사할 때는 `vercel-labs/skills`를 보고, `react-best-practices` 같은 Vercel의 기술 Skill을 검토할 때는 `vercel-labs/agent-skills`를 본다.

## 적합한 사용 방식

Vercel Agent Skills는 다음과 같이 구체적인 기술 작업에 붙이는 편이 적합하다.

- React component를 작성하거나 성능 문제를 review한다.
- Next.js data fetching과 bundle 구성을 점검한다.
- UI의 접근성과 interaction을 검사한다.
- React Native·Expo 구현 패턴을 확인한다.
- Vercel 배포 지표를 바탕으로 비용과 병목을 조사한다.
- Vercel 배포를 수행한다.

반대로 실제 사용자 문제의 발견, 여러 architecture 대안의 비교, 전체 작업 계획과 완료 gate처럼 범용 software engineering workflow가 필요하면 별도의 planning·debugging·review Skill이나 repository 검증 체계를 결합해야 한다. 기술 Skill의 지침과 project의 성공 조건을 같은 것으로 간주하지 않는다.

## 권한과 검증 경계

검토 중심 Skill과 외부 상태를 바꾸는 Skill은 위험 범위가 다르다.

- `react-best-practices`, `composition-patterns`와 `web-design-guidelines`는 주로 code와 UI의 판단 기준을 제공한다.
- `vercel-optimize`는 배포 지표와 billing 관련 정보를 읽을 수 있으므로 접근 범위를 확인해야 한다.
- `vercel-deploy-claimable`은 project package를 외부 서비스에 전송하고 실제 배포를 만든다. 단순한 code 조언과 같은 권한으로 취급하지 않는다.

Skill이 test나 검토를 지시하더라도 agent의 자기 보고만으로 완료를 확정하지 않는다. Repository의 test·lint·typecheck·build와 필요한 실제 사용자 흐름을 별도의 검증 command나 CI에서 확인한다.

## 한계

이 문서는 저장소 README가 공개한 목적과 Skill 목록을 정리한 것이다. 각 Skill의 전체 `SKILL.md`, script 구현, agent host별 trigger 정확도와 실제 품질 향상 효과를 독립적으로 검증한 평가는 아니다. 저장소가 변경되면 Skill 이름과 기능도 달라질 수 있으므로 설치 전 현재 README와 대상 Skill 원문을 다시 확인한다.

## See Also

- [find-skills로 Agent Skill 탐색과 설치하기](find-skills-discovery-and-installation.md)
- [Vercel Agent Skills·Superpowers·gstack·Harness Engineering 비교](vercel-agent-skills-superpowers-gstack-harness-comparison-2026-09-16.md)
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
