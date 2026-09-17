# find-skills로 Agent Skill 탐색과 설치하기

> Sources: Vercel Labs, Unknown; skills.sh, Unknown; GODGOD126, 2026-03-20; kucherenko, 2026-05-29
> Raw: [Find Skills 원문](../../raw/ai-agents/vercel-find-skills.md); [Skills CLI README 발췌](../../raw/ai-agents/vercel-skills-cli-readme-extract.md); [Vercel Agent Skills README](../../raw/ai-agents/vercel-agent-skills-readme.md); [skills.sh 검색 index 누락 사례](../../raw/ai-agents/2026-03-20-skills-find-indexing-gap.md); [Skills 검색 관련도 사례](../../raw/ai-agents/2026-05-29-skills-find-search-relevance.md)
> Updated: 2026-09-16

## Overview

`find-skills`는 사용자의 요구를 설치 가능한 Agent Skill의 검색어로 바꾸고, `skills.sh`와 Skills CLI에서 후보를 찾은 뒤 품질을 검증해 제안하는 Skill이다. 독립 저장소가 아니라 `npx skills` CLI와 여러 Skill을 함께 관리하는 `vercel-labs/skills` 저장소의 `skills/find-skills/SKILL.md`에 있다. `skills.sh/vercel-labs/skills/find-skills`는 이 원문을 찾아보고 설치 정보를 확인하는 registry 페이지다.

## 세 링크가 가리키는 역할

```text
skills.sh/vercel-labs/skills/find-skills
    ↓ 탐색·배포용 registry 페이지
github.com/vercel-labs/skills
    ↓ Skills CLI와 여러 Skill을 관리하는 저장소
skills/find-skills/SKILL.md
    ↓ find-skills의 실제 지침 원문
```

따라서 `find-skills`만을 위한 별도 GitHub repository를 찾을 필요는 없다. 설치할 때는 상위 repository를 source로 지정하고 `--skill find-skills`로 그 안의 Skill 하나를 선택한다.

```bash
npx skills add https://github.com/vercel-labs/skills --skill find-skills
```

## `vercel-labs/agent-skills`는 별도 Skill 모음이다

`vercel-labs/skills`와 `vercel-labs/agent-skills`는 이름이 비슷하지만 역할이 다르다. 전자는 Skills CLI와 `find-skills`를 관리하고, 후자는 `react-best-practices`, `web-design-guidelines`, `composition-patterns`와 Vercel 운영·배포 Skill을 제공한다.

```text
vercel-labs/skills
└── Skill 탐색·설치 도구와 find-skills

vercel-labs/agent-skills
└── React·UI·문서·Vercel 관련 실제 Skill 모음
```

따라서 Skill을 찾는 절차를 설치하려면 `vercel-labs/skills`를, Vercel의 기술별 Skill을 설치하려면 `vercel-labs/agent-skills`를 source로 지정한다.

## find-skills가 해결하는 문제

Agent에게 막연하게 Skill 탐색을 맡기면 넓은 검색 결과를 그대로 추천하기 쉽다. `find-skills`는 다음 순서로 탐색과 추천을 분리한다.

1. 요구에서 domain과 구체적인 task를 찾는다.
2. `skills.sh` leaderboard에서 널리 쓰이는 후보를 먼저 확인한다.
3. 적합한 후보가 없으면 `npx skills find`로 keyword를 검색한다.
4. install count, source 평판과 GitHub stars를 확인한다.
5. 이름, 용도, install count, source, 설치 명령과 상세 링크를 함께 제시한다.
6. 사용자가 원할 때만 설치한다.

핵심은 검색 결과와 추천을 같은 것으로 취급하지 않는 데 있다. 원문은 install count가 `1K+`인 Skill을 우선하고 `100` 미만이면 주의하며, GitHub stars가 `100` 미만인 repository도 신중하게 평가하도록 안내한다. 이 수치는 절대적인 품질 보증이 아니라 후보를 거르는 초기 신호다.

## 검색어를 만드는 방법

Skills CLI는 keyword 또는 대화형 검색을 지원한다.

```bash
npx skills find typescript
```

특정 GitHub owner가 알려져 있다면 검색 범위를 제한할 수 있다.

```bash
npx skills find react --owner vercel
```

긴 질문을 그대로 검색하기보다 domain과 task를 짧은 keyword로 나누는 편이 `find-skills`의 설계에 가깝다.

| 자연어 요구 | 검색어 예시 |
|---|---|
| React application을 빠르게 만들고 싶다 | `react performance` |
| Pull request review를 돕는 Skill이 필요하다 | `pr review` |
| Changelog를 만들고 싶다 | `changelog` |

첫 검색이 빗나가면 동의어나 인접 용어로 다시 검색한다. 예를 들어 `deploy` 결과가 약하면 `deployment`나 `ci-cd`를 시도한다. 이름이나 source를 이미 안다면 free-text 검색을 반복하기보다 `--owner`로 범위를 줄이거나 repository의 Skill 목록을 직접 확인하는 편이 낫다.

## 검색 결과에는 누락과 순위 오류가 있을 수 있다

`npx skills find`와 `skills.sh` 검색 결과는 registry index에 의존하므로 source of truth가 아니다. 공식 저장소 issue에는 다음과 같은 사례가 보고됐다.

- 설치할 수 있고 `skills.sh` 상세 페이지도 존재하지만 `npx skills find`와 검색 API에는 나오지 않은 사례
- Skill 이름의 exact match보다 description 본문에 같은 단어가 들어간 다른 Skill이 먼저 노출된 사례

따라서 검색 결과가 없다고 Skill이 존재하지 않는다고 단정할 수 없고, 첫 결과라는 이유만으로 가장 적합하다고 판단할 수도 없다. GitHub repository, 실제 `SKILL.md`, source의 관리 상태를 함께 확인해야 한다.

## Project 설치와 Global 설치

Skills CLI는 기본적으로 현재 project에 설치하며, `-g` 또는 `--global`을 지정하면 사용자 범위에 설치한다. `--skill`은 repository 안에서 설치할 Skill을 선택하고, `--agent`는 연결할 coding agent를 선택한다.

| 목적 | 핵심 옵션 |
|---|---|
| 현재 project에서만 사용 | 기본값 |
| 모든 project에서 사용 | `--global` |
| repository에서 `find-skills`만 선택 | `--skill find-skills` |
| Codex에 연결 | `--agent codex` |
| 확인 prompt 생략 | `--yes` |

Codex에서 사용자 범위로 사용하려면 다음과 같이 범위와 agent를 명시한다.

```bash
npx skills add https://github.com/vercel-labs/skills --skill find-skills --global --agent codex --yes
```

공식 README는 Codex의 project 경로를 `.agents/skills/`, global 경로를 `~/.codex/skills/`로 안내한다. Skills CLI는 설치된 coding agent를 자동 감지하며, 감지하지 못하면 설치 대상을 선택하도록 요청한다. 설치 결과는 경로만 추측하지 말고 `npx skills list`로 scope와 연결된 agent를 확인하는 편이 안전하다.

## 안전한 사용 순서

```text
요구에서 domain·task 추출
    → leaderboard와 keyword 검색
    → 동의어·owner로 재검색
    → 실제 SKILL.md와 source 확인
    → install count·repository 신뢰도 검토
    → 후보와 근거 제시
    → 사용자 선택 후 설치
    → 설치 scope와 agent 연결 확인
```

`find-skills`는 좋은 Skill을 자동으로 보증하는 도구가 아니다. 검색 범위를 넓히고 후보 평가 절차를 빠뜨리지 않도록 만드는 탐색 workflow다.

## See Also

- [Vercel Agent Skills의 구조와 활용 범위](vercel-agent-skills-structure-and-scope.md)
- [Matt Pocock Skills의 Repository 설정 방식](matt-pocock-skills-repository-setup.md)
- [gstack으로 AI 개발 Workflow 이해하기](gstack-ai-engineering-workflow.md)
- [Claude Code ELI5 Skill로 쉬운 시각 설명 만들기](claude-code-eli5-skill.md)
